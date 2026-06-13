---
title: "Configuration"
description: "Detailed reference for s3lim Lambda environment variables and tuning parameters."
weight: 30
tech_metadata:
  dependencies: "AWS Lambda, Amazon S3, Amazon CloudWatch"
  proficiency: "Intermediate"
---

# Lambda Configuration

The `s3lim` Lambda function is configured primarily through environment variables. These settings allow you to tune the analysis depth, performance, and reporting behavior.

## Core Analysis Settings

| Variable | Default | Description |
|----------|---------|-------------|
| `TOP_K` | `100` | The number of "heavy hitter" prefixes or objects to track per aggregator. |
| `MAX_PREFIX_DEPTH` | `10` | Maximum depth for recursive prefix aggregation (e.g., `/a/b/c/` is depth 3). |
| `MIN_PREFIX_DEPTH` | `1` | Minimum depth to start prefix aggregation. |
| `DELIMITER` | `/` | The character used to separate prefix levels. |
| `SMALL_FILE_THRESHOLD_KB` | `128` | Objects smaller than this threshold (in KB) are counted by the Small File aggregator. |

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

The Lambda function requires the following minimum permissions:

### 1. S3 Read Access
Required to read the inventory manifest and the data files (CSV, Parquet, or ORC).
- `s3:GetObject` on `arn:aws:s3:::<InventoryBucket>/<Prefix>/*`
- `s3:ListBucket` on `arn:aws:s3:::<InventoryBucket>`

### 2. CloudWatch Metrics
Required to publish the analyzed metrics.
- `cloudwatch:PutMetricData`

## Advanced Diagnostics

| Variable | Default | Description |
|----------|---------|-------------|
| `INVENTORY_DESTINATION` | - | S3 URI where inventories are delivered. Required for scheduled scans. |
| `S3LIM_ENABLE_XRAY` | `false` | Enable AWS X-Ray tracing for deep observability. |
| `DIAGNOSTICS_DEST` | - | Optional S3 URI to upload CPU/Mem profiles for troubleshooting. |
