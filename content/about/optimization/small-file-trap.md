---
title: "The Small File Trap"
description: "How small files can inflate your S3 costs and how to detect them with s3lim."
howto_steps:
  - name: "Run a s3lim Analysis"
    text: "Deploy s3lim to your AWS account and run a scan on the target bucket to gather metrics."
  - name: "Identify Small File Density"
    text: "Review the 'Small File Density' metric in the CloudWatch dashboard to find prefixes where objects are consistently under 128KB."
  - name: "Batch Data"
    text: "Modify your upstream application to aggregate records into larger Parquet or Avro files (aiming for 128MB+)."
  - name: "Consolidate Existing Objects"
    text: "Run a one-time compaction job to merge existing small objects into larger archives to reduce request and storage costs."
---

# The Small File Trap

One of the most common causes of inflated S3 bills is the "Small File Trap." This occurs when an application stores millions of small objects (typically under 128 KB) rather than aggregating them into larger files.

## Why Small Files Are Expensive

There are three main reasons why small files drive up your AWS bill:

### 1. Request Costs Overpower Storage Costs
S3 Standard storage costs approximately **$0.023 per GB**. However, a PUT request costs **$0.005 per 1,000 requests**.
- If you store **one 1 GB file**, you pay $0.023/month for storage and a negligible amount for the request.
- If you store **one million 1 KB files** (totaling 1 GB), you still pay $0.023/month for storage, but you pay **$5.00** just to upload them.

### 2. Minimum Storage Charges
Storage classes like **S3 Standard-IA** and **S3 One Zone-IA** have a minimum object size of **128 KB**. If you store a 10 KB file in Standard-IA, you are billed for 128 KB of storage. This can increase your storage costs by over 10x for very small files.

### 3. Metadata Overhead
Every object in S3 has index metadata. Managing millions of small objects increases the time and cost of running S3 Inventory reports and Lifecycle transitions.

## Detecting the Trap with s3lim

`s3lim` makes it easy to identify if your buckets are suffering from the small file trap. By analyzing your inventory, `s3lim` provides:

- **Average Object Size**: A quick indicator of overall bucket health.
- **Size Distribution**: Histograms showing how much of your storage is consumed by small objects vs. large ones.
- **Small File Density**: Reports highlighting prefixes with high counts of small files.

## How to Fix It

If `s3lim` identifies a small file problem, consider the following strategies:
- **Batching**: Aggregate small records into larger Parquet or Avro files before uploading.
- **S3 Object Lambda**: Use Lambda to serve small logical objects from a larger physical archive.
- **Compaction Jobs**: Periodically run jobs to merge small files in your data lake.
