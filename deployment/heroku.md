# Heroku

Pegasus supports deploying to Heroku as a standard Python application or using containers.

Before getting started, first take the following steps in Heroku:

1. In the Heroku dashboard, create a new app.
2. Set up the [Heroku CLI](https://devcenter.heroku.com/articles/heroku-command-line) and run `heroku login` locally.

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

### Set up Databases

To set up your Postgres database, first enable the addon in the UI or by running:

```
heroku addons:create heroku-postgresql
```

Database migrations should be handled automatically by Heroku.

If you want to use Redis as a cache or to use Celery, you will need to install [the Heroku Redis addon](https://elements.heroku.com/addons/heroku-redis)
from the UI or by running:

```
heroku addons:create heroku-redis
```


### Setting allowed hosts

In your `settings_heroku.py` file make sure to change the `ALLOWED_HOSTS` setting 
to include whatever app you're deploying.

```
ALLOWED_HOSTS = [
    'myapp.herokuapp.com',
]
```

### Deploying

Both builds can be deployed using Heroku's standard git integration.
After you've connected your project's git repository to Heroku, just run:

```
git push heroku main
```

You can also configure Heroku to automatically build from a branch of your git repository.


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

### Building the front end

As of Pegasus version 0.19, Heroku container builds will automatically build your front end files for you.
You don't need to do anything to set this up. 

If you're using Heroku's Python support you can also configure Heroku to build your front-end files for you.

To set this up, all you need to do is add the `heroku/nodejs` buildpack to your application from the settings page.

Just make sure that this buildpack runs *before* the `heroku/python` buildpack, so that the compiled files
are available when the `collectstatic` command runs.

### Celery support

The Heroku environment supports Celery out-of-the-box.

Additionally, you may need to run the following command to initialize a Celery worker:

```
heroku ps:scale worker=1
```

This process should be the same for Python and containerized builds.
