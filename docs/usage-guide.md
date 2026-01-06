# Usage Guide

This comprehensive guide walks you through setting up and using all components of the Smartsheet Integration Suite.

## Prerequisites

Before setting up any integration, ensure you have the following:

### Required Accounts & Access

- [x] **Smartsheet Account** with API access enabled
- [x] **Smartsheet API Access Token** (Admin level recommended)
- [x] **Supabase Account** (for repositories using database sync)
- [x] **GitHub Account** (for GitHub Actions workflows)
- [x] **ProMax ERP Access** (for PDF generation workflows)

### Required Software

| Software | Version | Purpose | Installation |
|----------|---------|---------|--------------|
| **Python** | 3.9+ | Run Python-based integrations | [python.org](https://python.org) |
| **Node.js** | 16+ | Run TypeScript integrations | [nodejs.org](https://nodejs.org) |
| **Git** | Latest | Clone repositories | [git-scm.com](https://git-scm.com) |
| **pip** | Latest | Python package management | Included with Python |
| **npm** | Latest | Node package management | Included with Node.js |

### Optional Tools

- **Docker** - For containerized deployments
- **systemd** - For Linux service deployment
- **PM2** - For Node.js process management

## Environment Setup

### 1. Generate API Tokens

#### Smartsheet API Token

1. Log into Smartsheet
2. Click your profile icon → **Apps & Integrations**
3. Click **API Access**
4. Click **Generate new access token**
5. Name it (e.g., "Integration Suite Token")
6. Copy and save the token securely

!!! danger "Security Warning"
    Treat your API token like a password. Never commit it to version control or share it publicly.

#### Supabase Credentials

1. Log into your [Supabase Dashboard](https://app.supabase.com)
2. Select your project
3. Go to **Settings** → **API**
4. Copy the following:
   - Project URL (`SUPABASE_URL`)
   - `anon/public` key (`SUPABASE_ANON_KEY`) or `service_role` key (`SUPABASE_KEY`)

### 2. Identify Smartsheet Resources

Each integration requires specific Smartsheet IDs:

#### Finding Sheet IDs

1. Open the sheet in Smartsheet
2. Look at the URL: `https://app.smartsheet.com/sheets/[SHEET_ID]`
3. Copy the numeric ID

#### Finding Column IDs

**Method 1: Using the API**
```bash
curl -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  "https://api.smartsheet.com/2.0/sheets/SHEET_ID"
```

**Method 2: Using Python**
```python
import smartsheet
client = smartsheet.Smartsheet('YOUR_ACCESS_TOKEN')
sheet = client.Sheets.get_sheet(SHEET_ID)
for column in sheet.columns:
    print(f"{column.title}: {column.id}")
```

**Method 3: Browser Developer Tools**
1. Open sheet in browser
2. Open Developer Tools (F12)
3. Go to Network tab
4. Refresh the page
5. Find the API request to `smartsheet.com/2.0/sheets/`
6. View the response JSON for column IDs

## Repository-Specific Setup

### 1. Supabase Smartsheet Promax Offload

**Purpose**: Sync data from Supabase to Smartsheet

#### Clone & Install
```bash
git clone https://github.com/JFlo21/supabase-smartsheet-promax-offload.git
cd supabase-smartsheet-promax-offload
pip install -r requirements.txt
```

#### Configure Environment
Create a `.env` file:
```env
SMARTSHEET_ACCESS_TOKEN=your_token_here
SUPABASE_URL=https://your-project.supabase.co
SUPABASE_KEY=your_supabase_key
SHEET_ID=your_sheet_id
```

#### Run
```bash
python main.py
```

---

### 2. Smartsheet Supabase Sync

**Purpose**: Sync data from Smartsheet to Supabase (GitHub Actions)

#### Fork & Configure
1. Fork the repository to your GitHub account
2. Go to **Settings** → **Secrets and variables** → **Actions**
3. Add the following secrets:
   - `SMARTSHEET_ACCESS_TOKEN`
   - `SUPABASE_URL`
   - `SUPABASE_ANON_KEY` or `SUPABASE_KEY`

#### Local Development
```bash
git clone https://github.com/YOUR_USERNAME/Smartsheet-supabase-sync.git
cd Smartsheet-supabase-sync
npm install
```

Create `.env`:
```env
SMARTSHEET_ACCESS_TOKEN=your_token_here
SUPABASE_URL=https://your-project.supabase.co
SUPABASE_KEY=your_supabase_key
```

#### Run Locally
```bash
npm run start
# or
npm run dev
```

#### GitHub Actions Workflow
The workflow runs automatically on schedule. View `.github/workflows/sync.yml` for configuration.

---

### 3. Master to Sibling Smartsheet Function

**Purpose**: Copy data from master sheet to sibling sheets

#### Clone & Install
```bash
git clone https://github.com/JFlo21/master-to-sibling-smartsheet-function.git
cd master-to-sibling-smartsheet-function
pip install -r requirements.txt
```

#### Configure Environment
```env
SMARTSHEET_ACCESS_TOKEN=your_token_here
MASTER_SHEET_ID=master_sheet_id
SIBLING_SHEET_IDS=sibling1_id,sibling2_id,sibling3_id
COLUMN_ID_NAME=column_id_for_name
COLUMN_ID_VALUE=column_id_for_value
```

#### Run
```bash
python master_to_sibling.py
```

---

### 4. Generate Job Numbers

**Purpose**: Automatically assign job numbers to rows

#### Clone & Install
```bash
git clone https://github.com/JFlo21/generate-job-numbers.git
cd generate-job-numbers
pip install -r requirements.txt
```

#### Configure Environment
```env
SMARTSHEET_ACCESS_TOKEN=your_token_here
SHEET_ID=your_sheet_id
COLUMN_ID_JOB_NUMBER=column_id_for_job_numbers
STARTING_NUMBER=1000
```

#### Run
```bash
python generate_job_numbers.py
```

---

### 5. Generate Weekly PDFs DSR Resiliency

**Purpose**: Generate weekly PDF reports from ProMax and Excel data

#### Clone & Install
```bash
git clone https://github.com/JFlo21/Generate-Weekly-PDFs-DSR-Resiliency.git
cd Generate-Weekly-PDFs-DSR-Resiliency
pip install -r requirements.txt
```

#### Configure Environment
```env
SMARTSHEET_ACCESS_TOKEN=your_token_here
SHEET_ID=your_sheet_id
PROMAX_DATA_PATH=/path/to/promax/exports
EXCEL_FILE_PATH=/path/to/excel/data.xlsx
OUTPUT_PDF_DIR=/path/to/output/pdfs
```

#### Run
```bash
python generate_weekly_pdfs.py
```

#### Schedule with Cron (Linux)
```bash
# Run every Monday at 6 AM
0 6 * * 1 cd /path/to/repo && python generate_weekly_pdfs.py
```

---

### 6. Resiliency PDF Restructure UG Work

**Purpose**: Validate PDF structure and extract CU codes

#### Clone & Install
```bash
git clone https://github.com/JFlo21/Resiliency-pdf-restructure-ug-work.git
cd Resiliency-pdf-restructure-ug-work
pip install -r requirements.txt
```

#### Configure Environment
```env
SMARTSHEET_ACCESS_TOKEN=your_token_here
SHEET_ID=your_sheet_id
PDF_DIRECTORY=/path/to/pdfs
COLUMN_ID_CU_CODE=column_id_for_cu_codes
COLUMN_ID_VALIDATION=column_id_for_validation_status
```

#### Run
```bash
python pdf_restructure.py
```

---

## Deployment Options

### Option 1: Manual Execution

Run scripts manually when needed:
```bash
cd /path/to/repository
python script_name.py
```

### Option 2: Cron Jobs (Linux/Mac)

Schedule automatic execution:
```bash
# Edit crontab
crontab -e

# Add entries (examples)
0 */6 * * * cd /path/to/repo && python main.py  # Every 6 hours
0 8 * * 1 cd /path/to/weekly-pdfs && python generate_weekly_pdfs.py  # Monday 8 AM
```

### Option 3: systemd Service (Linux)

Create a service file `/etc/systemd/system/smartsheet-sync.service`:
```ini
[Unit]
Description=Smartsheet Integration Service
After=network.target

[Service]
Type=simple
User=your_user
WorkingDirectory=/path/to/repository
Environment="PATH=/usr/bin:/usr/local/bin"
ExecStart=/usr/bin/python3 /path/to/repository/main.py
Restart=on-failure
RestartSec=10

[Install]
WantedBy=multi-user.target
```

Enable and start:
```bash
sudo systemctl daemon-reload
sudo systemctl enable smartsheet-sync
sudo systemctl start smartsheet-sync
```

### Option 4: Docker Container

Create `Dockerfile`:
```dockerfile
FROM python:3.9-slim

WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .
CMD ["python", "main.py"]
```

Build and run:
```bash
docker build -t smartsheet-integration .
docker run -d --env-file .env smartsheet-integration
```

### Option 5: GitHub Actions

Already configured for `Smartsheet-supabase-sync`. For others, create `.github/workflows/schedule.yml`:
```yaml
name: Scheduled Sync
on:
  schedule:
    - cron: '0 */6 * * *'  # Every 6 hours
  workflow_dispatch:

jobs:
  sync:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: '3.9'
      - run: pip install -r requirements.txt
      - run: python main.py
        env:
          SMARTSHEET_ACCESS_TOKEN: ${{ secrets.SMARTSHEET_ACCESS_TOKEN }}
          SUPABASE_URL: ${{ secrets.SUPABASE_URL }}
          SUPABASE_KEY: ${{ secrets.SUPABASE_KEY }}
```

## Running the Applications

### Development Mode

Most Python scripts support verbose logging:
```bash
python main.py --verbose
# or
python main.py -v
```

### Production Mode

Use logging to files:
```bash
python main.py >> /var/log/smartsheet-sync.log 2>&1
```

### Testing Configuration

Most repositories support a dry-run mode:
```bash
python main.py --dry-run
# or
python main.py --test
```

!!! tip "Testing First"
    Always test with `--dry-run` or against a test sheet before running in production.

## Monitoring & Logs

### Log Locations

- **Manual execution**: Terminal output
- **Cron jobs**: Check `/var/mail/username` or redirect to files
- **systemd**: `sudo journalctl -u smartsheet-sync -f`
- **Docker**: `docker logs container_name`
- **GitHub Actions**: Actions tab in repository

### Health Checks

Create a monitoring script `check_sync.sh`:
```bash
#!/bin/bash
LOG_FILE="/var/log/smartsheet-sync.log"
ERROR_COUNT=$(tail -n 100 "$LOG_FILE" | grep -i error | wc -l)

if [ "$ERROR_COUNT" -gt 5 ]; then
    echo "High error count detected: $ERROR_COUNT"
    # Send alert (email, Slack, etc.)
fi
```

## Next Steps

Now that your environment is set up:

1. **Review**: [Smartsheet Integration](smartsheet-integration.md) for data flow details
2. **Be Aware**: [Watch Out For](watch-out-for.md) critical pitfalls
3. **Maintain**: [Maintenance Guide](maintenance-guide.md) for ongoing updates
4. **Troubleshoot**: [Troubleshooting](troubleshooting.md) when issues arise

## Quick Reference

### Common Commands

```bash
# Python virtual environment
python -m venv venv
source venv/bin/activate  # Linux/Mac
venv\Scripts\activate     # Windows

# Install dependencies
pip install -r requirements.txt
npm install

# Run with logging
python main.py 2>&1 | tee -a sync.log

# Check Python version
python --version

# Check installed packages
pip list
npm list
```

### Environment Variable Template

```env
# Smartsheet
SMARTSHEET_ACCESS_TOKEN=your_token_here

# Supabase
SUPABASE_URL=https://your-project.supabase.co
SUPABASE_KEY=your_key_here
SUPABASE_ANON_KEY=your_anon_key_here

# Sheet IDs (customize per project)
SHEET_ID=12345678901234
MASTER_SHEET_ID=12345678901234
SIBLING_SHEET_IDS=11111111111,22222222222

# Column IDs (get from API)
COLUMN_ID_NAME=1234567890123456
COLUMN_ID_JOB_NUMBER=2345678901234567

# Paths
PROMAX_DATA_PATH=/path/to/promax/data
EXCEL_FILE_PATH=/path/to/excel/file.xlsx
OUTPUT_PDF_DIR=/path/to/output
PDF_DIRECTORY=/path/to/pdfs

# Options
STARTING_NUMBER=1000
DRY_RUN=false
VERBOSE=true
```
