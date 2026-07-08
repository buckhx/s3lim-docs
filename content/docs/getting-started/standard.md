---
title: "Standard Mode"
weight: 2
---

# Standard Mode Walkthrough

This guide is for users who **already have S3 Inventory reports** being configured and delivered to a destination bucket, and want `s3lim` to analyze them using automated, scoped IAM role generation.

## Use Case
Ideal for users who want to analyze pre-existing inventory pipelines with a simple, managed deployment. The SAM template automatically creates the analysis Lambda function and scope-limited IAM execution roles so you don't have to configure policies manually.

## Prerequisites
* An AWS account with permissions to deploy Lambda, IAM Roles, and CloudFormation.
* A pre-existing S3 Inventory report pipeline delivering reports to a destination S3 bucket.
* The S3 URI where your existing inventory manifests are delivered (e.g., `s3://my-inventory-bucket/inventory/`).

## Deployment Steps

1. **Get the Destination Path**: Locate the S3 bucket and prefix path where your source bucket delivers its inventory manifests.
2. **Launch Standard Stack**: Navigate to the [s3lim-standard](https://serverlessrepo.aws.amazon.com/applications) application on the AWS Serverless Application Repository.
3. **Configure Stack Parameters**:
   * Enter `InventoryDestination` (e.g., `s3://my-inventory-bucket/inventory/`).
   * Enter `SourceBucketName` (optional, required if you want to run previews using S3 ListObjectsV2 API).
   * For other options, see the [Deployment Specifications]({{< relref "docs/reference/deployment.md#standard-mode-spec" >}}).
4. **Deploy**: Click **Deploy** to start provisioning.

## Configure Analysis Trigger
Because Standard Mode operates on pre-existing inventories, by default it sets up a **daily scheduled trigger (cron)** to scan the destination bucket for new manifests.

If you want real-time analysis immediately upon report delivery:
1. Copy the `AnalyzeFunctionArn` from the CloudFormation stack outputs.
2. Navigate to your S3 Inventory destination bucket in the AWS Console.
3. Go to the **Properties** tab and scroll to **Event notifications**.
4. Click **Create event notification** and configure:
   * **Prefix**: The folder path of your inventory reports (e.g., `inventory/`).
   * **Suffix**: `manifest.json`.
   * **Event types**: `All object create events` (`s3:ObjectCreated:*`).
   * **Destination**: Lambda Function (select your `s3lim` function).

## Verification
Once a new S3 Inventory report is delivered, the Lambda will automatically execute. You can view the output analysis and metrics in your AWS CloudWatch dashboard.
