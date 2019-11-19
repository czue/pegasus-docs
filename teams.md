Using Teams
===========

Teams are designed to provide sandboxes for groups of users collaborating on single project.
Users can join one or more teams, invite other users to their teams, and give different team members different roles.

Pegasus provides the building blocks to setup a team-based application.
Some of those building blocks are documented here.

**Note: all of the following examples assume you have setup Pegasus with teams enabled.**

## Models

Teams use three primary models - `apps.users.CustomUser`, `apps.teams.Team`, and `apps.teams.Membership`.

The `Membership` model uses [Django's "through" support](https://docs.djangoproject.com/en/2.2/ref/models/fields/#django.db.models.ManyToManyField.through) 
to extend the `User`/`Team` relationship with additional fields.

By default, a `role` field is added to represent the `User`'s role in the `Team` (admin or member).

## Team-based Views

At its core, all Team-based views need the following:

### Urls

See `apps.web.urls` for an example of these.
The convention Pegasus uses is to make all team-based URLs of the form
`https://example.com/a/<team_slug>/`. The `team_slug` is a human-readable, URL-friendly version
of the team name that is auto-generated for you.

### Views

See `apps.web.views` for example team views. 
The most important elements of a team-based view are permissions.

Team permissions are not managed for you but have to be explicitly set on your views (and data models). 

#### Permission Decorators

Pegasus includes two convenience decorators for use in team views.
These can be found in `apps.teams.decorators`.

##### The `login_and_team_required` decorator

This decorator can be used to ensure that the logged in user has access to the team in the view.
It requires your view's first argument be `team_slug`, as in the example views.
It can be used like this:

```python
@login_and_team_required
def a_team_view(request, team_slug):
    # other view logic here
    return render(request, 'web/my_template.html', context={
        'team': request.team,
    })

```

Note that `request.team` will automatically be populated. 
If the current user does not have access to the team they will see a 404 page.
If no user is logged in they'll be redirected to a login view, just like the `login_required` decorator.

##### The `team_admin_required` decorator

The `team_admin_required` decorator works just like the `login_and_team_required` decorator, except
in addition to checking team membership the role is also checked and if the user doesn't have
"admin" access they will not be able to access the view.

##### Superusers

The above decorators offer special exemptions for Django superusers, who are allowed to access
all teams and all pages by default.

## Watch the video

<div style="position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden; max-width: 100%; height: auto; margin-bottom: 1em;">
    <iframe src="https://www.youtube.com/embed/wMNYQCX4tTE" frameborder="0" allowfullscreen style="position: absolute; top: 0; left: 0; width: 100%; height: 100%;"></iframe>
</div>

Visual learner? The above video should get you going.
Else read on below for the play-by-play.

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

Follow one of the sections below depending on how you want to manage your virtualenvs.
 
### Using virtualenv

After installing prerequisites, run the following command wherever you want to create
the `virtualenv`. This will create the environment in the `{{ project_name_env }}` directory.

```
virtualenv -p python3 {{ project_name_env }}
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
mkvirtualenv -p python3 {{ project_name }}
```

Then to activate the environment you simply use:

```
workon {{ project_name }}
```

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

From there you should be able to access all bulit-in functionality and examples.

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
