## Set up - Define variables to given by the lab
Replace values within <> with the values provided by the lab
```
SPANNERINSTANCE=<Cloud Spanner Meta Instance>
DATABASE=<Cloud Spanner Meta Database>
REGION=<dynamically selected lab startup>
BQDATASET=<BigQuery Dataset>
export PROJECTID=$(gcloud info --format='value(config.project)')
```

## Task 1  - Create the following Google Cloud database resources:  
### Create a small, single node, Cloud Spanner instance named Cloud Spanner Meta Instance in the dynamically selected lab startup region.
Run the below command in Cloud Shell to create the spanner instance
```
gcloud spanner instances create $SPANNERINSTANCE --config=regional-$REGION \
--description=$SPANNERINSTANCE --nodes=1
```  
### Create a Cloud Spanner database named Cloud Spanner Meta Database in the Cloud Spanner metadata instance you just created.
Create database within the Cloud Spanner Instance using Cloud Shell
```
gcloud spanner databases create $DATABASE --instance=$SPANNERINSTANCE
```  

### Create a BigQuery dataset in your lab project, located in the dynamically selected lab startup region, called BigQuery Dataset .
Create BiqQuery dataset using Cloud Shell
```
bq --location=$REGION mk --project_id=$PROJECTID $BQDATASET
```  
## Task 2  - Create a Cloud Spanner change stream named ordersstream on the orders table described in the introduction. This table was created for you when the lab started, and is initially empty.
Create the change stream using Cloud Shell
```
gcloud spanner databases ddl update orders-db --instance=orders \
--ddl='CREATE CHANGE STREAM ordersstream FOR orders;'
```  
Alternatively, this can be created by going to the Cloud Spanner Instance, Selecting the database, clicking 'Change Streams' from the navigation menu then 'Create Change Stream'

[Create and manage change streams ](https://cloud.google.com/spanner/docs/change-streams/manage)
[gcloud spanner databases ddl update](https://cloud.google.com/sdk/gcloud/reference/spanner/databases/ddl/update)

## Task 3 - Create and run a regional Dataflow job named streamjob to import Cloud Spanner change streams data into BigQuery:
Use Cloud Shell to create the dataflow job
```
gcloud dataflow flex-template run streamjob --template-file-gcs-location gs://dataflow-templates-$REGION/latest/flex/Spanner_Change_Streams_to_BigQuery --region $REGION  --parameters spannerInstanceId=orders,spannerDatabase=orders-db,spannerMetadataInstanceId=$SPANNERINSTANCE,spannerMetadataDatabase=$DATABASE,spannerChangeStreamName=ordersstream,bigQueryDataset=$BQDATASET
```  
Alternatively navigate to Dataflow, click create and enter the following values, ensuring you replace values within <> with the values provided by the lab:
```
Job Name: streamjob 
Regional endpoint: <dynamically selected lab startup>
Cloud Spanner Instance ID: orders
Cloud Spanner database: orders-db
Cloud Spanner metadata instance ID: <Cloud Spanner Meta Instance>
Cloud Spanner metadata database: <Cloud Spanner Meta Database>
Cloud Spanner change stream: ordersstream 
BigQuery dataset: <BigQuery Dataset>
```  
Then click run job

## Task 4 - Insert data into the orders table to trigger the change streams logging process.
Use Cloud Shell to insert random data into the orders table
```
gcloud spanner databases execute-sql orders-db --instance=orders \
    --sql="INSERT orders (OrderID, CustomerID, OrderDate, Price, ProductID) VALUES (1, 2, CURRENT_DATE(), 4, 5)"

gcloud spanner databases execute-sql orders-db --instance=orders \
    --sql="INSERT orders (OrderID, CustomerID, OrderDate, Price, ProductID) VALUES (2, 2, CURRENT_DATE(), 4, 5)"

gcloud spanner databases execute-sql orders-db --instance=orders \
    --sql="INSERT orders (OrderID, CustomerID, OrderDate, Price, ProductID) VALUES (3, 2, CURRENT_DATE(), 4, 5)"
```  
Sit and wait a few minutes for the Dataflow job to process the new data before checking the lab progresses has updated

[Insert, update, and delete data using data manipulation language (DML)](https://cloud.google.com/spanner/docs/dml-tasks)