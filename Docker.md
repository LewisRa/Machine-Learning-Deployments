# Shadow Mode ML Code - Analyse Results
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
 	apt-get install -y curl && \
 	# for postgres driver
 	apt-get install -y libpq-dev

RUN pip install -r /opt/app/requirements/requirements.txt
ENV PYTHONPATH "${PYTHONPATH}:/opt/app/"

ADD . /opt/app
WORKDIR /opt/app
```

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
      - "5000:5000"   # expose webserver to localhost host:container
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
