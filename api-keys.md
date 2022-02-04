API Keys
========

Pegasus supports the use of API Keys to access APIs, built on top of the 
[Django REST Framework API Key](https://florimondmanca.github.io/djangorestframework-api-key/) project.

Pegasus includes the ability to create API keys, associate them with your User objects, and access APIs using the key.

## Creating and managing API keys

A simple UI for creating, viewing, and revoking API keys is available to end users from the Profile page.

More advanced/customized management of API keys---including the ability to associate names and expiry dates with keys---is
available through the Django admin interface.

Note that when an API key is created it will be displayed *once* and will not be available after that.

For more details on working with API keys see [the library documentation](https://florimondmanca.github.io/djangorestframework-api-key/guide/#creating-and-managing-api-keys).

## API keys and Users

Pegasus associates API keys with your Django `User` objects.
This is a good, practical way to get started with API key scoping.
All access granted by the key will the same as the associated `CustomUser` object, which allow you to easily
create APIs that work with logged-in users *or* API keys.

The `apps.api.models.UserAPIKey` class is used to associate an API key with a `CustomUser`.
You can then enable API keys for any user-specific views, by following the instructions for `APIView`s and `ViewSet`s below.

More complex API key permissions---for example, associating a key with a single API or a single team---can
be created by following [these instructions](https://florimondmanca.github.io/djangorestframework-api-key/guide/#api-key-models).

To enable API-key support for an `APIView`, or `ViewSet`, use the `IsAuthenticatedOrHasUserAPIKey` permission class 
in place of `IsAuthenticated`. This will allow either authenticated users or UserAPIKey users to access the APIs.
In either case, the associated user object will be available as `request.user`.

You can see an example `APIView` in the `EmployeeDataAPIView` class that ships with the Pegasus examples,
and an example `ViewSet` in the `EmployeeViewSet` code.

## Testing API keys

The easiest way to test API key functionality is to use a tool like [curl](https://curl.se/).

The following command can be used to test a user-based API key with a default Pegasus installation:

```
curl http://localhost:8000/pegasus/employees/api/employees/ -H "Authorization: Api-Key <your-api-key>"
```

You should replace `<your-api-key>` with the API key displayed when it is created.
