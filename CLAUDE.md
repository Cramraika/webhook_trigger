# Webhook Trigger

## Claude Preamble
<!-- VERSION: 2026-04-18-v7 -->
<!-- SYNC-SOURCE: ~/.claude/conventions/universal-claudemd.md -->

**Universal laws** (§4), **MCP routing** (§6), **Drift protocol** (§11), **Dynamic maintenance** (§14), **Capability resolution** (§15), **Subagent SKILL POLICY** (§16), **Session continuity** (§17), **Decision queue** (§17.a), **Attestation** (§18), **Cite format** (§19), **Three-way disagreement** (§20), **Pre-conditions** (§21), **Provenance markers** (§22), **Redaction rules** (§23), **Token budget** (§24), **Tool-failure fallback** (§25), **Prompt-injection rule** (§26), **Append-only discipline** (§27), **BLOCKED_BY markers** (§28), **Stop-loss ladder** (§29), **Business-invariant checks** (§30).

**All preloaded from** `~/.claude/conventions/universal-claudemd.md`. Before significant work: read universal file sections relevant to the task. Re-audit status: next due 2026-07-18. Sync script: `~/.claude/scripts/sync-preambles.py`.

## Product Overview

| Product | Bulk Webhook Firing Utility |
|---------|----------------------------|
| **What it does** | Reads HTTP endpoints from CSV files and fires them in parallel with configurable rate limiting, retry logic, and adaptive throttling. Used to trigger bulk operations in LeadSquared, n8n, or any HTTP-based system. |
| **Who uses it** | Operations/IT team at Coding Ninjas when they need to trigger hundreds or thousands of webhooks in bulk (e.g., mass CRM updates, bulk notifications, data migration triggers). Also used in CI/CD pipelines via deployment mode. |
| **Status** | Production. Runs locally via CLI or deployed on Railway/Coolify for scheduled/automated runs. |
| **Organization** | SMPL562 |

## Product Features and User Journeys

### 1. Bulk Webhook Execution (Core Use Case)
- **User journey**: Operator prepares a CSV file (`http_triggers.csv`) with columns: `webhook_url`, `method`, `payload`, `header`. Runs the script pointing to the CSV. Script fires all webhooks in parallel with rate limiting. Progress bar (tqdm) shows real-time status. Results saved to `webhook_results.json`.
- **Success signals**: All webhooks return 2xx. Results JSON shows 100% success rate. Execution completes within expected time window.
- **Failure signals**: High error rate triggers adaptive throttling (rate slows down). Specific rows fail repeatedly after retries. Target API rate-limits or blocks the requests.

### 2. Adaptive Rate Limiting (Reliability)
- **User journey**: Script starts at `BASE_RATE_LIMIT` speed. If errors spike within the error window, the `RateLimiter` automatically slows down. When errors clear, rate returns to normal.
- **Success signals**: Error rate stays below threshold. Adaptive throttling prevents API bans. No manual intervention needed.
- **Failure signals**: Rate drops to near-zero (target API is down, not rate-limited). Throttling doesn't recover after transient errors.

### 3. Deployment Mode (Automated/Scheduled Runs)
- **User journey**: Script is deployed on Railway/Coolify with `AUTO` flag. It scans for all CSV files in the directory and processes them sequentially. Environment variables control rate limit, workers, skip rows, etc.
- **Success signals**: All CSVs processed without manual intervention. Results persisted. Process exits cleanly (or stays alive with `--keep-alive`).
- **Failure signals**: Process crashes mid-CSV. No notification on failure. Results not saved before crash.

### 4. Resume/Skip (Interrupted Runs)
- **User journey**: A previous run was interrupted at row 500. Operator reruns with `SKIP_ROWS=500` to resume from where it left off.
- **Success signals**: Script skips the first 500 rows and continues. No duplicate webhook fires.
- **Failure signals**: Wrong skip count causes duplicates or missed rows.

## Known Product Limitations
- No built-in deduplication -- operator must manage skip counts manually
- No persistent state -- if the process crashes, you rely on `webhook_results.json` to determine where to resume
- No email/Slack notification on completion or failure
- Single-threaded rate limiter may bottleneck at very high concurrency

---

## Technical Reference

### Stack
- Python 3.11, requests, tqdm, concurrent.futures (ThreadPoolExecutor)

### File Organization
- Never save working files to root folder
- `webhook_trigger.py` - Single-file application with all logic
- `http_triggers.csv` - CSV input with columns: webhook_url, method, payload, header
- `Procfile` / `railway.json` - Deployment configs for Railway
- `requirements.txt` / `runtime.txt` - Python dependencies and version

### Key Architecture
- `RateLimiter` class with adaptive rate adjustment based on error window
- `ResultsTracker` saves JSON results to `webhook_results.json`
- Supports multiple CSV files in deployment mode (`AUTO` flag)
- Config via environment variables: `BASE_RATE_LIMIT`, `MAX_WORKERS`, `SKIP_ROWS`, etc.
- Two modes: deployment (env var driven) and local (argparse CLI)

### Build & Test
```bash
pip install -r requirements.txt                    # Install dependencies
python webhook_trigger.py http_triggers.csv        # Local: run with CSV file
python webhook_trigger.py http_triggers.csv --keep-alive  # Keep process alive after completion
```

### n8n Workflow Automation

This project can trigger and receive n8n workflows at `https://n8n.chinmayramraika.in`.

- **Webhook URL:** Set in `N8N_WEBHOOK_URL` env var
- **API Key:** Set in `N8N_API_KEY` env var (unique per project)
- **Auth Header:** `X-API-Key: <N8N_API_KEY>`
- **Workflow repo:** github.com/Cramraika/n8n-workflows (private)

### Security Rules
- NEVER hardcode API keys, secrets, or credentials in any file
- NEVER pass credentials as inline env vars in Bash commands
- NEVER commit .env, .claude/settings.local.json, or .mcp.json to git
