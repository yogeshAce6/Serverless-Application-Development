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
1. Open the API Gateway Console.
![imageAlt](https://github.com/yogeshAce6/Serverless-Application-Development/blob/425757ce44f85a073c063faa7bcfa75b10740677/images/api%20.png)
2. Select Create API → HTTP API → Build.
3. API name: CloudTaskAPI.
![imageAlt](https://github.com/yogeshAce6/Serverless-Application-Development/blob/425757ce44f85a073c063faa7bcfa75b10740677/images/api%20http.png)
4. Add your Lambda function CloudTaskFunction as the integration.
![imageAlt](https://github.com/yogeshAce6/Serverless-Application-Development/blob/425757ce44f85a073c063faa7bcfa75b10740677/images/api%20lambda%20api%20integration%20.png)
5. Create these routes:
![imageAlt](https://github.com/yogeshAce6/Serverless-Application-Development/blob/425757ce44f85a073c063faa7bcfa75b10740677/images/api.png)

## Step 4 — Create the frontend website
Create a folder on your computer named cloud-task-manager. Inside it, create a file named index.html.
```bash

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Cloud Task Manager</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background: #eef4ff;
      margin: 0;
      padding: 40px 16px;
    }
    main {
      max-width: 600px;
      margin: auto;
      padding: 28px;
      background: white;
      border-radius: 16px;
      box-shadow: 0 8px 30px #102a5015;
    }
    h1 { color: #174ea6; }
    form { display: flex; gap: 8px; }
    input {
      flex: 1;
      min-width: 0;
      padding: 12px;
    }
    button {
      padding: 12px 18px;
      background: #175cd3;
      color: white;
      border: 0;
      border-radius: 6px;
      cursor: pointer;
    }
    li { margin: 12px 0; }
    #status { color: #175cd3; }
  </style>
</head>
<body>
<main>
  <h1>☁️ Cloud Task Manager</h1>
  <p>Powered by AWS Serverless Services</p>

  <form id="taskForm">
    <input id="taskInput" placeholder="Enter a new task"
           required maxlength="200">
    <button type="submit">Add Task</button>
  </form>

  <p id="status" role="status"></p>
  <h2>Your Tasks</h2>
  <ul id="taskList"></ul>
  <button type="button" onclick="loadTasks()">Refresh Tasks</button>
</main>

<script>
  // Replace this with your API Gateway Invoke URL.
  const API_URL = "PASTE_YOUR_API_INVOKE_URL_HERE";

  async function loadTasks() {
    const status = document.getElementById("status");
    status.textContent = "Loading tasks...";

    try {
      const res = await fetch(`${API_URL}/tasks`);
      if (!res.ok) throw new Error("Unable to load tasks");

      const data = await res.json();
      const list = document.getElementById("taskList");
      list.replaceChildren();

      (data.items || []).forEach(item => {
        const li = document.createElement("li");
        li.textContent = item.task;
        list.appendChild(li);
      });

      status.textContent = "Tasks loaded.";
    } catch (error) {
      status.textContent = "Error loading tasks. Check API and CORS.";
    }
  }

  document.getElementById("taskForm").addEventListener(
    "submit", async event => {
      event.preventDefault();

      const input = document.getElementById("taskInput");
      const status = document.getElementById("status");

      try {
        const res = await fetch(`${API_URL}/tasks`, {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify({ task: input.value.trim() })
        });

        if (!res.ok) throw new Error("Task could not be saved");

        input.value = "";
        status.textContent = "Task saved successfully!";
        await loadTasks();
      } catch (error) {
        status.textContent = "Save failed. Check API, CORS and permissions.";
      }
    }
  );
</script>
</body>
</html>
```
## Important: Replace PASTE_YOUR_API_INVOKE_URL_HERE with your actual API Gateway Invoke URL, without adding /tasks at the end. The code appends that route automatically.
## Step 5 — Test the application
Before hosting the website, test the backend in this order:
1. Open API Gateway and check that the API is deployed.
2. Check that Lambda has the correct TABLE_NAME environment variable and DynamoDB permissions.
3. Test GET /tasks and confirm that it returns JSON.
4. Test POST /tasks with a JSON body such as {"task":"Complete AWS project"}.
5. Open DynamoDB → CloudTasks → Explore table items and confirm the task was stored.
6. Open your frontend website and test adding and refreshing tasks.
![imageAlt](https://github.com/yogeshAce6/Serverless-Application-Development/blob/425757ce44f85a073c063faa7bcfa75b10740677/images/Screenshot%202026-10-10%20125135.png)
## Step 6 — Host the website on Amazon S3
1. Open the Amazon S3 Console.
![imageAlt](https://github.com/yogeshAce6/Serverless-Application-Development/blob/425757ce44f85a073c063faa7bcfa75b10740677/images/Screenshot%202026-10-10%20135226.png)
2. Create a globally unique bucket name, for example cloud-task-manager-yourid.
3. Upload index.html.
![imageAlt](https://github.com/yogeshAce6/Serverless-Application-Development/blob/425757ce44f85a073c063faa7bcfa75b10740677/images/Screenshot%202026-10-10%20135527.png)
4. For a simple static website test, configure S3 static website hosting and the appropriate public-read bucket policy only if your account permits it.
![imageAlt](https://github.com/yogeshAce6/Serverless-Application-Development/blob/425757ce44f85a073c063faa7bcfa75b10740677/images/Screenshot%202026-10-10%20125447.png)

For a more secure deployment, keep S3 private and use CloudFront to deliver the website. Enable HTTPS and configure the frontend's CORS origin accordingly.
## Output 
![imageAlt](https://github.com/yogeshAce6/Serverless-Application-Development/blob/425757ce44f85a073c063faa7bcfa75b10740677/images/Screenshot%202026-10-10%20125135.png)
