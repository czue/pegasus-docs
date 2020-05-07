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

1. In `assets/index.js` remove the `require('./javascript/pegasus/examples/object-lifecycle.js');` line.
1. In `assets/javascript/pegasus/Pegasus.js` remove the `export { Charts } from './examples/Charts';` line.
1. In `webpack.config.js`, delete the `'object-lifecycle'` entry from `module.exports`.
1. Delete the `assets/javascript/pegasus/examples` folder.
1. Delete `static/js/object-lifecycle-bundle.js`


## Delete Teams Code

If you aren't using teams there are a few things you can cleanup on the front end code.

1. In `assets/index.js` remove the `require('./javascript/pegasus/teams.js');` line.
1. In `webpack.config.js`, delete the `teams` entry from `module.exports`.
1. Delete `assets/javascript/pegasus/teams.js`.
1. Delete `static/js/teams-bundle.js`


## Delete Stripe Code

If you aren't using Stripe you can remove the code and dependencies by doing the following steps.

**Note that this requires starting from a fresh database, or manually manipulating your migrations.**

1. In `settings.py` remove `djstripe` from `THIRD_PARTY_APPS`
1. In `settings.py` remove `STRIPE_` and `DJSTRIPE_` settings
1. Delete `pegasus/apps/components/stripe.py`
1. Delete `pegasus/utils/subscriptions.py`
1. Delete `pegasus/decorators.py`
1. In `pegasus/apps/users/models.py`, remove code referencing `SubscriptionModelMixin` (import and class inheritance)
1. In `pegasus/apps/users/models.py`, remove `customer` and `subscription` fields on the model
1. In `pegasus/apps/teams/models.py`, remove code referencing `SubscriptionModelMixin` (teams build only)
1. In `pegasus/apps/users/models.py`, remove `subscription` field on the model (teams build only)
1. Delete `templates/stripe` directory
1. In `assets/javascript/pegasus/Pegasus.js` remove `export { Payments } from './Payments';` line.
1. Delete `assets/javascript/pegasus/Payments.js`.
1. Delete your `apps/users/migrations` folder.
1. Delete your `apps/teams/migrations` folder.
1. Run `./manage.py makeimgrations users` (teams build only)
1. Run `./manage.py makeimgrations teams` (teams build only)
1. Run `./manage.py migrate`
1. In `requirements/requirement.in` and `requirements/requirement.txt` remove `stripe` and `dj-stripe` lines.
1. Run `pip-sync`

## Using the Payments example

To make the Payments example a part of your own application, see [this page](payments.md).
