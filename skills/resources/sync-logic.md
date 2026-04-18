# Sync Logic

## 1. Load State
- Read `agents/heypocket-sync/state.json`
- Capture `last_sync_timestamp`

## 2. First-Time Sync
If state file doesn't exist:
- Set `last_sync_timestamp` to `1970-01-01T00:00:00Z`

## 3. Derive Date Window
1. Convert `last_sync_timestamp` to UTC date `YYYY-MM-DD`
2. Subtract 5 minutes before date conversion to handle late writes
3. Set `end_date` to current UTC date

## 4. Fetch Recordings
```
GET /public/recordings?start_date={start_date}&end_date={end_date}&page={n}&limit=100
```

Paginate until exhausted (increment `page` until no more results).

## 5. Filter by Updated At
From results, keep only records where `updated_at > last_sync_timestamp`

Note: API doesn't support `updated_after` filter—use server-side date filter + client-side `updated_at` comparison.

## 6. Update State
After processing:
- Set `last_sync_timestamp` to maximum `updated_at` among processed records
- If no records processed, keep existing timestamp unchanged