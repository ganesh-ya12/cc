Absolutely bro 💥 Here's your **clean, ready-to-paste `README.md` in Markdown format**, fully covering **Experiment 11** – a complete **Canteen Management System using AWS SQS**, integrated with **DynamoDB, SNS, Lambda, IAM, and API Gateway**.

---

````markdown
# 🍽️ Experiment 11 – Canteen Management System using AWS SQS

Build a real-world canteen order management system using:
- **Amazon SQS** for messaging queues
- **Amazon DynamoDB** for data storage
- **Amazon SNS** for notifications (email & SMS)
- **AWS Lambda** for backend logic
- **API Gateway** for frontend communication

---

## 📦 Step 1: Create DynamoDB Tables

### 🗃️ a) `Orders` Table

| Key | Type |
|-----|------|
| `orderId` (Partition Key) | String |

**Attributes:**
- `customerEmail` (String)
- `status` (String) → `"Placed" → "Accepted" → "Ready" → "Delivered"`
- `prepTime` (Number)
- `deliveryBoyId` (String)

---

### 🗃️ b) `DeliveryStatus` Table

| Key | Type |
|-----|------|
| `deliveryId` (Partition Key) | String |

**Attributes:**
- `orderId` (String)
- `status` (String) → `"PickedUp"`, `"OnTheWay"`, `"ReachedDestination"`

---

### 🗃️ c) `Users` Table

| Key | Type |
|-----|------|
| `userId` (Partition Key) | String |

**Attributes:**
- `role` (String): `Customer`, `Owner`, `DeliveryBoy`
- `email` (String)
- `phone` (String)

---

## 📢 Step 2: Create SNS Topics

### 🔔 a) `OrderStatusTopic`
- Type: **Standard**
- Subscribers: Customer email addresses (for order updates)

### 🚚 b) `DeliveryNotificationsTopic`
- Subscribers:
  - Email (Customer)
  - SMS (Customer for delivery alerts)

### ⭐ c) `FeedbackTopic`
- Subscribers:
  - SMS only (for feedback request after delivery)

---

## 📨 Step 3: Create SQS Queues

| Queue Name | Used By | Purpose |
|------------|---------|---------|
| `OwnerQueue` | Owner | Get notified when order is placed |
| `DeliveryBoyQueue` | Delivery Boy | Get notified when food is ready |

---

## 💻 Step 4: Create Lambda Functions

---

### 🧾 1. `placeOrderLambda`  
**Trigger:** API Gateway  
**Action:**
- Generates `orderId`
- Saves order to DynamoDB
- Sends message to `OwnerQueue`

```python
import boto3, json, uuid

def lambda_handler(event, context):
    orderId = str(uuid.uuid4())
    body = json.loads(event['body'])
    customerEmail = body['email']

    dynamo = boto3.resource('dynamodb')
    table = dynamo.Table('Orders')
    table.put_item(Item={
        'orderId': orderId,
        'customerEmail': customerEmail,
        'status': 'Placed'
    })

    sqs = boto3.client('sqs')
    sqs.send_message(
        QueueUrl='https://sqs.us-east-1.amazonaws.com/443369304872/OwnerQueue',
        MessageBody=json.dumps({'orderId': orderId})
    )

    return {'statusCode': 200, 'body': 'Order Placed'}
````

---

### 🧾 2. `processOwnerQueueLambda`

**Trigger:** SQS (`OwnerQueue`)
**Action:**

* Updates order status to `"Accepted"`
* Sets `prepTime`
* Sends notification via `OrderStatusTopic`

```python
import boto3, json

def lambda_handler(event, context):
    dynamo = boto3.resource('dynamodb')
    table = dynamo.Table('Orders')
    sns = boto3.client('sns')

    for record in event['Records']:
        msg = json.loads(record['body'])
        orderId = msg['orderId']
        prepTime = 15

        table.update_item(
            Key={'orderId': orderId},
            UpdateExpression='SET #s = :s, prepTime = :p',
            ExpressionAttributeNames={'#s': 'status'},
            ExpressionAttributeValues={':s': 'Accepted', ':p': prepTime}
        )

        order = table.get_item(Key={'orderId': orderId})['Item']
        sns.publish(
            TopicArn='arn:aws:sns:us-east-1:443369304872:OrderStatusTopic',
            Message=f"Order accepted! Ready in {prepTime} minutes.",
            Subject="Order Accepted"
        )
```

---

### 🧾 3. `foodReadyLambda`

**Trigger:** API Gateway (owner dashboard)
**Action:**

* Updates order status to `"Ready"`
* Notifies customer via email
* Sends message to `DeliveryBoyQueue`

```python
import boto3, json

def lambda_handler(event, context):
    body = json.loads(event['body'])
    orderId = body['orderId']

    dynamo = boto3.resource('dynamodb')
    table = dynamo.Table('Orders')
    table.update_item(
        Key={'orderId': orderId},
        UpdateExpression='SET #s = :s',
        ExpressionAttributeNames={'#s': 'status'},
        ExpressionAttributeValues={':s': 'Ready'}
    )

    sns = boto3.client('sns')
    sns.publish(
        TopicArn='arn:aws:sns:us-east-1:443369304872:OrderStatusTopic',
        Message="Your order is ready! Being handed to delivery boy.",
        Subject="Food Ready"
    )

    sqs = boto3.client('sqs')
    sqs.send_message(
        QueueUrl='https://sqs.us-east-1.amazonaws.com/443369304872/DeliveryBoyQueue',
        MessageBody=json.dumps({'orderId': orderId})
    )

    return {
        'statusCode': 200,
        'body': json.dumps('Marked as Ready')
    }
```

---

### 🧾 4. `processDeliveryQueueLambda`

**Trigger:** SQS (`DeliveryBoyQueue`)
**Action:**

* Assigns delivery boy
* Sends delivery started notification

```python
import boto3, json

def lambda_handler(event, context):
    dynamo = boto3.resource('dynamodb')
    table = dynamo.Table('Orders')
    sns = boto3.client('sns')

    for record in event['Records']:
        msg = json.loads(record['body'])
        orderId = msg['orderId']
        deliveryBoyId = 'DELIVERY-123'

        table.update_item(
            Key={'orderId': orderId},
            UpdateExpression='SET deliveryBoyId = :d',
            ExpressionAttributeValues={':d': deliveryBoyId}
        )

        sns.publish(
            TopicArn='arn:aws:sns:us-east-1:443369304872:DeliveryNotificationsTopic',
            Message=f"Delivery boy assigned and en route for order {orderId}",
            Subject="Delivery Started"
        )
```

---

### 🧾 5. `deliveryBoyUpdatesLambda`

**Trigger:** API Gateway (Delivery Boy app)
**Action:**

* Sends arrival notification (email + SMS)
* Triggers feedback request via SMS

```python
import boto3, json

def lambda_handler(event, context):
    body = json.loads(event['body'])
    orderId = body['orderId']
    sns = boto3.client('sns')

    sns.publish(
        TopicArn='arn:aws:sns:us-east-1:443369304872:DeliveryNotificationsTopic',
        Message=f"Delivery boy has reached your location for order {orderId}.",
        Subject="Order Arrived"
    )

    sns.publish(
        TopicArn='arn:aws:sns:us-east-1:443369304872:FeedbackTopic',
        Message="Thank you for ordering! Please share your feedback."
    )

    return {
        'statusCode': 200,
        'body': json.dumps('Customer notified')
    }
```

---

## 🔐 Step 5: IAM Roles for Lambda

Each Lambda function must have IAM permissions:

### ✅ Required Policies:

* `AmazonDynamoDBFullAccess`
* `AmazonSQSFullAccess`
* `AmazonSNSFullAccess`
* (Optional) `CloudWatchLogsFullAccess`

### 🧩 Attach Roles:

1. Go to [IAM Console](https://console.aws.amazon.com/iam)
2. Create a new **role** for **Lambda**
3. Attach required policies
4. Go to **Lambda Console → Configuration → Permissions**
5. Attach your IAM role to each function

---

## 🌐 Step 6: API Gateway Setup

Create API endpoints to trigger Lambdas:

| Endpoint          | Method | Lambda                     |
| ----------------- | ------ | -------------------------- |
| `/placeOrder`     | POST   | `placeOrderLambda`         |
| `/markReady`      | POST   | `foodReadyLambda`          |
| `/deliveryUpdate` | POST   | `deliveryBoyUpdatesLambda` |

Enable **CORS** if using from frontend

---

## 🧪 API Testing

### 🟢 1. `/placeOrder`

```http
POST https://your-api-url/prod/placeOrder

Body:
{
  "email": "john.doe@example.com"
}
```

* ✅ Order placed
* ✅ Stored in DynamoDB
* ✅ SQS message sent

---

### 🟢 2. `/markReady`

```http
POST https://your-api-url/prod/markReady

Body:
{
  "orderId": "your-real-order-id"
}
```

* ✅ Status updated to `Ready`
* ✅ Email notification sent
* ✅ Message sent to `DeliveryBoyQueue`

---

### 🟢 3. `/deliveryUpdate`

```http
POST https://your-api-url/prod/deliveryUpdate

Body:
{
  "orderId": "your-real-order-id"
}
```

* ✅ Delivery boy arrival notification sent
* ✅ Feedback SMS sent

---

## ✅ Summary

| Component            | Tech        |
| -------------------- | ----------- |
| Order Status Updates | SNS         |
| Messaging Queues     | SQS         |
| Data Storage         | DynamoDB    |
| Business Logic       | Lambda      |
| API Interface        | API Gateway |
| Security             | IAM Roles   |

---

```

Let me know if you want this zipped or exported as a real `.md` file — or if you want the full repo scaffold (like `/lambda`, `/infra`, `/api`, etc.). Let's go 💣🔥
```
