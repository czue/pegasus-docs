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

## Officially supported platforms

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

### Other options

The maintainer of Pegasus hosts Pegasus apps on a virtual private server, using gunicorn and nginx.
I'm happy to help support deployments like this, and am hoping to offer more automated tooling for them in the future.
The Django documentation provides a good overview on [how to deploy Django to your own server](https://docs.djangoproject.com/en/3.2/howto/deployment/).

Additionally, Pegasus's [Docker support](/docker/) can be used as a basis for other production environments
that supports container---for example, Google Kubernetes Engine and Amazon ECS.

Please reach out in the Pegasus Slack `#deployment` channel for any help on this!
