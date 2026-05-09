---
description: Schedule /reddit-daily to run automatically on a cron. Wraps Claude Code's built-in scheduling with a Reddit-engine-specific default.
argument-hint: <natural-language schedule, e.g. "weekdays at 9am">
---

You register `/reddit-daily` as a scheduled task using Claude Code's built-in scheduling.

# Pre-flight check

- Read `$ARGUMENTS`. If empty, ask: "When should /reddit-daily run? (e.g., 'weekdays at 9am', 'every day at 7am', 'monday wednesday friday at noon')"
- Verify CWD is set up (has `product.md`, etc.) — same check as `/reddit-daily`. If not, refuse and tell the user to run `/reddit-setup` first.

# Procedure

1. Get the absolute CWD: `pwd`.
2. Use the `schedule` skill (or `mcp__scheduled-tasks__create_scheduled_task` directly if available) to register a task with:
   - **Schedule**: parsed from the user's natural-language input
   - **Working directory**: the CWD captured above (so the daily run uses the right product folder)
   - **Prompt**: `/reddit-daily`
   - **Name/label**: `rising: <product folder name> daily`

3. Confirm to the user:

```
Scheduled.
  Product folder: <CWD>
  Cadence: <human-readable cadence>
  Next run: <when>

Use /reddit-schedule <new-cadence> to change it, or your scheduled-tasks UI to remove it.
```

# Quality bar

- The schedule must include the working directory — without it, the scheduled run won't know which product folder to operate in.
- If multiple folders are set up, each should have its own scheduled task. Don't try to multiplex.
- If the user re-runs this command in the same folder, replace the existing scheduled task rather than creating a duplicate.
