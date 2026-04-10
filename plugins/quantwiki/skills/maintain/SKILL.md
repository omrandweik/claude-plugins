---
name: maintain
description: Use when the user says "status", "digest", "weekly digest", "rollback", "undo last ingest", "git log", or "what changed". Handles git operations, index management, weekly reports, and wiki infrastructure.
---

# Maintain

## When to Use

For git operations, index management, weekly reports, and wiki infrastructure. Also handles rollback requests and index scaling as the wiki grows.

## Protocol

### Git Workflow

**Post-Operation Commits:**

After every ingest, capture, lint fix, or auto-expansion, stage and commit:

```bash
git add -A
git commit -m "[operation] [brief description]"
```

**Commit message format:**
- `[ingest] Processed claude-chat-sessions.md — 34 pages created/updated`
- `[capture] Filed Kalman filter discussion into wiki`
- `[lint] Fixed 12 broken backlinks, flagged 3 stale pages`
- `[expand] Auto-researched ornstein-uhlenbeck-process.md`
- `[review] Resolved 2 disputed claims in market-impact pages`
- `[restructure] Migrated to hierarchical index (page count: 130)`

### Rollback Protocol

If the user says "undo last ingest" or "rollback":

1. Show the last 5 commits with `git log --oneline -5`
2. Ask the user to confirm which commit to revert to
3. Execute `git revert` (not `git reset` — preserve history)
4. Never force-push or reset without explicit user confirmation

### Weekly Digest

On request (or suggest weekly), generate a git-based changelog:

```bash
git log --oneline --since="7 days ago"
```

Format as a wiki growth report:

```
### Weekly Wiki Digest — [date range]
- Pages created: 12
- Pages updated: 28
- Sources ingested: 5
- Disputes flagged: 2 (1 resolved)
- Current total: 87 pages, 142,000 words
- Top growth areas: macro strategies (+8 pages), econometrics (+4 pages)
```

### Index Scaling

The single `index.md` works up to ~100-150 wiki pages. Beyond that, transition to hierarchical indexes:

```
wiki/
├── index.md                    # Top-level: links to sub-indexes, totals
├── index-concepts.md           # All concept pages
├── index-strategies.md         # All strategy pages
├── index-interview.md          # All interview prep pages
├── index-resume.md             # All resume pages
├── index-entities.md           # All entity pages
├── index-comparisons.md        # All comparison pages
```

**Trigger:** When page count exceeds 120, automatically restructure. Notify the user.

**Commit:** `[restructure] Migrated to hierarchical index (page count: N)`

### Phase 2: Search (>300 pages)

When page count exceeds 300, implement a search tool:

- `qmd` (BM25 + vector search + LLM re-ranking, local, has MCP server) — preferred
- Simple grep-based search script over `wiki/` as a stopgap
- Update query workflow: search first, then fall back to index browsing

## Examples

**Status check:** "status" → Show current page count, last commit, recent activity summary.

**Weekly digest:** "weekly digest" → Run `git log --since="7 days ago"`, format growth report.

**Rollback:** "undo last ingest" → Show last 5 commits, confirm target, execute `git revert`.

**Index scaling:** Page count hits 125 → Automatically split `index.md` into sub-indexes, update top-level, commit, notify user.

## Edge Cases

- **No git repo:** If git isn't initialized, run `git init`, create `.gitignore`, and make initial commit before proceeding.
- **Merge conflicts during revert:** Present the conflict to the user and ask for guidance rather than resolving automatically.
- **Large commits:** If a single operation touches 20+ files, still make one commit (not per-file). The commit message should summarize the scope.
