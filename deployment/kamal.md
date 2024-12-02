## Kamal (Deploy to any VPS)

Pegasus supports container-based deployment to any Linux server using [Kamal](https://kamal-deploy.org/).

Kamal is a deployment tool that uses Docker to deploy applications to servers. It is designed to be simple to use
and to work with a single server or a cluster of servers.

Kamal will deploy the app as Docker containers, and will also deploy the database and any other services that are
required. It will also configure a load balancer ([Traefik](https://traefik.io/) to route traffic to the app
as well as configure SSL certificates using LetsEncrypt.

In the setup here we will run all the services on a single server however Kamal is designed to work with multiple servers,
so you can easily move services to separate servers and update the Kamal configuration in `deploy/config/deploy.yml`.

*Note: Kamal support was added in a recent version of Pegasus. If you run into any issues, please get in touch!*

### Screencast

You can watch a screencast showing how to deploy to a Digital Ocean Droplet with Kamal here:

<div style="position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden; max-width: 100%; height: auto; margin-bottom: 1em;">
    <iframe src="https://www.youtube.com/embed/qOc2hZm2uCA" frameborder="0" allowfullscreen style="position: absolute; top: 0; left: 0; width: 100%; height: 100%;"></iframe>
</div>

Or follow along with the documentation below.

### Overview

Deploying on Kamal will require a few pieces:

1. A server running Linux (the latest Ubuntu LTS is recommended---version 22.04 as of this writing) and accessible via SSH.
3. A domain name for your app. You will need to create a DNS record pointing to your server's IP address.
2. A Docker registry to store your images. You can use [Docker Hub](https://hub.docker.com) or any other registry.
4. A development environment where you install and configure Kamal.

We'll walk through these in more detail in order below.

### Provision and prepare your server

The first step is to provision a server were you will host your application.
Some popular choices include:

1. Hetzner (get €20 credit and support Pegasus with [this link](https://hetzner.cloud/?ref=49vhF1w3TIyB)).
2. Digital Ocean Droplets (get $100 credit and support Pegasus with [this link](https://m.do.co/c/432e3abb37f3)).
3. [Linode](https://www.linode.com/).
4. [AWS](https://aws.amazon.com/) (Lightsail or EC2).
5. [Google Cloud](https://cloud.google.com/).
6. [Microsoft Azure](https://azure.microsoft.com/en-us).

It is recommended to choose the latest Ubuntu LTS---version 22.04 as of this writing---for your operating system.
Other operating systems might work, but are not officially tested.

We also recommend at least 2GB of RAM.

Once you've chosen a hosting company and provisioned a server, follow the instructions provided to login (SSH)
to the server. You will need to be able to log in remotely to complete the rest of the setup.

#### Install Docker

Although Kamal can install Docker for you, it is recommended that you install Docker yourself
so that Kamal does not need to use the root user account---which can expose your server to more attacks.

You can test if Docker is installed by running `docker -v` on the command line. You should see output like
the following if it is installed correctly.

```
Docker version 24.0.5, build 24.0.5-0ubuntu1~20.04.1
```

If you need to install it, you can find instructions in [Docker's documentation](https://docs.docker.com/engine/install/ubuntu/).
You only need to install Docker Engine, not Docker Desktop.

#### Prepare a user account for Kamal

Next, create a user for Kamal to use.
You can choose any username you like. In this example we will use `kamal`.
We'll also add this user to the `docker` group so that Kamal can run docker commands.

First login to your server as a user with root access. Then run the following commands:

```shell
sudo adduser kamal --disabled-password
sudo adduser kamal --add_extra_groups docker
```

Next, add your SSH key to the `kamal` user's `authorized_keys` file so you can login without a password.
If you need to generate an SSH key you can [follow these steps](https://www.digitalocean.com/community/tutorials/how-to-configure-ssh-key-based-authentication-on-a-linux-server):

```shell
sudo mkdir -p /home/kamal/.ssh
sudo cp ~/.ssh/authorized_keys /home/kamal/.ssh/authorized_keys
sudo chown -R kamal:kamal /home/kamal/.ssh
```

Next, test the login works. Exit out of your server and on your *local machine* run:

```shell
ssh kamal@<ip-address>
```

If you've set everything up properly the `kamal` user should be able to login with no password.

Once you're logged in, as a final test, ensure the `kamal` user can run docker commands by running:

```shell
docker run hello-world
```

If the command above completes without error you are ready to go!

#### Prepare Docker for deployment

Next, complete the following steps to get the Docker configuration ready for deployment.
These can be run by the `kamal` user on your remote server.

**Create the Docker Network**

Since we are running the app on a single server we need to use Docker networking to allow the containers to communicate
with each other. This requires a Docker network to be created on the server:

Run the following on your server, replacing `<your_app>` with your app ID/slug:

```shell
docker network create <your_app>-network
```
   
*Note: If you are running services on separate servers, you can skip this step and update the Kamal deploy configuration
to remove the references to the docker network.*

<!---
**Create Media Volume**

*This step is currently only recommended if you want to use Django to serve media files, which is not recommended.
To set up media it is recommended to follow the [media setup instructions here](../configuration.md#storing-media-files).* 

You can serve media directly from Django by using a volume mounted to the web container.
This provides a simple way to allow media files (e.g. profile pictures) to persist between deploys.

Run the following on your server, replacing `<your_app>` with your app ID/slug:

```shell
docker volume create <your_app>-media
```

*Note: If you use S3 or some other storage for media, you can skip this step and update the Kamal
deploy configuration to remove the volume mount (the following lines in `deploy.yml`):*

```
volumes:
  - "<your_app>-media:/code/media"
```

-->

**Create the LetsEncrypt storage**

This is needed if you want Traefik to automatically generate SSL certificates for you (recommended). If not, you can skip
this step and update the Kamal deploy configuration to remove the references to LetsEncrypt
(search for `letsencrypt` and `secure`).

On your server run the following command *as root*. You may need to logout `kamal` and back in to `root`:

```shell
sudo mkdir -p /letsencrypt && sudo touch /letsencrypt/acme.json && sudo chmod 600 /letsencrypt/acme.json
```

### Set up DNS

To set up SSL you will need a DNS record pointing at your sever. Create a new "A" record using
whatever tool you use to manage your DNS, and point it at the IP address of the server you created above.

The most common domain to use is `www.<yourdomain>.com`.

### Create the image repository on Docker Hub

Before doing deployment, you need a place to store your Docker images, also known as a *Docker registry*.
The most popular one is [Docker Hub](https://hub.docker.com/), so we'll use that one, though
you can choose a different one if you want, as described in the [Kamal docs](https://kamal-deploy.org/docs/configuration).

First create an account on [Docker Hub](https://hub.docker.com/) and note your username.

Then create a new repository, choosing a unique name for your app, and marking it "private".

Finally you will need to create an access token. Go to "My Account" --> "Security" and make a new access token,
giving it the default permissions of Read, Write, Delete.
**Save this token somewhere as you will need it in the next step and will only see it once.**

### Install and configure Kamal

Finally, we can set everything up to deploy our production application with Kamal.
If you have a Ruby environment available, you can install Kamal globally with:

```
gem install kamal
```

*Note: you may want to use [`rbenv`](https://github.com/rbenv/rbenv) to manage your environment.*

If you don't have Ruby running you can also use Docker to install Kamal, however this is a slightly more complicated.
See [running Kamal with Docker](#running-kamal-with-docker) below for more details.

#### Create `.env` file in the `deploy` directory

Kamal requires a `.env` file in this folder which will contain all the environment variables needed for deployment.
The `.env` file is not checked into source control. See `deploy/.env.kamal` for the required variables.

*Note that this file is distinct from the `.env` file used for development/production variables for your app.*

```bash
cd deploy
cp .env.kamal .env
```

#### Update the Kamal configuration files

The Kamal configuration is in `deploy/config/deploy.yml`.
You will need to update the following values:

* Docker image repo: `image: <namespace>/<repository-name>` - this is the repository you created above.
  If you're using Docker Hub, the `namespace` will typically be your username.
* Your server IP address (or hostname) `<IP-ADDRESS>` (this value is listed once per service).
* Your app domain name: `Host(``hostname.example.com``)`. This is the DNS endpoint you set up above.
* Docker registry username: `username: <DOCKER REGISTRY USERNAME>` - the username you chose above.

Additionally, in your `deploy/.env` file you should add the following variables:

* Set `KAMAL_REGISTRY_PASSWORD` to the access token value you created above.
* Choose secure, unique, and ideally random values for `POSTGRES_PASSWORD` and `SECRET_KEY`.
* Update the `DATABASE_URL` value (use the same password as `POSTGRES_PASSWORD`).

You can review other settings in `deploy.yml`, but those should be all that you need to set yourself
to do your first deployment.

### Deploy

Finally, we can use Kamal to do the rest of the setup. Run the following on your *local* machine.
The `kamal setup` command must be run from the `deploy` directory:

```bash
cd deploy/
kamal setup
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
 
If everything is set up properly then in five or so minutes you should be able to visit your new application
at the configured domain. You're done!

### Post-deployment steps

Once you've gotten everything set up, head on over to the [production checklist](./production-checklist.md) and
run through everything there.
In particular, you will have to set up media files using an external service like S3.

#### Manage changes after initial deployment

See the `deploy/README.md` file in your project repo for pointers on managing the production environment after
the initial deployment.

The key commands you will likely regularly run are `kamal env push` to update the project environment variables,
and `kamal deploy` to push new releases of your application.

### Settings and Secrets

Kamal builds use the `settings_production.py` file.
You can add settings here, and use environment variables to manage any secrets, following the pattern used
throughout the file. 
If you modify `settings_production.py` (or any other code) you will need to run:

```
kamal deploy
```

To push the changes to your servers.

Secrets should be managed in environment variables.
To add new environment variables you will need to update them in two places:

1. The variable *name* needs to be added to the `env` section at the top of `deploy/config/deploy.yml`.
2. The variable name *and value* needs to be added to `deploy/.env` (the same `.env` file we've been using above).

You can see examples of this for variables like `DATABASE_URL` in those two files.

Once you modify your environment variable files you will need to run:

```
kamal env push
```

To update the variables on the server. 

### Running one-off commands

The easiest way to run one-off commands on your server is to use the `kamal app exec` command.
For example:

```
kamal app exec 'python manage.py bootstrap_subscriptions'
```

If you want an interactive SSH-style shell you can run:

```
kamal app exec -i bash
```

You should now have a shell where you can run any Python/`manage.py` command.

You can also get a database shell by running:

```
kamal app exec 'python manage.py dbshell'
```

Or

```
kamal accessory exec postgres -i 'psql -h localhost -p 5432 -U <youruser>' --reuse
```

For more information see [Kamal commands](https://kamal-deploy.org/docs/commands).

### Configuration

There are a few configuration options you might want to update from the defaults.

**Speeding up builds.**

For convenience, Pegasus ships with the "multiarchitecture" flag enabled.
This allows you to build on one architecture (e.g. the ARM64 used on Macs) and deploy to another (e.g. the x86
architecture used on most Linux servers).

However, this can result in very long build/deploy times.
To speed this up you can update your build configuration
and either remove `multiarch: true` if you are building for the same architecture you are deploying to,
or setting up a remote builder.

The Kamal docs [have extensive guidance on optimizing your build setup](https://kamal-deploy.org/docs/configuration).

### Running Kamal with Docker

Here are the steps to run Kamal with Docker instead of native Ruby.
Note that these steps are slightly different from [the Kamal installation instructions](https://kamal-deploy.org/docs/installation).

First, use one of the following commands to set up an alias for a Kamal shell:

On MacOS: 

```bash
alias kamal-shell='docker run -it --rm -v "${PWD}:/workdir" -v "/run/host-services/ssh-auth.sock:/run/host-services/ssh-auth.sock" -e SSH_AUTH_SOCK="/run/host-services/ssh-auth.sock" --entrypoint "/bin/sh" -v /var/run/docker.sock:/var/run/docker.sock ghcr.io/basecamp/kamal:latest'
```

On Linux:

```bash
alias kamal-shell='docker run -it --rm -v "${PWD}:/workdir" -v "${SSH_AUTH_SOCK}:/ssh-agent" -v /var/run/docker.sock:/var/run/docker.sock -e "SSH_AUTH_SOCK=/ssh-agent" --entrypoint "/bin/sh" ghcr.io/basecamp/kamal:latest'
```

After setting up this alias, enter the Kamal container by running the alias **in your project's *root* directory**:

```
kamal-shell
```

After doing that you should have a shell that looks like this:

```html
/workdir #
```

From there change to the `deploy` directory:

```html
/workdir # cd deploy
/workdir/deploy # 
```

Now you can run kamal commands as normal:

```html
/workdir/deploy # kamal app details
  INFO [e959d464] Running docker ps --filter label=service=yourapp --filter label=role=celery on 5.161.214.37
  INFO [e959d464] Finished in 3.483 seconds with exit status 0 (successful).
App Host: 5.161.214.37
CONTAINER ID   IMAGE                                                                                  COMMAND                  CREATED      STATUS      PORTS     NAMES
a4132d07e3d8   you/yourapp:8f0f9cbe651d0b6ccb939b06a46badb3c521b038_uncommitted_634818cb72f626e6   "celery -A yourapp…"   6 days ago   Up 6 days             yourapp-celery-8f0f9cbe651d0b6ccb939b06a46badb3c521b038_uncommitted_634818cb72f626e6

  INFO [258c111d] Running docker ps --filter label=service=yourapp --filter label=role=web on 5.161.214.37
  INFO [258c111d] Finished in 0.920 seconds with exit status 0 (successful).
App Host: 5.161.214.37
CONTAINER ID   IMAGE                                                                                  COMMAND               CREATED      STATUS                PORTS      NAMES
fe8a5e4e81e1   you/yourapp:8f0f9cbe651d0b6ccb939b06a46badb3c521b038_uncommitted_634818cb72f626e6   "/bin/sh -c /start"   6 days ago   Up 6 days (healthy)   8000/tcp   yourapp-web-8f0f9cbe651d0b6ccb939b06a46badb3c521b038_uncommitted_634818cb72f626e6
```


### Troubleshooting

#### Something went wrong during setup

If the `kamal setup` command fails it should print out the error it got. Once you've resolved it,
you may need to set up the services individually instead of re-running it. You can do that with the commands below:

```shell
# rebuild the PostgreSQL container
kamal accessory reboot postgres

# rebuild the Redis container
kamal accessory reboot redis

# rebuild the Traefik container
kamal traefik reboot

# build the Traefik container (if it didn't succeed the first time)
kamal traefik boot

\# push the .env file to the servers
kamal env push

# deploy the app
kamal deploy
```

If deploy continues to fail, check the logs of your docker container, using:

```shell
kamal app logs
```

#### Resolving `ERROR exec /bin/sh: exec format error` 

If you see this error on your server/logs it is likely that the architecture used to build your image is not the
same as the one running on your server.

Review the `builder` section of your `deploy.yml` file and in particular make sure `multiarch` is set to `true`.
You can also explicitly build the image on the remote server, or set the target architecture using
other `builder` options as described [in the kamal docs](https://kamal-deploy.org/docs/configuration#using-remote-builder-for-native-multi-arch).

#### Resolving `ERROR /bin/sh: 1: /start: not found`

If you see this error on your server/logs it is likely that your `/start` script has the wrong line endings.
This can happen if you edit the `./deploy/docker_startup.sh` file in certain programs on the Windows operating system.

To fix this, change the line endings of the file from CRLF to LF using your preferred text editor (you can Google
or ask ChatGPT how to do this for your specific environment).

#### Health checks are failing because of `ALLOWED_HOSTS`

Kamal runs a "health check" during deploys to ensure your new application is ready to handle requests.
This involves pinging your workers at http://localhost:8000 and waiting for them to respond with a "200 OK" status code.

Since these requests are made via localhost, you must have `localhost` in your `ALLOWED_HOSTS`, in addition to your production
endpoints, or Django will reject the requests. So your setting should look something like:

```yaml
ALLOWED_HOSTS = [
  "example.com",  # use your app's domain here
  "localhost",
]
```

It is recommended to read [the security documentation](https://docs.djangoproject.com/en/5.0/topics/security/#host-headers-virtual-hosting)
for this feature to understand the implications of it being included.


### Deploy with GitHub Actions

As of version 2024.8, your repository also includes a GitHub Actions workflow that can be used to deploy your app to Kamal
(look for it in `.github/workflows/kamal_deploy.yml`). This is configured to use Docker Hub or AWS Elastic Container
Registry as the registry depending on your project configuration. If you are using a different registry you will need
to update the action. 

To configure the workflow you will need to set some secrets in your repository settings. In your repositories settings page,
go to the "Secrets and variables -> Actions" section and add the following secrets to the "Repository Secrets" section:

- `SSH_KEY`
  - A private key you use to SSH into your server. This must be a passwordless key.
- `SSH_KNOWN_HOSTS`
  - The known hosts file for your server. You can generate this by running `ssh-keyscan <your-server-ip>`.

#### Dockerhub

If you are using Dockerhub, you must also set the following secrets:

- `DOCKER_REGISTRY_USERNAME`
  - Your Docker Hub username.
- `DOCKER_REGISTRY_KEY`
  - The Docker Hub access token you created above.

With these secrets set, the GitHub Actions workflow will be able to deploy your app to Kamal. To test it
go to the "Actions" tab in your repository, click on the "Kamal Deploy" workflow, and then click "Run workflow".

#### AWS

Using AWS ECR requires a few extra steps:

##### Update the GitHub Actions workflow

Set the following environment variables in `env` section of the GitHub Actions workflow:

* AWS_ACCOUNT_ID
* AWS_REGION

##### Create an IAM role

Create a new IAM role following the instructions in the [GitHub documentation][1]. The role should be
called `github_deploy`, if you call it something else you will need to update the Github actions workflow.

[1]: https://docs.github.com/en/actions/security-for-github-actions/security-hardening-your-deployments/configuring-openid-connect-in-amazon-web-services

Once you have created the role you will need to give it permissions to push to your ECR repository. You can use the
following policy to do this (replacing `<AWS_ECR_REPO_ARN>` with the ARN of your ECR repository):

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Action": [
                "ecr:BatchCheckLayerAvailability",
                "ecr:CompleteLayerUpload",
                "ecr:InitiateLayerUpload",
                "ecr:PutImage",
                "ecr:UploadLayerPart"
            ],
            "Resource": "<AWS_ECR_REPO_ARN>",
            "Effect": "Allow",
            "Sid": "PushToECR"
        },
        {
            "Action": "ecr:GetAuthorizationToken",
            "Resource": "*",
            "Effect": "Allow",
            "Sid": "GetECRToken"
        }
    ]
}
```

You're now ready to deploy your app to Kamal using GitHub Actions. To test it go to the "Actions" tab in your
repository, click on the "Kamal Deploy" workflow, and then click "Run workflow".

### Cookbooks

#### Changing your site URL

To change your site's URL, do the following:

1. Set up a new DNS endpoint as outlined above.
2. Change all ``Host(`example.com`)`` values in your traefik configuration in `deploy.yml` to the new domain.
3. Update your `ALLOWED_HOSTS` setting / environment variable as needed.
4. Run `kamal traefik reboot`.
4. Run `kamal deploy`

Your app should now be running on your new domain.
