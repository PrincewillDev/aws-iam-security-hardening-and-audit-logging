# CloudTrail Reference Guide (CLI)

## Purpose

This document serves as my personal reference for configuring AWS CloudTrail using the AWS CLI. It documents the commands, concepts, workflow, and troubleshooting steps I used during my AWS IAM Security Hardening project.

---

# What is AWS CloudTrail?

CloudTrail is AWS's auditing service.

Its purpose is to record activities performed in an AWS account.

CloudTrail answers questions like:

* Who performed an action?
* What action was performed?
* When did it happen?
* From where?
* Was it successful or denied?

CloudTrail is mainly used for:

* Security auditing
* Compliance
* Incident investigation
* Troubleshooting
* Change tracking

---

# Management Events vs Data Events

Understanding this difference is the most important CloudTrail concept.

## Management Events

These record actions performed **on AWS resources**.

Examples:

* Create IAM User
* Delete IAM User
* Attach IAM Policy
* Create S3 Bucket
* Update CloudTrail
* Create EC2 Instance

Example CloudTrail event:

```json
"eventName": "CreateUser",
"eventCategory": "Management"
```

Management events are enabled by default.

---

## Data Events

These record actions performed **inside AWS resources**.

For S3 they include:

* PutObject
* GetObject
* DeleteObject

Example:

```json
"eventName": "PutObject",
"eventCategory": "Data"
```

Data Events are **NOT enabled by default** because they can generate a very large number of logs.

---

# Architecture

```
IAM User
      │
      │ performs action
      ▼
CloudTrail
      │
      │ records event
      ▼
S3 Log Bucket
      │
      ▼
JSON Log Files
```

CloudTrail does not store logs permanently.

Instead, it writes log files into an S3 bucket.

---

# Buckets Used

## Application Bucket

```
princewill-storage
```

Purpose:

* Used for testing the least-privilege IAM user.

---

## CloudTrail Log Bucket

```
princewill-cloudtrail-logs
```

Purpose:

* Stores CloudTrail audit logs.

---

# Why CloudTrail Needs a Bucket Policy

Creating an S3 bucket does **NOT** automatically allow CloudTrail to use it.

CloudTrail needs permission to:

1. Verify the bucket.

```
s3:GetBucketAcl
```

2. Upload log files.

```
s3:PutObject
```

This permission is granted through an S3 Bucket Policy.

---

# Creating the Trail

Create the trail:

```bash
aws cloudtrail create-trail \
    --name security-audit-trail \
    --s3-bucket-name princewill-cloudtrail-logs
```

Start logging:

```bash
aws cloudtrail start-logging \
    --name security-audit-trail
```

Verify:

```bash
aws cloudtrail get-trail-status \
    --name security-audit-trail
```

Expected:

```text
IsLogging: true
```

---

# Why Enable Data Events?

CloudTrail only records Management Events by default.

Since I wanted to audit S3 object operations such as:

* Upload
* Download
* Delete

I had to enable Data Events.

---

# Advanced Event Selector

CloudTrail accepts a configuration describing exactly what should be monitored.

I created:

```
configs/advanced-event-selectors.json
```

Contents:

```json
[
  {
    "Name": "S3DataEvents",
    "FieldSelectors": [
      {
        "Field": "eventCategory",
        "Equals": ["Data"]
      },
      {
        "Field": "resources.type",
        "Equals": ["AWS::S3::Object"]
      },
      {
        "Field": "resources.ARN",
        "StartsWith": [
          "arn:aws:s3:::princewill-storage/"
        ]
      }
    ]
  }
]
```

---

# Why This JSON Exists

The JSON file is **not uploaded to S3**.

It simply tells CloudTrail:

> Monitor Data Events occurring on S3 objects inside the `princewill-storage` bucket.

The AWS CLI reads the file and sends its contents to CloudTrail.

---

# Applying the Event Selector

```bash
aws cloudtrail put-event-selectors \
    --trail-name security-audit-trail \
    --advanced-event-selectors file://configs/advanced-event-selectors.json
```

Verify:

```bash
aws cloudtrail get-event-selectors \
    --trail-name security-audit-trail
```

---

# Testing the Trail

## Successful Operation

Using the restricted IAM user:

```bash
aws s3 cp test-upload.txt \
s3://princewill-storage/test-upload.txt \
--profile s3-writer
```

Result:

```
Upload succeeded.
```

CloudTrail recorded:

```
PutObject
```

Important fields:

```json
"eventCategory":"Data"
"managementEvent":false
```

This confirmed that Data Event logging was working.

---

## Denied Operation

Using the same restricted user:

```bash
aws s3api delete-object \
--bucket princewill-storage \
--key test-upload.txt \
--profile s3-writer
```

Result:

```
AccessDenied
```

Reason:

The IAM policy only allowed:

```
s3:PutObject
```

It did not allow:

```
s3:DeleteObject
```

CloudTrail recorded the denied request.

---

# Another Denied Operation

```bash
aws s3 ls --profile s3-writer
```

CloudTrail recorded:

```
ListBuckets
```

with

```
AccessDenied
```

This was a Management Event because listing buckets is an account-level operation.

---

# Reading CloudTrail Logs

CloudTrail stores log files inside:

```
AWSLogs/
    <ACCOUNT_ID>/
        CloudTrail/
            us-east-1/
                YEAR/
                    MONTH/
                        DAY/
```

List logs:

```bash
aws s3 ls s3://princewill-cloudtrail-logs/AWSLogs/<ACCOUNT_ID>/CloudTrail/us-east-1/ --recursive
```

Download:

```bash
aws s3 cp s3://princewill-cloudtrail-logs/<PATH_TO_FILE>.json.gz .
```

Extract the file.

Search for:

```
PutObject
```

or

```
DeleteObject
```

or

```
AccessDenied
```

---

# Common Errors

## InsufficientS3BucketPolicyException

Meaning:

CloudTrail could not write into the log bucket.

Solution:

Attach the required S3 bucket policy.

---

## Invalid JSON

Meaning:

The CLI could not parse the JSON file.

Possible causes:

* Missing comma
* Missing quotes
* Incorrect brackets
* Wrong JSON structure
* Missing `file://` prefix

---

## AccessDenied

Meaning:

The IAM user attempted an action that was not allowed.

Usually expected when testing least privilege.

---

# Commands Summary

## Create trail

```bash
aws cloudtrail create-trail
```

## Start logging

```bash
aws cloudtrail start-logging
```

## Check logging

```bash
aws cloudtrail get-trail-status
```

## Configure Data Events

```bash
aws cloudtrail put-event-selectors
```

## View selector

```bash
aws cloudtrail get-event-selectors
```

## List CloudTrail logs

```bash
aws s3 ls
```

## Download logs

```bash
aws s3 cp
```

---

# Lessons Learned

* CloudTrail records AWS activity for auditing.
* Management Events and Data Events are different.
* Data Events must be enabled manually.
* CloudTrail needs an S3 bucket and a bucket policy to store logs.
* Event selectors determine which activities CloudTrail records.
* S3 object operations such as `PutObject` and `DeleteObject` are Data Events.
* The `s3-writer-user` successfully demonstrated least privilege by allowing uploads while denying unauthorized actions.
* CloudTrail log files provide the evidence needed for auditing and security investigations.

---

# Task 3 Workflow

```
Create Log Bucket
        │
        ▼
Attach Bucket Policy
        │
        ▼
Create CloudTrail
        │
        ▼
Start Logging
        │
        ▼
Enable S3 Data Events
        │
        ▼
Verify Event Selector
        │
        ▼
Perform Allowed Operation (PutObject)
        │
        ▼
Perform Denied Operation (DeleteObject/ListBuckets)
        │
        ▼
Locate CloudTrail Log
        │
        ▼
Confirm Audit Record
```
