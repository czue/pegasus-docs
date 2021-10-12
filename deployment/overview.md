# Production Deployment

Pegasus---like Django---can be deployed on any standard cloud infrastructure.

The most common ways of deploying Pegasus are:

1. On a raw VPS / Virtual Machine, such Digital Ocean, Linode, or Amazon EC2 or Lightsail
2. On a platform-as-a-service (PaaS) platform, such as Heroku, or PythonAnywhere
3. In a containerized way, using Docker, and (optionally) Kubernetes

Choosing the right deployment architecture involves a complex set of trade-offs, and there's no one-size-fits-all solution.
PaaS and Docker-based solutions tend to be easier to get up and running, but can be more difficult to modify
and are often more expensive at scale.
Meanwhile, setting up a VPS can be error-prone but is a very cost-effective way to deploy small applications.

Much of the choice will also depend on the knowledge and comfort of you/your team with various tools and platforms.

## Officially supported platforms

Pegasus ships with configuration files to deploy to select platforms out-of-the-box.
The officially supported platforms are:

- Heroku (Python or Docker)
- Digital Ocean App Platform (Docker-based)
- Google Cloud Run (Docker-based)

Heroku is the most common choice, and is the recommended option for staging sites or MVPs.

If you would like to deploy to a platform that's not listed here, please get in touch on Slack or by emailing 
cory@saaspegasus.com and I'm happy to help!
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

To use celery you will also need to create a managed Redis database,
which can be done from [this link](https://cloud.digitalocean.com/databases/new?engine=redis).

### Deploying

Once you've configured the prerequisites, deploying is just a few steps.

1. Edit the `/deploy/app-spec.yaml` file. In particular, make sure to set your Github repository and branch.
   If you aren't using Celery, you should remove the sections related to redis, and the celery-worker.
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

You may also need to run additional commands to get up and running, e.g. `./manage.py bootstrap_subscriptions`
for initializing your Stripe plan data.

### Settings and Secrets

App platform builds use the `settings_do.py` file.
You can add settings here, and use environment variables to manage any secrets,
following the `SECRET_KEY` example.

### Celery Support

To run celery workers on Digital Ocean you will need to first add a managed Redis database, then the celery
worker image.


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
## Deploying to containers with Docker

Pegasus ships with [Docker support for development](/docker/).
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

