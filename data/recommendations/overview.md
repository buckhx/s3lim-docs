# S3 Optimization Recommendation Rules

`s3lim` includes a post-aggregation **Recommendation Engine** that evaluates 7 core optimization heuristics against aggregated inventory metrics. Rather than leaving raw metrics for manual triage, `s3lim` synthesizes prioritized, prescriptive remediation guidance using a **Dual Remediation Model**:

- **Direct Remediation (`DIRECT`)**: Generates exact copy-pasteable AWS CLI commands (`aws s3api put-bucket-lifecycle-configuration ...`) and Lifecycle JSON snippets that can be deployed immediately.
- **Architectural Playbooks (`PLAYBOOK`)**: Multi-step engineering guidance for complex patterns (e.g. Small File Traps, Duplicate Content) pairing immediate size-filter prevention with historical compaction manifests.

## Registered Optimization Rules

| ID | Name | Category | Remediation Type | Description |
|---|---|---|---|---|
| `delete-marker-cleanup` | Expired Delete Marker Cleanup | DeleteMarkers | `DIRECT` | Identifies expired object delete markers bloating version metadata and slowing S3 List requests. |
| `duplicate-content-dedup` | Duplicate Content Deduplication | Duplicates | `PLAYBOOK` | Identifies high duplicate content rates across the bucket and prescribes deduplication playbooks. |
| `ghost-version-expiration` | Ghost Version Expiration | GhostVersions | `DIRECT` | Identifies noncurrent object versions in versioned buckets accumulating invisible storage costs. |
| `incomplete-multipart-uploads` | Incomplete Multipart Upload Aborts | MultipartUploads | `DIRECT` | Identifies unfinalized multipart upload parts older than 7 days leaking storage costs silently. |
| `intelligent-tiering` | Intelligent-Tiering Transition | Tiering | `DIRECT` | Identifies Standard storage objects older than 90 days (>128KB) eligible for transition to INTELLIGENT_TIERING to reduce storage costs by up to 68%. |
| `small-file-trap` | Small-File Trap Mitigation | SmallFiles | `PLAYBOOK` | Identifies prefixes with high small-file density (<128KB in IA/IR/INT) incurring minimum 128KB billing per object, prescribing size filtering and batch compaction. |
| `stale-cache-expiration` | Stale Cache Expiration | StaleCache | `DIRECT` | Identifies temporary or cache prefixes containing stale objects unaccessed for >30 days. |

---

## Expired Delete Marker Cleanup (`delete-marker-cleanup`)

- **Category**: `DeleteMarkers`
- **Remediation Model**: `DIRECT`

### Description

Identifies expired object delete markers bloating version metadata and slowing S3 List requests.

## Duplicate Content Deduplication (`duplicate-content-dedup`)

- **Category**: `Duplicates`
- **Remediation Model**: `PLAYBOOK`

### Description

Identifies high duplicate content rates across the bucket and prescribes deduplication playbooks.

## Ghost Version Expiration (`ghost-version-expiration`)

- **Category**: `GhostVersions`
- **Remediation Model**: `DIRECT`

### Description

Identifies noncurrent object versions in versioned buckets accumulating invisible storage costs.

## Incomplete Multipart Upload Aborts (`incomplete-multipart-uploads`)

- **Category**: `MultipartUploads`
- **Remediation Model**: `DIRECT`

### Description

Identifies unfinalized multipart upload parts older than 7 days leaking storage costs silently.

## Intelligent-Tiering Transition (`intelligent-tiering`)

- **Category**: `Tiering`
- **Remediation Model**: `DIRECT`

### Description

Identifies Standard storage objects older than 90 days (>128KB) eligible for transition to INTELLIGENT_TIERING to reduce storage costs by up to 68%.

## Small-File Trap Mitigation (`small-file-trap`)

- **Category**: `SmallFiles`
- **Remediation Model**: `PLAYBOOK`

### Description

Identifies prefixes with high small-file density (<128KB in IA/IR/INT) incurring minimum 128KB billing per object, prescribing size filtering and batch compaction.

## Stale Cache Expiration (`stale-cache-expiration`)

- **Category**: `StaleCache`
- **Remediation Model**: `DIRECT`

### Description

Identifies temporary or cache prefixes containing stale objects unaccessed for >30 days.

