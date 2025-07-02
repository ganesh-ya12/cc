# Experiment 10: AWS SNS Canteen Management System

## Overview
Implement a canteen management message flow system using AWS SNS (Simple Notification Service) with Lambda functions to simulate the complete order lifecycle with automated notifications.

## Architecture
- **AWS SNS**: Message publishing and subscription service
- **AWS Lambda**: Serverless function to simulate order flow
- **Email/SMS**: Customer notification endpoints
- **Order Flow**: Placed → Preparing → Ready → Out for Delivery → Delivered

## Components
- SNS Topic for order notifications
- Email subscription for customer notifications
- Lambda function for order lifecycle simulation
- Automated messaging system

## Step-by-Step Instructions

### Step 1: Create SNS Topic
1. Go to **AWS Console → SNS**
2. Click **Topics → Create topic**
3. **Topic Configuration:**
   - Type: **Standard**
   - Name: **Orders**
   - Display name: **Canteen Orders**
   - Leave other settings as default
4. Click **Create topic**
5. **Note down the Topic ARN** (you'll need this for Lambda)

### Step 2: Create Email Subscription
1. In the **Orders** topic, click **Create subscription**
2. **Subscription Configuration:**
   - Protocol: **Email**
   - Endpoint: Enter customer email (e.g., `customer@example.com`)
3. Click **Create subscription**
4. **Confirm Subscription:**
   - Check the email inbox
   - Click the confirmation link in the email
   - Status should change to "Confirmed"

### Step 3: Create Lambda Function
1. Go to **AWS Console → Lambda**
2. Click **Create function**
3. **Function Configuration:**
   - Function name: **PlaceOrder**
   - Runtime: **Python 3.9**
   - Architecture: **x86_64**
   - Permissions: Create new role with basic Lambda permissions
4. Click **Create function**

### Step 4: Configure Lambda Code
Replace the default code with the following:

```python
import boto3
import time

# Initialize SNS client
sns = boto3.client('sns')

# Replace with your actual SNS Topic ARN
TOPIC_ARN = 'arn:aws:sns:us-east-1:123456789012:Orders'

# Order flow with messages and delays (in seconds)
order_flow = [
    ("Order Placed", "Order placed successfully. Thank you for ordering from our canteen!", 2),
    ("Preparing", "Your food is being freshly prepared by our kitchen team.", 5),
    ("Ready", "Your food is ready for pickup. Our delivery agent will arrive soon.", 5),
    ("Out for Delivery", "Your food is out for delivery. Please be available to receive it.", 5),
    ("Delivered", "Your food has been delivered. Enjoy your meal!", 0),
]

def publish_message(subject, message):
    """Publish message to SNS topic"""
    try:
        response = sns.publish(
            TopicArn=TOPIC_ARN,
            Subject=subject,
            Message=message
        )
        print(f"Message published: {subject}")
        return response
    except Exception as e:
        print(f"Error publishing message: {str(e)}")
        raise e

def lambda_handler(event, context):
    """Main Lambda handler function"""
    try:
        print("Starting order flow simulation...")
        
        for subject, message, delay in order_flow:
            # Publish message
            publish_message(subject, message)
            
            # Wait before next message (except for the last one)
            if delay > 0:
                time.sleep(delay)
        
        print("Order flow completed successfully!")
        
        return {
            'statusCode': 200,
            'body': 'Order flow completed and notifications sent successfully.'
        }
        
    except Exception as e:
        print(f"Error in order flow: {str(e)}")
        return {
            'statusCode': 500,
            'body': f'Error: {str(e)}'
        }
```

### Step 5: Update Topic ARN
1. **Get your SNS Topic ARN:**
   - Go to SNS Console → Topics → Click on "Orders"
   - Copy the ARN (format: `arn:aws:sns:region:account-id:Orders`)
2. **Update Lambda code:**
   - Replace `TOPIC_ARN` value with your actual Topic ARN
   - Click **Deploy** to save changes

### Step 6: Configure Lambda Timeout
1. In Lambda Console, go to **Configuration → General configuration**
2. Click **Edit**
3. **Settings:**
   - Timeout: **30 seconds** (or more if needed)
   - Memory: **128 MB** (default is fine)
4. Click **Save**

### Step 7: Test Lambda Function
1. Click **Test** in Lambda Console
2. **Create test event:**
   - Event name: **TestOrderFlow**
   - Template: **hello-world**
   - Use default JSON: `{}`
3. Click **Create**
4. Click **Test** to execute the function
5. Check **Execution results** for success message

### Step 8: Verify Email Notifications
After running the test, check the subscribed email inbox for:
1. **"Order Placed"** - Immediate notification
2. **"Preparing"** - After 2 seconds
3. **"Ready"** - After 5 more seconds  
4. **"Out for Delivery"** - After 5 more seconds
5. **"Delivered"** - After 5 more seconds

## Expected Email Flow
```
Subject: Order Placed
Message: Order placed successfully. Thank you for ordering from our canteen!

↓ (2 seconds delay)

Subject: Preparing  
Message: Your food is being freshly prepared by our kitchen team.

↓ (5 seconds delay)

Subject: Ready
Message: Your food is ready for pickup. Our delivery agent will arrive soon.

↓ (5 seconds delay)

Subject: Out for Delivery
Message: Your food is out for delivery. Please be available to receive it.

↓ (5 seconds delay)

Subject: Delivered
Message: Your food has been delivered. Enjoy your meal!
```

## Advanced Features (Optional)

### Add SMS Notifications
```python
# Create additional subscription with SMS
Protocol: SMS
Endpoint: +1234567890  # Customer phone number
```

### Add Order Details
```python
# Enhanced message with order details
def lambda_handler(event, context):
    # Extract order details from event
    order_id = event.get('order_id', 'ORD001')
    customer_name = event.get('customer_name', 'Customer')
    items = event.get('items', ['Sandwich', 'Coffee'])
    
    # Customize messages with order details
    enhanced_flow = [
        ("Order Placed", f"Hi {customer_name}! Order #{order_id} placed successfully. Items: {', '.join(items)}", 2),
        # ... rest of the flow
    ]
```

### Add Error Handling
```python
# Add retry logic and error notifications
import json
from botocore.exceptions import ClientError

def publish_with_retry(subject, message, max_retries=3):
    for attempt in range(max_retries):
        try:
            return sns.publish(
                TopicArn=TOPIC_ARN,
                Subject=subject,
                Message=message
            )
        except ClientError as e:
            if attempt == max_retries - 1:
                raise e
            time.sleep(2 ** attempt)  # Exponential backoff
```

## Monitoring and Logging

### CloudWatch Logs
- Lambda automatically logs to CloudWatch
- View logs: CloudWatch → Log groups → `/aws/lambda/PlaceOrder`

### SNS Metrics
- Monitor message delivery in SNS Console
- Check failed deliveries and bounce rates

## Troubleshooting

### Messages Not Being Received
1. **Check subscription status**: Must be "Confirmed"
2. **Verify email address**: Check for typos
3. **Check spam folder**: SNS emails might be filtered
4. **Verify Topic ARN**: Must match exactly in Lambda code

### Lambda Function Errors
1. **Check CloudWatch logs** for detailed error messages
2. **Verify IAM permissions**: Lambda needs SNS publish permissions
3. **Increase timeout**: Complex flows might need more time
4. **Test with simple payload**: Use empty JSON `{}`

### Permission Issues
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "sns:Publish"
            ],
            "Resource": "arn:aws:sns:*:*:Orders"
        }
    ]
}
```

## Cost Optimization
- SNS charges per message published ($0.50 per 1M messages)
- Lambda charges per invocation and duration
- Email notifications are free up to 1,000 per month
- SMS notifications have per-message charges

## Clean Up
To avoid charges:
1. **Delete Lambda function**
2. **Delete SNS subscriptions**
3. **Delete SNS topic**
4. **Remove CloudWatch logs** (optional)

## Real-World Applications
- Restaurant order tracking
- E-commerce order updates  
- Appointment reminders
- System alert notifications
- IoT device status updates

## Next Steps
- Integrate with API Gateway for web interface
- Add DynamoDB for order persistence
- Implement customer preference management
- Add delivery tracking with location updates
- Create admin dashboard for order management