# Forms

Pegasus ships with some extensions to Django forms to integrate with different CSS frameworks
and add some extensions.

## The `form_tags` module

You can use default Django form rendering for forms, but if you want all the built-in style support,
you should instead use the utilities in the `form_tags` module.

To use it, first include `form_tags` in any Django template file:  

```
{% load form_tags %}
```

Then, you can render a form using the `render_form_fields` template tag.
Here is a basic example:

```html
<form method="post">
  {% csrf_token %}
  {{ form.non_field_errors }}
  {% render_form_fields form %}
  <div class="mt-2">
    <button class="pg-button-primary" type="submit">{% translate "Submit Form" %}</button>
  </div>
</form>
```

You can also render individual fields using `render_field`:

```html
<form method="POST" action="{% url 'account_change_password' %}" class="password_change">
  {% csrf_token %}
  {{ form.non_field_errors }}
  {% render_field form.username %}
  {% render_field form.password %}
  <div class="mt-2">
    <input class="pg-button-primary" type="submit" value="{% translate 'Login' %}">
  </div>
</form>
```

## Dynamic forms with Alpine.js

*Added in version 2023.6*

The form rendering helpers also support adding attributes, which can be useful to add Alpine.js to make
a form more dynamic.

For example, you can bind a form value to an alpine model by passing it in `attrs` like this:

```python
class ExampleFormAlpine(forms.Form):
    YES_NO_OTHER = (
        ("yes", gettext("Yes")),
        ("no", gettext("No")),
        ("other", gettext("Other")),
    )
    like_django = forms.ChoiceField(
        label=gettext("Do you like Django?"),
        choices=YES_NO_OTHER,
        widget=forms.Select(attrs={"x-model": "likeDjango"}),  # this line will bind the value to an alpine model
    )
```

Then in the HTML template you have to add an alpine model to the form:

```html
<form method="post" x-data="{ likeDjango: '{{ form.like_django.data|default:"yes" }}', styleValue: '{{ form.styled_options.data|default:"regular" }}' }">
  <!--  other fields here  -->
  {% render_field form.like_django %} <!--  this will bind to `likeDjango` above  -->
```

The `render_field` tags support two special syntaxes to make using alpine easier:

1. Any attribute starting with `x` will be automatically converted to `x-`.
2. Double underscores (`__`) will be replaced with colons (`:`).

The following example alpine form and template (which also ship with Pegasus, available at
[http://localhost:8000/pegasus/forms/alpine/](http://localhost:8000/pegasus/forms/alpine/)) demonstrate this usage,
including hiding/showing a field based on the value of another field, rendering field values in labels,
and changing the style of a field based on its value.

Django form class:

```python
class ExampleFormAlpine(forms.Form):
    YES_NO_OTHER = (
        ("yes", gettext("Yes")),
        ("no", gettext("No")),
        ("other", gettext("Other")),
    )
    STYLES = (
        ("regular", gettext("Normal")),
        ("success", gettext("Success")),
        ("danger", gettext("Danger")),
    )
    like_django = forms.ChoiceField(
        label=gettext("Do you like Django?"),
        help_text=gettext("Try choosing 'other' to see unhiding a form field based on a value."),
        choices=YES_NO_OTHER,
        widget=forms.Select(attrs={"x-model": "likeDjango"}),
    )
    like_django_other = forms.CharField(label=gettext("Please specify more details about your answer."))
    styled_options = forms.ChoiceField(
        label=gettext("Styled Options"),
        help_text=gettext("Try picking an option to see how you can style a component based on its value."),
        choices=STYLES,
        widget=forms.Select(attrs={"x-model": "styleValue"}),
    )
```

Django template:

```html
<form method="post" x-data="{ likeDjango: '{{ form.like_django.data|default:"yes" }}', styleValue: '{{ form.styled_options.data|default:"regular" }}' }">
  {% csrf_token %}
  {% render_field form.like_django %}
  {% render_field form.like_django_other xshow="likeDjango === 'other'" xcloak='True' %}
  {% render_field form.styled_options xbind__class="'pg-bg-' + styleValue" %}
  <p class="mt-4">You can also use alpine to display selected values.
    <em>
      Like Django: <strong x-text="likeDjango"></strong>,
      Style: <strong x-text="styleValue"></strong>
    </em>
  </p>
  <input type="submit" value="Save Data" class="pg-button-secondary mt-2">
</form>
```
