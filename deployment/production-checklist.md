# Production checklist

The following are some recommendations for deploying production Pegasus applications.

## Run the Django deployment checklist

Django provides a [deployment checklist](https://docs.djangoproject.com/en/stable/howto/deployment/checklist/) that
helps ensure your site has some of the most important settings properly configured for production environments.
It is executed by running `manage.py check --deploy` on your production server.

It's recommended to run this on your production application and address any critical issues.

The default Pegasus configuration will contain some warnings, to help prevent misconfigurations which
can affect your site's availability. Not all warnings are serious issues and some may not be possible to address 
(e.g. if part of your site must be available over HTTP instead of HTTPS).
After running the `manage.py check --deploy` command you should read through the documentation for any issues you get
and update the relevant settings where necessary.

*Note: The "unable to guess serializer" warnings are safe to ignore, and will be fixed in a future version of Pegasus.*

(set-your-allowed-hosts)=
## Set your `ALLOWED_HOSTS`

In your app's `settings_production.py` be sure to update the [`ALLOWED_HOSTS` setting](https://docs.djangoproject.com/en/4.1/ref/settings/#allowed-hosts)
with the domain(s) you want the site to be available from, replacing the `'*'` that is there by default:

```python
ALLOWED_HOSTS = [
    'example.com',  # use your app's domain here
]
```

Failure to do this opens up your site to more HTTP host header attacks.

## Update your Django Site

In order for absolute URLs and JavaScript API clients to work, your Django site should match your application's domain.
See the documentation on [absolute URLs](../configuration.md#absolute-urls) to do this.

## Set up email

If you haven't already, you'll want to set up your site to [send email](../configuration.md#sending-email)

## Make sure your secrets are set

Application secrets (e.g. API keys, passwords, etc.) are managed in environment variables.
Ensure that you have configured the following variables (if you are using them):

- All apps should set `SECRET_KEY` to a long, randomly-generated value.
- If you're using Stripe, you should set the `STRIPE_TEST_PUBLIC_KEY`, `STRIPE_TEST_SECRET_KEY`, 
`STRIPE_LIVE_PUBLIC_KEY`, and `STRIPE_LIVE_SECRET_KEY` config vars (or whatever subset you are using).
- If you set up email, ensure whatever keys/secrets you need are set.
- If you're using Mailchimp, set `MAILCHIMP_API_KEY` and `MAILCHIMP_LIST_ID`.
- If you're using Health Checks, set `HEALTH_CHECK_TOKENS`.

Refer to your [chosen platform's documentation](/deployment.rst) for details on how to set environment variables in that platform.

## Sync Stripe data

After setting up your Stripe variables per above, you'll want to run:

```
python manage.py bootstrap_subscriptions
```

to initialize your subscription data.

See your [chosen platform's documentation](/deployment.rst) for how to run one-off commands.

## Set up media files

Some functionality, like user profile pictures, requires saving user-uploaded files.
In development these are saved to the file system, but in most production environments the file system
is not usable for it. Instead, you need to set up an external storage to handle these.

There is guidance on configuring media files in the [settings and configuration docs](../configuration.md#storing-media-files).

The most common choice of external storage is [Amazon S3](https://aws.amazon.com/s3/),
though many cloud providers have their own S3-compatible options, e.g. [Digital Ocean Spaces](https://www.digitalocean.com/products/spaces).

## Check your static file setup

By default, Pegasus uses [whitenoise](https://whitenoise.readthedocs.io/en/stable/index.html) for static files.
**If you keep the default setup, you do not need to change anything.**
Static files will be built and collected as part of the build process of your Docker container and should be available
on your production site.

If you decide to switch to serving files externally, for example, using Amazon S3,
then you may need to modify your static file set up for some platforms.
This is because production secrets necessary to save files to S3 may not be available during the Docker container build.

If this is the case, you should modify your deployment set up so that `python manage.py collectstatic --noinput` is run
at the same time as Django database migrations, so that the necessary secrets are available to the application.
The exact way to do this will vary by deployment platform.

## Optimize your front end

The front-end files that ship with Pegasus are the developer-friendly versions.
In production, these should be optimized.

First you should add the compiled files to your `.gitignore` as described in the [front end docs](../front-end.md#long-term-best-practices).
Then, as part of your CI/CD deployment process, you should build the bundle files directly on your production server 
(using `npm install && npm run build`).

This will ensure that the latest, optimized version of the front-end code is always deployed
as part of your production environment.

The platform-specific docs have some guidance on setting this up where possible.

## Update other configuration options

See [the configuration page](/configuration.md) for a larger list of options,
including social login, sign up flow changes, analytics, logging, adding captchas, and so on.

## Set up monitoring

It's highly recommended to enable Sentry and connect it to your application so that you can
see any errors that are encountered.

It's also recommended to enable the health check endpoint and connect it to a monitoring tool
like [StatusCake](https://www.statuscake.com/) or [Uptime Robot](https://uptimerobot.com/) so that
you can be alerted whenever your site or services are having an outage.
The URL you should connect is: `yourdomain.com/health/`.

If you have the "Health Check Endpoint" option enabled for your project you should also ensure that
you have set the `HEALTH_CHECK_TOKENS` environment variable to a secure value. This can be a comma-separated
list of tokens that are required to access the health check endpoint. For example:

```
HEALTH_CHECK_TOKENS=secrettoken1,secrettoken2
```

Then your health check endpoint will only be accessible at:
`https://yourdomain.com/health/?token=secrettoken1` and `https://yourdomain.com/health/?token=secrettoken2`

These URLs can then be connected to a monitoring tool to ensure that only your monitoring tool
(or anyone who knows the token) can access the health check endpoint.

## Double-check your language settings

Make sure your [internationalization settings](../internationalization.md) are correct, and you don't have
any extra languages in `settings.LANGUAGES` that you don't currently support.
This is especially important if you are using Wagtail, as links to pages in unsupported languages
may error or return the wrong results.

## Consider switching to `psycopg2` source distribution

For ease of development, Pegasus ships with the `psycopg2-binary` package which is used for connecting
to PostgreSQL however the [psycopg documentation](https://www.psycopg.org/docs/install.html#psycopg-vs-psycopg-binary)
recommends using the source distribution (`psycopg2`) in production environments.

The issues mentioned in the documentation mostly impact non-Docker deployments.

### Switching from `psycopg2-binary` to `psycopg2`

1. In `requirements/requirements.in`, replace `psycopg2-binary` with `psycopg2`
2. [Re-build](../customizations.md#python-packages) your requirement TXT files

If you are using the Dockerfiles shipped with Pegasus you should not need to make any changes
however if you are running your Pegasus app directly on a VM you will need to make sure the
[build prerequisites](https://www.psycopg.org/docs/install.html#build-prerequisites) are installed before
deploying the requirements changes.
