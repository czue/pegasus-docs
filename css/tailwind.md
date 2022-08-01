# Tailwind CSS

As of version 2022.8 Pegasus officially supports [Tailwind CSS](https://tailwindcss.com/) (Version 3).

Pegasus uses [daisyUI](https://daisyui.com/) to provide default, well-styled components with convenient CSS classes.

## Changing your theme

DaisyUI offers a number of out-of-the-box themes you can use in your Pegasus app.
To change themes, just add a `data-theme` attribute to your `<html>` declaration in `templates/web/base.html`.

```html
<html data-theme="cupcake"></html>
```

For a list of the available themes, and information about creating your on theme,
see the [daisyUI theme documentation](https://daisyui.com/docs/themes/) and their online [theme generator](https://daisyui.com/theme-generator/).
