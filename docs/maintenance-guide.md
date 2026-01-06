# Maintenance Guide

This guide provides procedures for safely updating, extending, and maintaining the Smartsheet Integration Suite.

## Pre-Update Checklist

Before making any changes to production integrations:

- [ ] **Backup current state**
  - Export affected Smartsheet sheets
  - Backup Supabase database (if applicable)
  - Save current `.env` files
  - Document current behavior
- [ ] **Test in development**
  - Use test sheets, not production
  - Verify expected behavior
  - Check error handling
- [ ] **Review dependencies**
  - Check for breaking changes in libraries
  - Test with current Python/Node versions
- [ ] **Plan rollback**
  - Know how to revert changes
  - Keep old code accessible
  - Have restore procedures ready
- [ ] **Schedule maintenance window**
  - Notify users if applicable
  - Choose low-traffic time
  - Have monitoring in place

## Adding New Smartsheets

### To Supabase-Smartsheet-Offload

**1. Add Sheet Configuration**

Update `.env`:
```env
# Existing sheets
SHEET_ID_MAIN=1234567890123456
SHEET_ID_SECONDARY=2345678901234567

# New sheet
SHEET_ID_NEW=3456789012345678
```

**2. Add Column Mappings**

```python
# In configuration file or script
NEW_SHEET_MAPPINGS = {
    'supabase_column': 'SMARTSHEET_COLUMN_ID',
    'id': 1111111111111111,
    'name': 2222222222222222,
    'status': 3333333333333333,
}

# Add to main mappings
ALL_MAPPINGS = {
    'main_sheet': MAIN_SHEET_MAPPINGS,
    'secondary_sheet': SECONDARY_SHEET_MAPPINGS,
    'new_sheet': NEW_SHEET_MAPPINGS,  # New
}
```

**3. Test**

```bash
# Dry run with new sheet only
python main.py --sheet new_sheet --dry-run

# Verify in logs
# Check row count matches expectations
```

### To Smartsheet-Supabase-Sync

**1. Update GitHub Actions Workflow**

Edit `.github/workflows/sync.yml`:
```yaml
env:
  SHEET_ID_NEW: 3456789012345678
```

**2. Update TypeScript Configuration**

```typescript
// src/config.ts
export const SHEETS = {
  main: process.env.SHEET_ID_MAIN,
  secondary: process.env.SHEET_ID_SECONDARY,
  new: process.env.SHEET_ID_NEW,  // Add
};
```

**3. Add Supabase Table**

```sql
CREATE TABLE new_sheet_data (
  id BIGSERIAL PRIMARY KEY,
  smartsheet_row_id BIGINT UNIQUE,
  name TEXT,
  status TEXT,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_new_sheet_row_id ON new_sheet_data(smartsheet_row_id);
```

**4. Update Sync Logic**

```typescript
// src/sync.ts
async function syncNewSheet() {
  const sheet = await getSheet(SHEETS.new);
  const rows = sheet.rows;
  
  for (const row of rows) {
    await upsertToDatabase('new_sheet_data', mapRow(row));
  }
}

// Add to main sync function
await syncNewSheet();
```

### To Master-to-Sibling

**Adding a New Sibling Sheet**

Update `.env`:
```env
MASTER_SHEET_ID=1234567890123456
SIBLING_SHEET_IDS=2345678901234567,3456789012345678,4567890123456789  # Added new ID
```

That's it! The script will automatically include the new sibling.

### To Generate-Job-Numbers

**Using with a New Sheet**

Create new `.env`:
```env
SMARTSHEET_ACCESS_TOKEN=your_token
SHEET_ID=9876543210987654  # New sheet
COLUMN_ID_JOB_NUMBER=1111111111111111
STARTING_NUMBER=5000  # Different starting point
```

Run separately:
```bash
python generate_job_numbers.py --config .env.new_sheet
```

## Modifying Column Mappings

### When Column Names Change

If using column names instead of IDs (not recommended):

```python
# Before
column_name = 'Old Name'

# After
column_name = 'New Name'
```

### When Adding Columns to Sync

**1. Get New Column ID**

```python
sheet = client.Sheets.get_sheet(SHEET_ID)
for column in sheet.columns:
    if column.title == 'New Column':
        print(f"Column ID: {column.id}")
```

**2. Update Environment**

```env
COLUMN_ID_NEW_FIELD=1234567890123456
```

**3. Add to Mapping**

```python
COLUMN_MAPPINGS = {
    # Existing mappings...
    'new_field': int(os.environ['COLUMN_ID_NEW_FIELD']),
}
```

**4. Update Database Schema** (if Supabase)

```sql
ALTER TABLE your_table 
ADD COLUMN new_field TEXT;
```

**5. Deploy and Test**

```bash
# Test first
python main.py --dry-run

# Deploy
python main.py
```

### When Removing Columns

**1. Remove from Mapping**

```python
COLUMN_MAPPINGS = {
    'field1': 1234567890123456,
    'field2': 2345678901234567,
    # 'old_field': 3456789012345678,  # Removed
}
```

**2. Update Database** (optional - may keep for history)

```sql
-- Option 1: Drop column
ALTER TABLE your_table DROP COLUMN old_field;

-- Option 2: Keep but stop using
-- Do nothing, just ignore the column
```

**3. Clean Up References**

Search codebase for references:
```bash
grep -r "old_field" .
```

## Updating Dependencies

### Python Dependencies

**1. Check Current Versions**

```bash
pip list
pip show smartsheet-python-sdk
```

**2. Update Requirements**

```txt
# requirements.txt
smartsheet-python-sdk>=3.0.0,<4.0.0  # Update version
supabase>=1.0.0
python-dotenv>=0.19.0
```

**3. Test in Virtual Environment**

```bash
python -m venv venv_test
source venv_test/bin/activate
pip install -r requirements.txt

# Run tests
python main.py --dry-run
```

**4. Deploy**

```bash
# Production environment
source venv/bin/activate
pip install -r requirements.txt --upgrade
```

### Node.js Dependencies

**1. Check for Updates**

```bash
npm outdated
```

**2. Update Package.json**

```json
{
  "dependencies": {
    "@supabase/supabase-js": "^2.38.0",
    "dotenv": "^16.0.0"
  }
}
```

**3. Install and Test**

```bash
npm install
npm test
npm run start -- --dry-run
```

### Dependency Security

**Check for Vulnerabilities**

```bash
# Python
pip check
pip install safety
safety check

# Node.js
npm audit
npm audit fix
```

## Database Schema Changes

### Adding Columns

```sql
-- Add new column
ALTER TABLE smartsheet_data 
ADD COLUMN new_field VARCHAR(255);

-- Add with default
ALTER TABLE smartsheet_data 
ADD COLUMN is_active BOOLEAN DEFAULT TRUE;

-- Add with constraint
ALTER TABLE smartsheet_data 
ADD COLUMN email VARCHAR(255) UNIQUE;
```

### Modifying Columns

```sql
-- Change type
ALTER TABLE smartsheet_data 
ALTER COLUMN status TYPE VARCHAR(100);

-- Add constraint
ALTER TABLE smartsheet_data 
ADD CONSTRAINT check_status 
CHECK (status IN ('active', 'inactive', 'pending'));

-- Set default
ALTER TABLE smartsheet_data 
ALTER COLUMN created_at SET DEFAULT NOW();
```

### Creating Indexes

```sql
-- Single column
CREATE INDEX idx_status ON smartsheet_data(status);

-- Multiple columns
CREATE INDEX idx_status_date ON smartsheet_data(status, created_at);

-- Unique index
CREATE UNIQUE INDEX idx_unique_email ON smartsheet_data(email);
```

### Migration Best Practices

1. **Backup First**
```bash
pg_dump -h localhost -U user database > backup.sql
```

2. **Test Migration**
```sql
BEGIN;
-- Your migration
ALTER TABLE smartsheet_data ADD COLUMN test TEXT;
-- Test it
SELECT * FROM smartsheet_data LIMIT 1;
-- Rollback if needed
ROLLBACK;
-- Or commit
COMMIT;
```

3. **Document Migration**
```sql
-- migrations/001_add_new_field.sql
-- Purpose: Add tracking field for sync status
-- Date: 2025-01-15
-- Author: Developer Name

ALTER TABLE smartsheet_data 
ADD COLUMN sync_status VARCHAR(50) DEFAULT 'pending';

CREATE INDEX idx_sync_status ON smartsheet_data(sync_status);
```

## Adding New Features

### Code Patterns to Follow

#### 1. Configuration Management

```python
import os
from typing import Optional

class Config:
    """Centralized configuration management"""
    
    def __init__(self):
        self.validate()
    
    @property
    def smartsheet_token(self) -> str:
        return os.environ['SMARTSHEET_ACCESS_TOKEN']
    
    @property
    def sheet_id(self) -> int:
        return int(os.environ['SHEET_ID'])
    
    def get_optional(self, key: str, default: Optional[str] = None) -> Optional[str]:
        return os.getenv(key, default)
    
    def validate(self):
        """Validate required configuration"""
        required = ['SMARTSHEET_ACCESS_TOKEN', 'SHEET_ID']
        missing = [k for k in required if not os.getenv(k)]
        if missing:
            raise ValueError(f"Missing required config: {missing}")

# Usage
config = Config()
```

#### 2. Error Handling

```python
import logging
from typing import Any, Callable
import time

logger = logging.getLogger(__name__)

def retry_on_error(
    func: Callable,
    max_retries: int = 3,
    backoff: int = 2
) -> Any:
    """Retry function with exponential backoff"""
    for attempt in range(max_retries):
        try:
            return func()
        except Exception as e:
            logger.warning(f"Attempt {attempt + 1} failed: {e}")
            if attempt < max_retries - 1:
                wait = backoff ** attempt
                logger.info(f"Retrying in {wait}s...")
                time.sleep(wait)
            else:
                logger.error("Max retries exceeded")
                raise

# Usage
result = retry_on_error(lambda: client.Sheets.get_sheet(sheet_id))
```

#### 3. Logging

```python
import logging
import sys

def setup_logging(level: str = 'INFO'):
    """Configure structured logging"""
    logging.basicConfig(
        level=getattr(logging, level.upper()),
        format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
        handlers=[
            logging.FileHandler('app.log'),
            logging.StreamHandler(sys.stdout)
        ]
    )

# Usage
setup_logging()
logger = logging.getLogger(__name__)
logger.info("Starting sync process")
```

#### 4. Data Validation

```python
from typing import Dict, Any
from datetime import datetime

def validate_row_data(row: Dict[str, Any]) -> bool:
    """Validate row data before syncing"""
    required_fields = ['id', 'name']
    
    # Check required fields
    for field in required_fields:
        if field not in row or not row[field]:
            logger.warning(f"Missing required field: {field}")
            return False
    
    # Validate types
    if not isinstance(row['id'], (int, str)):
        logger.warning(f"Invalid ID type: {type(row['id'])}")
        return False
    
    # Validate dates
    if 'created_at' in row:
        try:
            datetime.fromisoformat(str(row['created_at']))
        except ValueError:
            logger.warning(f"Invalid date: {row['created_at']}")
            return False
    
    return True
```

### Adding a New Repository

**1. Clone Template**

Use an existing repository as a template:
```bash
git clone https://github.com/JFlo21/master-to-sibling-smartsheet-function.git my-new-integration
cd my-new-integration
rm -rf .git
git init
```

**2. Update Configuration**

- Rename files appropriately
- Update README.md
- Modify `.env.example`
- Update `requirements.txt` or `package.json`

**3. Implement Logic**

Follow existing patterns for:
- Configuration loading
- Smartsheet client initialization
- Error handling
- Logging

**4. Test Thoroughly**

```bash
# Unit tests
python -m pytest tests/

# Integration test with dry-run
python main.py --dry-run

# Test with small dataset
python main.py --limit 10
```

**5. Document**

- Add to this documentation site
- Update master index
- Document environment variables
- Provide usage examples

## Best Practices

### 1. Version Control

```bash
# Always use branches
git checkout -b feature/add-new-sheet

# Commit frequently with descriptive messages
git commit -m "feat: add support for new project sheet"

# Test before merging
git checkout main
git merge feature/add-new-sheet
```

### 2. Environment Separation

```
.env.development
.env.staging
.env.production
```

```bash
# Use environment-specific files
python main.py --env production
```

### 3. Monitoring

```python
# Add health checks
def health_check():
    checks = {
        'smartsheet_api': test_smartsheet_connection(),
        'database': test_database_connection(),
        'disk_space': check_disk_space(),
    }
    
    for check, status in checks.items():
        logger.info(f"{check}: {'OK' if status else 'FAILED'}")
    
    return all(checks.values())

if not health_check():
    logger.critical("Health check failed!")
    sys.exit(1)
```

### 4. Documentation

Always update:
- Code comments
- README.md
- This documentation site
- Environment variable examples
- Changelog

### 5. Security

```bash
# Regular security audits
pip install bandit
bandit -r .

# Check for secrets
git diff | grep -i "token\|key\|password"
```

## Rollback Procedures

### Code Rollback

```bash
# Revert to previous commit
git revert HEAD
git push

# Or reset to specific commit
git reset --hard abc1234
git push -f  # Use with caution!
```

### Database Rollback

```bash
# Restore from backup
psql database < backup.sql

# Or if using migrations
python migrate.py down
```

### Configuration Rollback

```bash
# Restore .env from backup
cp .env.backup .env

# Restart services
systemctl restart smartsheet-sync
```

## Monitoring Integration Health

### Key Metrics to Track

1. **Sync Success Rate**: Percentage of successful syncs
2. **API Error Rate**: 4xx and 5xx errors
3. **Sync Duration**: Time taken for each sync
4. **Row Count**: Number of rows processed
5. **Rate Limit Usage**: Requests per minute

### Example Monitoring Script

```python
import json
from datetime import datetime

def log_metrics(metrics: dict):
    """Log metrics for monitoring"""
    metrics['timestamp'] = datetime.now().isoformat()
    
    with open('metrics.jsonl', 'a') as f:
        f.write(json.dumps(metrics) + '\n')

# Usage
log_metrics({
    'sync_duration': 45.2,
    'rows_processed': 150,
    'errors': 0,
    'status': 'success'
})
```

## Getting Help

If you encounter issues:

1. Check [Troubleshooting Guide](troubleshooting.md)
2. Review [Watch Out For](watch-out-for.md)
3. Check application logs
4. Verify environment configuration
5. Test with dry-run mode
6. Reach out to repository maintainers

## Next Steps

- **[Troubleshooting](troubleshooting.md)** - Debug common issues
- **[Watch Out For](watch-out-for.md)** - Critical warnings
- **[Smartsheet Integration](smartsheet-integration.md)** - Data flow details
