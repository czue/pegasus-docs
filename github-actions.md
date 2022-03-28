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

 - Run on every push to the `main` branch and every pull request to the `main` branch.
 - Run on Python version 3.8, 3.9, and 3.10.
 - Use the latest version of Postgres
 - Run `./manage.py tests`

All of these can be changed by modifying the relevant sections of the `.github/tests.yml` file.

## Building the Front End

The front end build is configured in `.github/build_frontend.yml`.

By default, it will:
 - Run on every push to the `main` branch and every pull request to the `main` branch.
 - Run on Node version 14, 16, and 17
 - Run `npm run build`

Any compilation errors in your JavaScript should show up as build failures.
