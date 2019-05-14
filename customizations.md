# Customizations

*This documentation is a work in progress*

## Update project metadata

In the generated `settings.py` file, update all variables in `PROJECT_METADATA` to 
the desired values for your project.

These will be used in the UI (e.g. page titles and social `<meta>` tags), and in 
generated absolute urls.

## Personalize your landing page

Pegasus ships with a stub landing page, however most projects will want to highly customize their landing page.
Unless you are planning on building a marketing site on a different platform, this is likely one of the first 
things you'll do.

To modify the default landing page, simply edit the `./apps/web/templates/web/landing_page.html` file.

You can make any customizations you like.

## Styles

**Customizing styles requires setting up the [front-end build](/front-end).**

This project uses Bulma's styles by default.
Bulma is very easy to customize to your needs, 
though you can always drop Bulma for your preferred CSS framework.

To edit styles first make sure you have configured your [front-end build pipeline](/front-end).

Then to change styles, edit the generated `{{project_name}}.scss` file in `assets/styles` with any custom styles you want
and run this command to rebuild them:

```bash
npm run build  # or "dev" or "dev-watch"
```


## Javascript

The project uses a webpack build pipeline to compile the javascript files.

The main javascript architecture is a series of page-specific React applications
that access data from Django Rest Framework using [CoreAPI](https://www.coreapi.org/).

 
