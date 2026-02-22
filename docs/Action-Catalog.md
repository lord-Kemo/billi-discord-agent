# Action Catalog (v1)

Updated: 2026-02-21  
Project: `billi-discord-agent`

## 1. Purpose
- Define all actions the agent can perform in phase 1.
- Separate automatic behaviors from owner-controlled commands and public mention commands.
- Establish a clear inventory before implementation.

## 2. Action Sources
- `system`: automatic background actions performed by the bot runtime.
- `mcp` (VS Code): owner-controlled commands via MCP tools.
- `mention` (Discord): restricted commands via bot mention in Discord text channels.

## 3. Core Principles
- VS Code is the main control panel.
- Mention commands are restricted and low-risk by default.
- Tool execution and mutating actions must be auditable.
- Add/remove configuration actions are owner-only (via VS Code/MCP).

## 4. Automatic Background Actions (`system`)

### 4.1 Channel Listening and Capture
- `watch.listen`
  - Continuously monitor configured watch channels.
  - Trigger on new messages in allowlisted channels.
- `capture.message.raw_append`
  - Save each watched message event to a local NDJSON log file.
- `capture.message.normalize_append`
  - Append a normalized row to a file-based "sheet staging" CSV (future Google Sheets sync target).
- `capture.message.attachment_ref`
  - Store attachment metadata/reference (URL, filename, size) without downloading file content by default.

### 4.2 Published Post Tracking
- `publish.detect_bot_posts`
  - Detect messages posted by the bot in configured broadcast channels.
- `publish.log_append`
  - Save published post metadata to local NDJSON for auditing and lookup.

### 4.3 Context Maintenance
- `context.channel.rollup`
  - Update rolling channel summary/context files from captured messages.
- `context.global.rollup`
  - Update global operational context (recent actions, notable events, pending items).

### 4.4 Runtime Health
- `health.self_check`
  - Periodically verify Discord gateway connection, file write access, and provider status.
- `health.reconnect`
  - Attempt safe reconnect when disconnected.

## 5. Owner Actions via VS Code (`mcp`)

### 5.1 Watch Channel Management (Add/Remove)
- `watch.add_channel`
  - Add a Discord channel to the watch list.
- `watch.remove_channel`
  - Remove a Discord channel from the watch list.
- `watch.list_channels`
  - Show all watched channels.
- `watch.set_channels`
  - Replace the entire watch list (high-risk config action).

### 5.2 Broadcast Channel Management (Add/Remove)
- `broadcast.add_channel`
  - Add a channel to allowed broadcast targets.
- `broadcast.remove_channel`
  - Remove a channel from allowed broadcast targets.
- `broadcast.list_channels`
  - List configured broadcast channels.
- `broadcast.set_channels`
  - Replace broadcast target list (high-risk config action).

### 5.3 Posting and Broadcast Operations
- `post.preview`
  - Render a themed post preview from template + variables.
- `post.publish`
  - Publish one formatted post to a specific allowed channel.
- `post.broadcast`
  - Publish one formatted post to multiple allowed channels.
- `post.edit`
  - Edit a previously posted bot message.
- `post.delete`
  - Delete a previously posted bot message.

### 5.4 Template Management (Add/Remove)
- `template.add`
  - Create a new post template.
- `template.update`
  - Update an existing template.
- `template.remove`
  - Remove a template (confirmation recommended if actively used).
- `template.list`
  - List available templates.
- `template.inspect`
  - Show template content and metadata.

### 5.5 Mention Command Policy Management (Add/Remove)
- `mention_command.enable`
  - Enable a mention command in policy.
- `mention_command.disable`
  - Disable a mention command in policy.
- `mention_command.list`
  - List mention commands and enabled state.
- `mention_channel.add`
  - Add a channel where mention commands are allowed.
- `mention_channel.remove`
  - Remove a channel from mention allowlist.
- `mention_channel.list`
  - List mention-enabled channels.

### 5.6 Provider Routing and Model Pool Management (Add/Remove)
- `provider.execution.set_primary`
  - Set pinned Gemini execution model.
- `provider.execution.set_fallbacks`
  - Set pinned OpenRouter fallback models.
- `provider.chat_pool.add_model`
  - Add a model to OpenRouter chat throughput pool.
- `provider.chat_pool.remove_model`
  - Remove a model from chat throughput pool.
- `provider.chat_pool.list_models`
  - List chat pool models.
- `provider.chat_pool.disable`
  - Disable chat pool routing temporarily (emergency control).
- `provider.chat_pool.enable`
  - Re-enable chat pool routing.

### 5.7 Data and Retention Controls (Add/Remove/Prune)
- `retention.set_policy`
  - Set retention windows for logs/context.
- `retention.get_policy`
  - View current retention policy.
- `retention.prune_now`
  - Run pruning manually (high-risk if irreversible).
- `context.channel.reset`
  - Reset one channel summary/context file (confirmation required).
- `context.global.reset`
  - Reset global context file (confirmation required).

### 5.8 Capture and Export Operations
- `capture.search`
  - Search captured messages with filters (channel, time, keyword).
- `capture.export_csv`
  - Export selected captured data to CSV.
- `capture.rebuild_staging`
  - Rebuild the sheet-staging file from raw logs.

### 5.9 Status, Audit, and Confirmation
- `status.runtime`
  - Show bot runtime health and uptime.
- `status.providers`
  - Show Gemini/OpenRouter provider health and recent failures.
- `status.watchers`
  - Show watch listeners and channel counts.
- `audit.list`
  - List recent actions and outcomes.
- `audit.get`
  - View one action record in detail.
- `confirm.action`
  - Confirm pending high-risk action using token.
- `cancel.action`
  - Cancel pending action.

## 6. Discord Mention Actions (`mention`)

### 6.1 General Chat / Fun
- `help`
  - Show allowed mention commands.
- `chat`
  - General conversation / fun responses.

### 6.2 Read-Only Utility
- `status`
  - Safe status summary (no sensitive details).
- `last_post`
  - Show last bot post in a configured broadcast channel.
- `posts_today`
  - Summarize what was posted today in configured broadcast channels.
- `find_logs`
  - Search watched logs for a keyword/timeframe (read-only).

### 6.3 Mention Restrictions
- Mention commands must be limited to allowlisted channels.
- Mention commands must not directly perform moderation or high-risk server actions.
- Mention commands may request data, summaries, and safe posting status checks only.

## 7. Deferred / Later Phase Actions
- Voice channel join/listen/speak (`voice.join`, `voice.listen`, `voice.speak`) deferred to later phase.
- Google Sheets direct sync (`sheets.append_rows`, `sheets.sync`) deferred; file staging only in phase 1.
- Local LLM model fallback controls deferred to phase 2.

## 8. Risk Categorization (Action-Level)
- `Low`
  - Read/list/status/search, template inspect/list, policy list actions.
- `Medium`
  - Post publish/edit/delete, add/remove non-destructive config entries.
- `High`
  - Replace full config lists, prune/reset context/data, moderation/structural actions (if later enabled).

## 9. Open Decisions
- Final mention command set for phase 1.
- Whether `post.delete` is medium or high in your server context.
- Whether `watch.set_channels` and `broadcast.set_channels` require confirmation by default.
- Whether template removal requires confirmation always or only when recently used.
