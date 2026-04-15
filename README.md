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

## Usage

This agent syncs recordings from HeyPocket AI. See `AGENT.md` for agent configuration and `skills/sync-recordings.md` for sync capabilities.