# AWS Serverless API Project 

##  Project Overview

This project demonstrates a serverless application
architecture using AWS cloud services.

The application uses Amazon API Gateway,
AWS Lambda, Amazon DynamoDB, and Amazon S3.

##  Architecture

![imageAlt](https://github.com/yogeshAce6/Serverless-Application-Development/blob/c887969a6974778d549a34310f21a53a64219594/images/serverless%20arch%20.png)

##  AWS Services Used

- Amazon API Gateway
- AWS Lambda
- Amazon DynamoDB
- Amazon S3
- AWS IAM

##  How It Works

1. Users send HTTP requests through API Gateway.
2. API Gateway triggers AWS Lambda.
3. Lambda processes the request.
4. DynamoDB stores and retrieves application data.
5. Amazon S3 stores files and objects.
6. Lambda returns the response through API Gateway.

##  Features

- Serverless Architecture
- REST API Integration
- No Server Management
- Scalable Backend
- Cloud Storage Integration
- Cost-Effective Infrastructure

## Technologies Used

- AWS Cloud
- AWS Lambda
- Amazon API Gateway
- Amazon DynamoDB
- Amazon S3
- Python

##  Future Improvements

- Add API Authentication
- Implement AWS CloudWatch Monitoring
- Add CI/CD Pipeline
- Automate Infrastructure Deployment
 ## Step-by-step implementation
 ## Step 1 — Create DynamoDB database
 1. Open the DynamoDB Console.
2. Select Create table.
3. Enter these details:
- Table name: CloudTasks
- Partition key: taskId
- Key type: String
![imageAlt](https://github.com/yogeshAce6/Serverless-Application-Development/blob/c887969a6974778d549a34310f21a53a64219594/images/Dynamodb%20table%20.png)
4. Choose Create table and wait until the table becomes active.
![imageAlt](https://github.com/yogeshAce6/Serverless-Application-Development/blob/c887969a6974778d549a34310f21a53a64219594/images/Dynamo%20db%20dasboard%20.png)
 ## Step 2 — Create AWS Lambda function
 1. Open the Lambda Console.
2. Select Create function → Author from scratch.
3. Enter:
- Function name: CloudTaskFunction
- Runtime: Python 3.12, if available
- Architecture: x86_64
- Permissions: Create a new execution role with basic Lambda permissions
4. Click Create function.
![imageAlt](https://github.com/yogeshAce6/Serverless-Application-Development/blob/c887969a6974778d549a34310f21a53a64219594/images/lambda%20function%20.png)
Next, replace the sample code in the Lambda editor with this beginner-friendly backend.
```bash

import json
import os
import uuid
import boto3
from decimal import Decimal

dynamodb = boto3.resource("dynamodb")
table = dynamodb.Table(os.environ["TABLE_NAME"])


def response(status, body):
    return {
        "statusCode": status,
        "headers": {
            "Content-Type": "application/json",
            "Access-Control-Allow-Origin": "*",
            "Access-Control-Allow-Headers": "Content-Type",
            "Access-Control-Allow-Methods": "GET,POST,OPTIONS"
        },
        "body": json.dumps(body, default=str)
    }


def lambda_handler(event, context):
    method = event.get("requestContext", {}).get(
        "http", {}
    ).get("method", "GET")

    if method == "OPTIONS":
        return response(200, {"message": "OK"})

    if method == "POST":
        try:
            data = json.loads(event.get("body") or "{}")
            task = data.get("task", "").strip()

            if not task:
                return response(400, {"error": "Task is required"})

            item = {
                "taskId": str(uuid.uuid4()),
                "task": task
            }
            table.put_item(Item=item)

            return response(201, {
                "message": "Task saved successfully",
                "item": item
            })

        except (ValueError, TypeError):
            return response(400, {"error": "Invalid JSON"})

    if method == "GET":
        result = table.scan()
        return response(200, {
            "items": result.get("Items", [])
        })

    return response(405, {"error": "Method not allowed"})
```
![imageAlt](https://github.com/yogeshAce6/Serverless-Application-Development/blob/c887969a6974778d549a34310f21a53a64219594/images/lambda%20code%20.png)
Click Deploy.
Now open Configuration → Environment variables → Edit and add:
- Key: TABLE_NAME
- Value: CloudTasks
![imageAlt](https://github.com/yogeshAce6/Serverless-Application-Development/blob/c887969a6974778d549a34310f21a53a64219594/images/lambda%20env.png)
![imageAlt](https://github.com/yogeshAce6/Serverless-Application-Development/blob/c887969a6974778d549a34310f21a53a64219594/images/env%20variable%20.png)
Save the changes.
## Step 3 — Configure API Gateway
