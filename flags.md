# Feature Flags

[Waffle][waffle] is the top library for managing feature flags in Django. Pegasus includes configuration
for using Waffle with or without teams.

If you are using [Teams](/teams.md) then the Waffle flags can be turned on based on the user or the team.
If you are not using Teams then flags only apply to users.

[waffle]: https://waffle.readthedocs.io/en/stable/

## Usage

Waffle can be used to turn on and off features. For example:

```python
import waffle

def my_view(request):
    if waffle.flag_is_active(request, 'flag_name'):
        """Behavior if flag is active."""
    else:
        """Behavior if flag is inactive."""
```

The flags themselves are managed via the Django Admin site where each flag can be activated for specific
users or teams, or based on certain conditions such as *superuser* status. Flags can also be managed
via the command line.

For full details on configuring flags see the [Flag Attributes][flag-attribs] of the Waffle docs.

[flag-attribs]: https://waffle.readthedocs.io/en/stable/types/flag.html#flag-attributes

Flags may be used in views, templates, JavaScript and more. For full details see the [Waffle docs][waffle-usage]

[waffle-usage]: https://waffle.readthedocs.io/en/stable/usage/index.html

## Usage with *Teams*

If you are using [Teams](/teams.md), Pegasus ships with a [custom flag model][custom-model] which allows you
to activate flags on a per-team basis in addition to the other default options.

[custom-model]: https://waffle.readthedocs.io/en/stable/types/flag.html#custom-flag-models

## Example usage

To see flags in actions look at the "Flags" example in the Pegasus Example Gallery.

The flag in the example is configured in [test mode][test-mode] which allows us to activate the
flag with a URL parameter.

[test-mode]: https://waffle.readthedocs.io/en/stable/testing/user.html
