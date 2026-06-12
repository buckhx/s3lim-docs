---
title: "Lambda Configuration"
weight: 30
---

# Lambda Configuration

The `s3lim` Lambda function is designed for high-performance processing of massive S3 inventory files. This guide details the configuration options and requirements for running `s3lim` in a serverless environment.

## Resource Requirements

For processing large inventory files (billions of objects), the following settings are recommended:

- **Memory**: 2048 MB (default). Probabilistic sketches ensure memory usage is bounded, but Go's GC and the Parquet/ORC parsers benefit from a larger heap.
- **Timeout**: 15 minutes (max). Large inventory files delivered in many parts may require the full execution time.
- **Architecture**: `arm64` (Graviton). Optimized for price/performance.

## Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `MAX_PREFIX_DEPTH` | `10` | Maximum depth for recursive prefix aggregation. |
| `INVENTORY_DESTINATION` | - | The S3 URI where inventories are delivered (e.g., `s3://bucket/prefix/`). Used for diagnostics. |
| `S3LIM_ENABLE_XRAY` | `false` | Enable AWS X-Ray tracing for S3 clients and aggregators. |
| `DIAGNOSTICS_DEST` | - | Optional S3 URI to upload CPU/Mem profiles (e.g., `s3://bucket/diag/`). |

## IAM Permissions

The Lambda function requires the following minimum permissions:

### 1. S3 Read Access
Required to read the inventory manifest and the data files (CSV, Parquet, or ORC).
- `s3:GetObject` on `arn:aws:s3:::<InventoryBucket>/<Prefix>/*`
- `s3:ListBucket` on `arn:aws:s3:::<InventoryBucket>`

### 2. CloudWatch Metrics
Required to publish the analyzed metrics.
- `cloudwatch:PutMetricData`

### 3. SQS DLQ (Optional)
Required if using a Dead Letter Queue for reprocessing.
- `sqs:SendMessage` on the DLQ ARN.
- `sqs:ReceiveMessage` and `sqs:DeleteMessage` (if the Redrive trigger is enabled).

## Error Handling & DLQ

`s3lim` is configured with an SQS Dead Letter Queue (DLQ). If an inventory file fails to process (e.g., due to a timeout or transient S3 error), the S3 event message is moved to the DLQ.

### Reprocessing (Redrive)
The `AnalyzeFunction` in the SAM template includes an SQS event source mapping for the DLQ that is **disabled** by default. To re-process failed inventories:

1.  Log in to the AWS Console.
2.  Navigate to the `s3lim` Lambda function.
3.  Go to the **Configuration** -> **Triggers** tab.
4.  Find the SQS trigger for the DLQ and click **Enable**.
5.  Once the queue is drained, **Disable** the trigger to prevent infinite loops if the error persists.

## Networking

`s3lim` does not require access to the public internet or a VPC unless your S3 buckets are configured with VPC Endpoints that restrict access to specific VPCs. By default, running outside a VPC is recommended for lower latency and cost.
