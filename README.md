# Serverless Cloud Dictionary Application
# video link of Serverless Cloud Dictionary Application
[click here] (https://drive.google.com/file/d/1ocg3oDFHvHjrAOZJTAD4NRNfCnZV54y-/view?usp=drive_link)

# Project Overview:


In this project, we'll be developing a serverless cloud dictionary application where users can:

Search for terms related to cloud technologies.

View the definitions of cloud terms.

Utilize a serverless architecture using AWS services.

The application will use Lambda for backend processing, API Gateway for managing the API endpoints, and DynamoDB for storing the dictionary terms and their definitions.


The frontend, a React application, will be hosted on AWS Amplify, and API requests will be made to interact with the database.

# Services Used 🛠


AWS Amplify: Host the frontend React application.

AWS Lambda: Handle API requests for retrieving terms and adding new ones

AWS API Gateway: Manage API endpoints to allow communication between frontend and Lambda functions.

AWS DynamoDB: Store dictionary terms and definitions.

IAM Roles & Policies: Secure access to AWS resources like Lambda, DynamoDB, and API Gateway.


# Project Architecture:
![image alt](https://github.com/subham1853/cloud-dictionary-v2/blob/f8a38091d157ec5d1348765b67a82388ba1ebc3d/serverless%20cloud%20directory.png)


Steps to be performed:

In the next few lessons, we'll be going through the following steps.

Setup frontend and host it on AWS Amplify

Configure DynamoDB to store Cloud Definitions

Create Lambda function for fetching terms

Setup API Gateway for API management




# Lambda Function Code:


import json
import boto3

# Create a DynamoDB client
dynamodb = boto3.client('dynamodb')

# Your DynamoDB table name
table_name = 'CloudDefinitions'

def lambda_handler(event, context):
    try:
        # Debug: Log the entire event
        print("Event received:", json.dumps(event))
        
        # Try to get term from multiple sources
        term = ''
        
        # Method 1: Query parameters (for GET requests)
        query_params = event.get('queryStringParameters') or {}
        if query_params and query_params.get('term'):
            term = query_params.get('term')
        
        # Method 2: Path parameters
        path_params = event.get('pathParameters') or {}
        if not term and path_params and path_params.get('term'):
            term = path_params.get('term')
        
        # Method 3: Body (for POST requests)
        if not term and event.get('body'):
            try:
                body = json.loads(event['body'])
                term = body.get('term', '')
            except:
                pass
        
        # Method 4: Direct event (for test events)
        if not term:
            term = event.get('term', '')
        
        print("Term extracted:", term)
        
        # If no term provided, return error with debug info
        if not term:
            return {
                'statusCode': 400,
                'headers': {
                    'Content-Type': 'application/json',
                    'Access-Control-Allow-Origin': '*',
                    'Access-Control-Allow-Methods': 'OPTIONS,GET,POST',
                    'Access-Control-Allow-Headers': 'Content-Type',
                },
                'body': json.dumps({
                    'message': 'Term parameter is required',
                    'debug': {
                        'queryStringParameters': event.get('queryStringParameters'),
                        'pathParameters': event.get('pathParameters'),
                        'body': event.get('body'),
                        'httpMethod': event.get('httpMethod'),
                        'event_keys': list(event.keys())
                    }
                })
            }
        
        # Query the DynamoDB table for the term
        response = dynamodb.get_item(
            TableName=table_name,
            Key={'term': {'S': term}}
        )
        
        # Check if the term exists in the table
        if 'Item' in response:
            definition = response['Item']['definition']['S']
            return {
                'statusCode': 200,
                'headers': {
                    'Content-Type': 'application/json',
                    'Access-Control-Allow-Origin': '*',
                    'Access-Control-Allow-Methods': 'OPTIONS,GET,POST',
                    'Access-Control-Allow-Headers': 'Content-Type',
                },
                'body': json.dumps({
                    'term': term,
                    'definition': definition
                })
            }
        else:
            return {
                'statusCode': 404,
                'headers': {
                    'Content-Type': 'application/json',
                    'Access-Control-Allow-Origin': '*',
                    'Access-Control-Allow-Methods': 'OPTIONS,GET,POST',
                    'Access-Control-Allow-Headers': 'Content-Type',
                },
                'body': json.dumps({'message': f'Term "{term}" not found'})
            }
            
    except Exception as e:
        return {
            'statusCode': 500,
            'headers': {
                'Content-Type': 'application/json',
                'Access-Control-Allow-Origin': '*',
                'Access-Control-Allow-Methods': 'OPTIONS,GET,POST',
                'Access-Control-Allow-Headers': 'Content-Type',
            },
            'body': json.dumps({
                'error': 'Internal server error',
                'message': str(e)
            })
        }

# TEST Event:


{
  "queryStringParameters":
  {
  
    "term": "AWS KMS"
  }
}

# DynamoDB AWS CLI Commands:

aws dynamodb batch-write-item --request-items file://records/records-1.json

aws dynamodb batch-write-item --request-items file://records/records-2.json

aws dynamodb batch-write-item --request-items file://records/records-3.json

aws dynamodb batch-write-item --request-items file://records/records-4.json
