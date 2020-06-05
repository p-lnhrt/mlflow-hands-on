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
docker exec -it mlflow_mysql_db mysql -u mlflowusr -p
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

## Using a SQL database as MLflow backend store
As backend store for the metadata, metrics and tags, user can choose between a file store and a DB-backed store. A file
store simply consist in storing data in a hierarchy of files on a filesystem like the local filesystem (and others ?). By default,
tracking data is stored on the local file system in the *./mlruns* directory. DB-backed storage consists in storing the 
metadata in a SQL database. 

MLflow uses [SQLAlchemy](https://www.sqlalchemy.org/), an object-relational mapper (ORM) for Python, as SQL toolkit. MLflow 
therefore support the SQL databases supported by SQLAlchemy which include: MySQL, SQLite, PostgreSQL, MSSQL (Microsoft SQL 
Server). Each of these SQL database types is called a dialect. To interact with a given SQL dialect, SQLAlchemy also needs
a Python client library (a Python package) called a driver. For each dialect, the user can choose among a variety of supported 
drivers. For example in the case of MySQL, supported drivers include: `mysql-python`, `mysqlclient`, `pymysql` and `mysql-connection`.
The user should keep in mind the following points:
* As the user can choose among a variety of dialects and drivers, the installation of the chosen Python driver is left to 
the user: **Python SQL drivers are not part of the installed MLflow dependencies**.
* Some drivers may introduce further dependencies (for example `mysqlclient` requires a few MySQL binaries) or introduce
specific contraints (for example `pymysql` does not provide an encoder for the `numpy.float64` type forcing the user to cast
affected objects into a supported type). In the case of the MySQL dialect, SQLAlchemy recommends using either `mysqlclient` 
or `pymysql`. We chose to use `pymysql`.

Notice: An ORM allows to programmatically work with all the databases it supports. In the case of Python, it means that 
the programmer can interact with a database using Python object. If working with a particular SQL database, the ORM 
is in charge of translating the operations performed on Python object in the appropriate SQL dialect. Such tools allow not
to couple our code to a particular database type or SQL flavor.