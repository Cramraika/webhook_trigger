# webhook_trigger — CLAUDE.md v2

**Date:** 2026-04-28 (S11B authoring)
**Supersedes:** v1 (final revision; archive banner only)
**Tier:** X (ARCHIVED on GitHub 2026-04-19) — **superseded by `bulk`**

---

## ARCHIVE BANNER

**STATUS: ARCHIVED on GitHub 2026-04-19. Superseded by `Cramraika/bulk` (renamed from `Cramraika/bulk_api_trigger` 2026-04-19).**

This repo is the v1 prototype of a bulk webhook firing utility (~17KB single-file). It has been **fully superseded** by `bulk` — v2.0 platform rewrite with Docker, Renovate, config-driven design, SQLite job tracking, watchdog auto-processing, REST status API, ~191 KB application.

**No new features. No new commits. No fork. Read-only.**

- **New work →** `~/Documents/Github/bulk/`
- **Migration path for active users:** switch CSV + env config to `bulk` format (compatible schema; see `bulk/CLAUDE.md`).
- **GitHub archive date:** 2026-04-19.
- **Superseded date:** 2026-04-12 (first `bulk_api_trigger`/`bulk` commit day).
- **Local dir retained for reference only.**

---

## Identity & Role (historical)

`webhook_trigger` was the v1 single-file Python prototype that demonstrated the CSV-driven webhook firing pattern. Its role transferred fully to `bulk` on 2026-04-12.

## Coverage Today

Not in scope — this is an archived shell. Per per-project-service-matrix, no row is maintained for archived repos.

## Stack (frozen)

- Python (single-file ~17KB)
- Lite reference implementation (no SQLite, no watchdog, no REST, no adaptive rate, no resume)

## Roadmap

None. Archived. New work goes to `bulk`.

## ADR Compliance

- **ADR-038 personal-scope:** ✓ — Cramraika org; archived 2026-04-19.

## Cross-references

- Successor: `~/Documents/Github/bulk/CLAUDE.md`
- `~/.claude/conventions/project-hygiene.md` § Rename Propagation Protocol
- `~/.claude/conventions/repo-inventory.md`
- `~/Documents/Github/CLAUDE.md` § Fleet Rename 2026-04-19 (Phase 1) — webhook_trigger ARCHIVED on GitHub (superseded by `bulk`)

## Migration from v1

**Major v1 → v2 changes:**
1. **Reduced to archive banner only** per dispatch directive ("`webhook_trigger` is ARCHIVED per CLAUDE.md root § Fleet Rename Phase 1; v2 = ARCHIVE banner only").
2. All technical sections trimmed; cross-reference to successor `bulk` is the canonical path forward.
