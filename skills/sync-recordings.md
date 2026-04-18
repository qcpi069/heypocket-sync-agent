---
name: sync-recordings
description: Sync voice recordings from HeyPocket AI into Obsidian meeting minutes. Use when the user mentions syncing recordings, importing from HeyPocket, or updating meeting notes from voice recordings.
---

# Skill: Sync Recordings

Quick reference for syncing HeyPocket recordings to Obsidian notes.

## Workflow

1. **Check last sync** → Read `agents/heypocket-sync/state.json` for `last_sync_timestamp`
2. **Fetch new recordings** → Call API with date window, filter by `updated_at`
3. **Create notes** → Write to `Recordings/YYYY-MM-DD - {Title}.md`
4. **Update action items** → Append to `Recordings/HeyPocket Action Items.md`
5. **Save state** → Update `last_sync_timestamp` to max processed `updated_at`

## Section Order (Required)
1. `# {Title}` (H1)
2. `## Mindmap` — Mermaid diagram (first for quick visual)
3. `## Summary` — concise one-paragraph overview
4. `## Executive Brief` — detailed content, subsections, key decisions

## Resources

Load these as needed:

| Resource | Purpose |
|----------|---------|
| `skills/resources/api-config.md` | Base URL, auth, endpoints |
| `skills/resources/sync-logic.md` | Date window calculation, filtering, pagination |
| `skills/resources/note-formatting.md` | YAML frontmatter, section order, content preprocessing |
| `skills/resources/filename-sanitization.md` | Title cleaning rules, truncation |
| `skills/resources/action-register.md` | Action item register format, task syntax |
| `skills/resources/mindmap-format.md` | Converting bullet lists to Mermaid |
| `skills/resources/validation.md` | Dry-run validation checklist |

## Notes

- **First sync**: If state.json missing, use `1970-01-01T00:00:00Z` to fetch all recordings
- **Duplicate handling**: Check both `heypocket_id` and `updated_at` before creating/overwriting
- **Error handling**: Handle 429 (rate limit) and 401 (invalid token)