---
# banner
banner:
  title: "Securely Analyze S3 Costs Without Data Egress"
  button: "Start Saving Now"
  button_link: "/getting-started/autopilot/"

# features
features:
  enable: true
  subtitle: "Actionable Storage Insights"
  title: "Stop S3 Bill Bloat"
  description: "Analyze billions of objects directly in your AWS account. s3lim surfaces hidden costs and optimization opportunities without your data ever leaving your VPC."
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
      title: "Native AWS Integration"
      content: "Deploys directly as a Lambda function. Manage everything via the AWS Console with zero infrastructure to maintain."

# how_it_works
how_it_works:   
  enable: true
  title: "How it Works"
  block:
  - title: "Massive Scale, Tiny Footprint"
    description: "Built in high-performance Go, s3lim analyzes billions of objects in minutes using constant memory. It's engineered to scale to the world's largest S3 buckets without breaking a sweat."
  - title: "Serverless Efficiency"
    description: "Running entirely on AWS Lambda, s3lim provides a cost-effective, on-demand analysis engine that only runs when you need it. No servers to manage, no idle costs."
---
