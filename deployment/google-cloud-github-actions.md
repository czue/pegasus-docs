 Automating Deployment to Cloud Run using GitHub Actions
 =======================================================

__*The following guide was contributed by Daan Vielen, a member of the Pegasus community.
Any questions or issues using it should be directed to the #deployment channel of the community Slack.*__

This guide walks you through setting up a GitHub Action to automatically deploy your Django application to Google Cloud Run when the tests pass on the `main` branch.

## What You'll Do:

1. Create two Google Cloud service accounts with appropriate permissions: one for deployments and one for running the application.
2. Store GCP credentials and configuration in GitHub Secrets.
3. Write the deployment workflow that triggers upon successful completion of the test workflow.
4. Use Artifact Registry for storing your Docker images.
5. Clearly distinguish between the deployment service account (DEPLOY_SERVICE_ACCOUNT) and the service account used to run the Cloud Run application (SERVICE_ACCOUNT). This distinction ensures security by limiting each service account's access to only the necessary resources.

---

## **Step 1: Create the Google Cloud Service Accounts**

### **1.1 Set Up Your Google Cloud Project**

First, set an environment variable for your Google Cloud project ID and deployment service account email. This ensures that all the commands referencing these values are consistent and reusable.

```bash
# Set your project ID (replace "replace-with-your-project-id" with your actual project ID)
export PROJECT_ID="replace-with-your-project-id"

# Set the deployment service account email and the service account email that will run the Cloud Run application
export DEPLOY_SERVICE_NAME="cloud-run-deployer"
export DEPLOY_SERVICE_ACCOUNT="${DEPLOY_SERVICE_NAME}@${PROJECT_ID}.iam.gserviceaccount.com"

# Set the service account that will run the Cloud Run application
export SERVICE_ACCOUNT="cloud-run-app@$PROJECT_ID.iam.gserviceaccount.com""cloud-run-deployer@$PROJECT_ID.iam.gserviceaccount.com"
```

### **1.2 Create the Deployment Service Account**

We will create a **separate deployment service account** (DEPLOY_SERVICE_ACCOUNT) for the following reasons:

1. **Security**: By using a dedicated service account for deployments, you can limit its permissions to only what's necessary for deploying the application and avoid granting unnecessary access to other resources.
2. **Impersonation**: The deployment service account (used by GitHub Actions) needs to impersonate another service account that is used to **run** the Cloud Run application. This allows it to delegate specific tasks (like database access, secret access, etc.) to that service account.

Run the following command to create the **deployment service account** (DEPLOY_SERVICE_ACCOUNT):

```bash
# Create the deployment service account
gcloud iam service-accounts create $DEPLOY_SERVICE_NAME \
    --description="Service account for deploying to Cloud Run" \
    --display-name="Cloud Run Deployer" \
    --project="$PROJECT_ID"
```

### **1.3 Grant Permissions to the Deployment Service Account**

The deployment service account (used by GitHub Actions to deploy the Cloud Run service) needs permissions to deploy applications and impersonate the service account that runs your Cloud Run application.

Grant the following roles to the **deployment service account** (DEPLOY_SERVICE_ACCOUNT):

```bash
# Grant Cloud Run Admin role (allows deploying to Cloud Run)
gcloud projects add-iam-policy-binding "$PROJECT_ID" \
    --member="serviceAccount:$DEPLOY_SERVICE_ACCOUNT" \
    --role="roles/run.admin"

# Grant Storage Admin role (to push Docker images to Artifact Registry)
gcloud projects add-iam-policy-binding "$PROJECT_ID" \
    --member="serviceAccount:$DEPLOY_SERVICE_ACCOUNT" \
    --role="roles/artifactregistry.repoAdmin"

# Grant Service Account User role (to allow the deployment service account to impersonate the service account that runs the Cloud Run application)
# Only for deploying cloud run
gcloud iam service-accounts add-iam-policy-binding $SERVICE_ACCOUNT \
    --member="serviceAccount:$DEPLOY_SERVICE_ACCOUNT" \
    --role="roles/iam.serviceAccountUser" \
    --project="$PROJECT_ID"

```

In the above command, replace `$SERVICE_ACCOUNT` with the service account that will **run** the Cloud Run application (more details on this later).

### **1.4 Create and Download the Deployment Service Account Key**

Now generate the deployment service account key, which you will upload to GitHub as a secret.

```bash
# Create and download the service account key
gcloud iam service-accounts keys create ~/cloud-run-deployer-key.json \
    --iam-account=$DEPLOY_SERVICE_ACCOUNT \
    --project="$PROJECT_ID"
```

This will generate a JSON key file (cloud-run-deployer-key.json). Save this file as you will need it to authenticate GitHub Actions with Google Cloud.

---

## **Step 2: Store GCP Credentials in GitHub Secrets**

1. Go to your GitHub repository.
2. Navigate to **Settings** > **Secrets and variables** > **Actions**.
3. Click **New repository secret** and add the following secrets:

- **GCP_DEPLOY_SA_KEY**: The content of your cloud-run-deployer-key.json file. Copy the entire contents of the JSON file.
- **DEPLOY_SERVICE_ACCOUNT**: The service account that will **deploy the Cloud Run application**.
- **GCP_PROJECT**: Your Google Cloud project ID (you already set this as `$PROJECT_ID` in the terminal).
- **CLOUDRUN_NAME**: The name of your Cloud Run service.
- **IMAGE_URL**: The URL for your container image. It typically follows the format: REGION-docker.pkg.dev/[PROJECT_ID]/[REPOSITORY]/[IMAGE_NAME]:latest.
- **REGION**: The Google Cloud region where your Cloud Run service is deployed (e.g., us-central1).
- **DATABASE_ADDRESS**: The Cloud SQL instance connection name (format: project:region:instance).
- **APPLICATION_SETTINGS**: Key reference for secrets manager (format: application_settings:latest).
- **SERVICE_ACCOUNT**: The service account that will **run the Cloud Run application**.

---

## **Step 3: Create the deploy.yml File**

Now that the test workflow is in place and runs successfully, we’ll create a deployment workflow that triggers when the tests pass.

In this deployment workflow, we will use the `DEPLOY_SERVICE_ACCOUNT` environment variable in both shell commands and GitHub Actions.

1. In your GitHub repository, create the following directory structure:

```bash
.github/
  workflows/
```

2. Inside the workflows folder, create a file named deploy.yml and add the following content:

```yaml
name: Deploy to Cloud Run

on:
  workflow_run:
    workflows: ["Run Django Tests"]
    types: [completed]
    branches: [main]

jobs:
  on-success:
    if: ${{ github.event.workflow_run.conclusion == 'success' }}
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Authenticate with Google Cloud
        uses: google-github-actions/auth@v1
        with:
          credentials_json: ${{ secrets.GCP_DEPLOY_SA_KEY }}

      - name: Configure Google Cloud
        run: |
          gcloud config set project ${{ secrets.GCP_PROJECT }}
          gcloud config set run/region ${{ secrets.REGION }}
          gcloud config set account ${{ secrets.DEPLOY_SERVICE_ACCOUNT }}

      - name: Authenticate Docker to push images to Artifact Registry
        run: |
          gcloud auth configure-docker ${{ secrets.REGION }}-docker.pkg.dev --quiet

      - name: Build Docker image
        run: |
          docker build -t ${{ secrets.IMAGE_URL }} . -f Dockerfile.web --platform linux/amd64

      - name: Push Docker image to Artifact Registry
        run: |
          docker push ${{ secrets.IMAGE_URL }}

      - name: Deploy to Cloud Run
        run: |
          gcloud run deploy ${{ secrets.CLOUDRUN_NAME }} \
            --region ${{ secrets.REGION }} \
            --update-env-vars DJANGO_SETTINGS_MODULE=agenda.settings_production \
            --image ${{ secrets.IMAGE_URL }} \
            --set-cloudsql-instances ${{ secrets.DATABASE_ADDRESS }} \
            --set-secrets APPLICATION_SETTINGS=${{ secrets.APPLICATION_SETTINGS }} \
            --service-account ${{ secrets.SERVICE_ACCOUNT }} \
            --allow-unauthenticated
```

---

## **Step 4: Test the Deployment Workflow**

Once everything is set up, test the workflow by making a change in the `main` branch or opening a pull request.

1. Push the changes or create a pull request targeting the `main` branch.
2. The `test` workflow will run automatically.
3. If the tests pass successfully, the `deploy.yml` workflow will trigger, deploying the new changes to Cloud Run.
