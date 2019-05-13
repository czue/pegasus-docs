# Customizations

*This documentation is a work in progress*

## Update project metadata

In the generated `settings.py` file, update all variables in `PROJECT_METADATA` to 
the desired values for your project.

These will be used in the UI (e.g. page titles and social `<meta>` tags), and in 
generated absolute urls.

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

 
