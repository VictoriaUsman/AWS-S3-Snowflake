
<img width="2642" height="1628" alt="09BA2FBB-C442-4515-95D6-A4A556EAD770" src="https://github.com/user-attachments/assets/807d2545-ccc9-49c0-86ca-fed54e797a64" />
🧱 Architecture Overview

The system follows this flow:
Weather API
     ↓
EventBridge (Schedule)
     ↓
Lambda (Ingestion)
     ↓
DynamoDB (Raw Data)
     ↓
DynamoDB Streams
     ↓
Lambda (Stream Processor)
     ↓
Amazon S3 (Staging)
     ↓
Snowflake (Analytics Tables)

<img width="2352" height="1202" alt="8DBDEC80-EEE5-4304-A90C-1FBC01F6B766" src="https://github.com/user-attachments/assets/038e67b6-1d80-4a2c-bf8a-53f3f260ba7d" />
<img width="2896" height="1566" alt="3C7938D5-8077-4732-BBB1-7FEE4D7C2FD2" src="https://github.com/user-attachments/assets/11f4c068-188d-4689-86ab-f5b07f60e712" />






⚙️ Components
1. Weather API

An external API that provides real-time weather data such as:

Temperature

Humidity

Conditions

City

Timestamps

2. Amazon EventBridge (Scheduler)

Runs on a fixed schedule (e.g. every 5 minutes) and triggers the ingestion Lambda.

Purpose:

Automates data collection

Eliminates manual execution

Ensures consistent ingestion

3. Lambda – Ingestion Layer

This Lambda:

Calls the Weather API

Parses the JSON response

Writes each record into DynamoDB

This Lambda is optimized for:

Short execution

High concurrency

Fault tolerance

4. DynamoDB (Raw Storage)

DynamoDB stores the raw weather data.

Why DynamoDB?

Ultra-fast writes

Fully serverless

Native streaming support

Each row represents a single weather snapshot.

5. DynamoDB Streams

Streams capture every change to the table in real-time.

Whenever a new weather record is inserted:

A stream event is generated

Lambda is triggered automatically

This creates a true streaming pipeline.

6. Lambda – Stream Processor

Triggered by DynamoDB Streams.

This Lambda:

Extracts new records

Converts DynamoDB JSON into a tabular format

Writes the data as CSV files to Amazon S3

This decouples ingestion from analytics.

7. Amazon S3 (Staging Layer)

S3 acts as the data lake staging layer for Snowflake.

Each stream batch is written as:

s3://iantristan-weatherbucket/snowflake/weather_<timestamp>.csv


This provides:

Auditable history

Easy reprocessing

Cheap long-term storage

8. Snowflake Integration

Snowflake uses:

An AWS IAM role

STS (Secure Token Service)

External stage on S3

Snowflake continuously loads new CSVs into analytics tables.

This allows:

SQL analytics

BI tools

Dashboards

Machine learning pipelines

🔐 Security Model

This project uses cross-cloud secure access.

Snowflake assumes an AWS IAM Role

AWS grants Snowflake temporary access via STS

Snowflake can only read from the specific S3 bucket

No access keys are stored anywhere.

📦 What This Architecture Solves
Problem	Solution
Real-time ingestion	DynamoDB Streams + Lambda
Scalable writes	DynamoDB
Analytics-ready format	CSV in S3
Secure cloud access	IAM + STS
Warehouse separation	Snowflake
Zero servers	Fully serverless
🚀 Why This Matters

This architecture matches what is used in:

Netflix

Uber

Airbnb

AWS internal pipelines

It demonstrates:

Event-driven systems

Streaming data engineering

Cloud-native ELT

Enterprise-grade security

📌 Use Cases

Real-time weather dashboards

Historical climate analysis

Forecast modeling

IoT sensor ingestion

Streaming data pipelines

🧠 Tech Stack

AWS Lambda

DynamoDB

DynamoDB Streams

Amazon S3

EventBridge

Snowflake

AWS IAM + STS

Python (boto3, pandas)
