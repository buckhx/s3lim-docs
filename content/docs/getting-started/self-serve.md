---
title: "Self-Serve Deployment"
weight: 3
---

# Self-Serve Mode (BYO-IAM) Walkthrough

This guide is for teams who require granular control over IAM roles, KMS encryption, and VPC networking. 

In Self-Serve Mode, the SAM template only deploys the analysis Lambda function, allowing you to attach a pre-created, manually audited IAM execution role.

## Use Case
Ideal for **strict enterprise environments** where security compliance requires manual review and approval of all IAM policies before deployment.

## Prerequisites
* An AWS account with permissions to deploy CloudFormation and create IAM roles.
* A pre-existing S3 Inventory report pipeline delivering reports to a destination S3 bucket.
* A pre-created IAM role for the Lambda function. For policy templates, see the [Self-Serve IAM Requirements]({{< relref "docs/reference/deployment.md#read-only-mode-spec-self-serve" >}}).

## Deployment Steps

1. **Create the IAM Role**: Establish an IAM Role for the Lambda function containing the required permissions. You can copy the exact S3 and CloudWatch/SQS policies from the [Self-Serve IAM Specification]({{< relref "docs/reference/deployment.md#read-only-mode-spec-self-serve" >}}).
2. **Launch Read-Only Stack**: Navigate to the [s3lim-readonly](https://serverlessrepo.aws.amazon.com/applications) application on the AWS Serverless Application Repository.
3. **Configure Stack Parameters**:
   * Enter `InventoryDestination` (e.g., `s3://my-inventory-bucket/inventory/`).
   * Enter `LambdaRoleArn` (the ARN of the IAM role you created in Step 1).
   * For other options, see the [Deployment Specifications]({{< relref "docs/reference/deployment.md#read-only-mode-spec-self-serve" >}}).
4. **Deploy**: Click **Deploy** to start provisioning.
5. **Add Trigger**: Manually add an S3 Event Trigger to the destination bucket targeting the `s3lim` Lambda function to execute scans on new manifest deliveries.

## Verification
Once a new S3 Inventory report is delivered, verify execution by inspecting the CloudWatch logs for the Lambda function.
