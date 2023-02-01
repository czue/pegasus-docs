# Customizations

This page outlines the basics of customizing Pegasus to meet your application's needs.

## Personalize your landing page

Pegasus ships with a stub landing page, however most projects will want to highly customize their landing page.
Unless you are planning on building a marketing site on a different platform, this is likely one of the first 
things you'll do.

To modify the default landing page, simply edit the `./templates/web/landing_page.html` file
and make any customizations you like.

If you want to start from Pegasus's example landing page, then just copy the contents of
`./templates/pegasus/examples/example_landing_page.html` into the landing page file.

Note that Pegasus's own landing page is only available in the Bulma CSS version.
For Bootstrap builds we recommend starting from a landing page option
or themes from the web, for example [here](https://themes.getbootstrap.com/)
or [here](https://dev.to/bootstrap/bootstrap-5-templates-91p).

## Update the logged-in experience

After you've tweaked your landing page, you'll likely want to dive into the nuts and bolts that make up your app.

To modify the logged-in default page, edit the `./templates/web/app_home.html` file to your liking.

### Changing the navigation

There are two levels of navigation that ship with Pegasus, the top nav and the sidebar nav.
You'll likely want to modify both.

To change the top nav edit the `./templates/web/components/top_nav.html` file.

To change the sidebar nav edit the `./templates/web/components/app_nav.html` file.

## Python Packages

You'll inevitably want to add new Python packages to your project.

Pegasus uses [pip tools](https://github.com/jazzband/pip-tools) to manage Python dependencies.
This allows for more explicit dependency management than a standard `requirements.txt` file.

You can install pip tools by running `pip install --upgrade pip-tools`.

Then, to add/modify your requirements, edit `requirements/requirements.in` with the new package and run:

```
pip-compile requirements/requirements.in
pip install -r requirements/requirements.txt
``` 

If you are using Docker in development, you'll also have to rebuild your containers.

The easiest way to do everything is to run `make requirements` as [described here](https://docs.saaspegasus.com/docker.html#updating-python-packages).

## Styles

**Customizing styles requires setting up the [front-end build pipeline](/front-end).**

Both Bootstrap and Bulma are designed to be customized to your needs.

To change styles, edit the generated `site-<framework>.scss` file in `assets/styles` with any custom styles you want
and run this command to rebuild them:

```bash
npm run build  # or "dev" or "dev-watch"
```

As a simple, impactful change to get started, you can set the following variable to change
the primary color/theme across the site:

```scss
$primary: #2e7636;  // change primary color to green
```

Just make sure to put this change at the top of your file before the `@import` statements!
More details on customizing themes can be found in the [CSS docs](/css/).

## Javascript

The project uses a webpack build pipeline to compile the javascript files.

For more details on how it works see the [front-end documentation](/front-end/).
