# Experiment 8: Amazon Lex Chatbot for Gold Rate Web Scraping

This experiment demonstrates how to create an Amazon Lex chatbot that retrieves the latest gold rates using web scraping through AWS Lambda.

## Prerequisites

- AWS Account with proper permissions
- Command line access
- Python 3.13 runtime support

## Step 1: Create Lambda Layer Structure

Open command prompt and run the following commands to create the layer structure and install required packages:

```bash
mkdir -p python
pip install requests -t python/
pip install beautifulsoup4 -t python/
cd ..
zip -r my-layer.zip my-layer/
```

## Step 2: Create Lambda Layer

1. Open the **Lambda service** in AWS Console
2. Click on **"Layers"** in the left menu
3. Click **Create layer**
4. Enter the layer name: `requests-bs4-layer`
5. Upload the `my-layer.zip` file
6. Select **Compatible runtimes** as: Python 3.13
7. Click **Create**

## Step 3: Create Lambda Function

1. Click **Create function**
2. Choose **Author from scratch**
3. Enter the function name: `GetGoldRates`
4. Select **Python 3.13** as the runtime
5. Click **Create function**

## Step 4: Attach Lambda Layer

1. Scroll down to the **Layers** section of the Lambda function page
2. Click **Add a layer**
3. Choose **Custom Layers**
4. Select the `requests-bs4-layer` created earlier
5. Version: 1
6. Click **Add**

## Step 5: Configure Lambda Settings

1. Go to Configuration
2. Click **Edit**
3. Memory: **512 MB**
4. Timeout: **45 seconds** (to ensure web scraping completes)
5. Click **Save**

## Step 6: Lambda Function Code

Go to the Code section and paste the following code:

```python
import requests
from bs4 import BeautifulSoup

def lambda_handler(event, context):
    url = 'https://www.bankbazaar.com/gold-rate.html'
    headers = {"User-Agent": "Mozilla/5.0"}
    response = requests.get(url, headers=headers)
    soup = BeautifulSoup(response.content, 'html.parser')
    
    table = soup.find('table')
    hyderabad_rate_24k = None
    
    if table:
        rows = table.find_all('tr')
        for row in rows[1:]:
            cols = row.find_all('td')
            if len(cols) >= 3:
                city = cols[0].get_text(strip=True).lower()
                rate_24k = cols[2].get_text(strip=True).split('(')[0].strip()
                if city == 'hyderabad':
                    hyderabad_rate_24k = rate_24k
                    break
    
    if hyderabad_rate_24k:
        message = f"The current 24K gold rate in Hyderabad is {hyderabad_rate_24k}."
    else:
        message = "Sorry, I couldn't find the gold rate for Hyderabad."
    
    # ✅ Very important: Return in Lex V2 format
    return {
        "sessionState": {
            "dialogAction": {
                "type": "Close"
            },
            "intent": {
                "name": "GetGoldRate",
                "state": "Fulfilled"
            }
        },
        "messages": [
            {
                "contentType": "PlainText",
                "content": message
            }
        ]
    }
```

**Important:** The intent name in the code should match with the intent name created in Lex.

Deploy the code and test it:
- Create a new test event named `goldtest`
- Event JSON: `{}`
- Save and run the test
- On successful execution, you should see Status: Succeeded

## Step 7: Create Lex Bot

1. Open the **Lex service** from the AWS Console
2. Click **Create bot**
3. Enter the bot name: `GoldRateBot`
4. Runtime role: Create a role with basic Amazon Lex permissions
5. COPPA: No
6. Select English (US) as the language
7. Click **Next**, then click **Done**

### Configure Intent

1. Go back to the intents list
2. Choose **Add Intent**
3. Name: `GetGoldRate`

### Add Sample Utterances

Add the following sample utterances:
- What is the gold rate?
- Tell me Hyderabad gold price
- Gold rate today
- What is the gold price in Hyderabad?

### Configure Fulfillment

1. Scroll to the Fulfillment section
2. Turn on **Lambda function fulfillment**
3. Click **Save Intent**

### Configure Aliases

1. Go to **Aliases** section
2. Click **TestBotAlias**
3. Go to **Languages** Section → Click on **English**
4. Select the Lambda function from dropdown
5. Click **Save**

## Step 8: Build and Test

1. Go back to the Intents list and **Build** the bot
2. Finally, **Test** the bot using the test interface

## Testing the Bot

Once built, you can test the bot by typing phrases like:
- "What is the gold rate?"
- "Tell me gold price"
- "Gold rate today"

The bot will respond with the current 24K gold rate in Hyderabad.

## Notes

- The web scraping targets Bank Bazaar's gold rate page
- The function specifically looks for Hyderabad gold rates
- Timeout is set to 45 seconds to allow sufficient time for web scraping
- The response format follows Lex V2 specifications
- Ensure the intent name in Lambda code matches the Lex intent name