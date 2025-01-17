Using Github Actions
====================

[GitHub Actions](https://github.com/features/actions) allows you to automate your software workflows.
Pegasus apps optionally ship with Github actions support for a few things to build off.

If you've built with Github actions support, they should successfully run the first time you push your code to Github.

Actions are configured in the `.github` directory in your project.
The following actions ship with Pegasus:

## Running Django Tests

The Django tests are configured in `.github/tests.yml`.

By default, it will:

 - Run on every push to the `main` branch and every pull request.
 - Run on Python version {{ default_python_version }} (other Python versions can be added by modifying the `python-version` list)
 - Use the latest version of Postgres
 - Run `./manage.py test`

All of these can be changed by modifying the relevant sections of the `.github/tests.yml` file.

## Building the Front End

The front end build is configured in `.github/build_frontend.yml`.

By default, it will:
 - Run on every push to the `main` branch and every pull request.
 - Run on Node version {{default_node_version}} (other Node versions can be added by modifying the `node-version` list).
 - Run `npm run build`, ensuring your front end builds properly.
 - Run `npm run type-check`, ensuring all type checks pass.

Any compilation errors in your JavaScript should show up as build failures.
