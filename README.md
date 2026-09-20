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
.github/workflows/review-pr.yml   # reusable workflow (workflow_call) that catalog calls
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
      REVIEWER_REPO_TOKEN: ${{ secrets.REVIEWER_REPO_TOKEN }}   # read access to this repo
```

The reusable workflow checks out the PR's branch and this repo, drops the skill into the
workspace and runs `/pr-reviewer <pr>` unattended. Trigger on mention only: reviewing every
push cost ~$2.90 per run on a large PR (see the disabled `pr-reviewer.disabled` in catalog).

## Not covered here

Processing review comments (`process-pr-comments`) still lives in `catalog` because it edits
code, commits and pushes on the PR branch. Moving it is a separate step.
