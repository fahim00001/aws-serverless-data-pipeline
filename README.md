# AWS Serverless Data Pipeline

A hands-on AWS project that demonstrates how to build a serverless data pipeline using Python, AWS Lambda, Amazon DynamoDB, Amazon S3, and EventBridge Scheduler.

## 1. Project Overview

The objective of this project is to retrieve user data from an external REST API, store it in Amazon DynamoDB, and automatically export the stored records into JSON files in an Amazon S3 bucket.

The project focuses on serverless architecture, AWS service integration, IAM security, automation, and monitoring.

## 2. Architecture

The planned data flow is:

```text
JSONPlaceholder REST API
          |
          | HTTP GET
          v
     AWS Lambda #1
     (Import Users)
          |
          | DynamoDB PutItem
          v
      Amazon DynamoDB
          |
          | DynamoDB Scan
          v
     AWS Lambda #2 <----- EventBridge Scheduler
     (Export Users)        (Every 2 Minutes)
          |
          | S3 PutObject
          v
       Amazon S3
    (JSON Backup Files)

Both Lambda Functions
          |
          v
     Amazon CloudWatch
      (Logs & Metrics)
```

*This represents the intended architecture. Individual components and automated execution will be documented as they are implemented and verified.*

## 3. AWS Services Used

| AWS Service | Responsibility |
|---|---|
| AWS Lambda | Executes Python functions without managing servers |
| Amazon DynamoDB | Stores user records |
| Amazon S3 | Stores JSON export files |
| IAM | Controls service permissions |
| EventBridge Scheduler | Triggers recurring backup exports |
| Amazon CloudWatch | Collects execution logs and metrics |

## 4. Repository Structure

```text
aws-serverless-data-pipeline/
│
├── iam_policies/
│   ├── lambda-backup-export-policy.json
│   └── lambda-import-putitem-policy.json
│
├── lambda_src/
│   ├── backup_users/
│   │   └── lambda_function.py
│   └── import_users/
│       └── lambda_function.py
│
├── lambda_zips/
│   ├── lambda-backup-to-s3.zip
│   └── lambda-import-users.zip
│
└── README.md
```

## 5. Data Source

The project uses the JSONPlaceholder public REST API:

https://jsonplaceholder.typicode.com/users

The API provides sample user records containing information such as names, email addresses, addresses, and company details.

