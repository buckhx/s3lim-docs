---
title: "Deployment Specifications"
description: "Exhaustive configuration settings, template parameters, IAM permissions, and outputs for all s3lim deployment models."
weight: 10
---

# Deployment Specifications

`s3lim` is deployed using AWS Serverless Application Model (SAM) templates. Depending on your environment, you will use one of three main template configurations: **Autopilot**, **Standard**, or **Read-Only** (used in Self-Serve).

---

## Autopilot Mode Spec
Template: `sam-template-autopilot.yaml`

Automatically configures the S3 Inventory report on your source bucket and provisions the analysis engine.

### Parameters
| Name | Type | Default | Description |
| :--- | :--- | :--- | :--- |
| `SourceBucketName` | String | **Required** | The name of the S3 bucket to analyze. |
| `InventoryDestination` | String | *(Generated)* | The S3 URI where S3 Inventory reports are delivered. Defaults to a new bucket named `<StackName>-inventory`. |
| `InventoryFormat` | String | `Parquet` | Format of inventory files (`CSV`, `ORC`, or `Parquet`). Parquet is recommended for speed. |
| `SourcePrefixFilter` | String | - | Optional: Filter inventory to only analyze files under this prefix. |
| `MaxPrefixDepth` | Number | `10` | Maximum depth for recursive prefix aggregation. |
| `CustomPrefixes` | String | - | Optional: Comma-separated list of explicit custom prefixes to track (e.g. `data/import/,temp/`). |

### Resources Created
* **S3 Bucket**: For inventory reports (if not using pre-existing bucket).
* **Lambda Function**: Core `s3lim` analysis processor.
* **SQS Queue**: Dead Letter Queue (DLQ) for analysis failure handling.
* **IAM Roles**: Execution roles with scoped read/write permissions for source and inventory buckets.
* **Custom Resource**: Script that auto-registers inventory configuration on your source bucket.

### Outputs
* `InventoryDestinationURI`: Location where S3 Inventory reports are stored.
* `AnalyzeFunctionArn`: ARN of the processor Lambda function.

---

## Standard Mode Spec
Template: `sam-template-standard.yaml`

Deploys `s3lim` to analyze existing S3 Inventory reports using automatically generated, scoped IAM permissions.

### Parameters
| Name | Type | Default | Description |
| :--- | :--- | :--- | :--- |
| `InventoryDestination` | String | **Required** | The S3 URI where existing inventories are delivered (e.g., `s3://my-bucket/inventory/`). |
| `MaxPrefixDepth` | Number | `10` | Maximum depth for recursive prefix aggregation. |
| `SourceBucketName` | String | - | Optional: The name of the source S3 bucket. Required if you want to run previews via `ListObjectsV2`. |
| `CustomPrefixes` | String | - | Optional: Comma-separated list of explicit custom prefixes to track (e.g. `data/import/,temp/`). |

### Resources Created
* **Lambda Function**: Core `s3lim` analysis processor.
* **SQS Queue**: Dead Letter Queue (DLQ) for analysis failure handling.
* **IAM Roles**: Scoped execution role with read permissions to your existing inventory destination bucket and CloudWatch write permissions.

### Outputs
* `AnalyzeFunctionArn`: ARN of the processor Lambda function.

---

## Read-Only Mode Spec (Self-Serve)
Template: `sam-template-readonly.yaml`

For strict enterprise environments. Deploys only the analysis Lambda and triggers, requiring you to attach a pre-created (BYO-IAM) execution role.

### Parameters
| Name | Type | Default | Description |
| :--- | :--- | :--- | :--- |
| `InventoryDestination` | String | **Required** | The S3 URI where existing inventories are delivered. |
| `LambdaRoleArn` | String | **Required** | The ARN of your pre-created, manually audited IAM execution role. |
| `MaxPrefixDepth` | Number | `10` | Maximum depth for recursive prefix aggregation. |
| `CustomPrefixes` | String | - | Optional: Comma-separated list of explicit custom prefixes to track. |

### IAM Requirements
Your pre-created IAM role must possess the following minimum permissions:

#### S3 Policy
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": ["s3:GetObject", "s3:ListBucket"],
            "Resource": [
                "arn:aws:s3:::your-inventory-bucket",
                "arn:aws:s3:::your-inventory-bucket/*"
            ]
        }
    ]
}
```

#### CloudWatch & SQS Policy
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "cloudwatch:PutMetricData",
                "logs:CreateLogStream",
                "logs:PutLogEvents"
            ],
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "sqs:SendMessage",
                "sqs:ReceiveMessage",
                "sqs:DeleteMessage",
                "sqs:GetQueueAttributes"
            ],
            "Resource": "arn:aws:sqs:*:*:s3lim-*"
        }
    ]
}
```

### Outputs
* `AnalyzeFunctionArn`: ARN of the processor Lambda function.

---

## Cross-Region Considerations

When deploying `s3lim` in Standard or Self-Serve modes, you may encounter cross-region architectures (e.g., the Lambda is deployed in `us-east-1` but analyzes S3 buckets in `us-west-2`). Keep the following in mind:

* **Data Transfer Costs**: `s3lim` operates in-place, but if the Lambda function and the S3 Inventory destination bucket reside in different regions, downloading the inventory reports will incur standard **AWS Data Transfer Out (egress)** costs. For large buckets with gigabytes of inventory reports, this can result in substantial data transfer charges.
* **Execution Timeout**: Cross-region S3 downloads suffer from higher latency. If you are analyzing a massive bucket with billions of keys, the extra latency from cross-region transfers may cause the Lambda execution to exceed its 15-minute (900-second) timeout limit.
* **KMS Encryption**: If your S3 Inventory destination bucket is encrypted using a Customer Managed Key (CMK) via KMS, the Lambda execution role must be granted cross-region decrypt permissions (`kms:Decrypt`) on the KMS key.
* **CloudWatch Namespace**: `s3lim` publishes metrics to the region in which the Lambda is running. If a Lambda in `us-east-1` analyzes a bucket in `us-west-2`, the custom metrics will appear in the `us-east-1` CloudWatch dashboard and namespace.

> [!IMPORTANT]
> **Best Practice**: Always deploy the `s3lim` stack in the **same region** as the S3 Inventory destination bucket to eliminate data transfer fees, minimize latency, and prevent execution timeouts.

---

## Multi-Bucket Analysis

If you want to use a single `s3lim` Lambda instance to process inventory reports from multiple S3 buckets:

### Using the Multi-Bucket Template (`sam-template-multibucket.yaml`)
For Standard Mode deployments, use the multi-bucket template. It supports:
* `InventoryDestination`: The primary S3 URI for your main inventory destination.
* `AdditionalInventoryBuckets`: A list of additional S3 inventory buckets (e.g., `bucket2, bucket3`) to grant read access to.

### Using Self-Serve Mode (BYO-IAM)
If you deploy in Self-Serve Mode and manage your own IAM execution role, you must explicitly expand the Lambda's S3 policy permissions to grant access to all inventory buckets:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": ["s3:GetObject", "s3:ListBucket"],
            "Resource": [
                "arn:aws:s3:::primary-inventory-bucket",
                "arn:aws:s3:::primary-inventory-bucket/*",
                "arn:aws:s3:::secondary-inventory-bucket",
                "arn:aws:s3:::secondary-inventory-bucket/*"
            ]
        }
    ]
}
```
