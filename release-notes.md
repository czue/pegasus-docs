Version History and Release Notes
=================================

Releases of [SaaS Pegasus: The Django SaaS Boilerplate](https://www.saaspegasus.com/) are documented here.

## Version 2025.1

This release includes mostly backend infrastructure changes to some Pegasus features to pave the way for a (future) plugin ecosystem.
This should make it easier to maintain Pegasus apps as well as possible (in the future) for other people to develop
apps that can seamlessly integrate into Pegasus.

### Separated the async group chat example from the async flag

Previously when you enabled async / websockets, you also got the group chat example application.
Now you can enable async features without this additional example app, and turn it on separately with a new configuration option.

### Added a flag to remove Celery (if possible)

Added a configuration option that will remove celery and all dependencies + configuration
***if no other parts of your application need it***.
It will also be removed from production deployment configurations.

Celery is still required (and will be automatically enabled) if you are using any of:

1. The Pegasus examples
2. Subscriptions with per-unit billing enabled
3. Any AI chat features

If you're not using these features and want to disable Celery you can do that from your project settings page.

### Organizational changes to apps for more consistency

The following changes don't have any new features or functionality, but they change small things about how the code is organized
for affected apps (AI chat, AI images, and async group chat).
It is hoped that these changes will make maintenance, upgrades, and future extensions to Pegasus easier.

- Moved app declarations for these apps to the end of `PROJECT_APPS` in `settings.py` 
- Moved url declarations for these apps to the end of `urls.py`.
- Moved settings and environment variables for these apps to be located together.
- Settings for these apps are now prefixed with `AI_CHAT_` or `AI_IMAGES_`, respectively.
  - **This also means that shared settings like `OPENAI_API_KEY` are now declared multiple times and need to be updated
    in multiple places.** See the "upgrading" section below on how to get around this duplication.
- Moved chat JavaScript setup to the end of `module.exports` in `webpack.config.js`.
- Depending on your configuration, the order of navigation tabs in the UI may change.
- Made minor tweaks to how channels urls are set up.
- Image logos used by the AI images app were moved to `/static/images/ai_images/`.
- The declaration for these apps has moved to a new "plugins" section of `pegasus-config.yml`.

### Other Changes

**Changed**

- **Upgraded default Python to Python 3.12.**
  - Bumped the Python version to 3.12 in CI, and dev/production Docker containers.
  - Also added [a `.python-version` file](https://docs.astral.sh/uv/concepts/python-versions/#python-version-files) for uv builds (set to 3.12)
- **Upgraded default Node to 22.**
  - Bumped the Node version to 22 in CI, and dev/production Docker containers.
- **Upgraded nearly all Python packages to their latest versions.**
  - Added a pin to `dj-stripe<2.9` because 2.9 is not yet supported.
- **Upgraded nearly all JavaScript packages to their latest versions.**
  - Tailwind v4 was not upgraded as it was just released and is not yet supported. 
- **Ruff and pre-commit will now sort imports by default.**
  - This also updates import sorting in a number of files.
- **Pre-commit now runs ruff with `--fix` enabled, which will automatically apply (but not stage) fixable errors.**
- Dependencies are now sorted in `pyproject.toml` (uv builds) and `requirements.in` (pip-tools builds)
- Added email address to admin search for team memberships and invitations. Thanks EJ for the suggestion!
- Made the "timezone" field editable in the user admin. Thanks Peter for the suggestion!
- Changed active tab variable for ai image app from "ai_images" to "ai-images" to match convention of other apps.
- Added a link from the user profile to manage email addresses if the user has more than one email registered.
  (Thanks Simon for the suggestion!)
- Make it so that `./manage.py` commands default to `uv run` if you build with uv enabled.
- The `chat_tags` template tag library was moved to the `web` app and renamed to `markdown_tags`,
  making it easier to use outside the chat application.

**Fixed**

- **Fixed an issue that caused Render deployments to fail when using uv.** (Thanks Jacob for reporting and helping fix!)
- Add `psycopg2-binary` to production requirements if using sqlite, since it still required for production deployments.
  (Thanks Randall for reporting!)
- Updated invitations to always store email addresses in lowercase to be consistent with account emails.
  Also fixed comparisons between invitations and sign up emails to be case-insensitive. (Thanks EJ for reporting and the fix!)
- Renamed `tailwind.config.js` to `tailwind.config.cjs` which prevents build failures on Node 22.
- Stopped including `pip-tools` in `dev-requirements` when using `uv`.

**Removed**

- Removed no-longer-used `payments.js` and `stripe.sass` files.

### Upgrading

**Python / Node updates**

You may need to manually modify your dev/production environment to upgrade to Python 3.12 and Node 22.
If you're using Docker, this should happen automatically by following the [upgrade process](./upgrading.md).

Pegasus apps should still run on Python 3.11 / Node 20, but will no longer be extensively tested on those versions
moving forwards.

**Settings Changes**

Some settings around AI API keys have been renamed and will need to be updated in your `settings.py` and `.env` files.
If you are using AI chat and AI images with OpenAI, the easiest way to use a shared API key is to add the following
to your `.env` / environment variables:

```
OPENAI_API_KEY="sk-***"
```

And then modify your settings variables to read from that value:

```python
# add an OPENAI_API_KEY setting, in case it was referenced elsewhere in your code
OPENAI_API_KEY = env("OPENAI_API_KEY", default="")
# modify the image/chat settings to use the same openai key instead of reading from new environment variables
AI_IMAGES_OPENAI_API_KEY = OPENAI_API_KEY
AI_CHAT_OPENAI_API_KEY = OPENAI_API_KEY
```

*Jan 27, 2025*

## Version 2024.12.1

This is a minor hotfix release for 2024.12

- **Fixed a bug where the delete workflow was broken for apps created by the Pegasus CLI on non-Tailwind builds.** 
  This happened becasue the "css_framework" cli option was accidentally missing from `pegasus-config.yml`.
  Thanks Robert for reporting!
- Updated the README instructions for setting up pre-commit hooks when using uv.

*Jan 13, 2025*

## Version 2024.12

This release adds first-class support for using uv as a complete replacement for development and production workflows
(see below), and has a handful of fixes/changes.

### UV support!

This release adds full support for [uv](https://docs.astral.sh/uv/) as a replacement package manager for your project.
You can use uv by selecting the new "uv" as your "Python package manager" on your project settings page.

When you select uv the following changes will be made:

- All requirements.in / requirements.txt files are removed.
- Your project requirements will now be listed in your `pyproject.toml` file.
  - Development and production dependencies will be listed under separate dependency-groups. 
- Your pinned project requirements will be listed in a new `uv.lock` file.
- Docker containers (in development and production) will use `uv` to set up and manage the Python environment.
- A `make uv` target will be added to Docker builds to run `uv` commands in your container.

The main benefits of using uv are:

- Speed. It is just way, way faster to anything related to package management.
- Easier to setup and install Python.
- Lock files (pinned versions) are consistent across any platform.
- More tooling.
- Speed. (It's so fast we put it twice.)

There will be a longer write up about uv released very soon, but in the meantime you can review the updated
[python documentation](./python/setup.md) and new [uv documentation](./python/uv.md).

The rest of the docs have been updated to accommodate uv, 
though it's possible there are some places that were missed.
If you spot any issues in the docs, get in touch!

### Other fixes

- **Upgraded the pegasus cli to fix an issue where the generated list views were not properly scoped to the
  appropriate team / user.** If you used the CLI to generate any apps it's highly recommended that you check
  that you are not exposing objects that should not be viewable.

### Other updates

- **Changed the default set up of social logins to use settings-based configuration instead of `SocialApps` in the database.**
  See the upgrade notes if you are using social logins to prevent issues.
  Thanks Alex for the suggestion and for helping with the updated documentation!
- Updated the default flowbite setup to disable the forms plugin. This was causing styling conflicts
  with the default DaisyUI styles on certain form elements, including checkboxes.
- Re-formatted the default form input template for readability.

### Upgrading

To migrate an existing project to `uv` see [this guide](./cookbooks.md#migrating-from-pip-tools-to-uv).

If your application was already using social logins defined in the database, the new settings-based declaration will
conflict and cause errors on social login.
To fix this you can either delete the `APPS` section of the relevant service in `settings.SOCIALACCOUNT_PROVIDERS`,
or you can move the credentials into your project environment (e.g. `.env`) and delete relevant the `SocialApp`
from the Django admin.

*November 29, 2024*

## Version 2024.11.3

This is a minor maintenance release with a few changes in preparation for adding `uv` support (coming soon!).

### Changed

- Pinned the version of `uv` used in CI and Dockerfiles.
- Added `venv` and `.venv` directories to the `.dockerignore` file and `make translations` target.
- The `make requirements` command now restarts containers in the background, making it easier to combine
  with other make targets.
- Added a catch-all to the `Makefile` to prevent error messages when running `make npm-install <package_name>`
  and similar commands.
- Updated README commands to consistently use `python manage.py` instead of just `./manage.py`.
- Made some minor formatting changes to `pyproject.toml`.
- Fixed the link to the multi-stage dockerfile docs in `Dockerfile.web`
- Upgraded a number of Python packages.
- Updated the `default_stages` of the `.pre-commit-config.yaml` file to the latest expected format (`pre-commit`).

*Nov 21, 2024*

## Version 2024.11.2

This release adds the ability to disable dark mode on Tailwind, upgrades front end libraries,
bumps the API client version, and has a handful of other small changes and fixes.

## Added

- **Added a new build option to disable dark mode for Tailwind builds.** (Thanks Arno for suggesting!)
- Added basic user-facing error messages to the standalone front end sign up and login workflows.

## Changed

- **Upgraded all JavaScript dependencies.**
- **Updated the API client to use the latest version 7.9.0, and updated the standalone front end to work
  with the latest changes.**
- Updated template-partials installation to be manually loaded, to allow for easier integration with other
  templating systems like django-cotton.
- Moved active tab highlighting to the base view in the example object demo.
- Made a few very minor edits to comments and whitespace in a few places.


## Fixed

- Fixed a bug where your migrations and tests would fail if your project name was > 50 characters
  (thanks Bernard for reporting!).
- Fixed a bug in the group chat demo where submitting an empty room name would take you to a 404 page.
- The `docker_startup.sh` file is no longer included if you are not using a docker-based deploy platform.
- Updated the `config/README` file which had outdated information that predated the migration to Kamal 2. 
  (Thanks Arno for reporting!)
- Improved comments in the kamal `secrets` file and `.env` files. (Thanks Arno for suggesting!)

## Removed

- The `.env` file is no longer included in zip downloads. This file was already removed from Github
  builds so this just makes the two consistent. Projects should create `.env` file from the `.env.example` file.
- Removed the `migrate_customers_to_teams` management command. This was added for an upgrade two years ago,
  and is assumed to be no longer needed.

*Nov 14 2024*

## Version 2024.11.1

This is a minor hotfix release.

### Fixed

- Fixed an issue where the team selector was accidentally transparent in Tailwind builds.
- Removed shadcn template that was accidentally included even if shadcn was disabled.

### Updated

- Removed extra whitespace from `form_tags.py`. (Thanks Brennon for reporting!)
- Updated `make help` to allow for commands defined in `custom.mk` with digits to also show up. (Thanks Arno for suggesting!)

*Nov 4 2024*

## Version 2024.11

This is a feature release with an emphasis on improving the Tailwind CSS experience with Pegasus.

Watch the video below for a demo, or read on for the highlights.

<div style="position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden; max-width: 100%; height: auto; margin-bottom: 1em;">
    <iframe src="https://www.youtube.com/embed/vPtxNWx21bw" frameborder="0" allowfullscreen style="position: absolute; top: 0; left: 0; width: 100%; height: 100%;"></iframe>
</div>

### Dark mode improvements

A dark mode selector was added to the navigation, allowing users to easily toggle between light, dark,
and "system default" mode.
The user's selection is preserved server-side in the session object, which also helps to prevent flickering across page loads.

### Better Theme Support

It's now easier than ever to change your project's theme.
Each project now supports a default light and dark theme which will be used throughout the site.
The default themes need only be changed in `tailwind.config.js`, and `settings.py` and everything else is taken care of.

See the updated [tailwind theme documentation](./css/tailwind.md#changing-your-themes) for more details.

### New shadcn integration and demo dashboard

A new build setting allows you to build your project with [shadcn/ui](https://ui.shadcn.com/) installed.
Shadcn is a great and versatile component library for React and Tailwind, but it is difficult to integrate it
into a Django project without building a separate front end.
Now Pegasus takes care of that integration for you, and provides a reference dashboard implementation of how to
work with the library.

The reference dashboard is a hybrid single-page React app served by Django.
It uses the same colors as the DaisyUI theme, and will update when you change your theme,
and has many interactive components.
However, it is not connected to any backend data—it is just a UI example.

Read more in the [shadcn docs here](./css/tailwind.md#shadcn).

### New flowbite integration and demo component page

Another new build setting allows you to build your project with [flowbite](https://flowbite.com/) installed.
Flowbite is another great component library for Tailwind and does *not* use React---making
it a great fit for htmx projects.
If you enable this setting, flowbite will automatically be installed and you can drop
flowbite components into any Django template.
The reference page has an example of a few of these components.

Read more in the [flowbite docs here](./css/tailwind.md#flowbite).

### Other updates

- **Upgraded all Python packages to their latest versions.**
- **[uv](https://docs.astral.sh/uv/) is now used to install Python packages in Docker files and Github actions.**
  - Also updated `make pip-compile` target to use `uv`.
  - This resulted in minor changes to all `requirements.txt` files.
- **Team invitation pages now prompt a user to log in instead of sign up if the email is associated with a known account.**
  (Thanks Daniel for suggesting!) 
- Your configured Github username, if available, will be used in a few places instead of a default value.
  (Thanks Richard for suggesting!)
- Added `bg-base-100` to the `<body>` tag of the base template and removed it from other components where it was 
  now redundant. This improves theming support when themes heavily modify the base color. (Tailwind builds only)
- Added equals signs to  `ENV` declarations in production Docker files, for consistency. (Thanks Denis for suggesting!)
- Slightly improved the styling of the e-commerce app.
- Overhauled the [Tailwind CSS documentation](./css/tailwind.md).

**Updates to the CLI ([release notes](https://github.com/saaspegasus/pegasus-cli/releases))**

- Fixed a bug on certain environments where the `pegasus` command conflicted with a local `pegasus` folder,
  causing import errors running the CLI.
- Apps created with `startapp` now use a `POST` for deletion instead of a `GET`.
- Deletion now includes a modal confirmation (Tailwind and Bulma builds only).

### Upgrading

If you're using Docker the `make upgrade` command won't work out-of-the-box due to the change
in how requirements files are managed. You will first have to rebuild your containers with:

```
make build
```

or

```
docker compose build
```

After that, you should be able to run `make upgrade` as normal.

*Nov 1, 2024*

## Version 2024.10

This release upgrades Kamal deployment to Kamal 2 and dramatically simplifies the Kamal deployment process.

### Kamal 2 deployment and related changes

In the upgrade to Kamal 2, the following changes were made:

- Updated Kamal to run from the root project directory instead of the `deploy` subdirectory.
  - Also moved the config file was also moved from `deploy/config/deploy.yml` to `config/deploy.yml`
- Moved environment secrets from `deploy/.env` to `.kamal/secrets` to match Kamal 2's recommendation.
- Kamal can now be installed and run with Docker without any additional workarounds [as described here](https://kamal-deploy.org/docs/installation/)
  The custom docker set up instructions have been removed.
- Kamal is now run as root by default, which dramatically simplifies the server setup process.
  There is now no need to run any manual steps to set up your server.
- Kamal now creates and manages its own docker network.
- Traefik has been dropped in favor of `kamal-proxy` for the proxy server, as per the new Kamal defaults.
- The `.gitignore` and `.dockerignore` files were updated to reflect the new structure.
- Added `apps.web.middleware.healthchecks.HealthCheckMiddleware` to workaround Kamal health checks and Django security
  features, [as outlined here](https://github.com/basecamp/kamal/issues/992#issuecomment-2381122195).
- Removed unnecessary media directory set up from `Dockerfile.web`. It is recommended to use an external storage service
  for media files and not the Docker container.

In addition, there were a few changes that affect projects that aren't using Kamal:

- `apps.web.locale_middleware` was moved to `apps.web.middleware.locale`
- `docker_startup.sh` was moved from the `deploy` folder to the project root.

The [Kamal documentation](./deployment/kamal.md) has been updated to reflect these changes.

### Other fixes

- **Subscriptions in a "past due" state are now treated as "active" for the purposes of feature gating
  and accessing the billing portal.**
  This is more consistent with [how Stripe treats subscriptions in this state](https://docs.stripe.com/api/subscriptions/object#subscription_object-status).
  (Thanks Luc for suggesting!)
- Fixed a bug where several `make` targets mistakenly included a `--no-deps` flag which would fail if your
  database container was not running. (Thanks Gary for reporting!)
- Fixed an issue where Stripe subscription webhooks weren't properly handled if you were using the embedded
  Stripe pricing table. (Thanks Andrew for reporting!)
- Fixed an issue introduced in 2024.9 where Stripe ecommerce webhooks weren't always processed correctly.
- Added a migration file to automatically work around [this dj-stripe issue](https://github.com/dj-stripe/dj-stripe/issues/2038)
  so that it wasn't a manual process.

*Oct 15, 2024*

## Version 2024.9.3

This release is mainly [an update to the CLI](https://github.com/saaspegasus/pegasus-cli/releases/tag/v0.3):

### CLI updates

- **You can now generate apps that work seamlessly with Pegasus teams** (will use `BaseTeamModel` and add the team
  slug and permissions checks to all urls and views).
- The CLI now generates a default `admin.py` config for each data model.
- User foreign keys now use `settings.AUTH_USER_MODEL` instead of being hardcoded to `apps.users.models.CustomUser`.

### Other changes

- Fixed an issue where HTMX links without href tags weren't showing a pointer cursor on some CSS frameworks.
- Add default region to Redis and Postgres configurations in `render.yaml` to make it easier to find/replace
  them when changing your project's region. (Thanks Jacob for suggesting!)

*Sep 26, 2024*

## Version 2024.9.2

This release fixes a bug that prevented the CLI from running on Windows machines.
Thanks Jonathan for reporting!

If you don't want to upgrade you can just `pip install pegasus-cli==0.2.1` to apply the fix.

*Sep 20, 2024*

## Version 2024.9.1

This release fixes a few things in the 2024.9 release.

- Updated the `bootstrap_ecommerce` management command to create `ProductConfiguration` objects for all active
  Products in Stripe.
- Fixed an issue on the ecommerce homepage where a closing `</div>` tag was misplaced if a product
  didn't have a default price set.

*Sep 18, 2024*

## Version 2024.9

There are two big updates in this release:

1. The Pegasus CLI, which allows you to instantly spin up new apps.
2. E-Commerce/Payments improvements.

### The Pegasus CLI

The [Pegasus CLI](https://github.com/saaspegasus/pegasus-cli/) is a standalone command-line tool that allows you
to instantly spin up new Django apps.
You can specify as many data models as you want and it will generate a starting CRUD interface for each of them.

Here's a quick demo:

<div style="position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden; max-width: 100%; height: auto; margin-bottom: 1em;">
    <iframe src="https://www.youtube.com/embed/wKS_bbD5RVs" frameborder="0" allowfullscreen style="position: absolute; top: 0; left: 0; width: 100%; height: 100%;"></iframe>
</div>

**At the moment the CLI only supports HTMX build of Pegasus.**
A React-based implementation is planned for a future date.

Huge thanks to Peter for his excellent [Pegasus example apps](https://github.com/pcherna/pegasus-example-apps-v2)
project which served as a reference for implementing the CRUD application and pagination.

### E-Commerce / Payments demo improvements

This is a series of updates designed to make it easier to build a functional end-to-end application
on top of the e-commerce demo.

- Added a `ProductConfiguration` model to attach additional metadata to products.
- E-Commerce product URLs and views now use the `ProductConfiguration` `slug` field instead of the Stripe Product IDs.
- Added a `@product_required` decorator that can be used to restrict access to views based on whether the user has purchased a product.
- Added a demo "access product" page that shows how to use the `@product_required` decorator.
- Added `user_owns_product` and `get_valid_user_purchase` helper functions.
- Improved the navigation and use of breadcrumbs in the demo UI.
- **See upgrade notes for information about migrating previous data to the new set up.**

See also: the updated [Payments docs](./payments.md).

### Other Changes

#### Added

- **Added `django-htmx` and `django-template-partials` as first-class dependencies to HTMX builds.**
  These libraries are used by the CLI and will be used for more HTMX-based functionality moving forwards. 
- Added `make manage` command to run arbitrary `manage.py` commands in a docker environment.
  E.g. `make manage ARGS='createsuperuser'`.
- Added the ability to pass arguments to `make test` in docker. E.g. `make tests ARGS='apps.teams --keepdb'`.
  (Thanks David for the suggestion!)

#### Changed

- Changed links on the tailwind signup page to use `pg-link` class instead of explict tailwind classes.
  (Thanks Peter for the suggestion!)
- Silenced extraneous djstripe warnings when running tests. (Thanks Chris for the suggestion!)
- Added `.vscode` and vs workspace files to the project `.gitignore`.
- Switched from `assert` statements to `raise ValueError` in the e-commerce Stripe checkout confirmation view.
- Moved some of the currency helper functions out of the `subscriptions` app into `utils.billing` so they can be
  used in ecommerce workflows even if subscriptions are disabled.
- Set `PYTHONUNBUFFERED` and `PYTHONDONTWRITEBYTECODE` in docker compose file for python containers.
  (Thanks Richard for the suggestion!)
- Upgraded Django to 5.1.1.

#### Fixed

- Fixed a typo in the help text for the `bootstrap_ecommerce` command.
- Fixed a bug where `user_teams` context processor could cause a crash if auth middeware didn't run
  (for example, on a 500 error page in production).

### Upgrade Notes

If you have existing `Purchase` data in your application you will need to migrate it to the new `ProductConfiguration` structure.

This is a three-step process:

First you will need to apply the database updates, but allow `Purchase.product_configuration` to be null.
Instead of running `./manage.py migrate` you will have to run the following command:

```bash
./manage.py migrate ecommerce 0002
```

After running this, you can run the following command to migrate the existing data:

```bash
./manage.py migrate_ecommerce
```

The `migrate_ecommerce` management command will:

1. Create `ProductConfiguration` objects for all products in `settings.ACTIVE_ECOMMERCE_PRODUCT_IDS`
2. Create `ProductConfiguration` objects for all products referenced in existing `Purchase` models.
3. Set `purchase.product_configuration` to the new `ProductConfiguration` object for each `Purchase`.

Finally, you can make the `Purchase.product_configuration` field non-null, by running:

```bash
./manage.py migrate ecommerce 0003
```

**New projects, or projects without any existing purchase data can skip these steps and run `./manage.py migrate` directly.**
However, you may still want to run `./manage.py migrate_ecommerce` to populate `ProductConfiguration` objects for your active products.

*Sep 17, 2024*

## Version 2024.8.2

This is a maintenance release that includes a number of mostly small fixes and updates,
and updates Django to version 5.1.

### Fixed

- **Fixed a few styling issues on Bulma builds**:
  - Disabled dark mode. The styling for Dark mode was not fully supported by Bulma and led to strange-looking layouts.
  - Fixed an issue where the active tab wasn't properly highlighted in certain cases on Bulma builds.
- Fixed an issue with sqlite builds where the default `DATABASE_URL` would cause the DB to switch to Postgres.
  (Thanks Harry and Richard for reporting!)
- Switched allauth from [Twitter](https://docs.allauth.org/en/latest/socialaccount/providers/twitter.html)
  (which seems no longer supported) to [Twitter Oauth2](https://docs.allauth.org/en/latest/socialaccount/providers/twitter_oauth2.html), which still works.
  (Thanks Bandi for reporting!)
- Fixed an issue introduced in version 2024.8 which caused Heroku Docker deploys to fail.
  Heroku [does not support caching](https://stackoverflow.com/a/78901250/8207), so it has been removed from Heroku Docker builds.
  (Thanks Richard for reporting!)
- Fixed a bug where the `team_nav_items.html` and `team_selector.html` templates could be accidentally included even
  if you built without teams.
- Changed the (unused) `text-muted` css class to `pg-text-muted` in a handful of places on Tailwind builds.
  (Thanks Peter for reporting!)
- Removed unused `AWS_S3_CUSTOM_DOMAIN` variable from `.env` files.

### Changed

- **Upgraded Django to version 5.1.**
- Upgraded all Python packages to their latest versions.
- Updated Pegasus color CSS variables to use the DaisyUI variables, so that they change
  when you change DaisyUI themes. (Thanks Peter for the suggestion!)
- Removed `custom.mk` if your project was not generated with a `Makefile`. (Thanks Finbar for reporting!)
- Removed "Containers started" message from `make start` command that never executed. (Thanks Richard for reporting!)
- Better style inputs of type `time` and `datetime-local` in forms on all CSS frameworks. (Thanks Peter for reporting and fixing!)
- Simplified Bulma navbar to use bulma native classes instead of custom CSS. (See upgrade note below.)
- Updated default Github repo in `app-spec.yml` to use raw project slug instead of the hyphenated version.
  (Digital Ocean deployments, only, thanks Richard for suggesting)
- Moved `SERVER_EMAIL` and `DEFAULT_FROM_EMAIL` from `settings_production.py` to main `settings.py` file,
  and made it possible to set them via the environment/`.env` file.
- Added many more common settings and secrets to the Kamal `deploy.yml` file.

### Documentation

- Improved the documentation on [customizing the Material Bootstrap theme](./css/bootstrap.md#customizing-the-material-theme).
- Added documentation for [deploying multiple apps to the same VPS with Kamal](./deployment/kamal.md#cookbooks).

### Upgrading

- Bulma builds may need to add the `is-tab` class to `navbar-items` in the top nav to mimic the updated navbar styling.

*August 23, 2024*

## Version 2024.8.1

This is a maintenance release which upgrades HTMX to version 2.0 and fixes a handful of minor bugs.

### Changed

- **Upgraded HTMX to [version 2.0](https://htmx.org/posts/2024-06-17-htmx-2-0-0-is-released/).**
  See upgrade note below.

### Fixed

- Fixed a bug on some environments where `make build-api-client` would wrong relative to the wrong directory.
  (Thanks Ben for finding and fixing!)
- Downgraded Postgres from 16 to 14 on Digital Ocean deployments, due to
  [an issue with permissions on version 16](https://www.digitalocean.com/community/questions/how-can-i-create-a-postgres-16-user-that-has-permission-to-create-tables-on-an-app-platform-dev-database)
  that was causing new Digital Ocean deployments to fail.
  (Thanks Panagiotis for reporting!)
- Switched the default celery pool to [solo](https://docs.celeryq.dev/en/stable/internals/reference/celery.concurrency.solo.html) in development,
  to fix issues running on Windows. See [updated docs](./celery.md).
- Updated in-app help hint to recommend running `./manage.py bootstrap_ecommerce` instead of `./manage.py djstripe_sync_models price`.

### Upgrading

Htmx 2.0 requires loading new extensions.
If you were loading HTMX extensions in your own templates, you will have to upgrade the location of those to the 2.0 versions.

Before:

```
<script src="https://unpkg.com/htmx.org/dist/ext/ws.js" defer></script>
```

After:
```
<script src="https://unpkg.com/htmx-ext-ws@2.0.0/ws.js" defer></script>
```

*August 13, 2024*

## Version 2024.8

This is a maintenance release with many small updates and fixes.

### Added

- **Added test cases for subscription decorators, feature gating, and views.**
  These can be extended/adapted to test custom subscription logic.
  Also added utility functions to create test products, subscriptions and mock requests.
- Added a test that will fail if your project is missing any database migrations.
  [More on this concept here](https://adamj.eu/tech/2024/06/23/django-test-pending-migrations/).
- **Added an example landing page to Tailwind builds, based largely on [Scriv's landing page](https://scriv.ai/).**
- Added `TURNSTILE_KEY` and `TURNSTILE_SECRET` to Kamal's default secrets.
- Added a section on configuring static files to the [production checklist](./deployment/production-checklist.md#check-your-static-file-setup).

### Changed

- **Code is now automatically formatted for all projects.**
  The "Autoformat code" check box has been renamed to "Enable linting and formatting" and now only controls whether
  `ruff` and the pre-commit hooks are included in the project download.
  Projects that had already enabled auto-formatting are unaffected by this change. (See upgrade notes below.)
- **The example landing pages are now used as the project's landing page instead of being listed in the examples**.
  (Bulma and Tailwind builds only.)
- **Team invitation emails are now better styled, matching the same format as account emails.** (Thanks EJ for the suggestion!)
- The `EMAIL_BACKEND` setting is now configurable via an environment variable.
  Also, added a commented-out example of how to set email settings for a production email provider (Mailgun).
- Apt and pip packages are now cached across Docker builds, which should result in faster build times after the first build.
  (Thanks Tobias for the suggestion!)
- Improved the display format of "role" in the team invitation list. (thanks Andy for the suggestion!)
- Change `user/` to `YOUR_GITHUB_USERNAME/` in the Digital Ocean `app-spec.yml` file to make it more obvious that
  it should be edited. (Thanks Stephen for suggesting!)
- Changed the UI of social logins on the "sign in" page to match that of the "sign up" page on the Material Bootstrap theme.
  This makes the implementation more extensible and more consistent with other CSS frameworks.
- **Upgraded all Python packages to the latest versions.**

### Fixed

- Fixed a bug where the formatting `make` targets were still calling `black` and `isort` instead of `ruff`.
  `make black` is now `make ruff-format` and `make isort` is now `make ruff-lint`.
- Fixed a bug where the sign up view tests would fail in your environment if `settings.TURNSTILE_SECRET` was set.
  (Thanks Finbar for reporting!)
- Fixed translations on the user profile form field names.
- Removed `svg` as an option for profile picture uploads, to prevent the possibility of using it as an XSS attack vector.
  ([More info on this threat here](https://medium.com/@rdillon73/hacktrick-stored-xss-via-a-svg-image-3def20968d9)).
- Disable debug toolbar in tests, which fixes test failures under certain conditions.
- Bumped the Postgres version used by Digital Ocean deployments from 12 to 16.
  Digital Ocean has deprecated support for version 12. (Thanks Stephen for reporting!)
- Simplified how the list of social login buttons is rendered, and make social login buttons work when
  configuring social applications in settings (previously buttons only showed up if you configured apps in the database).
  See upgrade note below.

### Removed

- Deleted the "sticky header" html and CSS code that was only used on the example landing pages.

### Upgrade Notes

- If you had **not** been using auto-formatting until now, you should first follow the instructions for
[migrating to auto-formatted code](./cookbooks.md#migrating-to-auto-formatted-code) prior to upgrading to this release.
Otherwise you will likely get a lot of formatting-related merge conflicts when trying to upgrade.
  - If you already enabled auto-formatting (most projects), you don't need to do anything.
- If you had previously configured allauth social applications in the database *and* in your settings file,
you may see a duplicate "Login with XXX" button on the sign up and login pages.
To fix this, remove the social application from either your settings or the database.

*August, 7, 2024*

## Version 2024.6.1

This is hotfix release that addresses a few issues from yesterday's update:

- Fix app styles accidentally being purged during the Docker build process.
  This caused styling on Docker-based deployments for tailwind builds. (Thanks Steve for reporting!)
- Moved channels url import to after Django initialization.
  This fixes an `AppRegistryNotReady` error when deploying asynchronous apps with the AI chat app enabled.
  (Thanks Roman for reporting!)
- Don't create the periodic task to sync subscriptions unless per-unit billing is enabled.

*June 6, 2024*

## Version 2024.6

This is a feature release with a few big updates and a lot of smaller ones.

### AI model changes

The library used for non-OpenAI LLMs has been changed from [`llm`](https://github.com/simonw/llm)
to [`litellm`](https://docs.litellm.ai/docs/). Reasons for this change include:

- It has far fewer additional dependencies.
- It supports async APIs out of the box (for most models).
- The `llm` library is more targeted for the command line use-case, whereas `litellm` offers similar functionality
  as a native Python library with a cleaner API.

Litellm can still be used with all common AI models, including OpenAI, Anthropic/Claude, and local models
(via ollama). For details on getting started with `litellm` see the updated [AI documentation](./ai.md).

### Formatting and linting now use Ruff

Black and isort have been replaced with [ruff](https://github.com/astral-sh/ruff)---a Python linter/formatter
that offers the same functionality as those tools but is much faster.

Additionally, Pegasus will now remove unused imports from your files automatically, both
when building your project and if you have set up `pre-commit`.

This change should be a relatively seamless drop-in replacement, though you may see some new lint errors
in your projects which you can choose to address.

### Spam prevention updates

There has been a dramatic increase in spam-bots over the last month.
Many of these bots target seemingly-innocuous functionality like sign up and password reset forms.

This version includes a few updates to help combat these bots.
First, you can now easily add [Cloudflare turnstile](https://www.cloudflare.com/products/turnstile/) to your sign up forms,
which will present the user with a captcha and should help reduce bot sign-ups.
See [the turnstile documentation](./configuration.md#turnstile) for information on setting this up.

Additionally, the `ACCOUNT_EMAIL_UNKNOWN_ACCOUNTS` setting has been set to `False` by default.
This prevents "forgot password" and "magic link" emails from being sent out to unknown accounts.
It should also help reduce unnecessary email sending.

Finally, the [admin dashboard](#admin-dashboard) no longer shows users with unconfirmed email addresses if you have set
`ACCOUNT_EMAIL_VERIFICATION = 'mandatory'`.
This helps filter out likely bots from the report to provide clearer visibilty of people actually signing up for your app.

### Complete changelog

Below is the complete set of changes in this release.

#### Added

- **Added configurable captcha support on sign up pages, using [Cloudflare turnstile](https://www.cloudflare.com/products/turnstile/).**
  See [the turnstile documentation](./configuration.md#turnstile) for more information on setting this up. (Thanks Troy, Jacob, Robert and others for suggesting.)
- Added API views for two-factor authentication, and to change the logged-in user's password. (Thanks Finbar for suggesting!)
- Add UI to tell users they need a verified email address prior to setting up two-factor auth.
  - Also added a `has_verified_email` helper class to the `CustomUser` model.
- Added tests for the delete team view for both team admins and members. (HTMX builds only)
- Added test for team member removal permissions.
- Add display and sort on the number of active members in the teams admin.

#### Fixed

- Fixed a bug where team names longer than 50 characters could cause a crash during sign up.
- Fixed a bug where multi-factor authentication QR codes had a dark background when dark mode was enabled (Tailwind builds only).
  (Thanks Artem for reporting!)
- Fixed a bug where it was possible to bypass two-factor-authentication when using the API authentication views.
  (Thanks Finbar for reporting and helping with the fix!)
- Fixed a bug where deleting the user's only team while impersonating them resulted in a temporary crash.
  (Thanks EJ for reporting!)
- Fixed a bug where creating an API key crashed if your user's first + last name combined to more than 40 characters.
  (Thanks Luc for reporting!)
- Improved the UI feedback when LLMs fail (e.g. if your API key is wrong or ollama is not running).
- Removed the `static/css` and `static/js` directories from the `.dockerignore` file so that other project files
  can be included in these directories.
  Also updated the production Docker build process so that any existing files are overwritten
  by the built versions. (Thanks Raul for reporting!)
- Made some performance improvements to the production Dockerfile build (don't rebuild the front end if there are
  no changes in the dependent files).
- Better support trialing subscriptions with no payment methods.
  The subscription UI will now show the date the trial ends and won't log errors about missing invoices. (Thanks Jarrett for reporting!)

#### Changed

- **Upgraded all Python packages to the latest versions.**
- **Upgraded all JavaScript packages to the latest versions.**
- **Non-OpenAI builds now use `litellm` instead of `llm`.** See above. (Thanks Sarthak for the suggestion!)
- **Changed the formatter/linter from `black` and `isort` to [ruff](https://github.com/astral-sh/ruff).** See above.
  - Also addressed a handful of minor linting errors that came up as a result of this change.
  - Codebase linting is now substantially faster.
  - Unused imports are now automatically removed when building your projects.
- **Celerybeat now uses the `django-celery-beat` library to store tasks in the database instead of on the filesystem.**
  This improves support for celerybeat on Docker-based platforms. (Thanks Peter and Artem for the suggestion!)
  - Also added a migration to save the default scheduled tasks in the database.
- The login API response has changed, to allow for two-factor auth prompts, and more machine-readable status fields.
- Removed the no-longer-used `use_json_field=True` argument from wagtail `StreamField`s.
- The admin dashboard no longer shows users with unconfirmed email addresses if you have set `ACCOUNT_EMAIL_VERIFICATION = 'mandatory'`.
- The admin dashboard now includes sign ups from the current date, by default.
- Changed behavior when team role checks fail from raising a `TeamPermissionError` to returning a 403 response,
  and updated affected tests. One side effect of this is that the stack traces are removed from successful test runs.
- Secret keys should no longer change every time you build your Pegasus project.
  They are also now clearly prefixed with `django-insecure-` to indicate that they should be changed in production.
- Updated the default OpenAI chat model to gpt-4o.
- Upgraded the openapi client generator to version 7.5.0 and also pinned the version used by `make build-api-client`
  to the same one.
- Team IDs are now optional on the create team page (HTMX builds only).
- Add clearer error message when charts are broken due to api config issue. (Thanks Yngve for reporting!)
- Added `assume_scheme="https"` to form `URLField`s to be compatible with Django 6 behavior.
- Added `FORMS_URLFIELD_ASSUME_HTTPS = True` to be compatible with Django 6 behavior.
- Set `ACCOUNT_EMAIL_UNKNOWN_ACCOUNTS = False` by default, so that "forgot password" emails do not get sent to unknown accounts.
  This can help prevent spam bots.

#### Removed

- Removed `black` and `isort` from dev-requirements, since they have been replaced by `ruff`.
- Removed `llm` library and associated code, since it has been replaced by `litellm`.
- Removed no longer used `TeamPermissionError` class.

#### Standalone front end

The following changes affect the experimental [standalone front end](./experimental/react-front-end.md):

- **The standalone React front end now supports two-factor-authentication.**
- Improve the UI when you have login issues in the standalone React front end.

*June 5, 2024*


## Version 2024.5.3

This is a hotfix release that fixes a bug where the landing and dashboard page image was accidentally
removed if you built without the examples enabled.

*May 21, 2024*

## Version 2024.5.2

This is a hotfix release that fixes a bug that prevented the team management page
from loading in certain browsers if you built with a React front end and with translations enabled.
Thanks Finbar for reporting!

- Added `defer` keyword to various bundle scripts so they are loaded after the JavaScript translation catalog.
- Updated references to `SiteJS` to run on the `DOMContentLoaded` event to allow for usage of the `defer` tag.

*May 16, 2024*

## Version 2024.5.1

This is a hotfix release that fixes issues running the [experimental React frontend](./experimental/react-front-end.md)
in Docker. Thanks Mohamed for reporting this!

- Fix `api-client` path in the frontend docker container and add to `optimizeDeps` in vite config.
- Mount `node_modules` as an anonymous volume in the frontend docker container, so it is not overwritten.
- Automatically create `./frontend/.env` when running `make init` if it doesn't exist.

*May 14, 2024*

## Version 2024.5

This is a major release with several big updates.
Here are a few highlights:

### New AI models

In addition to using OpenAI chat models, you can now build the Pegasus AI chat applicaiton
with the [`llm` library](https://github.com/simonw/llm). This lets you run the chat application
against any supported model---including the Anthropic family (Claude 3), and local models like Llama 3.

Additionally, the image generation demo now supports Dall-E-3 and Stable Diffusion 3.
For complete details, see the new [AI documentation](./ai.md).

### Health Checks

A new setting allows you to turn on health checks for your application, powered by
[django-health-check](https://django-health-check.readthedocs.io/en/latest/).
This will create an endpoint (at `/health` by default) that pings your database, Redis instance,
and Celery workers and returns a non-200 response code if there are any identified issues.

These endpoints can be connected to a monitoring tool like [StatusCake](https://www.statuscake.com/)
or [Uptime Robot](https://uptimerobot.com/) so that you can be alerted whenever your site is having issues.

See the section on [monitoring](./deployment/production-checklist.md#set-up-monitoring) in the
production checklist for more information.

### Allauth updates

The [django-allauth](https://docs.allauth.org/en/latest/) library was updated to the latest version,
which enabled several useful changes.

The first is a "sign in by email code" option which can be used in addition to the standard
username/password and social option.
Users can request a code be sent to their registered email and can then use that to login.
See [the magic code documentation](./configuration.md#enabling-sign-in-by-email-code) to enable/disable this.

The second is using the recent [multi-factor authentication](https://docs.allauth.org/en/latest/mfa/index.html)
support added to allauth in favor of the third-party `django-allauth-2fa` library.
This reduces dependencies and puts all of authentication functionality on a standard path moving forwards.

The complete release notes are below:

### Added

- **Added an optional health check endpoint at /health/.** (see above for details)
- **Added an option to connect the chatbot to other LLMs**. (see above for details)
- **The AI image generation now supports Dall-E 3 and Stability AI.**
- **All generated projects now include a `LICENSE.md` file.**
  The goal of the license file is not to change how Pegasus can be used in any way, but rather to document those
  terms in the codebase itself (previously they were only documented on the [terms page](https://www.saaspegasus.com/terms/)).
  For more information you can see the new [license page](https://www.saaspegasus.com/license/).
- **Added support for "magic-code login", where a user can login to the site by requesting a code to their email address.**
  [Documentation.](./configuration.md#enabling-sign-in-by-email-code)
- **Google cloud run builds now support Redis.** For details, see the [updated documentation](./deployment/google-cloud.md).
  (Thanks Forrest for suggesting!)
- Added a `custom.mk` file where you can add additional `make` targets without worrying about future Pegasus upgrades.
  (Thanks John for proposing this!)

### Changed

- Upgraded allauth to the latest version (0.62.1).
- **Migrated two-factor authentication from the third-party `django-allauth-2fa` to the `django-allauth` built-in implementation.**
  See upgrade notes below for migrating existing projects.
- Refactored how many allauth views work to be compatible with their new template override system.
- **Bootstrap and Bulma builds: Move sidebar navigation into the mobile menu instead of having it take up the top of the
  screen on mobile screens**, similar to how things already worked on Tailwind and Material. (Thanks Luc for the nudge!)
  - This includes splitting out the menu items into their own sub-template files so they can be included in both menus.
- Inline buttons are now spaced using the `gap` CSS property instead of the `pg-ml` class on individual buttons.
- `Alpine.start()` is now called on `DOMContentLoaded` loaded event instead of using `window.load`.
  This makes Alpine-powered UIs more responsive, especially when used on pages with lots of images.
- **Updated external JavaScript imports to use [the `defer` keyword](https://www.w3schools.com/tags/att_script_defer.asp)
  for slightly better page load performance.** (See upgrade note.)
  - Also updated inline JavaScript code in a handful of places to be compatible with deferred scripts.
- Added a Github logo to connected Github accounts on profile page.
- **The AI image demo and code has been moved to a first-class Pegasus application / tab.**
- Update the docker container registry used by Google Cloud to reflect the latest version in Google.
  Also push more Google Cloud configuration variables out of the Makefile and into the environment variables.
  (Thanks Erwin for reporting!)
- Added additional `.env` files to `.dockerignore` for Google Cloud builds.
- Bumped django to the latest `5.0.6` release.

### Fixed

- **SQLite build now properly parse `DATABASE_URL` if it is set. This fixes issues deploying to certain platforms
  when building with SQLite.** (Thanks Manasvini for reporting!)
- Updated allauth documentation links in the README to point to the new [allauth docs site](https://docs.allauth.org/).
  (Thanks Shantu for reporting!)

### Removed

- Removed several no-longer-needed allauth template files.
- Removed deprecated "version" field from the dev `docker-compose.yml` file. (Thanks Moyi for reporting!)
- Removed no-longer-used `pg-ml` css spacing class.
- Removed redundant type="text/javascript" declarations from a few `<script>` tags.
- Removed unused HTMX script import from employee app demo page.
- Removed the no-longer-used `openai_example` app (functionality has been moved to `apps.chat` and `apps.ai_images`).
- Removed the no-longer-needed `AccountAdapter` class. This class was previously used to add two-factor support
  to login, which is now handled natively by allauth.

### Upgrading

**Two-factor authentication**

If you are using two-factor authentication you must run:

```
python manage.py migrate_allauth_2fa
```

Which will bring across existing device set ups and recovery codes.
**If you don't do this, you will remove two-factor-authentication configuration for all users who have set it up,
compromising their security.**

**JavaScript defer changes**

The change of adding the `defer` keyword to `<script>` imports could have unintended consequences if you were
relying on functions / functionality in your scripts being available on page load.
This would most likely manifest as a browser JavaScript error of the form:
`Uncaught ReferenceError: htmx/SiteJS/etc. is not defined`.

To resolve this, make sure all additional dependencies are also loaded with `defer` (for external scripts),
or only referenced after the `'DOMContentLoaded'` event (for inline scripts).
Alternatively, you can remove the `defer` keyword from the `<script>` tags in `base.html` or affected templates
to restore the previous behavior.

*May 9, 2024*

## Version 2024.4.2

This is a maintenance release with a number of fixes and small changes.
The most notable change is that the OpenAI chat example is now fully asynchronous.

### Added

- **Kamal deployments now support celerybeat for scheduled tasks out-of-the-box.** (Thanks Peter for the suggestion!)
- Added an example celerybeat configuration to the built-in examples.


### Changed

- **The websocket OpenAI chat example is now fully asynchronous.** This should substantially improve the number
  of concurrent sessions supported by the app.
- Use alpine to disable submit and clear input in the OpenAI chat example (thanks Artem for the suggestion!)
- Renamed "OpenAI Demos" tab to "AI Demos".
- Renamed `get_chatgpt_response` task to `get_chat_response`.
- Change the default admin ordering for Users to date joined, descending.
- Google cloud builds only: `.env.production` is no longer included in Pegasus builds, since it was ignored by git.
  Instead `.env.production.example` is included. (Thanks Naveed for reporting!)
- Attach Stripe Customer to User/Team in `provision_subscription` function instead of the view, which makes it work
  in webhook processing as well.

### Fixed

- **Fixed a bug where non-admins of teams could not view their own team settings/members.**  (Thanks Bernard for reporting!)
- Fixed a bug where deleted subscriptions with `canceled_at_period_end=True` could cause a crash when trying to view them.
  This bug would typically only manifest in test mode or if you manually deleted subscriptions in Stripe.
- Fixed a crash when looking up the currency for a deleted subscription.
- Added a constraint to the `Membership` model to disallow users being "added" to a team more than once.
  (Thanks Zac for reporting!)
- Fix typo instructing you to "comment out" mailgun settings instead of "uncomment" them. (Thanks Adam for reporting!)
- Google cloud builds only: added `--platform linux/amd64` to the `gcp-build` make target,
  to address [build/deploy issues from Mac M2s](https://stackoverflow.com/a/68766137/8207). (Thanks Naveed for reporting!)


### Documentation

- Improved the [instructions for running Kamal with Docker](https://docs.saaspegasus.com/deployment/kamal/#running-kamal-with-docker).
  Thanks EJ and Simon for the help with this!

*April 22, 2024*

## Version 2024.4.1

This is a hotfix/security release that fixes an issue where the `deploy/.env` file was not properly
ignored by the `.dockerignore` when using Kamal deployment.
This resulted in credentials being included in built Docker containers.

You are affected if you:

1. Deployed your application with Kamal.
2. Made your built Docker image publicly available.

If this is the case it is recommended to immediately upgrade and cycle any credentials that were listed in `deploy/.env`.
Sorry about the inconvenience!
Thanks to Denis for reporting and to Erwin for pointing out the security implications.

Note: if you'd like to patch this issue without upgrading, you can replace `.env` in your `.dockerignore` file with
`**/.env`.

*Apr 17, 2024*

## Version 2024.4

There are two major updates in this release, a new websocket-based streaming chat UI,
and upgrading Bulma to version 1.0.


### Streaming Chat UI

The OpenAI Chat demo now streams responses in real time.
This provides a much better user experience for chat applications, as the user doesn't
stare at a "loading" screen for a long time waiting for the complete response.

This feature is currently only supported on HTMX builds.
It will be automatically enabled if you enable the "asynchronous" build option.
If you do not enable async you will still be able to use the old UI.

You can watch a demo of this and learn how it's built in this video:
[Building a ChatGPT clone with Django, Channels, and HTMX](https://www.youtube.com/watch?v=8JSiiPW4S0A).

### Bulma upgrade to 1.0

[Bulma 1.0](https://github.com/jgthms/bulma/releases/tag/1.0.0) was released in late March and Pegasus
has been updated to support it.
The UI will change slightly but there should not be any breaking or substantial changes as a result of this upgrade.
See details below.

### New

- **OpenAI Chat demo now supports streaming responses.** This is only turned n for HTMX builds with asynchronous support enabled.
  Big thanks to Artem for the implementation support!

### Updated

- **Upgraded Bulma to 1.0.** Related changes (bulma builds only):
  - Switched all variables from Sass variables to CSS variables.
  - Updated syntax of all responsive CSS to use `@include mixins` instead of `+from`, `+until`, etc.
  - Switched how bulma is initialized from `@import` to `@use`.
  - Removed some unnecessary variable declarations from `site-bulma.scss`
- **Upgraded other JavaScript packages to their latest versions.**
- Upgraded Django to 5.0.4

### Fixed

- Fixed a bug where the standalone front end Dockerfile did not install the right dependencies.
  In some cases, this caused the front end to not start properly.
- Fixed a bug where viewing a non-existent team would cause a 500 error instead of a 404.
- Removed some unused code in the OpenAI chat example if you were building with React.

*April 5, 2024*

## Version 2024.3

This is a big feature release!
The main goal of the release is to make it easier to use Pegasus with separate front ends and mobile applications.
Here are the highlights:

### API Authentication and JWT support

Pegasus now optionally includes API endpoints for registering, authenticating and verifying users (via JSON web tokens or JWT).
This provides all the pieces needed to make your Pegasus application a backend to a standalone front end,
for example a React or Next.js SPA, or a mobile application.

You can read more about this feature in the [Authentication API documentation](apis.md#authentication-apis).

### Standalone React (Vite) Front End (Experimental)

In addition to the authentication API support, Pegasus now includes an optional standalone React demo application
built with React, [Vite](https://vitejs.dev/) and [React Router](https://reactrouter.com/en/main).

The demo application includes sign up and login workflows, helper components to manage authentication and
protected routes, and a port of the employee demo application (using the same code as the Pegasus version).

You can watch a demo below and learn more about it in [the React front end documentation](experimental/react-front-end.md).

<div style="position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden; max-width: 100%; height: auto; margin-bottom: 1em;">
    <iframe src="https://www.youtube.com/embed/8CcTs2SdMLk" frameborder="0" allowfullscreen style="position: absolute; top: 0; left: 0; width: 100%; height: 100%;"></iframe>
</div>


### Teams Navigation Updates

Application navigation now uses a new team selector component.
This allows users to quickly see their team list and switch between teams directly from any page.
Team management has also been moved under this component, and the previous team section of
the navigation was removed.

### Complete release notes

#### Added

- **API Authentication and JWT support.** See above.
- **An Experimental Standalone React (Vite) Front End**. See above.

#### Changed

- **Team navigation overhaul**. See above.
- **Made changes to the Employee demo app to support being shared with the standalone front end.**
  - Move `BrowserRouter` declaration outside the `EmployeeApplication` component so the routes can be used in different routers.
  - Change all `Link` and `Route` declarations in `EmployeeApplication` to be relative paths instead of absolute.
  - Pass image references into the app as props instead of relying on an externally defined constant (`STATIC_FILES`).
    This allows the application to be served from multiple URL endpoints without modification.
  - Pass `urlBase` to the nested components, so that they can route back to different paths (e.g. when employees are saved).
  - Switch the department choice field from a passed-in `EMPLOYEE_DEPARTMENT_CHOICES` data structure to the `DepartmentEnum`
    object that is included with the API client.
  - Change file extension for several components from `.js` to `.jsx`.
- **Made changes to the JavaScript api client to support being shared with the standalone front end**
  - **Moved the api-client from `assets/javascript/api-client` to an `api-client` folder in the root of the repository.**
  - **The api client is now installed as a linked local dependency in `package.json` instead of referenced as part of the code.**
    This makes it easier to move the api client code or install it as a hosted npm package.
    Also updated all references to the api-client to reflect this change.
  - Renamed the `api-client` make target to `build-api-client` to avoid conflicting with the filesystem folder.
  - **The API client is now built dynamically, just in time, when you create your project. Also updated the API client version to 7.4.0.**
    This may result in slight differences in generated API client code, but the resulting code should be *more* correct and
    there should not be any breaking changes to existing Pegasus functionality.
- Be more explicit in the webpack config about using babel-loader presets instead of relying on `babelrc`
- Exclude frontend from the base type check config in `tsconfig.json`. The front end is checked independently.
- Auto-size the navigation column in `app_home.html` (Bootstrap builds only).
- Make ALLOWED_HOSTS configurable by environment variables in both dev and production. (Thanks Shawn for suggesting)
- **Removed `.env.docker`. Docker development environments are now managed by `.env` (same as native environments).**
  Also updated the documentation on this.
- Added a search input to Team admin UI.
- Added a `signup_closed.html` template which improves the styling of the "signups closed" page if you turn off public signups.
  (Thanks EJ for the suggestion!)
- Added an `account_inactive.html` template which improves the styling of the "inactive account" page if an inactive user
  tries to login. (Thanks Lauren for the suggestion!)
- Improved the contrast on help text in dark mode. (Tailwind builds only)
- Changed the main icon on the landing page and dashboard from the old rocket to the current SaaS Pegasus rocket.
- Added a "Built with SaaS Pegasus" line to the default generated footer.
- Bump Django to latest 5.0.3 release.

#### Fixed

- Fixed hard-coded reference to the Pegasus dev project in Google Cloud media configuration in `settings_production.py`
- Fixed a bug where the two-factor QR codes were very difficult to scan in dark mode. (Tailwind builds only)
- Made certain account pages (password reset, two-factor auth, etc.) darker in dark mode. (Tailwind builds only)
- Removed a redundant check for empty subscription from `view_subscription.html` (thanks Rob for reporting!)
- Fixed a bug that caused errors displaying prices in secondary currencies if they prices had decimals in them.
  (Thanks Matthew for reporting and the fix!)
- Fixed team signup test if you have disabled signups. (Thanks Saif for reporting and proposing the fix!)
- Fixed a bug where links to the user dashboard were accidentally missing if you disabled user impersonation. (Thanks Simon for finding!)

### Upgrade notes

If you are using Docker in development you might need to move/copy your `.env.docker` file to `.env`
when you update your project.

*Mar 22, 2024*

## Version 2024.2.4

This is another minor hotfix release. Details:

- Fixed Google Cloud `Makefile` targets to use the built project's ID instead of the Pegasus development project's ID.
  (Thanks Daan for reporting!)
- Minor changes to `fly.toml` to fix errors on certain environments/versions. (Thanks Denis for reporting!)

*Feb 26, 2024*

## Version 2024.2.3

This is a hotfix release that restores the `deploy/` folder that was accidentally not
present on certain Google Cloud builds. *Thanks Yogesh for reporting!*

*Feb 24, 2024*

## Version 2024.2.2

This is a big Pegasus release with a new Github integration for creating and upgrading projects,
as well as an overhaul of the Google Cloud deployment support and many small updates and fixes from the community.

### New Github distribution model

You can now create and upgrade your projects directly on Github!
This is a much smoother experience than the previous zip file model (which is still available).
You can also connect an existing project to Github by following the [instructions here](upgrading.md#connecting-an-existing-project-to-github).

For more details see the screencast below,
and the updated [Getting Started](/getting-started.md) and [Upgrading](/upgrading.md) pages.

<div style="position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden; max-width: 100%; height: auto; margin-bottom: 1em;">
    <iframe src="https://www.youtube.com/embed/5PLO79rb--A" frameborder="0" allowfullscreen style="position: absolute; top: 0; left: 0; width: 100%; height: 100%;"></iframe>
</div>

A few code changes were needed to support this workflow (in particular, it's no longer possible to deliver files which
were in the `.gitignore` file):

- Pegasus no longer ships with a `.env` or `.env.docker` file and instead these must be created from `.env.example`.
- `.env.example` will now use Docker-based URLs for your Postgres and Redis database if you build with Docker enabled.
- Added `make setup-env` command to create your `.env.docker` file from `.env.example`.
- Updated the `make init` create your `.env.docker` file if it doesn't already exist.
- Updated the setup documentation to reflect the above changes.

### Overhauled Google Cloud Deployment

The Google Cloud deployment support has been completely overhauled and brought in line with other deployment plastforms.
These are the major changes (affecting Google Cloud deployments only):

- Updated the Google Cloud set up and [setup documentation](/deployment/google-cloud.md) based on the latest Google guide.
- Google Cloud now uses the same Dockerfile as other deployment options.
- Google Cloud now uses whitenoise for Static files, the same as other deployment options.
  It still uses Google Cloud Storage for media files.
- Added `Makefile` targets for Google Cloud options for building, pushing, and deploying.
- Deleted legacy `cloudmigrate.yaml` and `cloud_run.sh` files.
- Simplified the `prod-requirements.in` file.
- Fixed a bug where uvicorn was not correctly installed in production for async builds.
- Fixed various settings to work with the latest Google Cloud guides and best practices and simplified
  many of the other settings.
- Updated the Django storages set up for the latest Django 4.2 / 5.0 settings.

*Big thanks to Conrad, Rob, Troy, and Nathan for helping find and work through these changes.*

### Other Fixes

- Include djstripe webhook url endpoints if building with ecommerce and not subscriptions. (Thanks Emiliano for reporting!)
- Fixed a bug where a Social App in the database for a provider that was not configured would cause a crash.
- Fixed a bug where teams tests would fail if `ACCOUNT_SIGNUP_PASSWORD_ENTER_TWICE` was enabled. (thanks Saif for reporting and the fix!)
- Fixed a bug where social button icons would not work with manifest file storage.

### Other Changes

- Updated most `make` targets that run commands in Docker to not require the `web` container to be running.
  They will now spin up and remove temporary containers instead. (Thanks Artem for the suggestion!)
- Slightly improve styling of page that shows when there are social authentication errors (thanks Finbar for the contribution!)
- Make `USE_HTTPS_IN_ABSOLUTE_URLS` setting configurable via an environment variable.
- (Render only) Make the casing of booleans in `render.yaml` consistent.
- (Kamal only) Added more environment variable declarations to the default Kamal setup.
- (fly.io only) Overhauled the `fly.toml` deployment file to be consistent with fly's latest format and removed unnecessary parts.
- (Bulma only) updated the sign up form so that the password(s) come before the team name to be consistent with other CSS frameworks.
- (Bulma only) allow changing the email address you sign up from when accepting a team invitation.

### Documentation updates

- Overhauled [Google Cloud setup docs](deployment/google-cloud.md).
- Added Github instructions to [upgrading documentation](upgrading.md).
- Added Github instructions to [getting started](getting-started.md).

*Feb 21, 2024*

## Version 2024.2.1

This release fixes the OpenAI demo to be compatible with the latest Python library:

- Fix ChatGPT example to work with latest OpenAI API.
- Fix Dall-E example to work with latest OpenAI API.
- Update a Github action to latest version that was missed in the last release.

*Feb 5, 2024*

## Version 2024.2

This release adds Django 5.0 support, upgrades all dependencies to their latest versions,
and includes a handful of other changes.

Details:

- **Support Django 5.0.** Django 4.2 LTS should also work, but 5.0 will be the default version tested and used moving forwards.
- **Upgraded all Python dependencies to their latest versions.**
- **Upgraded all JavaScript dependencies to their latest versions.**
- Upgraded all Github actions to use latest versions.
- Simplified the process for building the front end API client with Docker (thanks Finbar for the pointer!).
  [Api client docs](apis.md#generating-the-api-client) have been updated as well.
- Update links to all Django documentation to link to the latest stable release, instead of a specific version.
- Fixed a schema warning for the user signups dashboard API.
- Update link to the guide to celery progress bars to the new blog post url.
- Simplified the process of setting up your ecommerce config to a new management command: `./manage.py bootstrap_ecommerce`.
  - Also refactored related code to be shared between ecommerce and subscriptions.
- Added `subscription_is_active` helper function, to remove duplicate code that was running the same check.
  Also updated [the documentation](payments.md#set-up-your-development-environment).
- Fixed a bug where if you marked a subscription to be canceled at the period end and weren't running webhooks,
  the subscription page would crash.
- Added `staff_member_required` decorator to all superuser-only views, to ensure that the user is also active,
  and staff. (Thanks Felipe for the suggestion!)
- Removed `tailwindcss/forms` plugin, which conflicted with some default DaisyUI form elements and wasn't needed.
  (thanks Artem and Alex for the suggestion!)
- Added better styling for the socialaccount connections page. (thanks Finbar for the contribution!)


*Feb 2, 2024*

## Version 2024.1.2

This is a minor/hotfix release that fixes a few issues related to fly.io deployments due to changes
in the default values used by fly.
It also updates the [fly documentation](/deployment/fly.md) to use the latest `flyctl` workflow for launching
new applications.

*Thanks to Naveed for reporting this.*

*January 15, 2024*


## Version 2024.1.1

This is a minor/hotfix release that disables self-hosting media files with Kamal deployment,
since this wasn't working well for production environments.

For now it is recommended to use S3 to host media files with Kamal, though this may change in the future.

To get the hotfix without upgrading you can just remove these two lines from your `deploy.yml` file:

```yaml
volumes:
  - "{your-app-slug}-media:/code/media"
```

*January 9, 2024*

## Version 2024.1

This is a minor/hotfix release with some small updates/fixes to the group chat UI and Kamal deployment:

- Fixes an issue where there were missing templates if you installed with asynchronous support and without OpenAI. (Thanks Jose for reporting!)
- Chat templates have now been moved to `web/chat` so it is clear they can be re-used across apps.
- Added tests that would have caught the above issue.
- Renamed `TestLoginRequiredView` to `TestProfileViews` to be more clear what it is doing.
- Bumped Django to the latest 4.2.9 bugfix release.
- Don't run Celery with gevent in Kamal deployments, since it is not necessary and not always installed. (Thanks Luis for reporting!)
- Use `settings_production` in Kamal deployments by default, instead of requiring manually setting the environment variable. (Thanks Luis for reporting!)

*January 3, 2024*

## Version 2023.12.1

This hotfix release changes the `multiarch` value in Kamal deploys to `true`, which fixes errors when
deploying from Mac / Windows machines to Linux servers (thanks Luis for reporting).
There was also an overhaul of the [Kamal docs](/deployment/kamal.md).

If you want the hotfix without upgrading, you can just set `multiarch: true` in the `builder` section of your `deploy.yml` file.

*December 20, 2023*

## Version 2023.12

The big update in this release is official support for deploying Pegasus apps onto any linux server.
This allows you to deploy Pegasus apps onto any VPS, like Linodes, Digital Ocean Droplets, or Amazon EC2 / Lightsail instances.
The deployment uses Docker containers, managed using [Kamal](https://kamal-deploy.org/).
You can deploy your entire application onto a single server (the default), multiple servers, or mix-and-match
between self-hosted and managed services (e.g. Amazon RDS) with an easily customizable configuration.

For more details, see the new [Pegasus Kamal documentation](/deployment/kamal).

Below are the complete release notes:

### Added

- **Added support for deploying to any linux server using the new Kamal deployment option. [Documentation](/deployment/kamal/).**
- **Added first-class support for "login with Github."**
- Added a basic `.dockerignore` file.
- Added an optional argument to `make npm-install` for installing individual packages, and added a `make npm-uninstall` target
  for uninstalling packages. (Thanks Gary for the suggestion/contribution!)

### Changed

- **Load `request.team` in `TeamsMiddleware` even if the user doesn't have access to the team if `team_slug` is passed to the view.
  Since authorization is done in the view decorators like `login_and_team_required` this should be safe,
  and makes it easier to create team views that don't require authentication.**
  - Also updated tests to reflect this new behavior.
- Only show social apps which have been created in the database on the sign up and login pages, and clean up/standardize how
  social app code/buttons are added.
  - Also switch social logins to use POST and remove `SOCIALACCOUNT_LOGIN_ON_GET = True` from settings.
- **Add a `customer` object to the `CustomUser` model when ecommerce is enabled, and re-use the same customers
  when a user makes multiple purchases.**
- Silence `dj-stripe` warning about Stripe keys being kept in settings. This is standard practice for Pegasus applications.
- Mock out JavaScript translations and remove translation-based views when building without translations enabled.
  This should slightly improve page-load times when not using translations.
- Explicitly set `DEBUG=False` in the Render production environment.
- Explicitly set default region on fly.io deployments.
- Changed postgres connection strings from `postgres://` to `postgresql://`. Either one works in Django, but only the
  latter works with sqlalchemy, so using it allows the same connection string to be used with both tools.
- **Upgraded everything to run Node 20 instead of Node 18.**
- **Upgraded the base Docker images from bullseye (Debian 11) to bookworm (Debian 12).**
- **Overhauled the production Docker setup to use a [multi-stage build](https://docs.docker.com/build/guide/multi-stage/).
  This should allow for faster build times (partial builds can run in parallel) as well as faster rebuild time,
  as more steps are able to be cached more often.**
- Profile picture media files are now deleted when the associated user is deleted.
- Default the `PORT` variable used by django in production deployments to `8000` if not specified in the environment.
- Changed django database engine from `django.db.backends.postgresql_psycopg2` to `django.db.backends.postgresql`
  (these behave the same, but the latter is now recommended).


### Fixed

- Fixed description of `dev-requirements.txt` to indicate it installs development, not production requirements. (Thanks Yngve for reporting!)
- Fixed 500 error when trying to accept an invitation that was already accepted.
- Removed duplicate DB lookups on invitation acceptance page.
- Set avatar image filenames to be randomly generated. This fixes an issue where, under certain media configurations,
  uploaded profile pictures with the same filename could "clobber" each other.
- Fix test failures when manifest storage was configured, by overriding the `STORAGES` setting in affected tests.
- Fixed 500 error when attempting to manage social app connections from the profile page, due to extending a deleted `allauth` template.
- Attempt to add more `INTERNAL_IPS` to `settings.py` when using Docker in development, to get Django debug toolbar to show up.
  (Thanks Artem for reporting/contributing!)
- Fixed issue with missing `$PORT` reference caused fly.io deployments to sometimes fail.

### Removed

- Removed migration to auto-create social apps in the database.
- Cleaned up template imports in a few places.

### Upgrade notes

The `TeamsMiddleware` change may change the access rules of views that were relying on the absence of `request.team`,
to control authorization, instead of using `login_and_team_required` or similar approaches.

The removed migration file (`/web/migrations/0002_create_allauth_providers.py`) should not be removed if you have
already run the migration on your environment. Keeping it in the repository won't do any harm.

*December 12, 2023*

## Version 2023.11.1

This is a hotfix release that fixes an issue with calling dj-stripe's `get_subscriber_model` utility when teams were
enabled by adding an `email` property to the `Team` object.
The `email` property was accidentally not added for certain build configurations.

You can manually apply this change by adding the following property to the `Team` model, in `apps/teams/models.py`:

```python
@property
def email(self):
    return self.membership_set.filter(role=roles.ROLE_ADMIN).first().user.email
```

Thanks to Charley and Emilien for reporting!

*Nov 10 2023*

## Version 2023.11

This is a hotfix release that fixes the Node.js docker installation according to
[these nodesource changes](https://github.com/nodesource/distributions#new-update-%EF%B8%8F).

You can also manually apply this change by replacing the current node installation steps with the following
code in your `Dockerfile.dev` and `Dockerfile.web`.

```
# install node/npm
RUN curl -fsSL https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key | \
    gpg --dearmor -o /usr/share/keyrings/nodesource.gpg

RUN echo \
  "deb [signed-by=/usr/share/keyrings/nodesource.gpg] https://deb.nodesource.com/node_18.x nodistro main" | \
  tee /etc/apt/sources.list.d/nodesource.list && \
    apt-get update && \
    apt-get install nodejs -yqq
```

Thanks to Finbar on Slack for reporting and suggesting the fix!

*November 1, 2023*

## Version 2023.10.1

This is a minor release addressing a few small issues raised in the `2023.10` release.

- **Upgrade docker containers from `python:3.11-buster` to `python:3.11-bullseye`.
  This change substantially improves performance on certain Mac builds.**
- Upgrade django to the latest 4.2.6 security release.
- Reduce db queries on the e-commerce home page.
- Don't show "buy" links on the e-commerce page if the products do not have a default price set.
- Change default value of `ALLOWED_HOSTS` to `["*"]` and allow overriding it via an environment variable.
- Clean up whitespace in `user_dashboard.html`.

*Oct 9, 2023*

## Version 2023.10

This is a major release with three big updates: Async/Websocket support, an E-Commerce application,
and an admin user dashboard.

### Async / Websocket support

Pegasus now supports asynchronous Django and Websockets.
Included is an example group chat application that leverages these capabilities.
Watch the video below for more details, or see the new [async / websocket documentation](async.md):

<div style="position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden; max-width: 100%; height: auto; margin-bottom: 1em;">
    <iframe src="https://www.youtube.com/embed/J1hma14whz4" frameborder="0" allowfullscreen style="position: absolute; top: 0; left: 0; width: 100%; height: 100%;"></iframe>
</div>

### E-Commerce (One-Time Payment) Application

The previous Stripe Payments example has been converted into a full-blown E-Commerce store.
You can manage your products in Stripe and sync them to your application with a few lines of configuration.
Customers can purchase specific items and everything is linked to your Stripe dashboard.

Watch the video for more details, or see the new [E-Commerce documentation](payments.md)::

<div style="position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden; max-width: 100%; height: auto; margin-bottom: 1em;">
    <iframe src="https://www.youtube.com/embed/S4LlQtGD1jc" frameborder="0" allowfullscreen style="position: absolute; top: 0; left: 0; width: 100%; height: 100%;"></iframe>
</div>

### Admin Dashboard

An admin-facing dashboard has been added.
The dashboard lets you see User sign ups to your application over time, and is filterable by date range.
It's a useful tool to see how your app is growing, and a good launching point for building out more
dashboard capabilities.

Screenshot:

![Admin Dashboard](/images/user-dashboard.png)

Below are the complete release notes including several other substantial changes
and supporting work for the above.

### Added

- **Added asynchronous and websocket support via a new build option.** [Documentation](async.md).
  - **Related: Added the group chat example application if you enable asynchronous support.**
  - Related: Added `websocket_reverse` and `websocket_absolute_url` helper functions and tests.
  - Related: Added `.pg-message-sender` helper CSS class.
  - Related: If building with async your app will use `asgi` instead of `wsgi`.
- **Added the E-Commerce example application via a new build option.** [Documentation](payments.md).
- **Added the admin dashboard.**
  - Related: Added an admin-only user signup API
- Added a `pg-link` helper class to style links (especially on Tailwind and Material builds).
  Also applied this style to a few places.
- Added basic tests for some of the example views.
- Added an example of customizing existing DaisyUI themes to the [Tailwind docs](css/tailwind.md).
- Added `absolute_url` template tag for generating full URLs in e.g. email templates, and added tests for it.

### Changed

- **Added the `feature_gate_check` and `get_feature_gate_check` helper functions,
  for more fine-grained control of feature gate checking.**
  See the updated [feature-gating documentation](subscriptions.md#feature-gating) for more information.
  - Related: Modified the `active_subscription_required` decorator to use this function.
- Reduced number of DB queries made when provisioning a subscription.
- Made subscription provisioning an atomic action to reduce race conditions between Stripe Checkout callbacks and webhooks.
- Stripe subscription webhooks now explicitly only process checkout sessions that were created by the
  subscriptions application. This was done to enable apps to handle both e-commerce and subscription payments,
  and is handled by adding (and checking) a "source" value on the checkout
  session metadata.
- Created a `TestLoginRequiredViewBase` base test class, to test logged-in views, and updated existing tests to use it.
- Removed uppercase characters from `TestLoginRequiredView` test methods.
- Upgraded Chart.js to the latest version, and moved it to be installed from NPM instead of a CDN.
- Changed the example charts to use the NPM-installed Chart.js.
- Moved `get_stripe_module` to `apps.utils.billing` so it can be used by the e-commerce app and subscriptions.
- **Upgraded `django-allauth` to latest version (0.57.0).**
  - Related: Added a migration to create default `SocialApp` models for all enabled providers, otherwise the signup and login pages crash.
  - Related: Blank out help-text on the sign up form's password input.
  - Related: Added `allauth.account.middleware.AccountMiddleware` to `MIDDLEWARE`
  - Related: Fixed links to allauth docs in the generated README file.
- Upgraded Django to the latest 4.2.5 security release.
- Set `DEBUG=False` when running `collectstatic` on production Docker builds.
- Added some fields to the default `CustomUserSerializer`.
- Added `created_at` field to chat message admin list display / list filter.
- Removed some unused imports from subscription views.
- Refactored chat message list into a standalone template.

### Fixed

- Fixed absolute paths to Android-specific favicons to be relative. (Thanks Alexander for reporting!)
- Fixed issue where mobile menu content sometimes did not appear in front of page content on Tailwind builds.
- The object lifecycle home example view now requires login.
- Fixed issues with calling dj-stripe's `get_subscriber_model` utility when teams were enabled,
  by adding an `email` property to the `Team` object, and implementing `DJSTRIPE_SUBSCRIBER_MODEL_REQUEST_CALLBACK`,
  and added a test case that confirms this works moving forwards.
- Fix styling of date inputs on all CSS frameworks.
- Fixed tab highlighting of the "impersonate a user" navigation on the Bootstrap Material theme.

### Removed

- **Removed the previous Payments example.** Apps should refer to the new E-Commerce app to use one-time payments.
  - Related: Removed all migrations from the example app, which now has no models.
  - Related: Removed the previous Payments documentation.
- Removed the "removing Stripe" cookbook from the documentation. Stripe is no longer included
  unless you build with the E-Commerce example or Subscriptions enabled.

### Upgrade Notes

- To migrate an existing application to use asynchronous / websockets, you will have to set `DEBUG` in your
  production *environment* (not `settings_production.py`). More information in the [async documentation](async.md).

*Oct 4, 2023*

## Version 2023.9.2

This is a hotfix release that bumps `django-environ` from `0.11.1` to `0.11.2` which
fixes an issue with "$" in certain situations in environment variables. [details](https://github.com/joke2k/django-environ/issues/490)

Thanks Geoff for reporting!

*Sep 22, 2023*


## Version 2023.9.1

This is a hotfix release with a few small fixes and updates:

- Updated the version of `celery[redis]` to `5.3.4` since `5.3.3` was inexplicably deleted from PyPI.
- Fixed a crashing issue on the two-factor auth configuration pages caused by the recent `allauth-2fa` update.
  (Thanks Matthew for reporting and suggesting the fix)
- Added tests for a few logged-in views, including one that would have caught the two-factor issue above.
- Properly show errors if you enter the wrong two-factor token when trying to remove two-factor auth from your account.

*Sep 18, 2023*

## Version 2023.9

2023.9 has two main updates: Stripe embedded pricing table support, and a substantially improved Wagtail experience.

### Stripe Embedded Pricing Table

This release adds support for [Stripe's embedded pricing table](https://stripe.com/docs/payments/checkout/pricing-table),
which is a far simpler alternative to creating a pricing page than doing it natively in your application.

The pricing table was added as a new build configuration option.
Once enabled, the previous subscription UI will be replaced by Stripe's UI, and all changes to the pricing table
can be made directly within Stripe.

The pricing table option opens up a number of new billing options, including multi-currency support
and free trials, though it does not support per-unit billing very well.
For complete details, see the [updated subscription documentation](subscriptions.md).

### Wagtail Enhancements

The big Wagtail change is that most content pages now use Wagtail's `StreamField` instead of `RichTextField`.
This allows you to stitch together arbitrary blocks of content in all your pages, instead of being
forced into a single rich text model. It also enables re-use of individual structured components.
You can [read more about Wagtail `StreamField` here](https://docs.wagtail.org/en/v5.1.1/topics/streamfield.html).

There were also several smaller improvements.

*If you're upgrading from a previous version, see the upgrading notes below.*

### Complete release notes

#### Added

- **Stripe: Added embedded pricing table support, via a new build option.**
- Wagtail: Added `social_image` field to all content models (using `BaseContentPage`), so you can define a custom image to use for `og:meta` tags for any
  individual page.
- Wagtail: Added a `CaptionBlock`, for captioning images or code, which you can use as a reference to add additional block types to your app.
- Wagtail: Added a migration to port previous `RichTextField`s to `StreamField`s.
- Added tests for `get_image_url` template tag.

### Changed

- **Wagtail: Migrated `ContentPage.body` and `BlogPage.body` to use `StreamField` instead of `RichTextField`.
  This provides much more flexibility in laying out your pages and working with many different section types.**
- Wagtail: All content models now extend from `BaseContentPage` so that you can add fields that should be shared among all
  your different types of content.
- Wagtail: Updated the `bootstrap_content` management command to be compatible with the new structure.
- **Upgraded nearly all Python packages to their latest versions.**
  `django-allauth` was not upgraded, due to it having a large release just a few days ago.
- **Upgraded all JavaScript packages to their latest versions.**
- **Subscriptions: official support for multiple currencies ([docs](subscriptions.md#supporting-multiple-currencies)) (Stripe pricing-table only)**
- **Subscriptions: official support for free trials ([docs](subscriptions.md#free-trials))**
- **Subscriptions: Overhauled the [Subscriptions documentation](subscriptions.md) to make it clearer, and add the new pricing UI setting.**
- Subscriptions: Moved the `checkout_success` endpoint to be a global `confirm` endpoint instead
  of a team-specific endpoint.
- Subscriptions: Improved display of subscription price line items when using metered billing.

### Fixed

- Subscriptions: Fixed bug that caused trialing subscriptions to not be counted as active.
- Subscriptions: Show the correct currency in subscription details page if using Stripe's [multi-currency support](https://stripe.com/docs/payments/checkout/present-local-currencies?platform=multi-currency-prices).
  (Thanks Mario for reporting.)
- Fixed bug in `get_image_url` template tag that prevented it from properly resolving relative media URLs.
  Also added tests for this case.
- Updated the `bootstrap_subscriptions` management command to be compatible with the latest version of `dj-stripe`.
- Fixed a bug where the active products API would always crash if you had not defined `ACTIVE_PRODUCTS`

### Removed

- Removed no-longer-supported `DJSTRIPE_USE_NATIVE_JSONFIELD` setting.

### Upgrade notes

It is recommended to read through [the dj-stripe 2.8 release notes](https://github.com/dj-stripe/dj-stripe/releases/tag/2.8.0)
to confirm you aren't affected by any backwards-incompatible changes.

Customers switching to the new Stripe embedded pricing table will need to move any product
information (including names, descriptions and feature lists) from `metadata.py` into their
Stripe product and pricing page configuration.

Customers upgrading from existing wagtail installations that have been customized may need to do additional
work to update their content models to StreamFields. Look at the `0002_convert_stream_fields.py` migration
and apply the same pattern to any other fields you want to migrate.

*September 1, 2023*

## Version 2023.8.2

This is another bugfix release that fixes docker-based deployments (Digital Ocean, Heroku Docker, and fly.io).
To get the fix you don't need to upgrade, just change the node version in your `Dockerfile.web` from 16 to 18.
The updated line should look like this:


```
RUN \
  echo "deb https://deb.nodesource.com/node_18.x buster main" > /etc/apt/sources.list.d/nodesource.list && \
```

Thanks Matthias and Alexander for reporting this.

*Aug 30, 2023*

## Version 2023.8.1

This is a bugfix release that fixes deployment to render.

To get the fix you don't need to upgrade, just add these two lines to your `envVars` section in `render.yaml`
to explicitly bump the node version used from 14 to 18.

```yaml
      - key: NODE_VERSION
        value: 18.17.1
```

Thanks Greg and Michiel for the bug report and suggested fix.

*Aug 21, 2023*

## Version 2023.8

This release adds official support for three marketing email platforms (Mailchimp, ConvertKit, and Email Octopus),
adds dark mode on Tailwind builds, and has the usual smaller updates and fixes.

### Added

- **First class support for marketing email lists.** You can now select a platform (Mailchimp, ConvertKit, Email Octopus, or none),
  and your build will be customized for that platform, including settings/environment variables, and automatically subscribing
  new sign ups to your email list (if properly configured). See the updated [mailing list documentation](configuration.md#mailing-list)
  for more details.
- Added a management command to send test emails: `./manage.py send_test_email cory@example.com`.
  Useful when troubleshooting/changing how your server sends email.
- **Added dark mode support for TailwindCSS builds.** Your app should automatically use dark mode if the user's
  browser is configured for it. Components that weren't properly styled for dark mode now are.
  If you spot any issues please report them!
- The `get_next_unique_slug` helper function can now take filter arguments, so you can have unique fields dependent on other fields
  (for example, if you want to have slugs which are unique per team).
- Added tests for `get_next_unique_slug` (including testing the new functionality).
- Added view tests for the signup process with various edge-cases around team names.Docker
- Added a `Makefile` target, and documentation for rebuilding the API client with Docker.
  [Documentation](apis.md#generating-the-api-client) (Big thanks to Finbar for helping on this)

### Fixed

- Removed empty JavaScript files in certain builds that were causing `npm type-check` to fail. (Thanks George for reporting!)
- Only try to log mailing list errors to Sentry if building with Sentry enabled.
- Fixed a bug in `get_next_unique_slug` that was failed if you passed in a custom `slug_field_name`.
  Also added a test that would have caught it.
- Fixed a bug where unicode team names were creating teams with an empty slug, which was causing a crash on logging in.
- Fixed a typo in the `Makefile` (thanks Arno for reporting!)
- [Documentation] Fixed issue in the digital ocean setup docs that was accidentally resulting in the creation of two Postgres databases,
  one of which was unused. (Thanks Thomas for reporting!
- Removed links to user profile and signout views from the app navigation if there is no signed in user.

### Changed

- **Upgraded all JavaScript packages to their latest (as of late July) versions.**
- Use the project's slug in the `package.json` name instead of "pegasus".
- Changed Twitter change social card format to `summary_large_image`
- In the ChatGPT functionality, new chats are now not created until the first message is sent to them (HTMX only).
  This prevents empty chats from being created.
- Improved link styling of chats on tailwind builds
- Changed "loading-dots" CSS class to "add-loading-dots", to prevent conflict with DaisyUI class with the same name.
- Users' chats are now sorted by last modification time, descending.
- Profile picture validation now includes backend file-type checks, to avoid users uploading incorrect/malicious profile pictures.
  (Thanks Edward for reporting)
- Stopped explicitly specifying a `platform: ` in `docker-compose.yml`, and instead always fall back to the OS's default platform.
- Added `.jsx`, `.ts`, and `.tsx` as content roots in `tailwind.config.js`.


### Upgrading

If you were previously using the mailchimp email functionality, you will need to edit your project
and select "Mailchimp" under "Email Marketing Platform" to keep using it.

*Aug 8, 2023*

## Version 2023.7

This is a large maintenance release with many improvements and a few new features.

### Added

- **Expanded the built-in timezone support (if building with internationalization).** This includes:
  - A new timezone setting on the User model/profile.
  - A middleware that sets and unsets the timezone based on the user's setting.
  - A built-in list of default timezones.
- **Added the option to remove compiled static files at Pegasus build time.**
  If checked, your Pegasus build will not include any static files, and they will be added to the `.gitignore` file.
  This is useful to check after you have set up static file builds as part of a CI/CD pipeline.
  [More here](front-end.md#long-term-best-practices).
- **Added optional support for enabling Django's [admin docs](https://docs.djangoproject.com/en/4.2/ref/contrib/admin/admindocs/#module-django.contrib.admindocs)
  via a new project setting.**
- Added improved Docker support for ARM / Mac M2 architectures, via a new project build option.
  This should improve the performance using Docker for affected OS's.

### Changed

- **Removed all React dependencies and supporting code when building without React and without the built-in examples.**
- Made order of example navigation and example homepage cards consistent.
- Placed HTMX object lifecycle demo before the Vue one.
- Better styling of terms link in signup forms (Tailwind builds only)
- Moved `page_js` block to the bottom of the `<body>` in the base template.
  This allows using other imported libraries (e.g. site-bootstrap.js) in inherited templates. (Thanks Finbar for suggesting)
- Switched `UserLocaleMiddleware` to use the "new" style of Django middleware, using `__call__` instead of `process_request`
  and `process_response`.
- Bumped Django version to the latest 4.2.3 security release.

### Fixed

- **Refactored how custom components are added to Tailwind to follow the official guidance on
  [build-time imports](https://tailwindcss.com/docs/using-with-preprocessors#build-time-imports).**
  This fixes an issue where multiple style declarations of some classes were included, causing some CSS overrides
  to not work out of the box. It also results in improved CSS compile times and reduced output file sizes.
  Additionally, some tailwind styles were moved out of the main `site-tailwind.css` file and into other imported files.
  (Thanks Tyler for reporting and suggesting the fix!)
- More gracefully handle when a Stripe subscription is deleted (usually in test mode), by logging an
  error and clearing it from the associated user/team object.
- Added try/catch around Docker hostname setting for debug toolbar, which failed when running outside Docker
  on some OS's. (Thanks Geoff for the reporting/fixing)
- Moved inline comments in `.env.example` that failed on some environments. (Thanks Geoff for reporting/fixing)
- Stopped running `collectstatic` while building Docker containers on Google Cloud Run deployments,
  since the static files are managed outside the container for that platform. (Thanks Alexander for reporting)

*Jul 9, 2023*

## Version 2023.6.1

This is a hotfix release that fixes a bug in the new Alpine.js form/attrs released in 2023.6 for some CSS Frameworks.

*Jun 17, 2023*

## Version 2023.6

This is a minor release with some form updates and a bugfix for material Bootstrap builds.

### Added
- Added the ability to specify attrs on form fields, along with special helper parsing for Alpine.js forms.
- Added a forms example using Alpine.js to demonstrate Alpine.js form functionality, including hiding/showing
  a field based on the value of another field, rendering field values in labels, and changing the style of
  a field based on its value.
  The example is available at [http://localhost:8000/pegasus/forms/alpine/](http://localhost:8000/pegasus/forms/alpine/).
- Added [documentation on forms](forms.md) in Pegasus.
- Added `.pg-bg-danger` and `.pg-bg-success` helper classes for setting success/danger background colors.

### Fixed

- Fixed some styling issues with Bootstrap 5.3 and the material theme.
- Fixed the documentation for how to customize variables when using Bootstrap.

*Jun 12, 2023*

## Version 2023.5.1

This is a small hotfix release, that includes a fix to a build error when using Bootstrap 5.3 (release yesterday).

Details:

- **Fix build errors when using Bootstrap 5.3** (Thanks Allan for reporting/fixing).
- Remove `WAGTAILADMIN_BASE_URL` setting if not building with wagtail.
- Remove some accidentally included tailwind CSS files when not building with tailwind.
- Update the default copyright year in site footer to 2023 if JavaScript was not enabled.
- Add `date_joined` to user admin list display / filter.
- Cropped message content in Chat admin page.
- Added a note to README about configuring Redis if not using Docker (thanks Chris for suggesting)

*May 31, 2023*

## Version 2023.5

The big feature this release is a major enhancement to the OpenAI ChatGPT integration.
This release also upgrades to Django 4.2 LTS and comes with the usual fixes and improvements.

### Chat UI Overhaul

Here's a 3-minute demo video of the new ChatGPT functionality:

<div style="position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden; max-width: 100%; height: auto; margin-bottom: 1em;">
    <iframe src="https://www.youtube.com/embed/dT4kfQCCq1g" frameborder="0" allowfullscreen style="position: absolute; top: 0; left: 0; width: 100%; height: 100%;"></iframe>
</div>

**Details:**

- Made the Chat UI interactive using HTMX/React (depending on your project configration) and Celery, and greatly improved
  chat styling.
- Moved chat example to a new chat app.
- Added a chat history model and use it so that chats now have memory.
- Added a chat list view unique to each user.
- Added a `render_markdown` template filter, for converting Markdown to HTML in a template
- Added serializers and admin for chat models.
- Automatically set chat names based on the contents of the first message.
- Add ability to set which chat model you use with the new `OPENAI_MODEL` setting.
- Add `chat.css`/`chat.sass` files for framework-specific chat styling, with many helper UI classes related to the chat UI.
- Removed previous Chat UI, including `PromptForm` class, and related view/template.

### Other updates in this release

#### Added

- Added `makemigrations` and `migrate` steps to README on sqlite builds (thanks Patrick for suggesting).
- Added `{% block top_nav_app %}` to `app_base.html` to the material theme, so that the app nav can be overridden in sub-templates
- Added make targets for running black (`make black`), isort (`make isort`), and both (`make format`)
- Added `devtool: "eval-cheap-source-map"` to `webpack.config.js` to prevent source map warnings on Chrome. (Thanks Brett for suggesting)

#### Changed

- **Upgraded Django to 4.2 LTS, Wagtail to 5.0, and a handful of other Python packages to their latest versions.**
- Switched from deprecated `DEFAULT_FILE_STORAGE` and `STATICFILES_STORAGE` settings to `STORAGES` setting (added in Django 4.2).
- Set `EMAIL_SUBJECT_PREFIX` to be your app's name. This will be prepended to server admin emails.
- Added a 5MB file size limitation to profile pictures, and delete old profile pictures when new ones are uploaded. (Thanks Jonathan for the suggestions!)

#### Fixed

- Fixed a bug where the app would crash if people signed up but already had an account, when teams were enabled.
- Fixed crashing issues with running `./manage.py bootstrap_content` (and, resultingly, `make init`) multiple times in a row.
- Removed accidentally included `team_nav.html` file on some CSS frameworks when teams was not enabled.
- Fixed a bug where the `teams/manage/<path:path>` url route was accidentally included in HTMX builds and not included in React builds.
- Fixed a bug where profile picture upload styling wasn't applied on some CSS frameworks.

### Removed

- Removed deprecated `USE_L10N` setting.

*May 18, 2023*

## Version 2023.4.2

Another small hotfix release:

- Fixes crash on `robots.txt` if you built without wagtail.
- Added tests that would have caught the above issue (and also check other important pages).
- Removed unused `TermsSignupForm` when building with teams enabled.

*Apr 25, 2023*

## Version 2023.4.1

This release fixes two bugs:

- Fixes crash on sign up under certain conditions when teams was not enabled.
- Fixes crash when saving user profile data when API keys were not enabled.

Thanks to Simon for reporting these!

*April 21, 2023*

## Version 2023.4

This is a large maintenance release with many upgrades, cleanups, and a few small fixes.
The biggest changes are upgrading the default Python version to 3.11, Node version to 18,
and Docker compose version to 2.

### Python 3.11 update

This release makes Python 3.11 the default supported version for everything in Pegasus.
Details:

- **Changed default Python version to 3.11.**
  Older version of Python (back to 3.8) are still expected to work, but are no longer actively tested.
- Updated references in README and docs to use 3.11 everywhere.
- Updated development Docker image to use 3.11.
- Updated all deploy targets to default to 3.11.
- Updated `black` and `isort` configs to 3.11.
- Updated Github Actions to run tests on 3.11 only. Older versions can still be added back manually.
- **Upgraded most Python packages to latest compatible Python 3.11 versions.**
  Django was not upgraded to 4.2, because Wagtail has not released support for it yet.
  If you aren't using Wagtail, you can upgrade to 4.2 now with no known issues.

### Node 18 update

This release makes Node 18 the default supported version for everything in Pegasus.
Details:

- **Changed default Node version to 18**.
  Node 16 is still expected to work, but is not actively tested.
- Updated references in README and docs to use 18 everywhere.
- Updated development Docker image to use 18.
- Updated all deploy targets to default to 18.
- Updated Github Actions to run tests on 18 and 19 only. Older versions can be manually added back.
- **Upgraded all node packages to their latest versions.**

### Docker compose update

This release switches Docker compose to use version 2. Version 1 will be removed from Docker in a few months.
More details in the [Docker docs](https://docs.docker.com/compose/compose-v2/).

Details:

- Switched all instances of `docker-compose` (v1) to `docker compose` (v2) in the `Makefile`.
- Update all documentation to use `docker compose` instead of `docker-compose`


### Requirements update

This release updates the Python requirements files (again).
Apologies for the iteration on this---trying to find the best long-term workflow and hopefully this is it.

- The `requirements/dev-requirements.txt` (and `.in`) file no longer includes everything in `requirements/requirements.txt`.
  It now only has the requirements used *only* in development.
  And is (still) constrained to use the same requirements as `requirements/requirements.txt` if any duplicate packages are
  included, as [described here](https://pip-tools.readthedocs.io/en/latest/#workflow-for-layered-requirements).
- Added a `dev-requirements.txt` file in the root of the project to install both normal and dev-requirements.
- Updated usages of dev-requirements to use the new system, e.g. in Github Actions.

### Other changes

Smaller updates in this release are below.

#### Added

- **You can now automatically remove the teams example. Uncheck "include Teams example" in your project settings.**
- Added an "I agree to terms" checkbox on sign up for all CSS frameworks.
- Added link to impersonate a user to the app navigation on tailwind builds.
- Added a basic `robots.txt` file that disables crawling on the admin and wagtail admin sites. (Thanks Alex for suggesting)
- Added `OPENAI_API_KEY` to `.env` file if building with OpenAI examples enabled.

#### Changed

- Switched template setting to use `loaders` instead of `APP_DIRS` and disable template caching in development
  (thanks Michael for suggesting)
- Add apps directory to places Tailwind looks for templates, so that any CSS classes defined there are properly applied.

#### Fixed

- Fixed a bug where `./manage.py bootstrap_content` didn't work if you didn't have translations enabled.
- Fixed a bug where `black` and `isort` occasionally conflicted on import styles.
- Changed a few single-quotes strings in commented code to use double-quotes to match black styling.
- Added a missing trailing slash in a teams url.
- Added a default empty string to `AWS_ACCESS_KEY_ID` in `settings.py`
  (this avoids potential crashes running `collectstatic` if it wasn't set in the environment).
- Fixed a bug where custom form classes were not applied to input fields on Tailwind. (thanks Lars for reporting)
- Always include `"allauth.socialaccount"` in `INSTALLED_APPS`,
  otherwise deleting users fails. (thanks Jonathan for reporting)

#### Removed

- Removed internal subscriptions API endpoints from the generated API documentation and API clients.
  If you'd like to keep these, you can remove the `exclude=True` line from the `extend_schema` declaration
  in `subscriptions/views/api_views.py`, and then [rebuild the API client](apis.md#generating-the-api-client).

#### Documentation

- Overhauled the documentation on working with [virtual environments](python.md) and made
  `venv` the default recommendation over `virtualenv`.

### Upgrading / breaking changes

- If you don't have Docker compose V2 installed you will need to install it to continue using the `Makefile`.
  This is recommended, since V1 is being removed from Docker Desktop soon.
- There are no known breaking changes related to the Python and Node upgrades, but it is recommended to upgrade
  your projects if you haven't already.
  You may need to [rebuild your Python requirements](python.md#working-with-requirements)
  on older versions to get backports packages.


*April 20, 2023*

## Version 2023.3.5

A hotfix release, which fixes an issue introduced in `2023.3` where alpine.js was not properly
included in the base template if you built with React instead of HTMX.

This resulted in the subscription selection UI not appearing properly.

*March 25, 2023*

## Version 2023.3.4

Another minor release with a few small fixes.

### Changed

- Add `restart: unless-stopped` to web container docker-compose config, so that the web process still restarts
  on things like syntax errors. (Thanks Moritz for suggesting)
- Code formatting checks in Github Actions now run on the entire codebase instead of just the diffs in pull requests.

### Fixed

- Upgraded `cryptography` to version `39.0.2` which patches some high-severity vulnerabilities. (Thanks Michael for reporting)
- Fixed an issue with `boto3` accidentally not being included in requirements files when you enabled S3 media. (Thanks Elliott for reporting)
- Fixed an issue caused by [this issue behavior in Github Actions](https://github.com/actions/runner/issues/1189)
  that always caused code formatting checks to fail. (Thanks Elliott for reporting)

*March 22, 2023*

## Version 2023.3.3

The main feature in this minor version is an OpenAI demo,
showing how you can quickly integrate ChatGPT and DALL-E 2 into your Pegasus apps.

Here's a 3-minute demo video:

<div style="position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden; max-width: 100%; height: auto; margin-bottom: 1em;">
    <iframe src="https://www.youtube.com/embed/7DnIdEV2ygY" frameborder="0" allowfullscreen style="position: absolute; top: 0; left: 0; width: 100%; height: 100%;"></iframe>
</div>

There are also some small fixes (details below).

### Added

- ChatGPT and DALL-E 2 demos. To use these you need to check the "OpenAI demos" box in your project settings.
- You can now disable translations when building, which removes locale files and related code.

### Changed

- `dev-requirements.in` and `prod-requirements.in` now constrain package versions to those included in `requirements.in`.
  This should prevent library version conflicts between files.
- Remove `apps.utils.slug` and related test code if not building with teams enabled.
- Remove entire JavaScript API client if not building with any APIs enabled.
- Remove `storage_backends` if not building with S3 media support.

## Fixed

- Removed extraneous packages that were accidentally be included in `dev-requirements.txt` even if you
  had built without certain features enabled.
  If you saw a big diff after running `pip-compile requirements/dev-requirements.in` on a recent build,
  this was why.
- Made all package versions consistent between `requirements.txt` and `dev-requirements.txt`


*March 14, 2023*

## Version 2023.3.2

This release fixes two small bugs (introduced in `2023.3`):

- Fixed a syntax error in `render.yaml` that caused deployments to fail. Also removed unnecessary variables
  from that file if you weren't using subscriptions or sentry.
- Fixed an issue in the code formatting CI setup that caused the formatting checks to fail on commits made
  directly to a branch. Now formatting CI is only run on pull requests.

*March 6, 2023*

## Version 2023.3.1

This release fixes a bug (introduced in `2022.12`) that caused the `AUTH_PASSWORD_VALIDATORS` setting to be ignored
when building with teams enabled.

You can also manually patch this bug with the following change:

In `apps/teams/forms.py` replace this line:

```python
        cleaned_data = super(SignupForm, self).clean()
```

with

```python
        cleaned_data = super().clean()
```

Apologies to any affected users! Will be adding a test for this in a future release.

*March 5, 2023*

## Version 2023.3

This release includes several new features as well a lot of maintenance work.
These are the biggest changes:

### Code formatting

Pegasus will (optionally) now auto-format your Python code using [black](https://black.readthedocs.io).
In addition to formatting, Pegasus now ships with [pre-commit hooks](https://pre-commit.com)---which you
can install to ensure your code matches the expected format---and adds format checks to your Github actions CI.
Much more detail can be found in the new [code formatting docs](code-structure.md#code-formatting).
This option is enabled by default for new projects, and it's recommended that all existing Pegasus projects upgrade to this format,
as it will make future merges/upgrades much easier.
Guidance on upgrading [can be found here](cookbooks.md#migrating-to-auto-formatted-code).

### S3 production media support

You can now use S3 to store your project's production media with just a few lines of configuration.
To use S3, enable the "Use S3 for storing public media files" option in your project's configuration, and then
follow the new [S3 media documentation](configuration.md#setting-up-s3-media-storage).

### Django debug toolbar

Pegasus now (optionally) ships with the popular [django-debug-toolbar](https://github.com/jazzband/django-debug-toolbar) package.
You can enable this option in your project configuration, and it will be enabled by default in development
environments and turned off in production. More details in [the documentation](configuration.md#django-debug-toolbar).

### Alpine.js support

The [Alpine.js](https://alpinejs.dev/) library has been officially added as a dependency to all Pegasus builds.
Using Alpine allows replacing large amounts of custom JavaScript with a small amount of markup.
The subscriptions UI was updated to use Alpine, and more features will move to Alpine in the future.

*Below is the complete changelog for this release:*

### Added

- **Added pre-commit/black support, as described above and in the [code formatting docs](code-structure.md#code-formatting)**
- **Added django-debug-toolbar, as described above and in [the `debug-toolbar` documentation](configuration.md#django-debug-toolbar)**
- **Added S3 media support, as described above and in the [S3 media documentation](configuration.md#setting-up-s3-media-storage).**
- **Added Alpine.js as a top-level JavaScript dependency, included on all pages.**
- **Added `dev-requirements.in` and `dev-requirements.txt`, for requirements that should only be installed in development
  (e.g. `pip-tools`, `debug-toolbar`, `black`, etc.**
- Added health checks to Docker postgres and redis, to ensure they are ready before other containers start. (thanks Moritz for suggesting!)
- Added a `make npm-dev` command to build front end for development in Docker.
- Added a wrapping `meta` block to `base.html` to make overriding the page-level metadata more flexible.
  Wagtail blog post pages now use this to override the page title and description for social sharing.
- Added `.direnv` and `.envrc` files to `.gitignore`
- Added global `[x-cloak]` style to hide elements in Alpine.

### Changed

- **Migrated subscription selection flow from JavaScript to Alpine.js** and deleted a lot of custom JavaScript
  code that was no longer necessary as a result.
- Updated `ProductWithMetadata` serialization format remove monthly/annual/default prices,
  and add a dictionary of prices based on billing interval.
- As a result, **Pegasus now supports more than two billing intervals (you can now add any of Annual / Monthly / Weekly / Daily)**
- Migrated help text under the billing interval selector to the `PlanIntervalMetadata` helper class and removed front-end styling.
- Added `payment_amount` field to the Product/Price API serializer.
- **Removed stripe packages, dependencies, and all related code/styles if you build without subscriptions and without examples.**
  (thanks Brett for suggesting!)
- Moved stripe `card_element.html` component to `pegasus/examples/payments/components/card_element.html`, and only
  include it if you build with examples.
- Upgraded generated API client to version 6.4.0, and regenerated the API client.
- Upgraded django to 4.1.7 and celery-progress to 0.2.
- Added `SOCIALACCOUNT_LOGIN_ON_GET = True` to `settings.py`.
  This removes the extra confirmation page for social sign ups, improving the UX, though does
  open up a minor security risk [outlined here](https://github.com/pennersr/django-allauth/blob/master/ChangeLog.rst#security-notice-1).
  Remove this line if you prefer to keep the extra page.
- Saving a user profile now shows a confirmation message. (thanks Viktor for suggesting!)
- `make upgrade` now rebuilds your `requirements.txt` files and your front end. (thanks Brett for suggesting!)
- `STRIPE_LIVE_MODE` is now automatically set to `True` in Render deployments. (Thanks Adrian for suggesting!)
- Regenerated translation files for latest code changes.


### Fixed

- Removed "https" prefix from fly.io host checks, which caused them to fail.
- Fixed the url in the "Add a Password" link on the user's profile to go to the set password page.
  This link is only visible if the user signs up via social auth. (Thanks Blake for reporting)
- Added a workaround for an allauth bug that causes occasional 500 errors when users tried to sign in
  with a social account that was already tied to an existing email address, by using a `CustomSocialSignupForm`.
- [Details here](https://github.com/pennersr/django-allauth/blob/master/ChangeLog.rst#backwards-incompatible-changes-).
  (Thanks Simon for finding and fixing!)
- Fixed issue with `make npm-type-check` not being available if Wagtail wasn't enabled.
- Stop logging errors adding people to the mailing list if they were already on it.
- Improved styling of documentation link when subscriptions were improperly configured dn Tailwind builds.

### Removed

- Removed helper functions on `ProductWithMetadata` related to monthly/annual pricing (e.g. `monthly_price`).
- Removed the no-longer-used `get_payment_metadata_from_request` helper function.
- Removed the no-longer-used `catch_stripe_errors` decorator.
- Removed legacy styling markup from subscription details page. (thanks Viktor for reporting!)

### Documentation

- **Added write up about [the front end files](front-end.md#providing-site-wide-javascript).**
- **Added write up about managing [test vs live Stripe products](subscriptions.md#stripe-in-production)**
- **Improved the [internationalization/translation docs](internationalization.md).**
- **Added [a cookbook for how to enable auto-formatting on your existing project](cookbooks.md#migrating-to-auto-formatted-code).**

*March 3, 2023*

## Version 2023.2

This release was driven by feedback from the Pegasus community.
It includes many library upgrades (including updating to the latest Django),
the ability to build without the Pegasus examples, and small changes and fixes.

### Added

- **You can now build Pegasus projects without the built-in examples.**
- More type hints to return values in subscriptions module.
- `SubscriptionWrapper` object now has a `products` property to get associated Stripe products.

### Changed

- **Upgraded all Python libraries to their latest versions (including Django to 4.1.6)**
- **Upgraded all JavaScript libraries to their latest versions.**
- **Update HTMX installation to use the webpack build pipeline. (HTMX builds only)**
- Set CSRF token on the site body, for usage in HTMX, as [outlined here](https://django-htmx.readthedocs.io/en/latest/tips.html#make-htmx-pass-the-csrf-token).
- Added site name to admin sign up notification.
- Built-in admin emails now fail silently and don't cause sign-up errors if email sending fails.
- Made it possible to configure `ACCOUNT_EMAIL_VERIFICATION` separately by environment, so it can be enabled
  in production but disabled in dev.
- Made it possible to view/edit User's selected language in the Django admin.
- Updated `make pip-compile` to also compile your production requirements file (if relevant).
- Fly.io HTTP checks will now run using the site's configured HTTP HOST header.

### Fixed

- Fixed a bug introduced in 2022.12 that resulted in a duplicate team being created
  when users created an account while accepting a team invitation (Bootstrap and Tailwind only).
- Cleaned up styling of Monthly/Annual selector buttons on subscription UI (Tailwind only).
- Made icons in menus consistent sizes, to prevent minor alignment issues with wider/narrower icons (Tailwind only).
- Fixed an issue preventing user profile data from being saved if internationalization was enabled, but only a single
  language was configured.

### Upgrading

If you don't want to upgrade Django to 4.1 this upgrade *should* be backwards compatible.
Pin the Django version to 3.2.x in your requirements file and [rebuild requirements](python.md#working-with-requirements).

*Feb 1, 2023*

## Version 2022.12

This is a maintenance release with many small fixes and quality-of-life improvements suggested by the community.

Happy holidays!

### Added

- Added a `make upgrade` target to update docker containers and a local database after upgrading.
- Added a Redis instance to Github actions CI setup, so that any tests which depend on Redis can run without modification.
- Added default error pages and url routes for 400 and 403 errors.

### Changed

- **Updated usage of `.env` files. Python environments now use `.env`, docker uses `.env.docker`, and the example
  was renamed from `.env.dev.example` to `.env.example`. [Details here](configuration.md).** See upgrade notes.
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
For full details, see the new [feature flag documentation](flags.md).

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
See the new [Render deploy documentation](deployment/render.md).

### Feature: Fly.io deployment option

Pegasus now officially supports deploying to [fly.io](https://fly.io/).
See the new [Fly.io deploy documentation](deployment/fly.md).

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
  More [documentation here](./configuration.md#two-factor-authentication).

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
  For details see the updated [teams middleware docs](teams.md#middleware),
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

- **Pegasus now ships with a default, customizable logging configuration.** [Documentation](configuration.md#logging).
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
- **Pegasus now generates a sitemap for you.** [Documentation](page-metadata.md#sitemaps).
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
- **Added an `active_subscription_required` decorator for easier subscription feature-gating. [Docs](using-the-active-subscription-required-decorator)**
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
- Updated the [Sentry documentation](configuration.md#sentry) with the new setup and testing instructions.

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
This is a very similar issue to the one described in the [0.14.2 release notes](v-0-14-2).

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
See [the updated Subscription documentation](subscriptions.md#per-unit--per-seat-billing)
for an overview of this functionality, and bear in mind that it's a little complicated with several moving parts!

There were many code changes to support this work-as well as future planned improvements to the subscriptions module.
Additionally there were a number of smaller, unrelated updates and fixes.

You can read the complete notes below. The format for this release (and likely future releases) was borrowed from [keep a changelog](https://keepachangelog.com/).
The most significant changes are in **bold**.
Also please note the announcement of a few features that will be removed soon!


### Added

- **Support for per-unit / per-seat billing. See [the docs on using this](subscriptions.md#per-unit--per-seat-billing)**
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
- Extract Mailchimp mailing list logic to its own module and add [mailing list documentation](configuration.md#mailing-list).
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

(v-0-14-2)=
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
- Django added the new [Default primary key field type](https://docs.djangoproject.com/en/stable/ref/settings/#default-auto-field)
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
See the [deployment guide](deployment/digital-ocean.md) for details.

Additional updates:

- Remove duplicate `ACCOUNT_EMAIL_VERIFICATION` declaration in `settings.py`
- Rename development `Dockerfile` to `Dockerfile.dev` for clarity and ease of deployment to other platforms
- Fix SSL / mixed content errors when deploying on Google Cloud Run

*Nov 17, 2020*

## Version 0.10.4

This release adds experimental native support for deploying to Google Cloud Run.
More details can be found in the [deployment guide](deployment/google-cloud.md).

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

Release notes for earlier versions are no longer publicly available.
