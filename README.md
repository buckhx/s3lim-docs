# s3lim Documentation

This directory contains the source code for the `s3lim` documentation site, built using [Hugo](https://gohugo.io/).

## Structure

- `content/`: Markdown files for the site.
  - `cli/`: Auto-generated documentation for CLI commands.
  - `deployment/`: Auto-generated documentation for SAM templates.
  - `metrics/`: Auto-generated documentation for Aggregator metrics.
  - `getting-started/`: Manual onboarding guides.
- `themes/`: Hugo themes (managed via git submodules).

## Development

### Auto-Generation
Core technical documentation is auto-generated from the Go source code in the main repository. To refresh these files, run the following command from the **root of the s3lim repository**:

```bash
make docs
```

This runs the script at `scripts/gen-docs/main.go` which populates the `content/cli/`, `content/deployment/`, and `content/metrics/` directories.

### Local Preview
To serve the documentation locally with live reloading:

```bash
# From the docs/ directory
hugo server
```

The site will be available at `http://localhost:1313`.

## Deployment
The documentation is automatically published to GitHub Pages when changes are pushed to the `main` branch of the `s3lim-docs` repository (the remote for this submodule).
