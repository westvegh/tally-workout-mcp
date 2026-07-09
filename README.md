# Tally Workout — MCP Server

Remote [Model Context Protocol](https://modelcontextprotocol.io) server for
[Tally](https://tallyworkout.app), the strength-training app Claude can write
to. Plan your training in conversation — it lands in the app, ready to log.

> This repo is the public documentation anchor for the server. The server
> itself is first-party, closed-source, and runs as part of the Tally backend.

## Endpoint

```
https://mcp.tallyworkout.app/api/mcp
```

- **Transport:** Streamable HTTP
- **Auth:** OAuth 2.1 — PKCE + dynamic client registration. One-click from
  claude.ai custom connectors (free tier includes one custom connector slot).
- **Pricing:** Free. Bring your own Claude account.

## Setup (~2 minutes)

1. Get the free iOS app: [Tally on the App Store](https://apps.apple.com/us/app/tally-workout-app/id6758911546)
2. Follow the guide at [tallyworkout.app/connect](https://tallyworkout.app/connect) —
   paste the endpoint URL into claude.ai → Settings → Connectors → Add custom
   connector, then sign in with your Tally account.

Claude Code:

```bash
claude mcp add --transport http tally https://mcp.tallyworkout.app/api/mcp
```

## Tools

| Tool | What it does |
|------|--------------|
| `get_training_context` | Snapshot of recent training: volume, frequency, exercises, adherence |
| `get_workout_history` | Full workout history — sets, reps, weights, planned vs. actually performed |
| `get_planned_workout` | Read a planned day (for read-modify-write edits) |
| `plan_workout_day` | Write a planned workout for a single day into the app |
| `plan_workout_range` | Write a whole week (or any range) of planned workouts |
| `search_exercises` | Search Tally's exercise library |

## Design notes

- **Human edits win.** Writes use compare-and-swap semantics; anything you
  changed or logged on the gym floor is treated as truth and never overwritten
  by the model.
- **Planned vs. actual is first-class.** History payloads show what was
  planned *and* what you actually did, so programming advice is grounded in
  adherence, not aspirations.
- **Per-user isolation.** OAuth-scoped tokens with row-level security;
  revocable anytime from Claude's connector settings.

## Links

- App: https://apps.apple.com/us/app/tally-workout-app/id6758911546
- Site: https://tallyworkout.app
- Connect guide: https://tallyworkout.app/connect
- Support: support@tallyworkout.app
