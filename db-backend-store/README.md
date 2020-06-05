## Dockerfile
Our `mlflow-mysql` Dockerfile is based on the "official" [`mysql`](https://hub.docker.com/_/mysql/) image. 
Note that there exist a very similar image maintained by the MySQL Oracle team named [`mysql-server`](https://hub.docker.com/r/mysql/mysql-server/).

At container startup, we minimally initialize the database following the `mySQL` image [documentation](https://hub.docker.com/_/mysql/) and
using dedicated environment variables: 
* We set the password for the superuser account `root` using the `MYSQL_ROOT_PASSWORD` environment variable.
* We create a database named `mlflow` using the `MYSQL_DATABASE` environment variable.
* We create a user named `mlflowusr` and set its password to `mlflowusrpwd` using the `MYSQL_USER` and `MYSQL_PASSWORD` 
environment variables. This user is automatically granted superuser access (corresponding to the `GRANT ALL` SQL statement)
to the above-created database. 

For more elaborate setup strategies, especially using configuration files, secret files (so that passwords do not appear in clear), 
etc.: refer to the Docker image's documentation and MySQL documentation.

## Docker image
Change your current working directory to the `db-backend-store` directory:

```bash
cd db-backend-store
```

Build the MySQL Docker image (here with the name `mlflow-mysql`):

```bash
docker build -t mlflow-mysql:latest .
```

## Running the container 
Launch the container using the Docker image you just built with the following command:

```bash
docker run --name mlflow_mysql_db --init -d -p 3306:3306 -v $(pwd)/data:/var/lib/mysql mlflow-mysql:latest
```

About the command above:
* `--name mlflow_mysql_db`: We assign the `mlflow_mysql_db` friendly name to the started container.
* `--init`: Runs an init as PID 1 instead of having the container's entrypoint as PID1 which comes with several 
disadvantages. See this [Tini discussion](https://github.com/krallin/tini/issues/8), Tini being the init that has been 
integrated to Docker since Docker 1.13. 
* `-d`: The container is launched as background process (deamon).
* `-p 3306:3306`: The container's 3306 port is published to the local host's port 3306.
* `-v`: For data persistence, the *db-backend-store/data* directory from the local host system is mounted as */var/lib/mysql* inside 
the container, where MySQL by default will write its data files.

## Using the database's MySQL prompt
To use the MySQL prompt and execute SQL queries directly on the MySQL database running within the container,
first run the following command:

```bash
docker exec -it mlflow_mysql_db mysql -u mlflowuser -p
```

Give the password for the `mlflowuser` user and start submitting your SQL queries.

## Using a Python client to connect to the database
We provide here an example scripts that uses the official MySQL Python driver `mysql.connector`:

```python
import mysql.connector

cnx = mysql.connector.connect(user='mlflowuser', 
                              password='mlflowusrpwd',
                              host='localhost',
                              port=3306)

cursor = cnx.cursor(buffered=True)
cursor.execute('SHOW DATABASES')

for database, in cursor:
    print(database)

cursor.close()
cnx.close()
```
