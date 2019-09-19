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
1. Delete your root `urls.py`, remove the line containing `include('pegasus.apps.examples.urls')`.
1. In `apps/web/templates/web/components/top_nav.html`, remove the section linking to the Examples Gallery.
1. Delete the *entire* `pegasus/apps/examples` directory.

### Cleaning the front-end code

1. In `assets/index.js` remove the `require('./javascript/pegasus/examples/object-lifecycle.js');` line.
1. In `webpack.config.js`, delete the `'object-lifecycle'` entry from `module.exports`.
1. Delete the `assets/javascript/pegasus/examples` folder.
1. Delete `static/js/object-lifecycle-bundle.js`


## Delete Teams Code

If you aren't using teams, you can delete the following additional code around it.

1. In `apps/web/templates/web/components/app_nav.html`, remove the section enclosed by `{% if PEGASUS_USING_TEAMS %}`.
1. In `pegasus/context_processors`, remove `PEGASUS_USING_TEAMS` from the `pegasus_settings` function.
1. In `settings.py`, remove the `PEGASUS_USING_TEAMS` variable declaration.
1. In `settings.py`, remove `'pegasus.context_processors.pegasus_settings'` from `context_processors`
1. Delete the *entire* `pegasus/apps/teams` directory.
1. Delete the *entire* `pegasus/templates/teams` directory.

### Cleaning the front-end code

1. In `assets/index.js` remove the `require('./javascript/pegasus/teams.js');` line.
1. In `webpack.config.js`, delete the `teams` entry from `module.exports`.
1. Delete `assets/javascript/pegasus/teams.js`.
1. Delete `static/js/teams-bundle.js`


