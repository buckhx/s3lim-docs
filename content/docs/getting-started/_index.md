---
title: "Getting Started"
description: "Select the best deployment mode to start analyzing S3 inventory reports and optimizing your AWS S3 storage costs."
---

# Getting Started with s3lim

`s3lim` is designed to be highly modular, allowing you to choose the deployment model that best aligns with your team's AWS permissions, security constraints, and existing infrastructure.

To help you get started, we offer three main deployment modes:

1. **[Autopilot Mode]({{< relref "autopilot.md" >}})**: The quickest, fully automated setup. Best if you want `s3lim` to configure everything—including S3 Inventory reports on your source bucket—automatically.
2. **[Standard Mode]({{< relref "standard.md" >}})**: Best if you already have S3 Inventory reports configured and delivered to a bucket, and you want `s3lim` to analyze them using automated, scoped IAM role generation.
3. **[Self-Serve Mode (BYO-IAM)]({{< relref "self-serve.md" >}})**: Designed for strict enterprise environments. It deploys only the analysis Lambda, allowing you to attach pre-created, manually audited IAM execution roles.

---

## Choosing a Mode

* **Choose Autopilot** if you want to optimize a bucket quickly without reading S3 Inventory specs.
* **Choose Standard** if S3 Inventory is already running on your buckets and you want a standard, managed Lambda setup.
* **Choose Self-Serve** if your security team requires manual approval and auditing of all IAM roles before they are deployed to production.

**NOTE:** Always deploy `s3lim` in the **same AWS Region** as the S3 bucket with the inventory file to avoid cross-region data transfer fees and potential timeout issues.
