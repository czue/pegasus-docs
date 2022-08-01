# Internationalization

## Localization

Pegasus ships with full support localizing user facing text.

Currently, not all the user facing text is properly tagged for localization but this will be addressed
in future releases.

For full documentation on localization see the [Django docs](https://docs.djangoproject.com/en/4.0/topics/i18n/).

### Configuration

Define the list of languages that will be available on your site:

```python
from django.utils.translation import gettext_lazy

LANGUAGES = [
    ('en', gettext_lazy('English')),
    ('de', gettext_lazy('German')),
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

### Important note
Since string extraction is done by the xgettext command, only syntaxes supported by gettext are supported
by Django. In particular, Python f-strings are not yet supported by xgettext, and JavaScript template
strings need gettext 0.21+.
