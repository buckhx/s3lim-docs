---
title: "Autopilot Mode"
weight: 10
---

# Autopilot Deployment

Autopilot mode is the easiest way to get started with `s3lim`. It automatically configures S3 Inventory on your source bucket and sets up a Lambda function to process the results as they are delivered.

## Use Case
Recommended for users who want a **fully automated setup** with minimal manual configuration. It creates the necessary S3 buckets, policies, and triggers for you.

## Parameters

| Parameter | Required | Default | Description |
|-----------|----------|---------|-------------|
| `SourceBucketName` | **Yes** | - | The name of the existing S3 bucket you want to analyze. |
| `InventoryDestination` | No | *Generated* | The S3 URI where inventories will be delivered (e.g., `s3://my-bucket/inventory/`). If empty, a new bucket named `<StackName>-inventory` is created. |
| `InventoryFormat` | No | `Parquet` | The format for inventory files (`CSV`, `ORC`, or `Parquet`). Parquet is recommended for performance. |
| `SourcePrefixFilter` | No | - | Filter the inventory to only include objects under this prefix. |
| `MaxPrefixDepth` | No | `10` | Maximum depth for recursive prefix aggregation. |

## Resources Created
- **S3 Bucket**: (Optional) For inventory delivery.
- **Lambda Function**: The core `s3lim` analysis engine.
- **SQS Queue**: A Dead Letter Queue (DLQ) for handling processing failures.
- **IAM Roles**: Execution roles for Lambda with scoped permissions to your buckets.
- **Custom Resource**: Automatically registers the inventory configuration with your source bucket.

## Outputs
- `InventoryDestinationURI`: The S3 location where your inventories are stored.
- `AnalyzeFunctionArn`: The ARN of the processor Lambda function.
