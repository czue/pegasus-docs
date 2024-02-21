Github Integration
==================

As of February, 2024 you can connect your Pegasus projects directly to Github instead of downloading them as a zip file.
This makes for a more streamlined workflow---especially when changing or upgrading your project.

## Watch the video

The following video shows how to create and update a project using the Github integration.

<div style="position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden; max-width: 100%; height: auto; margin-bottom: 1em;">
    <iframe src="https://www.youtube.com/embed/5PLO79rb--A" frameborder="0" allowfullscreen style="position: absolute; top: 0; left: 0; width: 100%; height: 100%;"></iframe>
</div>

## Connecting your account

### Using "Connect Github"

The easiest way to connect your account is by using the "Connect Github" button on the project download page.
You will be prompted to accept permissions, and your account will be connected in a few seconds.

### Using Personal Access Tokens

If you prefer not to grant Pegasus access to your entire Github account, you can use
[Personal Access Tokens](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens)
to limit the scope of what Pegasus has access to.

#### With Classic Tokens

To use Pegasus with a classic token, visit the [Personal access tokens](https://github.com/settings/tokens) page on Github,
then select "Generate new token (classic)" from the dropdown, or [visit this page](https://github.com/settings/tokens/new).

Choose a note and expiration date for your token and grant the following scopes:

- repo (Full control of private repositories)
- workflow (Update GitHub Action workflows)

Then click "Generate token".
You will be taken to a page where your token is displayed.
Copy this value and paste it into the "personal access token" field from your project download page on Pegasus.
Note that you won't be able to view the token again!

#### With Fine-Grained Access Tokens

If you want the most control over your permissions, you should use a fine-grained access token,
which allow you to control access to specific repositories.

Note that if you use fine-grained tokens **you must create the repository for your project before creating the token**.
Pegasus cannot create the project for you with these tokens.

After creating the repository, [create a new fine-grained-token from this page](https://github.com/settings/personal-access-tokens/new).
Set a token name and expiration date, and then use "Only select repositories" to choose the repositories you want to
grant access to (the one you just created).

Then under "Permissions" --> "Repository Permissions" you must grant read and write access to:

- Contents
- Pull Requests
- Workflows

Then click "Generate token".
You will be taken to a page where your token is displayed.
Copy this value and paste it into the "personal access token" field from your project download page on Pegasus.
Note that you won't be able to view the token again!
