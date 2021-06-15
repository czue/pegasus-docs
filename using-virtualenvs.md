Using Virtual Environments
==========================

It's recommended that you setup your project in a `virtualenv`.
[Here are some reasons why](https://help.pythonanywhere.com/pages/VirtualenvsExplained/).

Follow one of the sections below depending on how you want to manage your virtualenvs.
 
### Using virtualenv

After installing prerequisites, run the following command wherever you want to create
the `virtualenv`. This will create the environment in the `{{ project_name_env }}` directory.
You should replace the "python3.8" variable with whatever version you want to use, 
but it *must be 3.7 or above*.

```
virtualenv -p python3.8 {{ project_name_env }}
```

To activate the environment then run:

```
source {{ project_name_env }}/bin/activate
```

***You will need to activate this environment every time you work on your project.***

### Alternative instructions using virtualenvwrapper

[Virtualenvwrapper](https://virtualenvwrapper.readthedocs.io/en/latest/) is an optional convenience 
tool that helps manage virturalenvs. 
 
If you choose to use `virtualenvwrapper` you can use the following command to create your environment.
This can be run from anywhere since `virtualenvwrapper` manages the location of your envs for you.

```
mkvirtualenv -p python3.8 {{ project_name }}
```

Then to activate the environment you use:

```
workon {{ project_name }}
```

