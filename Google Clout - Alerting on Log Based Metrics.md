## Task 1 - Create a log-based metric for the Logging app named Logging Metric
Navigate to Logs Explorer  
From the filters select 'Kubernetes Container'  
![](https://raw.githubusercontent.com/DavidCox88/GoogleClout-Challenge/feature/challenge3/Images/1-logging-kubernetes-container.png)  
The under Pod Names select the entry that starts 'logging-'  
![](https://raw.githubusercontent.com/DavidCox88/GoogleClout-Challenge/feature/challenge3/Images/2-logging.png)  
Click create metric, name the metric as per the lab instructions

## Task 2 - Create a log-based metric for the Crashing app named Crashing Metric .
Repeat the above but under Pod Names select the entry that starts 'crashing-'

## Task 3 - Create an alert for the Crashing metric called Crash Alert . You should use the sum operation with a threshold of 1 to ensure the alert fires as quickly as possible.
On the screen that pops up upon creating the above metric click 'Create a Metrics based alert'  
![](https://raw.githubusercontent.com/DavidCox88/GoogleClout-Challenge/feature/challenge3/Images/3-create-alert.png)  
On configure trigger change the threshold value to 1 and click next 
![](https://raw.githubusercontent.com/DavidCox88/GoogleClout-Challenge/feature/challenge3/Images/4-threshold-value.png)   
Under configure notifications, toggle off 'Use notification channels'  
![](https://raw.githubusercontent.com/DavidCox88/GoogleClout-Challenge/feature/challenge3/Images/5-notifications.png)  
Name the alert as per the lab instructions and click 'Creat Policy'  
![](https://raw.githubusercontent.com/DavidCox88/GoogleClout-Challenge/feature/challenge3/Images/6-alert-name.png)  

## Task 4 - Once an alert has fired, respond to the alert by acknowledging the alert incident in Google Cloud console.
Navigate to the alert page  
Refresh this page until 'Incidents firing' has a 1 and a red error alert 
symbol next to instructions  
![](https://raw.githubusercontent.com/DavidCox88/GoogleClout-Challenge/feature/challenge3/Images/7-incidents-firing.png)  
Click through the incident until you see the crashing alert error. Click the 3 dots and click 'Acknowledge' to complete the lab  
![](https://raw.githubusercontent.com/DavidCox88/GoogleClout-Challenge/feature/challenge3/Images/8-acknowledge-alert.png)