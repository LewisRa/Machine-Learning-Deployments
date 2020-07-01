# Shadow Mode ML Code - Analyse Results

Docker containers wrap a pieceof softwarein a complete **file system** (think Linux filesystem) that containes everything needed to run
- code
- runtime
- system tools
- system libraries 
- **everything that can be install on a server**

In other words, you can imagine Docker as small little servers that are running in your laptop that containes everything need to run operating system and program. (Application server vs database server)

Container include the application and all of its dependencies but share the kernel with other containers runnin as isolated proceses in **user space** on the host operation system

Docker is effiicent because it shares the kernel betweem containers

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
