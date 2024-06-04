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
