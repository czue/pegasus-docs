# Using Waffle

[Waffle][waffle] is the top library for managing feature flags in Django. Pegasus includes configuration
for using Waffle with or without teams.

If you are using [Teams](/teams.md) then the Waffle flags can be turned on based on the user or the team.
If you are not using Teams then flags only apply to users.

[waffle]: https://waffle.readthedocs.io/en/stable/

## Usage

Waffle can be used to turn on and off features:

```python
import waffle

def my_view(request):
    if waffle.flag_is_active(request, 'flag_name'):
        """Behavior if flag is active."""
    else:
        """Behavior if flag is inactive."""
```

The flags themselves are managed via the Django Admin site where each flag can be activated for specific
users or teams or based on certain conditions such as *superuser* status.

For full details on configuring flags see the [Flag Attributes][flag-attribs] of the Waffle docs.

[flag-attribs]: https://waffle.readthedocs.io/en/stable/types/flag.html#flag-attributes

Flags may be used in, views, templates, Javascript and more. For full details see the [Waffle docs][waffle-usage]

[waffle-usage]: https://waffle.readthedocs.io/en/stable/usage/index.html

## Usage with *Teams*

If you are using [Teams](/teams.md), Pegasus ships with a [custom flag model][custom-model] which allows you
to activate flags on a per team basis as well as the default options.

[custom-model]: https://waffle.readthedocs.io/en/stable/types/flag.html#custom-flag-models

## Example usage

The *Employees* example app demonstrates how to use Waffle in Python, Django templates and in JS files.

This example creates an `employee-salary` flag and uses it to determine if the employee 'salary' field
should be shown in the list and edit UI.

The flag is configured in [test mode][test-mode] which
allows us to activate the flag with a URL parameter. You can see it in action here:

[test-mode]: https://waffle.readthedocs.io/en/stable/testing/user.html

**Django Forms Example**
This uses the Python and Django template features of Waffle:

* With the flag active: [employees/objects/django/?dwft_employee-salary=1][django-active]
* With the flag inactive: [employees/objects/django/?dwft_employee-salary=0][django-inactive]

[django-active]: http://localhost:8000/pegasus/employees/objects/django/?dwft_employee-salary=1
[django-inactive]: http://localhost:8000/pegasus/employees/objects/django/?dwft_employee-salary=0

**React Example**
This uses the Javascript features of Waffle:


* With the flag active: [employees/objects/react/?dwft_employee-salary=1][react-active]
* With the flag inactive: [employees/objects/react/?dwft_employee-salary=0][react-inactive]

[react-active]: http://localhost:8000/pegasus/employees/objects/react/?dwft_employee-salary=1
[react-inactive]: http://localhost:8000/pegasus/employees/objects/react/?dwft_employee-salary=0

