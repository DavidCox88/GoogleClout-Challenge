## Set up - Define variables given by the lab
Replace values within <> with the values provided by the lab
```
BUCKET=gs://<SCRIPT-BUCKET>
SOURCE=gs://<SOURCE-BUCKET>/deploy-web-server.sh
VM=<WEB_SERVER>
export PROJECTID=$(gcloud info --format='value(config.project)')
SERVICEACCOUNT=web-admin-sa@$PROJECTID.iam.gserviceaccount.com
```

Run the below in cloud shell to impersonate the service account
```
gcloud config set auth/impersonate_service_account $SERVICEACCOUNT
```
[gcloud config set](https://cloud.google.com/sdk/gcloud/reference/config/set)

## Task 1 - Create a new storage bucket called gs://[SCRIPT-BUCKET]
Run the below in cloud shell to create the storage bucket
```
gsutil mb $BUCKET
```
[Create storage buckets](https://cloud.google.com/storage/docs/creating-buckets#storage-create-bucket-cli)

## Task 2 - Copy the startup script deploy-web-server.sh from the storage bucket gs://[SOURCE-BUCKET] to the storage bucket gs://[SCRIPT-BUCKET]
Run the below in Cloud Shell to copy the deploy-web-server.sh file to the newly created storage bucket
```
gsutil cp $SOURCE $BUCKET
```
[Copy files and objects](https://cloud.google.com/storage/docs/gsutil/commands/cp)

## Task 3 - Add a new firewall rule to the default VCP network that allows Instances containing the tag http-serverto respond to web traffic from all addresses
Run the below in Cloud Shell to create the firewall rule
```
gcloud compute firewall-rules create allowhttp --allow=tcp:80 --source-ranges 0.0.0.0/0 --target-tags=http-server
```
[Firewall rules create](https://cloud.google.com/sdk/gcloud/reference/compute/firewall-rules/create)

## Task 4 - Deploy a new virtual machine called WEB_SERVER that is configured to use deploy-web-server.sh as its startup script
Run the below in cloud shell to create the virtual machine. When promoted for the zone press 'y'
```
gcloud compute instances create $VM --tags http-server --metadata startup-script-url=$BUCKET/deploy-web-server.sh
```
[Compute instance create](https://cloud.google.com/sdk/gcloud/reference/compute/instances/create)