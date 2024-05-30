# Airline_Data_Ingestion
An end-to-end data ingestion pipeline for airline data, utilizing various AWS services to process and store flight information efficiently.

## Overview
This project demonstrates an end-to-end data ingestion pipeline for airline data using various AWS services. The pipeline ingests daily flight data, processes it, and stores it in Amazon Redshift. Notifications are sent for both successful and failed data processing events. The project uses the following AWS services:

Amazon S3
AWS EventBridge
AWS Step Functions
AWS Glue
Amazon Redshift
Amazon SNS

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

