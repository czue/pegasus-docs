Getting Started
===============

Here's everything you should need to create your first Pegasus project.

## Download Pegasus

If you haven't already, you'll need to purchase a Pegasus License at [http://www.saaspegasus.com/](http://www.saaspegasus.com/).

From the "Downloads" page, download the latest version of the Pegasus template file and save this somewhere on your local file system.

## Install Prerequisites

If you haven't already, you'll need to install the following:

- [Python 3](https://www.python.org/downloads/)
- [Virtualenv](https://virtualenv.pypa.io/en/stable/)
- [Virtualenvwrapper](https://virtualenvwrapper.readthedocs.io/en/latest/)

*Note: It is possible to use the template without the virtualenv tools, but these instructions assume you have them.*

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

If your project doesn't use teams you should skip this step.

## Make a directory and start project

```
mkdir {{ project_name }}
```

After setting environment variables, to start a project with the template run:

```bash
django-admin startproject {{ project_name }} ./{{ project_name }}/ --template=pegasus/ --name=assets/index.js,package.json,README.md
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

Go to [http://localhost:8000](http://localhost:8000) and you should see the default pegasus landing page!

Next, move on to [Customizations](/customizations)...
