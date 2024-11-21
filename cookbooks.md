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

## Migrating from pip-tools to uv

To migrate your project from pip-tools to uv follow these steps.

### Install uv

If you haven't already, [install uv](https://docs.astral.sh/uv/getting-started/installation/):

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

### Update your project code

It's recommended to do this in two steps:

1. [Upgrade your project](./upgrading.md) to the latest Pegasus version, keeping your pacakge manager as "pip-tools".
   Merge all conflicts and ensure your project is working properly on this version.
2. Then, change the package manager from pip-tools to uv in your project settings and do another upgrade/pull request.

At this point you will likely have conflicts in your requirements files, but hopefully nowhere else.
See the next sections for resolving these.

### Prepare to resolve conflicts

First, follow the github instructions to merge your project on your local machine, by checking out the pegasus upgrade
branch and merging the main branch into it.
You will have to update the command below with the exact branch name of the pull request created by Pegasus:

```
git fetch origin
git checkout pegasus-<version>-<timestamp>
git merge main
```

At this point you'll have a partially merged branch with conflicts.

### Migrate your requirements.in files

The uv build of Pegasus no longer uses requirements files, so any changes you've made to these will need
to be migrated to `pyproject.toml` and `uv.lock`.

You can use the [reqs-sync](https://github.com/saaspegasus/reqs-sync/) package to help with this.
Follow the steps below for any file with conflicts.

To migrate your main *requirements.in* file:

```bash
uv tool run reqs-sync reqs-to-toml requirements/requirements.in
```

To migrate your development *dev-requirements.in* file:

```bash
uv tool run reqs-sync reqs-to-toml requirements/dev-requirements.in --group=dev
```

To migrate your production *prod-requirements.in* file:

```bash
uv tool run reqs-sync reqs-to-toml requirements/prod-requirements.in --group=prod
```

These commands should copy all project requirements from your `requirements.in` file(s) to your `pyproject.toml` file
(into the appropriate group, if necessary).

### Update your uv.lock file

Next you should rebuild your `uv.lock` file from the updated `pyproject.toml` file:

```bash
uv lock
```

You should then check the versions that were added to the `uv.lock` file and update any as
needed based on the versions your requirements.txt files.

### Test the migration

Run your project (`uv run python manage.py runserver`) and verify everything works as expected.

### Remove your requirements files

Finally, run:

```bash
git rm requirements/*`
```

To remove all your requirements files.

Congratulations, you've migrated to uv!
Resolve any other conflicts, push and merge your code, and you're done!

## Migrating to auto-formatted code

As of February, 2023 all Pegasus projects have the option to auto-format your Python code.

To migrate a project from non-formatted to formatted code, you can go through the following steps:

1. First, do a full Pegasus upgrade to the version you want to update to, as described [here](./upgrading.md).
   **Do *not* check the "autoformat" checkbox yet.**
2. Next, run the formatting tools on your project's `main` branch: 
   1. Install ruff: `pip install ruff`
   2. Run ruff linting `ruff check --extend-exclude migrations --line-length 120 . --fix`
   3. Run ruff formatting: `ruff format --line-length 120 .`
3. Commit the result:
   1. `git add .`
   2. `git commit -m "apply formatting changes"`
4. Finally, check the "autoformat" box on your Pegasus project, and do *another* upgrade according to the same process.

## Delete Pegasus Examples

You can remove the Pegasus examples by unchecking the "Include Examples" checkbox
on your project page and re-downloading (or [upgrading](upgrading.md)) your codebase.

For earlier versions you can use [these instructions](https://github.com/czue/pegasus-docs/blob/1becc2cb8f86738eeba85c9faddb15f69b8ad7bc/cookbooks.md#delete-pegasus-examples).
