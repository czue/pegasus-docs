# Deployment

**Deployment support is a new feature in Pegasus under active development (as of November 2020)!
Watch this page for more updates and platforms, coming soon.**

Pegasus can be deployed on any standard cloud infrastructure.
The most common ways of deploying Pegasus are:

1. On a raw VPS / Virtual Machine, such Digital Ocean, Linode, or Amazon Lightsail
2. On a platform-as-a-service (PaaS) platform, such as Heroku, or PythonAnywhere
3. In a containerized way, using Docker, Kubernetes, or Google Cloud Platform

The following instructions are for specific platforms.

If you would like to deploy to a platform that's not listed here, please get in touch by emailing 
cory@saaspegasus.com!

## Deploying to containers with Docker

As of version 0.10.2, Pegasus ships with [Docker support for development](/docker/).
The Docker containers used for development can be modified to deploy to production environments
that support containers - for example, Google Cloud Run/Kubernetes Engine, Amazon ECS,
Heroku Container Registry, and Digital Ocean App Platform.

Each environment has its own quirks in getting set up with containers - some of which are documented below.

### Heroku Containers

Pegasus supports deploying to [Heroku with Docker](https://devcenter.heroku.com/categories/deploying-with-docker)
out of the box. For more details see the [Heroku documentation](#heroku).

### Google Cloud Run

The [Django on Cloud Run](https://codelabs.developers.google.com/codelabs/cloud-run-django) guide is a useful
resource for deploying Pegasus apps on [Google Cloud Run](https://cloud.google.com/run).
This [other guide](https://medium.com/@lhennessy/running-django-on-google-cloud-run-with-cloudsql-ac8141095b77) is
also helpful to better understand the steps involved, though does not cover
setting up a SQL database.

## Heroku

Pegasus supports deploying to Heroku as a standard Python application or using containers.

### Building using Heroku's Python support

To deploy with Heroku's Python module, first set up Pegasus using the "heroku" deploy platform option.
This will create your Heroku `Procfile`, `runtime.txt`, and additional requirements/settings files
needed for the Heroku platform.


### Building using Heroku's Docker container support

To deploy to Heroku using Docker, you should build Pegasus with the "heroku docker" deployment option.
This will create your production `Dockerfile`, a `heroku.yml` file you can use [to build and deploy your 
container](https://devcenter.heroku.com/articles/build-docker-images-heroku-yml), and 
additional requirements/settings needed for the Heroku platform.

After building and setting up Heroku you will also need to configure Heroku to
deploy with containers by running:

```
heroku stack:set container
```

### Deploying

Both builds can be deployed using Heroku's standard git integration.
After you've connected your project's git repository to Heroku, just run:

```
git push heroku master
```

For things to run you will also need to configure your settings and environment per the next section.

### Settings configuration

The Heroku deployment uses its own settings module (which extends the normal `settings.py`).
To tell Heroku to use it, set the `DJANGO_SETTINGS_MODULE` config var to `{ project_slug }.settings_heroku`.

If you're using Docker, this will happen automatically, but in the Python build you can set it
in the "settings" tab of your Heroku application (you may need to click to reveal the Config vars) or
in the CLI using the following command (replacing the `project_slug` with your app name):

```
heroku config:set DJANGO_SETTINGS_MODULE={ project_slug }.settings_heroku
```

If you need additional production settings, you can put them in the `settings_heroku.py` file,
or include them as config vars like this:

```python
SECRET_KEY = os.environ.get('SECRET_KEY')
```

We recommend also setting `SECRET_KEY` in your Heroku config vars to avoid having it in version control.


### Database migrations

If you're using Heroku's Python module, database migrations should run automatically.
However, in the Docker set up you will to manually run the following command to intialize your Database:

```
heroku run python manage.py migrate
```

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

This process should be the same for Python and containerized builds.
