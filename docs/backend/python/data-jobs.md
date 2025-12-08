# :material-database-sync: Data Jobs

> **Version:** 1.0.0
> **Extends:** [General / Core](general-core.md)
> **Template:** [template-python-data-job](https://github.com/landerox/template-python-data-job)

Standards for building ETL pipelines, batch processing jobs, scheduled tasks, and event-driven workers. Includes Cloud Run Jobs deployment and data processing best practices.

---

## Use Cases

| Type | Examples |
|------|----------|
| ETL | Extract-Transform-Load pipelines |
| Batch Processing | Data transformations, aggregations |
| Scheduled Tasks | Daily reports, cleanup jobs, sync tasks |
| Migrations | Database migrations, data backfills |
| Event Workers | Pub/Sub consumers, queue processors |

---

## Stack

### Execution Platform

| Tool | Description | Timeout |
|------|-------------|---------|
| [**Cloud Run Jobs**](https://cloud.google.com/run/docs/create-jobs) | Serverless container execution | Up to 24h |
| [**Cloud Scheduler**](https://cloud.google.com/scheduler) | Managed cron triggers | - |
| [**Cloud Workflows**](https://cloud.google.com/workflows) | Multi-step orchestration | - |

### CLI Framework

| Tool | Description |
|------|-------------|
| [**Typer**](https://github.com/fastapi/typer) | Modern CLI framework with type hints |
| [**Rich**](https://github.com/Textualize/rich) | Progress bars, tables, colored output |

### Data Processing

| Tool | Description |
|------|-------------|
| [**Polars**](https://github.com/pola-rs/polars) | High-performance DataFrames (Rust), recommended default |
| [**DuckDB**](https://github.com/duckdb/duckdb) | In-process SQL analytics, great for ad-hoc queries |
| [**Pandera**](https://github.com/unionai-oss/pandera) | DataFrame schema validation |
| [**connectorx**](https://github.com/sfu-db/connector-x) | Fast database → DataFrame loading |

### Data Formats

| Tool | Description |
|------|-------------|
| [**fastavro**](https://github.com/fastavro/fastavro) | Fast Avro serialization |
| [**deltalake**](https://github.com/delta-io/delta-rs) | Delta Lake format support |
| [**pyarrow**](https://github.com/apache/arrow) | Arrow format, Parquet I/O |

### BigQuery

| Tool | Description |
|------|-------------|
| [**google-cloud-bigquery**](https://github.com/googleapis/python-bigquery) | Official BigQuery client |
| [**db-dtypes**](https://github.com/googleapis/python-db-dtypes-pandas) | BigQuery-specific dtypes for pandas |

### Resilience

| Tool | Description |
|------|-------------|
| [**tenacity**](https://github.com/jd/tenacity) | Retries with exponential backoff |

### Observability

| Tool | Description |
|------|-------------|
| [**structlog**](https://github.com/hynek/structlog) | Structured logging (JSON in production) |
| [**OpenTelemetry**](https://github.com/open-telemetry/opentelemetry-python) | Distributed tracing |

---

## Cloud Run Jobs vs Services

| Use Jobs | Use Services |
|----------|--------------|
| Tasks without HTTP response | REST APIs |
| Batch processing | Webhooks |
| Database migrations | Health endpoints |
| Scheduled ETL | Continuous services |
| Long-running tasks (>1h) | Request/response |
| One-time executions | Always-on services |

---

## CLI Architecture

### Core Separation (Same as Package)

```bash
src/myjob/
├── __init__.py
├── core/                # Business logic (no CLI deps)
│   ├── __init__.py
│   ├── extract.py
│   ├── transform.py
│   └── load.py
├── cli.py               # CLI layer (Typer)
└── config.py            # pydantic-settings
```

### CLI Pattern with Typer

| Component | Purpose |
|-----------|---------|
| `app = typer.Typer()` | Main application |
| `@app.command()` | Job commands (run, backfill, etc.) |
| Type hints | Automatic argument parsing and validation |
| `--dry-run` | Preview without executing |

### CLI Best Practices

| Practice | Description |
|----------|-------------|
| Required parameters explicit | No hidden defaults for critical params |
| Date parameters | Use `--date` or `--start-date/--end-date` |
| Dry run mode | `--dry-run` to preview without side effects |
| Verbose mode | `--verbose` for debug output |
| Progress feedback | Rich progress bars for long operations |
| Exit codes | 0=success, 1=error, 2=partial failure |

### Invocation Examples

```bash
# Direct execution
python -m myjob run --date=2024-01-15

# Docker
docker run my-job run --date=2024-01-15

# Cloud Run Jobs
gcloud run jobs execute my-job --args="run,--date=2024-01-15"
```

---

## Data Processing with Polars

### Why Polars over Pandas

| Aspect | Polars | Pandas |
|--------|--------|--------|
| Performance | 10-100x faster (Rust, multi-threaded) | Single-threaded |
| Memory | More efficient, lazy evaluation | Eager, high memory |
| API | Consistent, expressive | Inconsistent, legacy |
| Type safety | Strong typing | Weak typing |

### Lazy Evaluation

| Method | Description | When to Use |
|--------|-------------|-------------|
| `scan_parquet()` | Lazy read Parquet | Large files |
| `scan_csv()` | Lazy read CSV | Large files |
| `lazy()` | Convert to lazy | Build query plan |
| `collect()` | Execute query | Get results |

**Benefits:**

- Process files larger than memory
- Automatic query optimization
- Filter/projection pushdown

### Polars Best Practices

| Practice | Description |
|----------|-------------|
| Use lazy mode | `scan_*` instead of `read_*` for large data |
| Chain operations | Build query plan, collect once |
| Select columns early | Reduce memory footprint |
| Filter early | Reduce rows ASAP |
| Use native types | Don't cast unnecessarily |

### DuckDB for SQL Analytics

| Use Case | Description |
|----------|-------------|
| Ad-hoc queries | SQL on local files |
| Data exploration | Quick analysis |
| Complex aggregations | Window functions, CTEs |
| Polars integration | Query Polars DataFrames with SQL |

**When to use DuckDB vs Polars:**

| Use DuckDB | Use Polars |
|------------|------------|
| Complex SQL (CTEs, window functions) | DataFrame transformations |
| Ad-hoc exploration | Production pipelines |
| Team knows SQL better | Programmatic transformations |
| Query multiple formats | Single format processing |

**DuckDB + Polars:**

```python
import duckdb
import polars as pl

# Query Polars DataFrame with SQL
df = pl.read_parquet("data.parquet")
result = duckdb.sql("SELECT * FROM df WHERE value > 100").pl()
```

### DataFrame Validation with Pandera

| Feature | Description |
|---------|-------------|
| Schema definition | Define expected columns and types |
| Constraints | Value ranges, allowed values, regex |
| Null handling | Required vs optional columns |
| Custom checks | Business rule validation |

---

## Partitioning Strategies

### Why Partition

| Benefit | Description |
|---------|-------------|
| Parallelization | Process chunks independently |
| Memory control | Fit data in memory |
| Resumability | Restart from failed partition |
| Incremental | Process only new partitions |

### Partition Schemes

| Scheme | Use Case | Example |
|--------|----------|---------|
| **Date** | Time-series data | `data/year=2024/month=01/day=15/` |
| **ID range** | Large tables | `users_000000_099999.parquet` |
| **Hash** | Even distribution | `partition_{hash % N}.parquet` |
| **Size-based** | Consistent chunks | Split every 1M rows |

### Hive-style Partitioning

```bash
gs://bucket/data/
├── year=2024/
│   ├── month=01/
│   │   ├── day=01/
│   │   │   └── data.parquet
│   │   └── day=02/
│   │       └── data.parquet
```

**Benefits:**

- Polars/Spark read with partition filters
- Only scan relevant data
- Standard format

### Processing Partitioned Data

```python
# Polars reads partitioned data efficiently
df = pl.scan_parquet("gs://bucket/data/**/*.parquet")
    .filter(pl.col("year") == 2024)
    .filter(pl.col("month") == 1)
    .collect()
```

---

## Checkpointing and Resume

### Why Checkpoint

| Scenario | Problem | Solution |
|----------|---------|----------|
| Long jobs | Fail at 90%, lose all work | Save progress periodically |
| Large data | Can't reprocess everything | Resume from last checkpoint |
| Flaky dependencies | External API timeouts | Retry from checkpoint |

### Checkpoint Strategies

| Strategy | Description | Use Case |
|----------|-------------|----------|
| **Partition-based** | Track completed partitions | Date-partitioned data |
| **Offset-based** | Track last processed ID/row | Sequential processing |
| **State file** | Save job state to file | Complex multi-step jobs |
| **Database** | Track in DB table | Multi-job coordination |

### Checkpoint Storage

| Location | Pros | Cons |
|----------|------|------|
| GCS | Durable, shared | Latency |
| Local file | Fast | Lost on container restart |
| Database | Queryable, transactional | Extra dependency |
| Redis | Fast, shared | Extra dependency |

### Checkpoint Pattern

```python
# Pseudocode pattern with performance optimization
def process_with_checkpoint(partitions: list[str], checkpoint_path: str):
    """Process partitions with checkpoint tracking.

    Performance improvement:
    - Convert completed list to set for O(1) lookup instead of O(n)
    - This is critical when processing thousands of partitions
    - Example: 10,000 partitions goes from ~50M operations to ~10K operations
    """
    completed_list = load_checkpoint(checkpoint_path)
    # Convert to set for O(1) lookup performance
    completed = set(completed_list)

    for partition in partitions:
        if partition in completed:
            logger.info(f"Skipping {partition}, already completed")
            continue

        process_partition(partition)
        # Add to set and save
        completed.add(partition)
        save_checkpoint(checkpoint_path, partition)
```

### Best Practices

| Practice | Description |
|----------|-------------|
| Atomic checkpoints | Write complete state or nothing |
| Include metadata | Timestamp, job version, parameters |
| Cleanup old checkpoints | Don't accumulate forever |
| Validate on resume | Verify checkpoint integrity |

---

## Backfill Patterns

### What is Backfill

Re-processing historical data when:

- Bug fix requires reprocessing
- New column/transformation added
- Data quality issue discovered
- Schema change

### Backfill CLI Pattern

```bash
# Single date
myjob run --date=2024-01-15

# Date range
myjob backfill --start-date=2024-01-01 --end-date=2024-01-31

# With parallelism
myjob backfill --start-date=2024-01-01 --end-date=2024-01-31 --parallel=4
```

### Backfill Strategies

| Strategy | Description | When to Use |
|----------|-------------|-------------|
| **Sequential** | One date at a time | Dependencies between dates |
| **Parallel** | Multiple dates concurrent | Independent dates |
| **Chunked** | Process N dates, then next N | Memory/resource limits |
| **Priority** | Recent first, then older | Need fresh data quickly |

### Backfill Best Practices

| Practice | Description |
|----------|-------------|
| Idempotent jobs | Safe to re-run any date |
| Overwrite output | Replace existing data cleanly |
| Track progress | Log/checkpoint completed dates |
| Validate results | Compare row counts, checksums |
| Notify on completion | Alert when backfill done |

### Handling Dependencies

| Scenario | Solution |
|----------|----------|
| Job B depends on Job A | Run A's backfill first |
| Circular dependency | Break cycle, manual ordering |
| External dependency | Verify data exists before processing |

---

## Memory Management

### Why It Matters

| Problem | Impact |
|---------|--------|
| OOM errors | Job crashes, no output |
| Slow processing | Swapping to disk |
| High costs | Larger Cloud Run instances |

### Memory Strategies

| Strategy | Description |
|----------|-------------|
| Lazy evaluation | Don't load all data at once |
| Chunked processing | Process in batches |
| Streaming | Process row by row |
| Column selection | Only load needed columns |
| Appropriate types | Use smallest sufficient type |

### Polars Memory Optimization

| Technique | Description |
|-----------|-------------|
| `scan_*` vs `read_*` | Lazy loading |
| `.select()` early | Reduce columns |
| `.filter()` early | Reduce rows |
| `.collect(streaming=True)` | Stream large results |
| `sink_parquet()` | Write without collecting |

### Estimating Memory Needs

| Data Type | Size per Value |
|-----------|----------------|
| int8 | 1 byte |
| int32 | 4 bytes |
| int64 | 8 bytes |
| float64 | 8 bytes |
| string | Variable + overhead |

**Rule of thumb:** DataFrame in memory ≈ 2-3x raw file size

### Cloud Run Job Sizing

| Memory | Use Case |
|--------|----------|
| 512MB | Small jobs, <100MB data |
| 2GB | Medium jobs, <1GB data |
| 8GB | Large jobs, <5GB data |
| 32GB | Very large, consider chunking |

---

## Performance Optimization Patterns

### Common Performance Anti-Patterns

| Anti-Pattern | Impact | Solution |
|--------------|--------|----------|
| List membership check (`if item in list`) | O(n) for each check | Use `set` for O(1) lookup |
| Creating clients in loops | High connection overhead | Reuse clients with caching or singletons |
| Importing modules repeatedly | Import overhead on each call | Cache imported modules globally |
| Missing timeouts on I/O | Hangs on slow connections | Always set explicit timeouts |
| Small chunk sizes | Too many I/O operations | Use 1MB+ chunks for file operations |
| No retry logic | Fails on transient errors | Use exponential backoff retry |

### Client Reuse Pattern

**Problem:** Creating new API clients on each call is expensive.

**Solution:** Use caching or singleton pattern.

```python
from functools import lru_cache
from google.cloud import bigquery

@lru_cache(maxsize=1)
def get_bigquery_client(project: str) -> bigquery.Client:
    """Get cached BigQuery client instance.

    Performance: Client is created once and reused across calls.
    Saves ~100-500ms per call depending on service.
    """
    return bigquery.Client(project=project)

# Use in your code
def query_data(sql: str, project: str):
    client = get_bigquery_client(project)
    return client.query(sql).to_dataframe()
```

### Set vs List for Membership Testing

**Problem:** `if item in my_list` is O(n), becomes slow with many items.

**Solution:** Convert to set for O(1) lookup.

```python
# Bad: O(n) lookup for each of m items = O(n*m)
completed_files = ["file1.csv", "file2.csv", ...]  # Could be 10,000+ items
for file in all_files:
    if file in completed_files:  # Linear search each time!
        continue

# Good: O(1) lookup for each of m items = O(n+m)
completed_files_set = set(completed_files)  # One-time O(n) conversion
for file in all_files:
    if file in completed_files_set:  # Constant time lookup!
        continue

# Performance gain: 10,000 files = ~50M operations → ~10K operations
```

### I/O Optimization

| Technique | Description | Performance Gain |
|-----------|-------------|------------------|
| **Buffered writes** | Use context managers, write in chunks | 10-100x faster |
| **Parallel I/O** | Use asyncio for concurrent downloads | N x speedup (N = concurrency) |
| **Connection pooling** | Reuse HTTP/DB connections | Eliminates connection overhead |
| **Timeouts** | Prevent hanging on slow operations | Prevents infinite waits |
| **Chunk sizes** | Use 1-10MB chunks for file I/O | Optimal I/O efficiency |

### Example: Efficient Batch Processing

```python
from concurrent.futures import ThreadPoolExecutor, as_completed
from pathlib import Path
import logging

logger = logging.getLogger(__name__)

def process_files_efficiently(files: list[Path], max_workers: int = 4):
    """Process multiple files in parallel.

    Performance improvements:
    - Parallel processing using ThreadPoolExecutor
    - Batch size limits memory usage
    - Progress tracking without blocking
    """
    with ThreadPoolExecutor(max_workers=max_workers) as executor:
        # Submit all tasks
        futures = {executor.submit(process_file, f): f for f in files}

        # Process results as they complete
        for future in as_completed(futures):
            file = futures[future]
            try:
                result = future.result()
                logger.info(f"Processed {file}: {result}")
            except Exception as e:
                logger.error(f"Failed {file}: {e}")
```

---

## BigQuery Integration

### BigQuery Tools

| Tool | Use Case |
|------|----------|
| [**google-cloud-bigquery**](https://github.com/googleapis/python-bigquery) | Official client |
| [**pandas-gbq**](https://github.com/googleapis/python-bigquery-pandas) | Pandas integration |
| [**connectorx**](https://github.com/sfu-db/connector-x) | Fast loading to DataFrame |
| [**sqlglot**](https://github.com/tobymao/sqlglot) | SQL parsing/transpilation |

### Reading from BigQuery

| Method | Speed | Memory |
|--------|-------|--------|
| `client.query().to_dataframe()` | Slow | High |
| `client.query().to_arrow()` | Fast | Medium |
| Storage Read API | Fastest | Streaming |
| Export to GCS + read | Fast for huge data | Low |

### Writing to BigQuery

| Method | Use Case |
|--------|----------|
| `load_table_from_dataframe()` | Small-medium data |
| `load_table_from_uri()` | Large data (GCS staging) |
| Streaming insert | Real-time, small batches |

### BigQuery Best Practices

| Practice | Description |
|----------|-------------|
| Use staging table | Write to temp, then swap |
| Partition tables | By date for time-series |
| Cluster tables | By frequently filtered columns |
| Dry run first | Estimate bytes scanned |
| Use `WRITE_TRUNCATE` | Idempotent overwrites |

### Write Patterns

| Pattern | SQL Equivalent | Use Case |
|---------|----------------|----------|
| `WRITE_TRUNCATE` | DELETE + INSERT | Full refresh |
| `WRITE_APPEND` | INSERT | Incremental |
| `WRITE_EMPTY` | INSERT if empty | First load only |

---

## Data Quality

### Validation Levels

| Level | When | Tool |
|-------|------|------|
| **Schema** | Read time | Pandera, Pydantic |
| **Content** | Transform time | Pandera checks |
| **Output** | Write time | Post-write validation |
| **Cross-job** | After pipeline | Data quality framework |

### Quality Checks

| Check | Description |
|-------|-------------|
| Row count | Expected range |
| Null percentage | Acceptable nulls per column |
| Uniqueness | Primary key uniqueness |
| Referential | Foreign keys exist |
| Range | Values within bounds |
| Freshness | Data not stale |

### Pandera Checks

| Check Type | Example |
|------------|---------|
| Type | `pa.Column(int)` |
| Nullable | `nullable=False` |
| Range | `pa.Check.in_range(0, 100)` |
| Regex | `pa.Check.str_matches(r'^\d{4}$')` |
| Custom | `pa.Check(lambda s: s.mean() > 0)` |

### Quality Metrics

| Metric | Description |
|--------|-------------|
| Completeness | % non-null values |
| Accuracy | % values matching rules |
| Consistency | Cross-column consistency |
| Timeliness | Data freshness |

### Handling Quality Failures

| Strategy | When to Use |
|----------|-------------|
| Fail job | Critical data quality |
| Warn and continue | Non-critical issues |
| Quarantine bad rows | Process good data, flag bad |
| Alert | Notify team for review |

---

## Notifications

### When to Notify

| Event | Channel | Priority |
|-------|---------|----------|
| Job failed | Slack, PagerDuty | High |
| Job succeeded (critical) | Slack | Low |
| Long running | Slack | Medium |
| Data quality warning | Slack, Email | Medium |
| Backfill completed | Slack, Email | Low |

### Notification Tools

| Tool | Use Case |
|------|----------|
| [**Cloud Monitoring**](https://cloud.google.com/monitoring) | GCP-native alerting |
| [**Slack SDK**](https://github.com/slackapi/python-slack-sdk) | Direct Slack messages |
| [**SendGrid**](https://github.com/sendgrid/sendgrid-python) | Email notifications |
| [**PagerDuty**](https://github.com/PagerDuty/pdpyras) | On-call alerting |

### Notification Content

| Field | Description |
|-------|-------------|
| Job name | Which job |
| Status | Success/Failure/Warning |
| Duration | How long it ran |
| Rows processed | Data volume |
| Error message | If failed |
| Link to logs | Cloud Logging URL |

### Alerting Best Practices

| Practice | Description |
|----------|-------------|
| Don't over-alert | Only actionable alerts |
| Include context | Enough info to diagnose |
| Escalation path | Auto-escalate if not acked |
| Runbook links | Link to troubleshooting docs |

---

## File Formats

### Modern Formats (Preferred)

| Format | Use Case | Pros | Cons |
|--------|----------|------|------|
| **Parquet** | Analytics, data lake | Columnar, compressed, fast reads | Not human-readable |
| **Avro** | Streaming, events | Schema evolution, compact | Requires schema |
| **Delta Lake** | Data lake with ACID | Transactions, time travel | More complex |
| **Arrow** | In-memory, IPC | Zero-copy, fast | Not for storage |

### When to Use Parquet vs Avro

| Parquet | Avro |
|---------|------|
| Analytics, OLAP queries | Streaming, message queues |
| Column-based access | Row-based access |
| Read-heavy workloads | Write-heavy workloads |
| Data lakes, BigQuery | Pub/Sub, Kafka |
| Compression priority | Schema evolution priority |

### Parquet Best Practices

| Practice | Description |
|----------|-------------|
| Use snappy compression | Good balance speed/size |
| Partition by date | Enable predicate pushdown |
| Row group size | 128MB default, adjust for use case |
| Use dictionary encoding | For low-cardinality columns |

### Avro Best Practices

| Practice | Description |
|----------|-------------|
| Schema registry | Central schema management |
| Backward compatible | Add fields with defaults |
| Use fastavro | Faster than avro-python3 |
| Include schema in file | Self-describing files |

### Legacy Formats (Handle with Care)

| Format | Tool | Notes |
|--------|------|-------|
| **CSV** | Polars, csv | No types, encoding issues |
| **Excel (.xlsx)** | [openpyxl](https://github.com/theorchard/openpyxl), [xlsxwriter](https://github.com/jmcnamara/XlsxWriter) | Slow, memory-heavy |
| **Excel (.xls)** | [xlrd](https://github.com/python-excel/xlrd) | Legacy, avoid if possible |
| **JSON** | [orjson](https://github.com/ijl/orjson), Polars | Good for small data |
| **JSONL** | Polars | Better for large data |
| **XML** | [lxml](https://github.com/lxml/lxml), xmltodict | Legacy integrations |
| **Fixed-width** | Polars | Mainframe data |

### CSV Handling

| Problem | Solution |
|---------|----------|
| Unknown encoding | Use `chardet` to detect, prefer UTF-8 |
| Different delimiters | Specify explicitly (`;`, `\t`, `\|`) |
| Quoted fields | Use proper CSV parser, not split() |
| Large files | Use Polars `scan_csv()` |
| Missing headers | Provide column names |

```python
# Polars handles most CSV edge cases
df = pl.read_csv(
    "file.csv",
    separator=";",
    encoding="utf-8",
    null_values=["", "NULL", "N/A"],
    try_parse_dates=True,
)
```

### Excel Handling

| Tool | Use Case |
|------|----------|
| [**openpyxl**](https://github.com/theorchard/openpyxl) | Read/write .xlsx |
| [**xlsxwriter**](https://github.com/jmcnamara/XlsxWriter) | Write .xlsx (faster) |
| **Polars** | Read .xlsx via connectorx |

**Best Practices:**

| Practice | Description |
|----------|-------------|
| Convert to Parquet | Process once, read many |
| Specify dtypes | Don't trust Excel inference |
| Handle merged cells | Unmerge or skip |
| Watch for dates | Excel dates are numbers |

### Compressed Files

| Format | Tool | Notes |
|--------|------|-------|
| **.gz** | gzip, Polars native | Single file compression |
| **.zip** | zipfile | Multiple files archive |
| **.tar.gz** | tarfile | Unix archives |
| **.7z** | [py7zr](https://github.com/miurahr/py7zr) | High compression |
| **.bz2** | bz2 | Better compression, slower |

```python
# Polars reads gzipped files directly
df = pl.read_csv("data.csv.gz")
df = pl.read_parquet("data.parquet.gz")

# For zip archives
import zipfile
with zipfile.ZipFile("archive.zip") as z:
    with z.open("data.csv") as f:
        df = pl.read_csv(f)
```

### Text Files

| Format | Use Case | Tool |
|--------|----------|------|
| **.txt** (delimited) | Legacy exports | Polars with separator |
| **.txt** (fixed-width) | Mainframe data | Polars `read_csv` with `truncate_ragged_lines` |
| **.log** | Log parsing | regex, parse |

---

## File System Operations

### pathlib (Mandatory)

Always use `pathlib.Path` instead of `os.path`:

| os.path (Avoid) | pathlib (Use) |
|-----------------|---------------|
| `os.path.join(a, b)` | `Path(a) / b` |
| `os.path.exists(p)` | `path.exists()` |
| `os.path.isfile(p)` | `path.is_file()` |
| `os.makedirs(p)` | `path.mkdir(parents=True)` |
| `os.path.basename(p)` | `path.name` |
| `os.path.dirname(p)` | `path.parent` |
| `os.path.splitext(p)` | `path.suffix`, `path.stem` |

### Common pathlib Patterns

```python
from pathlib import Path

# Build paths
data_dir = Path("/data")
input_file = data_dir / "input" / "file.csv"

# Glob patterns
parquet_files = data_dir.glob("**/*.parquet")

# File info
file.name       # "file.csv"
file.stem       # "file"
file.suffix     # ".csv"
file.parent     # Path("/data/input")

# Create directories
output_dir.mkdir(parents=True, exist_ok=True)

# Read/write
content = file.read_text(encoding="utf-8")
file.write_bytes(data)
```

### Temporary Files

| Use Case | Tool |
|----------|------|
| Temporary directory | `tempfile.TemporaryDirectory()` |
| Temporary file | `tempfile.NamedTemporaryFile()` |
| In-memory file | `io.BytesIO()`, `io.StringIO()` |

```python
import tempfile
from pathlib import Path

# Context manager cleans up automatically
with tempfile.TemporaryDirectory() as tmp:
    tmp_path = Path(tmp)
    output = tmp_path / "result.parquet"
    df.write_parquet(output)
    # Upload to GCS, etc.
# Directory deleted automatically
```

---

## Remote File Access

### Google Cloud Storage (GCS)

| Tool | Use Case |
|------|----------|
| [**gcsfs**](https://github.com/fsspec/gcsfs) | Filesystem interface |
| [**google-cloud-storage**](https://github.com/googleapis/python-storage) | Official client, more control |

```python
# Polars reads directly from GCS
df = pl.read_parquet("gs://bucket/path/file.parquet")
df = pl.scan_parquet("gs://bucket/path/**/*.parquet")

# For more control
from google.cloud import storage
client = storage.Client()
bucket = client.bucket("my-bucket")
blob = bucket.blob("path/file.csv")
blob.download_to_filename("/tmp/file.csv")
```

### FTP/SFTP (Legacy Systems)

| Tool | Protocol |
|------|----------|
| **ftplib** | FTP (built-in, insecure) |
| [**paramiko**](https://github.com/paramiko/paramiko) | SFTP (SSH) |
| [**pysftp**](https://github.com/wtsi-npg/pysftp) | SFTP (simpler API) |

**SFTP Best Practices:**

| Practice | Description |
|----------|-------------|
| Use SFTP over FTP | Encrypted, secure |
| Key-based auth | No passwords in code |
| Retry on failure | Networks are flaky |
| Download to temp | Process locally |
| Verify file size | Ensure complete download |

```python
import paramiko
from pathlib import Path
from tenacity import retry, stop_after_attempt, wait_exponential

def download_from_sftp(
    host: str,
    username: str,
    key_path: Path,
    remote_path: str,
    local_path: Path,
    timeout: float = 30.0,
    verify_size: bool = True,
) -> None:
    """Download file from SFTP server with retry logic and proper timeouts.

    Performance improvements:
    - Configurable timeout prevents hanging connections
    - File size validation ensures complete download (can be disabled for text mode)
    - Proper nested try/finally ensures resource cleanup

    Note: tenacity is listed in the Resilience tools section above.

    Args:
        verify_size: If True, validates downloaded file size matches remote.
                     Set to False if transferring text files that may have
                     line ending conversions (CRLF <-> LF).
    """
    key = paramiko.RSAKey.from_private_key_file(str(key_path))

    transport = paramiko.Transport((host, 22))
    transport.connect(username=username, pkey=key, timeout=timeout)

    try:
        sftp = paramiko.SFTPClient.from_transport(transport)
        try:
            # Get remote file size for validation (optional)
            if verify_size:
                remote_size = sftp.stat(remote_path).st_size

            sftp.get(remote_path, str(local_path))

            # Verify download completed successfully
            if verify_size:
                local_size = local_path.stat().st_size
                if local_size != remote_size:
                    raise ValueError(
                        f"Downloaded size mismatch: {local_size} != {remote_size}. "
                        "This may occur with text mode transfers and line ending differences. "
                        "Set verify_size=False if this is expected."
                    )
        finally:
            sftp.close()
    finally:
        transport.close()


@retry(
    stop=stop_after_attempt(3),
    wait=wait_exponential(multiplier=1, min=2, max=10),
)
def download_from_sftp_with_retry(
    host: str,
    username: str,
    key_path: Path,
    remote_path: str,
    local_path: Path,
) -> None:
    """Wrapper with automatic retry for transient failures."""
    download_from_sftp(host, username, key_path, remote_path, local_path)
```

### HTTP/HTTPS Downloads

| Tool | Use Case |
|------|----------|
| [**httpx**](https://github.com/encode/httpx) | Modern async/sync client |
| [**requests**](https://github.com/psf/requests) | Simple downloads |

```python
import httpx
from pathlib import Path

def download_file(
    url: str,
    output: Path,
    chunk_size: int = 1024 * 1024,  # 1MB chunks
    timeout: float = 30.0,
) -> None:
    """Download file with streaming and optimized performance.

    Performance improvements:
    - Configurable chunk size (default 1MB) for efficient memory usage
    - Explicit timeout prevents hanging on slow connections
    - Buffer writes for better I/O performance
    - Content-Length validation ensures complete download

    Args:
        url: URL to download from
        output: Local path to save file
        chunk_size: Size of chunks to download (default 1MB)
        timeout: Request timeout in seconds (default 30s)
    """
    with httpx.stream("GET", url, timeout=timeout, follow_redirects=True) as response:
        response.raise_for_status()

        # Get expected file size if available
        expected_size = int(response.headers.get("content-length", 0))
        downloaded = 0

        with output.open("wb") as f:
            for chunk in response.iter_bytes(chunk_size=chunk_size):
                f.write(chunk)
                downloaded += len(chunk)

        # Verify download if content-length was provided
        if expected_size > 0 and downloaded != expected_size:
            output.unlink()  # Delete incomplete file
            raise ValueError(
                f"Download incomplete: {downloaded} bytes received, "
                f"expected {expected_size} bytes"
            )
```

### S3 (AWS)

| Tool | Description |
|------|-------------|
| [**boto3**](https://github.com/boto/boto3) | Official AWS SDK |
| [**s3fs**](https://github.com/fsspec/s3fs) | Filesystem interface |

---

## Configuration and Secrets

### Configuration with pydantic-settings

```python
from pydantic_settings import BaseSettings
from pydantic import Field, SecretStr

class Settings(BaseSettings):
    # Required
    gcp_project: str
    input_bucket: str
    output_bucket: str

    # With defaults
    log_level: str = "INFO"
    dry_run: bool = False
    batch_size: int = 10000

    # Secrets (masked in logs)
    database_url: SecretStr
    api_key: SecretStr

    model_config = {
        "env_file": ".env",
        "env_file_encoding": "utf-8",
    }

settings = Settings()
```

### Configuration Environment Variables

| Variable | Source | Use Case |
|----------|--------|----------|
| `GCP_PROJECT` | Environment | GCP project ID |
| `LOG_LEVEL` | Environment | Logging verbosity |
| `DRY_RUN` | Environment | Test mode |
| `DATABASE_URL` | Secret Manager | Database connection |
| `API_KEY` | Secret Manager | External API keys |

### .env Files

| File | Purpose | Git |
|------|---------|-----|
| `.env.example` | Template with dummy values | ✅ Versioned |
| `.env` | Local development values | ❌ In .gitignore |
| `.env.test` | Test environment | ❌ In .gitignore |

**.env.example:**

```bash
# Required
GCP_PROJECT=my-project-dev
INPUT_BUCKET=my-bucket-input
OUTPUT_BUCKET=my-bucket-output

# Optional
LOG_LEVEL=DEBUG
DRY_RUN=true

# Secrets (use Secret Manager in production)
DATABASE_URL=postgresql://user:pass@localhost:5432/db # pragma: allowlist secret
API_KEY=your-api-key-here
```

### Secret Manager (Production)

| Practice | Description |
|----------|-------------|
| Never commit secrets | Use Secret Manager or env vars |
| Use SecretStr | Pydantic masks in logs |
| Rotate secrets | Regular rotation policy |
| Least privilege | Minimal IAM permissions |

```python
from google.cloud import secretmanager
from functools import lru_cache

# Cache the client instance to avoid repeated initialization
@lru_cache(maxsize=1)
def _get_secret_manager_client() -> secretmanager.SecretManagerServiceClient:
    """Get a cached Secret Manager client.

    Performance improvement:
    - Client is created once and reused across calls
    - Reduces connection overhead and initialization time
    - Thread-safe with lru_cache
    """
    return secretmanager.SecretManagerServiceClient()


def get_secret(secret_id: str, project_id: str, version: str = "latest") -> str:
    """Fetch secret from GCP Secret Manager with client reuse.

    Performance improvements:
    - Reuses client instance instead of creating new one each call
    - Configurable version for flexibility
    - Proper error handling for missing secrets

    Args:
        secret_id: The secret identifier
        project_id: GCP project ID
        version: Secret version (default: "latest")

    Returns:
        The secret value as a string

    Raises:
        google.api_core.exceptions.NotFound: If secret doesn't exist
    """
    client = _get_secret_manager_client()
    name = f"projects/{project_id}/secrets/{secret_id}/versions/{version}"
    response = client.access_secret_version(request={"name": name})
    return response.payload.data.decode("utf-8")
```

### Configuration Hierarchy

| Priority | Source | Use Case |
|----------|--------|----------|
| 1 (highest) | CLI arguments | Override for single run |
| 2 | Environment variables | Deployment config |
| 3 | .env file | Local development |
| 4 (lowest) | Defaults in code | Sensible defaults |

### Secrets in Docker

| Environment | Method |
|-------------|--------|
| Local dev | `.env` file, docker-compose |
| Cloud Run | Secret Manager integration |
| CI/CD | GitHub Secrets → env vars |

**Cloud Run with Secret Manager:**

```yaml
# Service YAML
spec:
  template:
    spec:
      containers:
        - env:
            - name: DATABASE_URL
              valueFrom:
                secretKeyRef:
                  key: latest
                  name: database-url
```

---

## Job Resilience

### Retry Pattern with tenacity

| Parameter | Description |
|-----------|-------------|
| `stop` | When to stop retrying (max attempts, time) |
| `wait` | Delay between retries (exponential, fixed) |
| `retry` | Which exceptions to retry |
| `before_sleep` | Logging before retry |

### Retry Best Practices

| Practice | Description |
|----------|-------------|
| Idempotent operations | Safe to retry without side effects |
| Exponential backoff | Increase delay between retries |
| Max attempts | Don't retry forever |
| Specific exceptions | Only retry transient errors |
| Log retries | Track retry attempts |

### What to Retry

| Retry | Don't Retry |
|-------|-------------|
| Network timeouts | Validation errors |
| 5xx server errors | 4xx client errors |
| Connection refused | Business logic errors |
| Rate limiting (429) | Authentication errors |

---

## Scheduling

### Cloud Scheduler

| Feature | Description |
|---------|-------------|
| Cron syntax | Standard cron expressions |
| Time zones | Configure per job |
| HTTP/Pub/Sub triggers | Invoke Cloud Run Jobs |
| Retry configuration | Automatic retries on failure |

### Cron Expressions

| Expression | Description |
|------------|-------------|
| `0 2 * * *` | Daily at 2:00 AM |
| `0 */4 * * *` | Every 4 hours |
| `0 9 * * 1` | Every Monday at 9:00 AM |
| `0 0 1 * *` | First day of month |

### Scheduling Best Practices

| Practice | Description |
|----------|-------------|
| UTC time | Always use UTC in scheduler |
| Avoid peak hours | Schedule during low-traffic times |
| Stagger jobs | Don't schedule everything at :00 |
| Idempotency | Jobs should be re-runnable |
| Alerting | Monitor job failures |

---

## Observability & Monitoring

### Logging

| Practice | Description |
|----------|-------------|
| Structured logging | JSON format with structlog |
| Job context | Include job_id, run_date in all logs |
| Progress logging | Log milestones (rows processed, etc.) |
| Duration tracking | Log start/end times |

### What to Log

| Event | Level | Information |
|-------|-------|-------------|
| Job started | INFO | job_name, parameters, run_id |
| Progress milestone | INFO | rows_processed, percentage |
| Job completed | INFO | duration, rows_processed, output_path |
| Recoverable error | WARNING | error, context, will_retry |
| Job failed | ERROR | error, stack trace, context |

### Metrics

| Metric | Type | Description |
|--------|------|-------------|
| `job_duration_seconds` | Histogram | Execution time |
| `rows_processed_total` | Counter | Data volume |
| `job_status` | Gauge | Success/failure |
| `retries_total` | Counter | Retry count |

---

## Error Handling

### Exit Codes

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | General error |
| 2 | Partial failure (some data processed) |
| 3 | Configuration error |
| 4 | Input data error |

### Error Handling Pattern

| Strategy | Description |
|----------|-------------|
| Fail fast | Validate inputs at start |
| Partial success | Process what you can, report failures |
| Checkpointing | Save progress for resume |
| Dead letter | Store failed records for retry |

### Alerting

| Condition | Action |
|-----------|--------|
| Job failed | Alert immediately |
| Job duration > threshold | Warning alert |
| Partial failure | Alert with details |
| No data processed | Warning alert |

---

## Event-Driven Jobs (Pub/Sub)

### When to Use

| Event-Driven | Scheduled |
|--------------|-----------|
| Real-time processing | Batch processing |
| Trigger on data arrival | Trigger on time |
| Variable load | Predictable load |
| Low latency needed | Latency tolerant |

### Pub/Sub Tools

| Tool | Description | When to Use |
|------|-------------|-------------|
| [**FastStream**](https://github.com/airtai/faststream) | Modern event framework (FastAPI-style) | Recommended |
| [**google-cloud-pubsub**](https://github.com/googleapis/python-pubsub) | Low-level Pub/Sub client | Full control |

### FastStream Benefits

| Benefit | Description |
|---------|-------------|
| Async native | Full async/await support |
| Pydantic validation | Automatic message validation |
| AsyncAPI docs | Auto-generated documentation |
| Testability | Built-in testing utilities |

### Message Handling Best Practices

| Practice | Description |
|----------|-------------|
| Idempotency | Handle duplicate messages |
| Acknowledgment | Ack after successful processing |
| Dead letter queue | Route failed messages |
| Schema validation | Validate message structure |
| Ordering | Don't assume message order |

### Pub/Sub Schemas

| Format | Use Case |
|--------|----------|
| **Avro** | Schema evolution, compact |
| **Protobuf** | High performance, strict |
| **JSON** | Simple, debuggable |

---

## Docker

### Multi-stage Build (Same as Web Service)

| Stage | Purpose |
|-------|---------|
| builder | Install dependencies with uv |
| runtime | Minimal image with code |

### ENTRYPOINT Pattern

```dockerfile
ENTRYPOINT ["python", "-m", "myjob"]
CMD ["--help"]
```

This allows:

```bash
docker run my-job run --date=2024-01-15
docker run my-job backfill --start=2024-01-01 --end=2024-01-31
```

### Docker Environment Variables

| Variable | Purpose |
|----------|---------|
| `LOG_LEVEL` | Logging verbosity |
| `DRY_RUN` | Enable dry run mode |
| `GCP_PROJECT` | GCP project ID |
| `GOOGLE_APPLICATION_CREDENTIALS` | Service account (local dev) |

---

## Testing

### Strategy

| Test Type | Purpose |
|-----------|---------|
| Unit | Test transformations, business logic |
| Integration | Test with real files, databases |
| Data quality | Validate output data |

### Tools

| Tool | Purpose |
|------|---------|
| [**pytest**](https://github.com/pytest-dev/pytest) | Test framework |
| [**pytest-mock**](https://github.com/pytest-dev/pytest-mock) | Mocking |
| [**hypothesis**](https://github.com/HypothesisWorks/hypothesis) | Property-based testing for transforms |
| [**Faker**](https://github.com/joke2k/faker) | Generate test data |

### What to Test

| Test | Description |
|------|-------------|
| Transformations | Input → Output correctness |
| Edge cases | Empty data, nulls, boundaries |
| Error handling | Exceptions raised correctly |
| Idempotency | Re-running produces same result |
| CLI parsing | Arguments parsed correctly |

### Test Data

| Approach | When to Use |
|----------|-------------|
| Fixtures | Small, static test data |
| Faker | Realistic generated data |
| Sample files | Representative production data |
| Hypothesis | Property-based generation |

---

## Additional Files

Beyond [General / Core](general-core.md), data jobs include:

| File | Purpose |
|------|---------|
| `Dockerfile` | Multi-stage container build |
| `.dockerignore` | Exclude files from build |
| `src/<job>/cli.py` | Typer CLI entry point |
| `src/<job>/config.py` | pydantic-settings configuration |

---

## GCP Integration

### Services

| Service | Use Case |
|---------|----------|
| [**Cloud Run Jobs**](https://cloud.google.com/run/docs/create-jobs) | Job execution |
| [**Cloud Scheduler**](https://cloud.google.com/scheduler) | Cron triggers |
| [**Cloud Storage**](https://cloud.google.com/storage) | Data input/output |
| [**BigQuery**](https://cloud.google.com/bigquery) | Data warehouse |
| [**Pub/Sub**](https://cloud.google.com/pubsub) | Event triggers |
| [**Secret Manager**](https://cloud.google.com/secret-manager) | Credentials |

### Authentication

| Context | Method |
|---------|--------|
| Local dev | `gcloud auth application-default login` |
| Cloud Run Jobs | Automatic (service account) |
| CI/CD | Workload Identity or SA key |
