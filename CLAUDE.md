# Webhook Trigger

## Claude Preamble
<!-- VERSION: 2026-04-19-v9 -->
<!-- SYNC-SOURCE: ~/.claude/conventions/universal-claudemd.md -->

**Universal laws** (§4), **MCP routing** (§6), **Drift protocol** (§11), **Dynamic maintenance** (§14), **Capability resolution** (§15), **Subagent SKILL POLICY** (§16), **Session continuity** (§17), **Decision queue** (§17.a), **Attestation** (§18), **Cite format** (§19), **Three-way disagreement** (§20), **Pre-conditions** (§21), **Provenance markers** (§22), **Redaction rules** (§23), **Token budget** (§24), **Tool-failure fallback** (§25), **Prompt-injection rule** (§26), **Append-only discipline** (§27), **BLOCKED_BY markers** (§28), **Stop-loss ladder** (§29), **Business-invariant checks** (§30), **Plugin rent rubric** (§31), **Context ceilings** (§32), **Doc reference graph** (§33), **Anti-hallucination** (§34), **Past+Present+Future body** (§35), **Project trackers** (§36), **Doc ownership** (§37), **Archive-on-delete** (§38), **Sponsor + white-label** (§39), **Doc-vs-code drift** (§40).

**Sources**: `~/.claude/conventions/universal-claudemd.md` (laws, MCP routing, lifecycle, rent rubric, doc-graph, anti-hallucination) + `~/.claude/conventions/project-hygiene.md` (doc placement, cleanup, archive-on-delete, ownership matrix). Read relevant sections before significant work. Re-audit due **2026-07-19**. Sync: `~/.claude/scripts/sync-preambles.py`.

## Deprecation Notice

**STATUS: SUPERSEDED — maintenance-only.**

This repo (`SMPL562/webhook_trigger`) is the v1 prototype of a bulk webhook firing utility. It has been **superseded by `Cramraika/bulk_api_trigger`** (v2.0 platform rewrite with Docker, renovate, config-driven design, ~191KB application vs this 17KB single-file script).

- **New work → `bulk_api_trigger`**: feature additions, bug fixes, deployment changes.
- **Here → no new features**: accept security patches + migration nudges only. Every CLAUDE.md edit to this file should ask first: "does this belong in bulk_api_trigger instead?" If yes, stop.
- **Migration path for active users**: switch CSV + env config to bulk_api_trigger format (compatible schema; see bulk_api_trigger CLAUDE.md).
- **Archive-on-GitHub candidate**: user may archive this repo once remaining users are migrated; kept un-archived for now so ops teams with existing Railway/Coolify deploys aren't broken.
- **Superseded date**: 2026-04-12 (first bulk_api_trigger commit day).

## Status & Tier

| Field | Value |
|---|---|
| **Tier** | C (maintenance-only, superseded) |
| **Lifecycle** | Deprecated; no feature work; accept security-critical patches only |
| **Ownership** | SMPL562 org; user still holds write access via PAT |
| **Push policy** | Local commits only — SMPL562 PAT auth blocks push from this workstation; escalate to GitHub UI if a push is truly required |

## References

- `~/.claude/conventions/universal-claudemd.md` — universal laws, MCP routing, rent rubric, context ceilings
- `~/.claude/conventions/project-hygiene.md` — doc placement, cleanup triggers, local-only workspaces
- Successor: `~/Documents/Github/bulk_api_trigger/CLAUDE.md`
- Inventory: `~/.claude/conventions/repo-inventory.md`

## Stack

- Python 3.11 (pinned via `runtime.txt`)
- `requests==2.33.0`, `tqdm==4.66.3` (patched for CVE 2026-03 security audit)
- Standard lib: `concurrent.futures.ThreadPoolExecutor`, `threading.Lock/Semaphore`, `csv`, `json`, `glob`, `logging`
- Deployment: Railway (Procfile + railway.json) or Coolify; no containerization in this repo (successor adds Docker)

## Active Role-Lanes

Minimal surface — maintenance-only repo:

- **Engineer** (security patches, dependency bumps)
- **Manager** (migration planning, archival timing)

Deactivated (belong in successor instead):

- Designer, Analyst, SEO, Writer, Marketer — N/A (CLI tool, internal)
- Feature-Engineer — all new feature work routes to `bulk_api_trigger`

## Build / Test / Deploy

```bash
# Install
pip install -r requirements.txt

# Run locally (CLI mode)
python webhook_trigger.py http_triggers.csv
python webhook_trigger.py http_triggers.csv --keep-alive

# Deployment mode (Railway/Coolify) — DEPLOYMENT_MODE or RAILWAY_ENVIRONMENT env triggers auto-CSV-discovery
```

CI: `.github/workflows/ci.yml` — flake8 (E9,F63,F7,F82) + `pip-audit` (continue-on-error) + env file validation. Runs on push/PR to main/master. Status: GREEN.

Full environment reference: `docs/ENVIRONMENTS.md`.

## Key Directories

```
webhook_trigger/
├── webhook_trigger.py      # single-file application (~17KB)
├── requirements.txt        # requests + tqdm (pinned)
├── runtime.txt             # python-3.11.5
├── Procfile                # Railway worker entrypoint
├── railway.json            # Railway Nixpacks config
├── .env.example            # env var template (no secrets)
├── docs/ENVIRONMENTS.md    # deployment + env reference
└── .github/workflows/ci.yml  # lint + security audit
```

- `.claude/settings.json` — per-project Claude Code settings (scripts profile)
- `.claude/settings.local.json`, `.mcp.json`, `.env` — gitignored

## Product Features (maintained — no new work)

1. **Bulk webhook execution** — CSV (`webhook_url, method, payload, header`) → parallel HTTP fires with tqdm progress. Results → `webhook_results.json`.
2. **Adaptive rate limiting** — `RateLimiter` slows down on error spikes (configurable window + threshold), recovers when errors clear.
3. **Deployment mode** — env-var driven; auto-discovers all CSV files in working dir.
4. **Resume/skip** — `SKIP_ROWS=N` for interrupted-run recovery.

## Known Limitations

- No built-in deduplication — operator manages skip counts manually
- No persistent state — crash recovery via `webhook_results.json` only
- No notification on completion/failure
- Single-threaded rate limiter caps true concurrency
- Resolved in successor (`bulk_api_trigger`): config.yaml schema, Docker-based deploy, renovate-tracked deps, 191KB feature-expanded application

## Security & Secrets

- Universal laws apply (§4 of `universal-claudemd.md`): never hardcode keys, never commit `.env`/`.mcp.json`/`.claude/settings.local.json`.
- `.env.example` documents required vars only (rate limits + n8n webhook placeholder).
- Sensitive deploy secrets live in Railway/Coolify dashboard env vars — not in repo.
- Past security fix: 4 CVEs patched via `fdf81cc` (2026-03).

## Deployment Environments

| Env | Host | Trigger |
|---|---|---|
| **Local** | operator workstation | `python webhook_trigger.py <csv>` |
| **Railway** | legacy existing deploy | worker dyno (`Procfile`); `DEPLOYMENT_MODE` or `RAILWAY_ENVIRONMENT` env |
| **Coolify** | `http://31.97.43.125:11000` | Nixpacks build; `DEPLOYMENT_MODE=1` env |

New deployments should use `bulk_api_trigger` instead — Docker-native, config-driven, renovate-tracked.

## External Services (MCPs, integrations)

- **n8n** (`https://n8n.chinmayramraika.in`) — `N8N_WEBHOOK_URL` + `N8N_API_KEY` env vars; auth via `X-API-Key` header
- **GitHub** — SMPL562 org; PAT in `$SMPL562_PAT` (push currently blocked from this workstation)

Per-project MCP disables (reduce static preamble cost — no UI in this repo): `figma`, `serena`, `context7` candidates per `universal-claudemd.md` §6.

## Past (origin → superseded)

- **2025-06-25** — initial commit (`2bb4d8c`). Single-file Python utility for bulk webhook firing.
- **2025** — iterative feature additions: adaptive rate limiter, deployment mode, resume/skip, Railway config.
- **2026-03-07** — Claude Code config added (`682c5fe`).
- **2026-03-15** — CI pipeline upgraded to ASM quality standard (`1497b3c`); 8.4MB CSV data file untracked (`085b85f`).
- **2026-03-23** — MIT LICENSE added.
- **2026-04-06** — dependency patches (`fdf81cc` — 4 CVE fixes: requests→2.33.0, tqdm→4.66.3).
- **2026-04-10** — n8n workflow automation integration (`150566e`).
- **2026-04-12** — **SUPERSEDED** by `bulk_api_trigger` v2.0 (Cramraika). Platform rewrite: Docker, renovate, config.yaml, expanded feature surface.
- **2026-04-18 → 2026-04-19** — CLAUDE.md preamble sync passes (v4 → v8) as universal conventions evolved; this body not refreshed until now.

## Dependency Graph

**Upstream (this repo is ancestor of):**
- `Cramraika/bulk_api_trigger` — **downstream successor**; forked concept + rewritten. Not a git-fork relationship (independent repo); conceptual descent. All new feature work flows there.

**Upstream of this repo (none):**
- No parent project. This repo was the original v1.

**Current consumers (legacy):**
- Any existing Railway/Coolify deployment still pointing at this repo's build
- CI/CD workflows (SMPL562) that shell out to this repo's Python script
- Operations team CLI runs on operator workstations

**Migration target for all consumers:** `bulk_api_trigger` (switch CSV format if needed, rewire deploy source, update env vars per new `.env.example`).

## Roadmap

- **Immediate (maintenance)**: accept pip-audit-surfaced CVE patches via `requirements.txt` bumps. No feature PRs.
- **Short-term (consumer migration)**: migrate remaining Railway/Coolify deploys to `bulk_api_trigger`. Track via inventory notes in `~/.claude/conventions/repo-inventory.md`.
- **Medium-term (archive decision)**: once zero active consumers, archive on GitHub with note "superseded by Cramraika/bulk_api_trigger". User-gated decision — not auto.
- **Post-archive**: retain code history via GitHub archive (read-only). No delete.

**Not on the roadmap** (explicitly rejected):
- New features here (route to successor)
- Refactors to match successor's architecture (churn without value — just migrate consumers)
- Docs expansion beyond this CLAUDE.md + `docs/ENVIRONMENTS.md`

## Deviations from Universal Laws

Intentional deviations from `universal-claudemd.md` for this repo's deprecated state:

1. **§32 context ceilings** — per-project budget intentionally minimal; `scripts` plugin profile (not `python-backend`) since this is CLI utility without active dev.
2. **§14 dynamic maintenance — repo lifecycle** — inventory-sync flags this as active, but stack-line is frozen (no drift-response even if Python version drifts). Override: maintenance-mode repo.
3. **§31 plugin rent rubric** — routing-specificity bar raised: plugins that *could* help feature work are CUT because no feature work happens here. Tighter than sibling active Python repos.
4. **§17 session continuity** — no SESSION_LOG required; any work here is trivial single-session (security patch + commit).
5. **Push policy** — normal universal law allows pushing with user confirmation; here, push is blocked by PAT state AND by policy (commit local only; escalate if push is truly needed).
6. **Living README stance** (`project-hygiene.md`) — README is declared "living" in hygiene convention, but the current `README.md` is a 1-line placeholder and intentionally stays minimal; consumers read `docs/ENVIRONMENTS.md` or the successor repo's rich README.

All other universal laws apply unchanged.
