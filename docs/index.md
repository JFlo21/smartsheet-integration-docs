# Welcome to the Smartsheet Integration Suite

<div class="card-grid">
  <div class="card">
    <h3>📚 Documentation Hub</h3>
    <p>Central documentation for all Smartsheet integration repositories, providing comprehensive guides and references.</p>
  </div>
  <div class="card">
    <h3>🔄 Automated Workflows</h3>
    <p>Seamlessly sync data between Smartsheet, Supabase, and Excel with robust Python and TypeScript integrations.</p>
  </div>
  <div class="card">
    <h3>🛠️ Production Ready</h3>
    <p>Battle-tested tools for job number generation, PDF reports, and data validation in production environments.</p>
  </div>
</div>

## Overview

The Smartsheet Integration Suite is a collection of Python and TypeScript applications designed to automate data synchronization, reporting, and validation workflows for Linetec's business operations. These tools connect Smartsheet with various systems including Supabase databases, ProMax ERP, and custom reporting solutions.

## Technology Stack

![Python](https://img.shields.io/badge/Python-3.9+-3776ab?style=for-the-badge&logo=python&logoColor=white)
![TypeScript](https://img.shields.io/badge/TypeScript-4.0+-3178c6?style=for-the-badge&logo=typescript&logoColor=white)
![Smartsheet](https://img.shields.io/badge/Smartsheet-API-1a1a1a?style=for-the-badge&logo=smartsheet&logoColor=white)
![Supabase](https://img.shields.io/badge/Supabase-PostgreSQL-3ecf8e?style=for-the-badge&logo=supabase&logoColor=white)
![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-CI/CD-2088FF?style=for-the-badge&logo=github-actions&logoColor=white)

## Architecture Overview

The integration suite follows a distributed architecture where each component focuses on a specific business function:

```mermaid
graph TB
    subgraph "Data Sources"
        SS[Smartsheet Sheets]
        PM[ProMax ERP]
        EX[Excel Files]
    end
    
    subgraph "Integration Layer"
        SSS[Smartsheet-Supabase Sync]
        SPS[Supabase-Smartsheet Offload]
        MSS[Master-to-Sibling Sync]
        GJN[Job Number Generator]
        GPF[Weekly PDF Generator]
        RPR[PDF Restructure Validator]
    end
    
    subgraph "Data Storage"
        SB[(Supabase Database)]
        FS[File Storage]
    end
    
    subgraph "Output & Reports"
        PDF[PDF Reports]
        REC[Updated Records]
    end
    
    SS -->|Read/Write| SSS
    SSS -->|Sync| SB
    SB -->|Offload| SPS
    SPS -->|Write| SS
    SS -->|Master Data| MSS
    MSS -->|Replicate| SS
    SS -->|Job Requests| GJN
    GJN -->|Assign Numbers| SS
    PM -->|Weekly Data| GPF
    GPF -->|Generate| PDF
    PDF -->|Validate| RPR
    RPR -->|Update| SS
    EX -->|Import| GPF
    PDF -->|Store| FS
    REC -->|Update| SS
    
    style SS fill:#f9f,stroke:#333,stroke-width:2px
    style SB fill:#3ecf8e,stroke:#333,stroke-width:2px
    style PDF fill:#ff9,stroke:#333,stroke-width:2px
```

## Quick Start

Ready to get started? Here's what you need to know:

1. **[Master Index](master-index.md)** - Complete catalog of all 6 repositories
2. **[Usage Guide](usage-guide.md)** - Step-by-step setup and configuration
3. **[Smartsheet Integration](smartsheet-integration.md)** - Data flows and mappings
4. **[Watch Out For](watch-out-for.md)** - Critical pitfalls to avoid

## Key Features

<div class="feature-list">

- **Bidirectional Sync**: Real-time data synchronization between Smartsheet and Supabase
- **Automated Job Numbering**: Intelligent job number assignment with conflict prevention
- **Weekly Reporting**: Automated PDF generation from ProMax and Excel data
- **Data Validation**: PDF structure validation and CU code verification
- **Master-Sibling Replication**: Automated sheet-to-sheet data copying
- **GitHub Actions Integration**: Scheduled workflows for hands-free operation
- **Error Handling**: Robust retry logic and comprehensive logging
- **Environment Isolation**: Separate configurations for development and production

</div>

## Repository Ecosystem

The suite consists of 6 specialized repositories, each handling a specific aspect of the integration workflow:

| Repository | Language | Primary Function | Status |
|-----------|----------|------------------|--------|
| [Supabase Smartsheet Promax Offload](repositories/supabase-smartsheet-promax-offload.md) | Python | Sync from Supabase to Smartsheet | <span class="status-badge status-active">Active</span> |
| [Smartsheet Supabase Sync](repositories/smartsheet-supabase-sync.md) | TypeScript | Sync from Smartsheet to Supabase | <span class="status-badge status-active">Active</span> |
| [Master to Sibling Function](repositories/master-to-sibling-smartsheet-function.md) | Python | Sheet-to-sheet replication | <span class="status-badge status-active">Active</span> |
| [Generate Job Numbers](repositories/generate-job-numbers.md) | Python | Automated job numbering | <span class="status-badge status-active">Active</span> |
| [Generate Weekly PDFs](repositories/generate-weekly-pdfs-dsr-resiliency.md) | Python | Weekly PDF reports | <span class="status-badge status-active">Active</span> |
| [Resiliency PDF Restructure](repositories/resiliency-pdf-restructure-ug-work.md) | Python | PDF validation | <span class="status-badge status-active">Active</span> |

## Getting Help

!!! tip "Need Help?"
    - Check the **[Troubleshooting Guide](troubleshooting.md)** for common issues
    - Review **[Watch Out For](watch-out-for.md)** for critical warnings
    - See **[Maintenance Guide](maintenance-guide.md)** for update procedures

## Contributing

This documentation is maintained alongside the integration repositories. To suggest improvements:

1. Visit the [documentation repository](https://github.com/JFlo21/smartsheet-integration-docs)
2. Open an issue or submit a pull request
3. All pages have an "Edit this page" link in the top right

---

<div style="text-align: center; margin-top: 3rem; color: var(--md-default-fg-color--light);">
  <p><strong>Smartsheet Integration Suite</strong></p>
  <p>Built with ❤️ for efficient business operations</p>
</div>
