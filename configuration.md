# Settings and Configuration

This section describes some of the settings and configuration detials you can change inside Pegasus.

## Project Metadata

Pegasus uses the `PROJECT_METADATA` setting in `settings.py` to generate various things
like page titles and social sharing information.

These should be changed to match the desired values for your own project.

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

Of particular importance is the `URL` setting, which is used to generate absolute 
URLs in your application.

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

The [anymail documentation](https://anymail.readthedocs.io/en/stable/) has much more information on these options.

The following django settings should also be set:

```python
SERVER_EMAIL = 'noreply@{{project_name}}.com'
DEFAULT_FROM_EMAIL = 'you@{{project_name}.com'
ADMINS = [('Your Name', 'you@{{project_name}}.com'),]
```

See [Sending email](https://docs.djangoproject.com/en/2.2/topics/email/) in the django docs for more information.

## User Sign Up

The sign up workflow is managed by [django-allauth](https://www.intenct.nl/projects/django-allauth/)
with a sensible set of defaults and templates.

### Requiring email confirmation

Pegasus does not require users to confirm their email addresses prior to logging in. 
However, this can be easily changed by changing the following value in `settings.py`

```python
ACCOUNT_EMAIL_VERIFICATION = 'optional'  # change to "mandatory" to require users to confirm email before signing in.
```

### Social login

It's easy to add social login (e.g. login with Google, Facebook, Twitter) to your site using allauth.

For details on how to set this up for a particular provider [see this page](https://django-allauth.readthedocs.io/en/latest/providers.html).

If you need help setting this up feel free to get in touch!

### Further configuration

Allauth is highly configurable.
It's recommended that you look into the various [configuration settings availabile within allauth](https://django-allauth.readthedocs.io/en/latest/configuration.html)
for any advanced customization.


## Google Analytics

To enable Google Analytics, just add your analytics tracking ID to `settings.py`:

```python
GOOGLE_ANALYTICS_ID = 'UA-XXXXXXX-1' 
```

## Sentry

[Sentry](https://sentry.io/) is the gold standard for tracking errors in Django applications and
Pegasus can connect to it with just a few lines of configuration.

To get setup, simply add the following code snippet to your `settings.py` file:

```python
import sentry_sdk
from sentry_sdk.integrations.django import DjangoIntegration

sentry_sdk.init(
    dsn='https://****@sentry.io/12345',
    integrations=[DjangoIntegration()]
)
```

If you are starting from `settings_production.example.py` then you just need to populate `SENTRY_DSN` from your
Sentry project settings.


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
