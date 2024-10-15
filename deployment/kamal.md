## Kamal (Deploy to any VPS)

Pegasus supports container-based deployment to any Linux server using [Kamal](https://kamal-deploy.org/).

Kamal is a deployment tool that uses Docker to deploy applications to servers. It is designed to be simple to use
and to work with a single server or a cluster of servers.
It can also be used to deploy multiple apps to the same server.

Kamal will deploy the app as Docker containers, and will also deploy the database and any other services that are
required. It will also configure a load balancer ([kamal-proxy](https://github.com/basecamp/kamal-proxy)) to route
traffic to the app and configure SSL certificates using LetsEncrypt.

By default, Pegasus will run all the services on a single server, but Kamal is designed to work with multiple servers,
so you can easily move services to separate servers and update the Kamal configuration in `config/deploy.yml`.

### Screencast

You can watch a screencast showing how to deploy to a Digital Ocean Droplet with Kamal here:

<div style="position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden; max-width: 100%; height: auto; margin-bottom: 1em;">
    <iframe src="https://www.youtube.com/embed/qOc2hZm2uCA" frameborder="0" allowfullscreen style="position: absolute; top: 0; left: 0; width: 100%; height: 100%;"></iframe>
</div>

Or follow along with the documentation below.

### Overview

Deploying on Kamal will require a few pieces:

1. A server running Linux (the latest Ubuntu LTS is recommended---version 24.04 as of this writing) and accessible via SSH.
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

It is recommended to choose the latest Ubuntu LTS---version 24.04 as of this writing---for your operating system.
Other operating systems might work, but are not tested or officially supported.

We also recommend at least 2GB of RAM.

Once you've chosen a hosting company and provisioned a server, follow the instructions provided to login (SSH)
to the server. You will need to be able to log in remotely to complete the rest of the setup.

The rest of these instructions will run kamal as the root user.
If you prefer to run kamal as a different user---which can prevent certain kinds of attacks---see
the note below.

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

If you don't have Ruby running you can also use Docker to install Kamal, by creating an alias command
as described [in the Kamal docs here](https://kamal-deploy.org/docs/installation/).

#### Create `secrets` file in the `.kamal` directory

Kamal expects a `.kamal/secrets` file in this folder which will contain all the environment variables needed for deployment.
The `secrets` file should not be checked into source control. See `.kamal/secrets.example` for the required variables.

```bash
cp .kamal/secrets.example .kamal/secrets
```

#### Update the Kamal configuration files

The Kamal configuration is in `config/deploy.yml`.
You will need to update the following values:

* Docker image repo: `image: <namespace>/<repository-name>` - this is the repository you created above.
  If you're using Docker Hub, the `namespace` will typically be your username.
* Docker registry username: `username: <DOCKER REGISTRY USERNAME>` - the username you chose above.
* Your server IP address (or hostname) `<IP-ADDRESS>` (this value is listed once per service).
* Your app domain name at proxy --> host, if this is not correct.

Additionally, in your `.kamal/secrets` file you should add the following variables:

* Set `KAMAL_REGISTRY_PASSWORD` to the access token value you created above.
* Choose secure, unique, and ideally random values for `POSTGRES_PASSWORD` and `SECRET_KEY`.
* Update the `DATABASE_URL` value (use the same password as `POSTGRES_PASSWORD`).

You can review other settings in `deploy.yml`, but those should be all that you need to set yourself
to do your first deployment.

### Deploy

Finally, we can use Kamal to do the rest of the setup. Run the following on your *local* machine,
from the project root directory.

```bash
kamal setup
```
 
This will perform all the tasks necessary to deploy your application
(duplicated below from the [Kamal docs](https://kamal-deploy.org/docs/installation)):
 
- Connect to the servers over SSH (using root by default, authenticated by your SSH key).
- Install Docker on any server that might be missing it (using get.docker.com): root access is needed via SSH for this.
- Log into the registry both locally and remotely.
- Build the image using the standard Dockerfile in the root of the application.
- Push the image to the registry.
- Pull the image from the registry onto the servers.
- Ensure kamal-proxy is running and accepting traffic on ports 80 and 443.
- Start a new container with the version of the app that matches the current Git version hash.
- Tell kamal-proxy to route traffic to the new container once it is responding with 200 OK to GET /up.
- Stop the old container running the previous version of the app.
- Prune unused images and stopped containers to ensure servers don’t fill up.

If everything is set up properly then in five or so minutes you should be able to visit your new application
at the configured domain. You're done!

### Post-deployment steps

Once you've gotten everything set up, head on over to the [production checklist](./production-checklist.md) and
run through everything there.
In particular, you will have to set up media files using an external service like S3.

#### Manage changes after initial deployment

See the `config/README.md` file in your project repo for pointers on managing the production environment after
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

1. The variable *name* needs to be added to the `env` section at the top of `config/deploy.yml`.
2. The variable name *and value* needs to be added to `.kamal/secrets` (the same `secrets` file we've been using above).

You can see examples of this for variables like `DATABASE_URL` in those two files.

Once you modify your environment variable files you will need to run:

```
kamal env push
kamal app boot
```

To update the variables on the server and restart the app.

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

### Cookbooks

#### Changing your site URL

To change your site's URL, do the following:

1. Set up a new DNS endpoint as outlined above.
2. Change all ``Host(`example.com`)`` values in your traefik configuration in `deploy.yml` to the new domain.
3. Update your `ALLOWED_HOSTS` setting / environment variable as needed.
4. Run `kamal traefik reboot`.
4. Run `kamal deploy`

Your app should now be running on your new domain.

#### Getting a database backup

Here is one way to get a database dump of your server:

First you can run the following command to save a database dump to the *host* machine:

```
kamal accessory exec postgres 'pg_dump -h localhost -p 5432 -U <your_app_user> <your_app_db_name> > db_dump.sql' --reuse
```

This should create a file on the *host* machine at `/home/kamal/db_dump.sql`.

If you want to copy this file locally, you can run:

```
scp kamal@yourapp.com:db_dump.sql ./
```

Note: you may want to zip or gzip this file first if you have a large database.

#### Doing a database restore

To restore a database you first put the backup file on the host:

```
scp ./db_dump.sql kamal@yourapp.com:
```

Then create the DB:

```
kamal accessory exec postgres 'createdb -h localhost -p 5432 -U <your_app_user> <your_app_db_name>' --reuse
```

After that you will need to login to the *host* machine:

```
ssh kamal@yourapp.com
```

And copy the database dump onto the DB machine.
Run `docker ps` to get the container id of the DB machine. Then run: 

```
docker cp db_dump.sql <CONTAINER ID>:/tmp/db_dump.sql
```

Finally, login to the DB container:

```
docker exec -it <CONTAINER ID> /bin/bash
```

And restore the data:
```
psql -h localhost -p 5432 -U <your_app_user> <your_app_db_name> < /tmp/db_dump.sql
```

#### Deploying multiple apps to the same server

One of the major benefits of the VPS-based approach is that you can easily host multiple apps on the same hardware,
which is usually a substantial cost advantage over hosting each one on its own.

This is now supported out of the box by Kamal and Pegasus.
To deploy multiple applications to the same server, just set up Kamal individually for each application
and run through the steps above.
Once multiple sites are set up, `kamal-proxy` will automatically route traffic to the right app based
on the site URL.

#### Running Docker as a non-root user

Follow these steps if you don't want to run kamal and Docker as the root user.

##### Manually Install Docker

If you don't run kamal as root you'll have to install Docker yourself.

You can test if Docker is installed by running `docker -v` on the command line. You should see output like
the following if it is installed correctly.

```
Docker version 24.0.5, build 24.0.5-0ubuntu1~20.04.1
```

If you need to install it, you can find instructions in [Docker's documentation](https://docs.docker.com/engine/install/ubuntu/).
You only need to install Docker Engine, not Docker Desktop.

##### Prepare a user account for Kamal

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

Finally, update your `config/deploy.yml` file to specify a different user by adding
an ssh section, as [described in the docs](https://kamal-deploy.org/docs/configuration/ssh/#the-ssh-user):

```yaml
ssh:
  user: kamal
```
