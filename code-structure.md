# Pegasus's Code Structure

## Overall structure

This is the overall structure of a new Pegasus project:

```
{{project_name}}/
├── {{project_name}}
├── apps
│   ├── teams
│   ├── users
│   └── web
├── pegasus
│   ├── apps
│   └── templates
├── requirements
├── assets
│   ├── javascript
│   └── styles
└── static
    ├── css
    ├── images
    └── js
```

The first three directories are Python modules while the last three are not.

## Your `{{project_name}}` module

This is your Django project root directory. 
It's where your settings, root urlconf and `wsgi.py` file will live.

## Your `apps` module

This is where your project's apps will live.
It is pre-populated with a few skeleton apps for you to further customize to your needs.

It is expected that you will heavily add to and change the contents of this module.

The module starts with three apps (two if you aren't using teams).

- `users` is where your user model is defined.
- `teams` is where team models are defined.
- `web` contains utilities and components related to the generic views, layouts and templates

## The `pegasus` module

This is where most of Pegasus's functionality resides, including templates,
many base models, and the example gallery.

In general it is not expected that you'll need to modify much in this module, though feel free to do so!

## The `requirements` folder

This is where you define your project's Python requrements.

Requirements are managed using `pip-tools`. 
For more information on using it see [their documentation](https://github.com/jazzband/pip-tools).

## The `assets` folder

This is where the source files for your site's JavaScript and CSS live.
These files are what you should edit to change your JS and CSS.

See [front-end](/front-end) for more information on how to compile these files.


## The `static` folder

This folder contains your project's static files, including the compiled output files
from the `assets` folder as well as images.
