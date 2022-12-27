Version History and Release Notes
=================================

Releases of [SaaS Pegasus: The Django SaaS Boilerplate](https://www.saaspegasus.com/) are documented here.

## Version 2022.12

This is a maintenance release with many small fixes and quality-of-life improvements suggested by the community.

Happy holidays!

### Added

- Added a `make upgrade` target to update docker containers and a local database after upgrading.
- Added a Redis instance to Github actions CI setup, so that any tests which depend on Redis can run without modification.
- Added default error pages and url routes for 400 and 403 errors.

### Changed

- **Updated usage of `.env` files. Python environments now use `.env`, docker uses `.env.docker`, and the example
  was renamed from `.env.dev.example` to `.env.example`. [Details here](./configuration.md).** See upgrade notes.
- **Invitations can now only be accepted from the email address that was invited.** See upgrade notes.
- Blog posts in wagtail are now listed in descending order (by date)
- Optimized the `Dockerfile.dev` so that requirements are installed prior to copying the complete source code of the project.
  This results in substantially reduced image build times when requirements have not changed.
- Replaced instances of `%` string formatting with `str.format()`
- Update `make pip-compile` and `make requirements` build targets to also build production requirements. (thanks Brett!)
- Updated error pages to use a shared base template.
- Added translation markup to a handful of places.


### Fixed

- Remove `customer` from the user admin for team builds
  (this was causing the User admin to error for team-based builds if subscriptions were enabled).
- Update the `bootstrap_content` management command to publish content (in addition to creating it).
- Fixed crashing error on `@active_subscription_required` decorator if user was not logged-in.
- Fixed image responsiveness on Wagtail blog posts on Bootstrap-based builds.
- Set `LOGIN_URL` in settings, which prevents issues arising if you change the default location of `allauth` urls.
- Fixed image styling on the teams list page if you weren't a member of any teams. (htmx builds)
- Fixed a minor markdown-styling issue in the README.
- Automatically sync Stripe API keys from settings/environment to database when running `bootstrap_subscriptions`.
  This fixes the following error on new projectsc: "You don't have any API Keys in the database. Did you forget to add them?"
- Removed unused import of C3 styles from the charts example template.
- Fixed a static image reference on the example pricing page.

### Upgrade notes

**`.env` updates**

The moving around of `.env` files may impact existing development environments.
 For those using Docker, it is recommended to rename your `.env.dev` to `.env.docker`.

**Invitation changes**

If you wish to preserve the previous behavior that allowed accepting an invitation from any email address:

1. Remove the email address validation check in `apps.teams.forms.TeamSignupForm` by deleting these three lines:

```python
    if invite.email != email:
        raise forms.ValidationError(_(
            'You must sign up with the email address that the invitation was sent to.'
        ))
```

2. In `templates/account/signup.html` make the email field editable by changing the following line

```html
    <input type="hidden" name="{{ form.email.name }}" value="{{ invitation.email }}"/>
```

to:

```
    {% render_text_input form.email %}
```

*Dec 27 2022*

## Version 2022.11.1

This release is a minor/hotfix update with a few small changes.

### Added

- Added `clear_cached_subscription` helper function to `SubscriptionModelBase`

### Changed

- Upgraded dj-stripe to 2.7.2. This fixes several crashing errors when using more complex subscription/billing models.

### Fixed

- Catch and fix Stripe errors when trying to view a cancelled subscription that has not been synchronized with Stripe 

*Nov 4 2022*

## Version 2022.11

There are a number of big updates in this release.

### Feature: Feature flag support

Pegasus now supports using feature flags with waffle.
For full details, see the new [feature flag documentation](./flags.md).

Included in the implementation:

- A custom `Flag` model that allows turning features on and off for an entire `Team`.
- A new example page showing how to use feature flags in Python, Django templates, and JavaScript
- Added some helper CSS classes to display badges (used in the example)

### Cleanup: Settings Overhaul

The main change is to move most configurable settings into environment variables (now managed by
[`django-environ`](https://django-environ.readthedocs.io/en/latest/)), and reduce the number of settings files used.

Supporting/related work:

- **Switch environment variables in settings to use `django-environ` and made more settings configurable via environment variables.**
- Support configuring database with single `DATABASE_URL` setting if defined.
- Moved redis configuration to default `settings.py` and allow overriding with environment variables.
- Renamed all platform-specific settings files (e.g. `settings_heroku.py`) to `settings_production.py`.
- **Removed `settings_docker.py` which was used in development with Docker. Docker-specific settings are now overridden
  via environment variables in the `.env.dev` file.**
- Replaced usage of `django-heroku` with normal settings. Previously this was used
  to configure the database URL and whitenoise for Heroku. Both of those changes
  have been rolled into the default settings files.

### Feature: Render deployment option

Pegasus now officially supports deploying to [Render](https://render.com/).
See the new [Render deploy documentation](./deployment/render.md).

### Feature: Fly.io deployment option

Pegasus now officially supports deploying to [fly.io](https://fly.io/).
See the new [Fly.io deploy documentation](./deployment/fly.md).

### Feature + Cleanup: Subscription updates

Most of these changes are backend cleanups to help improve future Subscription work.
*Full support for multiple subscriptions and metered billing will hopefully be in an upcoming release.*

**Enabled backend support for subscriptions with multiple products.**

Related changes:

- Switched all references of djstripe's deprecated `Plan` model to use the `Price` model.
- Updated Subscription serialization to support multiple items / prices / products and changed
  `PlanSerializer` to `PriceSerializer`.
- Changed suffix in property names in `ProductWithMetadata` class from `_plan` to `_price`.
- `active_subscription_required` decorator now checks all prices/products associated with a subscription if
  `limit_to_plans` is specified.
- Removed `get_product_and_metadata_for_subscription` and `get_subscription_metadata` functions.
- Added `SubscriptionWrapper` class to help encapsulate more complex subscription logic to a single place,
  and use it instead of `Subscription` objects and lots of extra context variables in templates.
- Added `InvoiceFacade` class to provide a few utilities to help display a Stripe Invoice object.

**Enabled backend support for [usage-based/metered billing](https://stripe.com/docs/billing/subscriptions/usage-based).**

Related changes:

- Fixed crashes on subscription signup and details pages if your plan was configured with metered billing.
- Added demo form for reporting usage to Stripe for metered plans.

**Changed the Stripe Customer object to be associated with the Team (instead of the User) for Team-based projects.**

Related changes:

- Remove `customer` from the `CustomUser` model and add it to `SubscriptionModelBase` (which will go to the `Team` 
  on team-based builds)
- Set the `DJSTRIPE_SUBSCRIBER_MODEL` to the Team object, if using teams.
- Updated logic that gets/sets the customer to use the Team instead of the user.
- No longer force using the logged-in User's email address at checkout (for team based builds),  
  so they can specify a different billing email.
- No longer re-use credit cards / Customers when the same user signs up for subscriptions in multiple teams.

*See the upgrading guide below for details on navigating this change.*

### Other Changes

- **Upgraded font awesome to the latest version (6.2) and load the CSS from a CDN.**
- Upgraded djstripe to version 2.6.2
- Added user and team metadata to the Stripe subscription object during Checkout, so Subscriptions
  can be more easily linked back to your app from the Stripe Dashboard.
- Added translation markup to a handful of pages.
- Switched Google / Twitter brand icons to be displayed inline, and use icon-sized images.
- **Replaced "icon" CSS class with "pg-icon" to avoid conflicts with framework classes.**
  In particular, this fixes some issues with Creative Tim themes beyond what's included in Pegasus.
- Removed icons from subscription plan selector.
- **Use whitenoise for static files in development if deployment is configured for it.**
  This makes development environments more like production, though is largely invisible.
- **All docker-based deployments now build front end assets as part of the deploy step**.
  Previously this was only done for Heroku-based deploys.
- **All docker-based deployments are now based off the `buster` image.** 
- Added `--noinput` to heroku migrations command.
- Deployments that run Celery now run it with the `--beat` option by default, and a concurrency of 2.
- Increased length of generated secret key in production environment files.
- Upgraded Node versions that run on CI to 16, 18, and 19 to reflect the current releases.
- Upgraded Github actions versions for several steps to fix Node deprecation warnings.
- Production Dockerfiles now run gunicorn by default (this can be overridden in the platform-specific tools,
  e.g. to run celery)
- Rebuilt the JavaScript API client to reflect the latest API views and serializers.

### Other Fixes

- Fixed all schema warnings from `drf-specatcular`, by adding inline serializers or annotations to several APIs.
- Fixed styling bug in showing a user's connected accounts on Tailwind builds.
- Properly hide delete button text on small screens in HTMX object demo.
- Replaced many instances of `trans` with `translate` and `blocktrans` with `blocktranslate`.
- Set the page title of Wagtail blog posts to the blog post title.
- Fix active tab highlighting in examples navigation for some CSS frameworks.

### Removed

- Deleted no-longer-needed static images for Twitter / Google logos.
- Removed unused "is-small" class from various icon markup.
- Removed unused styling block for `th` elements in `subscriptions.sass`

### Upgrading

The major change that requires care in upgrading is the migration of the `customer` field from the User to the Team
for team-based builds.

If you do NOT have any live customers associated with your Users the migration process is standard,
just run `./manage.py makemigrations` followed by `./manage.py migrate`.
**Note: this will drop all User-->Customer relationships.** 

If you DO have live customers associated with your Users, and you want to preserve this data,
the migration process is more complicated:

First, add the following line *back* to the `CustomUser` model:

```python
   customer = models.ForeignKey(Customer, null=True, blank=True, on_delete=models.SET_NULL)
```

Next, create and run migrations:

```bash
./manage.py makemigrations 
./manage.py migrate
```

Next you need to move the `Customer` from the user to their team. You can do this by running:

```
./manage.py migrate_customers_to_teams
```

If the customer only had a single subscription, it will be correctly applied to the right team.
If the customer had multiple subscriptions, the command will print out errors *which you must resolve manually*.
You can either choose to drop this information, or associate the customer with a single Team. 

Once you are happy with how you've migrated the data you can remove the `customer` field from above
from the user, and run `./manage.py makemigrations` followed by `./manage.py migrate` to drop it from the user table.

Feel free to reach out in #support on Slack if you need any assistance with this process!

*Nov 4 2022*

## Version 2022.10

This release adds two-factor authentication, and has a number of smaller improvements and fixes.

### Added

- **Two-factor authentication.** Users can now set up two-factor authentication on their account (using Google Authenticator or similar),
  and will be required to enter a token to login. This is configured from the user's profile page.
  More [documentation here](2fa.md).

### Changed

- Improved UI feedback in React Teams UI and Employee example when the API client can't load data.
- Wagtail and admin login pages now redirect to the main login page (this prevents users from bypassing two-factor
  authentication, if enabled).
- Added top navigation bar to tailwind account pages.
- The teams middleware will now raise a 404 if a user tries to access a page associated with a team they were not
  a member of. Previously this was only enforced by the `login_and_team_required` decorator, not the middleware.
  Also added/updated tests for this case.
- Added type hints in a few more places.
- Made minor improvements to API schemas, including fixing some warnings that arose during schema generation.
- Upgraded generated API client to version 6.2.0, and brought in changes from above schema changes.
- Upgraded Django to the latest security LTS release (3.2.16)

### Fixed

- Fix an issue in Bootstrap builds where some material styles were being applied to the initial download even if you
  didn't choose the material theme.
- Fixed a bug where certain pages would fail to load if teams were enabled but the logged in user was not a member
  of any teams.

*October 6, 2022*

## Version 2022.9

The major addition in this release is support for internationalization, a.k.a. translating your app into multiple languages.
Here is a quick demo:

<div style="position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden; max-width: 100%; height: auto; margin-bottom: 1em;">
    <iframe src="https://www.youtube.com/embed/MhxKdkUFUj8" frameborder="0" allowfullscreen style="position: absolute; top: 0; left: 0; width: 100%; height: 100%;"></iframe>
</div>

This includes support for text localization in Python, Django templates, JavaScript and Wagtail.
Translations can be set anonymously (via a cookie) or based on the logged-in user's selected language.

See the new [Internationalization docs](internationalization.md) for more information.

Major related changes:

- Added a language selector to the site footer for users who aren't logged in. 
- Added a `language` field to the `CustomUser` model and profile page.
- Added `UserLocaleMiddleware` to set the user's language whenever they are logged-in.
- Update wagtail bootstrap code to generate localizable pages.
- Added translations markup to a substantial subset of code across Django views, templates and JavaScript.
  More translation markup will be added incrementally with new releases.
- Replaced `trans` templatetag with `translate` in a number of places. 

### Other Changes

- **Attach team models to the request in middleware instead of view decorator.**
  This means that `request.team` will be available on every request, so team navigation will be available
  even on pages like a user profile that are not team-specific.
  For details see the updated [teams middleware docs](https://docs.saaspegasus.com/teams#middleware),
  as well as the upgrade notes below. 
- Added tests for the above middleware, and updated other tests to be compatible with it.
- Added a context processor so that `team` is always available in the template context.
  Also removed code setting `team` in context in `TeamObjectViewMixin`.

### Fixes

- Don't show pages on the blog index if they are published then unpublished (Wagtail only, thanks Peter for reporting!)

### Upgrade notes

- If you use *teams* and are upgrading from a older version you must make sure that your
  middleware includes `apps.teams.middleware.TeamsMiddleware`. It should be placed directly
  after `django.contrib.auth.middleware.AuthenticationMiddleware`.


*September 1, 2022*

## Version 2022.8.2

This is another hotfix release. Thanks Daniel and Jacob for reporting bugs!

- Fix error loading API Schema file when building without API keys enabled.
- Added a test that would have caught the above error.
- Removed unused site-bundle.js file.
- Fix pyyaml requirements conflict when installing with Google Cloud enabled.

*August 5, 2022*

## Version 2022.8.1

This is a hotfix release with a few fixes from the last build.
Thanks to Jim and Logan for quickly reporting issues!

### Fixes 

- Fix environment variable name used for database port in settings.py.
- Removes accidentally added text from the landing page (Tailwind builds only).
- Added instructions to run `./manage.py makemigrations` when setting up a database to the README.

*August 3, 2022*


## Version 2022.8

The major addition in this release is official support for Tailwind CSS.
There were also a number of smaller fixes and improvements.

### Official Tailwind CSS support

Pegasus now officially supports [Tailwind CSS](https://tailwindcss.com/)! Here's a 3-minute walkthrough:

<div style="position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden; max-width: 100%; height: auto; margin-bottom: 1em;">
    <iframe src="https://www.youtube.com/embed/mNxVfmoDaOQ" frameborder="0" allowfullscreen style="position: absolute; top: 0; left: 0; width: 100%; height: 100%;"></iframe>
</div>

Some of the larger changes to the Tailwind functionality include:

- Upgraded TailwindCSS to version 3 and all dependencies to the latest versions.
- Added a [daisyUI](https://daisyui.com/) dependency and overhauled most of the tailwind templates with daisyUI components.
- Overhauled navigation bar and menus, and added a daisyUI-based mobile dropdown menu for mobile navigation.
- All Tailwind pages are now mobile-friendly.
- Added `tailwindcss/typography` plugin for content.
- Improved tailwind-based Django form rendering and integrated the forms with daisyUI components.
- Improved styling of app notifications/messages.
- Added modal dialogs to the team deletion and membership removal workflows (team builds only).
- Removed many no-longer-used classes form `site-tailwind.css`.

See the updated [Tailwind docs](/css/tailwind/) for more information and customization options.

### Other Additions

- **Pegasus now ships with a default, customizable logging configuration.** [Documentation](https://docs.saaspegasus.com/configuration.html#logging).
- Added several new helper CSS classes, including `pg-content` (for content), `pg-columns-reversed` (for reversed columns),
 `pg-align-items-center` (a flexbox utility), and various `pg-text-` classes for coloring text. 

### Other Changes

- **React-based team deletion now works like the HTMX version, with a pop-up confirmation modal.**
  The "Delete" button was removed from the team list UI.
- **Deleting a team/user will now automatically cancel any subscription associated with that team/user.** (thanks Florian for reporting!)
- **Updated many templates to use `pg-` Pegasus helper CSS classes instead of CSS-framework-specific ones.**
  Affected places include: several places in the Pegasus examples, `app_home.html`, the user `profile_form.html` and 
  social account connections page, the React teams editing UI, subscription helper pages, default landing page,
  password reset pages, team invitation pages, user impersonation, wagtail / blog pages, 404 / 500 pages and more.
- Replaced underscores with hyphens in example URLs.
- Increased margins between buttons and forms/controls in a few templates.
- Improved styling of the default email management page.
- Upgraded several NPM packages to the latest versions including Bootstrap version to 5.2.
- Overhauled [CSS docs](/css/). 
- Mark `help_text` as safe in form_tags (allows adding links and other HTML to your help text)
- Add trailing slash to urls in `apps/web/urls.py`.
- Updated default CI configuration to build all pull requests (was previously all pull requests to `main` only)


### Other Fixes

- **Fixed bug where non-admins did not see a link to manage their own membership from the team details page.** (React builds only)
- Fixed styling issues with some tables on small screens.
- Removed empty if/else block from `team_nav.html` if building without subscriptions.
- Fixed a bug in `@active_subscription_required` decorator where an invalid subscription could cause a crashing error
  instead of redirecting to the subscription management page. (thanks Jon for reporting!)
- In object home examples, list "four" technologies instead of "three"

*August 1, 2022*

## Version 2022.7.1

This is a hotfix release that fixes an "Undefined variable" error when building the front end
with the Bootstrap Material theme, and Bootstrap 5.2.

More details on the root issue can be [found here](https://github.com/twbs/bootstrap/issues/36785).

*July 22, 2022*

## Version 2022.7

This release was largely focused on addressing technical debt and further modernizing some of the front end code
that ships with Pegasus.

Due to the large number of updates, a few major changes have been grouped together.

### Added TypeScript support

The Pegasus front-end build pipeline now supports [TypeScript](https://www.typescriptlang.org/).
You can continue to write code in normal JavaScript, or introduce TypeScript incrementally on new code, 
on a file-by-file basis. Future versions of Pegasus are likely to incrementally port more code to TypeScript,
similar to how types have been incrementally introduced into the Python code.

Related changes:

- Added TypeScript transpilation to the front end build pipeline.
- Added `npm run type-check` and  `npm run type-check-watch` targets.
- Added `make npm-type-check` target for Docker environments.
- Run type checks as part of the Github Actions front end build step

### Upgraded all JavaScript packages

The `npm-check-updates` command was run on the entire front end, and with the exception of TailwindCSS,
*every package in `package.json` was upgraded to the latest version*.

Some of the larger updates (for which there were also code changes to make them compatible) include:

- Upgraded Vue.js from version 2 to version 3 and made Employee app compatible with Vue 3.
- Upgraded React to version 18 and react-router to version 6, and updated all code to be compatible with new versions.

### Overhauled the API documentation and API clients

This release removed the deprecated CoreAPI-based API schemas and JavaScript client and replaced it with OpenApi3 schemas
(using [drf-spectacular](https://drf-spectacular.readthedocs.io/)) and generated client code.
For more information on using APIs in your Pegasus app, see the new [API docs](/apis/).

Details related to this change:

- Added new `drf-spectacular` Python dependency and removed the `coreapi` Python dependency.
- Added `@extend_schema` markup to most APIs to improve and simplify the generated OpenAPI3 schemas
- **Added new default endpoints for the OpenAPI3 schema.yml file, Swagger API docs, and Redoc API docs.**
- **Added a new TypeScript API client that ships with the front end code.** 
- Updated all JS client code to use the new client, including the employee demos, React-based team management views, and chart demo.
- Removed no-longer-used coreAPI helper functions from `assets/javascript/api.js`.
- Removed no-longer-needed `assets/javascript/teams/api.js`.
- Removed no-longer-needed `app.Api` from the front end code.
- Added `openapitools.json` to `.gitignore`
- Added [documentation related to these changes](/apis/).

### Other Additions

- `project_meta` context processor now includes `server_url` (the absolute URL of your app) in the context.
- `get_server_root` helper function to support the above.
- Doc strings for helper functions in `apps/web/meta.py`
- Type hints for various helper methods inside Django models.
- Add `pip-tools` to dev Dockerfile so `requirements.txt` can be built in the container. (Thanks Brett for the suggestion!) 
- Add `make pip-compile` target for rebuilding `requirements.txt` and `make requirements` file for rebuilding requirements.txt, 
  and rebuilding/restarting your Docker containers.
- Team management URL is now included in team API urls. (React UI only)

### Other Fixes

- Properly handle team name and slug validation in team editing UI (React UI only)
- Changing the team slug now properly refreshes the page, so links don't break. (React UI only)
- Fixed typo in variable name in `get_team_api_url_templates`.

### Other Changes

- **Switched chart examples and supporting code from `c3.js` to [`Chart.js`](https://www.chartjs.org/).**
- Upgraded Django to the latest LTS security release (3.2.14).
- Dev docker set up now starts celery with the beat flag enabled.
- Increased the top margin between the navigation and content in content base template. (Bootstrap only, thanks Will for suggesting!)
- Deleted some accidentally-included commented out code in the Vue employee demo. 

*July 8, 2022*

## Version 2022.6

The main feature of this release is a brand-new integration with Wagtail.
[Wagtail](https://wagtail.org/) is a powerful CMS (Content Management System) built on top of Django.
You can use it to create rich websites that can be edited directly via an authoring admin interface without writing any code.
It's great for creating marketing sites, blogs, and other mostly-static content.

You can learn more about Wagtail and Pegasus in this 5-minute overview, or by heading over to the [Wagtail documentation](/wagtail).

<div style="position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden; max-width: 100%; height: auto; margin-bottom: 1em;">
    <iframe src="https://www.youtube.com/embed/YVRhuQ9CyuQ" frameborder="0" allowfullscreen style="position: absolute; top: 0; left: 0; width: 100%; height: 100%;"></iframe>
</div>

Complete release notes are below:

### Added

- **Pegasus now supports Wagtail! [Documentation](/wagtail/).** There are a fair number of changes to support this work, most
  of which are only relevant if you enable wagtail support. The main ones includ a new `content` app, a large number of new Python package dependencies
  (all stemming from `wagtail`), and some updates to the site navigation.
- **Pegasus now generates a sitemap for you.** [Documentation](/page-metadata.html#sitemaps).
- **Responsive HTML email templates were added for the default email confirmation and password reset emails.**
- Add `get_protocol` helper function to return the string "http" or "https" depending on `settings.USE_HTTPS_IN_ABSOLUTE_URLS`

### Changed

- Social sign ups will now automatically create a default team if not there.
- The `login_and_team_required` decorator now falls back to the default team from the session/user if not available from URL
  (thanks Van for the suggestion)
- Add link to the profile page to set a password if the user only has a social account setup.
- Improve the default UI for setting a password.
- Split the top navigation for the app template into its own template (`top_nav_app.html`). (Material Theme Only)
- Show top navigation menu even if user is logged in (but hide the sign up / sign in buttons) (Material Theme Only)
- Increase top margin on 404 and 500 error pages (Bootstrap only)
- Reduced whitespace in some html templates.
- Changed reference of `djstripe_sync_plans_from_stripe` to `bootstrap_subscriptions` in subscription error message. 
- Removed "[project name]" prefix from signup / password reset emails
- Refactor: use `get_protocol` and f-strings in `absolute_url` helper function
- Removed "More coming soon" text from the Pegasus examples homepage.

### Fixed

- Fixed `pyparsing` version conflict between regular and production requirements (Google Cloud builds only)
- Fixed subscription price displays when using graduated pricing. (Thanks Lachlan for reporting)
- Remove Heroku Dockerfile from build if not using it

*June 21, 2022*

## Version 2022.5

This release improves many of the details of the Stripe subscriptions integration.
To streamline these changes, Stripe Elements support has been removed, and subscriptions
will be Checkout-based moving forwards.

The release also many small fixes and improvements.

### Added

- **Re-use the Stripe `Customer` object when a User has multiple subscriptions / payments (by saving it on the User model).
  This prevents users from having to re-enter their cards multiple times.**
- **Added an `active_subscription_required` decorator for easier subscription feature-gating. [Docs](/subscriptions.html#using-the-active-subscription-required-decorator)**
- **Added a periodic task to sync subscriptions with Stripe every day when per-seat billing is enabled.**
- Added a slug field to `ProductMetadata` to be able to uniquely refer to products in code.
- Added `sync_subscription_model_with_stripe` helper function (logic was previously only in `sync_subscriptions` management command)
- Stripe subscriptions are now created with a human-readable description with the team/user associated with the subscription. 
- Show expiration date on subscription details when auto-renewal has been turned off
- Docker projects now include a `.env.dev.example` file to use as a reference for sharing across team members
  (the `.env.dev` file that ships with Pegasus was .gitignored)
- Added Docker-based instructions for running tests while watching for changes.
- Add `lang="en"` to base template for improved accessibility

### Fixed

- Fixed bootstrap button CSS classes in a few templates to use the `pg-` class styles instead of the legacy Bulma version. (thanks Lisa for reporting)
- Removed unreachable template code in `view_subscription.html`.
- Removed accidental references to teams from user-based subscription views.
- Fixed subscription payment display amount when subscription quantity is > 1
- `URLField` now renders with same styles as other inputs
- Don't sync Stripe subscriptions if they don't need to be changed (per-seat billing only)
- Don't show hidden labels for inputs in `render_field`/`render_form` tags
- Fixed typo in `update_billing_date_on_membership_deletion` doc string

### Changed

- **Upgraded Python packages to their latest versions.**
- **Heroku-based docker deployments now use a `buster` base image instead of `alpine`. This helps to simplify build issues when adding certain packages / libraries.**
- Simplified team-based subscription views to no longer have redundant functions
- Improved layout of `socialaccount/signup.html` (shown when someone signs up with a social account but the email is already in use). (thanks Lisa for reporting)
- Updated migration file in api app to match latest expected column sizes
- `upgrade_subscription_checkout.html` template has been combined with `upgrade_subscription.html`
- Upgrade htmx version from 1.5 to 1.7
- Improved error message when running `bootstrap_subscriptions` with bad Stripe credentials
- Extracted `get_price_display_with_currency` helper function from `get_friendly_currency_amount`
- Increased size of titles in socialaccount templates.
- Reduced whitespace in HTML templates that were modified from 4-spaces to 2-spaces.
- Added `celerybeat-schedule` to `.gitignore`

### Removed

- **The deprecated Stripe elements support for subscriptions has been removed. Subscriptions require using Checkout moving forwards.**

*May 30, 2022*

## Version 2022.4.2

This is a hotfix release, which adds a missing closing `</div>` tag to the signup template on Tailwind
builds when not using teams.

*April 27, 2022*

## Version 2022.4.1

This is a hotfix release, which fixes the `make init` command to start containers in the background
so that migrations will properly run.

*April 26, 2022*

## Version 2022.4

Version 2022.4 is a mix of new features and maintenance improvements.
Major updates include new "Login with Twitter" functionality, allowing users to manage their own social accounts,
and some Docker development improvements.

### Added

- **Added "login with Twitter" as a first-class supported authentication option.**
- **Users can now see and manage (connect and disconnect) social accounts (Google and Twitter) from their profile pages.**
- Added new Docker makefile targets, including `make start-bg` to run docker containers in the background (previous default behavior), 
  and `make restart` to restart docker containers
- Added view to simulate errors (by raising an Exception). This is helpful for testing Sentry integrations.
- Added ability to override page titles in templates using the `{% page_title %}` block.
- Added better page titles to several app pages.
- Continued adding translation markup in user-facing text that was modified (this will be an incremental effort towards full-site translation)

### Changed

- **Upgraded version of node that runs in the Docker container from 14 to 16.**
- **Sentry is now a first-class build option enabled in the UI. If enabled, it will be automatically included in requirements and configured in `settings.py`.**
- README file now includes Docker-specific instructions for everything that was included (Docker builds only)
- Made more settings variables configured via environment variables.
- Docker `make start` command now runs in the foreground instead of in the background.
- Merged `meta.html` partial template into `base.html` to enable overridding blocks inside of it.
- The `socialicon` CSS class is now shared in all CSS frameworks and also sets the max width/height of the icon.
- Updated outdated Django docs references to Django 3.2
- Extracted social login buttons to their own templates which are shared on signup and login pages.
- Upgraded Django to 3.2.13

### Fixed

- Filter out inactive (archived) Stripe prices/plans from the list of plans

### Removed

- Removed `dev-requirements` files. Pip tools should now be installed with `pip install --upgrade pip-tools`.
- No longer call `npm install` in the `Dockerfile`, since it wasn't working as expected.
- Cleaned up unnecessary `type="application/javascript"` markup from many `<script>` tags.

### Documentation

- Added [page metadata documentation](/page-metadata/) for working with project / page metadata (e.g. page titles).
- Updated the [upgrading documentation](/upgrading/) to recommend a branch-based approach.
- Updated the [Sentry documentation](configuration.html#sentry) with the new setup and testing instructions.

*April 22, 2022*

## Version 2022.3

The main feature of this release is [Github Actions](https://github.com/features/actions) support.
Pegasus projects can now (optionally) run back end tests and build the front end automatically on Github.

This should work automatically for all projects out-of-the-box.
See [the documentation](/github-actions/) for more details.

### Added

- [Github Actions support](/github-actions/).

### Changed

- Postgres database settings can now be configured by environment variables directly in `settings.py`
- Upgrade default Python version to 3.9 (including in all Docker files)
- Development Dockerfile now installs all npm packages when being built (thanks Brian for suggesting)
- Upgraded ~all Python packages to their latest versions, apart from leaving Django on 3.2 LTS

### Fixed

- More consistent support for using Celery with all versions of Heroku and Redis secure/insecure URLs (thanks Brett for reporting).


This release also introduces a new versioning convention, where future releases will be named: `yyyy.mm` or `yyyy.mm.n` in the case
of multiple releases in the same month.
This was done to make it clearer to customers when they last upgraded, because the previous release numbers were not meaningful,
and because the author had developed decision fatigue about when to declare a "1.0" release.

*March 28, 2022*

## Version 0.22.1

This is a hotfix release that fixes a crashing bug with Heroku deployments that don't use Redis (thanks James for reporting).

*March 15, 2022*

## Version 0.22

The main feature of this release is a new front-end theme for Pegasus based off of the 
[Material Kit](https://www.creative-tim.com/product/material-kit) and [Material Dashboard](https://www.creative-tim.com/product/material-dashboard)
products from Creative Tim.

You can get a sense of the theme in this 3-minute video:

<div style="position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden; max-width: 100%; height: auto; margin-bottom: 1em;">
    <iframe src="https://www.youtube.com/embed/WwcowKrwCl0" frameborder="0" allowfullscreen style="position: absolute; top: 0; left: 0; width: 100%; height: 100%;"></iframe>
</div>

This release also adds support for Django 4.0. See note below for details.

### Added

- New Bootstrap Material theme based on Creative Tim's Material Kit / Material Dashboard
- Added breadcrumbs to the Employee demo app pages
- Unofficial support for Django 4.0 (see note below).

### Package upgrades

- `dj-stripe` to 2.6.1

### Changed

- All tables are now responsive (scrollable) on Bootstrap builds
- Add vertical gutters to columns on Bootstrap builds
- Better spacing of grid layouts in the examples
- Default pegasus form input styling is now handled with `pg-input-group` class
- Updated `djstripe_settings` import to be compatible with new version
- Added top navigation bar to various account templates (password reset, password change, etc.). (thanks James for suggesting!)
- Use `render_field` in more account templates
- Heroku runtime now uses Python 3.8.12 (thanks Allan for reporting!)

### Fixed

- Fixed widget class overrides from not being populated for some widgets. 
  This also fixes "select all" functionality in the Django admin for Bootstrap builds. (Thanks Will for reporting!)
- Fixed connection strings for Redis on Heroku if you were not on the free tier, by adding `?ssl_cert_reqs=none` to Redis connection strings. (Thanks Reid for reporting!)
- Fixed 500 error in payments example if the "Pay" button was clicked before the JavaScript on the page fully loads.
- Replaced references to deprecated `ugettext` translation functions with `gettext` versions.
- Removed unused `manage/<path:path>` url route in HTMX-based teams builds (thanks Peter for reporting!)

### Django 4.0 support

This version unofficially supports Django 4.0, however still ships with 3.2.12.

This is due to one issue with the `dj-stripe` dependency [generating migration files](https://github.com/dj-stripe/dj-stripe/pull/1607)
that may cause migration problems across multiple Pegasus environments (e.g. dev and prod, or two developer machines).
This is a very similar issue to the one described in the [0.14.2 release notes](#0-14-2).

If you wish to use Django 4.0, update the pinned version to 4.0.2 (or latest) in your `requirements.txt` file after downloading Pegasus,
and all should work.

*Feb 28, 2022*


## Version 0.21.1

This maintenance release simplifies how users are accessed via API key authentication,
to make it easier to transition views.
It mostly undoes a small handful of changes from the 0.21 release.

### Changed

- `HasUserAPIKey` permission class now populates `request.user` with the associated user, if a valid `UserAPIKey` is present.
- API Views and Viewsets now continue to access users via `request.user` instead of the `get_user()` helper functions
- `TeamSerializer` now accesses the user from the request instead of the explicit `["user"]` context. 

### Removed

- Now-redundant `UserAPIKeyMixin` class

*Feb 4, 2022*

## Version 0.21

This release has one major feature: API Keys.
You can get an overview in this 2-minute video or check out [the documentation](/apis/) for details.

<div style="position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden; max-width: 100%; height: auto; margin-bottom: 1em;">
    <iframe src="https://www.youtube.com/embed/ivL10Y5P5L0" frameborder="0" allowfullscreen style="position: absolute; top: 0; left: 0; width: 100%; height: 100%;"></iframe>
</div>

There are also a number of smaller fixes and upgrades.

### Added

- API Key support! See the new [API key documentation](/apis/) for details.
- Example `DJSTRIPE_WEBHOOK_SECRET` to `.env.dev` file for local Docker deployments

### Package upgrades

- `django` to 3.2.11
- `psycopg2-binary` to 2.9.3 (fixes installation issues on latest macOS processors) (thanks Eric for reporting!)
- `djangorestframework` to 3.13.1

### Changed

- Updated how some type hints are done to reduce potential for circular imports caused by typing 
- Updated headings of account pages to be more consistent with rest of site

### Fixed

- Added missing closing `</section>` tag to `templates/accounts/password_change.html` (thanks Peter for reporting!)
- Removed redundant tailwind imports from `assets/styles/pegasus/tailwind.css` which were causing
  duplicate CSS definitions in the generated `site-tailwind.css` file (thanks Anna for reporting!)
- Fixed bug that triggered an error when subscribing new users to a Mailchimp email list
- The Docker development setup now works on the latest macOS processors

### Removed

- Building for multiple CSS frameworks has been removed. 
  To try different CSS frameworks you will need to edit your project and download each codebase separately. 

### Deprecated

**The following features will likely be removed in an upcoming release. Let me know if you would miss these!**

- **Stripe Elements support is deprecated and will be removed in favor of Stripe Checkout.**

*Feb 2, 2022*

## Version 0.20

This release involves a substantial update to Pegasus's subscriptions functionality.
The biggest new feature is that per-seat (also known as per-unit) billing is now supported (for Stripe Checkout only).
Team-based installments can now create and manage subscriptions based on the number of users in the team,
with built-in infrastructure to track changes and keep them in sync with Stripe.

You can find a 6-minute overview of the feature in this video:

<div style="position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden; max-width: 100%; height: auto; margin-bottom: 1em;">
    <iframe src="https://www.youtube.com/embed/v_ayMEj924w" frameborder="0" allowfullscreen style="position: absolute; top: 0; left: 0; width: 100%; height: 100%;"></iframe>
</div>

The implementation is customizable and can also be used by non-team builds with some additional configuration. 
See [the updated Subscription documentation](https://docs.saaspegasus.com/subscriptions.html#per-unit-per-seat-billing)
for an overview of this functionality, and bear in mind that it's a little complicated with several moving parts!

There were many code changes to support this work-as well as future planned improvements to the subscriptions module.
Additionally there were a number of smaller, unrelated updates and fixes.

You can read the complete notes below. The format for this release (and likely future releases) was borrowed from [keep a changelog](https://keepachangelog.com/).
The most significant changes are in **bold**.
Also please note the announcement of a few features that will be removed soon!


### Added

- **Support for per-unit / per-seat billing. See [the docs on using this](https://docs.saaspegasus.com/subscriptions.html#per-unit-per-seat-billing)**
- `SubscriptionSerializer` class for including Subscription objects in APIs
- Management command and logic to sync subscriptions when team membership changes (useful for per-seat billing workflows)
- APIs for creating Stripe checkout and portal sessions and returning the URL to be used by JavaScript front ends
- Stripe webhook support for provisioning a subscription after a sucessful checkout session
- Allow specifying promotion codes when using Stripe Checkout
- Add ability to override display prices for individual billing plans
- Celery on Digital Ocean app platform now supports multiple processes (using gevent)
- More type hints in various places

### Package upgrades

- Upgrade Django version to 3.2.10 LTS
- Upgrade dj-stripe to 2.5.1

### Changed

- **Changed URLs of Stripe integration API calls to be more consistent**
- **Removed `subscriptions.views.helpers` and merged into `subscriptions.helpers`**
- **Moved `SubscriptionModelMixin` from `apps.subscriptions.helpers` to `apps.subscriptions.models.SubscriptionModelBase` and made it an abstract model**
- Add `created_at` to `Membership` admin
- Add `subscription` and `has_active_subscription` to `Team` serializer
- Moved `ProductWithMetadataAPI` to `api_views` module
- Access `stripe` through `get_stripe_module` helper function
- Extract Stripe checkout and portal session creation to helper functions
- Extract subscription provisioning to helper function
- Add several properties and methods to `SubscriptionModelBase` to help track and use per-seat billing
- Removed redundant extra functions in `apps.subscriptions.checkout_views`
- Prices on subscriptions are no longer centered (it looked silly with only 1 or 2 plans)
- **Prices on subscription page for annual plans now display total amount with billing interval (e.g. $300/year) instead of monthly price ($25/month)**
- Changed `get_friendly_currency_amount` helper function to take a `Plan` object instead of an amount and a currency.


### Fixed

- Team and invitation list APIs now require authentication (results in a 403 instead of 500 error when accessed anonymously)
- Archived products are not included in the default product list generated by `bootstrap_subscriptions`
- Automatically select the first plan if none is configured as the default
- Don't crash displaying subscription plan prices for subscriptions with tiered pricing (instead displays "Unknown"
- Removed some subscriptions templates when building without subscriptions for certain CSS frameworks.
- [tailwind-only] Removed reference to team name from logged-in homepage when not building with teams

*Dec 23, 2021*

### Deprecated

**The following features will likely be removed in an upcoming release. Let me know if you would miss these!**

- **Building for multiple CSS frameworks is deprecated and will be removed in the next release.**
- **Stripe Elements support is deprecated and will be removed in favor of Stripe Checkout the next release.**


## Version 0.19

This release has two major updates - improved team member management and improved Heroku deployment support.

**Improved Team member management**

Most of the changes below are summarized in this video:

<div style="position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden; max-width: 100%; height: auto; margin-bottom: 1em;">
    <iframe src="https://www.youtube.com/embed/KuRg48t_D2s" frameborder="0" allowfullscreen style="position: absolute; top: 0; left: 0; width: 100%; height: 100%;"></iframe>
</div>

From a user's perspective there are three main updates:

- Admins can now change any team members' role from the new "member details" page.
- Admins can also remove anyone from a team.
- All team members (including admins) can leave teams they belong to.

The following supporting changes were also made:

- Added a comprehensive test suite supporting the above workflows.
- Fixed a bug that prevented re-inviting the same email address to a team even if the previous invitation had already
  been accepted. Duplicate pending invitations are still prevented. 
  This change also removed the DB constraint requiring team and email address to be unique.
- Fixed bug where "resend invitation" and "cancel invitation" buttons were incorrectly showing up for non-admins.
- Added helper `is_admin` method to team `Membership` objects.
- Added a `CustomUserSerializer` to pass user information to the front end.
- Added the `user_id` field to `MembershipSerializer`
- Add the site name to body of team email invitations.
- Add `__str__` function to team `Membership` objects.
- Introduce `TeamPermissionError` exception class and start using it for permission-related failures.
- Added better display of front-end validation issues when issuing invitations fails.
- Various bits of formatting and cleanup in teams React code.

**Heroku deployment improvements**

A few updates were made to make deployments to Heroku containers faster and more comprehensive:

- Bump `cryptography` to `3.4.8` and remove no-longer-necessary dependencies from production dockerfiles.
  This results in vastly improved build times for Heroku docker deployments.
- Build the front end files as part of Heroku container deployments
- Run database migrations by default on Heroku container deployments
- Set various production-recommended settings in `settings_heroku.py`. 
  For more details see the new [production checklist](/deployment/production-checklist) in the documentation.

**Other minor updates**

- Added fall back to `username` on User display functions if the user does not have an associated email address.
- Extracted logic for opening modals to a shared JavaScript function (Bulma builds only).
- Remove commented out / unused code from `templates/web/components/top_nav.html` and modal dialogs. (Tailwind builds only).
- Bump `django` to `3.2.8`
- Add `pg-text-muted` helper CSS class
- Add some more type hints to URL helpers
- Extracted React form validation logic to `ValidationErrors` component.

**Upgrade Notes**

If you are upgrading from a previous version you should *not* merge any changes in the teams migration files,
and run `./manage.py makemigrations` and `./manage.py migrate` to remove the DB constraint on team invitations
after upgrading.

*Oct 27, 2021*

## Version 0.18.1

This hotfix release fixes a bug affecting React builds that caused the sign up page to be broken.

*Oct 3, 2021*

## Version 0.18

This release adds a new Django + [HTMX](https://htmx.org/) implementation of Pegasus's team management functionality.
Now, you can choose whether to use React or HTMX for your teams UI.

This choice is configured by the new "Front-End Framework" setting in your project's Pegasus settings.
The setting currently only impacts the Teams UI (and has no impact on Pegasus builds that don't use teams),
but will be extended in future releases.

The HTMX teams UI also has a few user-facing improvements:

- You can now choose a role when inviting new team members.
- Team deletion has moved to the team details page, and now has a confirmation dialog.
- Editing the team name and ID now require the team admin role.

Additionally, there are bug fixes and code-related improvements throughout the teams functionality:

- Fixed "sign out" link on invitation acceptance page.
- Improved workflow when attempting to accept an invitation to a team you already belong to.
- Broke the teams `views.py` file into multiple Python modules.
- Set `team_membership` on `request` object in team decorator functions, so it can be easily accessed in views/templates.
- Cleaned up bits of HTML in existing teams React implementation.
- Various doc strings, type hints, and formatting cleanup throughout code.
- Combined `accept_invitation` and `accept_invitation_confirm` into a single view.
- Remove a few teams-related templates when not building with teams enabled.

Finally, there were minor updates to other functionality:

- Bootstrap JavaScript is now built and used from the local install instead of using a pinned CDN. (Bootstrap builds only)
- Extract Mailchimp mailing list logic to its own module and add [mailing list documentation](/configuration.html#mailing-list).
- Fix quirks in HTMX example when back-end validation fails.
- Minor cleanup of HTMX example code.
- Rename `payments` and `tasks` view modules to `payments_views` and `tasks_views`.
- Upgrade Django to `3.2.7`.
- Changed default value of `USE_HTTPS_IN_ABSOLUTE_URLS` setting to `False`

**Potentially breaking changes**

The change of the default value of `USE_HTTPS_IN_ABSOLUTE_URLS` could impact production environments
that have not overridden this setting. In particular, it would cause Stripe checkout / portal callbacks
and invitation emails to have `http` links instead of `https` links.

To fix this, add `USE_HTTPS_IN_ABSOLUTE_URLS = True` to all production settings files / environments.

*Sep 28, 2021*

## Version 0.17.1

This fixes a crashing issue when resending team invitations.

*Sep 8, 2021*

## Version 0.17

This release adds an HTMX demo, a Teams example app, and more.

This video provides a 4-minute overview of the key features, or read on for details.

<div style="position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden; max-width: 100%; height: auto; margin-bottom: 1em;">
    <iframe src="https://www.youtube.com/embed/Wpgmvc273Kw" frameborder="0" allowfullscreen style="position: absolute; top: 0; left: 0; width: 100%; height: 100%;"></iframe>
</div>

**The HTMX object demo**

The object demo application now has an [htmx](https://htmx.org/) implementation.
HTMX is a library that allows you to build rich, single-page, interactive experiences by adding HTML attributes.

The provided demo application is a single page app with no native JavaScript, backed by Django forms.

**The Teams Example app**

This release also adds a Teams example app, which has Create, Read, Update, Delete (CRUD) views for an example
model that is part of a team (called a "Player").

As part of this work, a new base model class, `BaseTeamModel` was added which can be extended
to create models that belong to a Team.

Additionally, two view mixin classes, `LoginAndTeamRequiredMixin` and `TeamAdminRequiredMixin` were added,
which can be used to easily create class-based model views on Team models.
A test suite for these mixins was also added.

More details can be found in the [Teams documentation](/teams/).

*A big thanks to Peter Cherna, who's [Pegasus example apps](https://github.com/pcherna/pegasus-example-apps/)
was a big inspiration for this example.*

**Other changes**

- Add generic breadcrumbs CSS classes compatible with Bulma, Bootstrap and Tailwind (used by teams example)
- Default all Pegasus apps to using `BigAutoField` instead of `AutoField` (see upgrade notes)
- Use f-strings instead of string templates in management commands
- Reduce indentation level in some html template files

**Upgrade notes**

The default ID for all Pegasus models was changed from `AutoField` to `BigAutoField`.
If you are upgrading a project using `AutoField` you should *not* merge any changes to initial migration
files in the affected apps. Then you can either:

1) To change your apps from `AutoField` to `BigAutoField`, run `./manage.py makemigrations` and `./manage.py migrate` 
1) To keep using `AutoField`, change the setting back to `AutoField` in `apps.py` for all Pegasus apps. 

*Aug 24, 2021*

## Version 0.16.2

This release upgrades `django-hijack` from 2.3 to 3.0 and adds hijack links to the admin site.

*Aug 11, 2021*

## Version 0.16.1

This minor release fixes styling issues with Tailwind builds that were accidentally introduced in 0.16.

*Aug 9, 2021*

## Version 0.16

This is a grab-bag release of mostly cleanup work and bug fixes.

***Please review the breaking changes closely if you are upgrading an existing project!
Not doing so could lead to unexpected behavior and demo data loss!***

**Potentially breaking changes:**

- Changed the `login_and_team_required` and `team_admin_required` view decorators to no longer allow superusers
  to access the views. It is recommended to use the new user impersonation feature to allow superusers
  to access teams they don't belong to.
- Removed `user_can_access_team` and `user_can_administer_team` helper functions which provided team
  access to superusers.
- Removed `attrs` library dependency and switched its usages to `dataclasses`.
- The `Employee` example data model has moved, and running `migrate` will drop all of your demo `Employee` data.

**New: Improved Django form support:**

- Added a Django forms implementation of the employee / object demo
- Rename `bootstrap_form_fields` and `bulma_form_fields` templatetags to `form_fields` if not using 
  multiple-css-framework support  
- Add `render_form_fields` helper function to render an entire form. (Note: this currently only supports text-style, 
  checkbox, and select input types.)
- Add `render_checkbox_input` to bootstrap form rendering tags
- Added a basic non-model form example

**Fixes:**

- Fix cryptography build issue affecting Heroku docker deployments
- Improve textarea styling on Bootstrap forms
- Better default styling of checkboxes in Bootstrap forms.
- Don't require that `team_slug` be the first argument to a url to use the team-based decorator functions.

**Cleanup:**

- Upgrade Django to 3.2.6
- Remove redundant `null=True` from user avatars
- Added more type hints
- Split object / employee example out into its own app and template folder.
- Added help text to employee example model fields
- Changed some navigation and text around the employee app example
- Remove inline style declarations from a few examples
- Added some instructions about initializing your database to the README
- Update migrations to be compatible with latest Django / Pegasus code 

**Upgrade notes:**

If you receive a warning like this when generating Database migrations:

*You are trying to change the nullable field 'avatar' on customuser to non-nullable without a default;
we can't do that (the database needs something to populate existing rows).*

It is safe to choose option 2, "Ignore for now...".
There won't be any NULL values in the database, since Django automatically populates `FileField`s with an
empty string.

*Aug 6, 2021*

## Version 0.15

Version 0.15 is a major release with two big features: Stripe Checkout and User Impersonation.

For 3-minute summary you can watch the video below, or read on for details.

<div style="position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden; max-width: 100%; height: auto; margin-bottom: 1em;">
    <iframe src="https://www.youtube.com/embed/4dQ63KCd-E8" frameborder="0" allowfullscreen style="position: absolute; top: 0; left: 0; width: 100%; height: 100%;"></iframe>
</div>

**Stripe Checkout Support**

Now you can use [Stripe Checkout](https://stripe.com/payments/checkout) for payments instead of - or in addition to -
the Stripe Elements-based embedded UI. Stripe Checkout support has been added to the subscriptions page, as well
as the payment example. You can build new Pegasus projects with Checkout, Elements, or toggle between them
via a `settings.py` setting.

**User Impersonation**

This release also adds optional user impersonation functionality for superusers, allowing admins and support staff
to "login as" another user with a few clicks. This is a great tool for troubleshooting user-specific issues.
This feature depends on [django-hijack](https://github.com/django-hijack/django-hijack) and can be disabled
at build time.

**Other updates**

- Make http/https in absolute URLs (e.g. email invitations) configurable via the `USE_HTTPS_IN_ABSOLUTE_URLS` setting
- Start adding type hints to the codebase, primarily in helper functions. More coming soon!
- Added more tests for teams helper functions 
- Split out `subscription/views.py` into multiple files to make it clearer which views affect which functionality.
- Break `upgrade_subscription.html` into multiple files to support elements and checkout-based flows.
- Improve default string representation of user model
- Improve the default styling of select widgets (Bootstrap builds only)
- Clean up doc strings and code for team API permission helpers (teams builds only)
- Rename `teams.util` to `teams.helpers` to be consistent with other apps (teams builds only)
- Fix issue with `apps/subscriptions/__init__.py` not being created for certain configurations
- Various minor cleanups of whitespace, url ordering, etc.

*July 22, 2021*

## Version 0.14.5

Another grab-bag of minor fixes and improvements.

- Reduce scope of hiding file inputs to just the profile upload form
- Add `NoNewUsersAccountAdapter` to allow disabling public sign ups for an app
- Extract Bulma and Bootstrap form rendering to helper template tags
- Fix subscription details start date display (subscriptions builds only)
- Explicitly set `DEFAULT_AUTO_FIELD` to `AutoField` in settings to remove runtime warnings.
- Minor README improvements
- Upgrade Django to 3.2.5

Changes only affecting builds with teams enabled:

- Fix error on sign up if email passes front-end validation but fails back-end validation
- Change `/teams/` URL to go to the teams management page instead of API docs
- Improve styling of teams page when no teams exist on Bootstrap
- Fix React warning in teams JavaScript

*July 15, 2021*

## Version 0.14.4

A very minor release:

- Don't include `djstripe` and associated settings in `INSTALLED_APPS` if not using subscriptions

*June 30, 2021*

## Version 0.14.3

A very minor release:

- Update Django to 3.2.4
- Remove `DEFAULT_AUTO_FIELD` declaration from settings. 
  This was causing issues with allauth migrations being generated, as a result of [this issue](https://github.com/pennersr/django-allauth/pull/2829).
  Until more libraries have worked around these migration issues, Pegasus will just ship with the default setting.

*June 17, 2021*

## Version 0.14.2

This release upgrades `dj-stripe` to version `2.4.4` which should fix cross-environment migration issues.

**Upgrade notes:**

Projects using a "clean" 0.14 or 0.14.1 release may have been affected by [this issue in dj-stripe](https://github.com/dj-stripe/dj-stripe/issues/1344)
where running `makemigrations` automatically generated a migration file inside the `dj-stripe` library code.
Local app migrations would then add a dependency to this migration, which would not be available on other environments.

This often manifested as an error along the lines of the following when setting up a second environment:

```
django.db.migrations.exceptions.NodeNotFoundError: 
  Migration .0003_auto_20210524_1532 dependencies reference nonexistent parent node ('djstripe', '0008_auto_20210521_2314')`
```

To fix this there are a few steps:

1. You should explicitly uninstall and reinstall `dj-stripe` instead of just installing requirements to ensure the 
   generated migration file is removed. `pip uninstall dj-stripe && pip install dj-stripe==2.4.4`
2. You should rebuild your own app's migrations to no longer depend on the generated `dj-stripe` migration,
   by deleting the bad files and re-running `makemigrations` on a fresh environment.
3. You will need to "fake" migrations for any existing DBs, by manually deleting relevant rows from the `django_migrations`
   table and then running `./manage migrate <appname> --fake` for each affected app.
   
Please reach out for support on Slack if you run into any issues with this, and I'm happy to help!

*May 26, 2021*

## Version 0.14.1

This is a minor release including the latest Django security updates, the official Bootstrap 5.0 version,
and a few small features and fixes.

- Fix incorrectly generated `urls.py` if you built with teams and without subscriptions
- Add password confirmation field to sign up pages if configured by allauth
- Add `project_settings` example for passing project-level settings to templates via context processor
- Update `bootstrap` to `5.0.1`.
- Update `django` from `3.2` to `3.2.3`
- Remove unused `kombu` dependencies from requirements.

**Changes affecting [experimental features](/experimental) only:**

- Slightly improved styling of Tailwind sign up page

*May 19. 2021*

## Version 0.14

Version 0.14 is a major release with a focus on Teams and package upgrades.
It also upgrades Pegasus to Django's 3.2 LTS release.
Details are below:

**Teams Upgrade**

- Make team name optional on signup, and auto-generate a team if none specified.
- Update team-based URL-routing to be more consistent (put all team urls behind `/a/team-slug/`)
- Migrate teams React code to functional components / hooks
- Add URL-routing and back button support to teams page
- Restrict ability to edit/delete teams to team admins only
- Restrict ability to manage team invitations via API to team admins only
- Add manage/view team page to main team navigation
- Other minor teams JS cleanup (replace `var` with `const`, etc.)
- Delete `team_nav.html` when not using teams
- Improve loading screen for teams page and extract to shared component

**Package Upgrades**

- Upgrade all python packages to latest versions (including Django to 3.2 LTS)
- Updated settings file to the version that ships with Django 3.2
- Update formatting of requirements files to the latest version used by `pip-tools`
- Upgrade all JS packages to latest versions 
- Switch from deprecated node-sass to dart-sass

**Other updates**

- Set base URL in React object demo from Django
- Fix doc strings of npm-related `make` commands
- Extract logic for getting CoreAPI JavaScript client to a shared function
- Add support for celery workers to Digital Ocean deployments (and [updated docs](/deployment))
- Add `make dbshell` docker command to get a database shell
- Remove some tailwind files that were accidentally included in non-tailwind builds

**Changes affecting [experimental features](/experimental) only:**

- Improve Tailwind styling of examples, teams and other built-in pages
- Add `@tailwind/forms` plug in for improved form styling
- Split Pegasus Tailwind CSS classes into a new file to mirror Bootstrap/Bulma implementations
- Fix collectstatic errors when building with tailwind and whitenoise (affected Heroku and Digital Ocean deployments)
- Remove broken landing page and pricing page examples, and point people to Tailwind UI instead 

**Upgrade notes:**
- Django added the new [Default primary key field type](https://docs.djangoproject.com/en/3.2/ref/settings/#default-auto-field)
  setting. If you wish to use the default in 3.2 you will have to add DB migrations for it. Otherwise you
  can change the value of `DEFAULT_AUTO_FIELD` to `'django.db.models.AutoField'` in your `settings.py` file.
- Bootstrap users may need to run `npm update bootstrap` before building static assets to get all styling 
  of the examples working again.

*April 28, 2021*

## Version 0.13.2

This is a minor release primarily focusing on an improved Docker experience and
updates to the experimental TailwindCSS support:

- Update development `Dockerfile` from `alpine` to `buster` and install front-end dependencies.
- Add `Makefile` with self-documenting targets for various common operations ([see docs](/docker/))
- Update generated `README` with better Docker instructions
- Use double quotes for description and name settings to reduce issues with apostrophes

Changes affecting [experimental features](/experimental) only:

- Properly support for using PurgeCSS with Tailwind
- Include (purged) Tailwind bundle files with Pegasus

*March 10, 2021*

## Version 0.13.1

This is a minor maintenance release with a few small changes and fixes.

- Fix Docker image when deploying to Heroko containers
- Fix SSL mixed content issues on Heroku Docker builds
- Remove teams JavaScript files if not using teams.
- Add `body_wrapper` block for overriding the whole base template
- Label closing `endblock` tags in base template
- Add `.DS_Store` files to `.gitignore`
- Minor compatibility fixes to `teams` CSS.

*February 24, 2021*

## Version 0.13.0

This release adds support for the [Bootstrap CSS framework](https://getbootstrap.com/) and includes several changes
to how the CSS files are structured in Pegasus.
See the new [CSS documentation](/css/) for an overview of the new structure.

Major changes:

- Full support for Bootstrap CSS!
- Restructure CSS files, creating framework-level subfolders, splitting `site-base` and `site-<framework>` files out, 
  and making Sass imports implicit in the Bulma files.
- Examples CSS overhaul, converting styles to `pg-` classes and adding the `pegasus/<framework>.sass` files to
  style examples and JavaScript.

Minor changes:

- Improved UI of object demo tables on small screens.
- Remove unused "tagline" CSS styles and replace with the CSS frameworks' vertical centering utilities.
- Update progress bar styles to use CSS variables instead of Sass variables.
- Remove unused redundant "section" classes from a number of places.
- Split profile page into multiple HTML templates.
- Remove duplicate cookie-handling code JavaScript on profile page and use the same JS code used in other places.
- Remove unused `plan-price` and `plan-tagline` classes from subscription page.
- Fixed bad reference to `subsriptions.sass` when not building with subscriptions.
- Remove unused `has-vcentered-cells` class.
- Start splitting out teams JavaScript into multiple files.
- Remove redundant `custom-file-upload` CSS class.
- Add /404 and /500 endpoints for viewing custom 404 and 500 pages.
- Remove unnecessary raw string formatting from severeal `urls.py` files.
- Upgrade Django to `3.1.6`


*February 8, 2021*

## Version 0.12.1

This release continues the overhaul of the Pegasus CSS and templates to include experimental support for 
the Bootstrap CSS framework. Details on using Bootstrap [can be found here](/experimental).

Changes affecting everyone:

- Fix "view subscription" page start date display, and accommodate when start date is not set in Stripe.
- Default `ACCOUNT_LOGIN_ON_EMAIL_CONFIRMATION = True` to avoid double-login after email confirmation.
- Move navbar avatar styling to standalone `navbar.sass` file to be shared across CSS frameworks.
- Change navbar avatar class and CSS from `.avatar` to `.navbar-avatar`.
- Split subscription details on the "view subscription" page into its own component templates.
- Split hero section on the "upgrade subscription" page into its own component templates.
- In `subscriptions.sass`, use css variables instead of sass variables for colors.
- On subscription page, change "upgrade-features" selector from an ID to a class in HTML and CSS. 
- Remove unused styles from `subscriptions.sass`
- Clean up whitespace in a number of files


Changes affecting experimental features:

- Booststrap CSS build! [Details here](https://docs.saaspegasus.com/experimental#bootstrap-css-support).
- Building for multiple CSS frameworks now includes Bootstrap (along with Bulma and Tailwind)
- Building for Tailwind now overwrites Bulma templates instead of maintaining them in a separate
  directory.

*January 26, 2021*

## Version 0.12.0

This release is a set of changes laying the groundwork for future Pegasus improvements.

*Existing Pegasus users will need to upgrade their installer to use the latest version
(`pip install --upgrade pegasus-installer`)*

**Experimental support for Tailwind CSS**

More details can be found in the [Tailwind documentation](/experimental).

*Note: this feature is not yet fully supported and is in an experimental pre-release.*

**Simplified upgrade process**

Pegasus now saves your configuration when you install it, to simplify the upgrade process.
Instructions on upgrading can be found on the ["upgrading" page](/upgrading).

**Additional related changes and fixes**

- Remove `package-lock.json` from Pegasus for improved compatibility across NPM / OS versions. 
  It is recommended to run `npm install` and then check in the resulting `package-lock.json` file to source control.
  More details can be found in the [front end docs](/front-end). 
- Switch some styling from Sass variables to CSS variables (e.g. colors) for compatibility with multiple CSS frameworks
- Add a `site-base.sass` file for base Pegasus styles that aren't dependent on any CSS framework.
- Fix issue with `PlanSerializer` and `dj-stripe` version 2.4.1
- Remove no-longer-used `subscription_title.html` template.
- Remove no-longer-used tooltip styles
- Remove unused HTML from subscriptions upgrade page and split it out into partial templates
- Change a few default settings, e.g. project and author name.
- Upgrade django to 3.1.5.

*January 13, 2021*

## Version 0.11.3

This release overhauls the Payments example to use the Stripe `PaymentIntent` API 
and [support 3DS / SCA cards](https://stripe.com/docs/strong-customer-authentication).

Minor changes:

- Upgrade `dj-stripe` to `2.4.1`
- Pull `DJSTRIPE_WEBHOOK_SECRET` setting from environment variable if available 

If you are upgrading from a previous installation, you may need to change
the following two values in `settings.py`:

```python
DJSTRIPE_FOREIGN_KEY_TO_FIELD = 'id'  # change to 'djstripe_id' if not a new installation
DJSTRIPE_USE_NATIVE_JSONFIELD = True  # change to False if not a new installation
```

*December 17, 2020*

## Version 0.11.2

- Fixes to subscription workflow when using a trial period - automatically refresh page after
  card is accepted and support 3D-Secure validation for credit cards when using a trial. 

*December 9, 2020*

## Version 0.11.1

- Fix bug in development Dockerfile from a new OS dependency introduced by the `django-allauth` upgrade.

*December 7, 2020*

## Version 0.11.0

This release is a grab-bag of fixes and upgrades based on recent feedback.

- Force users to reconfirm their email when changing it and email verification is enabled
- Switch from `extract-text-webpack-plugin` to `mini-css-extract-plugin` for CSS handling in Webpack
- Rename `assets/index.js` to `assets/site.js` to support more styles in the future.
- Use randomly generated identicons for users instead of a default empty profile picture
- Move `.env.dev.example` to `.env.dev` and `.env.production.example` to `.env.production` so
  they don't have to be copied to get running (Docker builds only)
- Clean up payments UI a little
- Remove accidentally included `team_admin.html` template when not building with teams
- Remove accidentally included `Dockerfile.dev` template when not building with Docker
- Remove unused CSS classes from examples 
- Upgrade various NPM packages to latest versions
- Upgrade `django-allauth` to `0.44.0`
- Upgrade `django` to `3.1.4`

*December 2, 2020*

## Version 0.10.5

This release adds native support for deploying to Digital Ocean app platform.
See the [deployment guide](/deployment/#digital-ocean) for details.

Additional updates:

- Remove duplicate `ACCOUNT_EMAIL_VERIFICATION` declaration in `settings.py`
- Rename development `Dockerfile` to `Dockerfile.dev` for clarity and ease of deployment to other platforms
- Fix SSL / mixed content errors when deploying on Google Cloud Run

*Nov 17, 2020*

## Version 0.10.4

This release adds experimental native support for deploying to Google Cloud Run.
More details can be found in the [deployment guide](/deployment/#google-cloud).

Additional updates:

- Fix static file references to favicons.
- Upgrade `certifi` to `2020.11.8`
- Rename `heroku-requirements.txt` to `prod-requirements.txt` to be consistent across platforms (Heroku builds only)
- Switch production Docker images from Python-alpine to Python-slim (Docker builds only)

*Nov 10, 2020*

## Version 0.10.3

This release adds native support for deploying to Heroku using Docker containers.
More details can be found in the new [deployment guide](/deployment/).

Additional minor updates:

- Upgrade `urllib` to `1.25.11`
- Allow requirements to load from multiple sources when using Docker
- Add static directories and config files to `.gitignore`

*Nov 5, 2020*

## Version 0.10.2

This release adds support for using [Docker](https://www.docker.com/) in development.
More details can be found in the new [Docker documentation](/docker/).

*Oct 28, 2020*

## Version 0.10.1

This release adds [Heroku](https://www.heroku.com/) deployment support.
More details can be found in the new [deployment guide](/deployment/).

*Oct 26, 2020*

## Version 0.10.0

This is largely a maintenance release with mostly minor updates and fixes, 
but there are enough library upgrades that it warrants a version bump.

- Upgrade all Python packages including upgrading to Django 3.1.2
- Upgrade all npm packages
- Add a new API for products and metadata at `/subscriptions/api/active-products/`.
  Also includes adding some serialization classes and helper functions to subscriptions models.
  *Subscriptions only.* 
- Fix a bug where clicking on "Dashboard" didn't always take you to the right team.
  Also set team ID in the request session so it can be accessed across requests, and
  add `get_default_team` helper function to pull the last/current team from a request.
  *Teams only.*
- Fix default styling of textarea widgets in Django forms
- Use `const` and `let` in subscriptions page JavaScript
- Add `has-vcentered-cells` table formatting class to center table rows, and use
  in teams UI and object lifecyle demos
- Remove unnecessary `subscriptions.sass` file when Pegasus is built without subscriptions enabled

*Oct 14, 2020*

## Version 0.9.0

This release is a large overhaul of the React example that ships with Pegasus, including:

- Add url-routing support. Add/edit URLs now update and are linkable. 
  This also enables back button support
- Add validation feedback missing / bad data
- Switch all components from using classes to [hooks](https://reactjs.org/docs/hooks-intro.html)
- Split React components out into their own files
- Better loading UX

Other minor updates:

- Upgrade Django to 3.0.10 (3.1 support coming soon)
- Upgrade various JavaScript dependencies
- Generate random `SECRET_KEY` for each new installation
- Upgrade to Bulma 0.9.0
- Remove some spacing utility classes in favor of the ones that ship with bulma

Upgrading:

**Existing Pegasus users will need to upgrade the installer to run this.**
```
pip install --upgrade pegasus-installer>=0.0.2
```

*Sep 4, 2020*

## Version 0.8.3

- Fix default styling of number inputs in forms.
- Default `ACCOUNT_CONFIRM_EMAIL_ON_GET` to `True` if using email confirmation.
- Fix issue building front end on certain newer versions of nodejs/npm
- Upgrade `celery-progress` from `0.0.10` to `0.0.12`


*Aug 31, 2020*

## Version 0.8.2

This release fixes an edge case in the invitation accepting logic that didn't work if
a user left the page and came back later. 

*Aug 24, 2020*

## Version 0.8.1

A minor maintenance/bugfix release:

- Fixed a bug in the invitation workflow that prevented invitations from being accepted
  when creating accounts with social logins. Note that this requires changing the `ACCOUNT_ADAPTER`
  setting to  `'apps.teams.adapter.AcceptInvitationAdapter'`
- Make site branding in the navigation stay visible on mobile devices
- Make it more obvious that `settings.SECRET_KEY` should be overridden.
- Upgrade Django to 3.0.9

*Aug 21, 2020*

## Version 0.8.0

Stripe billing portal integration is here!

You can now use Stripe's new [customer portal](https://stripe.com/docs/billing/subscriptions/customer-portal)
to manage subscriptions in your app.

Supported operations and configurations include upgrading and downgrading subscriptions,
subscription cancellations (both immediately, and at period end), and subscription
renewals.

Additional documentation can be found in the [subscription docs](/subscriptions/).

Details and other changes:

- Added "manage billing" redirect to subscription pages if you have an active subscription,
  which goes to the Stripe customer portal
- Added webhook functionality to sync subscription changes and cancellations made via the Stripe portal
- Removed built-in "Cancel" option and supporting code.
- Upgraded `stripe` library to version `2.48.0`
- Removed the `.customer` field from `CustomUser` object. Customers are now always accessed via their
  associated subscriptions. 
- Removed a lot of no-longer-needed code from `SubscriptionModelMixin` related to accessing subscriptions
  from the `.customer` field. References to `.stripe_subscription` should be changed to simple `.subscription`
  on the `CustomUser` and `Team` objects.
- Added `avatar` to `CustomUser` admin.
- Fixed bug where "http://" was incorrectly assigned to the `Site.domain` object (fixes issues using `absolute_url`)

*July 16, 2020*

## Version 0.7.4

This is another minor release with mostly small fixes and updates to the front end.

- Added number formatting of Salary to object demo examples
- Fixed styling of number form inputs
- Removed unnecessary imports from `assets/index.js`
- Fixed incorrect distinction between `dependencies` and `devDependencies` in `package.json`
- Upgraded React to 16.13.1
- Moved React object lifecycle example to a `react` subfolder
- Started splitting up React object lifecycle demo into multiple files, and refactoring it to use hooks
- Renamed React object lifecycle example bundle file from `object-lifecycle-bundle.js` to `react-object-lifecycle-bundle.js`
- Removed unused "bower_components" exclusion in `webpack.config`

*July 13 2020*

## Version 0.7.3

- Upgraded to Django 3.0.7
- Fixed display of renewal details in subscription view to work with the latest Stripe Prices API
- Added ability to resend invitations from the team management page
- Added sort order to team member list (by email address)
- Cleaned up teams JavaSript (removed console logging statements, updated whitespace, removed commented code)

*June 30 2020*

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
