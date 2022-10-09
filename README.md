# Udacity Cloud Run deployment walkthrough

<!-- TOC -->

- [Udacity Cloud Run deployment walkthrough](#udacity-cloud-run-deployment-walkthrough)
    - [what is this ?](#what-is-this-)
        - [what's in the box ?](#whats-in-the-box-)
        - [prerequisites](#prerequisites)
            - [gcloud CLI and GCP project setup](#gcloud-cli-and-gcp-project-setup)
    - [run and test locally](#run-and-test-locally)
    - [steps to deploy an app' on Cloud Run](#steps-to-deploy-an-app-on-cloud-run)
        - [manual deployment flow: from Docker image to Cloud Run](#manual-deployment-flow-from-docker-image-to-cloud-run)
            - [build and push your API image to Google Cloud Artifact Registry](#build-and-push-your-api-image-to-google-cloud-artifact-registry)
            - [deploy the image on Cloud Run](#deploy-the-image-on-cloud-run)
                - [a live deployment out of the box](#a-live-deployment-out-of-the-box)
        - [automatic deployment flow](#automatic-deployment-flow)
            - [clean up](#clean-up)

<!-- /TOC -->

## what is this ?

This forkable repo aims at giving a walkthrough on how to deploy applications on Google Cloud Run. Here, we just deploy a very simple sample Flask app'.

### what's in the box ?

- an `app.py` containing the app' to deploy
- a `Dockerfile`, containing the image that we are to build from
- the CI/CD build steps in `gcp/cloudbuild.yaml`

### prerequisites

- you use GitHub
- Docker Desktop
- have the `gcloud` CLI installed on your machine, a GCP project, and a Docker repository set up
- OPTIONAL: having bought a domain name, preferably on Google Domains

#### `gcloud` CLI and GCP project setup

- `sudo apt update && sudo apt upgrade`
- [install](https://cloud.google.com/sdk/docs/install#deb) the `gcloud` CLI on your machine or run a `gcloud components update` to update your `gcloud` tools
- have a GCP project ready and make sure billing is enabled for it
- GCP APIs that must be enabled in your project (you can do this from your GCP browser console) =>
  - `Artifact Registry API`
  - `Cloud Build API`
  - `Compute Engine API`
  - `Container Analysis API`
- you may need to [connect your GCP identity/repo to GitHub](https://cloud.google.com/build/docs/automating-builds/github/connect-repo-github)
- initialize `gcloud` CLI => `gcloud init`
- then set the project ID =>  `gcloud config set project PROJECT_ID`
- set your default region, replacing the placeholders (without the `{}`, to replace with the relevant Google Cloud region, for instance `europe-west6`) => `gcloud config set run/region {gCloudRegion}`
- authenticate your local Docker install to Artifact Registry, replacing the placeholders (without the `{}`, to replace with the relevant Google Cloud region) => `gcloud auth configure-docker {gCloudRegion}-docker.pkg.dev`
- create a Docker repository in the artifact registry

## run and test locally

- `docker build -t udacity-cicd-local -f prod.Dockerfile .`
- run and check on localhost => `docker run -e PORT=5000 -p 80:5000 udacity-cicd-local`
- test => `docker run udacity-cicd-local bash -c "python test_flaskr.py"`

## steps to deploy an app' on Cloud Run

### manual deployment flow: from Docker image to Cloud Run

Manual deploys are useful during the development process, if you want to see the results live in dev or staging environment

#### build and push your API image to Google Cloud Artifact Registry

- build and tag the relevant Docker image locally, replacing the placeholders (without the `{}`, to replace with the data of your Google Cloud project) => `docker build -t {gCloudRegion}-docker.pkg.dev/{projectId}/{nameOfTheArtifactRegistryRepo}/{nameOfYourContainer}:{tag} -f ./prod.Dockerfile .`
- push the images to the Artifact Registry, replacing the placeholders (without the `{}`, to replace with the data of your Google Cloud project) => `docker push {gCloudRegion}-docker.pkg.dev/{projectId}/{nameOfTheArtifactRegistryRepo}/{nameOfYourContainer}:{tag}`

#### deploy the image on Cloud Run

europe-west6-docker.pkg.dev/yactouat/udacity-cicd/sampleflask:0.0.1

- to deploy from source, replacing the placeholders (without the `{}`, to replace with the data of your Google Cloud project) => `gcloud run deploy {serviceName-staging|prod} --image={gCloudRegion}-docker.pkg.dev/{projectId}/{nameOfTheArtifactRegistryRepo}/{nameOfYourContainer}:{tag} --port={portOfYourService} --allow-unauthenticated`
- for instance, staging env Cloud Run service name can be `udacity-cicd-staging`
- if you have permissions issues because you are running this for the first time, just wait a few minutes and retry later
- when prompted, allow for unauthenticated invocations if it's a public API
- when app' deployed, the wizard should reveal the service URL that you can visit in your browser

##### a live deployment out of the box

When you deployed your Cloud Run instance, every revision that will make on it will have the same TLS protected URL out of the box ! This auto-generated URL is a safe place to test stuff in real world conditions, even in staging.
For instance, you could deploy a different service for each locale so you can try internationalization without even having to point any domain to the real thing ;)
Add build triggers on top of that and you'll be able to do fairly complex stuff without the headaches ! this is why I like Google Cloud Platform so much :)

### automatic deployment flow

One build trigger to configure in the GCP UI =>

- run on tagging the main branch; tags must follow the pattern `^v(\d+)\.(\d+)\.(\d+)$`
- automated tests are run in the pipeline steps
- to push a tag => `git tag tag_name && git push origin tag_name`
- cloud build steps are specified in the `./cloudbuild.yaml` folder
- all images are sent to GCP's Artifact Registry

#### clean up

- when you first deployed your app', your artifact was sent to Container Registry, this one can be deleted after you have setup the CI/CD pipeline
- remember to delete the initial trigger when you created the service as well !
