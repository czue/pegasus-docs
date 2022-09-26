## Fly.io

Pegasus supports deploying to [Fly.io](https://fly.io/) as a standard Python application or using containers.

### Prerequisites

If you haven't already, install the [flyctl CLI](https://fly.io/docs/hands-on/install-flyctl/).

The create an account with `fly auth signup` or login with `fly auth login`.

### Setup

Once you have logged in via the CLI you can create your app and the services it will need.
For each of the commands below follow the prompts given.

```shell
# Answer 'yes' to the first question:
#   An existing fly.toml file was found for app wedding-plan
#   Would you like to copy its configuration to the new app? (y/N) y

$ fly launch --dockerfile Dockerfile.web --no-deploy

$ fly postgres create
$ fly postgres attach attach {postgres name} -a {app name}
# this will set the DATABASE_URL secret for you

$ fly redis create
$ fly secrets set REDIS_URL={url}
```

### Deploying

You are now ready to deploy your app.

```shell
$ fly deploy
```

### Running Database Migrations

Database migrations are applied via during deploy. This is configured in the `fly.toml` file.

You may also need to run additional commands to get up and running, e.g. `./manage.py bootstrap_subscriptions`
for initializing your Stripe plan data. This can be done via a shell:

```shell
$ fly ssh console

app $ ./manage.py [command]
```

### Settings and Secrets

Fly.io is configured to use the `settings_production.py` file.
You can add settings here or in the base `settings.py` file. Use environment variables to manage any secrets,
following the `SECRET_KEY` example.

Secrets are managed in Fly.io via the web UI or on the command line using the CLI:

```shell
$ fly secrets set MY_VAR=secret_value
```

### Celery Support

Out of the box, Pegasus is configured to run Celery using the [multiprocess support][multiprocess] provided by
Fly.io.

For alternatives see https://fly.io/docs/app-guides/multiple-processes/

[multiprocess]: https://fly.io/docs/reference/configuration/#the-processes-section
 
