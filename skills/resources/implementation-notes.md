# Implementation Notes

## Pagination
- Use `limit=100` and iterate `page` until no more results
- Check response for `pagination.total` to determine total count

## Error Handling
- **429 (Rate Limit)**: Back off and retry after appropriate delay
- **401 (Invalid Token)**: Verify `HEYPOCKET_API_TOKEN` is correct

## Action Item Register Format
- File path: `Recordings/HeyPocket Action Items.md`
- Per recording section:
  - Recording heading (linked): `## [[YYYY-MM-DD - {Title}]]`
  - Action list as Obsidian task lines:
    - Open: `- [ ] {label} (assignee: {name}, due: {date_or_none}, type: {action_type})`
    - Completed: `- [x] {label} (assignee: {name}, due: {date_or_none}, type: {action_type})`
- Omit sections when no actions available
- Append new records to end; do not modify existing entries

## Mind Map Format
- If API returns bulleted list: convert to hierarchical Mermaid mindmap
- Use recording title as `root` node
- Group related bullets under logical parent categories
- Wrap entire block in ```mermaid ... ```
- If API already provides Mermaid, preserve hierarchy and ensure wrapping