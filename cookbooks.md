# Cookbooks

Step-by-step guide to some different you might want to do with Pegasus.

Pegasus customers can request content be added to this page! Just get in touch via support.

## Use the Django Admin UI

Pegasus ships with a simple script to promote any user to a superuser who can access
the Django admin.

After going through the sign up flow, to convert your newly-created user into an admin, 
run the following command, being sure to replace the email address with the one you used to sign up:

```
python ./manage.py promote_user_to_superuser yourname@example.com
``` 

Now you should be able to access the django admin at http://localhost:8000/admin

## Delete Pegasus Examples

The Pegasus examples are great for seeing a fully-working end-to-end feature,
but in your app you probably don't want them around long term. To remove the examples entirely, 
take the following steps:

### Cleaning the back-end code

1. In `settings.py`, remove `'pegasus.apps.examples.apps.PegasusExamplesConfig'` from `PEGASUS_APPS`.
1. In your root `urls.py`, remove the line containing `include('pegasus.apps.examples.urls')`.
1. In `apps/web/templates/web/components/top_nav.html`, remove the section linking to the Examples Gallery.
1. Delete the *entire* `pegasus/apps/examples` directory.
1. Delete the *entire* `templates/pegasus/examples` directory.

### Cleaning the front-end code

1. In `webpack.config.js`, delete the `'react-object-lifecycle'`, `vue-object-lifecycle` and `'pegasus'` entries from `module.exports`.
1. Delete the `assets/javascript/pegasus/` folder.
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
