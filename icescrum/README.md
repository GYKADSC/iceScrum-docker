# iceScrum official Docker image

This is the official iceScrum Docker image, with support for recent Docker versions.

iceScrum is an open-minded and expert agile project management tool based on the Scrum methodology: https://www.icescrum.com/features/.

Tags:
- iceScrum R6#14.8: `latest`, `R6#14.8`.

## Environment variables

__Important notice:__ iceScrum must know beforehand the unique external URL that will be used to open it in a browser. By default, this URL is `http://localhost:8080/icescrum`.

If you don't want or cannot expose iceScrum on this URL, our image supports environment variables to define a custom URL.

Pass one or more environment variables to the iceScrum container by adding to the `docker run` command the `-e VARIABLE=value` argument.

### `ICESCRUM_HTTPS`
If set, the protocol will be `https` instead of `http` in the URL. Be careful: this is all that this variables does, it does not configure the SSL connection at all.

### `ICESCRUM_HOST`
__Required if you use docker-machine, e.g. to use Docker on OS X or Windows__, in such case set the IP of your Docker host, provided by `docker-machine ip yourmachine`.

### `ICESCRUM_PORT`
The iceScrum Docker image will always have iceScrum running on its internal port `8080`, but nothing prevents you from defining a different external port (e.g. by exposing a different port in `docker run` via the `-p` argument).

### `ICESCRUM_CONTEXT`
It's the name that comes after "/" in the URL. You can either define another one or provide `/` to have an empty context.

## Basic usage - Default embedded HSQLDB database

Be careful, the HSQLDB default embedded DBMS __is not reliable for production use__, so we recommend that you rather use an external DBMS such as MySQL.

Start iceScrum with HSQLDB on Linux:
```
docker run --name icescrum -v /mycomputer/icescrum/home:/root -p 8080:8080 icescrum/icescrum
```

Start iceScrum with HSQLDB on OS X / Windows / docker-machine:
```
docker run --name icescrum -e ICESCRUM_HOST=ipOfYourDockerHost -v /mycomputer/icescrum/home:/root -p 8080:8080 icescrum/icescrum
```

The iceScrum data (config.groovy, logs...) is persisted on your computer into `/mycomputer/icescrum/home` (replace by an absolute or relative path from your computer) and the HSQLDB files are stored under its `hsqldb` directory.

## Use with MySQL or PostgreSQL

This integration makes use of the Docker "networks"" feature, we chose to not use the "link"" feature that seems to be deprecated. Starting both containers on the same network simply allows iceScrum to access your MySQL or PostgreSQL container by its name thanks to an automatic `/etc/hosts` entry.

### 1. Create the network

```
docker network create --driver bridge is_network
```

### 2. Start the DB container (pick one!)

* __MySQL__

At first startup you will need to provide a password for the MySQL `root` user.

The iceScrum MySQL image is just a standard MySQL image that creates an database named `icescrum` with the `utf8_general_ci` collation at the first startup.

```
docker run --name mysql -v /mycomputer/icescrum/mysql:/var/lib/mysql --net=is_network -e MYSQL_ROOT_PASSWORD=myPass -d icescrum/mysql
```

MySQL data is persisted on your computer into `/mycomputer/icescrum/mysql` (replace by an absolute or relative path from your computer).

* __PostgreSQL__

At first startup you will need to provide a password for the `postgre` user.

The iceScrum PostgreSQL image is just a standard PostgreSQL image that creates an `icescrum` database with the `en_US.UTF-8` encoding at the first startup.

Start PostgreSQL on Linux, its data is persisted on your computer into `/mycomputer/icescrum/postgres` (replace by an absolute or relative path from your computer):
```
docker run --name postgres -v /mycomputer/icescrum/postgres:/var/lib/postgresql/data --net=is_network -e POSTGRES_PASSWORD=myPass -d icescrum/postgres
```

On OS X / Windows / docker-machine __mounting a volume from your OS will not work__, see https://github.com/docker-library/postgres/issues/28, so you will need to keep the PostgreSQL data inside the container. Use the command:

```
docker run --name postgres --net=is_network -e POSTGRES_PASSWORD=myPass -d icescrum/postgres
```

### 3. Start the iceScrum container

Start iceScrum with MySQL / PostgreSQL on Linux:

```
docker run --name icescrum -v /mycomputer/icescrum/home:/root --net=is_network -p 8080:8080 icescrum/icescrum
```

Start iceScrum with MySQL / PostgreSQL on OS X / Windows / docker-machine:

```
docker run --name icescrum -e ICESCRUM_HOST=ipOfYourDockerHost -v /mycomputer/icescrum/home:/root --net=is_network -p 8080:8080 icescrum/icescrum
```

Don't start it in background so you will be able to check that everything goes well in the logs.

iceScrum data (config.groovy, logs...) is persisted on your computer into `/mycomputer/icescrum/home` (replace by an absolute or relative path from your computer).

## Startup

The very first line of the iceScrum container output displays the external URL of iceScrum (according to the provided environment variables).

Wait until your see "Server startup in XXXX ms" then iceScrum should be available at the provided URL.

## Setup wizard

If it's the first time you use iceScrum, you will have to configure iceScrum through a user friendly wizard. Here is the documentation: https://www.icescrum.com/documentation/install-guide/#settings

Some settings are pre-filled for you (mainly the location where iceScrum stores file, so they can be mapped to your computer).

* __HSQLDB__

If you want to keep the HSQLDB database then the database configuration is prefilled and you can just click next.

* __MySQL__

If you use the MySQL container, choose the MySQL database in the settings and configure it:
- _URL_: replace "localhost" by the name of the MySQL container (in our example: `mysql`)
- _Username_: `root`
- _Password_: the one defined when starting the MySQL container (in our example: `myPass`)

When clicking on next, a database connection is tried and if you get no error then it is successful.
You will have to restart the container at the end of the startup so iceScrum can start on your custom DB.

* __PostgreSQL__

If you use the PostgreSQL container, choose the PostgreSQL database in the settings and configure it:
- _URL_: replace "localhost" by the name of the PostgreSQL container (in our example: `postgres`)
- _Username_: `postgres`
- _Password_: the one defined when starting the PostgreSQL container (in our example: `myPass`)

When clicking on next, a database connection is tried and if you get no error then it is successful.
You will have to restart the container at the end of the startup so iceScrum can start on your custom DB.

## Switch database

To migrate from one database to another:

1. Export the projects you want to keep from the running iceScrum application (project > export).

2. Stop the iceScrum container.

3. Change the DB configuration manually in the `config.groovy` file stored on your computer in the directory you defined, see https://www.icescrum.com/documentation/config-groovy-file/#database. If you come from HSQLDB, you can alternatively delete the `hsqldb` directory so the setup wizard will show up again on next startup.

4. Start the iceScrum container and import your projects (project > import).

## Docker Compose

Here is an example docker-compose.yml file that starts iceScrum and MySQL
```yml
version: '2'
services:
  mysql:
    image: icescrum/mysql
    volumes:
      - /mycomputer/icescrum/mysql:/var/lib/mysql
    environment:
      - MYSQL_ROOT_PASSWORD=myPass

  icescrum:
    image: icescrum/icescrum
    ports:
      - "8080:8080"
    volumes:
      - /mycomputer/icescrum/home:/root
    links:
      - mysql
```

## Information

The iceScrum Docker image is maintained by the company who develops iceScrum: __Kagilum__. More information on our website: https://www.icescrum.com/.

We would like to thank Caner Candan who was the first to develop and maintain an iceScrum docker image!