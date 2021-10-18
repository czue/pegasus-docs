# Upgrading

There are two common approaches to upgrading Pegasus projects.
Both are used by community members, though the "opt in" approach is more commmonly recommended.

## The "Opt-In" approach

In the "opt-in" approach, you view a diff of two clean Pegasus builds and manually bring any changes across to your main project.

It works like this:

1. Download a "clean" version of your Pegasus project on your *current* version, and commit it to a git branch or repository.
2. Upgrade your Pegasus version, and download the new codebase.
3. Copy your `.git` directory from your "clean" project into your new project.
   E.g. `cp -r path/to/yourapp/.git path/to/newapp/`.
4. In your new project directory, view the diff files and manually copy any changes you want to bring in into your main project.
   **Don't make any changes in the "clean" branch/repo.**
5. When you've finished, commit the new version into the "clean" branch/repo - which will allow you to repeat the process on the next release.

## The "Opt-Out" approach

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
