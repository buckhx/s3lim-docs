---
title: "Getting Started"
description: "Select the best deployment mode to start analyzing S3 inventory reports and optimizing your AWS S3 storage costs."
---

# Getting Started with s3lim

`s3lim` is designed to be highly modular, allowing you to choose the deployment model that best aligns with your team's AWS permissions, security constraints, and existing infrastructure.

To help you get started, we offer three main deployment modes:

1. **[Autopilot Mode]({{< relref "autopilot.md" >}})**: The quickest, fully automated setup. Best if you want `s3lim` to configure everything—including S3 Inventory reports on your source bucket—automatically.
2. **[Standard Mode]({{< relref "standard.md" >}})**: Best if you already have S3 Inventory reports configured. Supports both **Managed IAM** (automated least-privilege roles) and **BYO-IAM** (pre-audited customer execution roles).
3. **[Multi-Region Mode (StackSets)]({{< relref "multi-region.md" >}})**: Best if you want to deploy `s3lim` across multiple regions and accounts simultaneously using AWS CloudFormation StackSets.
4. **[MCP Gateway Setup]({{< relref "mcp.md" >}})**: Guide on how to enable the Amazon Bedrock AgentCore MCP Gateway (by setting `EnableMCPGateway` to `true` during deployment) to connect AI assistants.

---

## Choosing a Mode

* **Choose Autopilot** if you want to optimize a bucket quickly without reading S3 Inventory specs.
* **Choose Standard** if S3 Inventory is already running on your buckets. Use default settings for managed IAM, or provide `LambdaRoleArn` if your security team requires pre-audited BYO-IAM roles.
* **Choose Multi-Region (StackSets)** if you have data distributed across multiple AWS regions or accounts and want to manage a centralized analytics pipeline.
* **Want natural language cost audits from AI agents?** Enable the [MCP Gateway Setup]({{< relref "mcp.md" >}}) on your stack.

**NOTE:** Always deploy `s3lim` in the **same AWS Region** as the S3 bucket with the inventory file to avoid cross-region data transfer fees and potential timeout issues.
