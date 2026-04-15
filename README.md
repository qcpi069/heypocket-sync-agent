# HeyPocket Sync Agent

Sync voice recordings from HeyPocket AI into meeting minutes in your Obsidian vault.

## Setup

### Prerequisites

- HeyPocket API key

### Obtaining Your API Key

1. Open the Pocket app on your iOS or Android device
2. Go to **Settings** → **Developer** → **API Keys**
3. Create a new API key (starts with `pk_...`)

Your API key will look like: `pk_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`

### Environment Variables

Set the `HEYPOCKET_API_TOKEN` environment variable:

```bash
export HEYPOCKET_API_TOKEN="pk_your_api_key_here"
```

## Documentation

- [API Reference](https://docs.heypocketai.com/docs/api) — Full API documentation for recordings, transcriptions, and summaries
- [Pocket Guide](https://guide.heypocket.com/) — Getting started and device setup

## Compatible AI Assistants

Tested with:
- [OpenCode](https://opencode-tutorial.com/en/getting-started)
- [Gemini](https://developers.google.com/gemini-code-assist)
- [Claude](https://code.claude.com/docs/en/desktop-quickstart.md) (Desktop / Claude Code)
- [Copilot](https://cursor.com/help/getting-started/first-project)

## Usage

> **Important:** This agent and sync skill expect to be run inside your Obsidian vault folder. The generated meeting notes will be saved directly to the `Recordings/` folder in your vault.
>
> If you run this agent outside of your Obsidian vault, you will need to manually move the generated notes into your vault.

1. Initialize this repo with your AI assistant inside your Obsidian vault folder
2. Run a sync command

### Commands

- "Sync my HeyPocket recordings"
- "Sync new recordings"
- "Sync all HeyPocket recordings"
- "Sync recordings from [date]"

## Output

Meeting notes are saved to the `Recordings/` folder in your Obsidian vault.

## Features

- **Incremental sync** — Fetches all recordings since the last sync (primary)
- Date-based sync — Sync recordings from a specific date
- Full sync — Sync all available recordings
- Search and filter — Find recordings by keywords, tags, or date range

See `skills/sync-recordings.md` for full capability details.