Using Teams
===========

Teams are designed to provide sandboxes for groups of users collaborating on single project.
Users can join one or more teams, invite other users to their teams, and give different team members different roles.

Pegasus provides the building blocks to setup a team-based application.
Some of those building blocks are documented here.

**Note: all of the following examples assume you have setup Pegasus with teams enabled.**

## Example App

As of version 0.17, Pegasus ships with a built-in example application demonstrating the basics of working with team-based
models and views.

The example app includes:

1. A data model that belongs to a team.
2. A set of class based views for working with that data model, limited to the context of a team.


#### Third party examples

A Pegasus user Peter Cherna has created some more [example applications](https://github.com/pcherna/pegasus-example-apps/)
that demonstrate additional team-based examples, including functional views, pagination, APIs and working with "global" objects.

They are a great place to start for inspiration and getting something up and running quickly!

*Note: the example apps are not officially sanctioned/supported by Pegasus---though 
features from them will be continually incorporated into future releases.*

## Data Models

Teams use three primary models - `apps.users.CustomUser`, `apps.teams.Team`, and `apps.teams.Membership`.

The `Membership` model uses [Django's "through" support](https://docs.djangoproject.com/en/stable/ref/models/fields/#django.db.models.ManyToManyField.through) 
to extend the `User`/`Team` relationship with additional fields.

By default, a `role` field is added to represent the `User`'s role in the `Team` (admin or member).

### Team-based models 

Data models that "belong" to a Team can subclass `BaseTeamModel`.
See the example app for usage. 

## Team-based Views

At its core, all Team-based views need the following:

### Urls

See `apps.team.urls` for an example of how to set these up in your apps, and
your main `apps.{project}.urls` file for how to add them to your site's URLs.

Anything that goes into `team_urlpatterns` in `apps.{project}.urls` will automatically be added under the
URL `https://example.com/a/<team_slug>/`. The `team_slug` is a human-readable, URL-friendly version
of the team name that is auto-generated for you.

### Middleware

The `apps.teams.middleware.TeamsMiddleware` must be included in the list of middleware. It must be placed
after `django.contrib.auth.middleware.AuthenticationMiddleware`. The purpose of this middleware is to
set `request.team` and `request.team_membership` based on the current request. It will attempt to load
the team as follows:

* From the `team_slug` in the request path if available
* From the the current session if available
* From the user's list of teams if available

If the `team_slug` is available from the request path but it does not match a team that the user has access to 
then the request will terminate with a 404. Apart from this the middleware does not do any validation of the
team or the team membership. That is left to the decorators described below.

### Views

See `apps.team.views` for example team views.
All views that are referenced under `team_urlpatterns` must contain `team_slug` as the first argument.

In addition to adding this field, you will likely want to use one of the built-in permission
decorators (see below) to ensure the logged-in user can access the selected team.

Additionally, you will have to scope any data model access to the relevant Team
in any Database/ORM queries you make inside your views.

### Permission Control

Pegasus includes two convenience decorators for use in team views.
These can be found in `apps.teams.decorators`.

#### The `login_and_team_required` decorator

This decorator can be used to ensure that the logged in user has access to the team in the view.
It requires your view takes in a `team_slug`, as in the example views.
It can be used in functional views like this:

```python
@login_and_team_required
def a_team_view(request, team_slug):
    # other view logic here
    return render(request, 'web/my_template.html', context={
        'team': request.team,
    })
```

Or in class-based views like this:

```python
@method_decorator(login_and_team_required, name='dispatch')
class ATeamView(View):
    # other view details go here 
```

If the current user does not have access to the team they will see a 404 page.
If no user is logged in they'll be redirected to a login view, just like the `login_required` decorator.

#### The `team_admin_required` decorator

The `team_admin_required` decorator works just like the `login_and_team_required` decorator, except
in addition to checking team membership the role is also checked and if the user doesn't have
"admin" access they will not be able to access the view.

#### The `LoginAndTeamRequiredMixin` and `TeamAdminRequiredMixin` classes

These mixins provide the same functionality as the decorators, but are designed to work with Django's generic
class-based views. They can be used like this:

```python
class ATeamModelListView(LoginAndTeamRequiredMixin, ListView):
    model = MyModel
```

See the example app for more details.

### Template tags

In addition to the decorators, you can also use template tags to check user / team access from a template.

This can be useful for hiding/showing certain content based on a user's team role.
The `is_member_of` filter can be used to check team membership, and the `is_admin_of` filter can be used
to check if _a_ user is a team admin. For example, the following will show only if the logged in user
is an admin of the associated team:

```html
{% load team_tags %}
{% if team and request.user|is_admin_of:team %}
  <p>You're an admin of {{team.name}}.</p>
{% elif team and request.user|is_member_of:team %}
  <p>You're a member of {{team.name}}.</p>
{% else %}
  <p>Sorry you don't have access to {{team.name}}.</p>
{% endif %}
```
