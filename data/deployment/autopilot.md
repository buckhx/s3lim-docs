# Autopilot Deployment Reference

s3lim (Autopilot) - Automatically configure and analyze S3 inventory.


## Parameters

| Name | Type | Default | Description |
|------|------|---------|-------------|
| `CustomPrefixes` | String |  | Optional: Comma-separated list of explicit custom prefixes to track (e.g. 'data/import/,temp/'). |
| `EnableMCPGateway` | String | false | Expose the CoreFunction via an HTTP Function URL and enable MCP integrations. |
| `GatewayName` | String | s3lim-mcp | Optional: The name of the Bedrock AgentCore MCP Gateway. |
| `InventoryDestination` | String |  | Optional: The S3 URI where inventories will be delivered (e.g. s3://my-bucket/inventory/). If not provided, a bucket will be generated. |
| `InventoryFormat` | String | Parquet | The format of the inventory files. |
| `MarketplaceCustomerID` | String |  | Optional: The resolved customer identifier associated with the buyer's subscription. |
| `MaxPrefixDepth` | Number | 10 | Maximum depth for recursive prefix aggregation. |
| `SkipMarketplaceValidation` | String | false | Optional: Skip validating AWS Marketplace customer entitlement (useful for development/testing). |
| `SourceBucketName` | String | - | The name of the existing S3 bucket to create an inventory for. |
| `SourcePrefixFilter` | String |  | Optional: The prefix of the objects in the source bucket to include in the inventory. Leave empty for the entire bucket. |

## Outputs

| Name | Description |
|------|-------------|
| `CoreFunctionArn` | Lambda function ARN |
| `InventoryDestinationURI` | S3 URI where inventories are delivered |

