# Command and Permission Matrix (v1)

Updated: 2026-02-21  
Project: `billi-discord-agent`

## 1. Purpose
- Define who can run which commands, from where, and under what safety conditions.
- Make policy decisions explicit before implementation.

## 2. Risk Tier Definitions
- `Low`: read/list/status/search and non-mutating operations.
- `Medium`: posting, editing, deleting bot-managed content, add/remove config entries.
- `High`: full-list replacement, destructive cleanup/reset, moderation/structural actions.

## 3. Matrix

| command | source | allowed actor | allowed channels | risk tier | confirmation required | audit fields |
|---|---|---|---|---|---|---|
| `watch.listen` | `system` | runtime | watch allowlist only | Low | No | `event_id`, `channel_id`, `message_id`, `result`, `ts` |
| `capture.message.raw_append` | `system` | runtime | watch allowlist only | Low | No | `message_id`, `channel_id`, `file`, `result`, `ts` |
| `capture.message.normalize_append` | `system` | runtime | watch allowlist only | Low | No | `message_id`, `channel_id`, `csv_file`, `result`, `ts` |
| `context.channel.rollup` | `system` | runtime | watched channels | Low | No | `channel_id`, `input_range`, `output_file`, `result`, `ts` |
| `context.global.rollup` | `system` | runtime | n/a | Low | No | `input_sources`, `output_file`, `result`, `ts` |
| `health.self_check` | `system` | runtime | n/a | Low | No | `checks`, `status`, `latency_ms`, `ts` |
| `watch.add_channel` | `mcp` | owner | VS Code control only | Medium | No | `actor`, `channel_id`, `old_state`, `new_state`, `result`, `ts` |
| `watch.remove_channel` | `mcp` | owner | VS Code control only | Medium | No | `actor`, `channel_id`, `old_state`, `new_state`, `result`, `ts` |
| `watch.list_channels` | `mcp` | owner | VS Code control only | Low | No | `actor`, `count`, `result`, `ts` |
| `watch.set_channels` | `mcp` | owner | VS Code control only | High | Yes | `actor`, `old_list_hash`, `new_list_hash`, `result`, `ts` |
| `broadcast.add_channel` | `mcp` | owner | VS Code control only | Medium | No | `actor`, `channel_id`, `old_state`, `new_state`, `result`, `ts` |
| `broadcast.remove_channel` | `mcp` | owner | VS Code control only | Medium | No | `actor`, `channel_id`, `old_state`, `new_state`, `result`, `ts` |
| `broadcast.list_channels` | `mcp` | owner | VS Code control only | Low | No | `actor`, `count`, `result`, `ts` |
| `broadcast.set_channels` | `mcp` | owner | VS Code control only | High | Yes | `actor`, `old_list_hash`, `new_list_hash`, `result`, `ts` |
| `post.preview` | `mcp` | owner | target must be in broadcast allowlist | Low | No | `actor`, `template`, `target_channel`, `preview_hash`, `result`, `ts` |
| `post.publish` | `mcp` | owner | target must be in broadcast allowlist | Medium | No | `actor`, `template`, `target_channel`, `message_hash`, `result`, `ts` |
| `post.broadcast` | `mcp` | owner | targets must be in broadcast allowlist | Medium | No | `actor`, `template`, `target_channels`, `message_hash`, `result`, `ts` |
| `post.edit` | `mcp` | owner | bot-managed messages in allowed channels | Medium | No | `actor`, `message_id`, `target_channel`, `new_hash`, `result`, `ts` |
| `post.delete` | `mcp` | owner | bot-managed messages in allowed channels | Medium (or High if you choose) | Optional (recommended) | `actor`, `message_id`, `target_channel`, `reason`, `result`, `ts` |
| `template.add` | `mcp` | owner | VS Code control only | Medium | No | `actor`, `template_name`, `template_hash`, `result`, `ts` |
| `template.update` | `mcp` | owner | VS Code control only | Medium | No | `actor`, `template_name`, `old_hash`, `new_hash`, `result`, `ts` |
| `template.remove` | `mcp` | owner | VS Code control only | Medium | Optional (recommended) | `actor`, `template_name`, `reason`, `result`, `ts` |
| `template.list` | `mcp` | owner | VS Code control only | Low | No | `actor`, `count`, `result`, `ts` |
| `template.inspect` | `mcp` | owner | VS Code control only | Low | No | `actor`, `template_name`, `result`, `ts` |
| `mention_command.enable` | `mcp` | owner | VS Code control only | Medium | No | `actor`, `command`, `old_state`, `new_state`, `result`, `ts` |
| `mention_command.disable` | `mcp` | owner | VS Code control only | Medium | No | `actor`, `command`, `old_state`, `new_state`, `result`, `ts` |
| `mention_command.list` | `mcp` | owner | VS Code control only | Low | No | `actor`, `count`, `result`, `ts` |
| `mention_channel.add` | `mcp` | owner | VS Code control only | Medium | No | `actor`, `channel_id`, `old_state`, `new_state`, `result`, `ts` |
| `mention_channel.remove` | `mcp` | owner | VS Code control only | Medium | No | `actor`, `channel_id`, `old_state`, `new_state`, `result`, `ts` |
| `mention_channel.list` | `mcp` | owner | VS Code control only | Low | No | `actor`, `count`, `result`, `ts` |
| `provider.execution.set_primary` | `mcp` | owner | VS Code control only | High | Yes | `actor`, `old_model`, `new_model`, `result`, `ts` |
| `provider.execution.set_fallbacks` | `mcp` | owner | VS Code control only | High | Yes | `actor`, `old_models`, `new_models`, `result`, `ts` |
| `provider.chat_pool.add_model` | `mcp` | owner | VS Code control only | Medium | No | `actor`, `model_id`, `old_state`, `new_state`, `result`, `ts` |
| `provider.chat_pool.remove_model` | `mcp` | owner | VS Code control only | Medium | No | `actor`, `model_id`, `old_state`, `new_state`, `result`, `ts` |
| `provider.chat_pool.list_models` | `mcp` | owner | VS Code control only | Low | No | `actor`, `count`, `result`, `ts` |
| `provider.chat_pool.disable` | `mcp` | owner | VS Code control only | Medium | No | `actor`, `old_state`, `new_state`, `result`, `ts` |
| `provider.chat_pool.enable` | `mcp` | owner | VS Code control only | Medium | No | `actor`, `old_state`, `new_state`, `result`, `ts` |
| `retention.set_policy` | `mcp` | owner | VS Code control only | High | Yes | `actor`, `old_policy_hash`, `new_policy_hash`, `result`, `ts` |
| `retention.get_policy` | `mcp` | owner | VS Code control only | Low | No | `actor`, `result`, `ts` |
| `retention.prune_now` | `mcp` | owner | VS Code control only | High | Yes | `actor`, `scope`, `deleted_counts`, `result`, `ts` |
| `context.channel.reset` | `mcp` | owner | VS Code control only | High | Yes | `actor`, `channel_id`, `reason`, `result`, `ts` |
| `context.global.reset` | `mcp` | owner | VS Code control only | High | Yes | `actor`, `reason`, `result`, `ts` |
| `capture.search` | `mcp` | owner | VS Code control only | Low | No | `actor`, `filters`, `result_count`, `result`, `ts` |
| `capture.export_csv` | `mcp` | owner | VS Code control only | Low | No | `actor`, `filters`, `output_file`, `result`, `ts` |
| `capture.rebuild_staging` | `mcp` | owner | VS Code control only | High | Yes | `actor`, `input_range`, `output_file`, `result`, `ts` |
| `status.runtime` | `mcp` | owner | VS Code control only | Low | No | `actor`, `uptime`, `status`, `ts` |
| `status.providers` | `mcp` | owner | VS Code control only | Low | No | `actor`, `provider_status`, `failover_count`, `ts` |
| `status.watchers` | `mcp` | owner | VS Code control only | Low | No | `actor`, `watch_count`, `status`, `ts` |
| `audit.list` | `mcp` | owner | VS Code control only | Low | No | `actor`, `limit`, `result_count`, `ts` |
| `audit.get` | `mcp` | owner | VS Code control only | Low | No | `actor`, `action_id`, `result`, `ts` |
| `confirm.action` | `mcp` | owner | VS Code control only | High | n/a | `actor`, `action_id`, `token_valid`, `result`, `ts` |
| `cancel.action` | `mcp` | owner | VS Code control only | Medium | No | `actor`, `action_id`, `result`, `ts` |
| `help` | `mention` | any member | mention allowlist channels | Low | No | `actor`, `channel_id`, `result`, `ts` |
| `chat` | `mention` | any member | mention allowlist channels | Low | No | `actor`, `channel_id`, `provider`, `model`, `result`, `ts` |
| `status` | `mention` | any member | mention allowlist channels | Low | No | `actor`, `channel_id`, `result`, `ts` |
| `last_post` | `mention` | any member | mention allowlist channels | Low | No | `actor`, `channel_id`, `target_channel`, `result`, `ts` |
| `posts_today` | `mention` | any member | mention allowlist channels | Low | No | `actor`, `channel_id`, `target_scope`, `result`, `ts` |
| `find_logs` | `mention` | trusted role or any member (decide) | mention allowlist channels | Low | No | `actor`, `channel_id`, `query`, `result_count`, `ts` |

## 4. Explicit Denials (Phase 1)
- Mention source cannot run moderation actions (`kick`, `ban`, role changes, channel deletes).
- Mention source cannot change config (`watch`, `broadcast`, `template`, `provider`, `retention`).
- Chat lane output cannot directly bypass this matrix to execute tools.

## 5. Open Decisions
- Final rule for `find_logs` access (`any member` vs `trusted role`).
- Final risk tier for `post.delete`.
- Whether `template.remove` should always require confirmation.
- Whether `provider.chat_pool.disable` should require confirmation.
