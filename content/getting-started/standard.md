---
title: "Standard Mode"
weight: 15
---

# Standard Deployment

Standard mode is designed for users who **already have S3 Inventory reports** being delivered to a bucket and want `s3lim` to analyze them with managed IAM permissions.

## Use Case
Ideal for users who have existing inventory pipelines and want a simple deployment that **automatically handles IAM roles and policies** for the analysis Lambda.

## Parameters

| Parameter | Required | Default | Description |
|-----------|----------|---------|-------------|
| `InventoryDestination` | **Yes** | - | The S3 URI where your existing inventories are delivered (e.g., `s3://my-bucket/inventory/`). |
| `MaxPrefixDepth` | No | `10` | Maximum depth for recursive prefix aggregation. |

## Resources Created
- **Lambda Function**: The core `s3lim` analysis engine.
- **SQS Queue**: A Dead Letter Queue (DLQ) for handling processing failures.
- **IAM Roles**: Execution roles for Lambda with automatically generated policies to read your inventory bucket and publish metrics.

## Operational Note
By default, this mode includes a **daily scheduled trigger** (cron) to scan the destination bucket for new manifests. You can also manually add an S3 Event Notification to the destination bucket to trigger the Lambda in real-time.

## Outputs
- `AnalyzeFunctionArn`: The ARN of the processor Lambda function.
