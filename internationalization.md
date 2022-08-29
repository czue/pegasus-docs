# Internationalization

## Localization

Pegasus ships with full support for localizing user facing text.

Currently, not all the user facing text is properly tagged for localization but this will be addressed
in future releases.

For full documentation on localization see the [Django docs](https://docs.djangoproject.com/en/4.0/topics/i18n/).

### Configuration

Define the list of languages that will be available on your site:

```python
from django.utils.translation import gettext_lazy

LANGUAGES = [
    ('en', gettext_lazy('English')),
    ('fr', gettext_lazy('French')),
]
```

When adding a new language you will need to create the language files by running:

```shell
python ./manage.py makemessages -l [new lang code] --ignore node_modules --ignore venv
```

In steady state the language files can be kept up to date by running:
```shell
python ./manage.py makemessage --all --ignore node_modules --ignore venv
python ./manage.py makemessages -d djangojs --all --ignore node_modules --ignore venv
python ./manage.py compilemessages
```

## Technical notes
Pegasus is configured to use cookies to track the current locale instead of prefixed URLs.
This allows localization to work for both authenticated and unauthenticated users.

More information on this approach is available the Django docs: [How Django discoveres language preference][1]

[1]: https://docs.djangoproject.com/en/4.1/topics/i18n/translation/#how-django-discovers-language-preference
