<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Integrated Canteen Order System - AWS SNS & SQS</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }
        
        body {
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
            line-height: 1.6;
            color: #333;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            min-height: 100vh;
            padding: 20px;
        }
        
        .container {
            max-width: 1000px;
            margin: 0 auto;
            background: white;
            border-radius: 20px;
            box-shadow: 0 20px 40px rgba(0,0,0,0.1);
            overflow: hidden;
        }
        
        .header {
            background: linear-gradient(135deg, #ff6b6b, #ee5a24);
            color: white;
            padding: 40px;
            text-align: center;
        }
        
        .header h1 {
            font-size: 2.5rem;
            margin-bottom: 10px;
            display: flex;
            align-items: center;
            justify-content: center;
            gap: 15px;
        }
        
        .header p {
            font-size: 1.2rem;
            opacity: 0.9;
        }
        
        .content {
            padding: 40px;
        }
        
        .section {
            margin-bottom: 40px;
            padding: 30px;
            background: #f8f9fa;
            border-radius: 15px;
            border-left: 5px solid #667eea;
        }
        
        .section h2 {
            color: #2c3e50;
            margin-bottom: 20px;
            display: flex;
            align-items: center;
            gap: 10px;
            font-size: 1.8rem;
        }
        
        .step {
            background: white;
            padding: 25px;
            margin: 20px 0;
            border-radius: 10px;
            box-shadow: 0 5px 15px rgba(0,0,0,0.08);
            border-left: 4px solid #00d2d3;
        }
        
        .step h3 {
            color: #2c3e50;
            margin-bottom: 15px;
            display: flex;
            align-items: center;
            gap: 10px;
        }
        
        .step-number {
            background: #00d2d3;
            color: white;
            width: 30px;
            height: 30px;
            border-radius: 50%;
            display: flex;
            align-items: center;
            justify-content: center;
            font-weight: bold;
            font-size: 0.9rem;
        }
        
        .code-block {
            background: #1e1e1e;
            color: #d4d4d4;
            padding: 20px;
            border-radius: 8px;
            margin: 15px 0;
            position: relative;
            overflow-x: auto;
            font-family: 'Monaco', 'Menlo', 'Ubuntu Mono', monospace;
            font-size: 14px;
            line-height: 1.5;
        }
        
        .copy-btn {
            position: absolute;
            top: 10px;
            right: 10px;
            background: #007acc;
            color: white;
            border: none;
            padding: 8px 12px;
            border-radius: 5px;
            cursor: pointer;
            font-size: 12px;
            transition: all 0.3s ease;
        }
        
        .copy-btn:hover {
            background: #005a9e;
            transform: translateY(-2px);
        }
        
        .copy-btn.copied {
            background: #28a745;
        }
        
        .important-note {
            background: #fff3cd;
            border: 1px solid #ffeaa7;
            border-radius: 8px;
            padding: 15px;
            margin: 15px 0;
            border-left: 4px solid #fdcb6e;
        }
        
        .important-note strong {
            color: #b8860b;
        }
        
        .integration-flow {
            background: linear-gradient(135deg, #667eea, #764ba2);
            color: white;
            padding: 30px;
            border-radius: 15px;
            margin: 25px 0;
        }
        
        .flow-steps {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
            gap: 20px;
            margin-top: 20px;
        }
        
        .flow-step {
            background: rgba(255,255,255,0.1);
            padding: 20px;
            border-radius: 10px;
            text-align: center;
            backdrop-filter: blur(10px);
        }
        
        .flow-step h4 {
            margin-bottom: 10px;
            font-size: 1.1rem;
        }
        
        .flow-step p {
            font-size: 0.9rem;
            opacity: 0.9;
        }
        
        .tech-stack {
            display: flex;
            gap: 15px;
            flex-wrap: wrap;
            margin: 20px 0;
        }
        
        .tech-item {
            background: #e3f2fd;
            color: #1565c0;
            padding: 8px 16px;
            border-radius: 20px;
            font-weight: 500;
            font-size: 0.9rem;
        }
        
        @media (max-width: 768px) {
            .container {
                margin: 10px;
                border-radius: 15px;
            }
            
            .header {
                padding: 30px 20px;
            }
            
            .header h1 {
                font-size: 2rem;
                flex-direction: column;
                gap: 10px;
            }
            
            .content {
                padding: 20px;
            }
            
            .section {
                padding: 20px;
            }
            
            .flow-steps {
                grid-template-columns: 1fr;
            }
        }
    </style>
</head>
<body>
    <div class="container">
        <div class="header">
            <h1>
                🍽️ Integrated Canteen Order System
            </h1>
            <p>Complete order flow with AWS SNS + SQS working together</p>
        </div>
        
        <div class="content">
            <!-- Integration Overview -->
            <div class="integration-flow">
                <h2 style="color: white; margin-bottom: 20px;">🔄 How It All Works Together</h2>
                <div class="flow-steps">
                    <div class="flow-step">
                        <h4>1. Order Placed</h4>
                        <p>Customer places order → Sent to SQS Queue</p>
                    </div>
                    <div class="flow-step">
                        <h4>2. Order Processing</h4>
                        <p>Lambda processes queue → Sends SNS notifications</p>
                    </div>
                    <div class="flow-step">
                        <h4>3. Real-time Updates</h4>
                        <p>Customer gets email updates at each stage</p>
                    </div>
                    <div class="flow-step">
                        <h4>4. Complete Flow</h4>
                        <p>Order → Queue → Process → Notify → Complete</p>
                    </div>
                </div>
            </div>

            <!-- Prerequisites -->
            <div class="section">
                <h2>📋 Prerequisites</h2>
                <div class="step">
                    <h3><span class="step-number">✓</span> What You Need</h3>
                    <div class="tech-stack">
                        <div class="tech-item">AWS Account</div>
                        <div class="tech-item">Valid Email</div>
                        <div class="tech-item">Node.js (Optional)</div>
                        <div class="tech-item">AWS Console Access</div>
                    </div>
                </div>
            </div>

            <!-- Step 1: SQS Queue Setup -->
            <div class="section">
                <h2>📬 Step 1: Create SQS Queue (Order Intake)</h2>
                
                <div class="step">
                    <h3><span class="step-number">1</span> Create the Order Queue</h3>
                    <ol style="margin: 15px 0 15px 20px; line-height: 1.8;">
                        <li>Go to <strong>SQS Console</strong></li>
                        <li>Click <strong>Create Queue</strong></li>
                        <li>Name: <code>CanteenOrderQueue</code></li>
                        <li>Type: <strong>Standard</strong></li>
                        <li>Click <strong>Create Queue</strong></li>
                        <li><strong>Save the Queue URL</strong> - you'll need it!</li>
                    </ol>
                </div>
            </div>

            <!-- Step 2: SNS Topic Setup -->
            <div class="section">
                <h2>📧 Step 2: Create SNS Topic (Notifications)</h2>
                
                <div class="step">
                    <h3><span class="step-number">2</span> Create Notification Topic</h3>
                    <ol style="margin: 15px 0 15px 20px; line-height: 1.8;">
                        <li>Go to <strong>SNS Console</strong></li>
                        <li>Click <strong>Create Topic</strong></li>
                        <li>Name: <code>CanteenNotifications</code></li>
                        <li>Type: <strong>Standard</strong></li>
                        <li>Click <strong>Create Topic</strong></li>
                        <li><strong>Save the Topic ARN</strong></li>
                    </ol>
                </div>

                <div class="step">
                    <h3><span class="step-number">3</span> Add Email Subscription</h3>
                    <ol style="margin: 15px 0 15px 20px; line-height: 1.8;">
                        <li>Open your <strong>CanteenNotifications</strong> topic</li>
                        <li>Click <strong>Create Subscription</strong></li>
                        <li>Protocol: <strong>Email</strong></li>
                        <li>Endpoint: Your email address</li>
                        <li>Confirm subscription via email</li>
                    </ol>
                </div>
            </div>

            <!-- Step 3: Integrated Lambda Function -->
            <div class="section">
                <h2>⚡ Step 3: Create Processing Lambda (The Brain)</h2>
                
                <div class="step">
                    <h3><span class="step-number">4</span> Create Lambda Function</h3>
                    <ol style="margin: 15px 0 15px 20px; line-height: 1.8;">
                        <li>Go to <strong>Lambda Console</strong></li>
                        <li>Click <strong>Create Function</strong></li>
                        <li>Name: <code>CanteenOrderProcessor</code></li>
                        <li>Runtime: <strong>Python 3.9</strong></li>
                        <li>Create with basic execution role</li>
                    </ol>
                </div>

                <div class="step">
                    <h3><span class="step-number">5</span> Add the Integrated Code</h3>
                    <p>This Lambda will process SQS messages AND send SNS notifications:</p>
                    
                    <div class="code-block">
                        <button class="copy-btn" onclick="copyCode(this)">Copy</button>
                        <pre>import json
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
        }</pre>
                    </div>
                    
                    <div class="important-note">
                        <strong>Critical:</strong> Replace QUEUE_URL and TOPIC_ARN with your actual values from steps 1 and 2!
                    </div>
                </div>

                <div class="step">
                    <h3><span class="step-number">6</span> Configure Lambda Settings</h3>
                    <ol style="margin: 15px 0 15px 20px; line-height: 1.8;">
                        <li>Go to <strong>Configuration → General Configuration</strong></li>
                        <li>Set <strong>Timeout</strong> to <code>2 minutes</code></li>
                        <li>Go to <strong>Configuration → Permissions</strong></li>
                        <li>Click on the execution role</li>
                        <li>Add <strong>AmazonSNSFullAccess</strong> and <strong>AmazonSQSFullAccess</strong> policies</li>
                    </ol>
                </div>
            </div>

            <!-- Step 4: Connect SQS to Lambda -->
            <div class="section">
                <h2>🔗 Step 4: Connect SQS to Lambda (Auto-Processing)</h2>
                
                <div class="step">
                    <h3><span class="step-number">7</span> Add SQS Trigger</h3>
                    <ol style="margin: 15px 0 15px 20px; line-height: 1.8;">
                        <li>In your Lambda function, click <strong>Add Trigger</strong></li>
                        <li>Select <strong>SQS</strong></li>
                        <li>Choose your <strong>CanteenOrderQueue</strong></li>
                        <li>Batch size: <code>1</code> (process one order at a time)</li>
                        <li>Click <strong>Add</strong></li>
                    </ol>
                    
                    <div class="important-note">
                        <strong>Now it's fully automated!</strong> Any message sent to SQS will automatically trigger the Lambda function!
                    </div>
                </div>
            </div>

            <!-- Step 5: Testing -->
            <div class="section">
                <h2>🧪 Step 5: Test the Complete System</h2>
                
                <div class="step">
                    <h3><span class="step-number">8</span> Test with Direct Lambda Call</h3>
                    <ol style="margin: 15px 0 15px 20px; line-height: 1.8;">
                        <li>In Lambda console, click <strong>Test</strong></li>
                        <li>Create new test event (default JSON is fine)</li>
                        <li>Click <strong>Test</strong></li>
                        <li>Watch the logs and check your email!</li>
                    </ol>
                    
                    <p style="margin-top: 15px;"><strong>What happens:</strong></p>
                    <ul style="margin-left: 20px; line-height: 1.8;">
                        <li>Lambda creates a sample order</li>
                        <li>Sends it to SQS queue</li>
                        <li>SQS triggers Lambda again</li>
                        <li>Lambda processes and sends SNS notifications</li>
                        <li>You get 4 emails showing order progress!</li>
                    </ul>
                </div>

                <div class="step">
                    <h3><span class="step-number">9</span> Test with Node.js Order Sender (Optional)</h3>
                    <p>Create a simple order sender to test the full flow:</p>
                    
                    <div class="code-block">
                        <button class="copy-btn" onclick="copyCode(this)">Copy</button>
                        <pre>// order-sender.js
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
placeOrder("Alice Johnson", ["Pizza Margherita", "Garlic Bread", "Coke"], 24.99);</pre>
                    </div>
                    
                    <div class="code-block">
                        <button class="copy-btn" onclick="copyCode(this)">Copy</button>
                        <pre># Setup and run
npm install @aws-sdk/client-sqs
node order-sender.js</pre>
                    </div>
                </div>
            </div>

            <!-- System Flow -->
            <div class="section">
                <h2>🔄 Complete System Flow</h2>
                <div class="step">
                    <h3>🎯 What Happens When You Place an Order</h3>
                    <ol style="margin: 15px 0 15px 20px; line-height: 2;">
                        <li><strong>Order Placed</strong> → Sent to SQS Queue</li>
                        <li><strong>SQS Triggers Lambda</strong> → Automatic processing begins</li>
                        <li><strong>Lambda Processes Order</strong> → Goes through 4 stages</li>
                        <li><strong>SNS Sends Emails</strong> → Customer gets real-time updates</li>
                        <li><strong>Order Complete</strong> → Customer satisfaction!</li>
                    </ol>
                    
                    <div class="important-note">
                        <strong>The Beauty:</strong> Once set up, the entire system runs automatically. Orders flow from SQS → Lambda → SNS without any manual intervention!
                    </div>
                </div>
            </div>

            <!-- Next Steps -->
            <div class="section">
                <h2>🚀 Next Level Enhancements</h2>
                <div class="step">
                    <h3>🎯 Make it Production Ready</h3>
                    <ul style="margin-left: 20px; line-height: 2;">
                        <li><strong>Web Frontend:</strong> Create a React order form</li>
                        <li><strong>API Gateway:</strong> RESTful API for order placement</li>
                        <li><strong>DynamoDB:</strong> Store order history and tracking</li>
                        <li><strong>SMS Notifications:</strong> Add SMS via SNS</li>
                        <li><strong>Payment Integration:</strong> Stripe/PayPal integration</li>
                        <li><strong>Admin Dashboard:</strong> Monitor orders in real-time</li>
                        <li><strong>Error Handling:</strong> Dead letter queues for failed orders</li>
                    </ul>
                </div>
            </div>

            <!-- Troubleshooting -->
            <div class="section">
                <h2>🔧 Common Issues & Solutions</h2>
                <div class="step">
                    <h3>❌ Lambda Not Triggered by SQS</h3>
                    <p>Check that SQS trigger is properly configured and Lambda has SQS permissions.</p>
                </div>
                
                <div class="step">
                    <h3>❌ No Email Notifications</h3>
                    <p>Verify SNS topic ARN is correct and email subscription is confirmed.</p>
                </div>
                
                <div class="step">
                    <h3>❌ Lambda Timeout</h3>
                    <p>Increase timeout to 2-3 minutes for the complete order flow.</p>
                </div>
            </div>
        </div>
    </div>

    <script>
        function copyCode(button) {
            const codeBlock = button.nextElementSibling;
            const code = codeBlock.textContent;
            
            navigator.clipboard.writeText(code).then(function() {
                button.textContent = 'Copied!';
                button.classList.add('copied');
                
                setTimeout(function() {
                    button.textContent = 'Copy';
                    button.classList.remove('copied');
                }, 2000);
            }).catch(function(err) {
                console.error('Could not copy text: ', err);
                button.textContent = 'Error!';
                setTimeout(function() {
                    button.textContent = 'Copy';
                }, 2000);
            });
        }
    </script>
</body>
</html>