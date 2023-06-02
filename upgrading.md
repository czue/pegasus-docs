# Upgrading

Upgrading Pegasus is currently a manual process though there are plans to improve it in the future.

New projects are recommended to use the branch-based approach.
Older projects can use the patch-based approach.

The upgrade process can also be used when changing any Pegasus configuration variables.

## Using branches (recommended)

With this option you maintain a "pure" Pegasus branch in your repository with no other modifications.
Then, you merge this branch into your main app when you upgrade.

This process is outlined below, and also in the below screencast which shows a live example on a real Pegasus project.

<div style="position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden; max-width: 100%; height: auto; margin-bottom: 1em;">
    <iframe src="https://www.youtube.com/embed/A6fjoXw6f5g" frameborder="0" allowfullscreen style="position: absolute; top: 0; left: 0; width: 100%; height: 100%;"></iframe>
</div>

Here are the steps to take:

### 1. Create a branch for the upgrade

First [checkout the first commit](https://stackoverflow.com/questions/43197105/how-do-you-jump-to-the-first-commit-in-git)
in your repository and create a new branch from there.

After finding and checking out the initial commit, run:

```
git branch pegasus
git checkout pegasus
```

*Note: if you created the `pegasus` branch when you set up your codebase you can skip this step.
Alternatively, if you don't have any commit with pure pegasus code, see the instructions at the bottom
of this page to create one.*

Next, make sure the branch is up-to-date with your current Pegasus version:

1. Download your Pegasus project on your *current* version and unzip the code.
2. Copy the `.git` folder from your main project into the downloaded codebase.
3. Make sure you are on the `pegasus` branch (`git checkout pegasus`)
4. Commit all changes (`git add .` then `git commit -am "ready to upgrade"`)

### 2. Upgrade the code in the branch

1. Upgrade your project on saaspegasus.com
2. Download the latest codebase and unzip the code.
3. Copy the `.git` folder from step 1 into this new folder.
4. Commit all changes (`git add .` then `git commit -am "upgrade to latest Pegasus"`)

### 3. Merge into your main branch

1. Checkout the main branch (`git checkout main`)
2. Merge the code (`git merge pegasus`)

Alternatively you may wish to do this in a new branch and then submit a pull request to the main branch from there:

1. Create a new branch off of the main branch (`git checkout main; git checkout -b upgrade-pegasus`)
2. Merge the code (`git merge pegasus`)

In the merging step you should look at the modifications being made, and you may have to manually resolve conflicts that come up.
You may also need to run `./manage.py makemigrations` to create any database migrations that were not included with Pegasus. 

## Using patches (if you can't use branches)

You can also follow a similar process to the above using Git patches.
Patches do not require working in the same repository or having a previously created branch.

At a high level you will:

1. Create a patch file containing the changes in the upgrade.
2. Apply the patch to your app.

Here we'll walk through the steps in more detail.

### 1. Creating the patch file

Follow these steps to create your patch file:

1. Download a "clean" version of your Pegasus project on your *current* version, and commit it to a git branch or repository.
2. Upgrade your Pegasus version (or change your configuration), and download the new codebase.
3. Copy your `.git` directory from your "clean" project in step 1 into your new project in step 2.
   E.g. `cp -r path/to/yourapp/.git path/to/newapp/`.
4. In your new project directory, *commit all of the changes* in a single commit. 
5. Create a patchfile for the commit using [git-format-patch](https://git-scm.com/docs/git-format-patch).
   The recommended command to run is `git format-patch -1 HEAD`.

You should now see a file in your repository root with a name like `0001-branch-details.patch`. This is your patch file.

### 2. Applying the patch file

Now return to your main branch in your application's repository.

First, use [git-apply](https://git-scm.com/docs/git-apply) to apply the patch.
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

## Other notes

After upgrading you may also need to reinstall requirements (`pip install -r requirements.txt`),
npm packages (`npm install`), etc. depending on what has changed.

You will also need to rebuild your front end if you've made any changes there (`npm run dev` or `npm run build`)

If you are using docker you can use the 'upgrade' make target to do this:

```
make upgrade
```

This will rebuild the Docker images and create and run any database migrations that are needed.
**Note: your web container needs to be running when you run this or it will fail.**

## If you don't have a "pure" Pegasus branch

In some cases you may not have a "clean" Pegasus branch.
This could happen if you did substantial development before your first commit, merged Pegasus into another project,
or did several upgrades.

In this case you can fake a pure Pegasus branch by taking the following steps.
*Note: this process destroys `git blame` for most of your project.*

1. Save your current code (from the "main" branch).
2. Make a new branch (e.g. called "pure-pegasus")
3. Download your codebase from saaspegasus.com on the *last release your project used/upgraded to*.
4. Put the unmodified download of Pegasus 2022.3 onto that branch, without any of your own code.
   The easiest way to do that is to copy the `.git` folder into your downloaded project and immediately commit the result.
   This will "brutally" overwrite all your customizations in your git history. *Note this commit id.*
5. Then repeat this process, but instead, do the reverse. Copy the `.git` folder from the Pegasus download back into your
   (unmodified) copy of the `main` code and again commit the result.
   This will create a single commit containing all customizations you've made to your project.
6. Review this code and merge it back into `main`. If you did everything correctly you should have two huge commits
   but the pull request will contain no changes.

After this process git will believe that the pure pegasus code is fully merged to `main`.
You can then use the commit id you noted in step 4 as the starting point for your upgrade
([step 1](https://docs.saaspegasus.com/upgrading.html#create-a-branch-for-the-upgrade)),
and jump to [step 2 above](https://docs.saaspegasus.com/upgrading.html#upgrade-the-code-in-the-branch).
