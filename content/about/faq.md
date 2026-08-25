---
title: "Frequently Asked Questions"
description: "Answers to common questions about s3lim."
faq_items:
  - question: "What is s3lim?"
    answer: "s3lim is a high-performance analysis tool for object storage with specific suport for Amazon S3. It processes large inventory reports to provide actionable metrics on storage usage, allowing you to optimize costs and identify inefficiencies."
  - question: "How does it work?"
    answer: "s3lim utilizes a high-performance analysis engine to aggregate data on storage classes, object sizes, duplicate files, and more. It deploys directly into your AWS account via AWS Marketplace."
  - question: "Are there limits?"
    answer: "In Lite Mode, a single Lambda execution safely processes up to 100 million objects. For multi-billion object enterprise data lakes, Distributed Mode uses AWS Step Functions Distributed Map to scale horizontally across concurrent Worker Lambdas."
  - question: "Is my data secure?"
    answer: "Yes. s3lim analyzes the metadata provided in S3 Inventory reports, not the actual contents of your files. When deployed via our SAM templates, it runs entirely within your own AWS account. No inventory data is sent to external servers."
  - question: "How much does it cost?"
    answer: "s3lim is billed directly through AWS Marketplace at $0.15 per million objects scanned and $0.05 per custom prefix tracked per month. The first 1,000,000 objects per bucket each month are completely free."
  - question: "How does monthly billing work?"
    answer: "Billing uses a Monthly High-Water Mark (HWM) per bucket. You only pay for the peak volume scanned in a calendar month. Running automated daily scans does not bill you 30 times. If your bucket starts at 2M objects and grows by 500k mid-month, you only pay for the net new 1M boundary crossed. On the 1st of each month, the baseline resets."
  - question: "Do I need to be an AWS expert to use this?"
    answer: "Not at all. We provide an 'Autopilot' deployment template that sets up everything you need with a single click, automatically configuring the necessary buckets, triggers, and permissions."
  - question: "What about other cloud providers like GCP or Azure"
    answer: "The s3lim core engine has no technical issues in supporting Google Cloud Storage or Azure Blob Storage. Customer demand will drive support for other clouds, so if you are interested in abslim or gcslim variants be sure to make a request."
---

# FAQ

Have a question about s3lim? Check below for answers to common technical and security inquiries.
