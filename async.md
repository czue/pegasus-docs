Async and Websocket Support
===========================

As of version 2023.10, Pegasus provides support [asynchronous support](https://docs.djangoproject.com/en/stable/topics/async/),
as well as support for websockets via the [channels library](https://channels.readthedocs.io/).

## Enabling Async Support

You can enable Async support by checking the "Use Async / Websockets" option in your project settings.
Enabling Async will:

1. Change your default development server to [Daphne](https://docs.djangoproject.com/en/stable/howto/deployment/asgi/daphne/).
2. Change your default production server to [Uvicorn](https://www.uvicorn.org/) (via gunciorn).
3. Add and configure `channels` in your project for websocket support.

In addition to the above configuration changes, enabling async will also use it for LLM chats if available.
Finally, there is an optional group chat application you can separately add (details below).

## The Async / Websocket Demo Application

Pegasus includes an optional demo application to demonstrate the asynchronous and socket capabilities.
The demo application is an extension of the demo application that you build while completing the
[channels tutorial](https://channels.readthedocs.io/en/latest/tutorial/index.html).
You can see a demo below.

<div style="position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden; max-width: 100%; height: auto; margin-bottom: 1em;">
    <iframe src="https://www.youtube.com/embed/J1hma14whz4" frameborder="0" allowfullscreen style="position: absolute; top: 0; left: 0; width: 100%; height: 100%;"></iframe>
</div>


The demo application uses the [HTMX websockets extension](https://htmx.org/extensions/web-sockets/) to simplify
the implementation. If you prefer not to use HTMX at all, you can change your websocket connection logic
to use vanilla JavaScript instead, as shown in the [channels tutorial here](https://channels.readthedocs.io/en/latest/tutorial/part_2.html#add-the-room-view).

A React-based websocket demo is on the roadmap.

## Websocket urls

Websocket URLs are defined separately from your app's main `urls.py` file.
In Pegasus, the convention is to put your websocket urls in `channels_urls.py`
in your project folder (the same one containing `urls.py`).

Because websocket urls are separate from your main app, and because they follow a different protocol,
they must be referenced as absolute URLs in your front end (including prepending "ws://" or "wss://" depending on whether 
you're using HTTPS).

Pegasus ships with two helper functions you can use to assist with working with URLs, so long as you follow Pegasus conventions.
The `websocket_reverse` function will reverse a relative websocket URL, and the `websocket_absolute_url` function
will turn a relative URL into an absolute websocket URL based on your Site address and the `USE_HTTPS_IN_ABSOLUTE_URLS` setting.

You can combine these functions like so to pass the URL of a websocket endpoint to a template:

```python
room_ws_url = websocket_absolute_url(websocket_reverse("ws_group_chat", args=[room_id]))
```

You can then use the websocket URL in a template/JavaScript like this:

```
const chatSocket = new WebSocket({{ room_ws_url}});
chatSocket.onmessage = function(e) {
  // handle message 
};
```

## Asynchronous web servers

There are several ASGI servers supported by Django.
By default, Pegasus uses the Daphne web server in development and the Uvicorn web server in production,
for reasons described below.
That said, you can customize your app to use whichever server you prefer.

### Daphne

In development, Pegasus uses the [Daphne](https://pypi.org/project/daphne/) web server for its tight integration with Django's `runserver` command,
as [outlined in the Django docs](https://docs.djangoproject.com/en/4.2/howto/deployment/asgi/daphne/).

Daphne is installed via `dev-requirements` and will be added to your `INSTALLED_APPS` whenever `settings.DEBUG` is `True`.

### Uvicorn

In production, Pegasus uses the [Uvicorn](https://www.uvicorn.org/) web server.
Uvicorn has a seamless integration with `gunicorn`, making transitioning to it very easy.

Uvicorn is installed via `prod-requirements`, and if you build with async features enabled, your `gunicorn` command
will be updated to use it.

## Troubleshooting

**The chat app loads but nothing happens when I send a message.**

The most likely reason this would happen is if your site URLs are not set up properly,
which would cause the websocket endpoints to not hit the right address.

See the documentation on [absolute URLs](configuration.md#absolute-urls) to fix this,
and in particular make sure your Django site object has the right domain.
In development this should be set to `localhost:8000`.

**I'm getting an error: No module named 'daphne'**

If you are getting this error *in production* it is likely because your `DEBUG` environment variable is not set.

Due to the order in which settings are imported, you *must* define `DEBUG=False` in your *environment*,
`.env` file, or main `settings.py` file.
This is in addition to (or instead of) setting `DEBUG=False` in your `settings_production.py` file.

If you are getting this error *in development*, be sure that Daphne is installed.
You should have the a `channels[daphne]` entry in your `dev-requirements.in` file, and you should
[build and install your requirements](customizations.md#python-packages) as needed.

To do this in a non-Docker environment, run:

```
pip-compile requirements/dev-requirements.in
pip install -r requirements/dev-requirements.txt
```

**I'm having another issue deploying to production.**

Since this is a new feature there may be some speed-bumps getting it into production on all platforms.
While every deployment platform is expected to work, it is not possible to test every app/configuration.
So, if you have any issues please reach out over email (cory@saaspegasus.com) or on Slack and I will do my best to help! 
