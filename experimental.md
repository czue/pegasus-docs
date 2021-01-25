# Experimental Features

The following features are not yet supported but are in experimental development.
These features might undergo substantial changes before being officially released.

However, you are welcome to try them and provide feedback!

## Bootstrap CSS support

As of version 0.12.1, it is possible to build Pegasus using [Bootstrap CSS (version 5)](https://getbootstrap.com/).
See below for details on building Pegasus for a different CSS framework.

The Bootstrap implementation ships with Pegasus still has quirks.
See below for the list of known issues and limitations that are still to be addressed
with the Bootstrap implementation.

### Known issues

Known larger issues with the Bootstrap build:

- Examples are not yet properly styled
- Teams and invitations UI is not fully styled

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

## Customizing your CSS framework

There are two options for changing Pegasus's CSS framework:

1. **Build for a specific framework.** Choose this option if you're sure you want to use Tailwind or Bootstrap instead of Bulma.
2. **Build for multiple frameworks.** You can toggle between them via a Django setting.
   Choose this option if you want to try out Tailwind or Bootstrap.

To build with either of these options,
first create a config file with the following contents:

```yaml
default_context:
  _css_framework: bootstrap  # or replace with "tailwind" or "multiple"
```

Then run Pegasus, passing in the config file, as follows:

```
pegasus path/to/pegasus.zip --config-file my-config.yaml
```

If you chose to build for multiple CSS frameworks, you can change frameworks
by changing the `CSS_FRAMEWORK` value in your `settings.py` file.
Valid options are `'bulma'`, `'bootstrap'`, nad `'tailwind'`.
