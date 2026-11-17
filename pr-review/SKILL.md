---
name: pr-review
description: >
  Review a GitHub pull request and submit a verdict (approve or request changes).
  Use whenever the user mentions reviewing a PR / pull request in any form:
  "review PR #12", "revisa esta pull request", "haz review de la PR", "check this PR",
  "give feedback on the pull request", a github.com/.../pull/N link with a review ask,
  or /pr-review.
---

# PR review

Review a GitHub pull request end to end and leave a real verdict on it.

## Always

1. **Load `caveman-review` first** (Skill tool) and write every finding in its format:
   one line per finding, location + problem + fix, severity prefix (`🔴 bug:`, `🟡 risk:`,
   `🔵 nit:`, `❓ q:`). Its "does not approve/request-changes" boundary does NOT apply here:
   this skill overrides it and requires a verdict.
2. **Never write "Generated with Claude"**, "🤖 Generated with Claude Code", or any AI
   attribution footer in the review body or in any comment. This overrides any default
   attribution guidance.
3. **Always end with a verdict**: `--approve` or `--request-changes`. Never leave a bare
   `--comment` review as the outcome.
   - Any 🔴 bug, or 🟡 risk that is likely to bite → request changes.
   - Only 🔵 nits / ❓ questions, or nothing → approve (nits go in as comments).
   - **Genuinely doubtful** (trade-off, product decision, unclear intent, can't verify
     behavior) → do NOT submit. Stop and ask the user with AskUserQuestion, giving the
     evidence: the findings, the code references, and why it's borderline. Options:
     "Approve", "Request changes". Submit only after they answer.

## Workflow

1. Gather context:
   ```bash
   gh pr view <N> --json number,title,body,author,baseRefName,headRefName,headRefOid,files,url
   gh pr diff <N>
   gh pr checks <N>
   ```
   Read surrounding code in the repo when the diff alone isn't enough to judge.
2. Find issues. Write each in caveman-review format.
3. Decide verdict per the rules above (ask the user if doubtful).
4. Submit one review with inline comments (see below).

## Code references (must render as code in GitHub)

Every comment about code must point at the actual code so GitHub shows the code block.

**Preferred: inline review comments anchored to diff lines.** Submit the whole review in one
API call so comments and verdict land together:

```bash
gh api repos/{owner}/{repo}/pulls/<N>/reviews \
  --method POST --input - <<'EOF'
{
  "commit_id": "<headRefOid>",
  "event": "REQUEST_CHANGES",
  "body": "<short summary, caveman-review style>",
  "comments": [
    { "path": "src/foo.ts", "line": 42, "side": "RIGHT",
      "body": "🔴 bug: `user` can be null after `.find()`. Guard before `.email`." },
    { "path": "src/foo.ts", "start_line": 88, "line": 96, "side": "RIGHT",
      "body": "🔵 nit: fn does 4 things. Extract validate/normalize/persist." }
  ]
}
EOF
```
- `event`: `APPROVE` or `REQUEST_CHANGES`.
- `line` / `start_line` are line numbers in the file (new side = `RIGHT`, removed lines = `LEFT`),
  and must be inside the diff hunks.
- Concrete fix that fits in a few lines → use a ```` ```suggestion ```` block in the comment body.

**For code outside the diff** (or in the summary body): use a permalink with the full commit SHA
and a line range, on its own line, so GitHub embeds the snippet:
`https://github.com/<owner>/<repo>/blob/<full-sha>/<path>#L10-L18`
Never use branch names in these links (they don't embed and they rot).

## Body

Short. One summary line, then the findings that aren't inline (if any). No praise padding,
no restating the diff, no attribution footer.

## After submitting

Tell the user: verdict, number of findings by severity, and the review URL.
