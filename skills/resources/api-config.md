# API Configuration

## Base URL
`https://public.heypocketai.com/api/v1`

## Authentication
- **Method**: Bearer Token
- **Source**: `HEYPOCKET_API_TOKEN` environment variable

## Endpoints

### List Recordings
```
GET /public/recordings
```

**Query Parameters**:
| Parameter | Type | Description |
|-----------|------|-------------|
| `start_date` | string | UTC date `YYYY-MM-DD` |
| `end_date` | string | UTC date `YYYY-MM-DD` |
| `page` | integer | Page number (default: 1) |
| `limit` | integer | Results per page (default: 100) |
| `tag_ids` | string | Filter by tags |

### Get Recording Details
```
GET /public/recordings/{id}
```

Returns: summary, mind map, action items, metadata.

## Error Handling
- **401**: Invalid token → check `HEYPOCKET_API_TOKEN`
- **429**: Rate limited → implement exponential backoff