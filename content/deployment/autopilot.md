---
title: "Autopilot Deployment"
weight: 10
---

# Autopilot Deployment Reference

s3lim (Autopilot) - Automatically configure and analyze S3 inventory.


## Parameters

| Name | Type | Default | Description |
|------|------|---------|-------------|
| `InventoryDestination` | String |  | Optional: The S3 URI where inventories will be delivered (e.g. s3://my-bucket/inventory/). If not provided, a bucket will be generated. |
| `InventoryFormat` | String | Parquet | The format of the inventory files. |
| `MaxPrefixDepth` | Number | 10 | Maximum depth for recursive prefix aggregation. |
| `SourceBucketName` | String | - | The name of the existing S3 bucket to create an inventory for. |
| `SourcePrefixFilter` | String |  | Optional: The prefix of the objects in the source bucket to include in the inventory. Leave empty for the entire bucket. |

## Outputs

| Name | Description |
|------|-------------|
| `AnalyzeFunctionArn` | Lambda function ARN |
| `InventoryDestinationURI` | S3 URI where inventories are delivered |

