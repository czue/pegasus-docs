Legacy Installation
==================

*These instructions are for older users and no longer officially supported.*

# Download the Pegasus Source
 
From the ["Downloads" page](https://www.saaspegasus.com/releases/), download the latest version of the Pegasus template
file and save it somewhere on your local file system.

## Setup a Python 3.7+ virtualenv

See [Using Virtual Environments](/using-virtualenvs/) for details on this process.

## Setup Pegasus

To run pegasus you'll first need to install the installer and its dependencies.

To do this, from inside your virtualenv just run:

```
pip install pegasus-installer
```

## Create your project

Go into the directory where you wish to create your project and run the following command.
You will need to replace `/path/to/pegasus.zip` with the path to the downloaded file above.

```
pegasus /path/to/pegasus.zip
```

The command will run you through a series of configuration prompts. 
Just fill in your project information for all the sections and let it complete.

Once the command runs a new folder with your project's name will be created.

## Choose how you'll run your project

The easiest way to get up and running from here is to use Docker.
Head over to the [Docker docs](/docker/) to finalize your set up.

Alternatively, if you prefer not to use Docker, you can follow the steps below.


## Enter the project directory

```
cd {{ project_name }}
```

You should see a lot of newly created files for your project including a `manage.py` file.

## Install package requirements

```
pip install -r requirements.txt
```

## Set up database (Postgres only)

If you installed with Postgres, edit the `DATABASES` value in `{{ project_name }}/settings.py` with
the appropriate details.

You will also need to create a database for your project if you haven't already:

```bash
sudo -u postgres createdb {{ project_name }}
```

## Create database migrations

```bash
python ./manage.py makemigrations
```

## Run database migrations

```bash
python ./manage.py migrate
```

## Run server

```bash
python ./manage.py runserver
```

Go to [http://localhost:8000](http://localhost:8000) and you should see the default Pegasus landing page!

![Landing Page](images/pegasus-landing-page.png)

## Create a User

To create your first user account, just go through the sign up flow in your web browser.

From there you should be able to access all built-in functionality and examples.

## (Optional) Enable admin access

Follow [these instructions](cookbooks#use-the-django-admin-ui) to enable access to the Django Admin site.

## (Optional) Set up your Stripe Subscriptions

If you've installed with subscriptions, you'll want to set things up next.

Head to the [subscriptions documentation](/subscriptions) and follow the steps there!

## (Optional) Set up Background Tasks

For the progress bar example to work you'll need a Celery environment running.

Head to [celery](/celery) and follow the steps there!

## Building Your Application

At this point, Pegasus has installed scaffolding for all of the user management, authentication, and (optionally) 
team views and Stripe subscriptions, and given you a beautiful base UI template and clear code 
structure to work from. 
 
Now that you're up and running it's time for the fun part: building your new application!

The can obviously be done however you like.
Some examples of things you might want to do next include:

- Customize your landing page and setup a pricing page
- Start modifying the list of navigation tabs and logged-in user experience
- Create a new django app and begin building out your data models in `models.py`

For some initial pointers on where to make Pegasus your own, head on over to the 
[Customizations Page](/customizations).

For the nitty-gritty details on setting up things like email, error logging, sign up flow, analytics, and more 
go to [Settings and Configuration](/configuration).
