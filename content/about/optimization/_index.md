---
title: "S3 Cost Optimization"
description: "Strategies and patterns for reducing your AWS S3 bill using s3lim."
---

# S3 Cost Optimization

AWS S3 is incredibly powerful, but its pricing model can be deceptive. While storage is cheap, request costs and architectural patterns can lead to unexpected "bill shocks."

In this section, we explore common S3 cost pitfalls and how to use `s3lim` to identify and fix them.

## Optimization Guides

- **[The Small File Trap](/about/optimization/small-file-trap/)**: Why millions of tiny files are costing you more in requests than storage.
- **Duplicate Object Detection** (Coming Soon): Finding and removing redundant data.
- **Lifecycle Drift** (Coming Soon): Identifying objects that should have been transitioned to cheaper storage.
