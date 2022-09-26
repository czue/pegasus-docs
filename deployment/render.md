## Render

Pegasus supports deploying to [Render](https://render.com/) as a standard Python application or using containers.

### Prerequisites

If you haven't already, create your Render account.

To use celery you will need to upgrade to a paid plan.

### Deploying

Once you've logged into Render you can create your app as follows:

1. In the Render dashboard, create a new blueprint
2. Connect your GitHub or Gitlab account and select your project's repository
3. Configure the branch you want to deploy from
4. Review the configuration, add settings and click 'Approve'

This will kick off the process to create your PostgreSQL database and Redis instances as well
as deploy your application.

### Running Database Migrations

Database migrations are applied via the Docker startup script or else the build script if you are
not using Docker.

You may also need to run additional commands to get up and running, e.g. `./manage.py bootstrap_subscriptions`
for initializing your Stripe plan data. This can be done via a Render shell (paid plan required).

### Settings and Secrets

Render builds use the `settings_production.py` file.
You can add settings here or in the base `settings.py` file, and use environment variables to manage any secrets,
following the `SECRET_KEY` example.

### Celery Support

To run celery workers on Render you will need a paid plan.

Then in your `render.yaml` file uncomment the 'celery' section.
