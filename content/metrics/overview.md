---
title: "Available Metrics"
weight: 20
---

# Available CloudWatch Metrics

`s3lim` surfaces storage insights through two primary mechanisms in AWS CloudWatch: **Direct Metrics** and **Log-Based Metrics**. These metrics are automatically displayed in the CloudWatch Dashboard provided by the deployment templates.

## Metric Types & Naming

### Rollup vs. Direct Metrics
- **Rollup Metrics** (default): Aggregate data recursively across prefix levels. For example, a rollup metric for the `/data/` prefix includes all objects in `/data/logs/` and `/data/temp/`. These are essential for high-level cost deconstruction.
- **Direct Metrics**: Only include objects located exactly within that specific prefix level. These are useful for surgical analysis of a single folder's contents.

### Metric Dimensions
Most metrics are partitioned by:
- `Bucket`: The name of the S3 bucket being analyzed.
- `Prefix`: The specific folder path (for prefix-based metrics).

---

## Storage Distribution

These metrics help you understand how your storage is distributed across your bucket's hierarchy.

- **`TotalSize`**: The aggregate size of all objects under a prefix (Rollup).
- **`DirectSize`**: The size of objects located specifically in this prefix.
- **`ObjectCount`**: The total count of objects under a prefix.
- **`SumAgeSeconds`**: The cumulative age of all objects in seconds. Used to identify "cold" data that hasn't been accessed or modified in a long time.

## Cost Optimization

Metrics designed specifically to identify recoverable spend and suboptimal storage configurations.

- **`DuplicatePercent`**: The percentage of total storage occupied by redundant data (files with identical ETags).
- **`DuplicateObjectSize`**: Identifies the specific largest redundant files across the bucket.
- **`ITCandidateBytes`**: Storage that meets the requirements for Intelligent Tiering but is currently in a more expensive tier (e.g., Standard).
- **`SmallFileTrapCount`**: Identifies files that are too small to be cost-effectively moved to colder tiers (IA/Glacier) due to per-object overhead.

## Metadata Insights

Deep-dive metrics into object metadata that often hides significant costs.

- **`GhostVersionSize`**: Size occupied by non-current object versions.
- **`DeleteMarkerCount`**: The number of expired object delete markers occupying metadata space.
- **`MultipartUploadSize`**: The total size of incomplete multipart uploads that are still taking up space and costing money.
- **`ZeroByteCount`**: The count of empty (0 byte) objects that may indicate application bugs or failed processing.
- **`SmallFileCount`**: The number of objects smaller than the `SMALL_FILE_THRESHOLD_KB` (default 128KB).

## Analysis Audit

Operational metrics for the `s3lim` analysis engine itself.

- **`RowsProcessed`**: The total number of object rows successfully scanned during the run.
- **`TotalBytes`**: The cumulative size of the inventory data processed.
- **`ErrorCount`**: The number of inventory rows that failed to parse correctly.
