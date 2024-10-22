Your Python Environment
=======================

## Choosing between Docker and Native Python

You can either run Python in a Docker container or natively on your development machine.

Docker is easier to set up and provides a more consistent way to package your application,
however it is slower, takes more resources, and is more complex to integrate with IDEs, debuggers,
and other development tools.

Native Python can be more difficult to set up, especially on Windows, but once it is working it
is typically easier to work with.

If you're not sure which you want, it's recommended to start with Docker and switch to native Python
if you are unhappy with the Docker experience.

## Using Docker

See the [Docker documentation](/docker.md) to set up your development environment with Docker.

## Using Native / System Python (with Virtual Environments)

If you're not using Docker, it's strongly recommended that you set up your project in a virtual environment,
which avoids dependency conflicts and makes it easier to run multiple Python apps on your machine. 

Follow one of the sections below depending on how you want to manage your virtualenvs.

### Using your IDE

Many IDEs will manage your environments for you.
This is a great and simple option that you won't have to fiddle with.
Check your specific IDE's docs for guidance on how to do this.

- [Virtual environments in VS Code](https://code.visualstudio.com/docs/python/environments)
- [Virtual environments in PyCharm](https://www.jetbrains.com/help/pycharm/creating-virtual-environment.html)

**Be sure to choose Python 3.11 when setting up your virtual environment.**
If you don't see 3.11 as an option, you may need to install it first.

### Manually managing environments

Follow these steps if you want to manage your virtual environments outside your IDE.
Using `venv` is recommended if you're not sure which option to use. 

#### Using venv

The easiest way to set up a virtual environment manually is to use Python's built in
[`venv` tool](https://docs.python.org/3/library/venv.html#module-venv):

```bash
python3.11 -m venv /path/to/environment
```

In the command below, you should replace `python3.11` with the Python version you are using (3.9 or higher), and 
`/path/to/environment/` with the location on your system where you want to store the environment.
This location can be somewhere in your project directory or anywhere else on your system.
`/home/<user>/.virtualenvs/<project>` is a common choice that works well with `virtualenvwrapper` (see below).

To activate/use the environment run:

```
source /path/to/environment/bin/activate
```

**You will need to activate this environment every time you work on your project.**
 
#### Using virtualenv

[virtualenv](https://virtualenv.pypa.io/en/stable/) is an alternate option to `venv`.
On later versions of Python there's no real reason to use it, but if you're familiar with it
you can keep using it without any issues.
First make sure [it's installed](https://virtualenv.pypa.io/en/stable/installation.html)
and then run the following command:

```bash
virtualenv -p python3.11 /path/to/environment
```

Like above, you should replace the `python3.11` variable with the version you want to use (3.9 or higher),
and the `/path/to/environment` with wherever you want to set up the environment.

Like with `venv`, to activate the environment run:

```
source /path/to/environment/bin/activate
```

And, like `venv`, **you will need to activate this environment every time you work on your project.**

#### Using virtualenvwrapper

[Virtualenvwrapper](https://virtualenvwrapper.readthedocs.io/en/latest/) is an optional convenience 
tool that helps manage virtural environments.
You can use it with either `venv` or `virtualenv` above.
 
If you choose to use `virtualenvwrapper` you can use the following command to create your environment.
This can be run from anywhere since `virtualenvwrapper` manages the location of your envs for you
(usually in `/home/<user>/.virtualenvs/`).

```bash
mkvirtualenv -p python3.11 {{ project_name }}
```

Then to activate the environment you use:

```
workon {{ project_name }}
```

You can use `virtualenvwrapper` no matter how you created the environment.
It provides a nice set of helper tools, but can be a bit finicky to set up.

## Working with Python Packages

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
