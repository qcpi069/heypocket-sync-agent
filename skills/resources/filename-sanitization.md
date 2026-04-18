# Filename Sanitization

Before creating or looking up a recording file, sanitize the title:

| Replace | With |
|---------|------|
| `/`, `\`, `:`, `*`, `?`, `"`, `<`, `>`, `\|` | `-` |
| Multiple spaces | Single space |
| Leading/trailing whitespace | (removed) |
| Leading/trailing dots | (removed) |

- Truncate title to **100 characters max** (after sanitization)
- Preserve the `YYYY-MM-DD - ` prefix unchanged

## Examples

- `1-on-1 with: Bob / Smith` → `1-on-1 with - Bob - Smith`
- `  Interview - Alice.  ` → `Interview - Alice`
- `Very long title that goes on and on and exceeds reasonable length limits` → `Very long title that goes on and on and exceeds reasonable length li`