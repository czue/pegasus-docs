## Kamal Deploy

Pegasus supports container-based deployment to any remove VM using [Kamal](https://kamal-deploy.org/).

Kamal is a deployment tool that uses Docker to deploy applications to servers. It is designed to be simple to use
and to work with a single server or a cluster of servers.

Kamal will deploy the app as Docker containers, and will also deploy the database and any other services that are
required. It will also configure a load balancer ([Traefik](https://traefik.io/) to route traffic to the app
as well as configure SSL certificates using LetsEncrypt.

In the setup here we will run all the services on a single VM however Kamal is designed to work with multiple servers
so you can easily move services to separate servers and update the Kamal configuration in `deploy/config/deploy.yml`.

In the rest of the document, all the `kamal` commands are run from the `deploy` directory.

## Setup

### Install Kamal

If you haven't already, [install Ruby](https://www.ruby-lang.org/en/documentation/installation/) and then run:

```bash
gem install kamal -v '> 1.0.0'
```

### Create `.env` file in the `deploy` directory

Kamal requires a `.env` file in this folder which will contain all the environment variables needed for deployment.
The `.env` file is not checked into source control. See `deploy/.env.kamal` for the required variables.

*Note that this file is distinct from the `.env` file used for development/production variables for your app.*

```bash
cd deploy
cp .env.kamal .env
```

### Prepare your machine

The machine you are going to deploy to needs to be accessible via SSH and have Docker installed.
You also need to setup a user and a Docker network.

#### Install Docker

Although Kamal can install Docker for you, it is recommended that you install Docker yourself
so that Kamal does not need to use the root user account.

You can test if Docker is installed by running `docker -v` on the command line. You should see output like
the following if it is installed correctly.

```
Docker version 24.0.5, build 24.0.5-0ubuntu1~20.04.1
```

If you need to install it, you can find instructions in [Docker's documentation](https://docs.docker.com/engine/install/).

#### Prepare a user account for Kamal

First create a user for Kamal to use.
You can choose any username you like. In this example we will use `kamal`.
We'll also add this user to the `docker` group so that Kamal can run docker commands.

```shell
<remote vm>$ sudo adduser kamal --disabled-password
<remote vm>$ sudo adduser kamal --add_extra_groups docker
```

Now you can add your SSH key to the `kamal` user's `authorized_keys` file:

```shell
<remote vm>$ sudo mkdir -p /home/kamal/.ssh
<remote vm>$ sudo cp ~/.ssh/authorized_keys /home/kamal/.ssh/authorized_keys
<remote vm>$ sudo chown -R kamal:kamal /home/kamal/.ssh
```

Test the login works:

```shell
<local>$ ssh kamal@<ip-address>
```

If you've set everything up properly the user should be able to login.

#### Prepare Docker for deployment

Complete the following steps to get the Docker configuration ready for deployment. 

**Create the Docker Network**

Since we are running the app on a single server we need to use Docker networking to allow the containers to communicate
with each other. This requires a Docker network to be created on the server:

```shell
<remote vm>$ docker network create <your_app>-network
```

**Create Media Volume**

This volume will be mounted to the Django media folder to allow the media to persist between deploys.

```shell
<remote vm>$ docker volume create <your_app>-media
```

If you use S3 or some other storage for media, you can skip this step but you must also update the Kamal
deploy configuration to remove the volume mount (the following lines in `deploy.yml`):

```
volumes:
  - "<your_app>-media:/code/media"
```

**Create the LetsEncrypt storage**

This is needed if you want Traefik to automatically generate SSL certificates for you (recommended).

```shell
<remote vm>$ sudo mkdir -p /letsencrypt && sudo touch /letsencrypt/acme.json && sudo chmod 600 /letsencrypt/acme.json
```

### Create the image repository on Docker Hub

Before doing deployment, you need a place to store your Docker images, also known as a *Docker registry*.
The most popular one is [Docker Hub](https://hub.docker.com/), so we'll use that one, though
you can choose a different one if you want, as described in the [Kamal docs](https://kamal-deploy.org/docs/configuration).

First create an account on [Docker Hub](https://hub.docker.com/) and note your username.

Then create a new repository, choosing a unique name for your app, and marking it "private".

Finally you will need to create an access token. Go to "Account Settings" --> "Security" and make a new access token,
giving it the default permissions of Read, Write, Delete.
**Save this token somewhere as you will need it in the next step and will only see it once.**

### Update the Kamal configuration files

The Kamal configuration is in `deploy/config/deploy.yml`.
You will need to update the following values:

* Docker image repo: `image: <namespace>/<appname>` - this is the repository you created above.
  The `namespace` will typically be your username.
* Your server IP address (or hostname) `<IP-ADDRESS>` (this value is listed once per service).
* Your app domain name: `Host(``hostname.example.com``)`
* Docker registry username: `username: <DOCKER REGISTRY USERNAME>` - the username you chose above.
* LetsEncrypt email: `certificatesResolvers.letsencrypt.acme.email: "<my-devops@example.com>"`

Additionally, in your `deploy/.env` file you should add the following variables:

* Set `KAMAL_REGISTRY_PASSWORD` to the access token value you created above.
* Choose secure, unique, and ideally random values for `POSTGRES_PASSWORD` and `SECRET_KEY`.
* Update the `DATABASE_URL` value (use the same password as `POSTGRES_PASSWORD`).

### Deploy

Finally, we can use Kamal to do the rest of the setup. Run the following on your *local* machine,
from the `deploy` directory:

```bash
<local>$ kamal setup
```
 
This will perform all the tasks necessary to deploy your application
(duplicated below from the [Kamal docs](https://kamal-deploy.org/docs/installation)):
 
* Connect to the servers over SSH
* Log into the registry both locally and remotely
* Build the image using `Dockerfile.web` in the root of the application.
* Push the image to the registry.
* Pull the image from the registry onto the servers.
* Push the `.env` file onto the servers.
* Ensure Traefik is running and accepting traffic on port 80.
* Ensure your app responds with 200 OK to GET /up (you must have curl installed inside your app image!).
* Start a new container with the version of the app that matches the current git version hash.
* Stop the old container running the previous version of the app.
* Prune unused images and stopped containers to ensure servers don’t fill up. 
 
#### Troubleshooting

If the setup fails you may need to set up the services individually:

```shell
# rebuild the PostgreSQL container
kamal accessory reboot postgres

# rebuild the Redis container
kamal accessory reboot redis

# rebuild the Traefik container
kamal traefik reboot

# build the Traefik container (if it didn't succeed the first time)
kamal traefik boot

# push the .env file to the servers
kamal env push

# deploy the app
kamal deploy
```

If deploy continues to fail, check the logs of your docker container.

## Manage changes after initial deployment

See the `deploy/README.md` file in your project repo for pointers on managing the production environment after
the initial deployment.

The key commands you will likely regularly run are `kamal env push` to update the project environment variables,
and `kamal deploy` to push new releases of your application.
