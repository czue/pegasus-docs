Running Pegasus natively / in a Virtualenv
==========================================

These instructions are for installing Pegasus natively in your operating system.
Running natively is more effort to set up than [using Docker](/docker), but makes certain things
like debugging and IDE integration more straightforward.

## Install Prerequisites

If you haven't already, you'll need to install the following.

- [Python 3.8 or higher](https://www.python.org/downloads/) (Python 3.9 is recommended)
- [Virtualenv](https://virtualenv.pypa.io/en/stable/)

*Note: It is possible to use the template without `virtualenv`, 
but it is highly recommended, and these instructions assume you are using it.*

If you're using Postgres, you'll also want to make sure [you have it installed](https://www.postgresql.org/download/).

## Setup a Python 3.8+ virtualenv

See [Using Virtual Environments](/using-virtualenvs/) for details on this process.

## Enter the project directory

```
cd {{ project_name }}
```

You should see a lot of newly created files for your project including a `manage.py` file.

## Install package requirements

```
pip install -r requirements.txt
```

Note: if you have issues installing `psycopg2`, try installing the dependencies outlined in 
[this thread](https://stackoverflow.com/questions/22938679/error-trying-to-install-postgres-for-python-psycopg2) 
(specifically `python3-dev` and `libpq-dev`.

On Macs you may also need to follow the instructions from [this thread](https://stackoverflow.com/a/58722268/8207). And specifically, run:

```
brew reinstall openssl
export LIBRARY_PATH=$LIBRARY_PATH:/usr/local/opt/openssl/lib/
```

## Set up database (Postgres only)

If you installed with Postgres, edit the `DATABASES` value in `{{ project_name }}/settings.py` with
the appropriate details.

You will also need to create a database for your project if you haven't already:

```bash
sudo -u postgres createdb {{ project_name }}
```

## Create database migrations

```bash
python ./manage.py makemigrations
```

## Run database migrations

```bash
python ./manage.py migrate
```

## Run server

```bash
python ./manage.py runserver
```

Go to [http://localhost:8000](http://localhost:8000) and you should see the default Pegasus landing page!

![Landing Page](images/pegasus-landing-page.png)
