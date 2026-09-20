# Posting the review

Requires `gh` authenticated with `pull-requests: write`. Repo: `volkmen/catalog`.

## Open threads (for de-duplication)

```bash
gh api graphql -f query='
  query($owner:String!,$repo:String!,$pr:Int!){repository(owner:$owner,name:$repo){pullRequest(number:$pr){
    reviewThreads(first:100){nodes{isResolved path line comments(first:5){nodes{author{login} body}}}}}}}' \
  -f owner=volkmen -f repo=catalog -F pr=<n>
```

Keep `isResolved: false` entries.

## Diff-line check

`gh pr diff <n>` shows the hunks. An inline comment must point at a line present in the diff on the
`RIGHT` side (added or context line in the new file). Otherwise GitHub rejects the whole review.
Move it to the nearest changed line in that file or fold it into the review body.

## Submit (one call)

Build the payload as JSON; `commit_id` pins the review to the reviewed head.

```bash
cat > "$TMPDIR/review.json" <<'JSON'
{
  "commit_id": "<headRefOid>",
  "event": "REQUEST_CHANGES",
  "body": "**[Reviewer]** <summary>",
  "comments": [
    {"path": "backend/open_webui/routers/knowledge.py", "line": 123, "side": "RIGHT",
     "body": "**[Reviewer]** <finding + fix>"}
  ]
}
JSON
gh api repos/volkmen/catalog/pulls/<n>/reviews -X POST --input "$TMPDIR/review.json"
```

`event` is `APPROVE`, `COMMENT` or `REQUEST_CHANGES`. An approve/request-changes on one's own PR is
rejected by GitHub; fall back to `COMMENT` and say so in the body.
