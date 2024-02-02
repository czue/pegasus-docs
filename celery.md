Celery
======

[Celery](https://docs.celeryq.dev/) is a distributed task queue used to run background tasks.

It is optional, but is required for the "background task" example to work.

**If you're using [Docker in development](/docker/) then Celery should automatically be configured and running.
The following instructions are for running Celery outside of Docker, or in production.**

## Quick Start

The easiest way to get going is to [download and install Redis](https://redis.io/download) 
(if you don't already have it) and then run:

```python
celery -A {{ project_name }} worker -l info
```

### Running Celery on Windows

Celery 4.x [no longer officially supports Windows](https://docs.celeryq.dev/en/4.0/whatsnew-4.0.html#removed-features). 
To use Celery on Windows for development or test purposes, change the concurrency pool implementation to ``gevent`` instead.

``` console
pip install gevent
celery -A {{ project_name }} worker -l info -P gevent
```

## Setup and Configuration

The above setup uses [Redis](https://redis.io/) as a message broker and result backend.
If you want to use a different message broker, for example [RabbitMQ](https://www.rabbitmq.com/),
you will need to modify the `CELERY_BROKER_URL` and `CELERY_RESULT_BACKEND` values in `settings.py`.

More details can be found in the [Celery documentation](https://docs.celeryq.dev/en/stable/getting-started/backends-and-brokers/index.html).

## Monitoring with Flower

[Flower](https://flower.readthedocs.io/en/latest/) is an open-source web application for monitoring and managing Celery clusters.
It provides real-time information about the status of Celery workers and tasks.

If you'd like to use Flower in development, add the following to the `services` section of your `docker-compose.yml`:

```yaml
  flower:
    image: mher/flower
    environment:
      - CELERY_BROKER_URL=redis://redis:6379
    command: celery flower
    ports:
      - 5555:5555
    depends_on:
      - redis
```

In production you will likely want to run Flower behind a private VPN, or [set up authentication](https://flower.readthedocs.io/en/latest/auth.html)
on your Flower instance, and use a [reverse proxy](https://flower.readthedocs.io/en/latest/reverse-proxy.html) to expose it.
