# Internationalization

## Demo

This two-minute demo highlights how translations work in Pegasus apps.

<div style="position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden; max-width: 100%; height: auto; margin-bottom: 1em;">
    <iframe src="https://www.youtube.com/embed/MhxKdkUFUj8" frameborder="0" allowfullscreen style="position: absolute; top: 0; left: 0; width: 100%; height: 100%;"></iframe>
</div>

## Localization

Pegasus ships with full support for localizing user-facing text.

Currently, not all the user-facing text is properly tagged for localization but this will be incrementally addressed
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

After running this, you can update the language files as new strings are added by running:

```shell
python ./manage.py makemessage --all --ignore node_modules --ignore venv
python ./manage.py makemessages -d djangojs --all --ignore node_modules --ignore venv
python ./manage.py compilemessages
```

### Basic Usage

**In Python:**
```python
from django.utils.translation import gettext

def my_view(request):
    output = gettext("Welcome to my site.")
    return HttpResponse(output)
```

See the [Django docs](https://docs.djangoproject.com/en/4.0/topics/i18n/translation/#internationalization-in-python-code) for more.

**In Django templates:**
```djangotemplate
{% load i18n %}
<title>{% translate "This is the title." %}</title>
```

See the [Django docs](https://docs.djangoproject.com/en/4.0/topics/i18n/translation/#internationalization-in-template-code) for more.

**In JavaScript:**
```javascript
document.write(gettext('this is to be translated'));
```

See the [Django docs](https://docs.djangoproject.com/en/4.0/topics/i18n/translation/#internationalization-in-javascript-code) for more.

**In Wagtail:**

See the [Wagtail docs](https://docs.saaspegasus.com/wagtail#internationalization).

## Technical notes

Pegasus is configured to use cookies to track the current locale.
This allows localization to work for both authenticated and unauthenticated users.

More information on this approach is available the Django docs: [How Django discovers language preference][1]

[1]: https://docs.djangoproject.com/en/4.1/topics/i18n/translation/#how-django-discovers-language-preference
