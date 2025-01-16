Environment Setup
=================

## Choosing between Docker and native Python

You can either run Python in a Docker container or natively on your development machine,
through various different options.

Docker is easier to set up and provides a more consistent way to package your application,
however it is slower, takes more resources, and is more complex to integrate with IDEs, debuggers,
and other development tools.

Native Python can be more difficult to set up, especially on Windows, but once it is working it
is typically easier to work with.

Docker and uv are the recommended options.
You can pick either of these and then switch if you run into problems.

## Using Docker

See the [Docker documentation](/docker.md) to set up your development environment with Docker.

Docker environments support using uv or pip-tools as a package manager. Uv is recommended.
For help adding and removing Python packages after setup, see the documentation for 
[uv](./uv.md) or [pip-tools](./packages.md).

## Using uv

It's recommended that new projects not using docker use [uv](https://docs.astral.sh/uv/) to manage their Python environments.
It is faster and simpler to use than other alternatives, and can even install
and set up Python for you.

***Note: uv support is only available from Pegasus version 2024.12 onwards.
To use uv you must select it under the "Python package manager" setting in your project configuration.***

To set up Python with uv, first [install uv](https://docs.astral.sh/uv/getting-started/installation/).
*Pegasus requires uv version 0.5 or higher.*

On Linux / Mac:

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

On Windows:

```bash
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

After installing `uv`, go into your project directory and run:

```bash
uv sync
```

This should:

1. Install the right version of Python (if necessary).
2. Create a new virtual environment in a `.venv` folder inside your project.
3. Install all the project dependencies.

To see if it worked, run:

```bash
uv run manage.py shell
```

If you get a Python shell that looks something like this, it worked!

```
$ uv run manage.py shell
Python 3.12.6 (main, Sep  9 2024, 22:11:19) [Clang 18.1.8 ] on linux
Type "help", "copyright", "credits" or "license" for more information.
(InteractiveConsole)
>>> 
```

You should be able to use `uv run` to run any Python command on your project, or you can run:

```bash
source .venv/bin/activate
```

in your project root to use Python and other commands normally.

## Using Native / System Python (with Virtual Environments)

The following are other options---which are typically recommended for developers
who are already familiar with Python and one of these choices.

Unlike `docker` and `uv`, most of these require having Python installed on your machine,
so if you haven't already, first install Python version 3.11+:

- On Mac and windows you can [download Python 3.11 installers from here](https://www.python.org/downloads/).
- On Ubuntu it's recommended to [use the deadsnakes repo](https://www.debugpoint.com/install-python-3-11-ubuntu/).

*Note: running on older Python versions may work, but 3.11 is what's tested and supported.*

After installing Python, set up your virtual environment through one of the following methods:

#### Using your IDE

Many IDEs will manage your environments for you.
This is a great and simple option that you won't have to fiddle with.
Check your specific IDE's docs for guidance on how to do this.

- [Virtual environments in VS Code](https://code.visualstudio.com/docs/python/environments)
- [Virtual environments in PyCharm](https://www.jetbrains.com/help/pycharm/creating-virtual-environment.html)

**Be sure to choose Python 3.11 when setting up your virtual environment.**
If you don't see 3.11 as an option, you may need to install it first.

#### Using venv

The easiest way to set up a virtual environment manually is to use Python's built in
[`venv` tool](https://docs.python.org/3/library/venv.html#module-venv):

```bash
python3.11 -m venv /path/to/environment
```

In the command below, you should replace `python3.11` with the Python version you are using, and 
`/path/to/environment/` with the location on your system where you want to store the environment.
This location can be somewhere in your project directory (`.venv` and `venv` are common choices)
or anywhere else on your system.
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
tool that helps manage virtual environments.
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

Note: You can use `virtualenvwrapper` no matter how you created the environment.
It provides a nice set of helper tools, but can be a bit finicky to set up.
