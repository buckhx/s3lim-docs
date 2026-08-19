---
title: "Lambda"
description: "Detailed reference for s3lim Lambda environment variables, resource sizing, and direct configuration overrides."
weight: 30
tech_metadata:
  dependencies: "AWS Lambda, Amazon S3, Amazon CloudWatch"
  proficiency: "Intermediate"
---

# Lambda

The `s3lim` Lambda function (`CoreFunction`) analyzes S3 inventory reports, maintains probabilistic sketch aggregators, and publishes metrics to Amazon CloudWatch.

> [!NOTE]
> **Deployment Template Defaults vs Direct Environment Overrides**:
> Most of these configuration settings are automatically populated by the AWS SAM deployment templates (`data-plane-template.yaml` and `data-plane-stackset.yaml`) based on template parameters (such as `InventoryDestination`, `CustomPrefixes`, `EnableMCPGateway`, etc.).
>
> However, any of these settings can be customized or overridden directly in the Lambda function's environment variables via the AWS Lambda Console, AWS CLI, or Infrastructure-as-Code pipelines without modifying or redeploying the base application binary.

## Core Analysis Settings

| Variable | Default | Description |
|----------|---------|-------------|
| `TOP_K` | `100` | The number of "heavy hitter" prefixes or objects to track per aggregator. |
| `MAX_PREFIX_DEPTH` | `10` | Maximum depth for recursive prefix aggregation (e.g., `/a/b/c/` is depth 3). |
| `MIN_PREFIX_DEPTH` | `1` | Minimum depth to start prefix aggregation. |
| `DELIMITER` | `/` | The character used to separate prefix levels. |
| `SMALL_FILE_THRESHOLD_KB` | `128` | Objects smaller than this threshold (in KB) are counted by the Small File aggregator. |
| `CUSTOM_PREFIXES` | - | Comma-separated list of explicit custom prefixes to track (e.g., `data/import/,temp/`). |

## Performance & Scaling

| Variable | Default | Description |
|----------|---------|-------------|
| `CONCURRENCY` | *Auto* | Number of concurrent workers. Defaults to the number of vCPUs available (derived from Lambda memory). |
| `BATCH_SIZE` | `1024` | Number of objects to process in a single batch before updating sketches. |
| `K_BUFFER` | `20` | Percentage of extra capacity for internal Top-K sketches to improve accuracy. |

## Metrics & Reporting

| Variable | Default | Description |
|----------|---------|-------------|
| `CW_METRICS_ENABLED` | `true` | Whether to publish custom CloudWatch metrics. |
| `CW_NAMESPACE` | `s3lim` | The CloudWatch namespace for published metrics. |
| `CW_TOP_K` | `5` | The number of top prefixes to publish as individual CloudWatch metrics. |
| `CW_STATS` | `object-count,bytes,duplicates,audit` | Comma-separated list of statistic categories to publish. |
| `OUTPUT_FORMAT` | `cloudwatch` | Primary output format (`cloudwatch`, `json`, or `console`). |

## Resource Requirements

For processing large inventory files (billions of objects), the following settings are recommended:

- **Memory**: 2048 MB (default). Probabilistic sketches ensure memory usage is bounded, but Go's GC and the Parquet/ORC parsers benefit from a larger heap.
- **Timeout**: 15 minutes (max). Large inventory files delivered in many parts may require the full execution time.
- **Architecture**: `arm64` (Graviton). Optimized for price/performance.

## IAM Permissions

The Lambda function execution role requires the following permissions:

### 1. S3 Bucket & Inventory Access
- `s3:GetObject` on `arn:aws:s3:::<InventoryBucket>/*` (Read inventory manifests and data files)
- `s3:ListBucket` on `arn:aws:s3:::<InventoryBucket>` (List manifests in destination bucket)
- `s3:PutObject`, `s3:DeleteObject`, `s3:DeleteObjectVersion` on `arn:aws:s3:::<InventoryBucket>/.s3lim-intermediate/*` (*Distributed Mode*: Intermediate state storage and cleanup)

### 2. Step Functions Workflow (*Distributed Mode*)
- `states:StartExecution`, `states:DescribeExecution`, `states:GetExecutionHistory` on `arn:aws:states:*:*:stateMachine:s3lim-*`

### 3. Telemetry & Metering
- `cloudwatch:PutMetricData` (Publish custom metrics to the configured namespace)
- `logs:CreateLogStream`, `logs:PutLogEvents`, `logs:FilterLogEvents` on `/aws/lambda/s3lim-*`
- `sqs:SendMessage`, `sqs:ReceiveMessage`, `sqs:DeleteMessage` on Dead Letter Queues
- `aws-marketplace:BatchMeterUsage`, `aws-marketplace:GetEntitlements` (AWS Marketplace billing)

## Advanced Diagnostics

| Variable | Default | Description |
|----------|---------|-------------|
| `INVENTORY_DESTINATION` | - | S3 URI where inventories are delivered. Required for scheduled scans. |
| `S3LIM_ENABLE_XRAY` | `false` | Enable AWS X-Ray tracing for deep observability. |
| `DIAGNOSTICS_DEST` | - | Optional S3 URI to upload CPU/Mem profiles for troubleshooting. |

## Custom Prefix Tracking

Custom Prefix Tracking allows you to explicitly configure specific S3 prefixes (directories) to monitor with 100% precision. Unlike standard Top-K prefix aggregation (which dynamically retains only the most active prefixes using sketching algorithms), custom prefixes are guaranteed to never be evicted and will be audited with full precision across all active prefix-based aggregators.

To configure custom prefixes, pass them as a comma-separated list:
- **SAM Template**: Specify the `CustomPrefixes` template parameter during deployment (e.g., `CustomPrefixes="data/import/,temp/"`).
- **Lambda Environment**: Set the `CUSTOM_PREFIXES` environment variable directly in Lambda (e.g., `CUSTOM_PREFIXES="data/import/,temp/"`).
