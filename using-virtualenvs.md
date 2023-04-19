Using Virtual Environments
==========================

It's strongly recommended that you set up your project in a virtual environment.

Follow one of the sections below depending on how you want to manage your virtualenvs.

### Using your IDE

Many IDEs will manage your environments for you.
This is a great and simple option that you won't have to fiddle with.
Check your specific IDE's docs for guidance on how to do this.

### Using venv

The easiest way to set up a virtual environment manually is to use Python's built in
[`venv` tool](https://docs.python.org/3/library/venv.html#module-venv):

```
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

***You will need to activate this environment every time you work on your project.***
 
### Using virtualenv

[virtualenv](https://virtualenv.pypa.io/en/stable/) is an alternate option to `venv`.
On later versions of Python there's no real reason to use it, but if you're familiar with it
you can keep using it. First make sure [it's installed](https://virtualenv.pypa.io/en/stable/installation.html)
and then run the following command:

```
virtualenv -p python3.11 /path/to/environment
```

Like above, you should replace the `python3.11` variable with the version you want to use (3.9 or higher),
and the `/path/to/environment` with wherever you want to set up the environment.

Like with `venv`, to activate the environment run:

```
source /path/to/environment/bin/activate
```

***You will need to activate this environment every time you work on your project.***

### Using virtualenvwrapper

[Virtualenvwrapper](https://virtualenvwrapper.readthedocs.io/en/latest/) is an optional convenience 
tool that helps manage virtural environments.
You can use it with either `venv` or `virtualenv` above.
 
If you choose to use `virtualenvwrapper` you can use the following command to create your environment.
This can be run from anywhere since `virtualenvwrapper` manages the location of your envs for you
(usually in `/home/<user>/.virtualenvs/`).

```
mkvirtualenv -p python3.11 {{ project_name }}
```

Then to activate the environment you use:

```
workon {{ project_name }}
```

You can use `virtualenvwrapper` no matter how you created the environment.
It provides a nice set of helper tools, but can be a bit finicky to set up.
