Environment Setup
=================

## Choosing between Docker and native Python

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

### Using uv (recommended)

It's recommended that new projects use [uv](https://docs.astral.sh/uv/) to manage their Python environments.
It is faster and simpler to use than other alternatives, and can even install
and set up Python for you.

To use uv first [install it](https://docs.astral.sh/uv/getting-started/installation/).

Then in your project root directory run:

```bash
uv sync
```

After that you should be good to go!
You should be able to use `uv run` to run any Python command on your project,
or you can run:

```bash
source .venv/bin/activate
```

To use Python and other commands normally.

**Note: uv is currently in beta testing but is planned to become the default set up for new projects soon.**

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
