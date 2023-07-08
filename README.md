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

<img width="960" alt="2" src="https://github.com/Nekoi-Mirai/Serverless-Lab/assets/126063968/56a321ff-7c6e-42e7-b1e2-e5847513dc34">

![3](https://github.com/Nekoi-Mirai/Serverless-Lab/assets/126063968/fdfb3d5b-6a42-46b1-b20b-31d493d6fc1f)
<img width="960" alt="4" src="https://github.com/Nekoi-Mirai/Serverless-Lab/assets/126063968/a750157e-54ca-44dc-af24-33607f520594">
<img width="960" alt="5" src="https://github.com/Nekoi-Mirai/Serverless-Lab/assets/126063968/21ac1d42-96d7-4815-8340-deccc4c1bb81">
<img width="960" alt="6" src="https://github.com/Nekoi-Mirai/Serverless-Lab/assets/126063968/2bc8fd2b-456e-4ce1-a57a-a8418c7a9576">

![7](https://github.com/Nekoi-Mirai/Serverless-Lab/assets/126063968/39a7aca4-e98a-4c2e-9585-2848b2fe7fff)

![8](https://github.com/Nekoi-Mirai/Serverless-Lab/assets/126063968/6a8597d4-a9f3-42e3-a91a-0146c14bf0e0)

![9](https://github.com/Nekoi-Mirai/Serverless-Lab/assets/126063968/9237a482-048f-4e56-86dd-322e41ade8e0)


### Create Lambda Function

**To create the function**
1. Click "Create function" in AWS Lambda Console

![10](https://github.com/Nekoi-Mirai/Serverless-Lab/assets/126063968/98e8b87f-d044-4dc0-bb17-a5af44c0f793)


2. Select "Author from scratch". Use name **LambdaFunctionOverHttps** , select **Python 3.7** as Runtime. Under Permissions, select "Use an existing role", and select **lambda-apigateway-role** that we created, from the drop down
![11](https://github.com/Nekoi-Mirai/Serverless-Lab/assets/126063968/14aa1b6d-695f-4b04-a86b-5a6b7f439c50)


3. Click "Create function"

![12](https://github.com/Nekoi-Mirai/Serverless-Lab/assets/126063968/2274c78d-5e92-4eda-8a96-4dd095acfb45)


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
<img width="960" alt="13" src="https://github.com/Nekoi-Mirai/Serverless-Lab/assets/126063968/f3ada729-5707-4eb5-94e6-5a3721b6d596">



### Test Lambda Function

Let's test our newly created function. We haven't created DynamoDB and the API yet, so we'll do a sample echo operation. The function should output whatever input we pass.
1. Click the arrow on "Select a test event" and click "Configure test events"

<img width="960" alt="14" src="https://github.com/Nekoi-Mirai/Serverless-Lab/assets/126063968/5a3b8107-2185-49a2-9fac-eade92355f78">

![15](https://github.com/Nekoi-Mirai/Serverless-Lab/assets/126063968/26ea396c-3add-4b8e-ad46-61c2340b7aad)

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
![16](https://github.com/Nekoi-Mirai/Serverless-Lab/assets/126063968/eba23d94-de74-429b-9558-4cd303656f11)


3. Click "Test", and it will execute the test event. You should see the output in the console

<img width="960" alt="17" src="https://github.com/Nekoi-Mirai/Serverless-Lab/assets/126063968/94bd6e81-14f2-4aff-91c8-0e084ab5811e">

We're all set to create DynamoDB table and an API using our lambda as backend.


### Create DynamoDB Table

Create the DynamoDB table that the Lambda function uses.

**To create a DynamoDB table**


![18](https://github.com/Nekoi-Mirai/Serverless-Lab/assets/126063968/d28c4ff4-1036-4319-bb1b-84752d4d67b5)
![19](https://github.com/Nekoi-Mirai/Serverless-Lab/assets/126063968/fea4ee2e-c1b5-4af1-b19f-76abbdedc25b)


### Create API

**To create the API**
1. Go to API Gateway console
2. Click Create API
3. Scroll down and select "Build" for REST API
![20](https://github.com/Nekoi-Mirai/Serverless-Lab/assets/126063968/1ebf121a-9994-4f44-8042-270684fe4c59)

4. Give the API name as "DynamoDBOperations", keep everything as is, click "Create API"
<img width="960" alt="21" src="https://github.com/Nekoi-Mirai/Serverless-Lab/assets/126063968/81913d43-9c88-4f71-af9a-edd14f889ac9">

5. Click "Actions", then click "Create Resource"
6. Input "DynamoDBManager" in the Resource Name, Resource Path will get populated. Click "Create Resource"

<img width="960" alt="22" src="https://github.com/Nekoi-Mirai/Serverless-Lab/assets/126063968/b903b2ec-367f-4fe3-a23a-b75199ece1c7">

7. Let's create a POST Method for our API. With the "/dynamodbmanager" resource selected, Click "Actions" again and click "Create Method". 

![23](https://github.com/Nekoi-Mirai/Serverless-Lab/assets/126063968/da01b997-91c5-4320-b7a9-b31f72215207)

8. Select "POST" from drop down , then click checkmark

<img width="960" alt="24" src="https://github.com/Nekoi-Mirai/Serverless-Lab/assets/126063968/56398796-e16d-4c96-84ef-9ec9b5693389">

![25](https://github.com/Nekoi-Mirai/Serverless-Lab/assets/126063968/278e031c-f5a0-4e72-8e73-a2a1de12e766)

![26](https://github.com/Nekoi-Mirai/Serverless-Lab/assets/126063968/52a495f0-3620-44fd-abbd-d63dbd754da6)


Our API-Lambda integration is done!

### Deploy the API

In this step, we deploy the API that we created to a stage called prod.

1. Click "Actions", select "Deploy API"

![27](https://github.com/Nekoi-Mirai/Serverless-Lab/assets/126063968/e873a438-39a1-4e83-93a9-64ec1785e5ec)

<img width="960" alt="28" src="https://github.com/Nekoi-Mirai/Serverless-Lab/assets/126063968/6e5b07c0-784a-4ba9-ad88-f3c01bed13ba">

2. Now it is going to ask you about a stage. Select "[New Stage]" for "Deployment stage". Give "Prod" as "Stage name". Click "Deploy"

3. We're all set to run our solution! To invoke our API endpoint, we need the endpoint url. In the "Stages" screen, expand the stage "Prod", select "POST" method, and copy the "Invoke URL" from screen

<img width="960" alt="29" src="https://github.com/Nekoi-Mirai/Serverless-Lab/assets/126063968/53062407-29fb-4020-a4c9-c8b21283d31e">


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

![30](https://github.com/Nekoi-Mirai/Serverless-Lab/assets/126063968/11fb7bdc-5563-482d-aa66-1c6f0fc6950e)

2. I used Postman to execute my API from local machine.
    * To run this from Postman, select "POST" , paste the API invoke url. Then under "Body" select "raw" and paste the above JSON. Click "Send". API should execute and return "HTTPStatusCode" 200.

![32](https://github.com/Nekoi-Mirai/Serverless-Lab/assets/126063968/4f0702a7-344e-497f-8524-ff3e41892d06)

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

![34](https://github.com/Nekoi-Mirai/Serverless-Lab/assets/126063968/8aba24a7-b5e1-4657-990a-168dc2a17159)

4. To validate that the item is indeed inserted into DynamoDB table, go to Dynamo console, select "lambda-apigateway" table, select "Items" tab, and the newly inserted item should be displayed.

![36](https://github.com/Nekoi-Mirai/Serverless-Lab/assets/126063968/9961a2f5-9fe9-4cfa-9460-4518f8b8165a)
![37](https://github.com/Nekoi-Mirai/Serverless-Lab/assets/126063968/159dc4b0-8025-4d81-b6a6-ea75c8dcba8c)

5.To get all the inserted items from the table, we can use the "list" operation of Lambda using the same API. Pass the following JSON to the API, and it will return all the items from the Dynamo table

```json
{
    "operation": "list",
    "tableName": "lambda-apigateway",
    "payload": {
    }
}
```

![38](https://github.com/Nekoi-Mirai/Serverless-Lab/assets/126063968/994af0b4-cac4-4075-9397-4b64c889f1ff)

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

![40](https://github.com/Nekoi-Mirai/Serverless-Lab/assets/126063968/fedb3b31-acd6-432d-a36e-3473d44d9596)

7.To delete a specific item from the table, we can use the "delete" operation of Lambda using the same API. To request this operation, use the following JSON:

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

![42](https://github.com/Nekoi-Mirai/Serverless-Lab/assets/126063968/f95ee7ef-0c0a-4c56-bcd3-ee50a9138919)
![43](https://github.com/Nekoi-Mirai/Serverless-Lab/assets/126063968/015e8e4d-5223-49a1-93fb-634e67a9cb22)
![44](https://github.com/Nekoi-Mirai/Serverless-Lab/assets/126063968/491adab5-5cad-448f-b189-6860a862147b)
![45](https://github.com/Nekoi-Mirai/Serverless-Lab/assets/126063968/45c5925c-ff4a-48dc-b526-1075fa0d561e)

I have successfully created a serverless API using API Gateway, Lambda, and DynamoDB!
