Celery
======

[Celery](https://docs.celeryq.dev/) is a distributed task queue used to run background tasks.

It is required by several Pegasus features, including:

1. The "background task" example.
2. Per-unit subscriptions (celery runs the background task to sync unit amounts with Stripe).
3. AI Chat (it is used in all builds to set chat names, and, if async is not enabled, for the chats themselves).

If you aren't using any of the above features, you can disable celery by unchecking the "use celery" 
option---added in version 2025.1---in your project settings.
**If you *are* using any of the above features, this option will not do anything.**

## Quick Start

**If you're using [Docker in development](/docker/) then Celery should automatically be configured and running.
The instructions in this section are for running Celery outside of Docker.**

The easiest way to get going in development is to [download and install Redis](https://redis.io/download) 
(if you don't already have it) and then run:

*With uv:*

```python
uv run celery -A {{ project_name }} worker -l info --pool=solo
```

*With standard Python:*

```python
celery -A {{ project_name }} worker -l info --pool=solo
```

Note that the 'solo' pool is recommended for development but not for production. When running in production,
you should use a more robust pool implementation such as `prefork` (for CPU bound tasks) or `gevent` (for I/O bound
tasks).

### Running Celery with Gevent

The `gevent` pool is useful when running tasks that are I/O bound which tends to be 90% of tasks. The same
configuration can also be used to run Celery on Windows (if the `solo` pool is not suitable) since 
Celery 4.x [no longer officially supports Windows](https://docs.celeryq.dev/en/4.0/whatsnew-4.0.html#removed-features).

To use the `gevent` pool, change the concurrency pool implementation to ``gevent`` instead.

``` console
pip install gevent
celery -A {{ project_name }} worker -l info -P gevent
```

For more information see the [Celery documentation](https://docs.celeryq.dev/en/stable/userguide/concurrency/gevent.html).

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

In production, you will likely want to run Flower behind a private VPN, or [set up authentication](https://flower.readthedocs.io/en/latest/auth.html)
on your Flower instance, and use a [reverse proxy](https://flower.readthedocs.io/en/latest/reverse-proxy.html) to expose it.
