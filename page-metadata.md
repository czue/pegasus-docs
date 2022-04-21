Project/Page Metadata and SEO
=============================

Pegasus comes with some built in tools and best-practices for setting page-level metadata (e.g. title, image URL, etc.).

## The `PROJECT_METADATA` setting

Your Pegasus project will ship with a `settings.py` variable called `PROJECT_METADATA` with the following values:

```python
PROJECT_METADATA = {
    'NAME': '<your app name>',
    'URL': '<your app domain>',
    'DESCRIPTION': '<your app description>',
    'IMAGE': 'https://upload.wikimedia.org/wikipedia/commons/2/20/PEO-pegasus_black.svg',
    'KEYWORDS': 'SaaS, django',
    'CONTACT_EMAIL': '<your email>',
}
```

This information will be available in every view under the variable name `project_meta`.
Out of the box, the values are used in a number of places, though can be overridden/modified at the view level.

## Page Titles

The default title for your pages will be your project name and description from `PROJECT_METADATA`.

If you want to add a custom page title, you can pass a `page_title` context variable to the template.

For example:

```python
def my_new_view(request):
    return render('a/template.html', {'page_title': 'My New Page'})
```

Pegasus will then set your title to be "My New Page | <project name>".

If you'd like to change the way the title is formatted (e.g. remove the project name), you can change
that behavior in `web.templatetags.meta_tags.get_title`.

In Pegasus versions after 2022.4 you can also override the title directly in a template by overriding the `page_title` block.

For example:

```html
{% block page_title %}This title will be used instead of the Pegasus versions{% endblock %}
```
