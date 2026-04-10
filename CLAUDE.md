# Webhook Trigger

## Project Overview
- **Stack**: Python 3.11, requests, tqdm, concurrent.futures (ThreadPoolExecutor)
- **Description**: Parallel webhook trigger utility that reads HTTP endpoints from CSV files and fires them with configurable rate limiting, retry logic, and adaptive throttling. Deployable on Railway/Coolify or run locally via CLI.

## File Organization
- Never save working files to root folder
- `webhook_trigger.py` - Single-file application with all logic
- `http_triggers.csv` - CSV input with columns: webhook_url, method, payload, header
- `Procfile` / `railway.json` - Deployment configs for Railway
- `requirements.txt` / `runtime.txt` - Python dependencies and version

## Key Architecture
- `RateLimiter` class with adaptive rate adjustment based on error window
- `ResultsTracker` saves JSON results to `webhook_results.json`
- Supports multiple CSV files in deployment mode (`AUTO` flag)
- Config via environment variables: `BASE_RATE_LIMIT`, `MAX_WORKERS`, `SKIP_ROWS`, etc.
- Two modes: deployment (env var driven) and local (argparse CLI)

## Build & Test
```bash
pip install -r requirements.txt                    # Install dependencies
python webhook_trigger.py http_triggers.csv        # Local: run with CSV file
python webhook_trigger.py http_triggers.csv --keep-alive  # Keep process alive after completion
```

## n8n Workflow Automation

This project can trigger and receive n8n workflows at `https://n8n.chinmayramraika.in`.

- **Webhook URL:** Set in `N8N_WEBHOOK_URL` env var
- **API Key:** Set in `N8N_API_KEY` env var (unique per project)
- **Auth Header:** `X-API-Key: <N8N_API_KEY>`
- **Workflow repo:** github.com/Cramraika/n8n-workflows (private)

## Security Rules
- NEVER hardcode API keys, secrets, or credentials in any file
- NEVER pass credentials as inline env vars in Bash commands
- NEVER commit .env, .claude/settings.local.json, or .mcp.json to git
