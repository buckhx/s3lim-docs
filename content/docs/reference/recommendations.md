---
title: "Optimization Recommendations"
weight: 25
description: "Reference guide for post-aggregation prescriptive recommendation rules and dual remediation playbooks."
---

# S3 Optimization Recommendations

`s3lim` transforms raw storage anomalies into prioritized, prescriptive action plans using its post-aggregation **Recommendation Engine**.

Rather than requiring operators to manually decipher dozens of prefix-level metrics, `s3lim` automatically synthesizes concrete S3 Lifecycle configurations and architectural playbooks.

---

## The Dual Remediation Model

Recommendations follow one of two execution paths based on architectural complexity:

1. **Direct Remediation (`DIRECT`)**:
   - Single declarative S3 configuration changes (e.g. S3 Lifecycle transitions, noncurrent version expirations, multipart upload aborts).
   - Generates exact copy-pasteable AWS CLI commands (`aws s3api put-bucket-lifecycle-configuration ...`) and Lifecycle JSON snippets ready for immediate execution.
2. **Architectural Playbooks (`PLAYBOOK`)**:
   - Multi-step remediation workflows for structural storage patterns (e.g. Small File Traps, Duplicate Content).
   - Combines immediate configuration filters (e.g., `ObjectSizeGreaterThan: 131072`) with guided compaction and deduplication procedures (S3 Batch Operations, Athena CTAS, or Kinesis Firehose buffer expansion).

---

## The 7 Core Optimization Rules

### 1. Intelligent-Tiering Transition (`intelligent-tiering`)
* **Category**: `Tiering`
* **Remediation Model**: `DIRECT`
* **Severity**: `CRITICAL` / `HIGH` / `MEDIUM`
* **Action**: Configures S3 Lifecycle transition to `INTELLIGENT_TIERING` after 90 days for Standard storage objects (>128KB).
* **Impact**: Up to 68% reduction in storage unit costs with zero retrieval fees or performance trade-offs.

### 2. Small-File Trap Mitigation (`small-file-trap`)
* **Category**: `SmallFiles`
* **Remediation Model**: `PLAYBOOK`
* **Severity**: `HIGH` / `MEDIUM`
* **Action**:
  1. Add `ObjectSizeGreaterThan: 131072` (128KB) filter to existing transition rules to halt billing penalty accumulation immediately.
  2. Run S3 Batch Operations or Athena CTAS compaction to combine small files into larger archival formats (e.g. Parquet/Tar/Zip).
  3. Tune upstream ingestion buffers (e.g. Kinesis Firehose buffer size) to prevent small file generation.

### 3. Incomplete Multipart Upload Aborts (`incomplete-multipart-uploads`)
* **Category**: `MultipartUploads`
* **Remediation Model**: `DIRECT`
* **Severity**: `CRITICAL` / `HIGH`
* **Action**: Configures `AbortIncompleteMultipartUpload` lifecycle rule to clean up orphaned upload chunks older than 7 days.
* **Impact**: Reclaims 100% of leaked multipart storage spend.

### 4. Ghost Version Expiration (`ghost-version-expiration`)
* **Category**: `GhostVersions`
* **Remediation Model**: `DIRECT`
* **Severity**: `HIGH` / `MEDIUM`
* **Action**: Configures `NoncurrentVersionExpiration` rule (e.g. 30 days) to prune historical versions accumulating invisible storage overhead.

### 5. Expired Delete Marker Cleanup (`delete-marker-cleanup`)
* **Category**: `DeleteMarkers`
* **Remediation Model**: `DIRECT`
* **Severity**: `MEDIUM` / `LOW`
* **Action**: Configures `ExpiredObjectDeleteMarker` cleanup rule to purge orphaned delete markers, improving List request latency and reducing metadata bloat.

### 6. Stale Cache Expiration (`stale-cache-expiration`)
* **Category**: `StaleCache`
* **Remediation Model**: `DIRECT`
* **Severity**: `HIGH` / `MEDIUM`
* **Action**: Configures automated prefix `Expiration` rules (e.g. 30 days) on transient and cache directories.

### 7. Duplicate Content Deduplication (`duplicate-content-dedup`)
* **Category**: `Duplicates`
* **Remediation Model**: `PLAYBOOK`
* **Severity**: `CRITICAL` / `HIGH` / `MEDIUM`
* **Action**:
  1. Run Athena queries against s3lim inventory logs to locate duplicate ETags.
  2. Execute S3 Batch Operations deletion/consolidation manifests.
  3. Transition upstream ingestion workflows to content-addressed storage (CAS) hashes.

---

## Accessing Recommendations

* **CloudWatch Dashboard**: Displayed in the top-level **Prioritized Optimization Action Queue** table widget directly beneath executive KPI cards.
* **MCP Server**: Queryable by AI assistants using `get_recommendations` and `explain_remediation` tools.
* **CLI**: Emitted in structured JSON output (`--output=json`) and formatted terminal summaries (`--output=console`).
