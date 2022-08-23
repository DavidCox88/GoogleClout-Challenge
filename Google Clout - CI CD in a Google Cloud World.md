
REGION=us-west1
PROJECTID=qwiklabs-gcp-03-e16c695b2d14
CLOUDBUILDSA=1034854940562@cloudbuild.gserviceaccount.com

gcloud beta builds triggers create cloud-source-repositories \
--repo=pop-kustomize \
--branch-pattern=./* \
--build-config=cloudbuild.yaml


gcloud projects add-iam-policy-binding $PROJECTID \
    --member=serviceAccount:$CLOUDBUILDSA \
    --role=roles/clouddeploy.releaser

gcloud projects add-iam-policy-binding $PROJECTID \
    --member=serviceAccount:$CLOUDBUILDSA \
    --role=roles/iam.serviceAccountUser




gcloud source repos clone pop-kustomize --project=$PROJECTID

cd pop-kustomize

sed -i 's/project-id-here/qwiklabs-gcp-03-e16c695b2d14/g' cloudbuild.yaml
sed -i 's/project-id-here/qwiklabs-gcp-03-e16c695b2d14/g' clouddeploy.yaml
sed -i 's/region-here/us-west1/g' cloudbuild.yaml
sed -i 's/region-here/us-west1/g' clouddeploy.yaml


gcloud deploy apply --file=clouddeploy.yaml --region=$REGION


gcloud artifacts repositories create pop-stats \
    --repository-format=docker \
    --location=$REGION


git config --global user.email "david.cox@example.com"
git config --global user.name "David Cox"

git add .
git commit -m 'Updating yaml'
git push origin


Go to cloud build and wait til it finishes
THen go to cloud deploy, wait for deployments in each stage and click promote

gcloud deploy releases promote --release=rel-a6b0fc8 --delivery-pipeline=pop-stats-pipeline --region=$REGION --to-target=staging

gcloud deploy releases promote --release=rel-a6b0fc8 --delivery-pipeline=pop-stats-pipeline --region=$REGION --to-target=prod


gcloud deploy rollouts approve test-rollout --delivery-pipeline=pop-stats-pipeline --release=rel-a6b0fc8 --region=$REGION


gcloud deploy rollouts approve rel-a6b0fc8-to-prod-0001 --delivery-pipeline=test-pipeline --release=test-release --region=us-central1


https://cloud.google.com/build/docs/automating-builds/create-manage-triggers#:~:text=A%20Cloud%20Build%20trigger%20automatically,changes%20that%20match%20certain%20criteria.
https://cloud.google.com/iam/docs/grant-role-console
https://cloud.google.com/sdk/gcloud/reference/artifacts/repositories/create
https://cloud.google.com/sdk/gcloud/reference/deploy/releases/promote