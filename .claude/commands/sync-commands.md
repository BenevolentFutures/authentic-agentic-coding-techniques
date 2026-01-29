# Sync Commands from Private Sources

Update this public repository with the latest versions of commands from the private Authentic sources.

**Note:** This command only works when run by maintainers who have access to the private source files.

## Source Locations

| Public File | Private Source |
|-------------|----------------|
| `commands/code-review.md` | `~/.claude/commands/code_review.md` |
| `commands/code-review-critical.md` | `~/.claude/commands/code_review_critical.md` |
| `commands/team-three-review.md` | `~/.claude/commands/team_three_review.md` |
| `commands/ship.md` | `~/Projects/Authentic/Engineering/MainRepos/.claude/commands/ship.md` |

## Instructions

1. **Check for changes** — Show diffs between current public files and private sources:

```bash
echo "=== code-review.md ==="
diff commands/code-review.md ~/.claude/commands/code_review.md || true

echo "=== code-review-critical.md ==="
diff commands/code-review-critical.md ~/.claude/commands/code_review_critical.md || true

echo "=== team-three-review.md ==="
diff commands/team-three-review.md ~/.claude/commands/team_three_review.md || true

echo "=== ship.md ==="
diff commands/ship.md ~/Projects/Authentic/Engineering/MainRepos/.claude/commands/ship.md || true
```

2. **Review changes** — Show the user a summary of what changed in each file.

3. **Copy updated files** — For each file with changes, ask if the user wants to update it:

```bash
# Only run these for files the user approves:
cp ~/.claude/commands/code_review.md commands/code-review.md
cp ~/.claude/commands/code_review_critical.md commands/code-review-critical.md
cp ~/.claude/commands/team_three_review.md commands/team-three-review.md
cp ~/Projects/Authentic/Engineering/MainRepos/.claude/commands/ship.md commands/ship.md
```

4. **Sanitize** — After copying, review each updated file and remove or generalize:
   - Authentic-specific references (AUT-123 → generic ticket IDs)
   - Internal tool references that won't work for external users
   - Private information

5. **Commit** — Stage and commit the changes:

```bash
git add commands/
git commit -m "sync: update commands from private sources

$(date +%Y-%m-%d)

Co-Authored-By: Claude <noreply@anthropic.com>"
```

6. **Output summary** — Tell the user what was updated.

## CLAUDE.md Examples

The `examples/` directory contains CLAUDE.md examples that are **manually curated**. Do NOT automatically sync these — they require human review to ensure no private information is shared.

Remind the user to manually review and update:
- `examples/global-claude-md.md`
- `examples/workspace-claude-md.md`
