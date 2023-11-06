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

### Kamal

(requires Ruby)

```bash
gem install kamal -v '> 1.0.0'
```

### Create `.env` file

Kamal requires a `.env` file in this folder which will contain all the environment variables needed for deployment.
The `.env` file is not checked into source control. See `deploy/.env.kamal` for the required variables.

```bash
cp deploy/.env.kamal deploy/.env
```

### Bootstrap VM

Kamal requires a VM to run on. This can be a local VM or a remote VM. The VM needs to have Docker installed and
accessible via SSH. Although Kamal can install Docker for you, it is recommended that you install Docker yourself
so that Kamal does not need to use the root user account.

See [Docker's documentation](https://docs.docker.com/engine/install/) for instructions on installing Docker.

1. Create a user account for Kamal to use.

    You can choose any username you like. In this example we will use `kamal`.
    We'll also add this user to the `docker` group so that Kamal can run docker commands.

    ```shell
    <remote vm>$ sudo adduser kamal --disabled-password --add_extra_groups docker
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

2. Create the Docker Network

    Since we are running the app on a single server we need to use Docker networking to allow the containers to communicate
    with each other. This requires a Docker network to be created on the server:
    
    ```shell
    <remote vm>$ docker network create wedding_plan
    ```

3. Create Media Volume

   This volume will be mounted to the Django media folder to allow the media to persist between deploys.

    ```shell
    <remote vm>$ docker volume create wedding_plan-media
    ```
    
    If you use S3 or some other storage for media, you can skip this step but you must also update the Kamal
    deploy configuration to remove the volume mount.

4. Create the LetsEncrypt storage

    This is only needed if you want Traefik to automatically generate SSL certificates for you.

    ```shell
    <remote vm>$ mkdir -p /letsencrypt && touch /letsencrypt/acme.json && chmod 600 /letsencrypt/acme.json
    ```
   
5. Update the Kamal configuration

    The Kamal configuration is in `deploy/config/deploy.yml`. You will need to update the following values:

    * Your server IP address `<IP-ADDRESS>`
    * Your app domain name: `Host(`wedding-plan.herokuapp.com`)`
    * Docker registry username: `<DOCKER REGISTRY USERNAME>`
    * LetsEncrypt email: `"<my-devops@example.com>"`

6. Deploy the Docker containers

    Now we can use Kamal to do the rest of the setup:

    ```bash
    <local>$ kamal setup
    ```
    
    This will:
    
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
    
    Voila! All the servers are now serving the app on port 80. If you’re just running a single server, you’re ready to go.
    If you’re running multiple servers, you need to put a load balancer in front of them. For subsequent deploys, or if 
    your servers already have Docker and curl installed, you can just run `kamal deploy`.

#### Troubleshooting

If the setup fails you may need to setup the services individually:

```shell
# rebuild the PostgreSQL container
kamal accessory reboot postgres

# rebuild the Redis container
kamal accessory reboot redis

# rebuild the Traefik container
kamal traefik reboot

# push the .env file to the servers
kamal env push

# deploy the app
kamal deploy
```

If deploy continues to fail, check the logs of your docker container.

## Basic commands

See the `deploy/README.md` file in your project repo.
