# Step-by-Step Project Instructions

Follow this guide to build the near real-time stock market data analytics pipeline on AWS.

---

## 1. Setting Up Data Streaming with Amazon Kinesis

### Step 1.1: Create Kinesis Data Stream
1.  Navigate to the **Amazon Kinesis** console in AWS.
2.  Click **Create data stream**.
3.  **Stream name**: `stock-market-stream`
4.  **Data Stream Capacity Mode**: Select **On-demand**.
5.  Click **Create data stream**.

### Step 1.2: Set Up Local Python Environment
1.  Ensure you have Python installed.
2.  Install the necessary libraries:
    ```bash
    pip install boto3 yfinance
    ```
3.  Configure your AWS credentials if you haven't already:
    ```bash
    aws configure
    ```

### Step 1.3: Create the Python Streaming Script
Create a new file named `stream_stock_data.py` and paste the following code.

```python
import boto3
import json
import time
import yfinance as yf

# AWS Kinesis Configuration
# IMPORTANT: Replace us-east-1 with your desired region
kinesis_client = boto3.client('kinesis', region_name='us-east-1') 

# IMPORTANT: Replace with your actual stream name
STREAM_NAME = "stock-market-stream"
STOCK_SYMBOL = "AAPL"
DELAY_TIME = 30  # Time delay in seconds

# Function to fetch stock data
def get_stock_data(symbol):
    try:
        stock = yf.Ticker(symbol)
        data = stock.history(period="2d")

        if len(data) < 2:
            raise ValueError("Insufficient data to fetch previous close.")

        stock_data = {
            "symbol": symbol,
            "open": round(data.iloc[-1]["Open"], 2),
            "high": round(data.iloc[-1]["High"], 2),
            "low": round(data.iloc[-1]["Low"], 2),
            "price": round(data.iloc[-1]["Close"], 2),
            "previous_close": round(data.iloc[-2]["Close"], 2),
            "change": round(data.iloc[-1]["Close"] - data.iloc[-2]["Close"], 2),
            "change_percent": round(((data.iloc[-1]["Close"] - data.iloc[-2]["Close"]) / data.iloc[-2]["Close"]) * 100, 2),
            "volume": int(data.iloc[-1]["Volume"]),
            "timestamp": time.strftime("%Y-%m-%dT%H:%M:%SZ", time.gmtime())
        }
        return stock_data
    except Exception as e:
        print(f"Error fetching stock data for {symbol}: {e}")
        return None

# Function to stream data into Kinesis
def send_to_kinesis():
    while True:
        try:
            stock_data = get_stock_data(STOCK_SYMBOL)
            if stock_data is None:
                print("Skipping this iteration due to API error.")
                time.sleep(DELAY_TIME)
                continue

            print(f"Sending: {stock_data}")

            # Send to Kinesis
            response = kinesis_client.put_record(
                StreamName=STREAM_NAME,
                Data=json.dumps(stock_data),
                PartitionKey=STOCK_SYMBOL
            )
            print(f"Kinesis Response: {response}")

            time.sleep(DELAY_TIME)

        except Exception as e:
            print(f"An error occurred: {e}")
            time.sleep(DELAY_TIME)

# Run the streaming function
if __name__ == "__main__":
    send_to_kinesis()
