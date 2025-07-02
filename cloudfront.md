# Experiment 14: CloudFront Distribution for React Feedback App

This experiment demonstrates setting up a CloudFront distribution to serve a React feedback application from an S3 bucket, with a complete serverless backend using Lambda, API Gateway, and DynamoDB.

## Architecture Overview

- **Frontend**: React application hosted on S3
- **CDN**: CloudFront distribution for global content delivery
- **API**: API Gateway for REST endpoints
- **Backend**: Lambda function for processing
- **Database**: DynamoDB for data storage

## Step 1: Create React Feedback Application

### 1.1 Initialize React App

```bash
npx create-react-app feedback
cd feedback
```

### 1.2 Install Dependencies

```bash
npm install axios
```

### 1.3 Create Feedback Component (src/App.js)

```javascript
import React, { useState } from 'react';
import axios from 'axios';
import './App.css';

function App() {
  const [rollNo, setRollNo] = useState('');
  const [name, setName] = useState('');
  const [subject, setSubject] = useState('Math');
  const [rating, setRating] = useState('5');

  const handleSubmit = async (e) => {
    e.preventDefault();
    const data = { rollNo, name, subject, rating };
    
    try {
      await axios.post('https://<API_GATEWAY_URL>/feedback', data);
      alert('Feedback submitted!');
    } catch (error) {
      console.error(error);
      alert('Submission failed.');
    }
  };

  return (
    <div className="App">
      <h1>Feedback Form</h1>
      <form onSubmit={handleSubmit}>
        <input 
          placeholder="Roll No" 
          value={rollNo} 
          onChange={(e) => setRollNo(e.target.value)} 
          required 
        /><br/><br/>
        
        <input 
          placeholder="Name" 
          value={name} 
          onChange={(e) => setName(e.target.value)} 
          required 
        /><br/><br/>
        
        <select value={subject} onChange={(e) => setSubject(e.target.value)}>
          <option value="Math">Math</option>
          <option value="Science">Science</option>
          <option value="English">English</option>
        </select><br/>
        </br><br/>
        
        <select value={rating} onChange={(e) => setRating(e.target.value)}>
          {[1, 2, 3, 4, 5].map(n => <option key={n} value={n}>{n}</option>)}
        </select><br/><br/>
        
        <button type="submit">Submit</button>
      </form>
    </div>
  );
}

export default App;
```

### 1.4 Build the Application

```bash
npm run build
```

## Step 2: Create and Configure S3 Bucket

### 2.1 Create S3 Bucket

1. Go to S3 → Create bucket
2. Name: `feedback-app-bucket`
3. Region: Select your preferred region
4. **Uncheck** "Block all public access"

### 2.2 Enable Static Website Hosting

1. Go to Properties → Static website hosting
2. Enable and configure:
   - Index document: `index.html`
   - Error document: `index.html`

### 2.3 Upload React Build Files

Upload all files from the `build/` folder to the S3 bucket.

### 2.4 Add Bucket Policy for Public Read Access

Go to Permissions → Bucket Policy and add:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "Statement1",
      "Effect": "Allow",
      "Principal": "*",
      "Action": [
        "s3:GetObject"
      ],
      "Resource": "arn:aws:s3:::feedback-app-bucket/*"
    }
  ]
}
```

**Note**: Replace `feedback-app-bucket` with your actual bucket name.

## Step 3: Setup CloudFront CDN

### 3.1 Create CloudFront Distribution

1. Go to CloudFront → Create Distribution
2. **Origin Domain**: Select the S3 bucket's static website endpoint
3. **Viewer Protocol Policy**: Redirect HTTP to HTTPS
4. **Default root object**: `index.html`
5. Create distribution and copy the CloudFront domain name

## Step 4: Create DynamoDB Table

1. Go to DynamoDB → Create Table
2. **Table name**: `FeedbackTable-22bd1a057u`
3. **Partition key**: `id` (String)
4. Leave other settings as default

## Step 5: Create Lambda Function

### 5.1 Lambda Function Code

```python
import json
import boto3
import uuid

dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('FeedbackTable-22bd1a057y')

def lambda_handler(event, context):
    body = json.loads(event['body'])
    
    item = {
        'id': str(uuid.uuid4()),
        'rollNo': body['rollNo'],
        'name': body['name'],
        'subject': body['subject'],
        'rating': body['rating']
    }
    
    table.put_item(Item=item)
    
    return {
        'statusCode': 200,
        'headers': { 'Access-Control-Allow-Origin': '*' },
        'body': json.dumps({ 'message': 'Feedback received!' })
    }
```

### 5.2 Create Lambda Function

1. Go to Lambda → Create function
2. **Name**: `SubmitFeedbackFunction`
3. **Runtime**: Python 3.9
4. Paste the code above

### 5.3 Add IAM Permissions

1. Go to Configuration → Permissions → Execution role
2. Attach `AmazonDynamoDBFullAccess` policy

## Step 6: Create API Gateway

### 6.1 Create HTTP API

1. Go to API Gateway → Create API → HTTP API
2. **Add Integration**: Lambda Function (`SubmitFeedbackFunction`)
3. **Route**: `POST /feedback`
4. **Stage**: Default

### 6.2 Enable CORS

1. Go to the "Routes" section
2. Select the route: `POST /feedback`
3. Click "Enable CORS"
4. Configure:
   - **Access-Control-Allow-Headers**: `Content-Type`
   - **Access-Control-Allow-Methods**: `POST`
   - **Access-Control-Allow-Origin**: `*`
5. Click "Add CORS configuration"

### 6.3 Deploy and Get API URL

Copy the endpoint URL (e.g., `https://0aiqr0bbb8.execute-api.us-east-1.amazonaws.com/`)

## Step 7: Update and Redeploy Frontend

### 7.1 Update API URL in React Code

Replace `<API_GATEWAY_URL>` in `src/App.js` with your actual API Gateway URL.

### 7.2 Rebuild and Redeploy

```bash
npm run build
```

Upload the new build files to S3, replacing the previous ones.

### 7.3 Access the Application

Your application is now available via the CloudFront URL:
`https://d1ezjgb2jgubot.cloudfront.net/`

## Testing the Application

1. Open the CloudFront URL in your browser
2. Fill out the feedback form:
   - Roll No
   - Name
   - Subject (Math/Science/English)
   - Rating (1-5)
3. Submit the form
4. Check DynamoDB table to verify data storage

## Key Benefits of This Architecture

- **Global Performance**: CloudFront provides low-latency access worldwide
- **Scalability**: Serverless architecture scales automatically
- **Cost-Effective**: Pay only for what you use
- **Security**: HTTPS enforcement and proper CORS configuration
- **Reliability**: Multiple AWS services provide high availability

## Troubleshooting

- **CORS Issues**: Ensure CORS is properly configured in API Gateway
- **403 Errors**: Check S3 bucket policy and public access settings
- **Lambda Errors**: Verify IAM permissions for DynamoDB access
- **API Issues**: Check API Gateway integration and deployment

## Architecture Diagram

```
User → CloudFront → S3 (Static Website)
                ↓
User Form → API Gateway → Lambda → DynamoDB
```

This setup provides a complete serverless web application with global content delivery capabilities.