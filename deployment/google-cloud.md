## Google Cloud

Pegasus can be deployed to Google Cloud Run using containers.
*This feature is in beta and Celery is not yet supported.*

To build for Google Cloud, choose the "google_cloud" option when installing Pegasus.
Then follow the steps below to deploy your app.

### Prerequisites

It is recommended to complete steps 1, 2, 3, and 5 of the 
[Django on Cloud Run](https://codelabs.developers.google.com/codelabs/cloud-run-django) guide
(Pegasus handles step 4 for you).
Pegasus generates a script that you can use as a starting point at
`./scripts/google/cloud_run.sh`, however *it is recommended you follow the steps in order*
to troubleshoot.

In particular you *must*:

- Set up your `gcloud` environment, services, and billing
- Set the the `PROJECT_ID` and `REGION` environment variables

Additionally, when completing [step 5](https://codelabs.developers.google.com/codelabs/cloud-run-django#4)
be sure to save your SQL user credentials in  a file named `.env.production` (Pegasus will provide 
a template for you that you can edit).

### Building images for Google Cloud

After settings up Pegasus and configuring your Google cloud environment, database, and secrets,
you can build your container for Google Cloud by running the following command:

```
gcloud builds submit --config cloudmigrate.yaml  --substitutions _REGION=$REGION
```

This will create your image, upload it to Google's container registry, run your database migrations,
and collect your static files.

You can then deploy it to Google Cloud Run by running the following command,
replacing the variables as needed:

```
gcloud run deploy my-project --platform managed --region $REGION --image gcr.io/$PROJECT_ID/{project_slug}-cloudrun --add-cloudsql-instances ${PROJECT_ID}:${REGION}:your_instance  --allow-unauthenticated --set-env-vars=DJANGO_SETTINGS_MODULE={project_slug}.settings_google
```

Pegasus will also generate this command for you as the last line of `./scripts/google/cloud_run.sh`.

### Settings and Secrets

To tell Google Cloud to use your production settings you will need to set the
`DJANGO_SETTINGS_MODULE=[project_slug].settings_google`.
If you use Pegasus's `cloudmigrate.yaml` file this will be handled automatically for you

You can use Google Secret Manager to add additional settings and secrets by adding them
to `.env.production` and uploading it to Secret Manager using:

```
gcloud secrets versions add {project_slug}_settings --data-file .env.production
``` 

See `settings_google.py` for examples of using these secrets in your settings file.
