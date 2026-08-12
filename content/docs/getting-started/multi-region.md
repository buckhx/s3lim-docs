---
title: "Multi-Region Mode"
weight: 4
---

# Multi-Region Mode Walkthrough

This guide is for users who want to deploy and manage `s3lim` data plane analysis engines across multiple AWS regions and accounts simultaneously.

## Use Case
Since S3 Inventory reports are region-locked (reports must reside in the same region as the source S3 bucket), customers with multi-region architectures need regional deployment of the analysis engine. 

Using AWS CloudFormation StackSets allows you to create a centralized coordinator stack in an administrator account/region, which automatically rolls out the regional `s3lim` stacks to all target regions and accounts. This eliminates cross-region data egress charges and Lambda timeout issues.

> [!NOTE]
> AWS CloudFormation StackSets natively support multi-account deployments and integration with AWS Organizations. This enables centralized governance and automatic rolling deployment of `s3lim` to any new AWS accounts or OUs added to your organization. Each regional `s3lim` deployment automatically processes inventory reports from **all source buckets** in that region delivering to the regional destination bucket.

## Prerequisites
* An AWS account with permissions to deploy CloudFormation StackSets, IAM Roles, and Lambda functions.
* **IAM Prerequisites (StackSet Administration & Execution Roles)**: The following roles must exist in the accounts you are targeting:
  - `AWSCloudFormationStackSetAdministrationRole` (in the admin account)
  - `AWSCloudFormationStackSetExecutionRole` (in target accounts/regions, trusting the admin role)
  
  For detailed guidelines, see the official [AWS CloudFormation StackSets Prerequisites Guide](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/stacksets-prereqs-self-managed.html).

  If you are running a single-account multi-region deployment for testing, you can use our consolidated [StackSet Prerequisites Helper Template](https://s3lim-test.s3.amazonaws.com/s3lim-helpers/aws/sam-template-stackset-prereqs.yaml) to provision both roles in your account:

  ```bash
  aws cloudformation create-stack \
    --stack-name s3lim-stackset-prereqs \
    --template-url https://s3lim-test.s3.amazonaws.com/s3lim-helpers/aws/sam-template-stackset-prereqs.yaml \
    --capabilities CAPABILITY_NAMED_IAM
  ```



## Deployment Steps

1. **Deploy StackSet Coordinator**: Launch the coordinator template:
   * **URL**: [s3lim-stackset](https://serverlessrepo.aws.amazon.com/applications) on the AWS Serverless Application Repository or deploy the local `sam-template-stackset.yaml` wrapper.
   * Provide template parameters:
     - `MarketplaceCustomerID`: The subscriber ID associated with your subscription.
     - `BaseMethod`: The regional base deployment mode (`autopilot`, `standard`, or `readonly`) to deploy per target instance.
2. **Deploy Stack Instances**: Use the AWS Console or the AWS CLI to deploy stack instances to your target accounts and regions.
3. **Parameter Overrides**: Override S3 bucket parameter settings (e.g. `InventoryDestination` and `SourceBucketName`) to configure the unique regional S3 buckets for each instance:

```bash
aws cloudformation create-stack-instances \
  --stack-set-name s3lim-processor-multiregion \
  --accounts 111122223333 \
  --regions us-east-1 us-west-2 \
  --parameter-overrides \
    ParameterKey=InventoryDestination,ParameterValue=s3://my-inventory-bucket-us-east-1/ \
    ParameterKey=SourceBucketName,ParameterValue=my-source-bucket-us-east-1
```

## Verification
Once S3 Inventory reports are delivered to any of your regional destination buckets, the corresponding regional Lambda function will run and publish local metrics and insights. You can manage and delete target region deployments centrally by managing the StackSet instances.
