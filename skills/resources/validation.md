# Dry-Run Validation Checklist

## 1. Load Sync State
- Read `agents/heypocket-sync/state.json`
- Capture `last_sync_timestamp`

## 2. Derive Date Window
- Compute `start_date` from `last_sync_timestamp` minus 5 minutes, formatted as UTC `YYYY-MM-DD`
- Set `end_date` to current UTC date `YYYY-MM-DD`

## 3. Call Unfiltered Baseline
- Request `GET /public/recordings?page=1&limit=100`
- Record `pagination.total`

## 4. Call Windowed List
- Request `GET /public/recordings?start_date={start_date}&end_date={end_date}&page=1&limit=100`
- Record `pagination.total`

## 5. Compute Incremental Candidates
- From windowed results, count items where `updated_at > last_sync_timestamp`

## 6. Expected Results
- **No new recordings**: incremental candidate count should be `0`
- **Windowed total**: should be ≤ unfiltered total
- **Duplicate check**: safeguard only, not primary incremental mechanism

## 7. State Update Rule
- If candidates processed → set `last_sync_timestamp` to max processed `updated_at`
- If candidates = `0` → leave `last_sync_timestamp` unchanged

## 8. Action Item Register Checks
- [ ] Updated (appended) only for recordings newly processed in current run
- [ ] Only recordings with ≥1 action item were added
- [ ] Each heading is an Obsidian link: `## [[YYYY-MM-DD - {Title}]]`
- [ ] Each action is a task line: `- [ ] ...` or `- [x] ...`
- [ ] No duplicate sections for same recording