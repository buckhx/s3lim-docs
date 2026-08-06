---
title: "Security Policy"
description: "Security Policy and Vulnerability Disclosure for s3lim and Slim Storage Solutions LLC."
hide_sidebar: true
---

<div class="text-left max-w-3xl mx-auto py-8">

# Security Policy
*Last Updated: August 5, 2026*

We are committed to ensuring the security and privacy of your data when using **s3lim**. If you believe you have discovered a security vulnerability or have concerns regarding our software's security posture, please follow our disclosure policy below.

## 1. Reporting Vulnerabilities
Please do **not** report security vulnerabilities through public GitHub issues or forums. Instead, report all findings directly to our security response team:

*   **Email**: [support+security@slimstorage.io](mailto:support+security@slimstorage.io)

To help us investigate and remediate the issue, please include:
*   A clear description of the vulnerability and its potential impact.
*   Step-by-step instructions to reproduce the issue (and any proof-of-concept scripts/logs).
*   Details about your environment (e.g. deployment region, SAM template type, Lambda runtime parameters).

## 2. Response Timeline
*   **Acknowledgment**: We will acknowledge receipt of your report within **24 hours**.
*   **Evaluation**: We will provide a status update and initial evaluation within **72 hours**.
*   **Remediation**: We will coordinate the release of a security patch or mitigation guidelines and notify all registered subscribers of the fix.

## 3. Zero-Egress Boundary
s3lim is built as an **in-account, zero-egress serverless application**. 
*   **No Data Exfiltration**: Your S3 inventory reports, file structures, analysis results, and raw object metadata are processed entirely within the boundaries of your own AWS account.
*   **No Vendor Access**: Slim Storage Solutions LLC has no technical mechanism to access or view your S3 storage metrics.
*   **Fulfillment Scope**: The only data transmitted to our control plane is the initial registration token and contact email submitted during subscription setup for licensing validation and billing integration.

</div>
