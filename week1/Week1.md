# Week 1



## Docker + Postgres

#### 1: Create `Dockerfile` that would be used to build Docker image

`Dockerfile`  is a text document containing commands that can be run on CLI to create a **Docker image** 

Most Dockerfiles start from a parent image. If you need to completely control the contents of your image, you might need to create a base image instead. Here’s the difference:

- A [parent image](https://docs.docker.com/glossary/#parent-image) is the image that your image is based on. It refers to the contents of the `FROM` directive in the Dockerfile. Each subsequent declaration in the Dockerfile modifies this parent image. Most Dockerfiles start from a parent image, rather than a base image. However, the terms are sometimes used interchangeably.
- A [base image](https://docs.docker.com/glossary/#base-image) has `FROM scratch` in its Dockerfile.

##### a. Dockerfile

Docker commands used in the Dockerfile:

`FROM python:3.9`:  installs the base image for the image, 

`RUN pip install pandas` will the run any commands that, 

`WORKDIR /app` will create directory where the files of the image will be stored, 

`COPY pipeline.py pipeline.py` will copy the files from the local pc directory where the file is to the WORKDIR

`ENTRYPOINT  ["python","pipeline.py"]` directive in a Dockerfile allows the containers created from the image to run an executable after creation. ENTRYPOINT has two forms *exec* form and *shell* form. Here in the basic Dockerfile the *exec* form of entry point is being used.

##### b. Building the image

`docker build -t test:pandas` : this command was used to build the `test` image with a tag of `pandas`

##### c. running a container of the image

`docker run -it test:pandas 2021-05-15` to this parameters can be passed as the `pipeline.py` is able to accept parameters using `sys.argv`.

One the container is run the only purpose of this container is to run the `pipeline.py` and the container closes once the execution of the container is completed.

#### 2. Postgres

for running a postgres container the following command is used

``` bash 
docker run -it \
 -e POSTGRES_USER="root" \
 -e POSTGRES_PASSWORD="root" \
 -e POSTGRES_DB="ny_taxi" \
 -v  ny_taxi_postgres_data:/var/lib/postgresql/data \
 -p 5432:5432 \
 postgres:13
```

volumes is a way to map the folder on the container to the host machine filesystem to physically store the records.

Docker doesnt keep the state next time when the container is started data must be available. We need to mount the folder on the host machine to the docker container. 

Port on the host machine must also be mapped to the port on the docker container to access the Postgres database

the `docker run` command looks for an image of `postgres:13`  if it does not find one then downloads it and creates an image and then runs a container

Side note: if the port is busy  us this to find the process at running at the port[[url](https://stackoverflow.com/questions/38249434/docker-postgres-failed-to-bind-tcp-0-0-0-05432-address-already-in-use)]
`sudo ss -lptn 'sport = :5432'` 
and then `kill <PID>`





