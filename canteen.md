# 🍽️ Integrated Canteen Order System - AWS SNS & SQS

Complete order flow with AWS SNS + SQS working together

## 🔄 How It All Works Together

1. **Order Placed** - Customer places order → Sent to SQS Queue
2. **Order Processing** - Lambda processes queue → Sends SNS notifications  
3. **Real-time Updates** - Customer gets email updates at each stage
4. **Complete Flow** - Order → Queue → Process → Notify → Complete

## 📋 Prerequisites

**What You Need:**
- AWS Account
- Valid Email
- Node.js (Optional)
- AWS Console Access

## 📬 Step 1: Create SQS Queue (Order Intake)

### Create the Order Queue

1. Go to **SQS Console**
2. Click **Create Queue**
3. Name: `CanteenOrderQueue`
4. Type: **Standard**
5. Click **Create Queue**
6. **Save the Queue URL** - you'll need it!

## 📧 Step 2: Create SNS Topic (Notifications)

### Create Notification Topic

1. Go to **SNS Console**
2. Click **Create Topic**
3. Name: `CanteenNotifications`
4. Type: **Standard**
5. Click **Create Topic**
6. **Save the Topic ARN**

### Add Email Subscription

1. Open your **CanteenNotifications** topic
2. Click **Create Subscription**
3. Protocol: **Email**
4. Endpoint: Your email address
5. Confirm subscription via email

## ⚡ Step 3: Create Processing Lambda (The Brain)

### Create Lambda Function

1. Go to **Lambda Console**
2. Click **Create Function**
3. Name: `CanteenOrderProcessor`
4. Runtime: **Python 3.9**
5. Create with basic execution role

### Add the Integrated Code

This Lambda will process SQS messages AND send SNS notifications:

```python
import json
import boto3
import time
from datetime import datetime

# Initialize AWS clients
sqs = boto3.client('sqs')
sns = boto3.client('sns')

# Configuration - REPLACE WITH YOUR ACTUAL VALUES
QUEUE_URL = 'https://sqs.us-east-1.amazonaws.com/123456789012/CanteenOrderQueue'
TOPIC_ARN = 'arn:aws:sns:us-east-1:123456789012:CanteenNotifications'

def send_notification(subject, message, order_id):
    """Send notification via SNS"""
    try:
        response = sns.publish(
            TopicArn=TOPIC_ARN,
            Subject=f"🍽️ {subject} - Order #{order_id}",
            Message=message
        )
        print(f"✅ Notification sent: {subject}")
        return response['MessageId']
    except Exception as e:
        print(f"❌ Failed to send notification: {e}")
        return None

def process_order_stages(order_data):
    """Process order through different stages"""
    order_id = order_data.get('orderId', 'UNKNOWN')
    customer_name = order_data.get('customerName', 'Customer')
    items = order_data.get('items', [])
    total = order_data.get('totalAmount', 0)
    
    # Order processing stages
    stages = [
        {
            "stage": "confirmed",
            "subject": "Order Confirmed! 🎉",
            "message": f"Hi {customer_name}!\n\nYour order has been confirmed!\n\nOrder Details:\n- Items: {', '.join(items)}\n- Total: ${total}\n- Order ID: {order_id}\n\nWe'll keep you updated on the progress!",
            "delay": 3
        },
        {
            "stage": "preparing",
            "subject": "Preparing Your Food 👨‍🍳",
            "message": f"Good news {customer_name}!\n\nOur chef is now preparing your delicious meal:\n{', '.join(items)}\n\nEstimated preparation time: 12-15 minutes",
            "delay": 10
        },
        {
            "stage": "ready",
            "subject": "Food Ready! ✅",
            "message": f"Great news {customer_name}!\n\nYour order is ready for pickup/delivery!\n\nOrder #{order_id}\nItems: {', '.join(items)}\n\nPlease come to the counter or expect delivery soon!",
            "delay": 8
        },
        {
            "stage": "completed",
            "subject": "Order Completed! 🎊",
            "message": f"Thank you {customer_name}!\n\nYour order has been completed successfully!\n\nWe hope you enjoy your meal. Don't forget to rate us!\n\nOrder #{order_id} - Total: ${total}",
            "delay": 0
        }
    ]
    
    # Process each stage
    for stage in stages:
        send_notification(stage["subject"], stage["message"], order_id)
        
        if stage["delay"] > 0:
            print(f"⏳ Waiting {stage['delay']} seconds before next stage...")
            time.sleep(stage["delay"])
    
    return f"Order {order_id} processed successfully!"

def lambda_handler(event, context):
    """Main Lambda handler - can be triggered by SQS or direct invoke"""
    
    try:
        # Check if this is an SQS event
        if 'Records' in event:
            print("📨 Processing SQS messages...")
            
            for record in event['Records']:
                # Parse the SQS message
                message_body = json.loads(record['body'])
                print(f"📦 Processing order: {message_body}")
                
                # Process the order
                result = process_order_stages(message_body)
                print(result)
                
        else:
            # Direct invocation - create a sample order
            print("🧪 Direct invocation - creating sample order...")
            
            sample_order = {
                "orderId": f"ORD-{int(datetime.now().timestamp())}",
                "customerName": "John Doe",
                "items": ["Chicken Burger", "Large Fries", "Coke"],
                "totalAmount": 18.50,
                "orderType": "dine-in",
                "timestamp": datetime.now().isoformat()
            }
            
            # Send sample order to SQS first
            sqs_response = sqs.send_message(
                QueueUrl=QUEUE_URL,
                MessageBody=json.dumps(sample_order)
            )
            
            print(f"📤 Sample order sent to queue: {sqs_response['MessageId']}")
            
            # Then process it
            result = process_order_stages(sample_order)
            print(result)
        
        return {
            'statusCode': 200,
            'body': json.dumps({
                'message': 'Orders processed successfully',
                'timestamp': datetime.now().isoformat()
            })
        }
        
    except Exception as e:
        print(f"❌ Error processing orders: {e}")
        return {
            'statusCode': 500,
            'body': json.dumps({
                'error': str(e),
                'timestamp': datetime.now().isoformat()
            })
        }
```

> **Critical:** Replace QUEUE_URL and TOPIC_ARN with your actual values from steps 1 and 2!

### Configure Lambda Settings

1. Go to **Configuration → General Configuration**
2. Set **Timeout** to `2 minutes`
3. Go to **Configuration → Permissions**
4. Click on the execution role
5. Add **AmazonSNSFullAccess** and **AmazonSQSFullAccess** policies

## 🔗 Step 4: Connect SQS to Lambda (Auto-Processing)

### Add SQS Trigger

1. In your Lambda function, click **Add Trigger**
2. Select **SQS**
3. Choose your **CanteenOrderQueue**
4. Batch size: `1` (process one order at a time)
5. Click **Add**

> **Now it's fully automated!** Any message sent to SQS will automatically trigger the Lambda function!

## 🧪 Step 5: Test the Complete System

### Test with Direct Lambda Call

1. In Lambda console, click **Test**
2. Create new test event (default JSON is fine)
3. Click **Test**
4. Watch the logs and check your email!

**What happens:**
- Lambda creates a sample order
- Sends it to SQS queue
- SQS triggers Lambda again
- Lambda processes and sends SNS notifications
- You get 4 emails showing order progress!

### Test with Node.js Order Sender (Optional)

Create a simple order sender to test the full flow:

```javascript
// order-sender.js
const { SQSClient, SendMessageCommand } = require("@aws-sdk/client-sqs");

const client = new SQSClient({ region: "us-east-1" });
const QUEUE_URL = "https://sqs.us-east-1.amazonaws.com/123456789012/CanteenOrderQueue";

async function placeOrder(customerName, items, totalAmount) {
    const order = {
        orderId: `ORD-${Date.now()}`,
        customerName: customerName,
        items: items,
        totalAmount: totalAmount,
        orderType: "online",
        timestamp: new Date().toISOString()
    };

    const command = new SendMessageCommand({
        QueueUrl: QUEUE_URL,
        MessageBody: JSON.stringify(order)
    });

    try {
        const response = await client.send(command);
        console.log(`🎉 Order placed successfully!`);
        console.log(`📧 Message ID: ${response.MessageId}`);
        console.log(`📦 Order: ${JSON.stringify(order, null, 2)}`);
        console.log(`📬 The order will be automatically processed and you'll receive email updates!`);
    } catch (error) {
        console.error('❌ Error placing order:', error);
    }
}

// Place a test order
placeOrder("Alice Johnson", ["Pizza Margherita", "Garlic Bread", "Coke"], 24.99);
```

Setup and run:
```bash
npm install @aws-sdk/client-sqs
node order-sender.js
```

## 🔄 Complete System Flow

### What Happens When You Place an Order

1. **Order Placed** → Sent to SQS Queue
2. **SQS Triggers Lambda** → Automatic processing begins
3. **Lambda Processes Order** → Goes through 4 stages
4. **SNS Sends Emails** → Customer gets real-time updates
5. **Order Complete** → Customer satisfaction!

> **The Beauty:** Once set up, the entire system runs automatically. Orders flow from SQS → Lambda → SNS without any manual intervention!

## 🚀 Next Level Enhancements

### Make it Production Ready

- **Web Frontend:** Create a React order form
- **API Gateway:** RESTful API for order placement
- **DynamoDB:** Store order history and tracking
- **SMS Notifications:** Add SMS via SNS
- **Payment Integration:** Stripe/PayPal integration
- **Admin Dashboard:** Monitor orders in real-time
- **Error Handling:** Dead letter queues for failed orders

## 🔧 Common Issues & Solutions

### ❌ Lambda Not Triggered by SQS
Check that SQS trigger is properly configured and Lambda has SQS permissions.

### ❌ No Email Notifications
Verify SNS topic ARN is correct and email subscription is confirmed.

### ❌ Lambda Timeout
Increase timeout to 2-3 minutes for the complete order flow.
