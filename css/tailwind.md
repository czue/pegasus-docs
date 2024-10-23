# Tailwind CSS

As of version 2022.8 Pegasus officially supports [Tailwind CSS](https://tailwindcss.com/) (Version 3).

## Demo
Here's a quick overview of what it looks like:

<div style="position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden; max-width: 100%; height: auto; margin-bottom: 1em;">
    <iframe src="https://www.youtube.com/embed/mNxVfmoDaOQ" frameborder="0" allowfullscreen style="position: absolute; top: 0; left: 0; width: 100%; height: 100%;"></iframe>
</div>

## Development

Because TailwindCSS only includes the styles found in your HTML / JavaScript files, you will need to actively rebuild
your CSS files any time you add new styles/components to your templates.
The easiest way to do this is by running (after installing Node packages):

```
npm run dev-watch
```

Or in Docker:

```
make npm-watch
```

See the [front-end docs](/front-end) for more information about working with these files.

## Customization

Pegasus uses [daisyUI](https://daisyui.com/) to provide default, well-styled components with convenient CSS classes.
Components from daisyUI can be brought in as needed by your app.
A full list of available components can be found at the [daisyUI component library](https://daisyui.com/components/). 


## Changing your theme

DaisyUI offers a number of out-of-the-box themes you can use in your Pegasus app.
To change themes, first make sure the theme is enabled in your `tailwind.config.js` with a `daisyui` section like this:

```
module.exports = {
  //...
  daisyui: {
    themes: ["light", "dark", "cupcake"],
  },
}
```

(The defaults are "light" and "dark".)

After adding your theme, rebuild your front end, and then add a `data-theme` attribute to
your `<html>` declaration in `templates/web/base.html`.

```html
<html data-theme="cupcake"></html>
```

For a list of the available themes, and information about creating your on theme,
see the [daisyUI theme documentation](https://daisyui.com/docs/themes/) and their online [theme generator](https://daisyui.com/theme-generator/).

### Extending themes

If you'd like to extend one of the built-in themes you can do that in your `tailwind.config.js` file.

For example, add a section like this to set custom primary and secondary colors for light and dark mode.

```javascript
daisyui: {
  themes: [
    {
      light: {
        ...require("daisyui/src/theming/themes")["light"],
        primary: "#0c2340",
        secondary: "#bd3039"
      },
      dark: {
        ...require("daisyui/src/theming/themes")["dark"],
        primary: "#bd3039",
        secondary: "#0c2340"
      }
    },
  ],
},
```

## Other products / themes

### Flowbite

[Flowbite](https://flowbite.com/) is a library with many great UI components---most of which are free and open source.

Flowbite can easily be added to a Pegasus project by installing it via `npm` and adding it to your global `site.js` and
`tailwind.config.js` files.

You can use [this git patch file](https://gist.github.com/czue/73cec7ff562aff0943e55538fe9b4550#file-flowbite-patch)
as a starting point and apply it to your project by running:

```
git apply flowbite.patch
```

This demonstrates basic installation and enabling the [datepicker plugin](https://flowbite.com/docs/plugins/datepicker/).

After applying the patch you will need to run `npm install` and `npm run dev-watch` to enable it on all pages.

If you build with the Pegasus examples, you should be able to see the datepicker at [http://localhost:8000/pegasus/forms/](http://localhost:8000/pegasus/forms/).

### Tailwind UI

[Tailwind UI](https://tailwindui.com/) is a great product for building more complex pages, including marketing sites and app UIs.
It another great option for getting help with UI components and pages, and should integrate seamlessly with the current Pegasus templates.

Note that you will have to rebuild styles when adding TailwindUI components, as described in the "Development" section above.

### shadcn

From shadcn's [installation guide](https://ui.shadcn.com/docs/installation/manual):

```bash
npm install tailwindcss-animate class-variance-authority clsx tailwind-merge
```

Install the icon library for your theme of choice. E.g. for New York:

```bash
npm install @radix-ui/react-icons
```

Updated `tsconfig.json` (still figuring out the exact right thing here).

Updated several other files (see commits).

```bash
npx shadcn@latest init
```

Adding a component

```
npx shadcn@latest add tabs
mv components/ui/* assets/javascript/components/ui/
```

Adding everything:

```
npx shadcn@latest add avatar button calendar card command dialog dropdown-menu input label popover select tabs
```

Process:

Cloned shadcn repo.
Copied the dashboard folder into pegasus.
Set up webpack.
Fixed imports.
Had to add react import.
Change Link to a



## Troubleshooting

### Styles aren't working after adding new components

Every time you use a new Tailwind class you need to rebuild your front end as described in the "[Development](#development)" section above.

After doing that, if they are still not showing up, be sure that you have hard-refreshed your browser (Ctrl-Shift-R) on most browers.
You can also disable browser caching when devtools are open by following these instructions
[for Chrome](https://stackoverflow.com/a/23944114/8207) or [for Firefox](https://stackoverflow.com/a/48027947/8207).

If you are building your front end in Docker, be sure to also read the troubleshooting section of the [front end documentation](../front-end.md)
for potential issues with cross-platform compatibility.
