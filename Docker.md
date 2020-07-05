# Docker

Docker containers wrap a pieceof softwarein a complete **file system** (think Linux filesystem) that containes everything needed to run
- code
- runtime
- system tools
- system libraries 
- **everything that can be install on a server**

In other words, you can imagine Docker as small little servers that are running in your laptop that containes everything need to run operating system and program. (Application server vs database server)

Container include the application and all of its dependencies but share the kernel with other containers runnin as isolated proceses in **user space** on the host operation system

Docker is efficcent because it shares the kernel between containers

A Docker file is a text file that creates some instructions  on how to create the container. The command you run to make create an image from the Docker file is **docker build** ---------------------> A Docker image is a binary file that lives in your in your file system and it's basically the blueprint of the  container you want to run so it has all the say the instructions and the and the components required to build that container. For a Docker image to  to become a an actual container that you can run, you should run the command **docker run**. Docker run convert image  to an actual container like an actual server.  --------------------->  Now you have a Docker container. The very first time you start  the container are you going to use **docker run <image> ** because the container is being executed against an image. After the first, you can use **docker start** and **docker stop**

To remove docker images: docker rmi <image>
To remove docker containers: docker rm <container>

## docker ps - all running containers
## docker ps -a - all available containers
## docker images - all available images
## Foreground vs background container
A foreground container is a container that starts the process and attaches the console or your terminal to thE container's process/bash/terminal.So, a foreground container is a container that you can interact with in real time. Then you have background containers which are associated with deamons (like deamon scripts). If you sny yo interact with background container, **docker exec** may help temporaily allow you to run container terminal.The difference between foreground containers can be accessed right after it is started. 
- The -v flag mounts a local container on the container
- The -i flag means there is an interative communications channel to that container
- The -t means there is a terminal open to that container
- The -d flag means the container is a detached or background container. (daemon)
- The -p flag allows you to pass in  a port  to the local container (which is port 80) and map port 80 back to the local host  so we can hit the web server from our browser in the laptop(localhost:80).

``` docker run -it ubantu:16:04 **/bin/bash**
(open up terminal in container to be interactive with  through localhost--- the terminal automatically opens up?)
exit (to shut docker down using open ubantu terminal)
```
```
docker run -d -p 80:80 --webserver nginx (nginx is now running as a deammon or a background service until you shut down down)
...
docker stop webserver
```
```

---
## Basic Flask App
- Just using python hello.py
- Write the Dockerfile
- Build the image with docker build -t hello-app .
- Run the container with docker run -d -p 5000:5000 --name hello-server hello-app
- Check that the container is running
- Go to localhost:5000. You should see Hello, World!
- Stop the container using docker stop hello-server
- Check that the container is not running with docker ps
- Check the container is still available with docker ps -a
- You can restart the server using docker start hello-server
- You can log in to the server using docker exec -it hello-server bash
- Caveat: if you change the code, it's not reflected on the container. We'll fix that next.
```
#### hello.py
```
from flask import Flask
app =  Flask(__name__)

@app.route('/')
def hello_world():
 return 'Hello World'
```
#### requirements.txt
```
Flask==0.11
```
#### DOCKERFILE
```
FROM python:3.4.5-slim

## make a local directory
RUN mkdir /opt/hello_app

# set "hello_app" as the working directory from which CMD, RUN, ADD references
WORKDIR /opt/hello_app

# copy the local requirements.txt to the /hello_app directory
ADD requirements.txt .

# pip install the local requirements.txt
RUN pip install -r requirements.txt

# now copy all the files in this directory to /hello_app directory
ADD . .

# Listen to port 5000 at runtime
EXPOSE 5000

# Environment variable that sets default app to be run by Flask
ENV FLASK_APP=hello.py

# Define our command to be run when launching the container
CMD ["flask", "run", "--host", "0.0.0.0"]


---
## When specifying the root user credentials for mysql container if you are using docker compose
--
## Docker - Production
```diff
FROM python:3.7.5-slim-buster

RUN mkdir -p /opt/app
COPY requirements /opt/app/requirements
RUN pip install --upgrade pip

# ensure we can run the make commands
RUN apt-get update -y && \
 	apt-get install -y make && \
 	apt-get install -y libffi-dev gcc && \
 	# for swagger
 	apt-get install -y curl && \
 	# for postgres driver(only need this download when we are running our API container but not for tests)
> 	apt-get install -y libpq-dev

RUN pip install -r /opt/app/requirements/requirements.txt
ENV PYTHONPATH "${PYTHONPATH}:/opt/app/"

ADD . /opt/app
WORKDIR /opt/app
```
#### make - make is a utility for building and maintaining groups of programs (and other types of files) from source code.
#### The libffi library is useful to anyone trying to build a bridge between interpreted and natively compiled code. Some notable users include CPython 
#### curl for swagger which UI allows anyone — be it your development team or your end consumers — to visualize and interact with the API’s resources without having any of the implementation logic in place. 
#### libpq-dev =Header files and static library for compiling C programs to link with the libpq library in order to communicate with a PostgreSQL database backend. 

#### PYTHONPATH is how to import custom python files/modules: https://www.youtube.com/watch?v=A7E18apPQJs
#### How do you add a path to PYTHONPATH in a Dockerfile: https://stackoverflow.com/questions/49631146/how-do-you-add-a-path-to-pythonpath-in-a-dockerfile
import sys
sys.path
create enviroment variables PYTHONPATH = /opt/app/ so that python will run .py files in opt/app/ like modules. 
#### ENV PYTHONPATH "${PYTHONPATH}:/your/custom/path"



```diff
version: '3'
services:
  ml_api:
    build:
      context: ../
      dockerfile: docker/Dockerfile
    environment:
      DB_HOST: database
      DB_PORT: 5432
      DB_USER: user
      DB_PASSWORD: ${DB_PASSWORD:-password}
      DB_NAME: ml_api_dev
    depends_on:
      - database
    ports:
>      - "5000:5000"   # expose webserver to localhost host:container
    command: bash -c "make db-migrations && make run-service-development"

  database:
    image: postgres:latest
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
      POSTGRES_DB: ml_api_dev
    ports:
      # expose postgres container on different host port to default (host:container)
>      - "6609:5432"
    volumes:
>      - my_dbdata:/var/lib/postgresql/data

volumes:
  my_dbdata:
```
---
## Docker - Testing

```
FROM python:3.7.5-slim-buster

RUN mkdir -p /opt/app
COPY requirements /opt/app/requirements
RUN pip install --upgrade pip

# ensure we can run the make commands
RUN apt-get update -y && \
 	apt-get install -y make && \
 	apt-get install -y libffi-dev gcc && \
 	# for swagger
 	apt-get install -y curl

ENV PYTHONPATH "${PYTHONPATH}:/opt/app"
RUN pip install -r /opt/app/requirements/test_requirements.txt

ADD . /opt/app
WORKDIR /opt/app
```

```
version: '3'
services:
  ml_api_test:
    image: christophergs/ml_api:master
    build:
      context: ../
      dockerfile: docker/Dockerfile.test
    environment:
      DB_HOST: test_database
      DB_PORT: 5432
      DB_USER: test_user
      DB_PASSWORD: ${DB_PASSWORD:-password}
      DB_NAME: ml_api_test
    depends_on:
      - test_database
    ports:
      - "5000:5000"   # expose webserver to localhost host:container
    command: bash -c "make db-migrations && make run-service-development"

  test_database:
    image: postgres:latest
    environment:
      POSTGRES_USER: test_user
      POSTGRES_PASSWORD: password
      POSTGRES_DB: ml_api_test
    ports:
      # expose postgres container on different host port to default (host:container)
>      - "6608:5432"
    volumes:
      - my_dbdata_test:/var/lib/postgresql/test_data

volumes:
  my_dbdata_test:
```
