---
title: "Getting Started"
description: "Select the best deployment method to start analyzing S3 inventory reports and optimizing your AWS S3 storage costs."
---

# Getting Started with s3lim

`s3lim` is designed to be highly modular, allowing you to choose the deployment method that best aligns with your team's AWS permissions, security constraints, and existing infrastructure.

To help you get started, we offer the following deployment methods and execution guides:

1. **[Autopilot Deployment]({{< relref "autopilot.md" >}})**: The quickest, fully automated setup. Best if you want `s3lim` to configure everything—including S3 Inventory reports on your source bucket—automatically.
2. **[Standard Deployment]({{< relref "standard.md" >}})**: Best if you already have S3 Inventory reports configured and delivered to a bucket. Supports both automated Managed IAM and custom Bring-Your-Own-IAM (BYO-IAM) roles.
3. **[Multi-Region Deployment (StackSets)]({{< relref "multi-region.md" >}})**: Best if you want to deploy `s3lim` across multiple regions and accounts simultaneously using AWS CloudFormation StackSets.
4. **[Execution Modes]({{< relref "execution-modes.md" >}})**: Detailed guide on choosing and sizing the runtime compute engine between Fast Mode (single-Lambda) and Distributed Mode (Step Functions Distributed Map).
5. **[MCP Gateway Setup]({{< relref "mcp.md" >}})**: Guide on how to enable the Amazon Bedrock AgentCore MCP Gateway (by setting `EnableMCPGateway` to `true` during deployment) to connect AI assistants.

---

## Choosing a Deployment Method

* **Choose Autopilot** if you want to optimize a bucket quickly without configuring S3 Inventory manually.
* **Choose Standard** if S3 Inventory is already running on your buckets. Use Managed IAM for automated permissions or BYO-IAM if your security team requires pre-audited IAM roles.
* **Choose Multi-Region (StackSets)** if you have data distributed across multiple AWS regions or accounts and want to manage a centralized analytics pipeline.
* **Need to scale to billions of objects?** Read the [Execution Modes Guide]({{< relref "execution-modes.md" >}}) to evaluate Fast vs Distributed mode.
* **Want natural language cost audits from AI agents?** Enable the [MCP Gateway Setup]({{< relref "mcp.md" >}}) on your stack.

**NOTE:** Always deploy `s3lim` in the **same AWS Region** as the S3 bucket with the inventory file to avoid cross-region data transfer fees and potential timeout issues.
