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

### Permission Decorators

Pegasus includes two convenience decorators for use in team views.
These can be found in `apps.teams.decorators`.

#### The `login_and_team_required` decorator

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

#### The `team_admin_required` decorator

The `team_admin_required` decorator works just like the `login_and_team_required` decorator, except
in addition to checking team membership the role is also checked and if the user doesn't have
"admin" access they will not be able to access the view.

#### Superusers

The above decorators offer special exemptions for Django superusers, who are allowed to access
all teams and all pages by default.

