# s3lim Documentation

This directory contains the source code for the `s3lim` documentation site, built using [Hugo](https://gohugo.io/). The documentation is hosted at [slimstorage.io](https://slimstorage.io).

## Structure

- `content/`: Source markdown files built by Hugo.
  - `docs/getting-started/`: Flat list of onboarding and deployment setup guides.
  - `docs/reference/`: Flat list of technical reference documentation (configuration, deployment models, metrics).
  - `about/`: Core concepts, optimization guides (such as the Small File Trap), FAQ, and glossary.
- `data/`: Auto-generated specification files.
- `themes/`: Theme layouts and assets (specifically the custom `s3lim-minimal` theme).

## Development

### Auto-Generation
Core technical specifications are auto-generated from Go code and AWS SAM templates. To refresh these static reference files, run the following command from the **root of the s3lim repository**:

```bash
make docs
```

This runs the script at `scripts/gen-docs/main.go` and populates the `docs/data/` subdirectories (`cli/`, `deployment/`, and `metrics/`). These files serve as a reference for writing and editing documentation under `content/`.

### Local Preview
To serve the documentation locally with live reloading:

```bash
# From the docs/ directory
hugo server
```

The site will be available at `http://localhost:1313`.

## Editing and Authoring Content

### Editing Existing Pages
To modify existing documentation, edit the markdown files directly under `content/docs/getting-started/`, `content/docs/reference/`, or `content/about/`.

### Adding New Pages
To add a new documentation page:
1. Create a new markdown file in the appropriate directory (e.g., `content/docs/reference/new-feature.md`).
2. Add Hugo front matter at the top:
   ```yaml
   ---
   title: "New Feature Spec"
   description: "Detailed spec for the new feature"
   ---
   ```

### Updating the Navigation Sidebar
The documentation sidebar is explicitly structured for a premium, clean design. When adding a new page, it will not appear in the sidebar automatically. You must add it to the consolidated sidebar partial template.

Add your new page to the sidebar list in:
* `themes/s3lim-minimal/layouts/partials/sidebar.html`

Use the `{{ with .Site.GetPage ... }}` helper to dynamically resolve the page and manage active page highlighting. For example:

```html
{{ with .Site.GetPage "/docs/reference/new-feature.md" }}
<li><a href="{{ .RelPermalink }}" class="sidebar-link{{ if eq $.RelPermalink .RelPermalink }} active{{ end }}">{{ .Title }}</a></li>
{{ end }}
```

## Deployment
The documentation is automatically published to GitHub Pages when changes are pushed to the `main` branch of the `s3lim-docs` repository (the remote for this submodule).
