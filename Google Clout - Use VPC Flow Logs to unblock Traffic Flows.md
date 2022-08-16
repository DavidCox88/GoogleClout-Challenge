## Set up - Define variables given by the lab
Replace values within <> with the values provided by the lab. For the subnet region, check in the custom-vpc network to find what region they are created in
```
CUSTOMVPC=<custom-vpc>
DATASET=flow_logs
REGION=<subnets region>
export PROJECTID=$(gcloud info --format='value(config.project)')
```

## Task 1 - Configure VPC Flow Logs on the custom-vpc VPC network.
Run the following commands in cloud shell to enable VPC Flow logs for the two subnets
```
gcloud compute networks subnets update subnet1 --enable-flow-logs --region=$REGION
gcloud compute networks subnets update subnet2 --enable-flow-logs --region=$REGION
```
[Enabling VPC Flow Logs for an existing subnet](https://cloud.google.com/vpc/docs/using-flow-logs#enable-logging-existing)

## Task 2 - Redirect VPC Flow Logs to a BigQuery dataset named flow_logs using a sink and identify the blocked traffic between the source VM ( source_vm ) and destination VM ( destination_vm ) in the custom-vpc VPC network.
Run the following command in Cloud Shell to create a BigQuery log sink
```
gcloud logging sinks create BQ_SINK bigquery.googleapis.com/projects/$PROJECTID/datasets/$DATASET --log-filter='resource.type="gce_subnetwork"'
```

Once the logs have started loading into bigquery, query the jsonPayload table. The query should look something like
```
SELECT jsonPayload
FROM `qwiklabs-gcp-00-fcbdc5036b72.flow_logs.compute_googleapis_com_vpc_flows_20220816` LIMIT 1000
```
Within this look for jsonPayload_destination_port. Identify the port numbers that needs to be opened, you should see an 80 (which is the http port) and another value for the udp port (in my case this was 32767)

[Create a sink](https://cloud.google.com/logging/docs/export/configure_export_v2)  
[gcloud logging sinks create](https://cloud.google.com/sdk/gcloud/reference/logging/sinks/create)  
[Viewing logs in BigQuery](https://cloud.google.com/logging/docs/export/using_exported_logs#bigquery-overview)

## Task 3 - Add Firewall rule(s) to allow the blocked traffic to flow.

Run the following in cloud shell to create the firewall rule for http traffic
```
gcloud compute --project=$PROJECTID firewall-rules create http --direction=INGRESS --priority=1000 --network=$CUSTOMVPC --action=ALLOW --rules=tcp:80 --source-ranges=10.0.0.2
```

Run the following in Cloud shell to create the firewall rule of udp traffic. Replace the 32767 with the value from the above
```
gcloud compute --project=$PROJECTID firewall-rules create udp --direction=INGRESS --priority=1000 --network=$CUSTOMVPC --action=ALLOW --rules=udp:32767 --source-ranges=10.0.0.2/32
```
[Firewall rules create](https://cloud.google.com/sdk/gcloud/reference/compute/firewall-rules/create)