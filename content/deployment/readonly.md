---
title: "Read-Only Mode"
weight: 20
---

# Read-Only Deployment

Read-Only mode is designed for environments where you already have S3 Inventory reports being delivered and want to analyze them without granting `s3lim` permission to modify your source bucket configurations.

## Use Case
Ideal for **Enterprise environments** with strict IAM controls or existing inventory pipelines. It requires an existing IAM role and does not create any new roles or bucket configurations.

## Parameters

| Parameter | Required | Default | Description |
|-----------|----------|---------|-------------|
| `InventoryDestination` | **Yes** | - | The S3 URI where existing inventories are delivered (e.g., `s3://my-bucket/inventory/`). |
| `LambdaRoleArn` | **Yes** | - | ARN of an existing IAM Role for the Lambda function. See [IAM Requirements](#iam-requirements) below. |
| `MaxPrefixDepth` | No | `10` | Maximum depth for recursive prefix aggregation. |

## IAM Requirements
The provided `LambdaRoleArn` must have the following minimum permissions:
1.  `s3:GetObject` and `s3:ListBucket` on the `InventoryDestination` bucket.
2.  `cloudwatch:PutMetricData` (Namespace: `s3lim`).
3.  `logs:CreateLogStream` and `logs:PutLogEvents` for the Lambda's log group.
4.  `sqs:SendMessage`, `sqs:ReceiveMessage`, and `sqs:DeleteMessage` on the stack's DLQ.

## Operational Note
By default, this mode uses a **daily scheduled trigger** (cron) to scan the destination bucket for new manifests. For real-time analysis, we recommend manually adding an S3 Object Created trigger to the `InventoryDestination` bucket.

## Outputs
- `AnalyzeFunctionArn`: The ARN of the processor Lambda function.
