## Option 1: Get started with this repo as a template:

Install Heroku CLI: https://devcenter.heroku.com/articles/heroku-cli

```
git clone https://github.com/evestera/heroku-hello-spring-boot-cra.git myapp
cd myapp
heroku login
heroku create
git push heroku master
heroku open
```

(`heroku login` will let you create a Heroku account if you don't have one.
If it stalls after saying it has logged you in, just Ctrl+C)


## Option 2: Start from scratch (same result, but will give you a better understanding):

### 1. Create a Spring Boot Backend

Create a Spring Boot app using start.spring.io:

```
mkdir myapp
cd myapp
curl https://start.spring.io/starter.zip -d type=gradle-project -d language=kotlin \
  -d platformVersion=2.4.0.RELEASE -d groupId=no.dossier -d artifactId=myapp \
  -d dependencies=web,devtools -d javaVersion=1.8 -o starter.zip
unzip starter.zip
rm starter.zip
echo "Hello world" >src/main/resources/static/index.html
git init
git add .
git commit -m "Initial commit"
```

Reminder: `gradle tasks` will show you available tasks, e.g. `gradle bootRun`
which will build and run the application.

You could also create another kind of app from the list at
https://devcenter.heroku.com/start

### 2. Deploy with heroku

Install Heroku CLI: https://devcenter.heroku.com/articles/heroku-cli

```
heroku login
heroku create
git push heroku master
heroku open
```

(`heroku login` will let you create a Heroku account if you don't have one.
If it stalls after saying it has logged you in, just Ctrl+C)

### 3. Add a frontend

Run `create-react-app` to create an app:

```
npx create-react-app frontend --template typescript
```

Add a `frontend/build.gradle.kts`:

```
plugins {
    id("com.github.node-gradle.node") version "2.2.4"
}

node {
    download = true
    version = "12.19.1"
}

tasks.getByName("yarn_build").dependsOn("yarn_install")
```

<details>
<summary>Expand for further notes on `frontend/build.gradle.kts`</summary>

To allow Gradle to cache the frontend build we need to tell it about the
inputs and outputs, i.e. use this instead of the last line:

```
tasks.getByName("yarn_build") {
    dependsOn("yarn_install")
    inputs.dir("src")
    inputs.files("package.json", "yarn.lock")
    outputs.dir("build")
    outputs.cacheIf { true }
}
```

This is not necessary as such, but it will make some builds faster.
</details>

Add this to the root `build.gradle.kts`:

```
tasks.withType<ProcessResources> {
    dependsOn(":frontend:yarn_build")
    from("frontend/build/") {
        into("static")
    }
}
```

And this to `settings.gradle.kts`:

```
include("frontend")
```

Delete any existing `index.html` (if created earlier):

```
rm src/main/resources/static/index.html
```

Update heroku deploy:

```
git add .
git commit -m "Add frontend"
git push heroku master
heroku open
```

The following steps have not been applied to the repo:

### 4. Add a database

Add a database to your heroku app:

```
heroku addons:create heroku-postgresql:hobby-dev
```

Add these to the `dependencies`-block of `build.gradle.kts`:

```
    implementation("org.springframework.boot:spring-boot-starter-jdbc")
    implementation("org.postgresql:postgresql")
    implementation("org.flywaydb:flyway-core")
```

Add a migration for the initial tables you want:

```
mkdir -p src/main/resources/db/migration
touch src/main/resources/db/migration/V001__create_table_nodes.sql
```

You may want to set up a local database to test with, e.g.

```
psql postgres -c "create user myapp with password 'myapp';"
createdb --owner=myapp myapp
```

To use this database locally you can provide the details as environment variables, e.g.

```
SPRING_DATASOURCE_URL=jdbc:postgresql://localhost:5432/myapp \
SPRING_DATASOURCE_USERNAME=myapp \
SPRING_DATASOURCE_PASSWORD=myapp \
gradle bootRun
```

...or use spring profiles and use an environment variable to activate the profile, e.g.

`src/main/resources/application-local.properties`:

```
spring.datasource.url=jdbc:postgresql://localhost:5432/myapp
spring.datasource.username=myapp
spring.datasource.password=myapp
```

```
SPRING_PROFILES_ACTIVE=local gradle bootRun
```

(IDEA run configuration for Spring Boot also has an "Active profiles" field)
