# Action Item Register

## File
`Recordings/HeyPocket Action Items.md`

## Format Per Recording

```markdown
## [[YYYY-MM-DD - {Title}]]

- [ ] {action label} (assignee: {name}, due: {date_or_none}, type: {action_type})
- [x] {action label} (assignee: {name}, due: {date_or_none}, type: {action_type})
```

## Rules
- **Only add recordings with action items** — omit groups entirely when no actions exist
- **Append** new entries to end of file — do not rewrite or modify existing entries
- **Order**: Process recordings chronologically by `recording_at` to maintain date-ascending structure
- **Heading**: Must be an Obsidian link (`[[...]]`)
- **Task syntax**: Use checkbox syntax `- [ ]` (open) or `- [x]` (completed)