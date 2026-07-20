---
title: "Model Context Protocol (MCP)"
description: "Reference guide for MCP tools, resources, schemas, and customer value in the s3lim analysis engine."
weight: 35
tech_metadata:
  dependencies: "Model Context Protocol"
  proficiency: "Intermediate"
---

# Model Context Protocol (MCP) Specification

`s3lim` implements the **Model Context Protocol (MCP)**, a standardized protocol that connects Large Language Models (LLMs) and agentic workflows to `s3lim`'s high-performance storage cost optimization engine. 

By exposing S3 inventory scan metrics through MCP, AI assistants (such as **Amazon Q Developer** or **Claude Desktop**) can perform **Natural Language Querying (NLQ)** over massive S3 buckets, autonomously identifying waste and recommending cost-saving actions.

---

## Integration Architecture

`s3lim` supports two runtime transports for MCP:

1. **Local Mode (Stdio)**: The `s3lim mcp` CLI subcommand runs as a local subprocess. The host application (e.g. Claude Desktop) communicates with the server via standard input/output (`stdin`/`stdout`).
2. **Serverless Mode (Direct Invoke)**: The AWS Lambda `AnalyzeFunction` acts as a private MCP server. **Amazon Bedrock AgentCore Gateway** handles the protocol translation and invokes the Lambda directly via IAM-authenticated `InvokeFunction` payloads containing the JSON-RPC envelope.

---

## Tool Reference & Customer Value

All tools automatically generate their input and output JSON Schemas from Go structures at runtime.

### 1. `list_waste_categories`

#### Description
Summarizes all categories of storage waste across the entire S3 bucket based on the latest inventory analysis.

#### Input Schema
An empty object (no parameters required).

#### Output Schema
```json
{
  "type": "object",
  "properties": {
    "bucket": { "type": "string" },
    "total_objects": { "type": "integer", "minimum": 0 },
    "duplicate_objects": { "type": "integer", "minimum": 0 },
    "duplicate_percent": { "type": "number" },
    "waste_categories": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "category": { "type": "string" },
          "description": { "type": "string" },
          "metric_value": { "type": "integer" },
          "unit": { "type": "string" }
        },
        "required": ["category", "description", "metric_value", "unit"]
      }
    }
  },
  "required": ["bucket", "total_objects", "duplicate_objects", "duplicate_percent", "waste_categories"]
}
```

#### Concrete Customer Value
This tool gives the LLM and the user an **executive summary of the primary cost drivers** in the bucket. Instead of digging through raw lines of reports, the customer instantly knows if their biggest optimization opportunity lies in:
- Compacting hundreds of thousands of **small files** (saving on S3 request charges).
- Deleting expired **delete markers** (improving S3 `LIST` latency and metadata size).
- Fine-tuning versioning policies to eliminate **ghost versions** (reclaiming storage).
- Cleaning up aborted or incomplete **multipart uploads**.
- Identifying redundant **duplicate files** (ETag duplication).

---

### 2. `query_prefix`

#### Description
Retrieves detailed S3 storage metrics, small file density, delete markers, ghost versions, and stale cache statistics for a specific object prefix.

#### Input Schema
```json
{
  "type": "object",
  "properties": {
    "prefix": {
      "type": "string",
      "description": "The S3 prefix path to query details for (e.g., 'logs/raw/')."
    }
  },
  "required": ["prefix"]
}
```

#### Output Schema
```json
{
  "type": "object",
  "properties": {
    "metrics": {
      "type": "object",
      "properties": {
        "prefix": { "type": "string" },
        "total_objects": { "type": "integer" },
        "total_size": { "type": "integer" },
        "storage_classes": { "type": "object" },
        "average_age_seconds": { "type": "integer" },
        "oldest_age_seconds": { "type": "integer" },
        "direct_size": { "type": "integer" },
        "small_file_count": { "type": "integer" },
        "direct_small_file_count": { "type": "integer" },
        "delete_markers": { "type": "integer" },
        "multipart_upload_size": { "type": "integer" },
        "multipart_upload_count": { "type": "integer" },
        "ghost_versions_size": { "type": "integer" },
        "ghost_versions_count": { "type": "integer" },
        "zero_byte_count": { "type": "integer" },
        "stale_cache_size": { "type": "integer" },
        "stale_cache_count": { "type": "integer" },
        "duplicate_count": { "type": "integer" },
        "duplicate_size": { "type": "integer" }
      }
    },
    "message": { "type": "string" }
  }
}
```

#### Concrete Customer Value
Allows the customer to **pinpoint and audit specific folders or application paths** that are generating storage bloat. For example, if the user asks: *"Why is my `backups/` prefix so expensive?"*, the LLM can use `query_prefix` to discover that `backups/` contains a high volume of non-current ghost versions and recommend a targeted lifecycle rule to transition them to Glacier or expire them after 14 days.

---

### 3. `list_top_prefixes`

#### Description
Lists and ranks the top-K hot prefixes in the bucket sorted by a specific waste or size metric.

#### Input Schema
```json
{
  "type": "object",
  "properties": {
    "metric": {
      "type": "string",
      "description": "The sorting metric: 'size', 'objects', 'small_files', 'delete_markers', 'ghost_versions', 'multipart_uploads', or 'duplicates'."
    },
    "limit": {
      "type": "integer",
      "description": "The maximum number of prefixes to return (default: 10)."
    }
  },
  "required": ["metric", "limit"]
}
```

#### Output Schema
```json
{
  "type": "object",
  "properties": {
    "metric": { "type": "string" },
    "prefixes": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "prefix": { "type": "string" },
          "metric_value": { "type": "integer" }
        },
        "required": ["prefix", "metric_value"]
      }
    }
  },
  "required": ["metric", "prefixes"]
}
```

#### Concrete Customer Value
Surfaces the **\"Top Offenders\"** across the entire bucket for a given waste category. Instead of auditing millions of directories, storage admins can ask: *\"What are the top 5 prefixes by delete markers?\"* or *\"Where are my largest incomplete multipart uploads?\"*. This yields an **immediate action list** where optimization efforts will achieve the maximum return on investment (ROI).

---

## Resource Reference & Customer Value

### `s3lim://reports/latest`

#### Description
Exposes the complete aggregated analysis report parsed from the latest `s3lim` execution.

#### MIME Type
`application/json`

#### Concrete Customer Value
This resource provides the LLM with a **complete structured dump of the raw database**. This allows advanced agents to:
- Correlate metrics across multiple tools.
- Perform complex custom logical reasoning (e.g. cross-referencing duplicate rates against prefix age).
- Answer open-ended questions about bucket topology and storage trends.

---

## Example LLM Agent Dialogue

Here is an example of how an AI assistant (like Amazon Q Developer) handles a user's cost optimization query using `s3lim`'s MCP server:

```
User: I need to optimize my S3 storage costs for bucket "prod-data-lake". Where should I start?

Agent: Let me check the latest s3lim optimization report for "prod-data-lake".
[Tool Call: list_waste_categories]
Response: {
  "bucket": "prod-data-lake",
  "total_objects": 450000,
  "duplicate_objects": 45000,
  "duplicate_percent": 10.0,
  "waste_categories": [
    { "category": "small_files", "metric_value": 312000, "unit": "count" },
    { "category": "delete_markers", "metric_value": 85000, "unit": "count" },
    { "category": "ghost_versions", "metric_value": 45000000000, "unit": "bytes" }
  ]
}

Agent: The scan results for "prod-data-lake" reveal three major cost optimization opportunities:
1. Small Files: You have 312,000 small files (under 128KB), which account for over 69% of your total object count. This is a classic "Small File Trap" inflating your S3 GET/PUT request billing.
2. Delete Markers: There are 85,000 expired delete markers that consume metadata space and slow down list operations.
3. Ghost Versions: You have 45GB of non-current object versions (ghost versions) occupying space.

Let me find the top directories contributing to the ghost version bloat.
[Tool Call: list_top_prefixes with metric="ghost_versions", limit=3]
Response: {
  "prefixes": [
    { "prefix": "analytics/temp/", "metric_value": 38000000000 },
    { "prefix": "exports/csv/", "metric_value": 7000000000 }
  ]
}

Agent: I found that the `analytics/temp/` prefix is the primary offender, holding 38GB of non-current versions.

Recommended Actions:
1. Set up an S3 Lifecycle Rule under prefix "analytics/temp/" to permanently delete non-current versions after 7 days. This will reclaim 38GB of storage.
2. Add a bucket-wide lifecycle rule to clean up expired delete markers automatically.
3. Consolidate your data ingestion pipeline to merge small files in "analytics/" into larger objects to reduce S3 request fees.
```
