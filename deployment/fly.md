## Fly.io

Pegasus supports deploying to [Fly.io](https://fly.io/) as a standard Python application or using containers.

### Prerequisites

If you haven't already, install the [flyctl CLI](https://fly.io/docs/hands-on/install-flyctl/).

The create an account with `fly auth signup` or login with `fly auth login`.

### Setup

Once you have logged in via the CLI you can create your app and the services it will need.
For each of the commands below follow the prompts given.

In the example below the "Chicago, Illinois (US) (ord)" region is selected. You may change
the region to suit your needs, but it should be consistent throughout the commands.

1. Create your app in Fly.io

    ```shell
    $ fly launch --dockerfile Dockerfile.web \
      --dockerignore-from-gitignore \
      --no-deploy \
      --name {app-name} \
      --region ord
    
    # Answer 'yes' to the first question:

    An existing fly.toml file was found for app {app-name}
    ? Would you like to copy its configuration to the new app? Yes
    
    Creating app in /path/to/app/source
    Using dockerfile Dockerfile.web
    Selected App Name: {app-name}
    
    # Select the organization you want to deploy the app to:

    ? Select organization: My Org (my_org)
    
    Created app {app-name} in organization my_org
    Wrote config file fly.toml
    Your app is ready. Deploy with `flyctl deploy`
    ```

2. Create the app database

    ```shell
    $ fly postgres create --name {app-name}-db --region ord
    
    # ? Select Organization: My Org
    # ? Select configuration: Development - Single node, 1x shared CPU, 256MB RAM, 1GB disk
    ```

3. Attach the DB to your app

    ```shell
    $ fly postgres attach {app-name}-db -a {app-name}
    
    Postgres cluster {app-name}-db is now attached to {app-name}
    The following secret was added to wedding-plan1:
      DATABASE_URL=postgres://.....
    ```

4. Create the Redis instance
    
    ```shell
    $ fly redis create --name {app-name}-redis --region ord
    
    ? Select Organization: My Org (my_org)
    ? Would you like to enable eviction? Yes
    ? Select an Upstash Redis plan Free: 100 MB Max Data Size
    Your Upstash Redis database {app-name}-redis is ready.
    Apps in the personal org can connect to at redis://.....

    ```

5. Set the `REDIS_URL` secret

    Using the Redis URL from the command above run:

    ```shell
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
 
