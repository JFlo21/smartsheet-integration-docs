# Repository Overview

This section provides detailed documentation for each of the 6 Smartsheet integration repositories in the suite.

## Repository Comparison

| Repository | Language | Type | Complexity | Best For |
|-----------|----------|------|------------|----------|
| [Supabase Smartsheet Promax Offload](supabase-smartsheet-promax-offload.md) | Python | Data Sync | Medium | Database → Sheet sync |
| [Smartsheet Supabase Sync](smartsheet-supabase-sync.md) | TypeScript | Data Sync | Medium | Sheet → Database sync |
| [Master to Sibling](master-to-sibling-smartsheet-function.md) | Python | Replication | Low | Sheet → Sheet copy |
| [Generate Job Numbers](generate-job-numbers.md) | Python | Automation | Low | Auto-numbering |
| [Generate Weekly PDFs](generate-weekly-pdfs-dsr-resiliency.md) | Python | Reporting | High | PDF generation |
| [Resiliency PDF Restructure](resiliency-pdf-restructure-ug-work.md) | Python | Validation | Medium | PDF validation |

## By Use Case

### Need to sync data between Smartsheet and a database?

<div class="card-grid">
  <div class="card">
    <h3>Smartsheet → Supabase</h3>
    <p><strong><a href="smartsheet-supabase-sync/">Smartsheet Supabase Sync</a></strong></p>
    <p>TypeScript, GitHub Actions, scheduled runs</p>
    <ul>
      <li>Reads from Smartsheet</li>
      <li>Writes to Supabase PostgreSQL</li>
      <li>Runs on schedule via GitHub Actions</li>
    </ul>
  </div>
  
  <div class="card">
    <h3>Supabase → Smartsheet</h3>
    <p><strong><a href="supabase-smartsheet-promax-offload/">Supabase Smartsheet Promax Offload</a></strong></p>
    <p>Python, manual/scheduled execution</p>
    <ul>
      <li>Queries Supabase database</li>
      <li>Bulk upserts to Smartsheet</li>
      <li>Handles large datasets</li>
    </ul>
  </div>
</div>

### Need to replicate data between Smartsheet sheets?

<div class="card">
  <h3>Master-Sibling Pattern</h3>
  <p><strong><a href="master-to-sibling-smartsheet-function/">Master to Sibling Smartsheet Function</a></strong></p>
  <p>Python, on-demand execution</p>
  <ul>
    <li>Copy from one master sheet</li>
    <li>Replicate to multiple sibling sheets</li>
    <li>Maintains data consistency</li>
  </ul>
</div>

### Need to automate job number assignment?

<div class="card">
  <h3>Job Numbering</h3>
  <p><strong><a href="generate-job-numbers/">Generate Job Numbers</a></strong></p>
  <p>Python, webhook/manual trigger</p>
  <ul>
    <li>Auto-assigns sequential job numbers</li>
    <li>Prevents duplicates</li>
    <li>Configurable starting number</li>
  </ul>
</div>

### Need to generate weekly PDF reports?

<div class="card">
  <h3>PDF Reporting</h3>
  <p><strong><a href="generate-weekly-pdfs-dsr-resiliency/">Generate Weekly PDFs DSR Resiliency</a></strong></p>
  <p>Python, weekly cron job</p>
  <ul>
    <li>Merges ProMax and Excel data</li>
    <li>Generates PDF per foreman</li>
    <li>Updates Smartsheet with links</li>
  </ul>
</div>

### Need to validate PDF structure?

<div class="card">
  <h3>PDF Validation</h3>
  <p><strong><a href="resiliency-pdf-restructure-ug-work/">Resiliency PDF Restructure UG Work</a></strong></p>
  <p>Python, post-generation execution</p>
  <ul>
    <li>Validates PDF structure</li>
    <li>Extracts CU codes</li>
    <li>Updates validation status</li>
  </ul>
</div>

## Technology Breakdown

### Python Repositories (5)

Python is the primary language for most integrations, chosen for:
- Rich ecosystem of data processing libraries
- Excellent Smartsheet SDK
- Easy scripting and automation
- Strong PDF manipulation capabilities

| Repository | Python Version | Key Libraries |
|-----------|----------------|---------------|
| Supabase Smartsheet Promax Offload | 3.9+ | smartsheet-python-sdk, supabase-py |
| Master to Sibling | 3.9+ | smartsheet-python-sdk |
| Generate Job Numbers | 3.9+ | smartsheet-python-sdk |
| Generate Weekly PDFs | 3.9+ | smartsheet-python-sdk, pandas, openpyxl, reportlab |
| Resiliency PDF Restructure | 3.9+ | smartsheet-python-sdk, PyPDF2, pdfplumber |

### TypeScript Repository (1)

TypeScript powers the GitHub Actions workflow:

| Repository | Node Version | Key Libraries |
|-----------|--------------|---------------|
| Smartsheet Supabase Sync | 16+ | @supabase/supabase-js, axios, dotenv |

## Deployment Patterns

### GitHub Actions (Serverless)
- **[Smartsheet Supabase Sync](smartsheet-supabase-sync.md)**
- No infrastructure needed
- Runs on schedule
- Uses GitHub secrets

### Cron Jobs (Server)
- **[Generate Weekly PDFs](generate-weekly-pdfs-dsr-resiliency.md)**
- **[Resiliency PDF Restructure](resiliency-pdf-restructure-ug-work.md)**
- Scheduled execution
- Requires server/VM
- Uses systemd or crontab

### On-Demand (Manual/Webhook)
- **[Supabase Smartsheet Promax Offload](supabase-smartsheet-promax-offload.md)**
- **[Master to Sibling](master-to-sibling-smartsheet-function.md)**
- **[Generate Job Numbers](generate-job-numbers.md)**
- Triggered by events
- Can be manual or automated
- Flexible execution

## Repository Details

Explore each repository for comprehensive documentation:

### Data Synchronization
1. **[Supabase Smartsheet Promax Offload](supabase-smartsheet-promax-offload.md)**
   - Database to Smartsheet sync
   - Bulk operations
   - Python-based

2. **[Smartsheet Supabase Sync](smartsheet-supabase-sync.md)**
   - Smartsheet to database sync
   - GitHub Actions workflow
   - TypeScript-based

### Data Replication
3. **[Master to Sibling Smartsheet Function](master-to-sibling-smartsheet-function.md)**
   - Sheet-to-sheet replication
   - Master-sibling pattern
   - Python-based

### Automation
4. **[Generate Job Numbers](generate-job-numbers.md)**
   - Automated job numbering
   - Duplicate prevention
   - Python-based

### Reporting & Validation
5. **[Generate Weekly PDFs DSR Resiliency](generate-weekly-pdfs-dsr-resiliency.md)**
   - Weekly PDF generation
   - ProMax and Excel integration
   - Python-based

6. **[Resiliency PDF Restructure UG Work](resiliency-pdf-restructure-ug-work.md)**
   - PDF structure validation
   - CU code extraction
   - Python-based

## Common Patterns

All repositories share these characteristics:

### Configuration
- Environment variables via `.env` files
- Separate configs for dev/staging/production
- Secure API token management

### Error Handling
- Retry logic for API failures
- Comprehensive logging
- Graceful failure modes

### Scalability
- Batch processing for large datasets
- Rate limit awareness
- Efficient API usage

### Maintainability
- Clear code structure
- Documentation
- Example configurations

## Getting Started

1. **Choose Your Repository**: Based on your use case above
2. **Read the Docs**: Click through to detailed repository pages
3. **Follow Setup**: Each page has setup instructions
4. **Test First**: Use dry-run mode before production
5. **Monitor**: Set up logging and monitoring

## Quick Links

- [Master Index](../master-index.md) - Complete catalog
- [Usage Guide](../usage-guide.md) - Setup instructions
- [Smartsheet Integration](../smartsheet-integration.md) - Data flows
- [Troubleshooting](../troubleshooting.md) - Common issues

---

<div style="text-align: center; margin-top: 2rem; padding: 1.5rem; background: var(--md-code-bg-color); border-radius: 8px;">
  <p><strong>Ready to dive deeper?</strong></p>
  <p>Select a repository above to see detailed documentation, code examples, and configuration guides.</p>
</div>
