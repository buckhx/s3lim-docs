---
title: "S3 Optimization Glossary"
description: "A comprehensive guide to key terms and concepts in S3 cost optimization."
glossary:
  - term: "Small File Trap"
    definition: "An architectural pitfall where an application stores millions of small objects (typically <128KB). This results in PUT/GET request costs and S3 minimum storage charges overpowering the actual storage cost of the data."
  - term: "Ghost Versions"
    definition: "Non-current object versions and expired delete markers that occupy storage space but are not visible during standard S3 listing operations. These can account for a significant portion of an older bucket's bill."
  - term: "Lifecycle Drift"
    definition: "A state where objects have remained in expensive storage tiers (like Standard) longer than their access pattern warrants, failing to transition to IA or Glacier as defined by organizational policy."
  - term: "Zero-Egress Analysis"
    definition: "A security pattern where data is analyzed entirely within the AWS account or VPC where it resides. No data is transferred across regions or to external third-party SaaS providers, eliminating data transfer costs and security risks."
  - term: "Top-K Aggregation"
    definition: "A probabilistic algorithm used by s3lim to identify the most significant 'heavy hitter' prefixes or objects within a massive dataset using constant memory, regardless of the number of unique keys."
---

# S3 Optimization Glossary

Understanding these key terms is essential for effective storage cost management and observability. Each term represents a core concept that `s3lim` is designed to detect and measure.
