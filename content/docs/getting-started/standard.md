---
title: "Standard Deployment"
weight: 2
---

# Standard Mode Walkthrough

This guide is for users who **already have S3 Inventory reports** configured and delivered to a destination bucket, and want `s3lim` to analyze them.

## Use Case
Ideal for analyzing pre-existing inventory pipelines. `s3lim` supports two IAM permission options:
- **Managed IAM (Default)**: Automatically provisions least-privilege IAM execution roles so you don't have to configure policies manually.
- **Bring-Your-Own-IAM (BYO-IAM)**: For strict enterprise compliance environments where IAM policies must be pre-audited, you can supply your own execution role via `LambdaRoleArn`.

## Prerequisites
* An AWS account with permissions to deploy Lambda and CloudFormation (and IAM roles if using Managed IAM).
* A pre-existing S3 Inventory report pipeline delivering reports to a destination S3 bucket.
* The S3 URI where your existing inventory manifests are delivered (e.g., `s3://my-inventory-bucket/inventory/`).
* *(BYO-IAM only)*: A pre-created IAM role ARN (`LambdaRoleArn`) configured with the permissions outlined in [BYO-IAM Permissions](#byo-iam-role-permissions-lambdarolearn) below.

## Deployment Steps

1. **Locate Inventory Destination**: Note the S3 bucket and prefix path where inventory manifests are delivered (e.g. `s3://my-inventory-bucket/inventory/`).
2. **Launch Stack**: Navigate to the [s3lim](https://serverlessrepo.aws.amazon.com/applications) application on the AWS Serverless Application Repository (or deploy `aws/customer/sam-template.yaml`).
3. **Configure Stack Parameters**:
   * **`InventoryDestination`**: Enter your inventory reports S3 URI.
   * **`LambdaRoleArn`** *(optional)*: For BYO-IAM mode, provide the ARN of your pre-audited IAM role. Leave blank for Managed IAM.
   * **`SourceBucketName`** *(optional)*: Required only if you want to run previews using S3 ListObjectsV2 API.
   * For other options, see the [Deployment Specifications]({{< relref "docs/reference/deployment.md#unified-data-plane-spec" >}}).
4. **Deploy**: Click **Deploy** to start provisioning.

---

## BYO-IAM Role Permissions (`LambdaRoleArn`)

If your enterprise security policy requires using a pre-audited IAM role, create an IAM role using the policies below and supply its ARN to `LambdaRoleArn`.

<div class="not-prose my-6 p-5 bg-blue-50/70 border border-blue-200 rounded-2xl space-y-3">
    <label for="byo-bucket-input" class="block text-xs font-bold uppercase tracking-wider text-blue-900 flex items-center gap-2">
        <svg class="w-4 h-4 text-blue-600" fill="none" stroke="currentColor" stroke-width="2" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" d="M3 7v10a2 2 0 002 2h14a2 2 0 002-2V9a2 2 0 00-2-2h-6l-2-2H5a2 2 0 00-2 2z"></path>
        </svg>
        Customize Policies: Enter Your Inventory Bucket Name
    </label>
    <div class="relative">
        <input type="text" id="byo-bucket-input" oninput="updateByoPolicies()" placeholder="e.g. my-inventory-destination-bucket" 
               class="w-full text-sm py-2.5 px-4 bg-white border border-blue-300 rounded-xl focus:outline-none focus:ring-2 focus:ring-blue-500 font-mono text-slate-800 shadow-sm" />
    </div>
    <p class="text-xs text-blue-700 m-0">Enter your bucket name (or S3 URI) above to automatically customize the resource ARNs in the policies below.</p>
</div>

### 1. Trust Relationship (Assume Role Policy)

The IAM execution role must trust the AWS Lambda service.

<div class="not-prose my-4 rounded-xl overflow-hidden border border-slate-800 bg-slate-950 shadow-md">
    <div class="flex items-center justify-between px-4 py-2.5 bg-slate-800 text-slate-200 text-xs font-mono border-b border-slate-700/80">
        <span class="font-medium text-slate-300">trust-policy.json</span>
        <button type="button" id="copy-trust-btn" onclick="copyCode('byo-trust-code', 'copy-trust-btn')" 
                class="flex items-center gap-1.5 bg-slate-700 hover:bg-slate-600 text-white px-2.5 py-1 rounded-md transition-colors text-xs font-sans">
            <svg class="w-3.5 h-3.5" fill="none" stroke="currentColor" stroke-width="2" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" d="M8 16H6a2 2 0 01-2-2V6a2 2 0 012-2h8a2 2 0 012 2v2m-6 12h8a2 2 0 002-2v-8a2 2 0 00-2-2h-8a2 2 0 00-2 2v8a2 2 0 002 2z"></path></svg>
            <span class="btn-text">Copy</span>
        </button>
    </div>
    <pre style="margin:0 !important; border:none !important; border-radius:0 !important; padding:1rem !important; background:#0f172a !important; color:#34d399 !important;" class="overflow-x-auto text-xs font-mono leading-relaxed"><code id="byo-trust-code" style="background:transparent !important; color:inherit !important; padding:0 !important;">{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "lambda.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}</code></pre>
</div>

### 2. IAM Permissions Policy

Attach this policy to the role. Resources are scoped strictly to the inventory bucket, `s3lim-*` log groups, and `s3lim-*` DLQs.

<div class="not-prose my-4 rounded-xl overflow-hidden border border-slate-800 bg-slate-950 shadow-md">
    <div class="flex items-center justify-between px-4 py-2.5 bg-slate-800 text-slate-200 text-xs font-mono border-b border-slate-700/80">
        <span class="font-medium text-slate-300">s3lim-execution-policy.json</span>
        <button type="button" id="copy-policy-btn" onclick="copyCode('byo-policy-code', 'copy-policy-btn')" 
                class="flex items-center gap-1.5 bg-slate-700 hover:bg-slate-600 text-white px-2.5 py-1 rounded-md transition-colors text-xs font-sans">
            <svg class="w-3.5 h-3.5" fill="none" stroke="currentColor" stroke-width="2" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" d="M8 16H6a2 2 0 01-2-2V6a2 2 0 012-2h8a2 2 0 012 2v2m-6 12h8a2 2 0 002-2v-8a2 2 0 00-2-2h-8a2 2 0 00-2 2v8a2 2 0 002 2z"></path></svg>
            <span class="btn-text">Copy</span>
        </button>
    </div>
    <pre style="margin:0 !important; border:none !important; border-radius:0 !important; padding:1rem !important; background:#0f172a !important; color:#34d399 !important;" class="overflow-x-auto text-xs font-mono leading-relaxed"><code id="byo-policy-code" style="background:transparent !important; color:inherit !important; padding:0 !important;">{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "S3InventoryReadAccess",
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::YOUR_INVENTORY_BUCKET",
        "arn:aws:s3:::YOUR_INVENTORY_BUCKET/*"
      ]
    },
    {
      "Sid": "CloudWatchMetrics",
      "Effect": "Allow",
      "Action": [
        "cloudwatch:PutMetricData"
      ],
      "Resource": "*"
    },
    {
      "Sid": "CloudWatchLogs",
      "Effect": "Allow",
      "Action": [
        "logs:CreateLogStream",
        "logs:PutLogEvents",
        "logs:FilterLogEvents",
        "logs:StartQuery",
        "logs:GetQueryResults",
        "logs:DescribeLogStreams",
        "logs:GetLogEvents"
      ],
      "Resource": "arn:aws:logs:*:*:log-group:/aws/lambda/s3lim-*"
    },
    {
      "Sid": "SQSDLQAccess",
      "Effect": "Allow",
      "Action": [
        "sqs:SendMessage",
        "sqs:ReceiveMessage",
        "sqs:DeleteMessage",
        "sqs:GetQueueAttributes"
      ],
      "Resource": "arn:aws:sqs:*:*:s3lim-*"
    },
    {
      "Sid": "MarketplaceMetering",
      "Effect": "Allow",
      "Action": [
        "aws-marketplace:BatchMeterUsage",
        "aws-marketplace:GetEntitlements"
      ],
      "Resource": "*"
    }
  ]
}</code></pre>
</div>

### 3. Optional KMS Key Policy

If your inventory destination bucket or manifests are encrypted with SSE-KMS using a Customer Managed Key (CMK), grant decryption permissions on that specific key ARN:

<div class="not-prose my-4 rounded-xl overflow-hidden border border-slate-800 bg-slate-950 shadow-md">
    <div class="flex items-center justify-between px-4 py-2.5 bg-slate-800 text-slate-200 text-xs font-mono border-b border-slate-700/80">
        <span class="font-medium text-slate-300">kms-decrypt-policy.json</span>
        <button type="button" id="copy-kms-btn" onclick="copyCode('byo-kms-code', 'copy-kms-btn')" 
                class="flex items-center gap-1.5 bg-slate-700 hover:bg-slate-600 text-white px-2.5 py-1 rounded-md transition-colors text-xs font-sans">
            <svg class="w-3.5 h-3.5" fill="none" stroke="currentColor" stroke-width="2" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" d="M8 16H6a2 2 0 01-2-2V6a2 2 0 012-2h8a2 2 0 012 2v2m-6 12h8a2 2 0 002-2v-8a2 2 0 00-2-2h-8a2 2 0 00-2 2v8a2 2 0 002 2z"></path></svg>
            <span class="btn-text">Copy</span>
        </button>
    </div>
    <pre style="margin:0 !important; border:none !important; border-radius:0 !important; padding:1rem !important; background:#0f172a !important; color:#34d399 !important;" class="overflow-x-auto text-xs font-mono leading-relaxed"><code id="byo-kms-code" style="background:transparent !important; color:inherit !important; padding:0 !important;">{
  "Sid": "KmsDecryptAccess",
  "Effect": "Allow",
  "Action": [
    "kms:Decrypt",
    "kms:DescribeKey"
  ],
  "Resource": "arn:aws:kms:REGION:ACCOUNT_ID:key/KEY_ID"
}</code></pre>
</div>

<script>
function updateByoPolicies() {
    var rawInput = document.getElementById("byo-bucket-input").value.trim();
    // Strip s3:// prefix and trailing slashes if customer pastes full URI
    var bucketName = rawInput.replace(/^s3:\/\//, '').replace(/\/.*$/, '').trim();
    var bucketPlaceholder = bucketName || "YOUR_INVENTORY_BUCKET";
    
    var policyCodeEl = document.getElementById("byo-policy-code");
    if (policyCodeEl) {
        var policy = {
          "Version": "2012-10-17",
          "Statement": [
            {
              "Sid": "S3InventoryReadAccess",
              "Effect": "Allow",
              "Action": [
                "s3:GetObject",
                "s3:ListBucket"
              ],
              "Resource": [
                "arn:aws:s3:::" + bucketPlaceholder,
                "arn:aws:s3:::" + bucketPlaceholder + "/*"
              ]
            },
            {
              "Sid": "CloudWatchMetrics",
              "Effect": "Allow",
              "Action": [
                "cloudwatch:PutMetricData"
              ],
              "Resource": "*"
            },
            {
              "Sid": "CloudWatchLogs",
              "Effect": "Allow",
              "Action": [
                "logs:CreateLogStream",
                "logs:PutLogEvents",
                "logs:FilterLogEvents",
                "logs:StartQuery",
                "logs:GetQueryResults",
                "logs:DescribeLogStreams",
                "logs:GetLogEvents"
              ],
              "Resource": "arn:aws:logs:*:*:log-group:/aws/lambda/s3lim-*"
            },
            {
              "Sid": "SQSDLQAccess",
              "Effect": "Allow",
              "Action": [
                "sqs:SendMessage",
                "sqs:ReceiveMessage",
                "sqs:DeleteMessage",
                "sqs:GetQueueAttributes"
              ],
              "Resource": "arn:aws:sqs:*:*:s3lim-*"
            },
            {
              "Sid": "MarketplaceMetering",
              "Effect": "Allow",
              "Action": [
                "aws-marketplace:BatchMeterUsage",
                "aws-marketplace:GetEntitlements"
              ],
              "Resource": "*"
            }
          ]
        };
        policyCodeEl.textContent = JSON.stringify(policy, null, 2);
    }
}

function copyCode(elementId, buttonId) {
    var el = document.getElementById(elementId);
    if (!el) return;
    var text = el.textContent || el.innerText;
    navigator.clipboard.writeText(text).then(function() {
        var btn = document.getElementById(buttonId);
        if (btn) {
            var textSpan = btn.querySelector('.btn-text');
            var originalText = textSpan ? textSpan.textContent : "Copy";
            if (textSpan) textSpan.textContent = "Copied!";
            btn.classList.add("bg-emerald-600", "text-white");
            btn.classList.remove("bg-slate-700");
            setTimeout(function() {
                if (textSpan) textSpan.textContent = originalText;
                btn.classList.remove("bg-emerald-600");
                btn.classList.add("bg-slate-700");
            }, 2000);
        }
    });
}
</script>

### Least-Privilege Scoping & Wildcard Rationale

All permissions follow strict **least-privilege scoping** with wildcards minimized. For enterprise security audits, each wildcard (`*`) is used only where mandated by AWS IAM APIs:

* **`arn:aws:s3:::<bucket>/*` (S3 Object Keys)**:
  * **Why `*` is required**: Bucket ARNs (`arn:aws:s3:::<bucket>`) only apply to bucket actions like `s3:ListBucket`. S3 requires object ARNs (`arn:aws:s3:::<bucket>/*`) for object-level read actions like `s3:GetObject` to read inventory manifests and data files across sub-prefixes.
* **`cloudwatch:PutMetricData` (`Resource: "*"`)**:
  * **Why `*` is required**: AWS CloudWatch does not support resource-level permissions (ARNs) for the `PutMetricData` API. Per AWS IAM authorization specifications, CloudWatch metric publishing requires `"Resource": "*"`.
* **`aws-marketplace:BatchMeterUsage` / `GetEntitlements` (`Resource: "*"`)**:
  * **Why `*` is required**: AWS Marketplace Metering APIs do not support resource-level ARNs. AWS requires `"Resource": "*"` to report usage dimensions.
* **`arn:aws:logs:*:*:log-group:/aws/lambda/s3lim-*` & `arn:aws:sqs:*:*:s3lim-*`**:
  * **Why region/account `*` is used**: Scopes resource access strictly to `s3lim` prefixes while allowing multi-region or StackSet operation. You can optionally replace `*:*` with your explicit AWS Region and Account ID (e.g. `arn:aws:logs:us-east-1:123456789012:log-group:/aws/lambda/s3lim-*`).

---

## Configure Analysis Trigger
By default, `s3lim` sets up a **daily scheduled trigger (cron)** to scan the destination bucket for new manifests.

If you want real-time analysis immediately upon report delivery:
1. Copy the `CoreFunctionArn` from the CloudFormation stack outputs.
2. Navigate to your S3 Inventory destination bucket in the AWS Console.
3. Go to the **Properties** tab and scroll to **Event notifications**.
4. Click **Create event notification** and configure:
   * **Prefix**: The folder path of your inventory reports (e.g., `inventory/`).
   * **Suffix**: `manifest.json`.
   * **Event types**: `All object create events` (`s3:ObjectCreated:*`).
   * **Destination**: Lambda Function (select your `s3lim` function).

## Verification
Once a new S3 Inventory report is delivered, the Lambda will automatically execute. You can view the output analysis and metrics in your AWS CloudWatch dashboard.
