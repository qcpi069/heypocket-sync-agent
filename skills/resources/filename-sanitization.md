# Filename Sanitization

## Rules

| Replace | With |
|---------|------|
| `/`, `\`, `:`, `*`, `?`, `"`, `<`, `>`, `\|` | `-` |
| Multiple spaces | Single space |
| Leading/trailing whitespace | (removed) |
| Leading/trailing dots | (removed) |

## Truncation
- Max **100 characters** after sanitization
- Preserve `YYYY-MM-DD - ` prefix unchanged

## Examples

| Input | Output |
|-------|--------|
| `1-on-1 with: Bob / Smith` | `1-on-1 with - Bob - Smith` |
| `  Interview - Alice.  ` | `Interview - Alice` |
| `Very long title...` (101+ chars) | `Very long title that goes on and on and exceeds reasonable length li` |