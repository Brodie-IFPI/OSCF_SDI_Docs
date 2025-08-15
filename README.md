# OSCF SDI Interface
Version 1.1.0

## Overview

OSCF (Open Scraper Crawler Framework) SDI (Simple Database Interactions) provides a Python interface to submit crawler results to IFPI's GCPN-backed API and retrieve operational insights. It now includes:

- Batch sending with retry/backoff
- Optional local database buffering and recovery
- API-driven crawler result filtering (filenames/URLs)
- Crawler result status lookups (by URLs or crawler names)
- Automatic authentication token refresh

Built with Python and requests, with loguru for structured logging.

For access, you need a GCPN account. Contact IFPI’s London development team.

Note for release on 15/08 - this is a major update im releasing with very limited user testing.
If there are any huge issues with the local db or filtering, these can be disabled and hopefully everything lives.
Besides from this, anything critical breaking I will either be hotfixing, blocking off the feature or worst case, reverting.
Best of luck!

## Quick Start

### Prerequisites

- Python 3.10 or higher (match case is in use)
- `pip` for package management

### Installation

- From source or your package index. For Git install (repo is private, please refer to distributed email).

  ```sh
  pip install git+https://github.com/Brodie-IFPI:[KEY]@github.com/OSCF_SDI.git#egg=sdi_interface
  ```

  If the repo is private, use a personal access token as appropriate.

### Basic usage

```python
from sdi_interface import SDI

# Enable local DB buffering and API-side filtering by default
sdi = SDI(
    username='you@ifpi.org',
    password='your-password',
    crawler_name='MyCrawler',
    use_local_db=True,
    filtering_enabled=True,
)

# Insert a single crawler result (runs on background threads)
sdi.insert_crawler_result(
    url="https://example.com/path/file.mp3",
    filename="file.mp3",
    referrer="https://google.com",
    description="Auto-detected sample",
    uploader="DetectorBot",
)


```

### Status lookups

```python
# By URLs
result_by_urls = sdi.get_crawler_results_status_for_urls([
    "https://example.com/path/file.mp3",
    "https://another.com/a"
])

# By crawler names
result_by_crawlers = sdi.get_crawler_results_status_for_crawlers([
    "MyCrawler",
    "PartnerCrawler"
])

# Or, if local db is active
results = sdi.get_crawler_results_status()

```

## Key capabilities

- **Batch sending and retry**
  - Results are queued and sent in batches (`Defaults.BATCH_SIZE`)
  - Automatic backoff and retry up to `Defaults.TRANSACTION_RETRY_LIMIT`
  - Partial batch failures are re-queued; successes are acknowledged

- **Local database buffering (optional)**
  - Durable queue for intermittent connectivity
  - Automatic startup via `DatabaseClientWrapper`
  - Periodic polling of ready-to-send items with unlock delay (`Defaults.UNLOCK_DELAY`)
  - Configurable path via `DEFAULT_LOCAL_DATABASE_PATH` env var
  - SQLite database is sitting in local_db service - dont be afriad, take a look! 

- **Crawler result filtering**
  - Pulls filename and URL filters from API at startup
  - Filters preempt submissions; when using local DB, filtered records are retained locally with a filtered flag

- **Status insights**
  - Query aggregate status for URLs or crawler names
  - Responses include mapped status descriptions when available

- **Authentication resilience**
  - Authenticator handles initial login and header provisioning
  - Token refresh on demand via `refresh_token()`

## Configuration

See `sdi_interface/constants.py` for defaults. I would only recommend changing these if you know what you're doing:

- `Defaults.BATCH_SIZE` (default 100)
- `Defaults.TRANSACTION_RETRY_LIMIT` (default 3)
- `Defaults.UNLOCK_DELAY` (seconds between record retries)
- `Defaults.DEFAULT_LOCAL_DATABASE_PATH` (SQLite path; override via env var `DEFAULT_LOCAL_DATABASE_PATH`)

## Module guide (high level)

- `sdi_interface/sdi.py`
  - SDI facade with background processing and convenience methods

- `sdi_interface/services`
  - `crawler_results_processor.py`: batch queue, retries, DB polling
  - `crawler_result_filtering.py`: API filters for filenames/URLs
  - `crawler_results_status.py`: status lookups and formatting helpers
  - `api/`
    - `authenticator.py`: token acquisition and refresh
    - `endpoints/`: typed endpoints for auth, crawler results, filters, status
  - `local_db/`
    - `db_client`, `db_controller`, `db_client_wrapper`: embedded service lifecycle
    - `orm.py`: SQLite models (`IngestedResult`, `SystemVariable`)

- `sdi_interface/utils`
  - `general.py`: Utils and `force_thread`

## Notes

- All API interactions run on background threads; no need to await.
- Logging uses loguru; adjust sink/level as needed in your application.

## Roadmap

- Enhanced success/error surfacing and metrics
- Additional endpoints for operational visibility and configuration
- Automated endpoint schema updates
- Extended response digestion and resilience to transient API failures
- OSCF DBUE (IFPI's internal database connection tool) integration
- GUI interface for uploading via SDI and viewing results
- Greater control over logs (I know they spam like crazy)
