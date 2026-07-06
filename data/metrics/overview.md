# Available CloudWatch Metrics

`s3lim` automatically publishes custom CloudWatch metrics based on the aggregators enabled. Below are the metrics produced by the core analysis engine.

## RollupPrefixAggregator

Tracks the top K prefixes by total object size, rolling up all objects under each prefix level.

### Published Metrics
- `TotalSize`
- `ObjectCount`
- `SumAgeSeconds`

## DirectPrefixAggregator

Tracks the top K prefixes by direct object size (non-recursive) strictly within their immediate parent directory.

### Published Metrics
- `TotalSize`
- `ObjectCount`
- `SumAgeSeconds`

## SmallFileAggregator

Tracks prefixes with the highest density of small files (configurable threshold).

### Published Metrics
- `SmallFileCount`

## DirectSmallFileAggregator

Tracks the count of small files (defined by a size threshold) strictly within their immediate parent directory.

### Published Metrics
- `SmallFileCount`

## LifecycleDriftAggregator

Identifies prefixes containing candidates for Intelligent-Tiering (large, old Standard objects) and Standard/IA small file traps.

### Published Metrics
- `ITCandidateBytes`
- `SmallFileTrapCount`

## BucketDuplicationAggregator

Estimates the overall duplicate content percentage across the entire bucket.

### Published Metrics
- `DuplicatePercent`

## DuplicateObjectAggregator

Identifies the top duplicate objects based on matching ETags.

### Published Metrics
- `DuplicateObjectSize`

## ZeroByteAggregator

Tracks the top K prefixes by count of zero-byte objects.

### Published Metrics
- `ZeroByteCount`

## StaleCacheAggregator

Identifies prefixes containing stale cache or ephemeral directories older than 30 days.

### Published Metrics
- `StaleCacheSize`

## AuditAggregator

Tracks high-level audit metrics for the entire inventory scan, including row counts and error rates.

### Published Metrics
- `RowsProcessed`
- `TotalBytes`
- `ErrorCount`
- `CustomPrefixCount`

## PerformanceAggregator

Tracks hash prefix caching performance (hits, misses, evictions) to optimize sketch accuracy and execution speed.

### Published Metrics
- `CacheHits`
- `CacheMisses`
- `CacheEvictions`
- `BlockHits`
- `BlockMisses`

## GhostVersionAggregator

Calculates the total size of non-current object versions (ghost versions) under each prefix.

### Published Metrics
- `GhostVersionSize`

## DeleteMarkerAggregator

Counts the number of delete markers under each prefix.

### Published Metrics
- `DeleteMarkerCount`

