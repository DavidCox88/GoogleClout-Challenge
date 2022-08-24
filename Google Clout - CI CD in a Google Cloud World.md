## Set up - Define variables given by the lab
Replace values within <> with the values provided by the lab. For the Cloud Build Service account, naivgate to IAM and find the service account that finishes '@cloudbuild.gserviceaccount.com'
```
REGION=<default region>
PROJECTID=<Project_ID>
CLOUDBUILDSA=<number>@cloudbuild.gserviceaccount.com
```

## Step 1 - Configure Cloud Build
### Create a build trigger that excutes the build defined in the cloudbuild.yaml file that is in the root of the pop-kustomize repository in Cloud Source Repository for the project

gcloud beta builds triggers create cloud-source-repositories \
--repo=pop-kustomize \
--branch-pattern=./* \
--build-config=cloudbuild.yaml

### Add the Cloud Deploy Releaser and Service Account User roles to the Cloud Build service account
gcloud projects add-iam-policy-binding $PROJECTID \
    --member=serviceAccount:$CLOUDBUILDSA \
    --role=roles/clouddeploy.releaser

gcloud projects add-iam-policy-binding $PROJECTID \
    --member=serviceAccount:$CLOUDBUILDSA \
    --role=roles/iam.serviceAccountUser

## Step 2 - Clone the pop-kustomize repository to Cloud Shell, configure the Cloud Build and Cloud Deploy YAML files, and create the Cloud Deploy pipeline
gcloud source repos clone pop-kustomize --project=$PROJECTID

### Edit and modify both the cloudbuild.yamland clouddeploy.yaml files as follows:
### Change all instances of the placeholder text project-id-here in both files to the lab Project ID: Project_ID
### Change all instances of the placeholder text region-here in both files to the lab region : Default Region
cd pop-kustomize

sed -i 's/project-id-here/qwiklabs-gcp-00-81f1fade33a2/g' cloudbuild.yaml
sed -i 's/project-id-here/qwiklabs-gcp-00-81f1fade33a2/g' clouddeploy.yaml
sed -i 's/region-here/us-east1/g' cloudbuild.yaml
sed -i 's/region-here/us-east1/g' clouddeploy.yaml

### Create the Cloud Deploy pipeline for the lab using the updated clouddeploy.yaml file that has the correct Project ID and Region settings for this lab
gcloud deploy apply --file=clouddeploy.yaml --region=$REGION

## Step 3 - Configure the Artifact Registry and trigger a build by pushing an update to your repo
#### Create a docker format repository in the Artifact Registry named pop-stats to store container images for this pipeline
gcloud artifacts repositories create pop-stats \
    --repository-format=docker \
    --location=$REGION

#### Commit the updated yaml files as a change to the application in your local clone of the pop-kustomize git repository and then push those updates to the Cloud Source Repository
git config --global user.email "david.cox@example.com"
git config --global user.name "David Cox"

git add .
git commit -m 'Updating yaml'
git push origin

### Step 4 - Promote and approve the Cloud Deploy pipeline stages from test through to production
#### Promote the application from Test to Staging
Go to cloud build and wait til it finishes
THen go to cloud deploy, wait for deployments in each stage and click promote.

Alternatively, look in CLoud Build to get the relase number and run the following in cloud shell

gcloud deploy releases promote --release=rel-baabca2 --delivery-pipeline=pop-stats-pipeline --region=$REGION --to-target=staging

#### Promote the application from Staging to Production
gcloud deploy releases promote --release=rel-baabca2 --delivery-pipeline=pop-stats-pipeline --region=$REGION --to-target=prod

#### Review and approve the release to Production
gcloud deploy rollouts approve rel-baabca2-to-prod-0001 --delivery-pipeline=pop-stats-pipeline --release=rel-baabca2 --region=$REGION




https://cloud.google.com/build/docs/automating-builds/create-manage-triggers#:~:text=A%20Cloud%20Build%20trigger%20automatically,changes%20that%20match%20certain%20criteria.
https://cloud.google.com/iam/docs/grant-role-console
https://cloud.google.com/sdk/gcloud/reference/artifacts/repositories/create
https://cloud.google.com/sdk/gcloud/reference/deploy/releases/promote