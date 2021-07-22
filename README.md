# Deployment of Snowplow on GCP

This repo contains files for automatic deployment of the Snowplow's open source technology (https://github.com/snowplow) on Google Cloud Platform.

Following GCP services have been used:
```
*) Dataflow
*) GKE (load balancing, ingression, compute)
*) GCS
*) Pub/sub
*) BigQuery
*) Deployment Manager
*) Cloud Build
*) Secret Manager
```

### Steps MANUAL
```
*) Connect your github repo to Cloud Build. The name of this github repo will be referenced in the cloudbuild infrastructure code
*) Create a temp bucket named X with path /temp/
*) Allocate a static IP address to be used for collector machine
*) Replace TODO comments with actual values in the source code
*) Push to cloud repo
*) Pull to instance  
*) Run gcloud deployment-manager deployments create dm-cb --config DM_cloudbuild.yaml

Continued:
* for bq loaders, the config files resolver and bigquery_config has to be base64 encoded (cat bigquery_config.json | base64 -w 0) --resolver $(cat iglu_resolver.json | base64 -w 0)
* When enrichment secret is edited, update the beam-enrich deployment
* When prod-dm deployment is redeployed, update sa_json with new service account json
* Set enrichment yaml to Secret Manager
* Set service account json to Secret Manager
```

## Preparations
* Enable required APIs:
```
$ gcloud services enable deploymentmanager.googleapis.com compute.googleapis.com pubsub.googleapis.com container.googleapis.com
sourcerepo.googleapis.com cloudbuild.googleapis.com bigquery.googleapis.com bigquerystorage.googleapis.com iamcredentials.googleapis.com dataflow.googleapis.com storage-component.googleapis.com
```
* Grant the Google API Service Account [PROJECT_NUMBER]@cloudservices.gserviceaccount.com the Owner Role
* Grant to the Cloud Build service account [PROJECT_NUMBER]@cloudbuild.gserviceaccount.com the IAM roles necessary for the steps in your build.
* Connect repo to Cloud Build in each environment

## Setup Steps  

### Instantiate a deployment
Using the create command to instantiate the deployment. Main DM and DM for cloudbuild must be created independently.
```
$ gcloud deployment-manager deployments create <NAME_OF_DEPLOYMENT> --config <RESOURCE_SPECIFICATION>.yaml
$ gcloud deployment-manager deployments create dm --config DM.yaml
$ gcloud deployment-manager deployments create dm-cb --config DM_cloudbuild.yaml
```

### Update the deployment
Use the update command

```
$ gcloud deployment-manager deployments update  --config DM.yaml
```

The above command compares the deployed deployment with the new definition and executes only the needed changes.

### Delete the deployment
```
$ gcloud deployment-manager deployments delete dm
```
