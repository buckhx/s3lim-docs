---
title: "Triggers"
weight: 40
---

# Analysis Triggers

`s3lim` can be triggered in multiple ways depending on your automation needs. The Lambda function is designed to handle S3 events, scheduled events, and manual JSON payloads.

## 1. Event Notifications (S3)
This is the recommended method for **real-time analysis**. Whenever a new `manifest.json` is created by S3 Inventory, the Lambda is triggered immediately.

- **Setup**: Add an "S3 Event Notification" to your inventory destination bucket.
- **Filter**: 
    - Prefix: (e.g., `inventory/`)
    - Suffix: `manifest.json`
- **Event Type**: `s3:ObjectCreated:*`

## 2. Scheduled Scans (Cron)
If you prefer to run analysis on a fixed schedule (e.g., once per day), you can use an EventBridge (CloudWatch Events) rule.

- **Behavior**: When triggered via a schedule, `s3lim` scans the `INVENTORY_DESTINATION` S3 path to find the most recent `manifest.json` for all buckets it has discovered.
- **Config**: Ensure the `INVENTORY_DESTINATION` environment variable is set.

## 3. Manual Trigger
You can manually invoke the Lambda function via the AWS Console or CLI for ad-hoc analysis.

- **Payload**: Provide a simple JSON string containing the S3 URI of the `manifest.json`.
- **Example**:
  ```json
  "s3://my-inventory-bucket/inventory/2026-06-12T00-00Z/manifest.json"
  ```
- **Alternative Payload**:
  ```json
  {
    "path": "s3://my-inventory-bucket/inventory/2026-06-12T00-00Z/manifest.json"
  }
  ```

## 4. DLQ Redrive (SQS)
`s3lim` includes a built-in mechanism for reprocessing failed inventories.

- **Trigger**: An SQS event source mapping from the `ProcessingDLQ`.
- **Action**: This trigger is **disabled by default**. To re-process failed runs, enable the trigger in the Lambda "Triggers" tab, wait for the queue to drain, and then disable it again.
