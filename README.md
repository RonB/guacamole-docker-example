gucamole-docker example
===

According to the *Guacamole Manual* [Chapter 3. Installing Guacamole with Docker](https://guacamole.incubator.apache.org/doc/gug/guacamole-docker.html).

As linking container is not supported since docker-compose v2.1 the guacamole services configuration is done using environment variables.

In the [docker-compose.yml](docker-compose.yml) services share the same ``.env`` file.

Create the ``.env`` file in the root of the project, don't forget changing the ``POSTGRES_PASSWORD``:

```shell
#guacd
GUACD_HOSTNAME=guacd
#GUACD_PORT=4822
#
#Database settigs
POSTGRES_USER=guacamole
POSTGRES_PASSWORD=secretdbpassword
PGDATA=/var/lib/postgresql/db
POSTGRES_DB=guacamole
POSTGRES_DATABASE=guacamole
POSTGRES_HOSTNAME=sql
#POSTGRES_PORT=5432
```

### PostgreSQL

It is PostgreSQL and the [official postgres docker image](https://hub.docker.com/_/postgres/) is used as gucacmole's authentication mechanism.

In the envirnment ``POSTGRES_DB`` is used by the **posgrges** container, ``POSTGRES_DATABASE`` is for **guacaole/guacamole** container, and must be the same.

Before the first usage the guacamole database must be initialized. The [docker-initdb.yml](docker-initdb.yml) is for initializing the posgrges db.

*You may want editing volumes in [docker-compose.yml](docker-compose.yml#L12-L18) and [docker-initdb.yml](docker-initdb.yml#L5-L15).* The **db** volume must be the same.

First, update the [sql script](initdb.d/initdb.sql):

```shell
docker run --rm guacamole/guacamole /opt/guacamole/bin/initdb.sh --postgres > initdb.d/initdb.sql
```

then initialize the database

```shell
docker-compose -f docker-initdb.yml up
```

(terminate the container when initialization completes).

### Starting Guacamole

Starting guacamole gateway is trivial:

```shell
docker-compose up -d
```

According to the [documentation](https://guacamole.incubator.apache.org/doc/gug/guacamole-docker.html#verifying-guacamole-docker)
> Once the Guacamole image is running, Guacamole should be accessible at ``http://HOSTNAME:8080/guacamole/``, where ``HOSTNAME`` is the hostname or address of the machine hosting Docker, and you should a login screen. If using MySQL or PostgreSQL, the database initialization scripts will have created a default administrative user called "guacadmin" with the password "guacadmin".

### Using HAProxy

If you wish using HAProxy as the gateway - un-comment the external **web:** network in the **guacamole** service in the [docker-compose.yml](docker-compose.yml#L44).

The ``haproxy_default`` network has to be pre-defined, for eg.:

```shell
docker network create -d bridge \
 --subnet=172.15.0.0/16 \
 --gateway=172.15.0.1 \
 haproxy_default
```

I prefer using [bringnow/docker-haproxy-letsencrypt](https://github.com/bringnow/docker-haproxy-letsencrypt), see the sample [haproxy.cfg](haproxy.cfg#L55-L59) with guacamole backend and https termination.

According to this configuration *guacamole* is accessible at ``https://example.tld/guacamole/``.
