# Readonly Deployment Reference

s3lim (Read-Only) - Process existing S3 Inventory reports.


## Parameters

| Name | Type | Default | Description |
|------|------|---------|-------------|
| `CustomPrefixes` | String |  | Optional: Comma-separated list of explicit custom prefixes to track (e.g. 'data/import/,temp/'). |
| `EnableMCPGateway` | String | false | Expose the CoreFunction via an HTTP Function URL and enable MCP integrations. |
| `GatewayName` | String | s3lim-mcp | Optional: The name of the Bedrock AgentCore MCP Gateway. |
| `InventoryDestination` | String | - | The S3 URI where existing inventories are delivered (e.g. s3://my-bucket/inventory/). |
| `LambdaRoleArn` | String | - | ARN of an existing IAM Role for the Lambda function. The role must have: 1. 's3:GetObject' and 's3:ListBucket' on your inventory destination. 2. 'cloudwatch:PutMetricData' (Namespace: s3lim). 3. 'logs:CreateLogStream', 'logs:PutLogEvents', 'logs:FilterLogEvents', 'logs:StartQuery', 'logs:GetQueryResults', 'logs:DescribeLogStreams', and 'logs:GetLogEvents'. 4. 'sqs:SendMessage', 'sqs:ReceiveMessage', 'sqs:DeleteMessage', and 'sqs:GetQueueAttributes' on the DLQ. 5. 'aws-marketplace:BatchMeterUsage' and 'aws-marketplace:GetEntitlements' to authenticate subscription and report usage.
 |
| `MarketplaceCustomerID` | String |  | Optional: The resolved customer identifier associated with the buyer's subscription. |
| `MaxPrefixDepth` | Number | 10 | Maximum depth for recursive prefix aggregation. |

## Outputs

| Name | Description |
|------|-------------|
| `CoreFunctionArn` | Lambda function ARN.  To improve performance and remove the daily cron: 1. Add an 'S3 Event Trigger' to your inventory bucket. 2. Filter for prefix: 'inventory/' and suffix: 'manifest.json'. 3. Grant 'lambda:InvokeFunction' permission to S3.
 |

