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

_This represents the intended architecture. Individual components and automated execution will be documented as they are implemented and verified._

## 3. AWS Services Used

| AWS Service           | Responsibility                                     |
| --------------------- | -------------------------------------------------- |
| AWS Lambda            | Executes Python functions without managing servers |
| Amazon DynamoDB       | Stores user records                                |
| Amazon S3             | Stores JSON export files                           |
| IAM                   | Controls service permissions                       |
| EventBridge Scheduler | Triggers recurring backup exports                  |
| Amazon CloudWatch     | Collects execution logs and metrics                |

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

## 6. Implementation Screenshots

This section documents the AWS infrastructure implementation through screenshots captured during the development process.

### 6.1 Amazon DynamoDB Table Creation

Amazon DynamoDB is used as the database layer of this project to store user records retrieved from the JSONPlaceholder REST API.

**Table Configuration**

| Configuration | Value                   |
| ------------- | ----------------------- |
| Table Name    | `jsonplaceholder-users` |
| Partition Key | `id`                    |
| Key Type      | Number                  |
| Capacity Mode | On-demand               |

**Implementation Screenshot**

![DynamoDB Table Configuration](docs/implementation-screenshots/01-dynamodb_table.png)

### 6.2 Amazon S3 Bucket Creation

Amazon S3 is used as the object storage layer of this project. It will store JSON backup files exported from the DynamoDB table by the second AWS Lambda function.

**Bucket Configuration**

| Configuration       | Value           |
| ------------------- | --------------- |
| Bucket Name         | jsonplaceholder |
| AWS Region          | us west -2      |
| Bucket Type         | General Purpose |
| Block Public Access | Enabled         |

**Bucket Creation Screenshot**

![S3 Bucket Creation](docs/implementation-screenshots/02-s3-bucket-created.png)

**Security Configuration**

The bucket security settings were configured to prevent public access and protect the stored backup data.

![S3 Security Configuration](docs/implementation-screenshots/03-s3-security-configuration.png)

### 6.3 IAM Execution Role for the Import Lambda

I created `lambda-import-users-role` for the Lambda function that will import user records from JSONPlaceholder into DynamoDB. The function will use this role to access AWS services without storing IAM user access keys in its code.

**Role Configuration**

| Setting                    | Value                          |
| -------------------------- | ------------------------------ |
| Role name                  | `lambda-import-users-role`     |
| Trusted service            | AWS Lambda                     |
| Logging policy             | `AWSLambdaBasicExecutionRole`  |
| DynamoDB inline policy     | `lambda-import-putitem-policy` |
| Allowed DynamoDB operation | `PutItem`                      |
| Target table               | `jsonplaceholder-users`        |
| Table Region               | `us-west-2`                    |

The inline policy restricts DynamoDB writes to the `jsonplaceholder-users` table. `PutItem` can create a new record or replace an existing record with the same primary key. The separate logging policy allows the function to write execution logs to CloudWatch Logs.

**Execution Role Screenshot**

![Import Lambda execution role and attached policies](docs/implementation-screenshots/04-lambda-import-role.png)

**DynamoDB Policy Screenshot**

### 6.4 Import Lambda Deployment and DynamoDB Data Insertion

After configuring the IAM execution role, I created and deployed the `lambda-import-users` function using Python 3.13.

The function retrieves user records from the JSONPlaceholder REST API and stores them in the `jsonplaceholder-users` DynamoDB table.

**Lambda Configuration**

| Configuration | Value |
|---|---|
| Function name | `lambda-import-users` |
| Runtime | Python 3.13 |
| Architecture | ARM64 |
| Handler | `lambda_function.lambda_handler` |
| Execution role | `lambda-import-users-role` |
| Environment variable | `TABLE_NAME=jsonplaceholder-users` |
| Memory | 128 MB |

**Execution Process**

1. The Lambda function sends an HTTP GET request to JSONPlaceholder.
2. It receives and validates the JSON user records.
3. It connects to DynamoDB using the configured execution role.
4. It inserts each user record using the `PutItem` operation.
5. It logs the execution results in Amazon CloudWatch.

**Lambda Execution Result**

The first test invocation completed successfully.

| Metric | Result |
|---|---|
| Execution status | Succeeded |
| Response status code | 200 |
| Records saved | 10 |
| Execution duration | 954.41 ms |
| Maximum memory used | 93 MB |

![Successful Lambda execution and response](docs/implementation-screenshots/06-import-lambda-success.png)

**DynamoDB Data Verification**

After execution, I scanned the `jsonplaceholder-users` table and verified that all 10 user records were present.

| Verification | Result |
|---|---|
| Items returned | 10 |
| Items scanned | 10 |
| Scan efficiency | 100% |
| Read capacity consumed | 2 RCUs |

![Imported user records in DynamoDB](docs/implementation-screenshots/07-dynamodb-imported-users.png)

