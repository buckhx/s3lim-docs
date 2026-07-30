# Standard Deployment Reference

s3lim (Standard) - Process existing S3 Inventory reports with managed permissions.


## Parameters

| Name | Type | Default | Description |
|------|------|---------|-------------|
| `CustomPrefixes` | String |  | Optional: Comma-separated list of explicit custom prefixes to track (e.g. 'data/import/,temp/'). |
| `EnableMCPGateway` | String | false | Expose the AnalyzeFunction via an HTTP Function URL and enable MCP integrations. |
| `GatewayName` | String | s3lim-mcp | Optional: The name of the Bedrock AgentCore MCP Gateway. |
| `InventoryDestination` | String | - | The S3 URI where existing inventories are delivered (e.g. s3://my-bucket/inventory/). |
| `MarketplaceCustomerID` | String |  | Optional: The resolved customer identifier associated with the buyer's subscription. |
| `MaxPrefixDepth` | Number | 10 | Maximum depth for recursive prefix aggregation. |
| `SkipMarketplaceValidation` | String | false | Optional: Skip validating AWS Marketplace customer entitlement (useful for development/testing). |
| `SourceBucketName` | String |  | Optional: The name of the source S3 bucket. Required if you want to run previews via ListObjectsV2. |

## Outputs

| Name | Description |
|------|-------------|
| `AnalyzeFunctionArn` | Lambda function ARN. You can manually add this as an S3 Event Trigger to your bucket. |

