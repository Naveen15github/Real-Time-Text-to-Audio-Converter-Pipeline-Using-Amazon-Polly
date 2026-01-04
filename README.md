# Real-Time Text-to-Audio Converter Pipeline Using Amazon Polly

## 📌 Project Summary

I designed and implemented a **serverless, event-driven text-to-speech pipeline** on AWS that converts text files into high-quality audio using **Amazon Polly**.
The system automatically processes uploaded text files in real time and stores the generated audio output in Amazon S3 with no manual intervention.

This project demonstrates my practical experience with **AWS serverless architecture, IAM security, S3 event triggers, and Amazon Polly integration**.

---

## 🎯 Problem Statement

Reading long blogs, articles, or documents is not always convenient, and written content is not accessible to all users.
I built this solution to **automatically transform written content into audio**, improving accessibility, usability, and content reach.

---

## 🧩 Solution Overview

Whenever I upload a `.txt` file to an S3 source bucket, the system:

1. Automatically triggers an AWS Lambda function
2. Reads the text content from S3
3. Converts the text into natural-sounding speech using Amazon Polly
4. Stores the generated `.mp3` file in a destination S3 bucket

The entire pipeline runs **serverlessly**, scales automatically, and requires no infrastructure management.

---

## 🏗️ Architecture Implemented

**End-to-End Flow:**

* Amazon S3 (Source) → AWS Lambda → Amazon Polly → Amazon S3 (Destination)

**AWS Services Used:**

* Amazon S3 – Object storage for input and output
* AWS Lambda – Serverless compute
* Amazon Polly – Text-to-Speech conversion
* AWS IAM – Secure access management
* Amazon CloudWatch – Logging and monitoring

---

## 🛠️ Implementation Details

### Step 1: AWS Account & Region Setup

I configured my AWS account and selected an appropriate region to deploy all services consistently.

---

### Step 2: S3 Bucket Creation

I created two S3 buckets with distinct responsibilities:

| Bucket                         | Purpose                       |
| ------------------------------ | ----------------------------- |
| `amc-polly-source-bucket`      | Stores uploaded `.txt` files  |
| `amc-polly-destination-bucket` | Stores generated `.mp3` files |

This separation ensures clear data flow and access control.

---

### Step 3: IAM Policy Design

I created a **custom IAM policy** that follows the **principle of least privilege**, allowing only the required permissions for the Lambda function.

**Policy Name:** `amc-polly-lambda-policy`

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:GetObject", "s3:PutObject"],
      "Resource": [
        "arn:aws:s3:::amc-polly-source-bucket/*",
        "arn:aws:s3:::amc-polly-destination-bucket/*"
      ]
    },
    {
      "Effect": "Allow",
      "Action": "polly:SynthesizeSpeech",
      "Resource": "*"
    }
  ]
}
```

---

### Step 4: IAM Role Configuration

I created an IAM role named **`amc-polly-lambda-role`** and attached:

* My custom Polly + S3 policy
* AWS managed policy `AWSLambdaBasicExecutionRole`

This role enables secure Lambda execution and CloudWatch logging.

---

### Step 5: Lambda Function Setup

I implemented the Lambda function with the following configuration:

* **Function Name:** `TextToSpeechFunction`
* **Runtime:** Python 3.8
* **Timeout:** 30 seconds
* **Execution Role:** `amc-polly-lambda-role`

#### Environment Variables:

| Key                  | Value                        |
| -------------------- | ---------------------------- |
| `SOURCE_BUCKET`      | amc-polly-source-bucket      |
| `DESTINATION_BUCKET` | amc-polly-destination-bucket |

---

### Step 6: S3 Event Trigger Configuration

I configured the source S3 bucket to automatically trigger the Lambda function when:

* A new object is created
* File type ends with `.txt`

This enables **real-time, event-driven execution** without manual invocation.

---

### Step 7: Lambda Function Implementation

I wrote the Lambda function in Python to:

* Read the uploaded text file from S3
* Invoke Amazon Polly using the Neural engine
* Generate an `.mp3` audio file
* Store the output in the destination S3 bucket

```python
import json
import boto3
import os

s3 = boto3.client('s3')
polly = boto3.client('polly')

SOURCE_BUCKET = os.environ['SOURCE_BUCKET']
DESTINATION_BUCKET = os.environ['DESTINATION_BUCKET']

def lambda_handler(event, context):
    record = event['Records'][0]
    bucket_name = record['s3']['bucket']['name']
    object_key = record['s3']['object']['key']

    response = s3.get_object(Bucket=bucket_name, Key=object_key)
    text = response['Body'].read().decode('utf-8')

    polly_response = polly.synthesize_speech(
        Text=text,
        OutputFormat='mp3',
        VoiceId='Joanna',
        Engine='neural'
    )

    audio_key = object_key.replace('.txt', '.mp3')

    s3.put_object(
        Bucket=DESTINATION_BUCKET,
        Key=audio_key,
        Body=polly_response['AudioStream'].read()
    )

    return {
        'statusCode': 200,
        'message': 'Audio file successfully generated'
    }
```

---

### Step 8: Testing & Validation

I validated the pipeline by:

1. Uploading a `.txt` file to the source bucket
2. Verifying Lambda execution in CloudWatch logs
3. Confirming `.mp3` output in the destination bucket
4. Playing the generated audio to ensure correctness

---

## 🧪 Sample Input

```
Hello, this is Naveen.
I built this real-time text-to-audio pipeline using Amazon Polly.
```

---

## 🔐 Security Considerations

* Implemented least-privilege IAM access
* Isolated S3 buckets for input and output
* Enabled CloudWatch logging for observability

---

## 📈 Key Outcomes & Learnings

* Designed a complete serverless workflow on AWS
* Implemented event-driven automation using S3 triggers
* Integrated Amazon Polly with Lambda for real-time processing
* Applied IAM security best practices

---

## 📌 Possible Enhancements

* Add API Gateway for direct text input
* Support multiple voices and languages
* Store metadata in DynamoDB
* Add error handling and retries


