The Front End 
======================

## Architecture

Pegasus's front-end architecture is a hybrid model, with a standalone front-end codebase
that is compiled and served inside Django templates.

The front end uses [Babel](https://babeljs.io/) and [Webpack](https://webpack.js.org/) to
compile the front-end code into bundle files that can be referenced using Django's 
static file system, as represented in the diagram below.

![Build Pipeline](images/js-pipeline-with-django.png)

Pegasus's styles use either the [Bootstrap](https://getbootstrap.com/) or [Bulma](https://bulma.io/) CSS frameworks,
and building the CSS files is included as part of the front-end build pipeline.
For more details on CSS in Pegasus, see the [CSS documentation](/css/).

**For a much more detailed overview of the rationale behind this architecture,
and the details of the set up see the [Modern JavaScript for Django Developers](https://www.saaspegasus.com/guides/modern-javascript-for-django-developers/)
series.**

## Front-end files

The source front-end files live in the `assets` directory, while the compiled files
get created in the `static` directory.

Generally you should only ever edit the front-end files in `assets` directly, 
and compile them using the instructions below.

## Prerequisites to building the front end

To compile the front-end JavaScript and CSS files it's expected that you have installed:

- [Node.js](https://nodejs.org/)
- [NPM](https://docs.npmjs.com/downloading-and-installing-node-js-and-npm)

Pegasus is developed and tested on the latest LTS releases, which (at the time of this writing)
are Node version 16.15.0 and npm 8.5.5
Later versions may work, but aren't well-tested.
Also it's recommended to use [`nvm`](https://github.com/nvm-sh/nvm) to manage different node/npm environments more easily.
`nvm` is essentially `virtualenv` for Node.js/npm.

## Initial setup

Getting started should be as simple as running:

```bash
npm install
```

or in Docker:

```
make npm-install
```

In your project's root directory.
This will install all the dependencies necessary to build the front end.

It will also generate a `package-lock.json` file.
It is recommended that you add this file to source control for consistency across installations.

## Building in Development

Whenever you make modifications to the front-end files you will need to run
the following command to rebuild the compiled JS bundles and CSS files:

```bash
npm run dev
```

You can also set it up to watch for changes by running:

```bash
npm run dev-watch
```

or in Docker:

```
make npm-watch
```

## Building for production

To build for production, run:

```bash
npm run build
```

or in Docker:

```
make npm-build
```

This will compress your files, remove logging statements, etc.

## Long-term best practices

For ease of initial set up, the front-end bundle files are included with the Pegasus codebase.
This allows you to get up and running with Pegasus without having to set up the front-end build pipeline.

However, keeping these files in source control will typically result in a lot of unnecessary changes and merge conflicts.
Instead, it is recommended that you add the compiled CSS and JavaScript bundle files to your `.gitignore`
so they are no longer managed by source control, and have your developers build them locally using the steps above.

For production deployment, see the [production guidance](https://docs.saaspegasus.com/deployment/production-checklist.html#optimize-your-front-end) on this.


## TypeScript and type checking

Since the 2022.6 release, Pegasus includes TypeScript as part of the front end code.
You can write TypeScript or JavaScript code and it will be transpiled to work in a browser as part of the
build pipeline.

The build pipeline does *not* explicitly do type checking.
To do type checking you can run:

```
npm run type-check
```

Or in Docker:

```
npm run type-check-watch
```

Type checks will also automatically run on new pull requests if you have enabled Github Actions on your project.
