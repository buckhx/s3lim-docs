---
title: "Deployment Specifications"
description: "Exhaustive configuration settings, template parameters, IAM permissions, and outputs for all s3lim deployment models."
weight: 10
---

# Deployment Specifications

`s3lim` is deployed using AWS Serverless Application Model (SAM) templates. All data plane capabilities are consolidated into a single unified template: **`data-plane-template.yaml`** (with multi-region orchestration supported via **`data-plane-stackset.yaml`**).

---

## Unified Data Plane Spec
Template: `data-plane-template.yaml`

A single template supporting all deployment modes via parameters:
* **Automated Setup Mode**: Set `SourceBucketName` to automatically configure S3 Inventory reporting on your source bucket(s) and auto-generate an inventory destination bucket if needed.
* **Existing Inventory Mode**: Set `InventoryDestination` to point to an existing S3 Inventory destination bucket with managed least-privilege IAM roles.
* **BYO-IAM Mode**: Set `LambdaRoleArn` to use a pre-created, manually audited IAM role without creating IAM roles in the stack.

### Parameters
| Name | Type | Default | Description |
| :--- | :--- | :--- | :--- |
| `SourceBucketName` | String | - | Optional: The name of the S3 bucket (or comma-separated list) to automatically configure inventory for. |
| `InventoryDestination` | String | - | Optional: The S3 URI where inventory reports are delivered (e.g. `s3://my-bucket/inventory/`). If empty and `SourceBucketName` is set, a dedicated bucket is generated. |
| `LambdaRoleArn` | String | - | Optional: ARN of an existing IAM role for Lambda execution (BYO-IAM mode). |
| `KmsKeyArn` | String | - | Optional: KMS Key ARN used for decrypting SSE-KMS encrypted manifests and data files. |
| `SubnetIds` | CommaDelimitedList | - | Optional: Subnet IDs if deploying inside a private VPC. |
| `SecurityGroupIds` | CommaDelimitedList | - | Optional: Security Group IDs if deploying inside a private VPC. |
| `EnableMCPGateway` | String | `false` | Expose the CoreFunction via an HTTP Function URL and enable MCP integrations. |
| `GatewayName` | String | `s3lim-mcp` | Optional: The name of the Bedrock AgentCore MCP Gateway. |
| `InventoryFormat` | String | `Parquet` | Format of inventory files (`CSV`, `ORC`, or `Parquet`). |
| `SourcePrefixFilter` | String | - | Optional: Filter inventory to only analyze files under this prefix. |
| `CustomPrefixes` | String | - | Optional: Comma-separated list of explicit custom prefixes to track (e.g. `data/import/,temp/`). |
| `MarketplaceCustomerAWSAccountId` | String | - | The customer's AWS Account ID associated with the Marketplace subscription. |
| `MarketplaceLicenseArn` | String | - | The license ARN associated with the Marketplace subscription. |
| `EnableScheduleTrigger` | String | `true` | Enable daily scan schedule for existing inventory destinations or fallback polling. |
| `LogRetentionInDays` | Number | `365` | Number of days to retain Lambda execution logs in CloudWatch. |
| `ExecutionMode` | String | `Fast` | Execution engine: `Fast` for single-Lambda in-process execution (≤100M objects), `Distributed` for Step Functions fan-out (multi-billion objects). |
| `WorkerMaxConcurrency` | Number | `100` | Maximum concurrent Worker Lambdas when `ExecutionMode` is `Distributed` (1–500). |

### Resources Created
* **Lambda Function (`CoreFunction`)**: Core `s3lim` analysis processor.
* **SQS Queue (`ProcessingDLQ`)**: Dead Letter Queue for analysis failure handling.
* **CloudWatch LogGroup (`CoreFunctionLogGroup`)**: Retains execution logs with configurable retention.
* **CloudWatch Alarms (`ErrorAlarm`, `DLQDepthAlarm`)**: Built-in operational monitoring for analysis failures and DLQ depth.
* **IAM Roles & Policies** *(managed mode only)*: Least-privilege execution roles and policies for S3 read, CloudWatch metrics, SQS, and marketplace metering.
* **Inventory Bucket & Custom Resource** *(automated setup mode only)*: Dedicated S3 destination bucket and custom resource Lambda to configure S3 inventory on source buckets.
* **MCP Gateway Stack** *(optional)*: Bedrock AgentCore MCP Gateway integration when enabled.

### Outputs
* `InventoryDestinationURI`: S3 URI where S3 Inventory reports are stored.
* `CoreFunctionArn`: ARN of the processor Lambda function.

---

## IAM Requirements (BYO-IAM Mode)
When using an existing IAM role via `LambdaRoleArn`, your pre-created IAM role must possess the following minimum permissions:

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

#### CloudWatch, SQS & Marketplace Policy
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "CloudWatchMetrics",
            "Effect": "Allow",
            "Action": [
                "cloudwatch:PutMetricData"
            ],
            "Resource": "*"
        },
        {
            "Sid": "CloudWatchLogs",
            "Effect": "Allow",
            "Action": [
                "logs:CreateLogStream",
                "logs:PutLogEvents",
                "logs:FilterLogEvents",
                "logs:StartQuery",
                "logs:GetQueryResults",
                "logs:DescribeLogStreams",
                "logs:GetLogEvents"
            ],
            "Resource": "arn:aws:logs:*:*:log-group:/aws/lambda/s3lim-*"
        },
        {
            "Sid": "SQSDLQAccess",
            "Effect": "Allow",
            "Action": [
                "sqs:SendMessage",
                "sqs:ReceiveMessage",
                "sqs:DeleteMessage",
                "sqs:GetQueueAttributes"
            ],
            "Resource": "arn:aws:sqs:*:*:s3lim-*"
        },
        {
            "Sid": "MarketplaceMetering",
            "Effect": "Allow",
            "Action": [
                "aws-marketplace:BatchMeterUsage",
                "aws-marketplace:GetEntitlements"
            ],
            "Resource": "*"
        }
    ]
}
```

#### Wildcard (`*`) Rationale in IAM Policies
* **`arn:aws:s3:::<bucket>/*`**: Required by AWS S3 for object-level actions like `s3:GetObject` across inventory folders and keys.
* **`cloudwatch:PutMetricData` (`Resource: "*"`): Required because the CloudWatch `PutMetricData` API does not support resource-level ARNs in AWS IAM.
* **`aws-marketplace:BatchMeterUsage` / `GetEntitlements` (`Resource: "*"`): Required because AWS Marketplace Metering APIs do not support resource-level ARNs in AWS IAM.
* **`arn:aws:logs:*:*:log-group:/aws/lambda/s3lim-*` & `arn:aws:sqs:*:*:s3lim-*`**: Scopes actions strictly to `s3lim` Lambda log groups and DLQ queues across deployment regions.

---

## Execution Modes Overview

`s3lim` supports two operational engines configured via the `ExecutionMode` parameter:

* **Fast Mode (`ExecutionMode: Fast`)**: Single-Lambda execution for inventories up to 100M objects with no extra AWS resources.
* **Distributed Mode (`ExecutionMode: Distributed`)**: AWS Step Functions Distributed Map fanning out concurrent Worker Lambdas for multi-billion object inventories.

For full architectural diagrams, limit protections, lifecycle steps, and IAM policies, see the comprehensive **[Execution Modes Reference]({{< relref "execution-modes.md" >}})**.

---

## Cross-Region Considerations

When deploying `s3lim`, keep the following in mind:

* **Data Transfer Costs**: `s3lim` operates in-place, but if the Lambda function and the S3 Inventory destination bucket reside in different regions, downloading the inventory reports will incur standard **AWS Data Transfer Out (egress)** costs.
* **Execution Timeout**: Cross-region S3 downloads suffer from higher latency. For large buckets with billions of keys, cross-region transfers may cause the Lambda execution to exceed its 15-minute (900-second) timeout limit.
* **KMS Encryption**: If your S3 Inventory destination bucket is encrypted using a Customer Managed Key (CMK) via KMS, the Lambda execution role must be granted cross-region decrypt permissions (`kms:Decrypt`) on the KMS key.
* **CloudWatch Namespace**: `s3lim` publishes metrics to the region in which the Lambda is running.

> [!IMPORTANT]
> **Best Practice**: Always deploy the `s3lim` stack in the **same region** as the S3 Inventory destination bucket to eliminate data transfer fees, minimize latency, and prevent execution timeouts.

---

## Multi-Region StackSet Spec
Template: `data-plane-stackset.yaml`

Deploys `s3lim` data planes across multiple AWS regions and accounts using CloudFormation StackSets. S3 Inventory reports are region-locked. Deploying a StackSet ensures each target region runs a local `s3lim` Lambda instance, avoiding cross-region egress charges and timeouts. StackSets also support multi-account environments and integration with AWS Organizations for centralized governance.

### Prerequisites
To deploy StackSets using the `SELF_MANAGED` permission model, the following IAM roles must exist in the deploying account:
- `AWSCloudFormationStackSetAdministrationRole`
- `AWSCloudFormationStackSetExecutionRole` (must trust the administration role)

For detailed instructions on setting up these roles, refer to the official [AWS CloudFormation StackSets Prerequisites Guide](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/stacksets-prereqs-self-managed.html).

### Parameters
| Name | Type | Default | Description |
| :--- | :--- | :--- | :--- |
| `MarketplaceCustomerAWSAccountId` | String | **Required** | The customer's AWS Account ID associated with the Marketplace subscription. |
| `MarketplaceLicenseArn` | String | **Required** | The license ARN associated with the Marketplace subscription. |
| `AppVersion` | String | `latest` | The version of the s3lim templates to deploy. |
| `AdministrationRoleARN` | String | - | Optional: Custom administration role ARN. |
| `ExecutionRoleName` | String | `AWSCloudFormationStackSetExecutionRole` | Name of the IAM execution role in target regions. |
| `InventoryDestination` | String | - | Default S3 URI where inventories are delivered (can be overridden per stack instance). |
| `SourceBucketName` | String | - | Default name of the S3 bucket to analyze (for automated setup). |
| `SourcePrefixFilter` | String | - | Optional: Filter inventory to only analyze files under this prefix. |
| `EnableMCPGateway` | String | `false` | Expose the CoreFunction via an HTTP Function URL and enable MCP integrations. |
| `KmsKeyArn` | String | - | Optional: KMS Key ARN for encrypted inventory files. |

### Deploying Stack Instances and Parameter Overrides
Deploying the coordinator template creates the StackSet container. You can override parameters per target region:

```bash
aws cloudformation create-stack-instances \
  --stack-set-name s3lim-processor-multiregion \
  --accounts 111122223333 \
  --regions us-west-2 \
  --parameter-overrides ParameterKey=InventoryDestination,ParameterValue=s3://my-inventory-bucket-us-west-2/
```
