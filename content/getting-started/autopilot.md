---
title: "Quickstart (Autopilot)"
weight: 1
---

# Quickstart (Autopilot)

This guide is for users who want to get up and running with `s3lim` as quickly as possible using the **Autopilot** deployment mode.

## Prerequisites
- An AWS account with permissions to deploy Lambda and CloudFormation.
- One or more S3 buckets you wish to analyze.

## Deployment Steps
1. Navigate to the [s3lim-autopilot](https://serverlessrepo.aws.amazon.com/applications) on the AWS Serverless Application Repository.
2. Enter your `SourceBucketName`.
3. Click **Deploy**.

## What Happens Next?
`s3lim` will automatically:
1. Configure an S3 Inventory report on your source bucket.
2. Create a destination bucket for inventory delivery.
3. Set up an S3 Event Notification to trigger analysis when new reports arrive.

**Note:** It can take up to 48 hours for AWS to deliver the first S3 Inventory report. Your dashboard will remain empty until then.
