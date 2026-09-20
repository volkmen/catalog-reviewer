# catalog-reviewer

Instructions (a Claude Code skill) for reviewing pull requests of
[`volkmen/catalog`](https://github.com/volkmen/catalog). The reviewer lives here, apart from
the code it reviews, so it can be changed, versioned and run without touching `catalog`.

## Layout

```
.claude/skills/pr-reviewer/
  SKILL.md                        # the workflow: resolve PR -> review -> verdict -> post
  references/
    catalog-checklist.md          # what to check that only a reviewer who knows catalog would
    posting.md                    # how to submit ONE review via gh (payload, diff-line check)
.claude/skills/process-pr-comments/SKILL.md   # fixes + replies + resolves review threads
.github/workflows/review-pr.yml   # reusable workflow (workflow_call) for the reviewer
.github/workflows/process-pr.yml  # reusable workflow for process-pr-comments
```

The skill deliberately does **not** copy catalog's rules. It tells the reviewer to read
`CLAUDE.md` and `docs/architecture/*` from the PR's checkout, so the rules cannot drift.
Only review-specific judgement (severity, verdict, what not to say) lives here.

## Using it

**Locally**, from a `catalog` checkout:

```bash
mkdir -p .claude/skills && cp -r ../catalog-reviwer/.claude/skills/pr-reviewer .claude/skills/
claude            # then:  /pr-reviewer 123
```

(or symlink it; don't commit it into catalog.)

**In CI**, add to `catalog/.github/workflows/` a small caller:

```yaml
name: PR Reviewer
on:
  issue_comment: { types: [created] }
jobs:
  review:
    if: >-
      github.event.issue.pull_request
      && contains(github.event.comment.body, '@volkmenYaryiClaude pr-reviewer')
      && contains(fromJSON('["OWNER","MEMBER","COLLABORATOR"]'), github.event.comment.author_association)
    uses: volkmen/catalog-reviewer/.github/workflows/review-pr.yml@main
    with:
      pr: ${{ github.event.issue.number }}
    secrets:
      CLAUDE_CODE_OAUTH_TOKEN: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
```

The reusable workflow checks out the PR's branch and this repo, drops the skill into the
workspace and runs `/pr-reviewer <pr>` unattended. Trigger on mention only: reviewing every
push cost ~$2.90 per run on a large PR (see the disabled `pr-reviewer.disabled` in catalog).

## process-pr-comments

`.claude/skills/process-pr-comments/` works through the unresolved review threads of a PR: fixes
code-change requests, replies, commits and pushes to the PR branch, and resolves threads whose
fix landed. It is self-contained (no dependency on catalog's `create-commit`/`push` skills) and
runs in CI via `.github/workflows/process-pr.yml`, called from catalog on a
`@volkmenYaryiClaude process` mention. Unlike the reviewer it needs `contents: write`.

## Usage report

After each review, `review-pr.yml` posts a PR comment with the run's token counts (input, output,
cache write/read), turns and approximate cost, read from the action's execution file.
**Not included:** the session/weekly subscription percentages shown by `/usage`. They are not in
the execution file; getting them needs a separate call to an undocumented Anthropic endpoint with
the OAuth token, which is not implemented.
