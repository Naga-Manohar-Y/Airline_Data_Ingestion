# Airline_Data_Ingestion
An end-to-end data ingestion pipeline for airline data, utilizing various AWS services to process and store flight information efficiently.

## Overview
This project demonstrates an end-to-end data ingestion pipeline for airline data using various AWS services. The pipeline ingests daily flight data, processes it, and stores it in Amazon Redshift. Notifications are sent for both successful and failed data processing events. The project uses the following AWS services:

* Amazon S3
* AWS EventBridge
* AWS Step Functions
* AWS Glue
* Amazon Redshift
* Amazon SNS

## Workflow
### Data Ingestion:

Flight data arrives daily in an S3 bucket in a Hive-style partition format.
An EventBridge rule captures the new data arrival event and triggers a Step Function.

### Data Orchestration:

The Step Function orchestrates the workflow:
Starts the Glue Crawler to catalog the new daily flight data in S3.
Waits for the Glue Crawler to complete.
Starts a Glue Job to process the flight data.

### Data Processing:

The Glue Job filters flights with delays of more than 60 minutes.
Joins the flight data with airport details using the airport dimension table in Redshift.
Loads the processed data into the flight fact table in Redshift.

### Notifications:

If the Glue Job is successful, a success notification is sent via SNS.
If the Glue Job fails, a failure notification along with the failed data is sent via SNS.

## S3 Bucket Setup
Create an S3 bucket: airline-data-landing-zone-xxx
Create two folders within the bucket:  

1.daily-flights  

2.dim  

Upload airport.csv to the dim folder.

## Redshift Cluster Setup along with Dimensional and Fact tables
Create a Redshift cluster along with IAM roles, username, and password. <br>
Open the Query Editor and connect to the Redshift cluster using the provided credentials. <br>
Create the airlines schema, airport_dim and daily_flights_fact tables using the SQL scripts in redshift_create_table.txt <br>
Copy the airports data to airport_dim table of redshift using copy script in redshift_create_table.txt change the S3 bucket and IAM role of redshift.

## AWS Glue Setup
### Glue Crawlers
Create Three Crawlers:

airport_dim_crawler for the dimension table in Redshift.  
daily_flights_fact_crawler for the fact table in Redshift.  
daily_flights_crawler for the daily flight data in S3.  

### Glue Crawlers Configuration:
Create a JDBC connection to Redshift.  
Ensure Redshift VPC has an S3 endpoint and the security group allows Redshift port in inbound rules.  
Configure Glue connections with Redshift credentials.  
Create Glue roles with full Redshift access and test the connection.  
Create and run the crawlers to catalog the tables in the airline_datamart database.  

## Glue Job
### Create a Glue Job using Visual ETL:
Add data source - Daily_flights_data: Select airline_datamart database and daily_flight table.  
Add a filter condition to filter delayed flights (>= 60 min).  
Source the airport_dims_data from using data source of redshift.  
Join the airport_dims_data with filtered data to add departure airport details (like city, state ) and modify the schema as required.  
Now again join the airport_dims_data with updated data to add arrival airport details (like city, state ) and modify the schema as required.  
Finally add the the data target flights_fact_table.  
Enable job bookmarking and save the job.  

## Step Functions Orchestration
### Configure Amazon EventBridge:
Enable EventBridge for the S3 bucket source.  
In the Step Function, include the 'startCrawler' API with the daily_flights_crawler name.  
Then use 'GetCrawler' to read the status of crawler.  
Add 'Choice' block if the crawler is still running then it waits for 5 seconds and fethces the status. If is completed then it strat the Glue job with 'GlueStartJobRun'.  
If the job fails in any step it directly sends faliure notification. After glue job run completely but the data didn't reach the destination then also it sends failure notification.  
We will check for Glue_Job_Status using 'Choice' block. If succeeded the Success notification else failure notification using SNS to the subscriber.

## Notifications
### Configure SNS:
Set up SNS topics for success and failure notifications.  
Subscribe email addresses to the SNS topics for notifications.

## How to Run
Set up the S3 bucket and upload the airport.csv file.  
Create and configure the Redshift cluster.  
Create Glue Crawlers and run them.  
Create and configure the Glue Job.  
Set up Step Functions and EventBridge rules.  
Subscribe to SNS topics for notifications.  
Trigger the workflow by uploading new flight data to the S3 bucket.  

## Conclusion
This project showcases a comprehensive data ingestion pipeline for airline data using AWS services. The architecture ensures efficient data processing, storage, and notification mechanisms, making it a robust solution for handling daily flight data.



