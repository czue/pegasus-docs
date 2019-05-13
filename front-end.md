Building the Front-End 
======================

The front end code uses webpack to allow for ES6 and JSX javascript templates,
as well as sass/scss style files.

The source front-end files live in the `assets` directory, while the compiled files
get created in the `static` directory.

Generally you should only ever edit the front-end files in `assets` directly, 
and compile them using the instructions below.

## Prerequisites

To compile the front-end JavaScript and CSS files it's expected that you have installed:

- [Node.js](https://nodejs.org/)
- [NPM](https://docs.npmjs.com/downloading-and-installing-node-js-and-npm)

## Initial setup

Getting started should be as simple as running:

```bash
npm install
```

In your project's root directory.

## Development

Whenever you make modifications to the front-end files you will need to run
the following command to rebuild the compiled JS bundles and CSS files:

```bash
npm run dev
```

You can also set it up to watch for changes by running:

```bash
npm run dev-watch
```
