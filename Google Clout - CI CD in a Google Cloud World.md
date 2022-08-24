## Set up - Define variables given by the lab
Replace values within <> with the values provided by the lab. For the Cloud Build Service account, naivgate to IAM and find the service account that finishes '@cloudbuild.gserviceaccount.com'
```
REGION=<default region>
PROJECTID=<Project_ID>
CLOUDBUILDSA=<number>@cloudbuild.gserviceaccount.com
```

## Step 1 - Configure Cloud Build
### Create a build trigger that excutes the build defined in the cloudbuild.yaml file that is in the root of the pop-kustomize repository in Cloud Source Repository for the project
Run the following command in Cloud Shell to create the Cloud Build Trigger  
```
gcloud beta builds triggers create cloud-source-repositories \
--repo=pop-kustomize \
--branch-pattern=./* \
--build-config=cloudbuild.yaml
```  
### Add the Cloud Deploy Releaser and Service Account User roles to the Cloud Build service account
Run the following command in Cloud Build to grant the roles to the Cloud Build Service Account:  
```
gcloud projects add-iam-policy-binding $PROJECTID \
    --member=serviceAccount:$CLOUDBUILDSA \
    --role=roles/clouddeploy.releaser

gcloud projects add-iam-policy-binding $PROJECTID \
    --member=serviceAccount:$CLOUDBUILDSA \
    --role=roles/iam.serviceAccountUser
```

## Step 2 - Clone the pop-kustomize repository to Cloud Shell, configure the Cloud Build and Cloud Deploy YAML files, and create the Cloud Deploy pipeline  
Run the following command in Cloud Shell to clone the repo:  
```
gcloud source repos clone pop-kustomize --project=$PROJECTID
```  
### Edit and modify both the cloudbuild.yamland clouddeploy.yaml files as follows:
#### Change all instances of the placeholder text project-id-here in both files to the lab Project ID: Project_ID
#### Change all instances of the placeholder text region-here in both files to the lab region : Default Region
In Cloud Shell, begin by changing directory to the newly clone repo:  
```
cd pop-kustomize
```  
Then run the below sed commands to replace project-id-here and region-here with the values provide by the lab. Make sure to replace the values in <> with the correct ones:  
```
sed -i 's/project-id-here/<Project_ID>/g' cloudbuild.yaml
sed -i 's/project-id-here/<Project_ID>/g' clouddeploy.yaml
sed -i 's/region-here/<default region>/g' cloudbuild.yaml
sed -i 's/region-here/<default region>/g' clouddeploy.yaml
``` 
#### Create the Cloud Deploy pipeline for the lab using the updated clouddeploy.yaml file that has the correct Project ID and Region settings for this lab
Run the following in CLoud Shell to create the Cloud Deploy pipeline
```
gcloud deploy apply --file=clouddeploy.yaml --region=$REGION
```  
## Step 3 - Configure the Artifact Registry and trigger a build by pushing an update to your repo
#### Create a docker format repository in the Artifact Registry named pop-stats to store container images for this pipeline
Run the following in CLoud Shell to create the Artifact Registry  
```
gcloud artifacts repositories create pop-stats \
    --repository-format=docker \
    --location=$REGION
```  
#### Commit the updated yaml files as a change to the application in your local clone of the pop-kustomize git repository and then push those updates to the Cloud Source Repository
In Cloud shell, begin by using git config to create the neccessary user credentials need to commit  
```
git config --global user.email "david.cox@example.com"
git config --global user.name "David Cox"
``` 
Then run the following command to stage the code changes, commit the code changes and then push the code changes to the repo:
```
git add .
git commit -m 'Updating yaml'
git push origin
```
### Step 4 - Promote and approve the Cloud Deploy pipeline stages from test through to production
### Cloud Console
The below instructions show how to do these steps using the CLoud Console
#### Promote the application from Test to Staging
Go to cloud build and wait until the build finishes, this should take around two minutes.
![](https://raw.githubusercontent.com/DavidCox88/GoogleClout-Challenge/feature/challenge7/Images/cloud-build.jpg)  
Go to Cloud Deploy, click the pipeline name and wait for deployments in the test stage to finish. A promote button will now appear, click this to promote to staging.
![](https://raw.githubusercontent.com/DavidCox88/GoogleClout-Challenge/feature/challenge7/Images/cloud-deploy-promote.jpg)  
This will cause a pop up box to appear, click promote
![](https://raw.githubusercontent.com/DavidCox88/GoogleClout-Challenge/feature/challenge7/Images/promote-dialogue.jpg.jpg)  
#### Promote the application from Staging to Production
Similar to above, wait for the deployemnt to finish to staging before clicking the promote button

#### Review and approve the release to Production
After doing the above, you will see 1 pending review in the pipeline between staging and prod. Click this.
![](https://raw.githubusercontent.com/DavidCox88/GoogleClout-Challenge/feature/challenge7/Images/cloud-deploy-review.jpg) 
You will then see the below in which you need to click review
![](https://raw.githubusercontent.com/DavidCox88/GoogleClout-Challenge/feature/challenge7/Images/cloud-deploy-approve.jpg) 
Upon clicking this, the below dialogue box should pop up. Click approve to promote the deployment to prod.
![](https://raw.githubusercontent.com/DavidCox88/GoogleClout-Challenge/feature/challenge7/Images/cloud-deploy-approve-dialogue.jpg) 
### Cloud Shell
The below instructions show how to do these steps using the CLoud Shell
#### Promote the application from Test to Staging
Go to Cloud Build and wait until the build finishes, this should take around two minutes. Copy the Commit ID and in Cloud Shell use it to create the following variable:
```
COMMIT=cd521ee
```
Then run the following command to promote the deployment form release to staging  
```
gcloud deploy releases promote --release=rel-$COMMIT --delivery-pipeline=pop-stats-pipeline --region=$REGION --to-target=staging
``` 
#### Promote the application from Staging to Production
Run the following command to promote the deployment from staging to prod  
```
gcloud deploy releases promote --release=rel-$COMMIT --delivery-pipeline=pop-stats-pipeline --region=$REGION --to-target=prod
```
#### Review and approve the release to Production
Run the following command to approve the deployment to prod  
```
gcloud deploy rollouts approve rel-$COMMIT-to-prod-0001 --delivery-pipeline=pop-stats-pipeline --release=rel-$COMMIT --region=$REGION
```

https://cloud.google.com/build/docs/automating-builds/create-manage-triggers#:~:text=A%20Cloud%20Build%20trigger%20automatically,changes%20that%20match%20certain%20criteria.
https://cloud.google.com/iam/docs/grant-role-console
https://cloud.google.com/sdk/gcloud/reference/artifacts/repositories/create
https://cloud.google.com/sdk/gcloud/reference/deploy/releases/promote




https://cloud.google.com/build/docs/automating-builds/create-manage-triggers#:~:text=A%20Cloud%20Build%20trigger%20automatically,changes%20that%20match%20certain%20criteria.
https://cloud.google.com/iam/docs/grant-role-console
https://cloud.google.com/sdk/gcloud/reference/artifacts/repositories/create
https://cloud.google.com/sdk/gcloud/reference/deploy/releases/promote