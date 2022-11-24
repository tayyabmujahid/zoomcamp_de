# Week 1

[Main Notes](https://docs.google.com/document/d/e/2PACX-1vRJUuGfzgIdbkalPgg2nQ884CnZkCg314T_OBq-_hfcowPxNIA0-z5OtMTDzuzute9VBHMjNYZFTCc1/pub), [other notes](https://github.com/ABZ-Aaron/DataEngineerZoomCamp/tree/master/week_1_basics_n_setup)

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

This step creates an image of Postgres db and uses `pgcli` to access the database

- *for running a Postgres container the following command is used*

``` bash 
docker run -it \
 -e POSTGRES_USER="root" \ # The name of our PostgreSQL user
 -e POSTGRES_PASSWORD="root" \ # The password for our user
 -e POSTGRES_DB="ny_taxi" \ # The name we want for our database
 -v  /home/mujahid/PycharmProjects/zoomcamp_de/week1/ny_taxi_postgres_data \
 :/var/lib/postgresql/data \
 -p 5432:5432 \ # This maps a postgres port on our host machine to one in our containe
 postgres:13
```

-v flag - volumes is a way to map the folder on the container to the host machine file system to physically store the records. Docker doesn't keep the state next time when the container is started data must be available. We need to mount the folder on the host machine to the docker container. 

Port on the host machine must also be mapped to the port on the docker container to access the Postgres database

the `docker run` command looks for an image of `postgres:13`  if it does not find one then downloads it and creates an image and then runs a container

Side note: if the port is busy  us this to find the process at running at the port[[url](https://stackoverflow.com/questions/38249434/docker-postgres-failed-to-bind-tcp-0-0-0-05432-address-already-in-use)]
`sudo ss -lptn 'sport = :5432'` 
and then `kill <PID>`

With the container running now we can access it 

- *Accessing the Postgres db with pgcli*

install pgcli - its a client for Postgres

``` bash
pip install pgcli
```

then connect to the pgcli client to the database

```bash
pgcli -h localhost -p 5432 -u root -d ny_taxi 
```

and then check list of tables using 

```bash
root@localhost:nytaxi> \dt
```

#### 3. Populating  Postgres db with Taxi rides dataset 

The goal here is to 

1. Create a table (by generating schema) in the database to match the dataframe datatype.

2. load the data into the database chunk by chunk using an iterator as the taxi dataset has more than a million rows.

   **Download data and convert to timestamp datatype for columns with timestamp related information**

- Download the data from [here](https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page)

  the data here is `.paraquet` format convert that to `.csv` using `df.to_csv` so that data can be handled easily and loaded in chunks.

- convert time related to colums to timestamp datatype using `pd.to_datetime`

  **Create schema from the dataframe of the dataset for creating table in the database**

- install sqlalchemy

  ```bash
  pip install sqlalchemy
  ```

- to load this taxi data into the database first a table needs to be created. To generate a schema for the table to generate a DDL(data description language) statement using the following statement

  ```python
  pd.is.sql.get_schema(df,name="yello_taxi_data")
  ```

- using sqlalchemy create_engine generate correct Postgres compatible format of DDL and pass this connection  engine to .get_schema statement to generate the schema

- Use this DDL statement to create table in the postgres database

  ```python
  # This creates a connection to the postgres db that is active at port 5432
  engine = create_engine('postgresql://root:root@localhost:5432/ny_taxi')
  engine.connect()
  
  # This creates a schema for postgres db with a connection at `con`
  print(pd.io.sql.get_schema(df,name='yellow_taxi_data',con=engine))
  ```

  

  **Load Data in chunks**

- Load the df in chunks using and iterator so that we dont enter

  - use the `iterator=True` for the `read_csv` statement; this converts the `df` to a standard python iterator.
  - `next(df)` can be used to access the chunks

  **Write data into the table**

  

  ```python
  #this creates only the table with the schema of df but does not add any data as df.head(0)
  df.head(0).to_sql(name='yellow_taxi_data',con=engine,if_exists='replace')
  # this appends data(but chunk wise) to the table
  df.to_sql(name='yellow_taxi_data',con=engine,if_exists='append')
  ```

  To enter all the chunks the following code can be used

  ```python
  while True:
      t_start = time()
      df = next(df_iter)
      df.tpep_pickup_datetime= pd.to_datetime(df.tpep_pickup_datetime)
      df.tpep_dropoff_datetime= pd.to_datetime(df.tpep_dropoff_datetime)
      df.to_sql(name='yellow_taxi_data',con=engine,if_exists='append')
      t_end = time()
      print('inserted another chunk, took %.3f second '%(t_end-t_start))
  ```

  

[video](https://www.youtube.com/watch?v=2JM-ziJt0WI&list=PL3MmuxUbc_hJed7dXYoJw8DoCuVHhGEQb)

**References**

https://github.com/ABZ-Aaron/DataEngineerZoomCamp/tree/master/week_1_basics_n_setup





