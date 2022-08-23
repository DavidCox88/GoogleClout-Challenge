
REPO=
REGION=
export PROJECTID=$(gcloud info --format='value(config.project)')

gcloud beta builds triggers create cloud-source-repositories \
--repo=$REPO \
--build-config=cloudbuild.yaml \
--service-account=$PROJECTID@cloudbuild.gserviceaccount.com \


gcloud projects add-iam-policy-binding $PROJECTID \
    --member=serviceAccount:$PROJECTID@cloudbuild.gserviceaccount.com
    --roles/clouddeploy.releaser

gcloud projects add-iam-policy-binding $PROJECTID \
    --member=serviceAccount:$PROJECTID@cloudbuild.gserviceaccount.com
    --roles/iam.serviceAccountUser


git clone $REPO


gcloud deploy apply --file=clouddeploy.yaml --region=$REGION


gcloud artifacts repositories create pop-stats \
    --repository-format=docker


git add .
git commit -m 'Updating yaml'
git push origin









https://cloud.google.com/build/docs/automating-builds/create-manage-triggers#:~:text=A%20Cloud%20Build%20trigger%20automatically,changes%20that%20match%20certain%20criteria.
https://cloud.google.com/iam/docs/grant-role-console
https://cloud.google.com/sdk/gcloud/reference/artifacts/repositories/create