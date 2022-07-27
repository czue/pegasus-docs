Choosing a CSS Theme
====================

There are four CSS themes available in Pegasus.

There are two [Bootstrap 5](https://getbootstrap.com/) themes, one based on Creative Tim's [Material Kit](https://www.creative-tim.com/product/material-kit)
and [Material Dashboard](https://www.creative-tim.com/product/material-dashboard) products,
and one that is "unthemed". Additionally, you can use [Bulma](https://bulma.io/) and, experimentally, [Tailwind CSS](https://tailwindcss.com/).

The look and feel of the site is slightly different for each framework, but the overall layout is the same.
Below are screenshots of the app in each of the four themes:

**Tailwind CSS:**

![Tailwind Home](/images/css/tailwind-home.png)

**Bootstrap Material Theme:**

![Material Home](/images/css/material-home.png)

You can also watch [this video](https://www.youtube.com/watch?v=WwcowKrwCl0) for more details.

**Bootstrap Default Theme:**

![Bootstrap Home](/images/css/bootstrap-home.png)

**Bulma:**

![Bulma Home](/images/css/bulma-home.png)

If you're not sure which framework you want to use,
you can change the setting on your project and download multiple copies of the codebase to try out different ones.


## Customizing your CSS theme

### Bootstrap (material)

The Material theme can be customized according to the [Material Dashboard documentation](https://www.creative-tim.com/learning-lab/bootstrap/overview/material-dashboard).

The theme files live in the `assets/material-dashboard` folder. You can see the modifications that have been made for Pegasus support [on Github here](https://github.com/creativetimofficial/material-dashboard/compare/master...czue:pegasus-tweaks).

### Bootstrap (default)

Pegasus's file structure is based on [the Bootstrap documentation](https://getbootstrap.com/docs/5.0/customize/sass/#importing).
Any of the variables used in Bootstrap can be changed by modifying the `assets/styles/site-bootstrap.scss` file.

A complete list of available variables can be found in `./node_modules/bootstrap/scss/variables`.

Try adding the following lines to the top of your file to see how it changes things:

```scss
$primary: #2e7636;  // change primary color to green
$body-color: #00008B;  // change main text to blue
```

You'll have to run `npm run dev` to see the changes take.

The [Bootstrap documentation](https://getbootstrap.com/docs/5.0/customize/sass/) has much more detail
on cusotmizing your theme!

### Bulma

Bulma is readily customizable via [Sass variables](https://bulma.io/documentation/customize/variables/).
Any of the variables used by Bulma can be changed by modifying the `assets/styles/site-bulma.scss` file.

Try adding the following lines to the top of your file to see how it changes things:

```scss
$primary: #2e7636;  // change primary color to green
$body-color: #00008B;  // change main text to blue
```

You'll have to run `npm run dev` to see the changes take.
