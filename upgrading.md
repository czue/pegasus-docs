# Upgrading

Upgrading Pegasus is currently a manual process though there are some plans to improve it in the future.

## Using patches
The following is the recommended way to upgrade a Pegasus project.
This process can also be used when changing any Pegasus configuration variables.

At a high level you will:

1. Create a patch file containing the changes in the upgrade.
2. Apply the patch to your app.

Here we'll walk through the steps in more detail.

### Creating the patch file

Follow these steps to create your patch file:

1. Download a "clean" version of your Pegasus project on your *current* version, and commit it to a git branch or repository.
2. Upgrade your Pegasus version (or change your configuration), and download the new codebase.
3. Copy your `.git` directory from your "clean" project in step 1 into your new project in step 2.
   E.g. `cp -r path/to/yourapp/.git path/to/newapp/`.
4. In your new project directory, *commit all of the changes* in a single commit. 
5. Create a patchfile for the commit using [`git-format-patch`](https://git-scm.com/docs/git-format-patch).
   The recommended command to run is `git format-patch -1 HEAD`.

You should now see a file in your repository root with a name like `0001-branch-details.patch`. This is your patch file.

### Applying the patch file

Now return to your main branch in your application's repository.

First, use [`git-apply`](https://git-scm.com/docs/git-apply) to apply the patch.
The recommended command to run is:

```
git apply --ignore-space-change --ignore-whitespace --reject /path/to/<patchname>.patch
```

substituting the path/name of the patchfile created above.

This command will do a best-effort application of the patch. 

For each affected file:

1. If updates could be applied cleanly, the file will be updated with the contents of the applied patch.
2. If updates could not be applied cleanly, a new diff file called `<filename>.rej` will be created, showing the diff that could not be applied. 

If the file was partially updated then the file will be modified *and* the remaining changes will
be visible in the `<filename>.rej` file.

The last step of the upgrade process is to go through each file and:

1. If the file has been modified, look at the modifications, see if you want them, and commit/reject them as necessary.
2. If the file has a `<filename>.rej` file, look at the proposed diff and see if you want to manually apply it, or ignore it.

After you have merged all changes to a file, you should delete the `<filename>.rej` file.

To understand the format of the `<filename>.rej` files, take a look at the [unified diff format](https://en.wikipedia.org/wiki/Diff#Unified_format).
Basically changes will look like the below, with a line starting with a minus sign, indicating a removal, and a plus
sign indicating an addition.

In this example, the type annotations were added to the function signature:

```
-def is_member(user, team):
+def is_member(user: CustomUser, team: apps.teams.models.Team) -> bool:
```

## No-longer recommended options

You can also upgrade without using patch files through one of the two approaches below.
Both of these have been used by community members, though the “opt in” approach is more common.

These approaches are typically **more work** and **more error prone** than the patch option,
but they don't involve diff files which may be unfamiliar/intimidating to some people.

### The "Opt-In" approach

In the "opt-in" approach, you view a diff of two clean Pegasus builds and manually bring any changes across to your main project.

It works like this:

1. Download a "clean" version of your Pegasus project on your *current* version, and commit it to a git branch or repository.
2. Upgrade your Pegasus version, and download the new codebase.
3. Copy your `.git` directory from your "clean" project into your new project.
   E.g. `cp -r path/to/yourapp/.git path/to/newapp/`.
4. In your new project directory, view the diff files and manually copy any changes you want to bring in into your main project.
   **Don't make any changes in the "clean" branch/repo.**
5. When you've finished, commit the new version into the "clean" branch/repo - which will allow you to repeat the process on the next release.

### The "Opt-Out" approach

In the "opt-out" approach, you copy a clean, upgraded Pegasus build on top of your existing codebase,
then merge or reject all changes.

Here are the steps:

1. Create a new branch in your existing project for the merge.
2. Download the code for your project for the latest Pegasus version.
3. Copy your `.git` directory from your *current* project into your new project.
   E.g. `cp -r path/to/yourapp/.git path/to/newapp/`.
4. In your new project directory, manually merge any changes you want to bring in, and *reject* any changes you don't want.
   *You should reject the deletion of any files you have added - likely entire apps and directories.*
5. When you're finished, commit the result and push it to Github/Gitlab.
6. Create a pull request, review the changes, and when you're happy, merge it.
7. Pull the changes back into your original project directory from Github/Gitlab.

## Other notes

After upgrading you may also need to reinstall requirements (`pip install -r requirements.txt`),
npm packages (`npm install`), etc. depending on what has changed.

You will also need to rebuild your front end if you've made any changes there (`npm run dev` or `npm run build`)
