# Watch Out For

!!! danger "Critical Warning"
    This page documents **critical pitfalls, risks, and gotchas** that can cause data loss, system failures, or production incidents. Read carefully before deploying or modifying integrations.

## Configuration Pitfalls

### Missing Environment Variables

**Risk Level**: 🔴 **CRITICAL**

**Problem**: Scripts fail silently or use wrong default values when environment variables are missing.

**Examples**:
```python
# BAD - Fails silently
sheet_id = os.getenv('SHEET_ID', '12345')  # Uses hardcoded default!

# GOOD - Fails loudly
sheet_id = os.environ['SHEET_ID']  # Raises KeyError if missing
```

**Prevention**:
- Always validate environment variables at startup
- Use a configuration validation function
- Never use default values for critical settings

```python
# Configuration validation example
def validate_config():
    required_vars = [
        'SMARTSHEET_ACCESS_TOKEN',
        'SHEET_ID',
        'COLUMN_ID_NAME',
    ]
    missing = [var for var in required_vars if not os.getenv(var)]
    if missing:
        raise ValueError(f"Missing required environment variables: {missing}")

validate_config()  # Call at startup
```

### Hardcoded IDs in Code

**Risk Level**: 🟠 **HIGH**

**Problem**: Sheet IDs, column IDs, or other identifiers hardcoded in scripts break when sheets are copied or columns change.

**Examples**:
```python
# BAD - Hardcoded
sheet_id = 1234567890123456

# GOOD - From environment
sheet_id = int(os.environ['SHEET_ID'])
```

**Where to Check**:
- Main application files
- Configuration modules
- Test files
- Helper scripts

### Wrong Sheet in Environment Files

**Risk Level**: 🔴 **CRITICAL**

**Problem**: Using production sheet ID in development or vice versa causes unintended data modifications.

**Prevention**:
- Use separate `.env.development` and `.env.production` files
- Add sheet name validation
- Use descriptive variable names

```python
# Add validation
SHEET_ID = os.environ['SHEET_ID']
EXPECTED_SHEET_NAME = os.environ.get('EXPECTED_SHEET_NAME')

if EXPECTED_SHEET_NAME:
    sheet = client.Sheets.get_sheet(SHEET_ID)
    if sheet.name != EXPECTED_SHEET_NAME:
        raise ValueError(f"Sheet name mismatch! Expected '{EXPECTED_SHEET_NAME}', got '{sheet.name}'")
```

### API Token Permissions

**Risk Level**: 🔴 **CRITICAL**

**Problem**: API token lacks permissions to read/write sheets, causing silent failures or errors.

**Symptoms**:
- 403 Forbidden errors
- "You do not have permission to access this resource"
- Empty data returns

**Prevention**:
- Use Admin-level API tokens
- Test token permissions before deployment
- Document required permissions

```python
# Test token at startup
try:
    user = client.Users.get_current_user()
    print(f"Authenticated as: {user.email}")
except Exception as e:
    raise ValueError(f"Invalid or expired API token: {e}")
```

## Data Integrity Risks

### Upsert Conflicts

**Risk Level**: 🟠 **HIGH**

**Problem**: Upserting with wrong unique key causes duplicates or overwrites wrong rows.

**Example Scenario**:
```python
# If "Job ID" isn't truly unique, this creates duplicates
for row in data:
    upsert_row(row['job_id'], row['data'])
```

**Prevention**:
- Verify unique key is actually unique
- Use Smartsheet's row ID when possible
- Implement conflict detection

```python
# Check for duplicates before upserting
existing_ids = get_all_job_ids(sheet_id)
duplicate_ids = [id for id in new_ids if new_ids.count(id) > 1]
if duplicate_ids:
    raise ValueError(f"Duplicate IDs detected: {duplicate_ids}")
```

### Historical Data Backfill

**Risk Level**: 🟠 **HIGH**

**Problem**: Running sync for the first time without limiting date range can:
- Hit API rate limits
- Take hours/days
- Create thousands of unexpected rows

**Prevention**:
```python
# Add date filter for initial sync
START_DATE = os.getenv('SYNC_START_DATE', '2025-01-01')

# Filter rows by date
filtered_rows = [
    row for row in rows 
    if row.created_date >= START_DATE
]
```

**Safe Backfill Process**:
1. Test with 1 day of data
2. Gradually increase date range
3. Monitor API usage
4. Use batch operations

### State Loss Between Runs

**Risk Level**: 🟡 **MEDIUM**

**Problem**: Script doesn't track last sync time, causing repeated processing of same data.

**Solutions**:

**Option 1: State File**
```python
import json
from datetime import datetime

STATE_FILE = 'sync_state.json'

def save_state(last_sync_time):
    with open(STATE_FILE, 'w') as f:
        json.dump({'last_sync': last_sync_time.isoformat()}, f)

def load_state():
    try:
        with open(STATE_FILE, 'r') as f:
            data = json.load(f)
            return datetime.fromisoformat(data['last_sync'])
    except FileNotFoundError:
        return None
```

**Option 2: Database Tracking**
```sql
CREATE TABLE sync_state (
    id SERIAL PRIMARY KEY,
    repository VARCHAR(100),
    last_sync TIMESTAMP,
    status VARCHAR(50)
);
```

### Formula Column Overwrites

**Risk Level**: 🟠 **HIGH**

**Problem**: Attempting to write to formula columns causes API errors and can break formulas if column type is changed.

**Identification**:
```python
# Check if column is a formula
sheet = client.Sheets.get_sheet(sheet_id)
for column in sheet.columns:
    if column.formula:
        print(f"WARNING: {column.title} is a formula column")
```

**Prevention**:
- Never write to formula columns
- Filter them out in mappings
- Document which columns are formulas

```python
# Filter out formula columns
def get_writable_columns(sheet_id):
    sheet = client.Sheets.get_sheet(sheet_id)
    return [col for col in sheet.columns if not col.formula]
```

## API & Rate Limiting Concerns

### Rate Limit Exhaustion

**Risk Level**: 🟠 **HIGH**

**Problem**: Hitting 300 requests/minute limit causes:
- Script failures
- Incomplete syncs
- Cascading failures across multiple integrations

**Symptoms**:
```
ApiError: 429 - Rate limit exceeded
```

**Prevention**:

**1. Use Bulk Operations**
```python
# BAD - 100 requests for 100 rows
for row in rows:
    client.Sheets.update_rows(sheet_id, [row])

# GOOD - 1 request for 100 rows
client.Sheets.update_rows(sheet_id, rows)
```

**2. Implement Exponential Backoff**
```python
import time

def api_call_with_backoff(func, max_retries=5):
    for attempt in range(max_retries):
        try:
            return func()
        except ApiError as e:
            if e.error.result.status_code == 429:
                wait_time = (2 ** attempt) * 1  # 1, 2, 4, 8, 16 seconds
                print(f"Rate limited. Waiting {wait_time}s...")
                time.sleep(wait_time)
            else:
                raise
    raise Exception("Max retries exceeded")
```

**3. Rate Limit Tracking**
```python
import time

class RateLimiter:
    def __init__(self, max_requests=250, window=60):
        self.max_requests = max_requests
        self.window = window
        self.requests = []
    
    def wait_if_needed(self):
        now = time.time()
        # Remove old requests
        self.requests = [t for t in self.requests if now - t < self.window]
        
        if len(self.requests) >= self.max_requests:
            sleep_time = self.window - (now - self.requests[0])
            time.sleep(sleep_time)
        
        self.requests.append(time.time())

limiter = RateLimiter()
# Before each API call
limiter.wait_if_needed()
```

### Concurrent Access Conflicts

**Risk Level**: 🟡 **MEDIUM**

**Problem**: Multiple scripts modifying same sheet simultaneously causes race conditions.

**Scenarios**:
- Two cron jobs overlap
- Manual run while scheduled run is active
- Multiple developers testing

**Prevention**:
```python
import fcntl

def acquire_lock(lock_file='/tmp/smartsheet_sync.lock'):
    lock = open(lock_file, 'w')
    try:
        fcntl.flock(lock.fileno(), fcntl.LOCK_EX | fcntl.LOCK_NB)
        return lock
    except IOError:
        print("Another instance is running. Exiting.")
        sys.exit(0)

# At start of script
lock = acquire_lock()
```

### Network Timeout Issues

**Risk Level**: 🟡 **MEDIUM**

**Problem**: Long-running API calls timeout without proper configuration.

**Prevention**:
```python
import smartsheet

# Set longer timeout
client = smartsheet.Smartsheet(token, user_agent="MyApp/1.0")
client.request_timeout = 300  # 5 minutes

# Or per-request
client.Sheets.get_sheet(sheet_id, request_timeout=60)
```

## Synchronization Gotchas

### Bi-directional Sync Loops

**Risk Level**: 🔴 **CRITICAL**

**Problem**: Smartsheet → Supabase and Supabase → Smartsheet syncs create infinite loops.

**Example**:
1. User updates Smartsheet row
2. SSS syncs to Supabase (adds `updated_at` timestamp)
3. SPS sees new `updated_at`, syncs back to Smartsheet
4. SSS sees change, syncs to Supabase
5. **INFINITE LOOP**

**Prevention**:

**Option 1: Timestamps with Threshold**
```python
# Only sync if modified more than 5 minutes ago
SYNC_THRESHOLD = 300  # seconds
last_modified = row.modified_at
if time.time() - last_modified.timestamp() < SYNC_THRESHOLD:
    continue  # Skip this row
```

**Option 2: Sync Direction Flags**
```python
# Add metadata column indicating source
COLUMN_ID_SYNC_SOURCE = 1234567890123456

# When syncing from Smartsheet to Supabase
row['sync_source'] = 'smartsheet'

# When syncing from Supabase to Smartsheet
if row['sync_source'] == 'smartsheet':
    continue  # Don't sync back
```

**Option 3: One-way Sync Windows**
```python
# Run syncs at different times
# SSS: Every hour at :00
# SPS: Every hour at :30
```

### Deleted Row Handling

**Risk Level**: 🟡 **MEDIUM**

**Problem**: Deleted rows in Smartsheet aren't deleted in Supabase (or vice versa).

**Solutions**:

**Soft Deletes**:
```python
# Mark as deleted instead of removing
row['deleted_at'] = datetime.now()
row['is_deleted'] = True
```

**Periodic Reconciliation**:
```python
# Weekly job to find orphaned records
smartsheet_ids = get_all_smartsheet_row_ids()
supabase_ids = get_all_supabase_record_ids()
orphaned = supabase_ids - smartsheet_ids

for orphan_id in orphaned:
    # Decide: delete or mark inactive
    mark_inactive(orphan_id)
```

### Column Type Mismatches

**Risk Level**: 🟠 **HIGH**

**Problem**: Writing wrong data type to column causes silent failures or data corruption.

**Examples**:
| Smartsheet Type | Bad Input | Result |
|-----------------|-----------|--------|
| DATE | "not a date" | Error or null |
| CHECKBOX | "yes" | Should be `True` |
| PICKLIST | "Invalid Option" | Error |
| CONTACT_LIST | "John Doe" | Should be email |

**Prevention**:
```python
def validate_column_type(column_type, value):
    validators = {
        'DATE': lambda v: isinstance(v, (str, datetime)),
        'CHECKBOX': lambda v: isinstance(v, bool),
        'CONTACT_LIST': lambda v: '@' in str(v),
        'PICKLIST': lambda v: v in VALID_OPTIONS,
    }
    
    validator = validators.get(column_type)
    if validator and not validator(value):
        raise ValueError(f"Invalid value {value} for type {column_type}")
```

## Environment-Specific Issues

### Development vs Production Confusion

**Risk Level**: 🔴 **CRITICAL**

**Problem**: Accidentally running development code against production data.

**Prevention**:

**1. Environment Indicators**
```python
ENV = os.getenv('ENVIRONMENT', 'development')

if ENV == 'production':
    print("⚠️  RUNNING IN PRODUCTION ⚠️")
    confirm = input("Type 'CONFIRM' to continue: ")
    if confirm != 'CONFIRM':
        sys.exit(0)
```

**2. Separate Configuration Files**
```
.env.development
.env.production
.env.staging
```

**3. Visual Indicators in Logs**
```python
if ENV == 'production':
    logging.basicConfig(
        format='[PROD] %(asctime)s - %(message)s',
        level=logging.INFO
    )
```

### Credential Leakage

**Risk Level**: 🔴 **CRITICAL**

**Problem**: API tokens or passwords committed to git or logged.

**Prevention**:

**1. .gitignore**
```
.env
.env.*
*.log
sync_state.json
secrets/
```

**2. Never Log Tokens**
```python
# BAD
print(f"Using token: {api_token}")

# GOOD
print(f"Using token: {api_token[:8]}...{api_token[-4:]}")
# Output: "Using token: sk_live_...x7Kp"
```

**3. Pre-commit Hook**
```bash
#!/bin/bash
# .git/hooks/pre-commit

if git diff --cached | grep -i "smartsheet_access_token\|api_key\|password"; then
    echo "ERROR: Potential credential in commit!"
    exit 1
fi
```

## Quick Reference: Error Indicators

| Symptom | Likely Cause | Check |
|---------|-------------|-------|
| 403 Forbidden | Token permissions | API token settings |
| 429 Rate Limit | Too many requests | Rate limiter, bulk ops |
| Duplicate rows | Bad unique key | Upsert logic |
| Missing data | Column ID mismatch | Column mappings |
| Stale data | No incremental sync | Last sync timestamp |
| Formula errors | Writing to formula column | Column type check |
| Timeout | Large dataset | Batch size, timeout config |
| Infinite loop | Bi-directional sync | Sync logic, timestamps |

## Emergency Procedures

### If You Accidentally Delete Data

1. **STOP** all running integrations immediately
2. Contact Smartsheet support for row recovery (14-day window)
3. Restore from database backup if applicable
4. Review and fix deletion logic before restarting

### If You Hit Rate Limits

1. **PAUSE** all integrations for 1 minute
2. Review recent API calls in logs
3. Implement rate limiting if missing
4. Consider spreading load across time
5. Contact Smartsheet for rate limit increase if needed

### If Sync Creates Duplicates

1. **STOP** the integration
2. Identify duplicate rows (manual or script)
3. Delete duplicates (keep oldest or most complete)
4. Fix unique key logic
5. Test thoroughly before restarting

## Next Steps

- **[Maintenance Guide](maintenance-guide.md)** - Safe update procedures
- **[Troubleshooting](troubleshooting.md)** - Debug specific issues
- **[Smartsheet Integration](smartsheet-integration.md)** - Understanding data flows
