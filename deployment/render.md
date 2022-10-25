## Render

Pegasus supports deploying to [Render](https://render.com/) as a standard Python application or using containers.

### Prerequisites

If you haven't already, create your Render account.

To use celery you will need to upgrade to a paid plan.

### Deploying

Once you've logged into Render you can create your app as follows:

1. In the Render dashboard, create a new blueprint
2. Connect your GitHub or Gitlab account and select your project's repository
3. Configure the *Service Group Name* and select the branch you want to deploy from
4. Review the configuration, add settings and click 'Apply'

This will kick off the process to create your PostgreSQL database and Redis instances as well
as deploy your web application.

### Start Script

The `docker_startup.sh` file is run by Render to start your app.
This allows running "release" commands along with the web process, as [described here](https://community.render.com/t/release-command-for-db-migrations/247/2).

Out of the box the `docker_startup.sh` file is where your database migrations will run,
just before your app starts.

If there are other commands, like `./manage.py bootstrap_subscriptions` that you want to run on every deploy you can add them here.

### Settings and Secrets

Render builds use the `settings_production.py` file.
You can add settings here or in the base `settings.py` file, and use environment variables to manage any secrets,
following the `SECRET_KEY` example.

### Running One-Off Commands 

You can run one-off commands in the Render shell (paid plan required) or [via SSH](https://render.com/docs/ssh).

### Celery Support

To run celery workers on Render you will need a paid plan.

Then in your `render.yaml` file uncomment the 'celery' section.
