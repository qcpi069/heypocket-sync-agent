# Skill: Sync Recordings

This skill enables the `heypocket-sync` subagent to synchronize new recordings from HeyPocket AI into the Obsidian vault.

## API Integration Details
- **Base URL**: `https://public.heypocketai.com/api/v1`
- **Authentication**: Bearer Token (sourced from the `HEYPOCKET_API_TOKEN` environment variable).
- **Endpoints**:
    - `GET /public/recordings`: List recordings with supported query parameters: `start_date`, `end_date`, `page`, `limit`, `tag_ids`.
    - `GET /public/recordings/{id}`: Fetch detailed recording data (summary, mind map, etc.).

## Sync Workflow
1. **Identify New Recordings**: 
    - Read the `last_sync_timestamp` from `agents/heypocket-sync/state.json`.
    - **First-Time Sync**: If the file doesn't exist, set `last_sync_timestamp` to `1970-01-01T00:00:00Z` (or null) to fetch all historical recordings.
    - Convert `last_sync_timestamp` to UTC date (`YYYY-MM-DD`) and apply a small overlap window (for example, subtract 5 minutes before date conversion) to prevent missing late writes.
    - Call `GET /public/recordings?start_date={derived_start_date}&end_date={today_utc}&page={n}&limit=100` and paginate until exhausted.
    - For each returned item, keep only records where `updated_at > last_sync_timestamp`.
2. **Fetch Assets for Each ID**:
    - For each recording in the response:
        - **Duplicate Check**: Verify if a note with the current `heypocket_id` already exists in `Recordings/`. If it does, skip to the next recording.
        - Call `GET /public/recordings/{id}` to fetch full details.

3. **Generate Obsidian Note**:
    - Create a file in `Recordings/YYYY-MM-DD - {Title}.md`.
    - Format with YAML properties: `date`, `heypocket_id`, `tags: [heypocket-sync, meeting-minutes]`.
    - **Strict Section Order**:
        1. `# {Title}`
        2. `## Summary` (A concise overview of the recording)
        3. `## Mindmap` (The Mermaid.js diagram)
        4. `## Executive Brief` (Detailed insights, key decisions, and nuance)
4. **Update Sync State**:
    - After a successful sync, update `agents/heypocket-sync/state.json` with the maximum `updated_at` timestamp among records actually processed.
    - If no newer records were processed, keep the existing `last_sync_timestamp` unchanged.

## Implementation Notes
- **Incremental Sync**: The `list recordings` endpoint does not expose an `updated_after` filter. Use `start_date/end_date` server-side filtering plus client-side `updated_at > last_sync_timestamp` checks for true incremental behavior.
- **Pagination**: Use `limit=100` and iterate `page` until no more results.
- **Error Handling**: Handle rate limits (429) and invalid tokens (401).
- **Mind Map Format**: 
    - If the mind map returned by the API is a bulleted list, convert it into a hierarchical Mermaid mindmap block.
    - Use the recording's title as the `root` node.
    - Group related bullet points under logical parent categories to create a clear visual hierarchy.
    - Ensure the entire block is wrapped in ```mermaid ... ``` for native rendering in Obsidian.
    - If the API already provides Mermaid format, ensure it is correctly wrapped and the hierarchy is preserved.

## Dry-Run Validation Checklist
Use this quick validation before or after changing sync logic.

1. **Load sync state**:
    - Read `agents/heypocket-sync/state.json` and capture `last_sync_timestamp`.
2. **Derive date window**:
    - Compute `start_date` from `last_sync_timestamp` minus 5 minutes, formatted as UTC `YYYY-MM-DD`.
    - Set `end_date` to current UTC date (`YYYY-MM-DD`).
3. **Call unfiltered baseline**:
    - Request `GET /public/recordings?page=1&limit=100` and record `pagination.total`.
4. **Call windowed list**:
    - Request `GET /public/recordings?start_date={start_date}&end_date={end_date}&page=1&limit=100` and record `pagination.total`.
5. **Compute incremental candidates**:
    - From windowed results, count items where `updated_at > last_sync_timestamp`.
6. **Expected results**:
    - If no new recordings exist, incremental candidate count should be `0`.
    - Windowed total should be less than or equal to unfiltered total.
    - Duplicate check should only act as a safeguard, not the primary incremental mechanism.
7. **State update rule check**:
    - If candidates were processed, set `last_sync_timestamp` to the maximum processed `updated_at`.
    - If candidates were `0`, leave `last_sync_timestamp` unchanged.
