---
title: "Getting Started"
description: "Select the best deployment mode to start analyzing S3 inventory reports and optimizing your AWS S3 storage costs."
---

# Getting Started with s3lim

`s3lim` is designed to be highly modular, allowing you to choose the deployment model that best aligns with your team's AWS permissions, security constraints, and existing infrastructure.

To help you get started, we offer three main deployment modes:

1. **Autopilot Mode**: The quickest, fully automated setup. Best if you want `s3lim` to configure everything—including S3 Inventory reports on your source bucket—automatically.
2. **Standard Mode**: Best if you already have S3 Inventory reports configured and delivered to a bucket, and you want `s3lim` to analyze them using automated, scoped IAM role generation.
3. **Self-Serve Mode (BYO-IAM)**: Designed for strict enterprise environments. It deploys only the analysis Lambda, allowing you to attach pre-created, manually audited IAM execution roles.

---

## Deployment Decision Matrix

Use the matrix below to select the right guide for your environment:

| Feature / Detail | [Autopilot]({{< relref "autopilot.md" >}}) | [Standard]({{< relref "standard.md" >}}) | [Self-Serve]({{< relref "self-serve.md" >}}) |
| :--- | :--- | :--- | :--- |
| **Setup Speed** | ⚡ Easiest (One-Click) | 🚀 Quick | ⚙️ Manual Configuration |
| **S3 Inventory Config** | Configured automatically | Uses existing setup | Uses existing setup |
| **IAM Execution Roles** | Managed & created | Managed & created | **Bring Your Own (BYO-IAM)** |
| **Target Environment** | Sandboxes, Dev, Single Teams | Standard Production | Strict Security / Enterprises |
| **Permissions Required** | Full Administrator / SAM | Managed SAM Roles | Pre-audited custom IAM role |
| **KMS / VPC Networking** | Default configurations | Default configurations | Full custom control |

---

## Choosing a Mode

* **Choose Autopilot** if you want to optimize a bucket quickly without reading S3 Inventory specs.
* **Choose Standard** if S3 Inventory is already running on your buckets and you want a standard, managed Lambda setup.
* **Choose Self-Serve** if your security team requires manual approval and auditing of all IAM roles before they are deployed to production.
