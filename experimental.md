# Experimental Features

The following features are not yet supported but are in experimental development.
These features might undergo substantial changes before being officially released.

However, you are welcome to try them and provide feedback!

## Tailwind CSS support

As of version 0.12, it is possible to build Pegasus using [Tailwind CSS](https://tailwindcss.com/) instead of Bulma.
This is particularly helpful if you intend to swap out the front-end for something like
[Tailwind UI](https://tailwindui.com/), and want to have the build pipeline working out-of-the-box.

Be advised - the Tailwind implementation ships with Pegasus is not 100% mobile-friendly,
and has other styling quirks.

### Building with Tailwind support

There are two options for adding tailwind support:

1. **Build for Tailwind only.** Choose this option if you're sure you want to use Tailwind.
2. **Build for Tailwind and Bulma.** You can toggle between them via a Django setting.
   Choose this option if you want to try out Tailwind.

To build with either of these options,
first create a config file with the following contents:

```yaml
default_context:
  _css_framework: tailwind  # or replace with "multiple" to try both frameworks
```

Then run Pegasus, passing in the config file, as follows:

```
pegasus path/to/pegasus.zip --config-file my-config.yaml
```

**After building your Pegasus project, you will additionally have to set up your
[front-end build pipeline](/front-end) and run the following commands:**

```
npm install
npm run dev
```

These will build the Tailwind CSS files.

If you chose to build for multiple CSS frameworks, you can change from Bulma 
to Tailwind and back by changing `USE_TAILWIND = True` in your `settings.py` file.

### Known issues

Known issues with the Tailwind build:

- Mobile styles are not well supported
- Several examples are not yet properly styled,
  including the landing page and pricing page
- CSS files are not yet [optimized for production](https://tailwindcss.com/docs/optimizing-for-production)
