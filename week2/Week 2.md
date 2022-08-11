Week 2:

configuring postgres by setting the configuration. The configuration is passed from the docker command 

- user, password and name of the db would be
- volumes: postgres is a db need to keep files in a file system
                  docker doesn't keep the state/data
                  mapping of data from the docker image to a host system folder-mounting 
                  the db on the container
- mapping of port from host machine to docker image

```
docker run -it -e POSTGRES_USER="root" \
			  -e POSTGRES_PASSWORD="root" \
			  -e POSTGRES_DB="ny_taxi" \
			  -v ny_taxi_postgres_data:/full_path/to/db_folder 
			  -p 5432:5432
			  postgres:13
```

: **Volume mapping/Persisting data on to the host** : changes on files are lost when container is closed/removed,

- volumes can be used to persist data on the host
- ability to connect specific filesystem paths back to the host system (folder)
- if the volume is mounted when the container is restarted the files are accessible from within the container from the host.

once the above command is run docker image with postgres gets started.

**Connect to DB**

To access the database open another terminal open a connection to the db using pgcli

```
pgcli -h localhost -p 5432 -u root -d ny_taxi
```

the above command is used to connect to the db '*ny_taxi*' that is running at *localhost* and *port 5432* with a user name *root* 