# Available CloudWatch Metrics

`s3lim` automatically publishes custom CloudWatch metrics based on the aggregators enabled. Below are the metrics produced by the core analysis engine.

## RollupPrefixAggregator

Tracks the top K prefixes by total object size, rolling up all objects under each prefix level.

### Published Metrics
- `TotalSize`
- `ObjectCount`
- `SumAgeSeconds`

## SmallFileAggregator

Tracks prefixes with the highest density of small files (configurable threshold).

### Published Metrics
- `SmallFileCount`

## AuditAggregator

Tracks high-level audit metrics for the entire inventory scan, including row counts and error rates.

### Published Metrics
- `RowsProcessed`
- `TotalBytes`
- `ErrorCount`

