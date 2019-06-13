Getting Started
===============

Here's everything you need to start your first Pegasus project.

## Install Prerequisites

If you haven't already, you'll need to install the following.

- [Python 3](https://www.python.org/downloads/)
- [Virtualenv](https://virtualenv.pypa.io/en/stable/)
- [Virtualenvwrapper](https://virtualenvwrapper.readthedocs.io/en/latest/)

*Note: It is possible to use the template without the virtualenv tools, 
but they are highly recommended and these instructions assume you have them.*

## Download Pegasus

If you haven't already, you'll need to purchase a Pegasus License at [saaspegasus.com](http://www.saaspegasus.com/).

From the "Downloads" page, download the latest version of the Pegasus template file and save it somewhere on your local file system.

## Setup a virtualenv and install Django

```
mkvirtualenv --no-site-packages -p python3 {{ project_name }}
pip install django
```

## Configure environment

Pegasus uses environment variables to help with its configuration.

If you are using teams you should set the `PEGASUS_USING_TEAMS` variable to `1`:

```
export PEGASUS_USING_TEAMS=1
```

If your project doesn't plan to use teams you should skip this step.

## Make a directory and start project

```
mkdir {{ project_name }}
```

After setting environment variables, to start a project with the template run the following command.
You will need to replace `/path/to/pegasus.tar` with the path to the downloaded file above.

```bash
django-admin startproject {{ project_name }} ./{{ project_name }}/ --template=/path/to/pegasus.tar --name=assets/index.js,package.json,README.md
```

## Enter the project directory

```
cd {{ project_name }}
```

You should see a lot of newly created files for your project including a `manage.py` file.

## Install requirements

```
pip install -r requirements.txt
```

## Setup database

```
./manage.py migrate
```

## Runserver

```
./manage.py runserver
```

Go to [http://localhost:8000](http://localhost:8000) and you should see the default Pegasus landing page!

![](images/pegasus-landing-page.png)

## Create a User

To create your first user account, just go through the sign up flow in your web browser.

### Promote your user to an admin

By default, accessing the examples is restricted to superusers only.
To convert your newly-created user into admin go through the following steps:

```
./manage.py shell
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

For some initial pointers on where to go from here, head on over to the [Customizations Page](/customizations).
