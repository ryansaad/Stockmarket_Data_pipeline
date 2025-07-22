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
get the code from the same file named in this repository
### Step 1.4: Run the Script and Verify
- Run the script from your terminal:
- Bash
- python stream_stock_data.py
- Go to the Kinesis console, select your stream, and check the Monitoring tab to see incoming records.

Important: Stop the script with CTRL+C in your terminal after a few minutes to avoid unnecessary costs.


## 2. Processing Data with AWS Lambda
### Step 2.1: Create DynamoDB Table
- Navigate to the DynamoDB console.
- Click Create table.
- Table name: stock-market-data
- Partition key: symbol (Type: String)
- Sort key: timestamp (Type: String)
- Click Create table.

### Step 2.2: Create S3 Bucket
- Navigate to the S3 console.
- Click Create bucket.
- Bucket name: Enter a globally unique name (e.g., stock-market-raw-data-yourinitials-date).
- Leave other settings as default and click Create bucket.

### Step 2.3: Create IAM Role for Lambda
- Go to the IAM console -> Roles -> Create role.
- Trusted entity type: AWS service.
- Use case: Lambda.
= Attach the following managed policies:
1 .AmazonKinesisFullAccess
2. AmazonDynamoDBFullAccess
3. AmazonS3FullAccess
4. AWSLambdaBasicExecutionRole.

-Name the role Lambda_Kinesis_Processing_Role and create it.

### Step 2.4: Create and Configure the Processing Lambda
- Go to the Lambda console -> Create function.
- Function Name: ProcessStockData
- Runtime: Python 3.12 (or newer)
- Execution Role: Choose Use an existing role and select Lambda_Kinesis_Processing_Role.
- Click Create function.
- In the function overview, click Add trigger. Select Kinesis, choose your stock-market-stream, set the Batch size to 2, and
- click Add.

Paste the following code into the Lambda code editor and click Deploy.
Get code from the lambda_file in this repository



### Step 2.5: Test the Integration
- Run your local stream_stock_data.py script for about 5-10 minutes.
- Check your S3 bucket to see folders of raw JSON files.
- Go to your DynamoDB table and click Explore table items to see the processed records.

Remember to stop the local script.


