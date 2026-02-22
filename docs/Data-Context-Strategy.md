# Data and Context Strategy (Phase 1, File-First)

Updated: 2026-02-21  
Project: `billi-discord-agent`

## 1. Purpose
- Define where the bot stores captured data, action logs, templates, and context.
- Use local files first for speed and simplicity.
- Keep a clear upgrade path to Google Sheets later.

## 2. Storage Principles
- Phase 1 is file-first and local-only.
- Use append-only logs for events and actions when possible.
- Keep configuration/state in small JSON files for easy manual inspection.
- Keep derived context separate from raw logs.
- Treat Google Sheets as a future sync target, not the primary source in phase 1.

## 3. Proposed Storage Layout

All paths below are relative to repo root:

```text
data/
  audit/
    actions.ndjson
    provider-calls.ndjson
  capture/
    watched-messages.ndjson
    attachments.ndjson
  exports/
    sheet-staging.csv
    manual-exports/
  publish/
    posts-log.ndjson
  state/
    watch-channels.json
    broadcast-channels.json
    mention-policy.json
    templates.json
    provider-policy.json
    retention-policy.json
    pending-confirmations.json
  context/
    global.md
    channels/
      <channel-id>.md
```

## 4. What Each File Stores

### 4.1 Raw Capture Files
- `data/capture/watched-messages.ndjson`
  - Raw captured message events from watched channels.
  - Primary source for reprocessing and rebuilding exports.
- `data/capture/attachments.ndjson`
  - Attachment metadata only by default (message ID, URL, type, size).

### 4.2 Sheet Staging (Google Sheets Later)
- `data/exports/sheet-staging.csv`
  - Normalized rows intended for future Google Sheets append.
  - Phase 1 writes to this file only.
- `data/exports/manual-exports/`
  - One-off CSV exports generated from searches or filtered ranges.

### 4.3 Publish History
- `data/publish/posts-log.ndjson`
  - Records bot-published messages in broadcast channels.
  - Supports mention queries like "what was posted today?" and "last post in X".

### 4.4 Audit and Provider Logs
- `data/audit/actions.ndjson`
  - Every command, tool call, policy denial, and action result.
- `data/audit/provider-calls.ndjson`
  - Provider/model selection, latency, errors, and failovers for observability.

### 4.5 Config and State Files (Owner Add/Remove Controls)
- `data/state/watch-channels.json`
  - Channels the bot listens to continuously.
  - Updated by `watch.add_channel` and `watch.remove_channel`.
- `data/state/broadcast-channels.json`
  - Channels allowed for posting/broadcast actions.
  - Updated by `broadcast.add_channel` and `broadcast.remove_channel`.
- `data/state/mention-policy.json`
  - Enabled mention commands and mention-allowlist channels.
  - Updated by `mention_command.*` and `mention_channel.*`.
- `data/state/templates.json`
  - Server-themed post templates and metadata.
  - Updated by `template.add`, `template.update`, `template.remove`.
- `data/state/provider-policy.json`
  - Pinned execution models and OpenRouter chat pool list.
  - Updated by provider config actions.
- `data/state/retention-policy.json`
  - Retention windows and pruning behavior.
  - Updated by `retention.set_policy`.
- `data/state/pending-confirmations.json`
  - High-risk actions awaiting owner confirmation token.

### 4.6 Context Files
- `data/context/channels/<channel-id>.md`
  - Rolling summary/memory for each watched channel.
- `data/context/global.md`
  - Global operational context: active configs, recent actions, pending issues.

## 5. File Formats (Recommended)

### 5.1 NDJSON for Logs
- One JSON object per line.
- Easy to append.
- Easy to parse, grep, and rebuild.

Example (conceptual):

```json
{"ts":"2026-02-21T20:00:00Z","channel_id":"123","message_id":"456","author_id":"789","content":"hello","source":"watch"}
```

### 5.2 JSON for State
- Human-readable structured config.
- Version each file with a top-level `version` field.
- Include `updated_at` and `updated_by` where applicable.

### 5.3 CSV for Sheet Staging
- Normalized columns only.
- Keep a stable header row to make later Google Sheets sync simple.
- Add a `row_id`/`event_id` column to avoid duplicates.

## 6. Context Strategy (What the Agent Remembers)

### 6.1 Context Layers
- Raw event layer:
  - Full captured messages (file log).
- Operational layer:
  - Actions, posts, denials, confirmations.
- Summarized channel layer:
  - Rolling summaries per watched channel.
- Global summary layer:
  - Cross-channel and operational state summary.

### 6.2 Memory Rules (Phase 1)
- Do not rely on the LLM provider to store memory.
- Persist everything needed for continuity in local files.
- Rebuild context summaries from raw logs if needed.
- Treat context files as derived, not authoritative.

## 7. Retention and Pruning (Configurable)
- Keep raw capture logs for a configurable window (example: 30-90 days).
- Keep audit logs longer than chat logs (example: 90-180 days).
- Keep publish logs longer for reporting/history (example: 180+ days).
- Context files are overwritten/rolled forward but can be snapshotted.
- Manual prune actions must be audited and usually require confirmation.

## 8. Privacy and Safety Notes
- Do not store secrets/tokens in `data/`.
- Store API keys in environment variables or secret store only.
- By default, store attachment metadata, not downloaded attachments.
- If later enabling attachment download, store only in an explicit opt-in path and policy.

## 9. Upgrade Path to Google Sheets (Later)
- Keep `sheet-staging.csv` as the canonical export format for phase 1.
- Add a sync job later that reads unsynced rows and appends to Google Sheets.
- Preserve `row_id` / `event_id` to support idempotent sync.
- Continue local file logging even after Google Sheets integration (for recovery and audit).

## 10. Open Decisions
- Exact CSV columns for `sheet-staging.csv`.
- Retention windows by file type.
- Whether attachments should remain metadata-only in phase 1.
- Whether `posts-log.ndjson` stores rendered content or content hashes only.
