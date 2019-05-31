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

The sign up workflow is managed by [django-allauth](https://www.intenct.nl/projects/django-allauth/).

### Requiring email confirmation

Pegasus does not require users to confirm their email addresses prior to logging in. 
However, this can be easily changed by changing the following value in `settings.py`

```python
ACCOUNT_EMAIL_VERIFICATION = 'optional'  # change to "mandatory" to require users to confirm email before signing in.
```

## Google Analytics

To enable Google Analytics, just add your analytics tracking ID to `settings.py`:

```python
GOOGLE_ANALYTICS_ID = 'UA-XXXXXXX-1' 
```
