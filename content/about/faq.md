---
title: "Frequently Asked Questions"
description: "Answers to common questions about s3lim."
faq_items:
  - question: "What is s3lim?"
    answer: "s3lim is a high-performance analysis tool for Amazon S3 inventories. It processes large inventory reports to provide actionable metrics on storage usage, allowing you to optimize costs and identify inefficiencies."
  - question: "How does it work?"
    answer: "s3lim utilizes optimized Go routines to parse your S3 Inventory CSV/Parquet files directly, aggregating data on storage classes, object sizes, duplicate files, and more. It can run locally on your machine or be deployed as an AWS Lambda function."
  - question: "Is my data secure?"
    answer: "Yes. s3lim analyzes the metadata provided in S3 Inventory reports, not the actual contents of your files. When deployed via our SAM templates, it runs entirely within your own AWS account. No inventory data is sent to external servers."
  - question: "How much does it cost?"
    answer: "The core CLI and engine are free and open source. If you choose to deploy s3lim via the AWS Serverless Application Repository, you are billed directly through your AWS account based on the volume of objects analyzed ($0.20 per million objects)."
  - question: "Do I need to be an AWS expert to use this?"
    answer: "Not at all. We provide an 'Autopilot' deployment template that sets up everything you need with a single click, automatically configuring the necessary buckets, triggers, and permissions."
---

# FAQ

Have a question about s3lim? Check below for answers to common technical and security inquiries.
