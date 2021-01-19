# Upgrading

The recommended workflow for upgrading an existing Pegasus project to the latest version is as follows:

1. Make sure you have the latest version of Pegasus: `pip install --upgrade pegasus-installer`
2. Create a new Pegasus project in a new directory:
   - If you are upgrading from verison `0.12` and later, you can use
     the generated configuration file to specify the default values: `pegasus path/to/pegasus.zip --config-file path/to/yourapp/pegasus-config.yaml --no-input`.
   - If upgrading from an older version you will have to run: `pegasus path/to/pegasus.zip` and fill in the prompts again.
3. Copy your `.git` directory from your old project into your new project.
   E.g. `cp -r path/to/yourapp/.git path/to/newapp/`.
4. In your new project directory, manually merge any changes you want to bring in,
   and commit and push the result to Github/Gitlab.
5. Pull the changes back into your original project directory from Github/Gitlab.

After upgrading you may also need to reinstall requirements (`pip install -r requirements.txt`),
npm packages (`npm install`), etc. depending on what has changed.
