---
name: sync-recordings
description: Sync voice recordings from HeyPocket AI into Obsidian meeting minutes. Use when the user mentions syncing recordings, importing from HeyPocket, or updating meeting notes from voice recordings.
---

> **Important:** This skill expects to be run inside your Obsidian vault folder. The generated meeting notes will be saved directly to the `Recordings/` folder in your vault.
>
> If you run this outside of your Obsidian vault, you will need to manually move the generated notes into your vault.

# Skill: Sync Recordings

This skill synchronizes new recordings from HeyPocket AI into the Obsidian vault.

## Resources

- [API Reference](skills/resources/api-reference.md) — Base URL, auth, endpoints
- [Filename Sanitization](skills/resources/filename-sanitization.md) — Title sanitization rules
- [Implementation Notes](skills/resources/implementation-notes.md) — Pagination, error handling, action items, mind maps
- [Validation Checklist](skills/resources/validation-checklist.md) — Dry-run validation steps

## Sync Workflow

1. **Identify New Recordings**: Read `state.json` for `last_sync_timestamp`. Convert to UTC date (subtract 5 min overlap), call list endpoint with `start_date/end_date`, filter client-side for `updated_at > last_sync_timestamp`.
2. **Fetch Details**: For each new recording, check existing file in `Recordings/` (by `heypocket_id` + `updated_at`), skip if up-to-date, otherwise fetch full details via `GET /public/recordings/{id}`.
3. **Generate Note**: Create `Recordings/YYYY-MM-DD - {Title}.md` with YAML frontmatter (`date`, `heypocket_id`, `heypocket_updated_at`, `tags`). Sanitize title first (see Filename Sanitization). Strip duplicate headings from Executive Brief. Use strict section order: Title (H1), Summary (H2), Mindmap (H2), Executive Brief (H2).
4. **Update Action Item Register**: For recordings with action items, append to `Recordings/HeyPocket Action Items.md` if not already present. Process in chronological order.
5. **Update Sync State**: After successful sync, update `state.json` with max `updated_at` processed. Keep unchanged if no new records.