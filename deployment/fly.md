## Fly.io

Pegasus supports container-based deployment to [Fly.io](https://fly.io/).

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
```

After running that, answer 'yes' to the first question:

```
An existing fly.toml file was found for app {app-name}
? Would you like to copy its configuration to the new app? Yes
```

Fly will output some details, then ask another question about customizing. Answer 'yes' to that as well:

```
Using dockerfile Dockerfile.web
Creating app in /path/to/app/source

We're about to launch your app on Fly.io. Here's what you're getting:

Organization: Your Name              (fly launch defaults to the personal org)
Name:         {app-name}             (specified on the command line)
Region:       Chicago, Illinois (US) (specified on the command line)
App Machines: shared-cpu-1x, 1GB RAM (most apps need about 1GB of RAM)
Postgres:     <none>                 (not requested)
Redis:        <none>                 (not requested)

? Do you want to tweak these settings before proceeding? (y/N) Yes
``` 

A browser tab should open where you should add a Fly Postgres database called {app-name}-db,
and an Upstash Redis server. You can leave the other defaults. It should look like this:

![Fly DB config](/images/deployment/fly-db-config.png)

Click "Confirm Settings" and then close the tab.
Back on the command line, Fly will output some more things and should eventually end with a message like this:

```
✓ Configuration is valid
Your app is ready! Deploy with `flyctl deploy`
```

If you see these two lines you are ready to deploy!
If not, see the "Troubleshooting" section below.

<!----
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
    The following secret was added to <app name>:
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

--->
### Deploying

You are now ready to deploy your app.

```shell
$ fly deploy
```

**After deploying, review the [production checklist](/deployment/production-checklist.md) for a list
of common next steps**

### Running Database Migrations

Database migrations are applied in the release command during deploy. This is configured in the `fly.toml` file.

### Settings and Secrets

Fly.io builds use the `settings_production.py` file.
You can add settings here or in the base `settings.py` file, and use environment variables to manage any secrets,
following the examples in these files.

Secrets are managed in Fly.io via the web UI or on the command line using the CLI:

```shell
$ fly secrets set MY_VAR=secret_value
```

### Running One-Off Commands

You can one-off commands via a shell:

```shell
$ fly ssh console

app $ ./code/manage.py [command]
```

### Celery Support

Out of the box, Pegasus is configured to run Celery using the [multiprocess support][multiprocess] provided by
Fly.io.

For alternatives see [https://fly.io/docs/app-guides/multiple-processes/](https://fly.io/docs/app-guides/multiple-processes/)

[multiprocess]: https://fly.io/docs/reference/configuration/#the-processes-section
 
### Troubleshooting

**My release / migrate command is failing.**

If you get an error like the following when running `fly deploy`

```
  django.db.utils.OperationalError: connection to server at "localhost" (127.0.0.1), port 5432 failed: Connection refused
  	Is the server running on that host and accepting TCP/IP connections?
```

it is likely that your Database is not set up properly.
You should confirm it is connected and attached to your app.
You can (re-)attach a database to an app by running:

```
fly postgres attach {your-app-db} -a {your-app-name}
```
