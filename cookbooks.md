# Cookbooks

Step-by-step guides to some different things you might want to do with Pegasus.

## Use the Django Admin UI

Pegasus ships with a simple script to promote any user to a superuser who can access
the Django admin.

After going through the sign up flow, to convert your newly-created user into an admin, 
run the following command, being sure to replace the email address with the one you used to sign up:

**Docker:**

```
docker compose exec web python ./manage.py promote_user_to_superuser yourname@example.com
```

**Native:**

```
python ./manage.py promote_user_to_superuser yourname@example.com
``` 

Now you should be able to access the django admin at http://localhost:8000/admin

## Migrating to auto-formatted code

As of February, 2023 all Pegasus projects have the option to auto-format your Python code.

To migrate a project from non-formatted to formatted code, you can go through the following steps:

1. First, do a full Pegasus upgrade to the version you want to update to, as described [here](./upgrading.md).
   **Do *not* check the "autoformat" checkbox yet.**
2. Next, run the formatting tools on your project's `main` branch: 
   1. Install black and isort: `pip install black isort`
   2. Run black: `black --extend-exclude migrations --line-length 120 .`
   3. Run isort: `isort -l 120 --profile black .`
3. Commit the result:
   1. `git add .`
   2. `git commit -m "apply formatting changes"`
4. Finally, check the "autoformat" box on your Pegasus project, and do *another* upgrade according to the same process.

## Delete Pegasus Examples

On versions 2023.2 and later, you can remove the Pegasus examples by updating your build configuration
and re-downloading (or [upgrading](upgrading.md)) your codebase.

For earlier versions you can use the following instructions:

### Remove the navigation elements

To *hide* the examples, remove the link from `apps/web/templates/web/components/top_nav.html` linking to the Examples Gallery.

The exact code will depend on your selected CSS framework/theme, but will look something like this:

```html
<li class="nav-item">
  <a class="nav-link {% if '/pegasus/' in request.path %}active{% endif %}" href="{% url 'pegasus_examples:examples_home' %}">
    {% translate "Examples Gallery" %}
  </a>
</li>
```

Note that hiding the navigation won't remove the examples.
Your users could still find them by navigating to the /pegasus/ endpoint in a browser.

To remove the examples entirely, take the following additional steps:

### Clean the back-end code

1. In `settings.py`, remove `PEGASUS_APPS` and the reference in `INSTALLED_APPS`.
1. In your root `urls.py`, remove the lines containing `include('pegasus.apps.examples.urls')` and `include('pegasus.apps.employees.urls')`.
1. Delete the *entire* top-level `pegasus/` directory.
1. Delete the *entire* `templates/pegasus/` directory.

### Clean the front-end code

1. In `webpack.config.js`, delete the `'react-object-lifecycle'`, `vue-object-lifecycle` and `'pegasus'` entries from `module.exports`.
1. Delete the `assets/javascript/pegasus/` folder.
1. Delete `assets/javascript/api-client/apis/PegasusApi.ts` and remove it from `assets/javascript/api-client/apis/index.ts`
1. Delete `static/js/react-object-lifecycle-bundle.js`, `static/js/vue-object-lifecycle-bundle.js` and `static/js/pegasus-bundle.js`

## Delete Stripe Code

If you aren't using Stripe you can remove the code and dependencies by doing the following steps.

**Note that this requires starting from a fresh database, or manually manipulating your migrations.**

1. In `settings.py` remove `djstripe` from `THIRD_PARTY_APPS`
1. In `settings.py` remove `STRIPE_` and `DJSTRIPE_` settings
1. Delete `apps/utils/decorators.py`
1. Delete `templates/stripe` directory
1. Delete the `@import "stripe.sass"` line from `assets/styles/app/_all.sass`
1. Delete `assets/styles/app/stripe.sass`

If you have not removed the examples using the method above, you also need
to delete the payments example. This can be done with the following steps:

1. Delete the paths starting with `/payments/` from `pegasus/apps/examples/urls.py`.
1. Delete references to `Payment` from `pegasus/apps/examples/admin.py`.
1. In `pegasus/apps/examples/views/__init__.py` delete line `from .payments import *`
1. Delete `pegasus/apps/examples/views/payments.py`.
1. In `pegasus/apps/examples/models/__init__.py` delete line `from .payments import Payment`
1. Delete `pegasus/apps/examples/models/payments.py`.
1. Delete the relevant sections of `templates/pegasus/examples/examples_home.html` and 
  `templates/pegasus/examples/components/examples_nav.html` 
1. Delete the `templates/pegasus/examples/payments/` folder.
1. In `assets/javascript/app.js` remove `export { Payments } from './Payments';` line.
1. Delete `assets/javascript/payments.js`.
1. Run `./manage.py makemigrations` and `./manage.py migrate`

Finally, remove the library dependencies:

1. In `requirements/requirement.in` and `requirements/requirement.txt` remove `stripe` and `dj-stripe` lines.
1. Run `pip-sync`

## Using the Payments example

To make the Payments example a part of your own application, see [this page](payments.md).
