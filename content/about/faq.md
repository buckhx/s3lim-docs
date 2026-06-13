---
title: "Frequently Asked Questions"
description: "Answers to common questions about s3lim."
faq_items:
  - question: "What is s3lim?"
    answer: "s3lim is a high-performance analysis tool for object storage with specific suport for Amazon S3. It processes large inventory reports to provide actionable metrics on storage usage, allowing you to optimize costs and identify inefficiencies."
  - question: "How does it work?"
    answer: "s3lim utilizes a high performance analysis engine to aggregate data on storage classes, object sizes, duplicate files, and more. It can run locally on your machine or be deployed as an AWS Lambda function."
  - question: "Are there limits?"
    answer: "A single lambda can safely process 100M objects and up to 1B when tuned properly. Bucket analysis of over 1B objects requires concurrent Lambda execution which is a more complex deployment than offered in the marketplace. Contact support with your use case to get set up at enterprise scale."
  - question: "Is my data secure?"
    answer: "Yes. s3lim analyzes the metadata provided in S3 Inventory reports, not the actual contents of your files. When deployed via our SAM templates, it runs entirely within your own AWS account. No inventory data is sent to external servers."
  - question: "How much does it cost?"
    answer: "When deploying s3lim via the AWS Serverless Application Repository, you are billed directly through your AWS account based on the largest number of objects processed in an analysis that month ($0.20 per million objects)."
  - question: "Do I need to be an AWS expert to use this?"
    answer: "Not at all. We provide an 'Autopilot' deployment template that sets up everything you need with a single click, automatically configuring the necessary buckets, triggers, and permissions."
  - question: "What about other cloud providers like GCP or Azure"
    answer: "The s3lim core engine has no technical issues in supporting Google Cloud Storage or Azure Blob Storage. Customer demand will drive support for other clouds, so if you are interested in abslim or gcslim variants be sure to make a request."
---

# FAQ

Have a question about s3lim? Check below for answers to common technical and security inquiries.
