---
name: sync-recordings
description: Sync voice recordings from HeyPocket AI into Obsidian meeting minutes. Use when the user mentions syncing recordings, importing from HeyPocket, or updating meeting notes from voice recordings.
---

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
        - **Duplicate/Stale Check**: Check if a note with the current `heypocket_id` already exists in `Recordings/`:
            - If file **does not exist**: proceed to fetch details and create note.
            - If file **exists with same `updated_at`**: skip (already up to date).
            - If file **exists but `updated_at` is older** (or missing in frontmatter): overwrite with fresh data.
            - If file **exists with no `heypocket_updated_at` in frontmatter**: compare `file.mtime` to API `updated_at`; overwrite if API is newer.
        - Call `GET /public/recordings/{id}` to fetch full details.

3. **Generate Obsidian Note**:
    - Create a file in `Recordings/YYYY-MM-DD - {Title}.md`.
    - **Sanitize the title** before creating the filename (see Filename Sanitization below).
    - Format with YAML properties: `date`, `heypocket_id`, `heypocket_updated_at`, `tags: [heypocket-sync, meeting-minutes]`.
    - **Preprocess content**: Before inserting Executive Brief content, strip any leading `### Executive Brief` heading (or similar variants like `### ✅ Action Items`) from the API response to avoid duplicate headings.
    - **Strict Section Order**:
        1. `# {Title}`
        2. `## Summary` (A concise overview of the recording)
        3. `## Mindmap` (The Mermaid.js diagram)
        4. `## Executive Brief` (Detailed insights, key decisions, and nuance)

### Filename Sanitization

Before creating or looking up a recording file, sanitize the title:

| Replace | With |
|---------|------|
| `/`, `\`, `:`, `*`, `?`, `"`, `<`, `>`, `\|` | `-` |
| Multiple spaces | Single space |
| Leading/trailing whitespace | (removed) |
| Leading/trailing dots | (removed) |

- Truncate title to **100 characters max** (after sanitization)
- Preserve the `YYYY-MM-DD - ` prefix unchanged

**Example transformations**:
- `1-on-1 with: Bob / Smith` → `1-on-1 with - Bob - Smith`
- `  Interview - Alice.  ` → `Interview - Alice`
- `Very long title that goes on and on and exceeds reasonable length limits` → `Very long title that goes on and on and exceeds reasonable length li`

4. **Update Action Item Register**:
    - For each **newly synced** recording that contains action items:
        - Check if a section for the recording already exists in `Recordings/HeyPocket Action Items.md` (by searching for its unique heading).
        - If it does not exist, append the recording heading and its action items to the end of the file.
        - Ensure recordings are processed in chronological order (`recording_at`) during the sync to maintain the register's date-ascending structure over time.
    - If a recording has no action items, skip adding it to the register.
5. **Update Sync State**:
    - After a successful sync, update `agents/heypocket-sync/state.json` with the maximum `updated_at` timestamp among records actually processed.
    - If no newer records were processed, keep the existing `last_sync_timestamp` unchanged.

## Implementation Notes
- **Incremental Sync**: The `list recordings` endpoint does not expose an `updated_after` filter. Use `start_date/end_date` server-side filtering plus client-side `updated_at > last_sync_timestamp` checks for true incremental behavior.
- **Pagination**: Use `limit=100` and iterate `page` until no more results.
- **Error Handling**: Handle rate limits (429) and invalid tokens (401).
- **Action Item Register Format**:
    - File path: `Recordings/HeyPocket Action Items.md`.
    - Required sections per group:
        - Recording heading (linked): `## [[YYYY-MM-DD - {Title}]]`
        - Action list from API as Obsidian task lines using checkbox syntax:
            - Open item: `- [ ] {action label} (assignee: {name}, due: {date_or_none}, type: {action_type})`
            - Completed item: `- [x] {action label} (assignee: {name}, due: {date_or_none}, type: {action_type})`
    - Omit groups entirely when no actions are available for that recording.
    - **Persistence**: Append new records to the end of the file. Do not rewrite or modify existing entries once they have been added.
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
8. **Action item register checks**:
    - Ensure `Recordings/HeyPocket Action Items.md` was updated (appended) only for recordings newly processed in the current run.
    - Verify only recordings with one or more action items were added.
    - Verify each recording heading is an Obsidian link in this format: `## [[YYYY-MM-DD - {Title}]]`.
    - Verify each action item is an Obsidian task line in this format: `- [ ] ...` or `- [x] ...`.
    - Verify no duplicate sections were created for the same recording.
