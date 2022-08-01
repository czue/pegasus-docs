# Bootstrap (and Material)

There are two Bootstrap themes, both of which use Bootstrap version 5.

## Choosing your theme

The default Bootstrap theme is based off the default settings that ship with Bootstrap.
It's recommended if you want a simple, practical starting point or if you want to use your own theme.

The Material theme is based on Creative Tim's [Material Kit](https://www.creative-tim.com/product/material-kit)
and [Material Dashboard](https://www.creative-tim.com/product/material-dashboard) products.
It's flashier than the default theme, though is more opinionated and less compatible with other products.

You can watch a 3-minute tour of the Material theme here:

<div style="position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden; max-width: 100%; height: auto; margin-bottom: 1em;">
    <iframe src="https://www.youtube.com/embed/WwcowKrwCl0" frameborder="0" allowfullscreen style="position: absolute; top: 0; left: 0; width: 100%; height: 100%;"></iframe>
</div>

## Customizing the default theme

Pegasus's file structure is based on [the Bootstrap documentation](https://getbootstrap.com/docs/5.0/customize/sass/#importing).
Any of the variables used in Bootstrap can be changed by modifying the `assets/styles/site-bootstrap.scss` file.

A complete list of available variables can be found in `./node_modules/bootstrap/scss/variables`.

Try adding the following lines to the top of your file to see how it changes things:

```scss
$primary: #2e7636;  // change primary color to green
$body-color: #00008B;  // change main text to blue
```

**You'll have to run `npm run dev` to see the changes take.**
For more details on building the CSS files, see the [front end documentation](/front-end.html).

The [Bootstrap documentation](https://getbootstrap.com/docs/5.0/customize/sass/) has much more detail
on customizing your theme!

## Customizing the Material theme

The Material theme can be customized according to the [Material Dashboard documentation](https://www.creative-tim.com/learning-lab/bootstrap/overview/material-dashboard).
The theme files live in the `assets/material-dashboard` folder.
You can see the modifications that have been made for Pegasus support [on Github here](https://github.com/creativetimofficial/material-dashboard/compare/master...czue:pegasus-tweaks).
In particular, a few bugs have been fixed, and the unused pro files have been removed.

Creative Tim offers pro versions of [Material Dashboard](https://www.creative-tim.com/product/material-dashboard-pro) and
[Material Kit](https://www.creative-tim.com/product/material-kit-pro) which are helpful if you want to have access to more
pages / components. These should integrate seamlessly with the Pegasus theme.