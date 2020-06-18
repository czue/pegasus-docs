Version History and Release Notes
=================================

## Version 0.7.2

- Improved styling of Stripe credit card forms in subscriptions and payments examples
- Fixed bug in subscriptions where not setting a default plan prevented the UI from working
- Fixed bug where monthly subscriptions would not work if you also had a quarterly or 6-month price configured
- Changed the order of some examples in examples home page and navigation

*June 17 2020*

## Version 0.7.1

- **Added ability to cancel a Subscription directly on the site.**
  [Demo](https://www.saaspegasus.com/subscriptions/) (you have to create a Subscription first)
- Don't show password change links if using social authentication (thanks Yaniv!)

*June 11, 2020*

## Version 0.7.0

Pegasus now supports [Vue.js](https://vuejs.org/)!

Version 0.7 adds a Vue.js implementation of the Object Lifecycle demo so you can
start with a foundation of either React or Vue.

Minor changes:

- Added `Membership` inline admin editing to `Teams` model (thanks Troels!)
- Added a few more spacing utility css classes to `utilities.sass`

*June 5, 2020*

## Version 0.6.1

- Upgrade `requests` to version `0.23.0` to fix installation version conflict.

*May 26, 2020*

## Version 0.6.0

This release begins the move of Pegasus's core functionality out of the "pegasus" app and into
the user-defined apps. It's a relatively big release from a code perspective, although there is very
little new / changed in terms of functionality.

### Philosophy

The philosophy guiding this change is "your starting code base should be as understandable as possible".

Historically, Pegasus has attempted to separate "Pegasus-owned" files from "user-owned" files.
The thinking behind this structure was that Pegasus upgrades and code merges would be as easy as possible since - 
in theory - users would not need to modify anything in the "Pegasus-owned" space.

In practice, the line between "Pegasus-owned" and "user-owned" was fuzzy, and customizing
an application often required editing files in the "Pegasus-owned" space.
So the benefit was not realized.

Furthermore, this split caused the initial codebase to be more confusing, since core functionality was split
across two places.

### Changelog

Changes related to the restructure above include:

- Moved all base templates from `templates/pegasus/` to `templates/web/`
- Moved team invitation templates from `templates/pegasus/email/` to `templates/teams/email/`
- Merged `pegasus/apps/users` and all related code into `apps/users`
- Merged `pegasus/apps/teams` and all related code into `apps/teams`
- Removed `pegasus/apps/components` and moved code to more specific apps - further details below
- Moved `promote_user_to_superuser` management command into `users` app
- Moved `bootstrap_subscriptions` management command into `subscriptions` app
- Moved `google_analytics_id` context processor to web app
- Moved `meta.py` from pegasus app to web app
- Moved `pegasus/utils/subscriptions.py` to `apps/subscriptions/helpers.py`
- Added `apps.utils` and moved most of `pegasus.utils` there
- Moved `pegasus/decorators.py` to `apps/utils/decorators.py`
- Moved `PegasusBaseModel` to `apps.utils` and renamed to `BaseModel`
- Removed unused `stripe.py` file (replacing functionality with equivalent functions in `djstripe`)
- Removed "Pegasus" from API url names
- Moved non-example JS imports out of `pegasus/Pegasus.js` and into `App.js`
- Lowercased (and kebab-cased) all JS file names for consistency
- Moved sass files from `assets/styles/pegasus`, to `assets/styles/app`
- Renamed various layout classes, e.g. `pegasus-two-column`, `pegasus-message`, etc. to `app-[name]`
- Moved `static/images/pegasus/undraw/` folder to `static/images/undraw`
- Removed unused `pegasus/teams/serializers.py` file


Other changes and fixes include:

- Fix accepting an invitation if you're already signed in
- Remove subscription-related fields from team and user models if not using subscriptions
- Only ship base migration files for users and teams, and have applications manage their own migrations 
- Improved checks and error-handling around accepting invitations multiple times
- Upgraded `bulma` CSS framework to `0.8.2`
- Upgraded `node-sass` to `4.14.1`
- Ran `npm audit fix` to update other JS library dependencies
- Upgraded `pip-tools` and other packages in `dev-requirements.txt`
- Added backend check to the payments example, to show how to prevent client-side exploits
- Improve password reset email copy
- Use `$link` color in `$navbar-item-active-color` and `$menu-item-active-background-color`
- Externalized styles on progress bar demo

### A Note on Database Migrations

Historically, Pegasus has shipped with complete database migrations.
However, maintaining a set of migrations for each possible Pegasus configuration or forcing all configurations
to use the same DB schema has proven unwieldy. Thus, migrations are now expected to be managed *outside of Pegasus*.

For new users, the only change is that prior to running `./manage.py migrate` for the first time,
you must first run `./manage.py makemigrations`.

For existing users you can either keep your current migrations folder, or you can run 
`./manage.py makemigrations` and then `./manage.py migrate --fake`. If you have changed the user or team
models, then you should keep your current folder.

*May 19, 2020*

## Version 0.5.2

- Fixed default Postgres DB settings (adding host and port)
- Removed Stripe webhooks from project urls if not using subscriptions
- Fixed but where subscription and teams templates were still being included even if not enabled for a project
- upgrade `celery-progress` to 0.0.10
- Cleaned up progress bar demo
  - Used javascript cookie library for CSRF token
  - Switched from `var` to `const` in a few places
  - Removed debug logging statements

*May 7, 2020*

## Version 0.5.1

Version 0.5.1 is a minor maintenance release with a few minor bug fixes and bits of cleanup:

- Upgraded Django to 3.0.5
- Fixed bug where certain input types were getting overridden in Django Forms (thanks Yaniv for reporting!) 
- Fixed bug with Object Lifecycle and Charts demos not working on Team installations (thanks Greg for reporting!)
- Moved example chart JavaScript to the webpack pipeline and share Api access variables (thanks Greg for the suggestion!)
- Switch `admin.py`  files to use the `@admin.register` decorator syntax

*April 17, 2020*

## Version 0.5

This is the biggest release to Pegasus since it's launch.
Read below for all the details.

### Subscriptions!

Added the [Stripe Subscriptions feature](https://www.saaspegasus.com/content/subscriptions-overview).

Documentation for subscriptions can be [found here](/subscriptions).

#### Model changes

- Added a `subscription` field to `Team` and `CustomUser` objects, and a `customer` field to `CustomUser`.
- Added `SubscriptionModelMixin` helper class for accessing / checking subscription status on a model.

### Javascript build changes

- Added `Pegasus.js` and made different modules available in front end code 
  (see subscriptions upgrade page example usage).

### Sass / CSS changes

- Added `tooltip` utilities.
- Added a few margin helper classes (e.g. `my-1`, `my-2` )

### New Python Library Dependencies

- [attrs](https://www.attrs.org/en/stable/)
- [dj-stripe](https://dj-stripe.readthedocs.io/en/stable/)

### New JavaScript Library Dependencies

- [js-cookie](https://github.com/js-cookie/js-cookie)

### Small fixes and changes:

- Moved app-specific templates from inside the apps to global templates directory as recommended by 
  Two Scoops of Django
- Remove redundant raw prefix on some `path` url declarations
- Reduced some duplicate access to `team` object when already available via the `request` object.
- Made team permission template tags more consistent with rest of site (also allow access to superusers)
- Removed `PEGASUS_USING_TEAMS` and `pegasus_settings` context processor. All config is now handled at installation
  time instead of by settings variables.
- Catch Stripe card errors in the payments example
- Upgraded various `npm` packages
- Upgraded Bulma to 0.8.0

### Documentation

- Added release notes page (this one)
- Added [subscriptions overview page](/subscriptions)
- Updated "delete teams code" cookbook to reflect latest team changes 
  (all the backend work is now done for you on installation)

*March 30, 2020*

## Version 0.4 and earlier

Release notes for earlier versions are tracked at [https://www.saaspegasus.com/releases/](https://www.saaspegasus.com/releases/).
