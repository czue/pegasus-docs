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

**Additionally, if you're using Tailwind, after building your Pegasus project, you will additionally have to set up your
[front-end build pipeline](/front-end) and run the following commands:**

```
npm install
npm run dev
```

These will build the Tailwind CSS files.

### Known issues

Known issues with the Tailwind build:

- Mobile styles are not well supported
- Several examples are not yet properly styled, including the landing page and pricing page
- CSS files are not yet [optimized for production](https://tailwindcss.com/docs/optimizing-for-production)
- CSS files are not bundled with Pegasus, but need to be built locally, as per above.

## Using Tailwind

The only way to use Tailwind is to [build Pegasus for multiple CSS frameworks](/css/),
then follow the instructions to change the framework to Tailwind in `settings.py`.
