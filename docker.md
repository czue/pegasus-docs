# Using Docker in Development

**Docker support is a new feature in Pegasus under active development (as of October 2020).
Watch this page for more updates soon.**

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

### Create your environment file

Create a local `.env.dev` file for your settings and secrets, by copying the `.env.dev.example` file:

```
cp .env.dev.example .env.dev
```

This file also tells your Docker environment to use your `settings_docker.py` as the `DJANGO_SETTINGS_MODULE`.

### Create and start your Docker containers

```python
docker-compose up
```

Or to run in the background:
 
```python
docker-compose up -d
```

### Create and run database migrations

```python
docker-compose exec web python manage.py makemigrations
docker-compose exec web python manage.py migrate --noinput
```

### Load server

Visit [http://localhost:8000/](http://localhost:8000/) in a browser and you
should be up and running!


## Architecture and how it works

### Containers 

The Docker configuration is primarily in `docker-compose.yml`.

There are four containers that start: a Postgres database, a Redis instance (for caching and use as a Celery broker),
a web container running your Django process, and a Celery container for background jobs.

### Settings

The Django and Celery settings use `settings_docker.py` which extends `settings.py`.
Additional environment variables are passed via `.env.dev`.

### Python environments

The Python environment is run in the containers, which means you do not need to have your
own local environment if you are always using Docker for development.
Python requirements are automatically installed when the container builds.

However, keep in mind that if you go this route, you will need to run all commmands inside the containers
as per the instructions below. 

## Running management commands

Running commands on the server can be done using `docker-compose`, as per the `migrate` command above.

For example, to bootstrap Stripe subscriptions, run:

```
docker-compose exec web python manage.py bootstrap_subscriptions
```

Or to promote a user to superuser, run: 
```
docker-compose exec web python manage.py promote_user_to_superuser me@example.com
```

## Other Resources

- [Dockerizing Django with Postgres, Gunicorn, and Nginx](https://testdriven.io/blog/dockerizing-django-with-postgres-gunicorn-and-nginx/)
  provides an overview of the setup, and has additional information about using Docker in production
- [Environment variables in Compose](https://docs.docker.com/compose/environment-variables/) is a good resource
  on the different ways to work with environment variables in Docker
