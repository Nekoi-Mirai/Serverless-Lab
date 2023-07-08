# Serverless Lab

## Lab Overview

This model of lab is original from Saha-Rajeep, and then I finished it and added my steps, images and code on the base of it.
![High Level Design](./images/high-level-design.jpg)
An Amazon API Gateway is a collection of resources and methods. For this tutorial, you create one resource (DynamoDBManager) and define one method (POST) on it. The method is backed by a Lambda function (LambdaFunctionOverHttps). That is, when you call the API through an HTTPS endpoint, Amazon API Gateway invokes the Lambda function.

The POST method on the DynamoDBManager resource supports the following DynamoDB operations:

* Create, update, and delete an item.
* Read an item.
* Scan an item.
* Other operations (echo, ping), not related to DynamoDB, that you can use for testing.


## Setup

### Create Lambda IAM Role
Create the execution role that gives function permission to access AWS resources.

<img width="960" alt="1" src="https://github.com/Nekoi-Mirai/Serverless-Lab/assets/126063968/a46f751e-3472-402b-b1f4-51a622cf8002">


```json
 {
    "Version": "2012-10-17",
    "Statement": [
    {
      "Sid": "Stmt1428341300017",
      "Action": [
        "dynamodb:DeleteItem",
        "dynamodb:GetItem",
        "dynamodb:PutItem",
        "dynamodb:Query",
        "dynamodb:Scan",
        "dynamodb:UpdateItem"
      ],
      "Effect": "Allow",
      "Resource": "*"
    },
    {
      "Sid": "",
      "Resource": "*",
      "Action": [
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents"
      ],
      "Effect": "Allow"
    }
    ]
    }
```

![image](https://github.com/Nekoi-Mirai/Serverless-Lab/assets/126063968/61c8366f-a189-4398-88a2-0b8b5f7c6d55)
![image](https://github.com/Nekoi-Mirai/Serverless-Lab/assets/126063968/a118e6e7-724d-4a07-9d89-a76e1524a4ac)
![image](https://github.com/Nekoi-Mirai/Serverless-Lab/assets/126063968/5cd52458-95a0-4d3a-92c9-bca6b2cd18ba)
![image](https://github.com/Nekoi-Mirai/Serverless-Lab/assets/126063968/18b60554-9957-4fbe-abd5-35638cab9864)
![image](https://github.com/Nekoi-Mirai/Serverless-Lab/assets/126063968/5103b2e0-8d46-4a5d-93f0-d04c1e09d5ee)
![image](https://github.com/Nekoi-Mirai/Serverless-Lab/assets/126063968/61f74c62-e7fc-4dae-9244-20ed0ec487c1)
![image](https://github.com/Nekoi-Mirai/Serverless-Lab/assets/126063968/bd8a95c5-591a-4590-8d83-afeab884ca2c)
![image](https://github.com/Nekoi-Mirai/Serverless-Lab/assets/126063968/b7d5e1b6-4929-4ad4-961a-41076cb06142)

### Create Lambda Function

**To create the function**
1. Click "Create function" in AWS Lambda Console

![image](https://github.com/Nekoi-Mirai/Serverless-Lab/assets/126063968/fa6383bf-4ffa-47aa-9693-4143b95b22fe)

2. Select "Author from scratch". Use name **LambdaFunctionOverHttps** , select **Python 3.7** as Runtime. Under Permissions, select "Use an existing role", and select **lambda-apigateway-role** that we created, from the drop down
![image](https://github.com/Nekoi-Mirai/Serverless-Lab/assets/126063968/79dfd5aa-0983-4094-bfe5-df92ea10fe66)

3. Click "Create function"

![image](https://github.com/Nekoi-Mirai/Serverless-Lab/assets/126063968/cf497eeb-0020-4b1d-8e15-a7bd23e336e1)


4. Replace the boilerplate coding with the following code snippet and click "Save"

**Example Python Code**
```python
from __future__ import print_function

import boto3
import json

print('Loading function')


def lambda_handler(event, context):
    '''Provide an event that contains the following keys:

      - operation: one of the operations in the operations dict below
      - tableName: required for operations that interact with DynamoDB
      - payload: a parameter to pass to the operation being performed
    '''
    #print("Received event: " + json.dumps(event, indent=2))

    operation = event['operation']

    if 'tableName' in event:
        dynamo = boto3.resource('dynamodb').Table(event['tableName'])

    operations = {
        'create': lambda x: dynamo.put_item(**x),
        'read': lambda x: dynamo.get_item(**x),
        'update': lambda x: dynamo.update_item(**x),
        'delete': lambda x: dynamo.delete_item(**x),
        'list': lambda x: dynamo.scan(**x),
        'echo': lambda x: x,
        'ping': lambda x: 'pong'
    }

    if operation in operations:
        return operations[operation](event.get('payload'))
    else:
        raise ValueError('Unrecognized operation "{}"'.format(operation))
```
![image](https://github.com/Nekoi-Mirai/Serverless-Lab/assets/126063968/8853dcf0-bb1f-48e0-a962-744d468985bb)


### Test Lambda Function

Let's test our newly created function. We haven't created DynamoDB and the API yet, so we'll do a sample echo operation. The function should output whatever input we pass.
1. Click the arrow on "Select a test event" and click "Configure test events"

![image](https://github.com/Nekoi-Mirai/Serverless-Lab/assets/126063968/dbe64d8a-0543-46ea-bce6-c40d8aef4fd0)
![image](https://github.com/Nekoi-Mirai/Serverless-Lab/assets/126063968/0388ee28-9e82-4efd-b299-18af0005f1ca)


2. Paste the following JSON into the event. The field "operation" dictates what the lambda function will perform. In this case, it'd simply return the payload from input event as output. Click "Create" to save
```json
{
    "operation": "echo",
    "payload": {
        "somekey1": "somevalue1",
        "somekey2": "somevalue2"
    }
}
```
![image](https://github.com/Nekoi-Mirai/Serverless-Lab/assets/126063968/a0d86d1b-5eb7-41a5-885e-161ce8181b7a)


3. Click "Test", and it will execute the test event. You should see the output in the console

![image](https://github.com/Nekoi-Mirai/Serverless-Lab/assets/126063968/7d442b19-852d-47a3-a1b6-66b1d95ec2bd)

We're all set to create DynamoDB table and an API using our lambda as backend.

### Create DynamoDB Table

Create the DynamoDB table that the Lambda function uses.

**To create a DynamoDB table**

![image](https://github.com/Nekoi-Mirai/Serverless-Lab/assets/126063968/163caea4-42b9-48e8-841f-f7610c357389)

![image](https://github.com/Nekoi-Mirai/Serverless-Lab/assets/126063968/ccd935fb-61ec-4aa5-afba-a51f6e22ac4b)


### Create API

**To create the API**
1. Go to API Gateway console
2. Click Create API

![image](https://github.com/Nekoi-Mirai/Serverless-Lab/assets/126063968/4b43f43a-e9a9-46a8-ba88-abb0f96fc3f8)

3. Scroll down and select "Build" for REST API

4. Give the API name as "DynamoDBOperations", keep everything as is, click "Create API"

![image](https://github.com/Nekoi-Mirai/Serverless-Lab/assets/126063968/799406de-bbed-4287-aecb-6d74d9605c84)

5. Click "Actions", then click "Create Resource"

![image](https://github.com/Nekoi-Mirai/Serverless-Lab/assets/126063968/d58ba4d5-2e43-46a5-ac60-b346694ea0cd)


6. Input "DynamoDBManager" in the Resource Name, Resource Path will get populated. Click "Create Resource"

![image](https://github.com/Nekoi-Mirai/Serverless-Lab/assets/126063968/dc3bd160-fa82-457e-8384-9bae05166079)

7. Let's create a POST Method for our API. With the "/dynamodbmanager" resource selected, Click "Actions" again and click "Create Method". 

![image](https://github.com/Nekoi-Mirai/Serverless-Lab/assets/126063968/2bf56e27-5c84-4532-be61-74d3a78d3e00)

8. Select "POST" from drop down , then click checkmark

![image](https://github.com/Nekoi-Mirai/Serverless-Lab/assets/126063968/0bad04cf-c62b-4b46-8656-2966b0cc7698)


![image](https://github.com/Nekoi-Mirai/Serverless-Lab/assets/126063968/3e3d6c48-1dc3-4411-9350-e2a5792a7ba4)


![image](https://github.com/Nekoi-Mirai/Serverless-Lab/assets/126063968/5220e497-c22f-43c6-8a5d-953532379344)

Our API-Lambda integration is done!

### Deploy the API

In this step, we deploy the API that we created to a stage called prod.

1. Click "Actions", select "Deploy API"

![image](https://github.com/Nekoi-Mirai/Serverless-Lab/assets/126063968/4a91da7b-5ffc-49ea-beb1-1e6595288702)

2. Now it is going to ask you about a stage. Select "[New Stage]" for "Deployment stage". Give "Prod" as "Stage name". Click "Deploy"

![image](https://github.com/Nekoi-Mirai/Serverless-Lab/assets/126063968/6b57f982-460d-44ad-a2d8-c993174bc8af)

3. We're all set to run our solution! To invoke our API endpoint, we need the endpoint url. In the "Stages" screen, expand the stage "Prod", select "POST" method, and copy the "Invoke URL" from screen

![image](https://github.com/Nekoi-Mirai/Serverless-Lab/assets/126063968/d92cc0ab-1b6c-4af7-806f-94c95ab68eab)


### Running our solution

1. The Lambda function supports using the create operation to create an item in your DynamoDB table. To request this operation, use the following JSON:

```json
{
    "operation": "create",
    "tableName": "lambda-apigateway",
    "payload": {
        "Item": {
            "id": "1234ABCD",
            "number": 5
        }
    }
}
```

![image](https://github.com/Nekoi-Mirai/Serverless-Lab/assets/126063968/8338ce5b-b6fc-49e1-89c5-72c469735bca)

2. I used Postman to execute my API from local machine.
    * To run this from Postman, select "POST" , paste the API invoke url. Then under "Body" select "raw" and paste the above JSON. Click "Send". API should execute and return "HTTPStatusCode" 200.

![image](https://github.com/Nekoi-Mirai/Serverless-Lab/assets/126063968/c4fb16e4-0be0-472f-b672-cbfbc4fbe255)



3.To create another different item in my DynamoDB table. 
```json
{
    "operation": "create",
    "tableName": "lambda-apigateway",
    "payload": {
        "Item": {
            "id": "123ABCD",
            "number": 6
        }
    }
}
```

![image](https://github.com/Nekoi-Mirai/Serverless-Lab/assets/126063968/4403232c-7680-4dd1-9972-dc4518e36f55)
![image](https://github.com/Nekoi-Mirai/Serverless-Lab/assets/126063968/2d77dae3-58d6-428a-a6bb-7aa4df0a366f)

4. To validate that the item is indeed inserted into DynamoDB table, go to Dynamo console, select "lambda-apigateway" table, select "Items" tab, and the newly inserted item should be displayed.

![image](https://github.com/Nekoi-Mirai/Serverless-Lab/assets/126063968/11f5230d-5ccc-4e34-ac2f-e4e05490bd89)
![image](https://github.com/Nekoi-Mirai/Serverless-Lab/assets/126063968/b854a39b-ef67-4fe3-b8fa-f665d05a5487)

5.To get all the inserted items from the table, we can use the "list" operation of Lambda using the same API. Pass the following JSON to the API, and it will return all the items from the Dynamo table

```json
{
    "operation": "list",
    "tableName": "lambda-apigateway",
    "payload": {
    }
}
```
![image](https://github.com/Nekoi-Mirai/Serverless-Lab/assets/126063968/00c303f1-dcbb-4c82-92b3-ee95637fa3dd)

6.To read a specific item from the table, we can use the "read" operation of Lambda using the same API. To request this operation, use the following JSON:

```json
{
    "operation": "read",
    "tableName": "lambda-apigateway",
    "payload": {
        "Key": {
            "id": "123ABCD"
        }
    }
}

```
![image](https://github.com/Nekoi-Mirai/Serverless-Lab/assets/126063968/a03a3394-fa3a-442f-8e90-73e74815570c)

6.To delete a specific item from the table, we can use the "delete" operation of Lambda using the same API. To request this operation, use the following JSON:

```json
{
    "operation": "delete",
    "tableName": "lambda-apigateway",
    "payload": {
        "Key": {
            "id": "1234ABCD"
        }
    }
}
```

![image](https://github.com/Nekoi-Mirai/Serverless-Lab/assets/126063968/f27e9f41-12d8-40b8-8520-5bc9ce7e8b0e)
![image](https://github.com/Nekoi-Mirai/Serverless-Lab/assets/126063968/5b75fb3d-7d0d-419b-a9b6-89c9b5e2d1aa)
![image](https://github.com/Nekoi-Mirai/Serverless-Lab/assets/126063968/ba609f2e-d774-4ea4-9ab2-75facc4bba8e)

I have successfully created a serverless API using API Gateway, Lambda, and DynamoDB!
