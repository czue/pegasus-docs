# Pegasus's Code Structure

## Overall structure

This is the overall structure of a new Pegasus project:

```
{{project_name}}/
├── {{project_name}}
├── apps
│   ├── subscriptions
│   ├── teams
│   ├── users
│   ├── utils
│   └── web
├── pegasus
│   ├── apps
├── assets
│   ├── javascript
│   └── styles
├── requirements
└── static
    ├── css
    ├── images
    └── js
├── templates
```

The first three directories are Python modules while the remaining ones are not.

## Your `{{project_name}}` module

This is your Django project root directory. 
It's where your settings, root urlconf and `wsgi.py` file will live.

## Your `apps` module

This is where your project's apps will live.
It is pre-populated with Pegasus's default apps for you to further customize to your needs.

The module starts with up to five apps, depending on your configuration.

- `subscriptions` is for functionality related to Stripe subscriptions.
- `users` is where your user models and views are defined.
- `teams` is where team models and views are defined.
- `utils` is a set of functionality shared across the project.
- `web` contains utilities and components related to the generic views, layouts and templates.

## The `pegasus` module

This is where the Pegasus examples live.

In general, it is not expected that you'll need to modify much in this module, though feel free to do so!

## The `requirements` folder

This is where you define your project's Python requirements.

Requirements are managed using `pip-tools`. 
For more information on using it see [their documentation](https://github.com/jazzband/pip-tools).

## The `assets` folder

This is where the source files for your site's JavaScript and CSS live.
These files are what you should edit to change your JS and CSS.

See [front-end](/front-end) for more information on how to compile these files.


## The `static` folder

This folder contains your project's static files, including the compiled output files
from the `assets` folder as well as images.

## The `templates` folder

This folder contains your project's Django templates.
There is one sub-folder for each application that has templates.
The majority of the project's base template layouts are in the `templates/web` folder.

## Code formatting

For projects that have enabled the `Autoformat code` option, the code will have been formatted
using [black](https://black.readthedocs.io/en/stable/) and [isort](https://pycqa.github.io/isort/).

The project will also include [pre-commit](https://pre-commit.com/) as a dependency in the requirements file
as well as the `.pre-commit-config.yaml` file in the root directory. pre-commit is a tool for managing pre-commit
hooks - which can be used to ensure your code matches the correct format when it's committed.

After installing the project dependencies you can install the pre-commit hooks:

```
$ pre-commit install --install-hooks
pre-commit installed at .git/hooks/pre-commit
```

The default configuration that ships with Pegasus will run `isort` and `black` prior to every Git
commit. If there are fixes that are needed you will be notified in the shell output.

### pre-commit Usage

**Manually running hooks**
```shell
# run all hooks against currently staged files
pre--commit run
# run all the hooks against all the files. This is a useful invocation if you are using pre-commit in CI.
pre-commit run --all-files
# run the isort hook against all staged files
pre-commit run isort
```

**Temporarily disable hooks**
See https://pre-commit.com/#temporarily-disabling-hooks

For more information on using and configuring pre-commit check out the 
[pre-commit docs](https://pre-commit.com/#quick-start) 


### Tool configurations
The configuration for the tools can be found in the [`pyproject.toml`][toml] file.

[toml]: https://black.readthedocs.io/en/stable/usage_and_configuration/the_basics.html#what-on-earth-is-a-pyproject-toml-file

**isort**

| Parameter    | Value   |
|--------------|---------|
| Filter Files | `true`  |
| Profile      | `black` | 
| Line Length  | `120`   |

[iSort docs](https://pycqa.github.io/isort/docs/configuration/options)

**black**

| Parameter    | Value   |
|--------------|---------|
| Line Length  | `120`   |

[Black docs](https://black.readthedocs.io/en/stable/usage_and_configuration/the_basics.html)
