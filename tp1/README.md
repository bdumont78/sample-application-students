# TP DB Devops

## Database

Go in the folder db.

### Step 1 - Basics


```bash
docker build -t tp.db .
docker run -p 5432:5432 -e POSTGRES_PASSWORD=pwd tp.db
```

### Step 2 - Init database

Add the sql files in the local folder name (here dbFiles/) and copy those files in the container.
Add this line in the Dockerfile and rebuild the image container
```Docker
COPY dbFiles/ /docker-entrypoint-initdb.d/
```
### Step 3 - Persist Data

Run the container with the the "-v" argument to store the db state in a local folder (here datastore/)
```bash
docker run -p 5432:5432 -e POSTGRES_PASSWORD=pwd -v datastore:/var/lib/postgresql/data tp.db
```
Add "--name" flag to give a name to your container (with the -d flag to detach the container of the terminal)
```bash
docker run --name db-tp-pg -d -p 5432:5432 -e POSTGRES_PASSWORD=pwd -v datastore:/var/lib/postgresql/data tp.db
```
You will be able to start your container like this (don't need to add a -d flag because the container will be detched by default):
```bash
docker start db-tp-pg
```

## Backend API
Go in the backendApi folder.

### Step 1 - Basics
Go in simple-app folder
Create Dockerfile with this content:
```Docker
FROM openjdk:11
COPY Main.java .
RUN javac Main.java
CMD java Main
```

After that, compile it and run it:
```bash
docker build -t backend-api-tp .
docker run backend-api-tp
```

### Step 2 - Multistage Build
Go in simple-app folder
Create a dockerfile with this content:
```Docker
FROM openjdk:11
WORKDIR /usr/src/
COPY Main.java .
RUN javac Main.java

FROM openjdk:11
COPY --from=0 /usr/src/Main.class .
CMD java Main
````
After that, compile it and run it:
```bash
docker build -t backend-api-tp .
docker run backend-api-tp
```

### Step 3 - Backend simple app
Go in spring folder in the backend-api folder
After generating the spring app, adding the GreetingController file in the project and the Dockerfile:
```bash
docker build -t backend-simple-app-tp .
docker run --name backend-simple-app -p 8080:8080 backend-simple-app-tp
```

Add this line in the Dockerfile to cache the maven depencies (add the line before the mvn package command):
```Docker
RUN mvn dependency:go-offline
```

### Step 4 - Backend API
Go in simple-api folder in the backend-api folder

Modify the jdbc url in simple-api/src/main/resources/application.yml (replace the host with the db container name, don't forget to rebuild the image)

Create a network:
```bash
docker network create -d bridge tp1-bridge
```

In your db folder, run your conatiner as:
```bash
docker run --name db-tp-pg -d -p 5432:5432 --network tp1-bridge -e POSTGRES_PASSWORD=pwd -v datastore:/var/lib/postgresql/data tp.db
```
The --network flag will connect the conatiner in the selected network (here tp1-bridge)

In your simple-api folder, run your container as:
```bash
docker run --name backend-simple-api --network tp1-bridge -p 8080:8080 simple-api 
```

Your simple-api is able to connect with your db.

List of endpoint:
/departments/ : get all departments
/departments/{departmentNAme}/ : get department with the given name
/departments/{departmentNAme}/students/ : get all students of a given department
/departments/{departmentNAme}/count/ : get the number of students of a given department

/ : print hello world and prints the number of time you reached this endpoint

/students/ : get all students
/students/{id} : get student by a given id
/students/ (POST): add new student with the given params (same as defined in the class student)
/students/{id} (PUT): modify an existing student with the given params and the id
/students/{id} (DELETE) : delete a student with the given id

## Http Server
Go in the httpServer folder.
### Step 1 - Basics

Create Dockerfile with:
```Docker
FROM httpd:2.4.43-alpine
WORKDIR /usr/local/apache2/htdocs/
COPY index.html .
```
then build the image and run it like this:
```bash
docker run --name simple-http-server -p 80:80 simple-http
```
Now you are able to reach yor http server [here](http://localhost)

### Step 2 - Configuration

To get the default apache configuration, you have to run this command:

```bash
docker exec simple-http-server cat /usr/local/apache2/conf/httpd.conf
```

### Step 3 - Reverse proxy

After you started the db and the simple-api, you have to add a COPY and RUN command in your http server Dockerfile (to copy the new apache conf):
```Docker
COPY custom.conf .
RUN cat custom.conf > /usr/local/apache2/conf/httpd.conf
```

After that, rebuild and run it in the samed network as:
```bash
docker run --name simple-http-server --network tp1-bridge -p 80:80 simple-http
```

Now you can reach the backend-api via this [url](http://localhost)

## Link application
### Step 1 - Docker-compose

See Docker-compose.yml at root of the project
to build and start the app, you can do as:
```bash
docker-compose up --build
```
(if you have already build it, you can remove the "--build" flag).

other useful commands:
```bash
docker-compose build
docker-compose start
docker-compose stop
```

(and many others)

Now you can reach the backend-api via this [url](http://localhost)

### Step 2 - Docker-compose