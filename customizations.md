# Customizations

This page outlines the basics of customizing Pegasus to meet your application's needs.

## Personalize your landing page

Pegasus ships with a stub landing page, however most projects will want to highly customize their landing page.
Unless you are planning on building a marketing site on a different platform, this is likely one of the first 
things you'll do.

To modify the default landing page, simply edit the `./apps/web/templates/web/landing_page.html` file
and make any customizations you like.

If you want to start from Pegasus's example landing page, then just copy the contents of
`./pegasus/apps/examples/templates/pegasus/examples/example_landing_page.html` into the landing page file.

## Update the logged-in experience

After you've tweaked your landing page, you'll likely want to dive into the nuts and bolts that make up your app.

To modify the logged-in default page, edit the `./apps/web/templates/web/app_home.html` file to your liking.

### Changing the navigation

There are two levels of navigation that ship with Pegasus, the top nav and the sidebar nav. You'll likely want to modify both.

To change the top nav edit the `./apps/web/templates/web/components/top_nav.html` file.

To change the sidebar nav edit the `./apps/web/templates/web/components/app_nav.html` file.

## Python Packages

You'll inevitably want to add new Python packages to your project.

Pegasus uses [pip tools](https://github.com/jazzband/pip-tools) to manage Python dependencies.
This allows for more explicit dependency management than a standard `requirements.txt` file.

You can install pip tools by running `pip install pip-tools` or `pip install -r requirements/dev-requirements.txt`.

Then, to add/modify your requirements, edit `requirements/requirements.in` with the new package and run:

```
pip-compile requirements/requirements.in
pip install -r requirements/requirements.txt
``` 

If you are using Docker in development, you'll also have to [rebuild your containers](/docker/#updating-python-packages).

## Styles

**Customizing styles requires setting up the [front-end build pipeline](/front-end).**

This project uses Bulma's styles by default.
Bulma is very easy to customize to your needs, 
though you can always drop Bulma for your preferred CSS framework.

Then to change styles, edit the generated `site.scss` file in `assets/styles` with any custom styles you want
and run this command to rebuild them:

```bash
npm run build  # or "dev" or "dev-watch"
```

As a simple, impactful change to get started, you can set the following variable to change
the primary color/theme across the site:

```scss
$primary: #2e7636;  // change primary color to green
```

## Javascript

The project uses a webpack build pipeline to compile the javascript files.

The main javascript architecture is a series of page-specific React applications
that access data from Django Rest Framework using [CoreAPI](https://www.coreapi.org/).
