# Environments - Webhook Trigger

**Repo**: SMPL562/webhook_trigger
**Stack**: Python 3.11, requests, tqdm, concurrent.futures

---

## Local Development

### Prerequisites

- Python 3.11+
- pip

### Installation

```bash
git clone https://github.com/SMPL562/webhook_trigger.git
cd webhook_trigger

# Create virtual environment (recommended)
python -m venv venv
source venv/bin/activate  # macOS/Linux
# venv\Scripts\activate   # Windows

# Install dependencies
pip install -r requirements.txt
```

### Dependencies

- `requests==2.31.0` -- HTTP client
- `tqdm==4.66.1` -- Progress bar

### Running Locally (CLI Mode)

```bash
# Basic usage -- provide a CSV file
python webhook_trigger.py http_triggers.csv

# Keep process alive after completion
python webhook_trigger.py http_triggers.csv --keep-alive

# Use a custom JSON config file
python webhook_trigger.py http_triggers.csv --config custom_config.json
```

### CSV Format

The input CSV file (`http_triggers.csv`) must have these columns:

| Column | Required | Description |
|--------|----------|-------------|
| `webhook_url` | Yes | Target URL |
| `method` | No | HTTP method (defaults to POST if payload exists, else GET) |
| `payload` | No | JSON request body |
| `header` | No | JSON headers object |

### Environment Setup

```bash
cp .env.example .env
# Edit .env with your configuration
```

---

## Environment Variables

All variables are optional. Defaults are used if not set.

| Variable | Default | Description |
|----------|---------|-------------|
| `BASE_RATE_LIMIT` | `3` | Minimum delay between requests (seconds) |
| `STARTING_RATE_LIMIT` | `3` | Initial rate limit |
| `MAX_RATE_LIMIT` | `5.0` | Maximum delay when errors occur |
| `WINDOW_SIZE` | `20` | Number of requests to analyze for rate adjustment |
| `ERROR_THRESHOLD` | `0.3` | Error rate threshold (30%) to trigger slowdown |
| `MAX_WORKERS` | `3` | Maximum parallel threads |
| `SKIP_ROWS` | `0` | Number of CSV rows to skip |
| `KEEP_ALIVE` | `true` | Keep process running after completion |
| `DEPLOYMENT_MODE` | (empty) | Set to any value to enable deployment mode |

---

## Production / Deployment

**Current status**: Deployable on Railway or Coolify.

### Railway Deployment

The repo includes Railway-specific files:

- `Procfile`: `worker: python webhook_trigger.py`
- `railway.json`: Nixpacks builder, restart on failure (max 3 retries)
- `runtime.txt`: Specifies Python version

In deployment mode (when `DEPLOYMENT_MODE` or `RAILWAY_ENVIRONMENT` is set), the app:
1. Reads from multiple CSV files automatically (`http_triggers.csv`, `http_triggers 2.csv`, etc.)
2. Uses environment variables for all configuration
3. Keeps the container alive after processing (if `KEEP_ALIVE=true`)

### Deployment Mode vs Local Mode

| Feature | Local (CLI) | Deployment (Railway/Coolify) |
|---------|------------|------------------------------|
| CSV input | CLI argument | Auto-discovers CSV files |
| Config | CLI args + env vars | Environment variables only |
| Keep alive | `--keep-alive` flag | `KEEP_ALIVE` env var |
| Activation | Always | When `DEPLOYMENT_MODE` or `RAILWAY_ENVIRONMENT` is set |

---

## CI/CD

**Pipeline**: GitHub Actions on `ubuntu-latest` (Python 3.11)
**Triggers**: Push/PR to `main` or `master`

| Step | Description |
|------|-------------|
| Install | `pip install flake8 pip-audit` + `requirements.txt` |
| Lint (flake8) | Checks for syntax errors and undefined names (`E9,F63,F7,F82`) |
| Security Audit | `pip-audit` (continue on error) |
| Env Validation | Checks for `requirements.txt`, `webhook_trigger.py`, `runtime.txt` |

---

## Troubleshooting

**"CSV file not found"**
- Ensure the CSV file path is correct relative to where you run the script
- In deployment mode, place CSV files in the working directory

**Rate limit errors from target APIs**
- Increase `BASE_RATE_LIMIT` to add more delay between requests
- Reduce `MAX_WORKERS` to lower concurrency
- The adaptive rate limiter will automatically slow down when error rate exceeds `ERROR_THRESHOLD`

**Results tracking**
- Results are saved to `webhook_results.json` after processing
- Logs are written to `webhook_trigger.log` and stdout

**Process exits immediately**
- Set `KEEP_ALIVE=true` or use `--keep-alive` flag
- Check logs for errors during CSV parsing
