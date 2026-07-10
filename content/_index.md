---
# banner
banner:
  title: "Deep S3 Observability"
  button_text: "Get Early Access"
  button_link: "/early-access/"

# features
features:
  enable: true
  subtitle: "Actionable Storage Insights"
  title: "Stop S3 Bill Bloat"
  description: "Break down billions of objects into their top prefixes directly in your account. s3lim surfaces hidden costs and optimization opportunities without your data ever leaving your VPC."
  features_blocks:
    - icon: "🛡️"
      title: "Zero Egress Security"
      content: "Analyze your storage in-place. No data ever leaves your account, ensuring 100% compliance and zero data transfer costs."
    - icon: "👯"
      title: "Duplicate Detection"
      content: "Identify identical files across prefixes and buckets. Surface redundant data that is silently inflating your monthly bill."
    - icon: "🔥"
      title: "Prefix Hotspots"
      content: "Instantly find which folders are growing fastest. Pinpoint 'hot' storage areas that need Lifecycle or Intelligent Tiering policies."
    - icon: "👻"
      title: "Ghost Versions"
      content: "Find and remove non-current object versions and delete markers that add up to significant hidden storage overhead."
    - icon: "📦"
      title: "Unfinished Uploads"
      content: "Surface incomplete multipart uploads that occupy space and cost money, but are invisible to standard S3 management tools."
    - icon: "☁️"
      title: "Native to AWS"
      content: "Deploys directly as a Lambda function. Manage everything via the AWS Console and analyze in CloudWatch with zero infrastructure to maintain."

# how_it_works
how_it_works:   
  enable: true
  title: "How it Works"
  block:
  - title: "Massive Scale, Tiny Footprint"
    description: "Built wth novel methods, s3lim analyzes billions of objects in minutes using constant memory. It's engineered to scale to the world's largest S3 buckets without breaking a sweat."
  - title: "Serverless Efficiency"
    description: "Running entirely on AWS Lambda, s3lim provides a cost-effective, on-demand analysis engine that only runs when you need it. No servers to manage, no idle costs."

# pricing
pricing:
  enable: true
  title: "Simple, Volume-Based Pricing"
  subtitle: "Pay Only for What You Analyze"
  description: "s3lim is billed directly through your AWS account. Tiered pricing is available at specific volumes. No extra procurement, no hidden fees."
  plans:
    - title: "Core Analysis"
      price: "$0.015"
      unit: "per million objects scanned"
      features:
        - "Full S3 Inventory Scan"
        - "Top-K Prefix Rollups"
        - "In-Account Processing"
        - "CloudWatch Metrics & Dashboards"
    - title: "Custom Insights"
      price: "+$0.005"
      unit: "per custom prefix tracked"
      features:
        - "Custom Prefix Rollups"
        - "Advanced Filtering"
        - "Multi-Level Aggregation"
        - "Dedicated Reporting"

# roadmap
roadmap:
  enable: true
  title: "On the Horizon"
  subtitle: "S3 Performance & Security Auditing"
  description: "We are actively building stateless, in-account analysis engines for S3 Server Access Logs to provide real-time performance troubleshooting and active threat detection."
  features:
    - icon: "⚡"
      title: "Latency Diagnostics"
      content: "Identify slow S3 operations and surface p95/p99 turnaround latency hotspots mapped directly to prefixes."
    - icon: "🕵️"
      title: "Shadow Access Audit"
      content: "Detect requests coming from external AWS accounts, public IPs, or unexpected IAM principals."
    - icon: "🚨"
      title: "Bulk Egress & Ransomware Alerts"
      content: "Get notified of sudden exfiltration volume spikes or mass delete operations per client IP in near-real-time."
---
