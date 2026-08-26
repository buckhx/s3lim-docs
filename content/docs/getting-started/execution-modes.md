---
title: "Execution Architecture"
description: "Architecture and operational guidance for s3lim's serverless Step Functions Distributed Map pipeline and local CLI engine."
weight: 4
---

# Execution Architecture

`s3lim` is engineered around a unified, horizontally scalable serverless architecture that processes S3 inventory reports of any size—from small gigabyte-scale buckets to multi-billion object enterprise data lakes—with zero Lambda timeout constraints and zero data egress.

---

## Architecture Overview

`s3lim` provides two execution targets tailored for different operational environments:

1. **Serverless Data Plane (AWS Marketplace & CloudFormation)**:
   All customer data plane deployments (`data-plane-template.yaml` and `data-plane-stackset.yaml`) use the unified **Step Functions Distributed Map** execution engine. Analysis jobs automatically fan out across concurrent Worker Lambdas, streaming and aggregating individual inventory shards in parallel.

2. **Local CLI Engine (`s3lim analyze`)**:
   For local developer workflows, continuous integration (CI) test suites, and standalone Model Context Protocol (MCP) tooling, `s3lim` runs as a high-performance, single-process streaming CLI binary.

---

## Serverless Distributed Map Pipeline

The serverless data plane orchestrates analysis using AWS Step Functions Express workflows and Distributed Map:

```mermaid
flowchart TD
    Manifest[S3 Inventory Manifest] --> Init["1. Init Lambda (Manifest Partitioning & Jitter)"]
    Init --> Map["2. Step Functions Distributed Map (Worker Fan-Out)"]
    subgraph Map ["Step Functions Distributed Map"]
        W1["Worker Lambda 1 (Shard 0)"]
        W2["Worker Lambda 2 (Shard 1)"]
        W3["Worker Lambda N (Shard N)"]
    end
    W1 --> S3State["Intermediate State (S3 .s3lim/)"]
    W2 --> S3State
    W3 --> S3State
    S3State --> Reducer["3. Reducer Lambda (Result Aggregation & Reporting)"]
    Reducer --> CW[CloudWatch Metrics & Dashboards]
    Reducer --> Rep[Audit Reports (S3)]
    Reducer --> MP[Marketplace Metering (HWM)]
    Reducer --> Cleanup["4. Cleanup (Intermediate State Pruning)"]
```

### Execution Lifecycle

1. **Init Lambda**:
   * Reads and parses `manifest.json` to extract inventory file locations, compression formats, and total shard counts.
   * Calculates dynamic start jitter (`0–9s`) to smoothly pace worker container cold-starts and prevent downstream throttling.
   * Partitions shards and passes the payload to the Step Functions Distributed Map.

2. **Worker Lambdas (Distributed Map)**:
   * Spawns concurrent Worker Lambda invocations up to the configured `MaxConcurrency`.
   * Each Worker downloads and streams its assigned inventory shard (CSV, Parquet, or ORC).
   * Computes independent metric aggregations across all storage categories (storage classes, size histograms, top prefixes, duplicate ETag detection, ghost versions).
   * Serializes aggregator state into a Gzip-compressed binary stream and writes to `s3://<DestinationBucket>/<Prefix>/.s3lim/intermediate/<run-id>/<shard-id>.gz`.

3. **Reducer Lambda**:
   * Streams and deserializes all intermediate shard states from S3.
   * Merges aggregator structures across all shards with zero information loss.
   * Generates and uploads the comprehensive JSON audit report to `s3://<DestinationBucket>/<Prefix>/.s3lim/reports/<timestamp>.json`.
   * Publishes custom CloudWatch metrics and dashboard widgets.
   * Evaluates Monthly High-Water Mark (HWM) usage and submits delta units to AWS Marketplace via `BatchMeterUsage`.

4. **Cleanup Task**:
   * Prunes all intermediate shard files under `.s3lim/intermediate/<run-id>/`.
   * Includes fallback cleanup handlers in Step Functions to guarantee zero orphaned temporary storage even if a run fails.

---

## Concurrency & Quota Management (`MaxConcurrency`)

Worker concurrency is controlled by the `MaxConcurrency` CloudFormation parameter (default: `50`).

| Parameter | Type | Default | Description |
| :--- | :--- | :--- | :--- |
| `MaxConcurrency` | Number | `50` | Maximum number of concurrent Worker Lambda invocations in Distributed Map execution. |

### Sizing Guidelines

* **Default Sizing (50 Workers)**: Suitable for most accounts. Balances fast analysis turnaround with minimal consumption of regional AWS Lambda unreserved concurrency quotas.
* **High-Throughput Scaling (100–500+ Workers)**: For multi-billion object data lakes with hundreds of inventory shards, increasing `MaxConcurrency` delivers near-linear throughput scaling.
* **Quota Protection**: Setting `MaxConcurrency` ensures that `s3lim` worker fan-out does not exhaust your account's unreserved concurrency or throttle other production workloads.

> [!NOTE]
> **Regional Lambda Concurrency Quotas**:
> AWS accounts typically have a default regional concurrency quota of 1,000 unreserved concurrent executions. If you plan to configure `MaxConcurrency` above 200, verify your account's regional concurrency limit or request a quota increase via the AWS Service Quotas console. For complete guidance, see the official [AWS Lambda Concurrency Documentation](https://docs.aws.amazon.com/lambda/latest/dg/lambda-concurrency.html).

---

## IAM Policy Requirements (BYO-IAM)

When deploying with a custom pre-audited IAM role (`LambdaRoleArn`), ensure the role includes permissions for Step Functions execution and intermediate S3 state management:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "S3limIntermediateStateAccess",
      "Effect": "Allow",
      "Action": [
        "s3:PutObject",
        "s3:GetObject",
        "s3:DeleteObject",
        "s3:DeleteObjectVersion"
      ],
      "Resource": [
        "arn:aws:s3:::YOUR_INVENTORY_BUCKET/*"
      ]
    },
    {
      "Sid": "DistributedWorkflowExecution",
      "Effect": "Allow",
      "Action": [
        "states:StartExecution",
        "states:DescribeExecution",
        "states:GetExecutionHistory"
      ],
      "Resource": "arn:aws:states:*:*:stateMachine:s3lim-*"
    }
  ]
}
```

*Note: For standard Managed IAM deployments (where `LambdaRoleArn` is omitted), these permissions and the Step Functions execution role are automatically provisioned with least-privilege scoping.*

---

## Performance & Metric Equivalence

* **100% Mathematical Equivalence**: Merging aggregator state across distributed workers produces identical counts, duplicate rates, size histograms, and top prefix rankings as single-process streaming analysis.
* **Benchmarked Speedup**: On multi-shard benchmarks (such as the ESA Sentinel-2 L1C dataset of 19.17M objects across 10 shards), distributed execution achieves over **3.0x speedup** on 8 workers compared to sequential processing, reducing analysis time from ~49 seconds to ~16 seconds.

