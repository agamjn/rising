---
description: Daily Reddit content engine run. Logs any new posts the user made, refreshes engagement on recent ones, researches trends across all configured subs in parallel, and produces 3 post proposals saved to drafts/YYYY-MM-DD.md.
argument-hint: (no args)
---

You are the daily run command for the **rising** plugin. Produce 3 post proposals tailored to the user's product, voice, goals, and what's been working for them.

# Pre-flight check

Verify the CWD is set up. Look for:
- `product.md`, `goal.md`, `writing-style.md`, `subreddits.md`, `posts-history.md`, `.rising/config.json`

If any are missing, stop and tell the user: "This folder doesn't look set up. Run `/reddit-setup` first."

Also get today's date: run `date +%Y-%m-%d` once and reuse it.

Capture the User-Agent for all Reddit calls in this run:

```bash
USERNAME=$(jq -r .reddit_username .rising/config.json)
UA="script:com.github.agamjn.rising:v0.1.0 (by /u/${USERNAME})"
```

(Use this `$UA` in every curl below. **Never use WebFetch for Reddit URLs** — it omits the conformant User-Agent and Reddit will reject or rate-limit.)

# Step 1 — Refresh engagement on recent posts

Do NOT ask the user whether they've posted anything new — that flow is owned by the separate `/reddit-add-post` command and the user is reminded to run it in the wrap-up. The daily run must never block on interactive input.

Read `posts-history.md`. Identify entries from the last ~30 days (parse the ISO date in each header). For each, re-fetch its post URL via curl in parallel — single message, multiple Bash tool calls. Save each to `.rising/cache/post-<id>.json`:

```bash
curl -sS --fail-with-body -A "$UA" --connect-timeout 10 --max-time 30 \
  "<post-url>.json" \
  -o ".rising/cache/post-<id>.json"
```

Then update `score` and `num_comments` in `posts-history.md` from the refreshed JSON (in place — don't change titles, bodies, or comments).

If `posts-history.md` has no entries from the last 30 days, skip Step 2.

# Step 2 — Refresh post-categories.md

If any scores were updated in Step 1, launch the **reddit-performance-analyzer** agent. It will rewrite `post-categories.md`.

# Step 3 — Pre-fetch all subreddit data (orchestrator does this, NOT the researcher agents)

Read `subreddits.md` to get the full list of subs (both `posting` and `non-posting`).

**Clear and rebuild the per-sub cache** so today's run sees fresh data:

```bash
# Clear yesterday's per-sub cache (leave config.json and *-submissions.json intact)
find .rising/cache -maxdepth 1 -type f \( -name '*-top.json' -o -name '*-hot.json' -o -name '*-rules.json' \) -delete
```

For every sub in `subreddits.md`, fan out parallel curls — single message, multiple Bash tool calls (one per sub × per endpoint, or batch by sub). Each fetches:

```bash
SUB=<sub>

curl -sS --fail-with-body -A "$UA" --connect-timeout 10 --max-time 30 \
  "https://www.reddit.com/r/${SUB}/top.json?t=week&limit=25" \
  -o ".rising/cache/${SUB}-top.json"

curl -sS --fail-with-body -A "$UA" --connect-timeout 10 --max-time 30 \
  "https://www.reddit.com/r/${SUB}/hot.json?limit=25" \
  -o ".rising/cache/${SUB}-hot.json"
```

Track which subs' fetches succeeded. **Skip dispatching researchers for any sub whose top.json or hot.json fetch failed** (note them in the wrap-up).

# Step 4 — Research all subreddits in parallel

For every sub whose cache fetch succeeded in Step 3, dispatch a **reddit-subreddit-researcher** agent in parallel — all in a single message with multiple Agent tool calls.

Pass each researcher:
- `subreddit`: the sub name
- `category`: posting or non-posting (from `subreddits.md`)
- `cache_dir`: `.rising/cache`
- `product_context`: contents of `product.md`
- `goal`: contents of `goal.md`

Each researcher returns a trend brief in their final message (it reads the cache, no network).

# Step 5 — Synthesize 3 proposals

Once all researchers report back, launch the **reddit-post-synthesizer** agent. Pass it:
- `trend_briefs`: the array of all returned briefs
- `output_path`: `drafts/<YYYY-MM-DD>.md` (use today's date)

The synthesizer reads the rest of CWD (writing-style, post-categories, rules cache) directly and writes the drafts file.

# Step 6 — Display proposals in chat AND wrap up

After the synthesizer finishes writing `drafts/<YYYY-MM-DD>.md`, **read that file and paste its full contents into your response message** so the user can review the proposals inline without having to switch to the editor. Don't summarize, don't truncate — paste the whole file verbatim.

Then, **below the pasted proposals**, print the wrap-up:

```
---

Saved to: drafts/<YYYY-MM-DD>.md

<If any subs failed to fetch in Step 3:>
Skipped subs (curl failed): r/<sub1>, r/<sub2>

Pick one (or all), edit if needed, post manually on Reddit.
When you do, run /reddit-add-post <url> so the engine learns from it.
```

# Quality bar

- **Never use WebFetch for Reddit URLs.** Use Bash + curl with `-A "$UA"` exclusively.
- Curl fan-out must be parallel: multiple Bash tool calls in a single message. Sequential curl is the most common mistake — don't do it.
- Researcher dispatch must also be parallel: multiple Agent tool calls in a single message.
- If Step 3 fails for some subs, the run still continues with the rest. Only abort if **all** subs fail.
- If any researcher returns `ERROR: r/<sub> cache miss`, treat it as a soft failure — note in the wrap-up but don't fail the whole run.
- The daily run is fully non-interactive — never prompt the user for new post URLs. Logging new posts is the job of `/reddit-add-post`.
- File writes only in CWD.
