Getting Started
===============

Here's everything you need to start your first Pegasus project.

## Install Prerequisites

If you haven't already, you'll need to install the following.

- [Python 3](https://www.python.org/downloads/)
- [Virtualenv](https://virtualenv.pypa.io/en/stable/)

*Note: It is possible to use the template without `virtualenv`, 
but it is highly recommended, and these instructions assume you are using it.*

## Download Pegasus

If you haven't already, you'll need to purchase a Pegasus License at [saaspegasus.com](http://www.saaspegasus.com/).

From the "Downloads" page, download the latest version of the Pegasus template file and save it somewhere on your local file system.

## Setup a Python3 virtualenv

It's recommended that you setup your project in a `virtualenv`.
[Here are some reasons why](https://help.pythonanywhere.com/pages/VirtualenvsExplained/).
 
### Using `virtualenv`

After installing prerequisites, run the following command wherever you want to create
the `virtualenv`. This will create the environment in the `{{ project_name_env }}` directory.

```
virtualenv -p python3 {{ project_name_env }}
```

To activate the environment then run:

```
source {{ project_name_env }}/bin/activate
```

**You will need to activate this environment every time you work on your project.**

### Alternative instructions using `virtualenvwrapper`

[Virtualenvwrapper](https://virtualenvwrapper.readthedocs.io/en/latest/) is an optional convenience 
tool that helps manage virturalenvs. 
 
If you choose to use `virtualenvwrapper` you can use the following command to create your environment.
This can be run from anywhere since `virtualenvwrapper` manages the location of your envs for you.

```
mkvirtualenv -p python3 {{ project_name }}
```

Then to activate the environment you simply use:

```
workon {{ project_name }}
```

## Install cookiecutter

This project uses [Cookiecutter](https://cookiecutter.readthedocs.io/en/latest/) to configure your template,
which must be installed before proceeding.

To do this, from inside your virtualenv just run:

```
pip install cookiecutter>=1.6.0
```

## Create your project

Go into the directory where you wish to create your project and run the following command.
You will need to replace `/path/to/pegasus.zip` with the path to the downloaded file above.

```
cookiecutter /path/to/pegasus.zip
```

The command will run you through a series of configuration prompts. 
Just fill in your project information for all the sections and let it complete.

Once the command runs a new folder with your project's name will be created.

## Enter the project directory

```
cd {{ project_name }}
```

You should see a lot of newly created files for your project including a `manage.py` file.

## Install package requirements

```
pip install -r requirements.txt
```

## Setup database

```
python ./manage.py migrate
```

## Runserver

```
python ./manage.py runserver
```

Go to [http://localhost:8000](http://localhost:8000) and you should see the default Pegasus landing page!

![](images/pegasus-landing-page.png)

## Create a User

To create your first user account, just go through the sign up flow in your web browser.

### Promote your user to an admin

By default, accessing the examples is restricted to superusers only.
To convert your newly-created user into admin go through the following steps:

```
python ./manage.py shell
>>> from apps.users.models import CustomUser
>>> user = CustomUser.objects.all()[0]
>>> user.is_superuser = True
>>> user.is_staff = True
>>> user.save()
>>> exit()
``` 

Refreshing the page should bring up the example gallery tab.

## Building Your Application

At this point, Pegasus has installed scaffolding for all of the user management, authentication, and (optionally) team views,
and given you a beautiful base UI template and clear code structure to work from. 
 
Now that you're up and running it's time for the fun part: building your new application!

The can obviously be done however you like.
Some examples of things you might want to do next include:

- Customize your landing page and setup a pricing page
- Start modifying the list of navigation tabs and logged-in user experience
- Create a new django app and begin building out your data models in `models.py`

For some initial pointers on where to to make Pegasus your own, head on over to the [Customizations Page](/customizations).

For the nitty-gritty details on setting up things like email, error logging, sign up flow, analytics, and more 
go to [Settings and Configuration](/configuration).
