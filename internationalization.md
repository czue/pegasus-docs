# Internationalization

Pegasus supports internationalization via built-in support for timezones and language translations.
To enable timezone and multi-language support, you must select the "use internationalization" option
in your project settings.

## Translation Demo

This two-minute demo highlights how translations work in Pegasus apps.

<div style="position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden; max-width: 100%; height: auto; margin-bottom: 1em;">
    <iframe src="https://www.youtube.com/embed/MhxKdkUFUj8" frameborder="0" allowfullscreen style="position: absolute; top: 0; left: 0; width: 100%; height: 100%;"></iframe>
</div>

## Localization

Pegasus ships with full support for localizing user-facing text.

Currently, not all the user-facing text is properly tagged for localization but this will be incrementally addressed
in future releases.

For full documentation on localization see the [Django docs](https://docs.djangoproject.com/en/4.1/topics/i18n/).

## Big picture

Big picture there are two steps to translation:

1. **Define the text you want to translate (in Python, HTML, or JavaScript)**. This step happens in your project's code.
2. **Add a translation for that text to other languages**. This step happens in your project's translation files, 
   which can be found in the `locale/<lang_code>/LC_MESSAGES/` folders (there will be one for each language).

## Managing enabled languages

There are two steps to updating the list of languages that will be available on your site.
The first step is to define it in `settings.LANGUAGES`.
Out of the box this will be English and French:

```python
from django.utils.translation import gettext_lazy

LANGUAGES = [
    ('en', gettext_lazy('English')),
    ('fr', gettext_lazy('French')),
    # add other languages here
]
```

The second step is to create the translations folder for the language.
This can be done by running:

```shell
python ./manage.py makemessages -l [new lang code] --ignore node_modules --ignore venv
```

Or in Docker:


```shell
docker compose exec web python manage.py makemessages -l [new lang code] --ignore node_modules --ignore venv
```

## Marking text in your app for translation

All text you want to be translatable must be tagged in your application. This can be done as follows:

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

## Creating / updating translation files

After you've marked text for translation, you'll need to update your language files.
This can be done by running:

```shell
python ./manage.py makemessages --all --ignore node_modules --ignore venv
python ./manage.py makemessages -d djangojs --all --ignore node_modules --ignore venv
```

Or in Docker:

```
make translations
```

Note: if you get any errors you may need to [install gettext](https://stackoverflow.com/q/35101850/8207).

## Adding actual translations for other languages

To add a translation for another language you need to edit that languages messages (.po) file.

For example, to edit a French translation, you would update
`locale/fr/LC_MESSAGES/django.po`.
Then search for the text you want to translate, and add the French translation:

```
msgid "My Team"
msgstr "Mon Équipe"
```

The above lines will replace "My Team" with "Mon Équipe" whenever the French language is configured.

After editing any message (.po) file, you will have to compile the messages for the updates to show up in your app.
This can be done by:

```shell
python ./manage.py compilemessages
```

Or in Docker:

```
make translations
```

## Technical notes

Pegasus is configured to use cookies to track the current locale.
This allows localization to work for both authenticated and unauthenticated users.

More information on this approach is available the Django docs: [How Django discovers language preference][1]

[1]: https://docs.djangoproject.com/en/4.2/topics/i18n/translation/#how-django-discovers-language-preference

## Timezones

Pegasus includes support for user's setting their own time zones via their profile (version 2023.7 and later).
When a user sets a timezone, it will be automatically activated by the `UserTimezoneMiddleware` so that
by default all dates and times will appear in their local time.

For more information on working with timezones in Django, see [Django's timezone documentation][2].

[2]: https://docs.djangoproject.com/en/4.2/topics/i18n/timezones/
