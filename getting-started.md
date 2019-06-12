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

Next, move on to [Customizations](/customizations)...
