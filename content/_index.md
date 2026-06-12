---
# banner
banner:
  title: "High-Performance S3 Inventory Analysis"
  button: "Get Started"
  button_link: "/getting-started/autopilot/"
  image: "images/banner-app.png"

# features
features:
  enable: true
  subtitle: "Core Capabilities"
  title: "Performance at <br> Scale"
  description: "s3lim is designed to handle billions of S3 objects with constant memory overhead, <br> providing actionable cost-optimization insights in minutes."
  features_blocks:
    - icon: "las la-bolt"
      title: "Blazing Fast"
      content: "Process massive inventory files (CSV, Parquet, ORC) using a high-concurrency Go engine optimized for S3 throughput."
    - icon: "las la-memory"
      title: "Bounded Memory"
      content: "Utilizes probabilistic sketches (Heavy Keepers, HLL) to ensure analysis doesn't scale with your data size."
    - icon: "las la-cloud-upload-alt"
      title: "Cloud Native"
      content: "Deploy as a containerized Lambda function with automated SAM templates for effortless serverless scaling."
    - icon: "las la-chart-bar"
      title: "Cost Insights"
      content: "Identify delete marker bloat, ghost versions, and multipart upload traps that inflate your S3 bill."
    - icon: "las la-terminal"
      title: "Powerful CLI"
      content: "A rich command-line interface for ad-hoc analysis, local testing, and flexible output formatting."
    - icon: "las la-project-diagram"
      title: "Extensible"
      content: "Modular architecture allows for custom aggregators and reporters to fit any storage auditing workflow."

# intro_video
intro_video:
  enable: false

# how_it_works
how_it_works:   
  enable: true
  title: "How it Works"
  block:
  - subtitle: "Analyze"
    title: "Deep Prefix Visibility"
    description: "Recursive prefix aggregation reveals exactly where your data is growing. Identify 'hot' prefixes and candidates for Intelligent Tiering or Lifecycle policies with surgical precision."
    image: "images/features-01.png"

  - subtitle: "Optimize"
    title: "Actionable Cost Reduction"
    description: "Surface hidden storage costs. s3lim detects incomplete multipart uploads and non-current versions that traditional tools miss, enabling immediate savings."
    image: "images/features-02.png"

# testimonials
testimonials:
  enable: false
---
