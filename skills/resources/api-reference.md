# API Reference

## Base URL
`https://public.heypocketai.com/api/v1`

## Authentication
Bearer Token sourced from `HEYPOCKET_API_TOKEN` environment variable.

## Endpoints

### List Recordings
`GET /public/recordings`

Query Parameters:
| Param | Type | Description |
|-------|------|-------------|
| `start_date` | string | Start date filter (YYYY-MM-DD UTC) |
| `end_date` | string | End date filter (YYYY-MM-DD UTC) |
| `page` | integer | Page number for pagination |
| `limit` | integer | Results per page (max 100) |
| `tag_ids` | string | Filter by tag IDs |

### Get Recording Details
`GET /public/recordings/{id}`

Returns full recording data including summary, mind map, and action items.