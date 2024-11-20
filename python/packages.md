Working with Python Packages (pip-tools)
========================================

Pegasus uses [pip tools](https://github.com/jazzband/pip-tools) to manage Python dependencies.
This allows for more explicit dependency management than a standard `requirements.txt` file.

### Requirements Files

Pegasus has multiple requirements files, which live in the `requirements/` folder.
For each set of requirements there are two files, one ending in `.in` and the other ending in `.txt`.

The files ending in `requirements.in` have the first-class packages your app depends on.
They do not have versions in them, though you can add version numbers if you want to.
**These are the files that you should edit when adding/removing packages**.

The files ending in `requirements.txt` have the full list of packages your app depends on,
including the dependencies of your dependencies (recursively).
This file is automatically generated from the `.in` counterpart, and **should typically not be edited by hand**.

The `requirements.in`/`.txt` files are the main requirements for your application, `dev-requirements.in`/`.txt` files
are requirements for development-only, and `prod-requirements.in`/`.txt` are for production-only.

### Working with requirements

To modify the requirements files, you first need to install `pip-tools`.
It is included as a dependency in the `dev-requirements.txt` file so if you've followed the local setup
steps it should already be installed.

Then follow the instructions below, depending on what you want to do:

#### Adding or removing a package

To add a package, add the package name to `requirements/requirements.in`.
To remove a package, remove it from `requirements/requirements.in`.

After finishing your edits, rebuild your `requirements.txt` file by running:

```
# native version
pip-compile requirements/requirements.in

# docker version
make pip-compile
``` 

After running this you should see the package and its dependencies added to the `requirements.txt` file.

From there you can install the new dependencies, as [described below](#installing-packages).

#### Upgrading a package

To upgrade a package, you can run the following command. In this example we are upgrading `django`:

```
# native version
pip-compile --upgrade-package django requirements/requirements.in

# docker version
make pip-complie ARGS="--upgrade-package django"
```

To upgrade *all* packages, you can run:

```
# native version
pip-compile --upgrade requirements/requirements.in

# docker version
make pip-compile ARGS="--upgrade"
```

From there you can install the new dependencies, as [described below](#installing-packages).

#### Installing Packages

If you're running Python natively, you can install your packages with the following command.
Run this after activating your virtual environment:

```
pip install -r requirements/requirements.txt
```

In Docker your Python packages are installed at container *build* time.
This means that any time you want to change your installed new packages, you have to rebuild your container.

You can do this by running

```
docker compose build
```

Confusingly, running `pip install` or `docker compose exec web pip install` does *not* work.

#### The `make requirements` shortcut for Docker

Pegasus ships with a convenience target for rebuilding requirements with Docker.
Any time you make changes to a `requirements.in` file you can run it with:

```
make requirements
```

Behind the scenes this will:

1. Rebuild all your `-requirements.txt` files from your `-requirements.in` files with `uv`.
2. Rebuild your containers (installing the new packages).
3. Restart your containers.
 
For more information, see the [docker documentation](docker.md#updating-python-packages).