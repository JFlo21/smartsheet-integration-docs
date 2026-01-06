# Smartsheet Integration Suite Documentation

![Documentation Status](https://img.shields.io/badge/docs-live-brightgreen)
![MkDocs](https://img.shields.io/badge/Built%20with-MkDocs%20Material-blue)
![License](https://img.shields.io/badge/license-MIT-blue)

> **Comprehensive documentation for all Smartsheet integration repositories**

📚 **Live Documentation**: [https://jflo21.github.io/smartsheet-integration-docs/](https://jflo21.github.io/smartsheet-integration-docs/)

## Overview

This repository hosts the centralized documentation for the Smartsheet Integration Suite - a collection of 6 Python and TypeScript applications that automate data synchronization, reporting, and validation workflows.

### Covered Repositories

1. **[Supabase Smartsheet Promax Offload](https://github.com/JFlo21/supabase-smartsheet-promax-offload)** - Database to Smartsheet sync
2. **[Smartsheet Supabase Sync](https://github.com/JFlo21/Smartsheet-supabase-sync)** - Smartsheet to database sync
3. **[Master to Sibling Smartsheet](https://github.com/JFlo21/master-to-sibling-smartsheet-function)** - Sheet-to-sheet replication
4. **[Generate Job Numbers](https://github.com/JFlo21/generate-job-numbers)** - Automated job numbering
5. **[Generate Weekly PDFs](https://github.com/JFlo21/Generate-Weekly-PDFs-DSR-Resiliency)** - Weekly PDF reports
6. **[Resiliency PDF Restructure](https://github.com/JFlo21/Resiliency-pdf-restructure-ug-work)** - PDF validation

## Features

- 📖 **Comprehensive Guides** - Setup, usage, and maintenance documentation
- 🏗️ **Architecture Diagrams** - Visual representation of data flows
- ⚠️ **Critical Warnings** - Watch out for common pitfalls
- 🔧 **Troubleshooting** - Solutions to common issues
- 🎨 **Beautiful Theme** - Material for MkDocs with dark/light mode
- 🔍 **Full-Text Search** - Find what you need quickly
- 📱 **Mobile Responsive** - Works on all devices

## Documentation Sections

- **[Home](https://jflo21.github.io/smartsheet-integration-docs/)** - Overview and quick start
- **[Master Index](https://jflo21.github.io/smartsheet-integration-docs/master-index/)** - Complete repository catalog
- **[Usage Guide](https://jflo21.github.io/smartsheet-integration-docs/usage-guide/)** - Setup and configuration
- **[Smartsheet Integration](https://jflo21.github.io/smartsheet-integration-docs/smartsheet-integration/)** - Data flows and mappings
- **[Watch Out For](https://jflo21.github.io/smartsheet-integration-docs/watch-out-for/)** - Critical pitfalls
- **[Maintenance Guide](https://jflo21.github.io/smartsheet-integration-docs/maintenance-guide/)** - Update procedures
- **[Troubleshooting](https://jflo21.github.io/smartsheet-integration-docs/troubleshooting/)** - Common issues
- **[Repositories](https://jflo21.github.io/smartsheet-integration-docs/repositories/)** - Detailed repository docs

## Local Development

### Prerequisites

- Python 3.9 or higher
- pip package manager

### Setup

1. **Clone the repository**
   ```bash
   git clone https://github.com/JFlo21/smartsheet-integration-docs.git
   cd smartsheet-integration-docs
   ```

2. **Install dependencies**
   ```bash
   pip install -r requirements.txt
   ```

3. **Serve locally**
   ```bash
   mkdocs serve
   ```

4. **Open in browser**
   ```
   http://127.0.0.1:8000
   ```

### Building

To build the static site:

```bash
mkdocs build
```

The site will be generated in the `site/` directory.

## Contributing

We welcome contributions! Here's how you can help:

### Editing Documentation

1. **Fork this repository**
2. **Create a branch** for your changes
3. **Edit markdown files** in the `docs/` directory
4. **Test locally** with `mkdocs serve`
5. **Submit a pull request**

### Quick Edits

Every page has an "Edit this page" button (✏️) in the top right that links directly to the source file on GitHub.

### Adding New Pages

1. Create a new `.md` file in the appropriate `docs/` subdirectory
2. Add the page to `nav` section in `mkdocs.yml`
3. Test the navigation works correctly

### Reporting Issues

Found an error or have a suggestion? [Open an issue](https://github.com/JFlo21/smartsheet-integration-docs/issues)!

## Project Structure

```
smartsheet-integration-docs/
├── .github/
│   └── workflows/
│       ├── deploy-pages.yml      # Deploys docs on push to main
│       └── sync-docs.yml         # Weekly sync from source repos
├── docs/
│   ├── index.md                  # Home page
│   ├── master-index.md           # Repository catalog
│   ├── usage-guide.md            # Setup guide
│   ├── smartsheet-integration.md # Integration details
│   ├── watch-out-for.md          # Critical warnings
│   ├── maintenance-guide.md      # Maintenance procedures
│   ├── troubleshooting.md        # Troubleshooting guide
│   ├── stylesheets/
│   │   └── extra.css             # Custom styles
│   └── repositories/
│       ├── index.md              # Repositories overview
│       └── *.md                  # Individual repo docs
├── mkdocs.yml                    # MkDocs configuration
├── requirements.txt              # Python dependencies
└── README.md                     # This file
```

## Technology Stack

- **[MkDocs](https://www.mkdocs.org/)** - Static site generator
- **[Material for MkDocs](https://squidfunk.github.io/mkdocs-material/)** - Beautiful theme
- **[GitHub Actions](https://github.com/features/actions)** - CI/CD automation
- **[GitHub Pages](https://pages.github.com/)** - Free hosting

## Deployment

The documentation is automatically deployed to GitHub Pages when changes are pushed to the `main` branch.

### Manual Deployment

To deploy manually:

```bash
mkdocs gh-deploy
```

This builds the site and pushes it to the `gh-pages` branch.

## License

MIT License - see [LICENSE](LICENSE) file for details.

## Support

- **Documentation Issues**: [Open an issue](https://github.com/JFlo21/smartsheet-integration-docs/issues)
- **Integration Issues**: Check the [Troubleshooting Guide](https://jflo21.github.io/smartsheet-integration-docs/troubleshooting/)
- **Questions**: Refer to individual repository documentation

---

<div align="center">
  <p>Built with ❤️ for efficient business operations</p>
  <p><strong><a href="https://jflo21.github.io/smartsheet-integration-docs/">📚 View Live Documentation</a></strong></p>
</div>