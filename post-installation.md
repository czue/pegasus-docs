Post-Installation Steps
=======================

At this point, Pegasus has installed scaffolding for user management, authentication, and (optionally) 
team views and Stripe subscriptions, and given you a beautiful base UI template and clear code 
structure to work from. 

Here are some common tasks you'll want to complete after installation:

## Create a User

To create your first user account, just go through the sign up flow in your web browser.

From there you should be able to access all built-in functionality and examples.

## Enable admin access

Follow [these instructions](cookbooks#use-the-django-admin-ui) to enable access to the Django Admin site.

## Set up your Stripe Subscriptions

If you've installed with subscriptions, you'll want to set things up next.

Head to the [subscriptions documentation](/subscriptions) and follow the steps there!

## Set up Background Tasks

For the progress bar example to work you'll need a Celery environment running.

If you're using Docker, this will be handled for you out-of-the-box. If not, head to [celery](/celery) and follow the steps there!

## Building Your Application

Now that you're up and running it's time for the fun part: building your new application!

The can obviously be done however you like.
Some examples of things you might want to do next include:

- Customize your landing page and setup a pricing page
- Start modifying the list of navigation tabs and logged-in user experience
- Create a new django app and begin building out your data models in `models.py`

For some initial pointers on where to to make Pegasus your own, head on over to the 
[Customizations Page](/customizations).

For the nitty-gritty details on setting up things like email, error logging, sign up flow, analytics, and more 
go to [Settings and Configuration](/configuration).
