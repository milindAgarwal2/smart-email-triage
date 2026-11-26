# CLAUDE.md: smart-email-triage

**Read `PLAN.md` first.** It is the full spec (use cases, schemas, milestones, tests, decision log). This file holds the rules that must hold in every session.

## What this is
Configurable LLM triage for **customer emails** of small/medium businesses: safety rules → LLM classifier → LLM judge (only on risky/uncertain cases) → routing decision. Labels come from `taxonomies/<tenant>.yaml`. HVAC is the first example. n8n is the integration layer; all AI logic lives in Python.

## Features
Full list with IDs, milestones and status: **PLAN.md §2A Feature catalog** (F-01 to F-34 in scope, F-40+ deferred). Reference feature IDs in commits, and update their status when done.

## Current status
- Milestone: **v0.1 Foundations**, not started (update this line at the end of each session)
- Next: see the PLAN.md §6 checkboxes

## Hard rules (do not break)
1. **Safety rules live in code** (`triage/safety.py`), run **before** any LLM call, and can never be disabled by config. False negatives are unacceptable; false positives are fine.
2. **Labels are never hard-coded.** Prompts, enums and validation are generated from the taxonomy YAML.
3. **Email text is untrusted.** Always fence it, strip spoofed markers, validate LLM output against the schema. Free text never drives routing.
4. Any provider error, invalid JSON after one retry, or unknown label → `needs_human`. Never crash, never drop an email.
5. **Tenant isolation:** never mix tenants' data in retrieval or storage. Store `taxonomy_version` with every label.
6. **Never evaluate on `split: seed` data** (that's retrieval/few-shot only).
7. Only synthetic data in the repo (`example.com` addresses). Secrets only in `.env`.
8. n8n does plumbing only. Don't move classification logic into n8n nodes.
9. Tests must pass **without an API key** (use `FakeProvider` / `rules_only`).

## Commands (once v0.1 exists)
```bash
make test        # unit + pipeline tests, no API key needed
make eval        # real-model eval → eval/results/  (needs ANTHROPIC_API_KEY)
make gen-data    # synthetic emails (review before committing)
make run         # FastAPI on :8000 (v0.5)
```

## Conventions
- Python 3.11+, Pydantic v2, pytest, ruff. Type hints on public functions.
- One milestone per branch: `feat/v0.x-<name>`.
- Model IDs come from `.env`/config, never hard-coded.
- When a design decision changes, update PLAN.md §11 (decision log).
- At the end of each session: tick the PLAN.md §6 checkboxes and update "Current status" above.

## Related repo
`milindAgarwal2/doc-intelligence-rag`: reuse patterns for fastembed + Chroma retrieval (`embeddingEngine.py`), prompt fencing (`chatBOT_engine._fence_context`), the eval harness style (`tests/eval/`) and fail-safe DB logging (`db.py`).
