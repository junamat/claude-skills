# claude-skills

Personal skills for Claude Code.

| Skill | What it does |
|-------|--------------|
| [`pr-review`](pr-review/SKILL.md) | Reviews a GitHub PR with `caveman-review` style findings, inline code-anchored comments, and an approve / request-changes verdict. |

## Install

Symlink each skill into `~/.claude/skills/`:

```bash
git clone git@github.com:junamat/claude-skills.git ~/claude-skills
for d in ~/claude-skills/*/; do ln -sfn "$d" ~/.claude/skills/"$(basename "$d")"; done
```

`pr-review` depends on the `caveman-review` skill being installed.
