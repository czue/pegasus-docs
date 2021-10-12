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
