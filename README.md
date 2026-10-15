# Tally Workout MCP server

Docs for the remote [Model Context Protocol](https://modelcontextprotocol.io) server of
[Tally Workout](https://tallyworkout.app), a free iPhone strength log. Your AI agent reads
your training history and plans workouts in the Tally app. You log every set at the gym.
Next week, the agent plans from what you actually lifted.

It works with Claude, Claude Code, Codex, ChatGPT, Muse, Hark, Perplexity, Dots and any
MCP client that supports OAuth 2.1 or a bearer token.

> This repo holds the public docs. The server is first-party and closed source. It runs
> as part of the Tally backend.

## Endpoint

```
https://mcp.tallyworkout.app/api/mcp
```

- **Transport:** Streamable HTTP. No SSE-only mode.
- **Auth, option 1:** OAuth 2.1, authorization code + PKCE (S256), public client, dynamic
  client registration (RFC 7591). Scope: `workouts`.
- **Auth, option 2:** an API key, for clients without OAuth. Create it in the Tally app
  under Settings → API keys. Send it as `Authorization: Bearer tally_ct_…`. A key is
  Read only or Read and plan. Keys do not expire. Revoke a key in the app.
- **Discovery:** [`/.well-known/mcp-server`](https://mcp.tallyworkout.app/.well-known/mcp-server),
  [agent brief (llms.txt)](https://mcp.tallyworkout.app/llms.txt).

## Setup by agent

You need the [Tally iPhone app](https://apps.apple.com/app/id6758911546) and a Tally account.
Each link has the full steps.

| Agent | Auth | Steps |
|-------|------|-------|
| Claude (web, desktop, phone) | OAuth | Add Tally Workout from [Claude's directory](https://claude.ai/directory/tally-workout). [Steps](https://tallyworkout.app/connect/claude) |
| Claude Code | OAuth or API key | `claude mcp add --scope user --transport http tally https://mcp.tallyworkout.app/api/mcp`, then `/mcp`. [Steps](https://tallyworkout.app/connect/claude-code) |
| Codex | API key | Three lines in `~/.codex/config.toml`. [Steps](https://tallyworkout.app/connect/codex) |
| ChatGPT | OAuth | Developer mode. Pro reads only. Business, Enterprise and Edu can plan. [Steps](https://tallyworkout.app/connect/chatgpt) |
| Muse | OAuth | Client ID `tally-muse`. [Steps](https://tallyworkout.app/connect/muse) |
| Hark | API key | [Steps](https://tallyworkout.app/connect/hark) |
| Perplexity | API key | [Steps](https://tallyworkout.app/connect/perplexity) |
| Dots | API key | Not tested yet. [Steps](https://tallyworkout.app/connect/dots) |
| Any MCP client | OAuth or API key | [Steps](https://tallyworkout.app/connect/other) |

## Tools

11 tools: 5 read, 6 write.

| Tool | Type | What it does |
|------|------|--------------|
| `get_training_context` | read | Returns planned days from today, the weight unit and, with `historyWeeks`, recent workouts. |
| `get_workout_history` | read | Returns logged workouts, newest first, set by set. |
| `get_exercise_progress` | read | Returns progress for up to 10 exercises over the last 365 days, with top sets and estimated 1RM. |
| `get_planned_workout` | read | Returns one planned day in full, with its version. |
| `search_exercises` | read | Matches exercise names to the user's recent exercises and the Tally library. |
| `plan_workout_day` | write | Creates or replaces one planned day. |
| `plan_workout_range` | write | Creates or replaces up to 31 planned days in one call. |
| `update_planned_workout` | write | Changes part of one planned day: add, remove or change exercises. |
| `move_planned_workout` | write | Moves a planned day to an empty date. |
| `skip_planned_workout` | write | Marks a planned day as skipped, or restores it. Only the status changes. |
| `delete_planned_workout` | write | Removes a planned day and returns it, so you can undo. |

## Rules

- **Logged sets are never written.** Write tools change planned days only.
- **Some days are protected.** A day the user built in the app, or edited, is protected.
  The plan, update, move and delete tools skip it unless the user asks for an override.
- **Edits check the version.** If a day changed after the agent read it, nothing is
  written and the result reports a conflict.
- **Each user sees only their own data.** OAuth tokens and API keys are scoped to one
  account. The user can revoke them at any time.

## Links

- Site: https://tallyworkout.app
- Builder reference: https://tallyworkout.app/connect/mcp
- App Store: https://apps.apple.com/app/id6758911546
- Support: support@tallyworkout.app
