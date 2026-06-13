---
title: "Frequently Asked Questions"
description: "Answers to common questions about s3lim."
---

# Frequently Asked Questions

## What is s3lim?
s3lim is a high-performance analysis tool for Amazon S3 inventories. It processes large inventory reports to provide actionable metrics on storage usage, allowing you to optimize costs and identify inefficiencies.

## How does it work?
s3lim utilizes optimized Go routines to parse your S3 Inventory CSV/Parquet files directly, aggregating data on storage classes, object sizes, duplicate files, and more. It can run locally on your machine or be deployed as an AWS Lambda function.

## Is my data secure?
**Yes.** s3lim analyzes the metadata provided in S3 Inventory reports, not the actual contents of your files. When deployed via our SAM templates, it runs entirely within your own AWS account. No inventory data is sent to external servers.

## How much does it cost?
The core CLI and engine are free and open source. If you choose to deploy s3lim via the AWS Serverless Application Repository, you are only responsible for the underlying AWS infrastructure costs (Lambda executions, minimal S3 storage).

## Do I need to be an AWS expert to use this?
Not at all. We provide an "Autopilot" deployment template that sets up everything you need with a single click, automatically configuring the necessary buckets, triggers, and permissions.
