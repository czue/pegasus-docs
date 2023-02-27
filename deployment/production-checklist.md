# Production checklist

The following are some recommendations for deploying production Pegasus applications.

## Run the Django deployment checklist

Django provides a [deployment checklist](https://docs.djangoproject.com/en/3.2/howto/deployment/checklist/) that
helps ensure your site has some of the most important settings properly configured for production environments.
It is executed by running `manage.py check --deploy` on your production server.

It's recommended to run this on your production application and address any critical issues.

The default Pegasus configuration will contain some warnings, to help prevent misconfigurations which
can affect your site's availability. Not all warnings are serious issues and some may not be possible to address 
(e.g. if part of your site must be available over HTTP instead of HTTPS).
After running the `manage.py check --deploy` command you should read through the documentation for any issues you get
and update the relevant settings where necessary.

*Note: The "unable to guess serializer" warnings are safe to ignore, and will be fixed in a future version of Pegasus.*

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
See the documentation on [absolute URLs](https://docs.saaspegasus.com/configuration.html#absolute-urls) to do this.

## Set up email

If you haven't already, you'll want to set up your site to [send email](https://docs.saaspegasus.com/configuration.html#sending-email)

## Make sure your secrets are set

Application secrets (e.g. API keys, passwords, etc.) are managed in environment variables.
Ensure that you have configured the following variables (if you are using them):

- All apps should set `SECRET_KEY` to a long, randomly-generated value.
- If you're using Stripe, you should set the `STRIPE_TEST_PUBLIC_KEY`, `STRIPE_TEST_SECRET_KEY`, 
`STRIPE_LIVE_PUBLIC_KEY`, and `STRIPE_LIVE_SECRET_KEY` config vars (or whatever subset you are using).
- If you set up email, ensure whatever keys/secrets you need are set.
- If you're using Mailchimp, set `MAILCHIMP_API_KEY` and `MAILCHIMP_LIST_ID`.

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

There is guidance on configuring media files in the [settings and configuration docs](https://docs.saaspegasus.com/configuration.html#storing-media-files).

The most common choice of external storage is [Amazon S3](https://aws.amazon.com/s3/),
though many cloud providers have their own S3-compatible options, e.g. [Digital Ocean Spaces](https://www.digitalocean.com/products/spaces).

## Optimize your front end

The front-end files that ship with Pegasus are the developer-friendly versions.
In production, these should be optimized.

First you should add the compiled files to your `.gitignore` as described in the [front end docs](https://docs.saaspegasus.com/front-end.html#long-term-best-practices).
Then, as part of your CI/CD deployment process, you should build the bundle files directly on your production server 
(using `npm install && npm run build`).

This will ensure that the latest, optimized version of the front-end code is always deployed
as part of your production environment.

The platform-specific docs have some guidance on setting this up where possible.

## Update other configuration options

See [the configuration page](/configuration.md) for a larger list of options,
including social login, sign up flow changes, analytics, logging, and so on.

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
