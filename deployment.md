# Deployment

**Deployment support is a new feature in Pegasus under active development (as of October 2020)!
Watch this page for more updates and platforms, coming soon.**

Pegasus can be deployed on any standard cloud infrastructure.
The most common ways of deploying Pegasus are:

1. On a raw VPS / Virtual Machine, such Digital Ocean, Linode, or Amazon Lightsail
2. On a platform-as-a-service (PaaS) platform, such as Heroku, or PythonAnywhere
3. In a containerized way, using Docker, Kubernetes, or Google Cloud Platform

The following instructions are for specific platforms.

If you would like to deploy to a platform that's not listed here, please get in touch by emailing 
cory@saaspegasus.com!

## Deploying to containers, with Docker

As of version 0.10.2, Pegasus ships with [Docker support for development](/docker/).
The Docker containers used for development can be modified to deploy to production environments
that support containers - for example, Google Cloud Run/Kubernetes Engine, Amazon ECS,
Heroku Container Registry, and Digital Ocean App Platform.

Each environment has its own quirks in getting set up with containers - some of which are documented below.

### Google Cloud Run

The [Django on Cloud Run](https://codelabs.developers.google.com/codelabs/cloud-run-django) guide is a useful
resource for deploying Pegasus apps on [Google Cloud Run](https://cloud.google.com/run).
This [other guide](https://medium.com/@lhennessy/running-django-on-google-cloud-run-with-cloudsql-ac8141095b77) is
also helpful to better understand the steps involved, though does not cover
setting up a SQL database.


## Heroku

To deploy to Heroku, first set up Pegasus using the "heroku" deploy platform option.
This will create your Heroku `Procfile`, `runtime.txt`, and additional requirements/settings files
needed for the Heroku platform.

### Settings configuration

The Heroku deployment uses its own settings module (which extends the normal `settings.py`).
To tell Heroku to use it, set the `DJANGO_SETTINGS_MODULE` config var to `{ project_slug }.settings_heroku`
in the "settings" tab of your Heroku application (you may need to click to reveal the Config vars).

If you need additional production settings, you can put them in the `settings_heroku.py` file,
or include them as config vars like this:

```python
SECRET_KEY = os.environ.get('SECRET_KEY')
```

We recommend also setting `SECRET_KEY` in your Heroku config vars to avoid having it in version control.

### Stripe support

If you're using Stripe, you will need to set the `STRIPE_TEST_PUBLIC_KEY`, `STRIPE_TEST_SECRET_KEY`, 
`STRIPE_LIVE_PUBLIC_KEY`, and `STRIPE_LIVE_SECRET_KEY` config vars (or whatever subset you are using).

After setting up your Stripe variables, you can run:

```
heroku run python manage.py bootstrap_subscriptions
```

to initialize your subscription data.

### Celery support

The Heroku environment supports Celery out-of-the-box.

However, you will need to install [the Heroku Redis addon](https://elements.heroku.com/addons/heroku-redis).

Additionally, you may need to run the following command to initialize a Celery worker:

```
heroku ps:scale worker=1
```
