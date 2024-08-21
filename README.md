# Real-time Order Processing on AWS

## Overview

This project demonstrates a real-time order processing system built on AWS using Kinesis, Lambda, DynamoDB, and S3. The system captures and processes order data in real time, storing the results in DynamoDB and archiving the raw data in S3.

## Architecture

The system architecture consists of the following components:

1. **AWS Kinesis**: Captures and streams order data in real time.
2. **AWS Lambda**: Processes the data in real-time and performs necessary transformations and validations.
3. **DynamoDB**: Stores processed order information for fast retrieval.
4. **S3**: Archives raw order data for future analysis and auditing.

## Prerequisites

- AWS Account
- AWS CLI configured
- Basic knowledge of AWS services: Kinesis, Lambda, DynamoDB, S3
- Python 3.x

## Setup

### Step 1: Create Kinesis Stream

1. Log in to your AWS Management Console.
2. Navigate to the Kinesis service.
3. Create a new Kinesis data stream named `OrderStream`.
4. Set the number of shards based on expected throughput.

### Step 2: Create DynamoDB Table

1. Navigate to the DynamoDB service.
2. Create a new table named `Orders`.
3. Define the primary key as `OrderID` (string).

### Step 3: Create S3 Bucket

1. Navigate to the S3 service.
2. Create a new S3 bucket for archiving raw order data.
3. Name the bucket `order-data-archive`.

### Step 4: Create Lambda Function

1. Navigate to the Lambda service.
2. Create a new Lambda function with Python 3.x runtime.
3. Use the following code for the Lambda function:

    ```python
    import json
    import boto3
    import uuid
    from datetime import datetime

    dynamodb = boto3.resource('dynamodb')
    s3 = boto3.client('s3')
    table = dynamodb.Table('Orders')

    def lambda_handler(event, context):
        for record in event['Records']:
            payload = json.loads(record['kinesis']['data'])
            
            # Processing order data
            order_id = str(uuid.uuid4())
            order_data = {
                'OrderID': order_id,
                'CustomerID': payload['customer_id'],
                'OrderAmount': payload['amount'],
                'OrderDate': datetime.now().isoformat()
            }
            
            # Store the processed data in DynamoDB
            table.put_item(Item=order_data)
            
            # Archive the raw data in S3
            s3.put_object(
                Bucket='order-data-archive',
                Key=f'orders/{order_id}.json',
                Body=json.dumps(payload)
            )
        
        return {
            'statusCode': 200,
            'body': json.dumps('Order processed successfully!')
        }
    ```

4. Set up a Kinesis trigger for the Lambda function using the `OrderStream`.

## Testing

1. Use the AWS CLI or SDK to send sample order data to the Kinesis stream:

    ```bash
    aws kinesis put-record \
        --stream-name OrderStream \
        --partition-key "partitionKey" \
        --data '{"customer_id": "12345", "amount": 250.00}'
    ```

2. Verify that the data is processed and stored in DynamoDB.
3. Check S3 for the archived raw data.

## Monitoring

- Use CloudWatch to monitor the performance of the Lambda function.
- Set up alarms for any errors or performance issues.

## Cleanup

To avoid incurring charges, ensure you delete the Kinesis stream, DynamoDB table, Lambda function, and S3 bucket once the project is complete.
