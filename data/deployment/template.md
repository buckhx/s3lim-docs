# s3lim Data Plane Template Deployment Reference

s3lim - High-performance S3 inventory analysis tool.

## Parameters

| Name | Type | Default | Description |
|------|------|---------|-------------|
| `CustomPrefixes` | String | - | Optional: Comma-separated list of explicit custom prefixes to track (e.g. 'data/import/,temp/'). |
| `EnableMCPGateway` | String | false | Expose the CoreFunction via an HTTP Function URL and enable MCP integrations. |
| `EnableScheduleTrigger` | String | true | Optional: Set to 'true' to enable the daily scan schedule when using existing inventory destinations or fallback polling. |
| `ExecutionMode` | String | Fast | Execution mode for inventory analysis. 'Fast' runs in a single Lambda for standard buckets (<=100M objects). 'Distributed' fans out using AWS Step Functions Distributed Map for high-scale inventories. |
| `GatewayName` | String | - | Optional: The name of the Bedrock AgentCore MCP Gateway (defaults to <StackName>-mcp). |
| `InventoryDestination` | String | - | Optional: The S3 URI where inventories are delivered (e.g. s3://my-bucket/inventory/). If not provided and SourceBucketName is set, a dedicated inventory bucket will be generated. |
| `InventoryFormat` | String | Parquet | The format of the inventory files when configuring new inventory reports. |
| `KmsKeyArn` | String | - | Optional: KMS Key ARN used for decrypting SSE-KMS encrypted S3 Inventory manifests and data files. |
| `LambdaRoleArn` | String | - | Optional: ARN of an existing IAM role to use for Lambda execution. If left empty, a managed IAM role with least-privilege permissions will be created. |
| `LogRetentionInDays` | Number | 365 | The number of days to retain Lambda execution logs in CloudWatch. Default is 365 days (1 year) as required by AWS Marketplace security policies. |
| `MarketplaceCustomerAWSAccountId` | String | - | The customer's AWS Account ID associated with the Marketplace subscription. |
| `MarketplaceLicenseArn` | String | - | The license ARN associated with the Marketplace concurrent agreement subscription. |
| `SecurityGroupIds` | CommaDelimitedList | - | Optional: Comma-separated list of Security Group IDs if deploying s3lim inside a private VPC. |
| `SourceBucketName` | String | - | Optional: The name of the S3 bucket (or a comma-separated list of buckets) to automatically configure inventory for (e.g. 'my-bucket' or 'bucket-a,bucket-b'). |
| `SourcePrefixFilter` | String | - | Optional: The prefix of the objects in the source bucket to filter inventory analysis. |
| `SubnetIds` | CommaDelimitedList | - | Optional: Comma-separated list of Subnet IDs if deploying s3lim inside a private VPC. |
| `WorkerMaxConcurrency` | Number | 100 | Maximum concurrent Worker Lambda invocations in Distributed mode. |

## Outputs

| Name | Description |
|------|-------------|
| `CoreFunctionArn` | Lambda function ARN |
| `ExecutionMode` | Selected analysis execution mode (Fast or Distributed) |
| `InventoryDestinationURI` | S3 URI where inventories are delivered |
| `StateMachineArn` | Step Functions Distributed Map state machine ARN |

