# Catalog checklist

Sources: `CLAUDE.md`, `docs/architecture/{backend-layers,testing,frontend-components}.md` in the
catalog checkout. Read them; this list only says what to look for.

## Tests (highest-value check)
- New/changed behaviour without an integration test that hits the real endpoint on a temp SQLite DB
  (mocks only at external boundaries: LLM, S3). Pure frontend change: vitest on logic extracted to
  `lib/utils/`.
- Bug fix with no test that would have failed before it.
- New pure function without a unit test.
- Mocks patched where the symbol is defined instead of where it is used; global processing queue not
  drained; `open_webui` imported above the `DATABASE_URL` override in `backend/tests/conftest.py`.
- A green frontend run is not evidence: `--passWithNoTests`. Path filters mean a frontend-only PR
  does not run pytest and vice versa.
- No tests and no explanation in the PR's Tests section: flag it.

## Layering (new code and heavily modified code only)
- Router body doing multi-step business logic or a direct `select()`; ruff complexity > 10 hidden by `# noqa`.
- Service importing `Request`/`HTTPException` or writing SQL; a new service -> router import.
- Repository (`models/*.py`) calling LLM/storage/HTTP.
- New logic added to `main.py`, `config.py`, `utils/middleware.py` instead of a router/service.
- Do not ask for a refactor of code the PR did not substantially touch.

## Auth and data access
- New/changed endpoint missing the ownership check + `AccessGrants.has_access(...)` + admin escape
  hatch shape used by neighbours, or missing `has_access_to_file` on file reads. Any new gate that
  differs from its router neighbours.
- Repository calls not threading `db=db` (one request = one transaction).
- Errors not raised via `HTTPException` with `ERROR_MESSAGES`.

## Knowledge-domain invariants
- `ai_overwiew` renamed without a migration.
- `services/file_analysis.py`: an LLM/parse failure that no longer returns "eligible" (must fail open).
- Embedding allowlist changes that stop non-allowlisted files from contributing AI context.
- Queue semantics (`In queue` -> `Processing` -> done): new paths that bypass the sequential queue.
- Alembic migrations: missing downgrade, non-idempotent, breaks on PostgreSQL vs SQLite.
- `statistics_outdated` flag replaced by a refetch.

## Upstream-merge cost
- Chat, channels, audio, images, ollama, openai, admin settings are upstream code: flag changes
  broader than the fix required, reformatting, or renames there.

## Frontend
- Svelte 5 patterns, tabs/single quotes are lint's job; check instead: component grown instead of
  split, logic left in the component instead of `lib/utils/`, API wrapper not matching the
  "capture error, then throw error" shape.
- New UI string without a key in `src/lib/i18n` (en-US and uk-UA only) or without `i18n:parse`.

## Hygiene
- Secrets, keys or tokens in the diff; committed large binaries; edits to `.claude/`, `.github/`
  or `dev.sh` flags (`--reload-exclude 'data/*'`) that weaken safeguards.
- Title not a conventional commit scoped by domain.
