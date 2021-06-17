# Upgrading

The easiest way to upgrade an existing Pegasus project to the latest version is as follows:

1. If you haven't already, [create a project on saaspegasus.com](https://www.saaspegasus.com/projects/) for your project.
   - If you are upgrading from verison `0.12` or later, you can find all the values you used in your `pegasus-config.yaml` 
     file in your project's root folder.
2. Download the code for your project for the latest Pegasus version.
3. Copy your `.git` directory from your old project into your new project.
   E.g. `cp -r path/to/yourapp/.git path/to/newapp/`.
4. In your new project directory, manually merge any changes you want to bring in,
   and commit and push the result to Github/Gitlab.
5. Pull the changes back into your original project directory from Github/Gitlab.

After upgrading you may also need to reinstall requirements (`pip install -r requirements.txt`),
npm packages (`npm install`), etc. depending on what has changed.
