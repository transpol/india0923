# Creating a simple website with Linux, Docker, and flask

## Install Docker and Login to Docker Hub

* Install docker on your computer or on a Linux VM
* Create a Login at Docker Hub
* In the command line ...

```bash
docker login
```

## Create a folder

Create a folder for the experiments and change into the folder:
```bash
mkdir docker
cd docker
```

## Check if Docker is running

Check if docker is installed. Enter in the command line (or power shell):

```bash
docker --version
```

Test docker:
```bash
docker run hello-world
```

More info about your Docker:
```bash
docker info
```

## Pulling an Image from Docker Hub

* Go to http://hub.docker.com/
* login
* search for Ubuntu
* What is the last version of Ubuntu?

So let us pull it from Dock Hub to your computer.

On you command line:
```bash
docker pull ubuntu
```

## List all Images

```bash
docker image ls
```

## List all Container

List all running containers:
```bash
docker container ls
```

List all containers:
```bash
docker container ls -a
```

## Running, stoping, and Attaching an Image

Starts Docker container and ends it right away:
```bash
docker run ubuntu
```

If you check the running containers, you see that there is no running container:
```bash
docker container ls
```

Starts Docker container and executes echo command:
```bash
docker run ubuntu echo "Hello BIPM"
```

Starts a Docker container and an interactive Bash. `-it` is short for `--interactive --tty` and will run it in the interactive mode:
```bash
docker run -it ubuntu bash
```

Some images have as a command line not Bash but a normal Shell `sh`. Normally you can always get the command line of a container.

You are now in the container and can interactively run commands:
```bash
pwd
```
```bash
ls
```
```bash
cd ~
```
```bash
ls
```
```bash
touch iwashere.txt
```
```bash
ls
```
To log out of the container, enter
```bash
exit
```

Now again the container is stopped.

If we login again,
```bash
docker run -it ubuntu bash
```
and go to your home directory
```bash
cd ~
```
check if the data is still there:
```bash
ls
```
The file is gone!

Check all containers.
```bash
docker container ls -a
```
```bash
CONTAINER ID        IMAGE                              COMMAND                  CREATED              STATUS                          PORTS               NAMES
a4cf0a5f41e7        ubuntu                             "bash"                   About a minute ago   Exited (0) 5 seconds ago                            objective_yonath
94cb0399ba60        ubuntu                             "bash"                   2 minutes ago        Exited (0) About a minute ago                       angry_goldstine
22863f7d24c0        ubuntu                             "bash"                   3 minutes ago        Exited (0) 3 minutes ago                            condescending_elgamal
```

The most right columnn you find `NAMES`. These names are automatically created. If we use the parameter `--name` we can name the container also by yourselfs.
You can also use the `CONTAINER ID` (first column) to identify a container  (e.g. a4cf0a5f41e7). There you do not have to write out the whole container ID but only use as many digits that are needed for identifying the container (normally two digits are sufficient)

Look not for the last but the the second last container (in this example `angry_goldstine` as NAME or `94cb0399ba60` as CONTAINER ID)

Restart the container (change the name or use the CONTAINER ID)
```bash
docker start angry_goldstine
```

Alternatively you could also use the CONTAINER ID (94cb0399ba60 in my case).
```bash
docker start 94cb0399ba60
```

You can als just use the first numbers of the CONATINER ID as long as they identify the container.
```bash
docker start 94
```

Login to an already running container with `docker exec` (change the CONTAINER ID);
```bash
docker exec -it 94 bash
```

`Ctrl+p` + `Ctrl+q` will turn the  interactive mode into a detached background (daemon) mode. That means you log out without stopping the container.

Check if the container is still running:
```bash
docker container ls
```

Login again. Alternatively to `docker exec -it` you can also use (change the CONTAINER ID)
```bash
docker attach 94
```

Go to your home directory
```bash
cd ~
```
check if the data is still there:
```bash
ls
```

The file is still here!

Use `Ctrl+p` + `Ctrl+q` to logout without stopping the container,

Check if the container is still running:
```bash
docker container ls
```

Stop the container if it is stil running (change 94cb0399ba60 to your container ID)
```bash
docker container stop 94cb0399ba60
```

Check if the container is still running:
```bash
docker container ls
```

## Deleting Containers

Check all containers:
```bash
docker container ls -a
```
Containers that you do not need anymore can be delete. This will delete all data on the container (change 94cb0399ba60 to your container ID)
```bash
docker container rm 94cb0399ba60
```

Check all containers:
```bash
docker container ls -a
```

You can delete all containers with
```bash
docker container prune
```
This is a dangerous command :-).

```bash
docker container ls -a
```
Everything gone.

## Containers and images

Let us start three different containers from the same image.
```bash
docker run -it -d --name ubuntu1 ubuntu bash
```
```bash
docker run -it -d --name ubuntu2 ubuntu bash
```
```bash
docker run -it -d --name ubuntu3 ubuntu bash
```

Meaning of the parameters:
* `--name` gives a container a name (e.g. ubuntu1)
* `-d` Detach: detaches the command line from the container so it is running in the background.


Check if all three containers running:
```bash
docker container ls
```

Start three more terminals, so that in total you have four terminals open.

In the second terminal, Enter
```bash
docker attach ubuntu1
```
In the third terminal, Enter
```bash
docker attach ubuntu2
```
In the fourth terminal, Enter
```bash
docker attach ubuntu3
```
In the termina of container ubuntu1 create a folder
```bash
mkdir iwashere
```
and check if it is there:
```bash
ls
```

In the termina of container ubuntu2 check if the folder `iwashere` is there:
```bash
ls
```

That means all containers are isolated.

Stop in the first terminal the ubuntu1 container.
```bash
docker container stop ubuntu1
```
What happens with your second terminal (where you are logged into ubuntu1)?

Stop all other containers.

## First Dockerfile - Start a simple website with nginx

Now we are building our first Docker image by means of a Dockerfile (named exactly like that), contents below. But wait, first we create a directory for our new project, 
i.e. simple_html and change to that directory with cd simple_html.
The Dockerfile:
```bash
FROM nginx:1.11-alpine
COPY index.html /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```
The first row is the "base image", a Linux-System with the nginx-webserver. Version 1.11-alpine guarantees stability. 
In the second row we are copying our index.html (which we should have) into the corresponding path in our docker container 
(the default of nginx for the website). 
In the third row Port 80 is opened to the outside. 
nginx is started as a background process in the last row.

Build the image:
```bash
docker build -t sit-server .
```
Don't forget the dot at the end ... what is it good for?

Now we can start the container (with Port-Forwarding) ... 
```bash
docker run -d -p 82:80 sit-server
```
The argument "-d" is for "detached mode" meaning that the container will run in the background. "-p 80:80" forwards the containers port to the hosts port. 

Now lets see the result ... 
```bash
<IP-oder-Web-Adresse>:<PORT-Nummer>
```

## Second Dockerfile - Create a simple REST Service with Java (JSON, Java, Maven, Spring Boot)

* For the following we need a backend REST Service ... and here is a tutorial to create a Java Rest Service with Java, Spring Boot and Maven: https://spring.io/guides/gs/rest-service/
* The Jar-File can be deployed in a docker-container

Here the Dockerfile:
```bash
FROM openjdk:17-jdk-slim
WORKDIR /app
COPY rest-service-complete-0.0.1-SNAPSHOT.jar /app
EXPOSE 8080
CMD ["java", "-jar", "rest-service-complete-0.0.1-SNAPSHOT.jar"]
```
Build it:

```bash
docker build -t sit-rest .
```

Run it:

```bash
docker run -d -p 8080:8080 sit-rest
```

docker run -d -p 8080:8080 sit-rest

The above mentioned JAR is provided in this Git-Repo ...

## Create Flask-App (Python-based, Docker-Projekt)

Legen Sie ein Verzeichnis für ein neues Docker-Projekt an. In diesem Verzeichnis: Create app.py (Text-Datei) with the following content:

```bash
from flask import Flask, render_template
import requests

app = Flask(__name__)

@app.route('/')
def home():
    response = requests.get('http://<IP-ADDRESS OF OWN MACHINE OR ubuntu22-001.lehre.hwr-berlin.de>:8080/greeting')
    todo = response.json()
    return render_template('index.html', todo=todo)

if __name__ == '__main__':
    app.run(debug=True, host='0.0.0.0')
```

This code defines a Flask application with a single route that consumes the REST API at *IP-Adresse:Port-Nummer* and passes the returned JSON object to an HTML template called index.html.

This is the content of templates/index.html (Text-Datei in Unterverzeichnis templates):

```bash
<!DOCTYPE html>
<html>
<head>
    <title>Flask Frontend</title>
</head>
<body>
    <h1>Todo</h1>
    <p>ID: {{ todo['id'] }}</p>
    <p>Content: {{ todo['content'] }}</p>
</body>
</html>
```

And this is the content of the Dockerfile (Text-Datei):
    
```bash
FROM python:3.8-slim-buster

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 5000

CMD ["python", "app.py"]
```

Ahhh, wait, requirements.txt (Text-Datei, for Python to work) is missing:

```bash
Flask==2.0.2
requests==2.26.0
```

This can built like before ...

```bash
docker build -t sit-frontend .
```

Deploy ...
```bash
docker run -d -p 5000:5000 sit-frontend
```

Now open localhost:5000 (or any other host on which you have deployed).
