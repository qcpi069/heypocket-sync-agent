# Validation Checklist

Use this quick validation before or after changing sync logic.

1. **Load sync state**: Read `state.json` and capture `last_sync_timestamp`.
2. **Derive date window**: Compute `start_date` from `last_sync_timestamp` minus 5 minutes (UTC YYYY-MM-DD). Set `end_date` to current UTC date.
3. **Call unfiltered baseline**: `GET /public/recordings?page=1&limit=100`, record `pagination.total`.
4. **Call windowed list**: `GET /public/recordings?start_date={start_date}&end_date={end_date}&page=1&limit=100`, record `pagination.total`.
5. **Compute incremental candidates**: From windowed results, count items where `updated_at > last_sync_timestamp`.
6. **Expected results**:
   - No new recordings = 0 incremental candidates
   - Windowed total ≤ unfiltered total
   - Duplicate check = safeguard, not primary mechanism
7. **State update rule**: If candidates processed, set `last_sync_timestamp` to max processed `updated_at`. If 0, leave unchanged.
8. **Action item register checks**:
   - Updated (appended) only for recordings newly processed this run
   - Only recordings with ≥1 action items added
   - Recording headings in format: `## [[YYYY-MM-DD - {Title}]]`
   - Action items as `- [ ] ...` or `- [x] ...`
   - No duplicate sections