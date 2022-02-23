# Experimental Features

The following features are not yet supported but are in experimental development.
These features might undergo substantial changes before being officially released - or never be released at all.

However, you are welcome to try them and provide feedback!

## Tailwind CSS support

As of version 0.12, it is possible to build Pegasus using [Tailwind CSS](https://tailwindcss.com/).
This is particularly helpful if you intend to swap out the front-end for something like
[Tailwind UI](https://tailwindui.com/), and want to have the build pipeline working out-of-the-box.

Be advised - the Tailwind implementation ships with Pegasus is not 100% mobile-friendly,
and has other styling quirks (see below).

Tailwind support is currently on version 2.0, though upgrading to 3.0 is on the near-term roadmap.

### Known issues

Known issues with the Tailwind build:

- Mobile styles are not well supported
- Several examples are not yet properly styled
