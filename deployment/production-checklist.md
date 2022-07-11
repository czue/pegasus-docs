# Production checklist

The following are some recommendations for deploying production Pegasus applications.

## Run the Django deployment checklist

Django provides a [deployment checklist](https://docs.djangoproject.com/en/3.2/howto/deployment/checklist/) that
helps ensure your site has some of the most important settings properly configured for production environments.
It is executed by running `manage.py check --deploy` on your production server.

It's recommended to run this on your production application and address any critical issues.

The default Pegasus configuration will contain some warnings, to help prevent misconfigurations which
can affect your site's availability. Not all warnings are serious issues and some may not be possible to address 
(e.g. if part of your site must be available over HTTP instead of HTTPS).
After running the `manage.py check --deploy` command you should read through the documentation for any issues you get
and update the relevant settings where necessary. 

## Keep your secrets secret

Your secrets---like production database passwords or Stripe API keys---should never be checked into source control.
Instead, you should either put secrets in environment variables or in a .gitignored production settings file that
lives only on the server.
For supported deployment platforms, recommendations for managing secrets can be found in the platform-specific documentation.

## Optimize your front end

The front-end files that ship with Pegasus are the developer-friendly versions.
In production, these should be optimized.

First you should add the compiled files to your `.gitignore` as described in the [front end docs](https://docs.saaspegasus.com/front-end.html#long-term-best-practices).
Then, as part of your CI/CD deployment process, you should build the bundle files directly on your production server 
(using `npm install && npm run build`).

This will ensure that the latest, optimized version of the front-end code is always deployed
as part of your production environment.

The platform-specific docs have some guidance on setting this up where possible.
