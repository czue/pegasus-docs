# Using Docker in Development

**Docker support is a new feature in Pegasus as of October 2020.
Please report any issues and watch this page for updates.**

Pegasus optionally includes support for [Docker](https://www.docker.com/) during development.
The Docker development setup can also be used as a foundation for deploying to containerized platforms.
See [our deployment page](/deployment/) for more details.

## Prerequisites

You need to install both [Docker](https://www.docker.com/get-started) and 
[Docker Compose](https://docs.docker.com/compose/install/) prior to setting up your environment.

## Getting Started

First build your Pegasus project with Docker enabled and using Postgres as a database
following the [getting started guide](/getting-started/).

### Enter the project directory

```
cd {{ project_name }}
```

### Run the initialization script

```python
make init
```

This will spin up a database, web worker, celery worker, and Redis broker and create and
run your database migrations.

Note: users of older versions of Windows may [need to install "make" separately to use it](https://stackoverflow.com/questions/32127524/how-to-install-and-use-make-in-windows).
Alternatively, you can just inspect the `Makefile` in the repository and run the commands manually
(e.g. `docker-compose up -d`).

### Load server

Visit [http://localhost:8000/](http://localhost:8000/) in a browser and you
should be up and running!

## Using the Makefile

Pegasus ships with a self-documenting `Makefile` that will run common commands for you,
including starting your containers, performing database operations, and building your front end.

You can run `make` to see list helper functions, and you can view the source
of the `Makefile` file in case you need to add to it or run any once-off commands.

For example, you can run management commands in containers using the same method 
used in the `Makefile`. E.g.

```
docker-compose exec web python manage.py createsuperuser
```

## Architecture and how it works

### Containers 

The Docker configuration is primarily in `docker-compose.yml`.

There are four containers that start: a Postgres database, a Redis instance (for caching and use as a Celery broker),
a web container running your Django process, and a Celery container for background jobs.

### Settings

The docker environment sets environment variables using the included `.env.dev` file.
This file also tells your Docker environment to use `settings_docker.py` (which extends `settings.py`)
as the `DJANGO_SETTINGS_MODULE`.

The `.env.dev` file is automatically ignored by git, so you can put any additional secrets there.
It generally should not be checked into source control.

### Python environments

The Python environment is run in the containers, which means you do not need to have your
own local environment if you are always using Docker for development.
Python requirements are automatically installed when the container builds.

However, keep in mind that if you go this route, you will need to run all commmands inside the containers
as per the instructions below. 

## Running once-off management commands

Running commands on the server can be done using `docker-compose`, by following
the pattern used in the `Makefile`.

For example, to bootstrap Stripe subscriptions, run:

```
docker-compose exec web python manage.py bootstrap_subscriptions
```

Or to promote a user to superuser, run: 
```
docker-compose exec web python manage.py promote_user_to_superuser me@example.com
```

Commonly used commands can be added to the `Makefile` for convenience.

## Updating Python packages

If you add or modify anything in your `requirements.txt` files, you will have to rebuild
your containers. This can be done by running:

```
docker-compose build
``` 

After than you can run

```
make start
``` 

to run your project with the latest dependencies.

## Other Resources

- [Dockerizing Django with Postgres, Gunicorn, and Nginx](https://testdriven.io/blog/dockerizing-django-with-postgres-gunicorn-and-nginx/)
  provides an overview of the setup, and has additional information about using Docker in production
- [Environment variables in Compose](https://docs.docker.com/compose/environment-variables/) is a good resource
  on the different ways to work with environment variables in Docker
