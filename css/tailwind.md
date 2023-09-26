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
The easiest way to do this is by running:

```
npm run dev-watch
```

Or in Docker:

```
make npm-watch
```

## Customization

Pegasus uses [daisyUI](https://daisyui.com/) to provide default, well-styled components with convenient CSS classes.
Components from daisyUI can be brought in as needed by your app.
A full list of available components can be found at the [daisyUI component library](https://daisyui.com/components/). 


## Changing your theme

DaisyUI offers a number of out-of-the-box themes you can use in your Pegasus app.
To change themes, just add a `data-theme` attribute to your `<html>` declaration in `templates/web/base.html`.

```html
<html data-theme="cupcake"></html>
```

For a list of the available themes, and information about creating your on theme,
see the [daisyUI theme documentation](https://daisyui.com/docs/themes/) and their online [theme generator](https://daisyui.com/theme-generator/).

## Other products / themes

[Tailwind UI](https://tailwindui.com/) is a great product for building more complex pages, including marketing sites and app UIs.
It another great option for getting help with UI components and pages, and should integrate seamlessly with the current Pegasus templates.

Note that you will have to rebuild styles when adding TailwindUI components, as described in the "Development" section above.

## Troubleshooting

### Styles aren't working after adding new components

Every time you use a new Tailwind class you need to rebuild your front end as described in the "[Development](#development)" section above.

After doing that, if they are still not showing up, be sure that you have hard-refreshed your browser (Ctrl-Shift-R) on most browers.

If you are building your front end in Docker, be sure to also read the troubleshooting section of the [front end documentation](front-end.md)
for potential issues with cross-platform compatibility.
