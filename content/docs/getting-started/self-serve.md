---
title: "Self-Serve Deployment"
weight: 2
---

# Self-Serve Deployment (BYO-IAM)

This guide is for teams who require granular control over IAM roles, KMS encryption, and VPC networking.
The SAM template will only manage the s3lim lambda and all other resources will need to be created.

## Architecture Overview
The Self-Serve deployment uses the `readonly` template, which creates no IAM roles. You must provide a pre-existing IAM role ARN for the Lambda function.

## Prerequisites
* Existing S3 Inventory Report Bucket Destination
* IAM Role 

## IAM Requirements
Your Lambda execution role must have the following permissions:

### S3 Permissions
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": ["s3:GetObject", "s3:ListBucket"],
            "Resource": [
                "arn:aws:s3:::your-inventory-bucket",
                "arn:aws:s3:::your-inventory-bucket/*"
            ]
        }
    ]
}
```

### CloudWatch Permissions
- `cloudwatch:PutMetricData` (Namespace: `s3lim`)
- `logs:CreateLogStream`
- `logs:PutLogEvents`

## Deployment Steps
1. Create the IAM role with the permissions above.
2. Deploy the `s3lim-readonly` stack via SAM or CloudFormation.
3. Provide the `LambdaRoleArn` as a parameter.
4. Manually configure your S3 Inventory to deliver to your chosen bucket.
5. Add an S3 Event Notification to trigger the `s3lim` Lambda ARN.
