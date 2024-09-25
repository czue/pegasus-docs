## Deployment Overview

Pegasus---like Django---can be deployed on any standard cloud infrastructure.

The most common ways of deploying Pegasus are:

1. On a raw VPS / Virtual Machine, such Digital Ocean, Linode, or Amazon EC2 or Lightsail
2. On a platform-as-a-service (PaaS) platform, such as Heroku, or PythonAnywhere
3. In a containerized way, using Docker, and (optionally) Kubernetes

Choosing the right deployment architecture involves a complex set of trade-offs, and there's no one-size-fits-all solution.
PaaS and Docker-based solutions tend to be easier to get up and running, but can be more difficult to modify
and are often more expensive at scale.
Meanwhile, setting up a VPS can be error-prone but is a very cost-effective way to deploy small applications.

Much of the choice will also depend on the knowledge and comfort of you/your team with various tools and platforms.
See this [Django Deployment Guide](https://www.saaspegasus.com/guides/django-deployment/) for a big-picture
overview on choosing a deployment strategy.

## Officially supported PaaS platforms

Pegasus ships with configuration files to deploy to select platforms out-of-the-box.
The officially supported platforms are:

- [Render](/deployment/render/) (Python-based)
- [Fly](/deployment/fly/) (Docker-based)
- [Heroku](/deployment/heroku/) (Python or Docker)
- [Digital Ocean App Platform](/deployment/digital-ocean/) (Docker-based)
- [Google Cloud Run](/deployment/google-cloud/) (Docker-based)

Render and Fly are comparable, and are the recommended options for staging sites or MVPs,
since they are easy to set up and have a generous free tier.

If you would like to deploy to a platform that's not listed here, please get in touch on Slack or by emailing 
cory@saaspegasus.com and I'm happy to help!

## Deployment to any VPS

In addition to the above platforms, you can use Pegasus's Kamal deployment support to deploy
your application onto any Linux server, using Docker containers. For more information on deploying to a VPS,
see the [kamal deployment documentation](./kamal.md).

## Other options

If, for whatever reason, you don't want to use the built in Kamal option to deploy to a VPS,
the Django documentation provides a good overview on [how to deploy Django to your own server](https://docs.djangoproject.com/en/stable/howto/deployment/).

Pegasus user [Mitja Martini](https://mitjamartini.com/) has documented how he [deploys his SaaS Pegasus
application to a VPS using Dokku](https://mitjamartini.com/blog/2024/09/22/deploying-django-on-dokku/) (an
open-source, self-hosted PaaS platform).

Pegasus user [Artem Gordinskiy](https://artem.cool/) has documented his experience 
[migrating Pegasus apps from Kamal to Coolify](https://artem.cool/blog/coolify-django/) (another open-source, self-hosted PaaS).


Pegasus's [Docker support](/docker/) can be used as a basis for other production environments
that supports container---for example, Google Kubernetes Engine and Amazon ECS.

Please reach out in the Pegasus Slack `#deployment` channel for any help on this!
