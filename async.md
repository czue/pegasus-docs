Async and Websocket Support
===========================

As of version 2023.10, Pegasus provides support [asynchronous support](https://docs.djangoproject.com/en/4.2/topics/async/),
as well as support for websockets via the [channels library](https://channels.readthedocs.io/).

## The Async / Websocket Demo Application

Pegasus includes an optional demo application to demonstrate the asynchronous and socket capabilities.
The demo application is an extension of the demo application that you build while completing the
[channels tutorial](https://channels.readthedocs.io/en/latest/tutorial/index.html).

The demo application uses the [HTMX websockets extension](https://htmx.org/extensions/web-sockets/) to simplify
the implementation. If you prefer not to use HTMX at all, you can change your websocket connection logic
to use vanilla JavaScript instead, as shown in the [channels tutorial here](https://channels.readthedocs.io/en/latest/tutorial/part_2.html#add-the-room-view).

## Websocket urls

Websocket URLs are defined separately from your app's main `urls.py` file.
In Pegasus, the convention is to put your websocket urls in `channels_urls.py`
in your project folder (the same one containing `urls.py`).

Because websocket urls are separate from your main app, and because they follow a different protocol,
they must be referenced as absolute URLs in your front end (including prepending "ws://" or "wss://" depending on whether 
you're using HTTPS).

Pegasus ships with two helper functions you can use to assist with working with URLs, so long as you follow Pegasus conventions.
The `websocket_reverse` function will reverse a relative websocket URL, and the `websocket_absolute_url` function
will turn a relative URL into an absolute webscoket URL based on your Site address and the `USE_HTTPS_IN_ABSOLUTE_URLS` setting.

You can combine these functions like so to pass the URL of a websocket endpoint to a template:

```python
room_ws_url = websocket_absolute_url(websocket_reverse("ws_group_chat", args=[room_id]))
```

You can then use the websocket URL in a template like this (this example uses HTMX web sockets):

```css

```
