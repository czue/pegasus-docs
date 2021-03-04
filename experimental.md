# Experimental Features

The following features are not yet supported but are in experimental development.
These features might undergo substantial changes before being officially released - or never be released at all.

However, you are welcome to try them and provide feedback!

## Tailwind CSS support

As of version 0.12, it is possible to build Pegasus using [Tailwind CSS](https://tailwindcss.com/) instead of Bulma.
This is particularly helpful if you intend to swap out the front-end for something like
[Tailwind UI](https://tailwindui.com/), and want to have the build pipeline working out-of-the-box.

Be advised - the Tailwind implementation ships with Pegasus is not 100% mobile-friendly,
and has other styling quirks (see below).

### Known issues

Known issues with the Tailwind build:

- Mobile styles are not well supported
- Several examples are not yet properly styled, including the landing page and pricing page

## Using Tailwind

There are two ways to use Tailwind:

To try out the tailwind build, you can [build Pegasus for multiple CSS frameworks](/css/##building-for-multiple-css-frameworks),
then follow the instructions to change the framework to Tailwind in `settings.py`.

If you're sure you want to use tailwind, you can also request access to the preview release
that includes native tailwind support. This can be done by [getting in touch](https://www.saaspegasus.com/support/).
