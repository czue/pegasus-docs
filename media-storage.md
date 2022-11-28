# Storing media files

SaaS Pegasus ships with the [default Django configuration][default] for storing files that are uploaded by users
such as profile pictures etc.

[default]: https://docs.djangoproject.com/en/4.1/topics/files/

This works well if you have access to persistent storage on your site such as a filesystem or
Docker volume however in many instances you will want to store user content in a separate system
such as S3.

This is very easy to configure using the [django-storages][django-storages] library which provides
storage backends for a variety of different services.

[django-storages]: https://django-storages.readthedocs.io/en/latest/index.html

## Setting up django-storages

1. Install the package

    ```
    pip install django-storages
    ```
   
    You should also add it to your `requirements.in` file and [re-build](customizations.md#python-packages) your `requirements.txt` file.

2. Follow the instructions in the django-storages docs

   Select the provider from the [list of available providers][providers] use and follow the setup instructions.

[providers]: https://django-storages.readthedocs.io/en/latest/index.html
