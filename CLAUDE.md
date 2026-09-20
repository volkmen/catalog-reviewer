# CLAUDE.md

This repo contains **instructions only** for reviewing `volkmen/catalog` PRs — no application
code. The products are `.claude/skills/pr-reviewer/` (read-only) and
`.claude/skills/process-pr-comments/` (edits, commits and pushes to the PR branch).

- Keep SKILL.md short and procedural; put long checklists in `references/`.
- Never duplicate catalog's `CLAUDE.md` / `docs/architecture` content here; point at it.
  Duplicated rules go stale silently.
- pr-reviewer only reads and posts. It must never edit catalog code, commit or push.
- process-pr-comments is the only skill that writes code; it must never force-push, skip hooks,
  or touch `.github/`.
- A change to the skill should say what review behaviour it changes and why.
- Only commit or push when asked.
