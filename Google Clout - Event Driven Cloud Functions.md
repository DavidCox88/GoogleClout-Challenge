## Set up - Define variables to given by the lab
Replace values within <> with the values provided by the lab, where bucket_name is in the lab instructions and sourcecode_bucket can be found by navigating to Cloud Storage and identifying the bucket called sourcecode_xxxxx
```
BUCKET=gs://<bucket_name>
SOURCE=gs://<sourcecode_bucket>
```

## Task 1  - Create a Cloud Storage bucket named Cloud Function Bucket

Use gsutil to create the storage bucket
```
gsutil mb $BUCKET
```

[Create storage buckets](https://cloud.google.com/storage/docs/creating-buckets#storage-create-bucket-cli)

## Task 2  - Create a Cloud Function using the sample code provided and deploy it to trigger on the google.storage.object.finalize event on the Cloud Storage bucket you created

Make a folder in your cloud shell to contain the function code
```
mkdir function
cd function
```

Copy the source code into the newly created folder
```
gsutil cp $SOURCE/* function
```

Use the code to creaste the cloud_function
```
gcloud functions deploy cloud_function \win
--runtime python39 \
--trigger-resource $BUCKET \
--trigger-event google.storage.object.finalize
```

[cp - Copy files and objects](https://cloud.google.com/storage/docs/gsutil/commands/cp)
[Google Cloud Storage Triggers](https://cloud.google.com/functions/docs/calling/storage)

## Task 3 - Trigger the Cloud Function by copying any file to the Cloud Storage bucket you created

Copy any file into the bucket to test. I'm copying the main.py file from the source code bucket
```
gsutil cp $SOURCE/main.py $BUCKET
```