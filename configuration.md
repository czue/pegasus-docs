# Settings and Configuration

This section describes some of the settings and configuration details you can change inside Pegasus.

## Project Metadata

When you first setup Pegasus it populated the `PROJECT_METADATA` setting in `settings.py` with various 
things like page titles and social sharing information.

These settings can later be changed as you like by editing the setting directly:

```python
PROJECT_METADATA = {
    'NAME': 'Your Project Name',
    'URL': 'http://www.example.com',
    'DESCRIPTION': 'My Amazing SaaS Application',
    'IMAGE': 'https://upload.wikimedia.org/wikipedia/commons/2/20/PEO-pegasus_black.svg',
    'KEYWORDS': 'SaaS, django',
    'CONTACT_EMAIL': 'you@example.com',
}
```

## Absolute URLs

When you first install Pegasus it will use the `URL` value from `PROJECT_METADATA` above to create
a Django `Site` object in your database. 
This `Site` is used to generate absolute URLs which are needed whenever you link to the site externally---for
example in emails, social authentication, or Stripe APIs.

If you need to change the URL after installation, you can go to the site admin at `admin/sites/site/` and
modify the values accordingly, leaving off any http/https prefix.

In development you'll typically want a domain name of `localhost:8000`, and in production this should
be the domain where your users access your app.


## Sending Email

Pegasus is setup to use [django-anymail](https://github.com/anymail/django-anymail) to send email
via Amazon SES, Mailgun, Postmark, and a variety of other email providers.

To use one of these email backends, simply change the the email backend in `settings.py` to:

```python
EMAIL_BACKEND = 'anymail.backends.mailgun.EmailBackend'
```

And populate the `ANYMAIL` setting with the required information. For example, to use [Mailgun](https://www.mailgun.com/)
you'd just populate the following values:

```python
ANYMAIL = {
    "MAILGUN_API_KEY": "key-****",
    "MAILGUN_SENDER_DOMAIN": '{{project_name}}.com',
}
```

If you are in the EU you may also need to add the following entry:

```python
    'MAILGUN_API_URL': 'https://api.eu.mailgun.net/v3',
```

The [anymail documentation](https://anymail.readthedocs.io/en/stable/) has much more information on these options.

The following django settings should also be set:

```python
SERVER_EMAIL = 'noreply@{{project_name}}.com'
DEFAULT_FROM_EMAIL = 'you@{{project_name}.com'
ADMINS = [('Your Name', 'you@{{project_name}}.com'),]
```

See [Sending email](https://docs.djangoproject.com/en/3.2/topics/email/) in the django docs for more information.

## User Sign Up

The sign up workflow is managed by [django-allauth](https://www.intenct.nl/projects/django-allauth/)
with a sensible set of defaults and templates.

### Google / Social login

Pegasus optionally ships with a "Login with Google" option. 
Just enter "y" to the option during installation to start with a default configuration.

You'll separately need to follow the steps [listed here](https://django-allauth.readthedocs.io/en/latest/providers.html#google)
to configure things on the Google side and in the Django Admin.

It's easy to add or change social login details (e.g. login with Facebook, Twitter, Github, etc.) using allauth.

For details on how to set this up for a particular provider [see this page](https://django-allauth.readthedocs.io/en/latest/providers.html).

If you need help setting this up feel free to get in touch!

### Requiring email confirmation

Pegasus does not require users to confirm their email addresses prior to logging in. 
However, this can be easily changed by changing the following value in `settings.py`

```python
ACCOUNT_EMAIL_VERIFICATION = 'optional'  # change to "mandatory" to require users to confirm email before signing in.
```

*Note: The email verification step will be skipped if using a Google / social login.*

### Disabling public sign ups

If you'd like to prevent everyone from signing up for your app, you set the following in your `settings.py`,
replacing the existing value:

```python
ACCOUNT_ADAPTER = 'apps.users.adapter.NoNewUsersAccountAdapter'
```

This will prevent all users from creating new accounts, though existing users can continue to login and use the app.

### Further configuration

Allauth is highly configurable.
It's recommended that you look into the various [configuration settings availabile within allauth](https://django-allauth.readthedocs.io/en/latest/configuration.html)
for any advanced customization.


## Stripe

If you're using [Stripe](https://www.stripe.com/) to collect payments you'll need to fill in the following in `settings.py`
(or populate them in the appropriate environment variables):

```python
STRIPE_LIVE_PUBLIC_KEY = os.environ.get("STRIPE_LIVE_PUBLIC_KEY", "<your publishable key>")
STRIPE_LIVE_SECRET_KEY = os.environ.get("STRIPE_LIVE_SECRET_KEY", "<your secret key>")
STRIPE_TEST_PUBLIC_KEY = os.environ.get("STRIPE_TEST_PUBLIC_KEY", "<your publishable key>")
STRIPE_TEST_SECRET_KEY = os.environ.get("STRIPE_TEST_SECRET_KEY", "<your secret key>")
STRIPE_LIVE_MODE = False  # Change to True in production
```

## Google Analytics

To enable Google Analytics, add your analytics tracking ID to `settings.py`:

```python
GOOGLE_ANALYTICS_ID = 'UA-XXXXXXX-1' 
```

Pegasus uses a "global site tag" with gtag.js by default, which is a simpler version of Google Analytics that can be rolled out
with zero additional configuration.
If you use Google Tag Manager, you can make changes in `templates/web/components/google_analytics.html` to
match the snippet provided by Google.

See [this article](https://support.google.com/tagmanager/answer/7582054) for more on the differences between
gtag.js and Google Tag Manager.


## Sentry

[Sentry](https://sentry.io/) is the gold standard for tracking errors in Django applications and
Pegasus can connect to it with a few lines of configuration.

To get setup, add the following code snippet to your `settings.py` file:

```python
import sentry_sdk
from sentry_sdk.integrations.django import DjangoIntegration

sentry_sdk.init(
    dsn='https://****@sentry.io/12345',
    integrations=[DjangoIntegration()]
)
```

If you are starting from `settings_production.example.py` then you need to populate `SENTRY_DSN` from your
Sentry project settings.


## Celery

See the [celery docs](/celery) for set up and configuration of Celery.


## Mailing List

Pegasus includes support for subscribing users to a [Mailchimp](https://mailchimp.com/) email list upon signup.

If you would like to enable this functionality you must:

1. Install the `mailchimp3` Python package.
2. Add the following variables to `settings.py`:

```python
MAILCHIMP_API_KEY = '<Mailchimp api key>'
MAILCHIMP_LIST_ID = '<Mailchimp list ID>'
```

Note that it is your responsibility to notify your users / get their consent as per your local privacy regulations.
