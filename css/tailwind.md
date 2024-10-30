# Tailwind CSS

As of version 2022.8 Pegasus officially supports [Tailwind CSS](https://tailwindcss.com/) (Version 3).

## Demo and Overview

Here's a quick overview of using TailwindCSS in Pegasus

<div style="position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden; max-width: 100%; height: auto; margin-bottom: 1em;">
    <iframe src="https://www.youtube.com/embed/vPtxNWx21bw" frameborder="0" allowfullscreen style="position: absolute; top: 0; left: 0; width: 100%; height: 100%;"></iframe>
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


### Changing your themes

Pegasus ships with the default DaisyUI light and dark themes which are used for regular and dark mode, respectively.
But DaisyUI offers a number of [out-of-the-box themes](https://daisyui.com/docs/themes/) you can use in your Pegasus app.
To change themes, make sure the theme is enabled in your `tailwind.config.js`'s `daisyui` section,
and specify the default dark mode theme if necessary:

```javascript
module.exports = {
  // changes the themes to "cupcake" and "coffee"
  daisyui: {
    themes: ["cupcake", "coffee"],
  },
  // sets the "coffee" theme as the one used for dark mode
  darkMode: ["class", '[data-theme="coffee"]'],
}
```
After changing these values you will have to [rebuild your front end](../front-end.md#building-in-development).

Finally, you will also have to update the default themes in your `settings.py`:

```
LIGHT_THEME = "cupcake"
DARK_THEME = "coffee"
```

After this, your app should be fully styled in the new themes!

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

### shadcn

[shadcn/ui](https://ui.shadcn.com/) is a React component library for Tailwind.
It includes many out-of-the-box components that you can install and use in your projects.

As of version 2024.11 Pegasus ships with a demo dashboard using shadcn.
To enable the dashboard you have to build with the Tailwind CSS framework and check the "Include Shadcn dashboard"
checkbox in your project settings.

Here's a screenshot:

![Shadcn Demo Dashboard](/images/css/shadcn-demo.png)

The dashboard is [a hybrid single-page React app](https://www.saaspegasus.com/guides/modern-javascript-for-django-developers/integrating-django-react/)
served by Django.
It uses the same colors as the DaisyUI theme, and will update when you change your theme,
and has many interactive components.
However it is *not* connected to any backend data---it is just a UI example.

#### Working with shadcn

The dashboard can be found in `assets/javascript/shadcn-dashboard`.
Shadcn components are stored in the `assets/javascript/components/ui` folder.

Components can be imported in other JavaScript files using the same import path syntax used by the dashboard:

```javascript
import { Button } from "@/components/ui/button"
```

You can use the [shadcn cli](https://ui.shadcn.com/docs/cli) to create components,
however it currently creates them in the wrong folder (this is surprisingly hard to change).
So adding a component is a two step process:

```
npx shadcn@latest add badge
mv components/ui/* assets/javascript/components/ui/
```

After that you should be able to import and use your component in your React code.

### Flowbite

[Flowbite](https://flowbite.com/) is a library with many great UI components---most of which are free and open source.
Also, unlike shadcn, it does *not* use React---making it a great fit for Django templates and htmx projects.

As of version 2024.11 Pegasus ships with the option to enable flowbite, along with a page demonstrating
some sample components.
To enable Flowbite, choose Tailwind CSS and check the "Include Flowbite" checkbox in your project settings.

If you enable this setting, flowbite will automatically be installed and you can drop
flowbite components into any Django template.
The reference page has an example of a few of these components.

### Tailwind UI

[Tailwind UI](https://tailwindui.com/) is a great product for building more complex pages, including marketing sites and app UIs.
It another great option for getting help with UI components and pages, and should integrate seamlessly with the current Pegasus templates.

Note that you will have to rebuild styles when adding TailwindUI components, as described in the "Development" section above.

## Troubleshooting

### Styles aren't working after adding new components

Every time you use a new Tailwind class you need to rebuild your front end as described in the "[Development](#development)" section above.

After doing that, if they are still not showing up, be sure that you have hard-refreshed your browser (Ctrl-Shift-R) on most browers.
You can also disable browser caching when devtools are open by following these instructions
[for Chrome](https://stackoverflow.com/a/23944114/8207) or [for Firefox](https://stackoverflow.com/a/48027947/8207).

If you are building your front end in Docker, be sure to also read the troubleshooting section of the [front end documentation](../front-end.md)
for potential issues with cross-platform compatibility.
