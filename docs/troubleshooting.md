# Troubleshooting Guide

This guide helps you diagnose and resolve common issues across all Smartsheet Integration repositories.

## Quick Diagnostic Table

| Symptom | Possible Cause | Quick Fix | Full Solution |
|---------|---------------|-----------|---------------|
| **403 Forbidden** | Invalid/expired token | Check token | [Authentication Issues](#authentication-issues) |
| **429 Rate Limit** | Too many API calls | Wait 60s | [Rate Limiting](#rate-limiting-errors) |
| **Connection timeout** | Network/firewall | Check network | [Connection Issues](#connection-issues) |
| **No data synced** | Column ID mismatch | Verify IDs | [Data Sync Issues](#data-sync-issues) |
| **Duplicate rows** | Bad unique key | Check upsert logic | [Duplicate Data](#duplicate-data) |
| **Import fails** | Missing dependencies | `pip install -r requirements.txt` | [Runtime Errors](#runtime-errors) |
| **Script hangs** | API timeout | Increase timeout | [Performance Issues](#performance-issues) |
| **Wrong data** | Column mapping error | Check mappings | [Data Integrity](#data-integrity-issues) |

## Connection Issues

### Cannot Connect to Smartsheet API

**Symptoms**:
```
ConnectionError: Unable to reach Smartsheet API
requests.exceptions.ConnectionError
```

**Diagnostic Steps**:

1. **Check Internet Connection**
```bash
ping api.smartsheet.com
curl -I https://api.smartsheet.com/2.0/
```

2. **Verify API Endpoint**
```python
import requests
response = requests.get('https://api.smartsheet.com/2.0/serverinfo')
print(response.status_code)  # Should be 200
```

3. **Check Firewall/Proxy**
```bash
# Test with curl
curl -v https://api.smartsheet.com/2.0/serverinfo

# If using proxy
export HTTP_PROXY=http://proxy.company.com:8080
export HTTPS_PROXY=http://proxy.company.com:8080
```

**Solutions**:

- **Corporate Network**: Configure proxy settings
```python
import smartsheet

client = smartsheet.Smartsheet(
    api_token,
    proxies={'https': 'http://proxy.company.com:8080'}
)
```

- **VPN Issues**: Ensure VPN allows API access
- **DNS Issues**: Try using IP address or alternate DNS

### Cannot Connect to Supabase

**Symptoms**:
```
ConnectionError: Unable to connect to Supabase
supabase.exceptions.SupabaseException
```

**Diagnostic Steps**:

1. **Verify Supabase URL**
```bash
curl https://your-project.supabase.co/rest/v1/
```

2. **Check Credentials**
```python
from supabase import create_client

url = os.environ['SUPABASE_URL']
key = os.environ['SUPABASE_KEY']

try:
    client = create_client(url, key)
    # Test query
    response = client.table('test').select('*').limit(1).execute()
    print("Connection successful")
except Exception as e:
    print(f"Connection failed: {e}")
```

**Solutions**:

- **Invalid URL**: Check project URL in Supabase dashboard
- **Wrong Key**: Use `service_role` key, not `anon` key for server-side
- **Network Issues**: Check firewall rules

### SSL Certificate Errors

**Symptoms**:
```
SSLError: [SSL: CERTIFICATE_VERIFY_FAILED]
```

**Solutions**:

```python
# Temporary workaround (NOT recommended for production)
import ssl
import certifi

# Use certifi certificates
import smartsheet
client = smartsheet.Smartsheet(api_token)

# Or update CA certificates
# Ubuntu/Debian
sudo apt-get install ca-certificates
sudo update-ca-certificates

# MacOS
pip install --upgrade certifi
```

## Authentication Issues

### Invalid or Expired API Token

**Symptoms**:
```
ApiError: 1004 - You are not authorized to perform this action
ApiError: 1002 - Your access token is invalid
```

**Diagnostic Steps**:

1. **Test Token**
```bash
curl -H "Authorization: Bearer YOUR_TOKEN" \
  https://api.smartsheet.com/2.0/users/me
```

2. **Check Token in Code**
```python
import smartsheet

client = smartsheet.Smartsheet(os.environ['SMARTSHEET_ACCESS_TOKEN'])
try:
    user = client.Users.get_current_user()
    print(f"Token valid for: {user.email}")
except Exception as e:
    print(f"Token invalid: {e}")
```

**Solutions**:

- **Regenerate Token**: Go to Smartsheet → Apps & Integrations → API Access
- **Check Environment**: Ensure `.env` file is loaded
```python
from dotenv import load_dotenv
load_dotenv()  # Add this at the start
```

- **Verify Token Format**: Should start with `ll` or similar prefix

### Insufficient Permissions

**Symptoms**:
```
ApiError: 1006 - You do not have permission to access this resource
```

**Solutions**:

1. **Check Sheet Sharing**:
   - Open sheet in Smartsheet
   - Click Share
   - Verify user has Editor or Admin access

2. **Use Admin Token**: Generate token from account with appropriate permissions

3. **Check Sheet ID**: Ensure using correct sheet ID
```python
# List all accessible sheets
sheets = client.Sheets.list_sheets(include_all=True)
for sheet in sheets.data:
    print(f"{sheet.name}: {sheet.id}")
```

## Data Sync Issues

### No Data Being Synced

**Diagnostic Steps**:

1. **Verify Sheet Has Data**
```python
sheet = client.Sheets.get_sheet(SHEET_ID)
print(f"Row count: {sheet.total_row_count}")
print(f"Columns: {[col.title for col in sheet.columns]}")
```

2. **Check Column IDs**
```python
# Compare expected vs actual
sheet = client.Sheets.get_sheet(SHEET_ID)
for column in sheet.columns:
    print(f"{column.title}: {column.id}")

# Compare with your config
print(f"Your COLUMN_ID_NAME: {COLUMN_ID_NAME}")
```

3. **Enable Verbose Logging**
```python
import logging
logging.basicConfig(level=logging.DEBUG)
```

**Solutions**:

- **Column ID Mismatch**: Update column IDs in `.env`
- **Empty Rows**: Check for filter conditions excluding all data
- **Permission Issues**: Verify read access to sheet

### Data Not Updating

**Symptoms**: Script runs successfully but data doesn't change

**Diagnostic Steps**:

1. **Check Dry-Run Mode**
```python
# Make sure dry-run is disabled
DRY_RUN = os.getenv('DRY_RUN', 'false').lower() == 'true'
if DRY_RUN:
    print("DRY RUN MODE - No changes will be made")
```

2. **Verify Update Payload**
```python
# Log what's being sent
logger.debug(f"Updating row {row.id} with: {row.to_dict()}")
result = client.Sheets.update_rows(sheet_id, [row])
logger.debug(f"Result: {result}")
```

3. **Check for Formula Columns**
```python
# Formula columns can't be updated
sheet = client.Sheets.get_sheet(SHEET_ID)
formula_columns = [col for col in sheet.columns if col.formula]
print(f"Formula columns: {[col.title for col in formula_columns]}")
```

**Solutions**:

- **Disable Dry-Run**: Set `DRY_RUN=false`
- **Check Return Status**: Look for errors in API response
- **Verify Column Write Access**: Can't write to formula or system columns

### Stale Data

**Symptoms**: Syncs old data repeatedly, misses new changes

**Solution**: Implement incremental sync

```python
import json
from datetime import datetime, timedelta

STATE_FILE = 'sync_state.json'

def get_last_sync_time():
    try:
        with open(STATE_FILE, 'r') as f:
            state = json.load(f)
            return datetime.fromisoformat(state['last_sync'])
    except FileNotFoundError:
        # First run - sync last 24 hours
        return datetime.now() - timedelta(days=1)

def save_sync_time():
    with open(STATE_FILE, 'w') as f:
        json.dump({'last_sync': datetime.now().isoformat()}, f)

# Use in sync logic
last_sync = get_last_sync_time()
sheet = client.Sheets.get_sheet(
    SHEET_ID,
    rows_modified_since=last_sync
)
# Process rows...
save_sync_time()
```

## Duplicate Data

### Duplicate Rows Created

**Diagnostic Steps**:

1. **Identify Duplicates**
```python
sheet = client.Sheets.get_sheet(SHEET_ID)
ids = []
duplicates = []

for row in sheet.rows:
    row_id = get_cell_value(row, COLUMN_ID_UNIQUE_KEY)
    if row_id in ids:
        duplicates.append(row_id)
    ids.append(row_id)

print(f"Found {len(duplicates)} duplicates: {set(duplicates)}")
```

2. **Check Upsert Logic**
```python
# Verify unique key is actually unique
unique_values = [get_cell_value(row, COLUMN_ID_UNIQUE_KEY) for row in sheet.rows]
if len(unique_values) != len(set(unique_values)):
    print("WARNING: Unique key has duplicates!")
```

**Solutions**:

**Option 1: Use External ID for Upsert**
```python
# When creating/updating rows
row = smartsheet.models.Row()
row.to_bottom = True
row.cells.append({
    'column_id': COLUMN_ID_NAME,
    'value': 'John Doe'
})

# Set external ID for upsert capability
row.external_id = f"user_{user_id}"

# Later, to upsert
row = smartsheet.models.Row()
row.external_id = f"user_{user_id}"  # Same external ID
row.cells.append({'column_id': COLUMN_ID_NAME, 'value': 'Jane Doe'})
# This will update existing row with same external ID
```

**Option 2: Query Before Insert**
```python
def row_exists(sheet_id, column_id, value):
    sheet = client.Sheets.get_sheet(sheet_id)
    for row in sheet.rows:
        cell_value = get_cell_value(row, column_id)
        if cell_value == value:
            return row
    return None

# Before inserting
existing_row = row_exists(SHEET_ID, COLUMN_ID_UNIQUE_KEY, new_value)
if existing_row:
    # Update existing
    update_row(existing_row.id, new_data)
else:
    # Insert new
    insert_row(new_data)
```

**Option 3: Clean Up Duplicates**
```python
def remove_duplicates(sheet_id, column_id):
    sheet = client.Sheets.get_sheet(sheet_id)
    seen = {}
    to_delete = []
    
    for row in sheet.rows:
        value = get_cell_value(row, column_id)
        if value in seen:
            # Keep older row, delete newer
            to_delete.append(row.id)
        else:
            seen[value] = row.id
    
    if to_delete:
        print(f"Deleting {len(to_delete)} duplicate rows...")
        client.Sheets.delete_rows(sheet_id, to_delete)
```

## Runtime Errors

### Module Not Found

**Symptoms**:
```
ModuleNotFoundError: No module named 'smartsheet'
ImportError: cannot import name 'create_client'
```

**Solutions**:

```bash
# Install dependencies
pip install -r requirements.txt

# Or specific packages
pip install smartsheet-python-sdk
pip install supabase
pip install python-dotenv

# For Node.js
npm install
```

### Python Version Issues

**Symptoms**:
```
SyntaxError: invalid syntax (using f-strings in Python 3.5)
```

**Solutions**:

```bash
# Check Python version
python --version
python3 --version

# Should be 3.9+
# Use specific version
python3.9 main.py

# Or update Python
sudo apt install python3.9
```

### Environment Variables Not Loading

**Symptoms**:
```
KeyError: 'SMARTSHEET_ACCESS_TOKEN'
```

**Solutions**:

```python
# Ensure .env is loaded
from dotenv import load_dotenv
load_dotenv()  # Add at the very start

# Or specify .env path
load_dotenv('/path/to/.env')

# Verify loading
import os
print(f"Token loaded: {'SMARTSHEET_ACCESS_TOKEN' in os.environ}")
```

```bash
# Or export manually
export SMARTSHEET_ACCESS_TOKEN=your_token
python main.py
```

### JSON Decode Error

**Symptoms**:
```
json.decoder.JSONDecodeError: Expecting value: line 1 column 1
```

**Diagnostic**:

```python
# Check API response
response = client.Sheets.get_sheet(SHEET_ID)
print(f"Response type: {type(response)}")
print(f"Response: {response}")

# If parsing JSON manually
try:
    data = json.loads(response_text)
except json.JSONDecodeError as e:
    print(f"Invalid JSON: {response_text[:200]}")
    print(f"Error at position {e.pos}: {e.msg}")
```

## Rate Limiting Errors

### 429 Too Many Requests

**Symptoms**:
```
ApiError: 429 - Rate limit exceeded
```

**Immediate Fix**:
```bash
# Wait 60 seconds, then retry
sleep 60
python main.py
```

**Long-term Solution**:

```python
import time
from smartsheet.exceptions import ApiError

def api_call_with_retry(func, max_retries=5):
    for attempt in range(max_retries):
        try:
            return func()
        except ApiError as e:
            if e.error.result.status_code == 429:
                wait_time = min(60 * (2 ** attempt), 300)  # Cap at 5 minutes
                logger.warning(f"Rate limited. Waiting {wait_time}s...")
                time.sleep(wait_time)
            else:
                raise
    raise Exception("Rate limit retries exceeded")

# Usage
result = api_call_with_retry(
    lambda: client.Sheets.update_rows(sheet_id, rows)
)
```

**Prevent Rate Limits**:

1. **Use Bulk Operations**
```python
# Instead of
for row in rows:
    client.Sheets.update_rows(sheet_id, [row])

# Do this
client.Sheets.update_rows(sheet_id, rows)
```

2. **Add Delays**
```python
import time

for batch in batches:
    process_batch(batch)
    time.sleep(1)  # 1 second between batches
```

3. **Track Request Count**
```python
class RateLimiter:
    def __init__(self, max_per_minute=250):
        self.max_per_minute = max_per_minute
        self.requests = []
    
    def wait_if_needed(self):
        now = time.time()
        # Remove requests older than 60 seconds
        self.requests = [t for t in self.requests if now - t < 60]
        
        if len(self.requests) >= self.max_per_minute:
            sleep_time = 60 - (now - self.requests[0])
            logger.info(f"Rate limit protection: sleeping {sleep_time:.1f}s")
            time.sleep(sleep_time)
            self.requests = []
        
        self.requests.append(time.time())

limiter = RateLimiter()

# Before each API call
limiter.wait_if_needed()
client.Sheets.update_rows(sheet_id, rows)
```

## Performance Issues

### Slow Sync Times

**Diagnostic**:

```python
import time

def timed_operation(name, func):
    start = time.time()
    result = func()
    duration = time.time() - start
    logger.info(f"{name} took {duration:.2f}s")
    return result

# Usage
sheet = timed_operation(
    "Fetch sheet",
    lambda: client.Sheets.get_sheet(SHEET_ID)
)

timed_operation(
    "Update rows",
    lambda: client.Sheets.update_rows(SHEET_ID, rows)
)
```

**Solutions**:

1. **Reduce Data Volume**
```python
# Only fetch needed columns
sheet = client.Sheets.get_sheet(
    SHEET_ID,
    column_ids=[COLUMN_ID_NAME, COLUMN_ID_STATUS]
)

# Limit rows
sheet = client.Sheets.get_sheet(
    SHEET_ID,
    page_size=100,
    page=1
)
```

2. **Use Parallel Processing**
```python
from concurrent.futures import ThreadPoolExecutor

def process_batch(batch):
    # Process batch
    pass

batches = [rows[i:i+100] for i in range(0, len(rows), 100)]

with ThreadPoolExecutor(max_workers=4) as executor:
    executor.map(process_batch, batches)
```

3. **Optimize Database Queries**
```sql
-- Add indexes
CREATE INDEX idx_smartsheet_row_id ON table_name(smartsheet_row_id);

-- Use specific columns
SELECT id, name, status FROM table  -- Instead of SELECT *
```

### Memory Issues

**Symptoms**:
```
MemoryError
Killed (OOM)
```

**Solutions**:

1. **Process in Batches**
```python
def process_in_batches(sheet_id, batch_size=100):
    page = 1
    while True:
        sheet = client.Sheets.get_sheet(
            sheet_id,
            page_size=batch_size,
            page=page
        )
        
        if not sheet.rows:
            break
        
        process_rows(sheet.rows)
        page += 1
```

2. **Use Generators**
```python
def row_generator(sheet_id):
    sheet = client.Sheets.get_sheet(sheet_id)
    for row in sheet.rows:
        yield row

# Instead of loading all at once
for row in row_generator(SHEET_ID):
    process_row(row)
```

## Data Integrity Issues

### Wrong Data in Cells

**Diagnostic**:

```python
# Check cell values and types
sheet = client.Sheets.get_sheet(SHEET_ID)
row = sheet.rows[0]

for cell in row.cells:
    column = next(col for col in sheet.columns if col.id == cell.column_id)
    print(f"{column.title}: {cell.value} (type: {type(cell.value)})")
```

**Solutions**:

1. **Validate Before Writing**
```python
def validate_cell_value(column_type, value):
    if column_type == 'DATE':
        try:
            datetime.fromisoformat(str(value))
            return True
        except ValueError:
            return False
    elif column_type == 'CHECKBOX':
        return isinstance(value, bool)
    return True

# Use before updating
if not validate_cell_value(column.type, new_value):
    logger.error(f"Invalid value {new_value} for column {column.title}")
```

2. **Handle Type Conversions**
```python
def convert_for_smartsheet(column_type, value):
    if value is None:
        return ''
    
    if column_type == 'DATE':
        if isinstance(value, datetime):
            return value.strftime('%Y-%m-%d')
        return value
    
    elif column_type == 'CHECKBOX':
        return bool(value)
    
    elif column_type == 'TEXT_NUMBER':
        return str(value)
    
    return value
```

## Debugging Tools

### Enable Debug Logging

```python
import logging

# Maximum verbosity
logging.basicConfig(
    level=logging.DEBUG,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
)

# Log API requests
import smartsheet
client = smartsheet.Smartsheet(token)
client.logger.setLevel(logging.DEBUG)
```

### Inspect API Responses

```python
# Capture raw response
response = client.Sheets.get_sheet(SHEET_ID)

# Print structure
import json
print(json.dumps(response.to_dict(), indent=2))
```

### Test Connectivity Script

```bash
#!/bin/bash
# test_connectivity.sh

echo "Testing Smartsheet API..."
curl -H "Authorization: Bearer $SMARTSHEET_ACCESS_TOKEN" \
  https://api.smartsheet.com/2.0/users/me

echo -e "\n\nTesting Supabase..."
curl "$SUPABASE_URL/rest/v1/"

echo -e "\n\nDone!"
```

## Getting Further Help

If issues persist:

1. **Check Logs**: Review application logs for detailed errors
2. **Enable Debug Mode**: Use verbose logging
3. **Test Components**: Test Smartsheet API, database, network separately
4. **Review Recent Changes**: What changed before the issue started?
5. **Check Service Status**: 
   - [Smartsheet Status](https://status.smartsheet.com/)
   - [Supabase Status](https://status.supabase.com/)

## Related Documentation

- **[Watch Out For](watch-out-for.md)** - Common pitfalls
- **[Maintenance Guide](maintenance-guide.md)** - Update procedures
- **[Smartsheet Integration](smartsheet-integration.md)** - Data flow details
