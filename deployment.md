# Deployment

**Deployment support is a new feature in Pegasus under active development (as of November 2020)!
Watch this page for more updates and platforms, coming soon.**

Pegasus can be deployed on any standard cloud infrastructure.
The most common ways of deploying Pegasus are:

1. On a raw VPS / Virtual Machine, such Digital Ocean, Linode, or Amazon Lightsail
2. On a platform-as-a-service (PaaS) platform, such as Heroku, or PythonAnywhere
3. In a containerized way, using Docker, Kubernetes, or Google Cloud Platform

Pegasus natively supports Heroku, Digital Ocean App Platform, and Google Cloud Run---though
can be deployed on any infrastructure that supports Django apps.
The following instructions are for specific supported platforms.

If you would like to deploy to a platform that's not listed here, please get in touch by emailing 
cory@saaspegasus.com and I'm happy to help!

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

Pegasus supports deploying to [Google Cloud Run](https://cloud.google.com/run).
See the [Google Cloud documentation](#google-cloud) for details.
Note that Celery is not yet supported on Google Cloud Run.

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


### Configure Django Settings

The Heroku deployment uses its own settings module (which extends the normal `settings.py`).
To tell Heroku to use it, set the `DJANGO_SETTINGS_MODULE` config var to `{ project_slug }.settings_heroku`.
This can be done in the "settings" tab of your Heroku application (you may need to click to reveal the Config vars)
or in the CLI using the following command (replacing the `project_slug` with your app name):

```
heroku config:set DJANGO_SETTINGS_MODULE={ project_slug }.settings_heroku
```

### Set up Database

To set up your database, first enable the addon in the UI or by running:

```
heroku addons:create heroku-postgresql
```

Then run your initial migrations using:

```
heroku run python manage.py migrate
```

Note: if you're using Heroku's Python module, migrations will run automatically.

### Deploying

Both builds can be deployed using Heroku's standard git integration.
After you've connected your project's git repository to Heroku, just run:

```
git push heroku master
```

### Additional settings configuration

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

However, you will need to install [the Heroku Redis addon](https://elements.heroku.com/addons/heroku-redis)
from the UI or by running:

```
heroku addons:create heroku-redis
```

Additionally, you may need to run the following command to initialize a Celery worker:

```
heroku ps:scale worker=1
```

This process should be the same for Python and containerized builds.

## Digital Ocean

Pegasus provides native support for Digital Ocean App Platform.
To build for Digital Ocean, choose the "digital_ocean_app_platform" option when installing Pegasus.
Then follow the steps below to deploy your app.

### Prerequisites

If you haven't already, create your Digital Ocean account.
**You can sign up with [this link](https://m.do.co/c/432e3abb37f3) to get $100 credit
and help support Pegasus.**

Next, install and configure the `doctl` command line tool by following [these instructions](https://www.digitalocean.com/docs/apis-clis/doctl/how-to/install/).

Additionally, you must connect Digital Ocean to your project's Github repository.
This can be done from inside App Platform, or by following [this link](https://cloud.digitalocean.com/apps/github/install).

### Deploying

Once you've configured the prerequisites, deploying is just a few steps.

1. Edit the `/deploy/app-spec.yaml` file. In particular, make sure to set your Github repository and branch.
2. Run `doctl apps create --spec deploy/app-spec.yaml`

That's it!
In a few minutes your app should be online.
You can [find and view it here](https://cloud.digitalocean.com/apps).

### Running Database Migrations

You'll need to run database migrations to get your app working properly.
The easiest way to do this is to click the "console" tab in app platform and just type in the command:

```
./manage.py migrate
```

See the screenshot below for what it looks like:

![Console Migrations](/images/deployment/running-migrations-do.png)

### Settings and Secrets

App platform builds use the `settings_do.py` file.
You can add settings here, and use environment variables to manage any secrets,
following the `SECRET_KEY` example.

## Google Cloud

Pegasus can be deployed to Google Cloud Run using containers.
*This feature is in beta and Celery is not yet supported.*

To build for Google Cloud, choose the "google_cloud" option when installing Pegasus.
Then follow the steps below to deploy your app.

### Prerequisites

It is recommended to complete steps 1, 2, 3, and 5 of the 
[Django on Cloud Run](https://codelabs.developers.google.com/codelabs/cloud-run-django) guide
(Pegasus handles step 4 for you).
Pegasus generates a script that you can use as a starting point at
`./scripts/google/cloud_run.sh`, however *it is recommended you follow the steps in order*
to troubleshoot.

In particular you *must*:

- Set up your `gcloud` environment, services, and billing
- Set the the `PROJECT_ID` and `REGION` environment variables

Additionally, when completing [step 5](https://codelabs.developers.google.com/codelabs/cloud-run-django#4)
be sure to save your SQL user credentials in  a file named `.env.production` (Pegasus will provide 
a template for you that you can edit).

### Building images for Google Cloud

After settings up Pegasus and configuring your Google cloud environment, database, and secrets,
you can build your container for Google Cloud by running the following command:

```
gcloud builds submit --config cloudmigrate.yaml  --substitutions _REGION=$REGION
```

This will create your image, upload it to Google's container registry, run your database migrations,
and collect your static files.

You can then deploy it to Google Cloud Run by running the following command,
replacing the variables as needed:

```
gcloud run deploy my-project --platform managed --region $REGION --image gcr.io/$PROJECT_ID/{project_slug}-cloudrun --add-cloudsql-instances ${PROJECT_ID}:${REGION}:your_instance  --allow-unauthenticated --set-env-vars=DJANGO_SETTINGS_MODULE={project_slug}.settings_google
```

Pegasus will also generate this command for you as the last line of `./scripts/google/cloud_run.sh`.

### Settings and Secrets

To tell Google Cloud to use your production settings you will need to set the
`DJANGO_SETTINGS_MODULE=[project_slug].settings_google`.
If you use Pegasus's `cloudmigrate.yaml` file this will be handled automatically for you

You can use Google Secret Manager to add additional settings and secrets by adding them
to `.env.production` and uploading it to Secret Manager using:

```
gcloud secrets versions add {project_slug}_settings --data-file .env.production
``` 

See `settings_google.py` for examples of using these secrets in your settings file.
