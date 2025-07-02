# Experiment 9 – Create a NoSQL Database using DynamoDB (Feedback System)

This lab helps you set up a NoSQL feedback database using AWS DynamoDB to collect customer feedback after an order.

## Requirements

- AWS account
- IAM user with DynamoDB permissions
- Access to AWS Management Console

## Step-by-Step Guide

### Step 1: Open DynamoDB Console

1. Go to [DynamoDB Console](https://console.aws.amazon.com/dynamodb/)
2. Click on **Tables** from the sidebar
3. Click **Create table**

### Step 2: Configure Table Details

- **Table Name**: `FeedbackTable_22bd1a057y`
- **Primary Key**: `order_id` (Partition key, type: String)

Leave all other settings as default unless instructed.

Click **Create Table**

### Step 3: Add Feedback Entries

After the table is created:

1. Go to **Explore table items**
2. Click **Create item**
3. Add fields like:

```json
{
  "order_id": "12345",
  "customer_name": "John Doe",
  "rating": 5,
  "comments": "Tasty food, fast delivery!"
}
```

4. Click **Save**

## Sample Feedback Item Structure

```json
{
  "order_id": "ORD1234",
  "customer_name": "Jane Smith",
  "rating": 4,
  "comments": "Great experience!"
}
```

## Optional: Add Feedback from Lambda

You can create a Lambda function to write feedback into DynamoDB:

```python
import boto3

dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('FeedbackTable_22bd1a057y')

def lambda_handler(event, context):
    feedback = {
        'order_id': event['order_id'],
        'customer_name': event['customer_name'],
        'rating': event['rating'],
        'comments': event['comments']
    }
    
    table.put_item(Item=feedback)
    
    return {
        'statusCode': 200,
        'body': 'Feedback submitted!'
    }
```

## Outcome

- You now have a fully working NoSQL DynamoDB Table to store feedback from customers
- Can be used in integration with SNS, SQS, or web forms

## Extra Tips

- Add timestamps using `time` or `datetime.now()` in Lambda
- Use DynamoDB Streams to trigger alerts/analytics
- Secure feedback write access with IAM roles

## Reference

- AWS DynamoDB Docs: https://docs.aws.amazon.com/dynamodb/
- This README is based on lab Experiment-9 from CC_LAB