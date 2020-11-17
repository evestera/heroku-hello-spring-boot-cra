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
curl https://start.spring.io/starter.zip -d type=gradle-project -d language=kotlin -d platformVersion=2.4.0.RELEASE -d groupId=no.dossier -d artifactId=myapp -d dependencies=web -d javaVersion=1.8 -o starter.zip
unzip starter.zip
rm starter.zip
echo "Hello world" >src/main/resources/static/index.html
git init
git add .
git commit -m "Initial commit"
```

You could also create another kind of app from the list at https://devcenter.heroku.com/start

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
