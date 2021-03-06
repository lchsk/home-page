---
title: How to connect to a host Postgres database from a Docker container?
created: 2019-10-05T00:00:00Z
description: Assume we have a web service running inside a Docker container. We also have a Postgres database running on a host system. This means we need to allow a Docker container to connect to a local Postgres database. the database and the container need to be configured in order to communicate. Here's how to do it.
keywords: docker, postgres, postgresql, containers, postgres configuration, add-host docker, postgresql.conf, docker networking
slug: how-to-connect-to-a-host-postgres-database-from-a-docker-container
---


Let's assume we have a web service running inside a Docker container. We also have a Postgres database running on a host system (i.e., outside of Docker as you probably don't want to run a database in Docker in production). This means we need to allow a Docker container to connect to a local Postgres database.
Assuming you don't want the container to share the network with the host (using `--network host` parameter when running it), the database and the container need to be configured in order to communicate. Here's how to do it.

## Postgres configuration changes

The database needs configuration changes to allow connections from Docker containers. 
First, find Postgres configuration files. They're usually available in `/etc/postgresql/<postgres version>/main`.

#### File `postgresql.conf`

Change the following line:

```
listen_addresses = 'localhost'
```

to

```
listen_addresses = '*'
```

This will allow Postgres to listen to all addresses, not just to localhost.

#### File `pg_hba.conf`

In this file, add the following line

```
host	all	all	172.17.0.1/16	md5
```

This line will allow connections to Postgres from `172.17.0.1/16` range of addresses. They belong to Docker and when running a container Docker will assign an IP to a container from this range.

## Docker configuration

When running your Docker container you need to configure it use the host system.

#### Tell the container about the host's IP address

Pass the host's IP using `--add-host` option, e.g.:

```
--add-host=host:<ip>
```

In this case `host` is a name of the host machine, and it will be added to container's `/etc/hosts` file. 
Inside the Docker container, you will be able to use the name `host` to connect to the host system.

#### Remember to publish container's ports

For instance, if your dockerized app is running a server on port 8000,
you can pass:

```
-p 8000:8000
```

to the `docker run` command to make your app available on the host on port `8000`. 
The full `docker run` command can look similar to this:

```
docker run -dt --rm --add-host=host:<ip> --name container_name -p 8000:8000 image_name
```

Those changes should be enough to make a Postgres database running on a host system work with a containerized app.

### Related posts

- {{@database-transactions-concurrency-isolation-levels-and-postgresql}}

- {{@benchmarking-concurrent-operations-in-postgresql}}

- {{@building-a-postgres-gui-client-with-c-and-gtk-sanchosql}}

- {{@stay-paranoid-and-trust-no-one-overview-of-common-security-vulnerabilities-in-web-applications}}
