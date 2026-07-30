---
title: "MCP Gateway (Beta)"
description: "Reference guide for MCP connection, authentication, and tools in s3lim."
weight: 35
tech_metadata:
  dependencies: "Model Context Protocol"
  proficiency: "Intermediate"
---

# MCP Gateway (Beta)

`s3lim` implements the **Model Context Protocol (MCP)**, connecting LLMs and agentic workflows to high-performance S3 storage cost optimization metrics. This allows AI assistants (like Amazon Q or Claude) to query bucket status, identify waste, and suggest remediations via Natural Language Querying (NLQ).

---

## Connection & Authentication

When `EnableMCPGateway` is set to `true`, the deployment automatically configures an **Amazon Bedrock AgentCore Gateway** proxy.

### 1. Get the Connection URL
Retrieve the direct connection URL by running the following AWS CLI command:

```bash
# One shot command to get gatewayURL

aws bedrock-agentcore-control list-gateways --query "items[?starts_with(name, 's3lim')].gatewayId | [0]" \
    --output text | xargs -I {} \
    aws bedrock-agentcore-control get-gateway --gateway-id {} --query "gatewayUrl" --output text

# URL Pattern will be
# "https://<GatewayId>.gateway.bedrock-agentcore.<Region>.amazonaws.com/mcp"
```

### 2. Authentication
All requests to the Bedrock AgentCore Gateway must be signed using **AWS IAM credentials (SigV4)** with the service name `bedrock-agentcore`.

* **For local MCP clients** (Claude Desktop, VS Code, or Antigravity): Use the official [`mcp-proxy-for-aws`](https://github.com/aws/mcp-proxy-for-aws) utility. It acts as a local bridge, automatically signing requests using your local AWS CLI credentials profile.
* **For cloud-based integrations**: Route the gateway through an API Gateway configured with OAuth2 or API keys. Refer to the [Amazon Bedrock API Keys documentation](https://docs.aws.amazon.com/bedrock/latest/userguide/api-keys.html) for details on creating and managing keys.

---

## Client Configuration

Add the server to your local client (such as Claude Desktop, Antigravity, or Amazon Q CLI) by configuring the proxy bridge in your client's settings file.


```json
{
  "mcpServers": {
    "s3lim-mcp": {
      "command": "uvx",
      "args": [
        "mcp-proxy-for-aws",
        "$GATEWAY_URL"
      ],
    }
  }
}
```

Note: some clients will strip the PATH out of the env in mcp configs. So using the full path to uvx and adding HOME and PATH to the mcp env may be needed. An exit code 143 is a good indication that there is an issue with the path.


---

## MCP Tools Reference

The `s3lim` server exposes the following tools to querying clients:

> [!NOTE]
> When invoking tools via **Amazon Bedrock AgentCore**, a static `tool_name` argument (e.g. `"list_waste_categories"`) is required in the tool's input parameters. This allows the backend Lambda function to correctly identify and route the tool request, since Bedrock AgentCore forwards parameters directly to the Lambda function without a tool-identifying JSON envelope. Standard MCP clients (like Claude Desktop or Antigravity) handle routing natively via the JSON-RPC protocol and do not require this parameter.

### 1. `list_analyzed_buckets`
Lists all S3 buckets that have S3lim optimization reports available.
* **Input**: None (Standard MCP)
* **Output**: `{"buckets": ["bucket-1", "bucket-2"]}`

### 2. `list_waste_categories`
Summarizes storage waste categories (small files, duplicates, delete markers, ghost versions, multipart uploads) and estimated monthly savings for a bucket.
* **Input**:
  * `bucket` (string, optional): S3 bucket name to query.
* **Output**: Detailed object/duplicate counts, overall duplicate percentage, and estimated monthly savings.

### 3. `query_prefix`
Retrieves detailed storage metrics and optimization opportunities for a specific prefix/folder path.
* **Input**:
  * `prefix` (string, required): The S3 prefix path to query (e.g., `"downloads/"`).
  * `bucket` (string, optional): S3 bucket name to query.
* **Output**: Count, size, age, duplicate, delete marker, and multipart upload statistics for the prefix.

### 4. `list_top_prefixes`
Lists and ranks the top-K directories sorted by a specific waste or size metric.
* **Input**:
  * `metric` (string, required): One of `size`, `objects`, `small_files`, `delete_markers`, `ghost_versions`, `multipart_uploads`, `duplicates`.
  * `limit` (integer, optional): Maximum prefixes to return (default: 10).
  * `bucket` (string, optional): S3 bucket name to query.
* **Output**: Ranked list of directories and their metric values.

---

## Resources Reference

* **`s3lim://reports/latest`**: Exposes the complete aggregated analysis report parsed from the latest `s3lim` execution in structured `application/json` format.
