# Multi-Region StackSet Deployment Reference

s3lim (Multi-Region StackSet) - Deploys s3lim data planes across multiple regions and accounts using StackSets.

## Parameters

| Name | Type | Default | Description |
|------|------|---------|-------------|
| `AdministrationRoleARN` | String | AWSCloudFormationStackSetAdministrationRole | ARN of the IAM role assumed by CloudFormation to manage StackSets. (Optional, defaults to AWSCloudFormationStackSetAdministrationRole) |
| `AppVersion` | String | latest | The version of the s3lim application templates to deploy. |
| `AssetsBucket` | String | slimstorage | The S3 bucket containing s3lim template artifacts. |
| `EnableMCPGateway` | String | false | Expose the CoreFunction via an HTTP Function URL and enable MCP integrations. |
| `EnableScheduleTrigger` | String | true | Optional: Set to 'false' to disable the daily scan schedule when using direct S3 event notifications. |
| `ExecutionRoleName` | String | AWSCloudFormationStackSetExecutionRole | Name of the IAM execution role in target accounts/regions. |
| `InventoryDestination` | String | - | Default S3 URI where inventories are delivered (can be overridden per regional stack instance). |
| `KmsKeyArn` | String | - | Optional: KMS Key ARN used for decrypting SSE-KMS encrypted S3 Inventory manifests and data files. |
| `MarketplaceCustomerAWSAccountId` | String | - | The customer's AWS Account ID associated with the Marketplace subscription. |
| `MarketplaceLicenseArn` | String | - | The license ARN associated with the Marketplace concurrent agreement subscription. |
| `SecurityGroupIds` | String | - | Optional: Comma-separated list of Security Group IDs if deploying s3lim inside a private VPC. |
| `SourceBucketName` | String | - | Default name of the S3 bucket to analyze (for Autopilot). |
| `SourcePrefixFilter` | String | - | Optional: The prefix of the objects in the source bucket to include in the inventory. |
| `SubnetIds` | String | - | Optional: Comma-separated list of Subnet IDs if deploying s3lim inside a private VPC. |

