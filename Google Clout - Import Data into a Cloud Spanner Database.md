## Set up - Define variables to given by the lab

Replace values within <> with the values provided by the lab
```
SPANNER_NAME=<spanner instance name>
DATABASE=<database name>
TABLE=<table name>
BUCKET=<bucket name>
```

## Task 1  - Create the Cloud Spanner instance and the Google Standard SQL database in that instance using the names specified above

Run the below command in Cloud Shell to create the spanner instance
```
gcloud spanner instances create $SPANNER_NAME --config=regional-us-central1 \
--description=$SPANNER_NAME --nodes=1
```

Set the default instance of Cloud Spanner to the newly create instance
```
gcloud config set spanner/instance $SPANNER_NAME
```

Create database within the Cloud Spanner Instance
```
gcloud spanner databases create $DATABASE
```

[Create and query a database using gcloud CLI ](https://cloud.google.com/spanner/docs/getting-started/gcloud)

## Task 2  - Create a table in the Cloud Spanner database with the correct name and schema as per the table above

Replace the <table name> with the table valued provided by the lab
```
gcloud spanner databases ddl update $DATABASE \
  --ddl='CREATE TABLE <table name>(
  ShipName STRING(MAX),
  Registry STRING(MAX),
  ShipClass STRING(MAX),
  Description STRING(MAX)
) PRIMARY KEY (Registry);'
```

## Task 3  - Create a Cloud Spanner Import Manifest file named startrek.json and store it in the source Cloud Storage bucket

Run the below command to create the startrek.json file
```
cat > startrek.json
```

Paste the below text into the Cloud Shell
```
{
  "tables": [
    {
      "table_name": <table name>,
      "file_patterns": [
        "gs://<bucket name>/startrek.csv"
      ],
      "columns": [
        {"column_name": "ShipName", "type_name": "STRING"},
        {"column_name": "Registry", "type_name": "STRING"},
        {"column_name": "ShipClass", "type_name": "STRING"},
		{"column_name": "Description", "type_name": "STRING"}
      ]
    }
  ]
}
```
Press ctrl + D to close and save the file

Copy the file to the Cloud Storage Bucket
```
gsutil cp startrek.json $BUCKET
```

[Import and export data in CSV format](https://cloud.google.com/spanner/docs/import-export-csv)
## Task 4  - Create and run a Dataflow template job to successfully import the text data into the Cloud Spanner database

Run the below command to crate the dataflow job. View this in the dataflow product page and wait for it to complete
```
gcloud dataflow jobs run Job --gcs-location gs://dataflow-templates-us-central1/latest/GCS_Text_to_Cloud_Spanner --region us-central1 \
--staging-location $BUCKET/temp --parameters instanceId=$SPANNER_NAME,databaseId=$DATABASE,importManifest=$BUCKET/startrek.json
```

[Cloud Storage Text to Cloud Spanner](https://cloud.google.com/dataflow/docs/guides/templates/provided-batch#gcs_text_to_cloud_spanner)