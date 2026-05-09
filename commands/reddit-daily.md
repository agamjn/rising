---
description: Daily Reddit content engine run. Logs any new posts the user made, refreshes engagement on recent ones, researches trends across all configured subs in parallel, and produces 3 post proposals saved to drafts/YYYY-MM-DD.md.
argument-hint: (no args)
---

You are the daily run command for the **rising** plugin. Produce 3 post proposals tailored to the user's product, voice, goals, and what's been working for them.

# Pre-flight check

Verify the CWD is set up. Look for:
- `product.md`, `goal.md`, `writing-style.md`, `subreddits.md`, `posts-history.md`

If any are missing, stop and tell the user: "This folder doesn't look set up. Run `/reddit-setup` first."

Also get today's date: run `date +%Y-%m-%d` once and reuse it.

# Step 1 — Log any new posts

Ask the user: "Posted anything on Reddit since last run? Paste URL(s) — one per line — or hit enter to skip."

If they paste URLs, hand them off to the same logic as `/reddit-add-post`:
1. For each URL, append `.json` and WebFetch it (User-Agent `rising/0.1`).
2. Extract title, selftext, subreddit, permalink, score, num_comments, created_utc, top 3 comments.
3. **Append** an entry to `posts-history.md` (don't overwrite — this file is append-only). Use the same format as `/reddit-setup`.
4. Note that we'll refresh `post-categories.md` later (after engagement refresh).

If skip, move on.

# Step 2 — Refresh engagement on recent posts

Read `posts-history.md`. Identify entries from the last ~30 days (parse the ISO date in each header). For each, re-fetch the post URL (`<permalink>.json`) and update `score` and `num_comments` in place. Don't change titles or bodies. This keeps performance data current as old posts accumulate engagement.

If nothing was logged in step 1 AND no recent posts exist, you can skip step 3.

# Step 3 — Refresh post-categories.md

If anything changed in `posts-history.md` (new posts logged or scores updated), launch the **reddit-performance-analyzer** agent. It will rewrite `post-categories.md`.

# Step 4 — Research all subreddits in parallel

Read `subreddits.md` to get the full list of subs (both `posting` and `non-posting`). For each sub, dispatch a **reddit-subreddit-researcher** agent in parallel — all in a single message with multiple Agent tool calls.

Pass each researcher:
- `subreddit`: the sub name
- `category`: posting or non-posting (from `subreddits.md`)
- `product_context`: contents of `product.md`
- `goal`: contents of `goal.md`

Each researcher returns a trend brief in their final message.

# Step 5 — Synthesize 3 proposals

Once all researchers report back, launch the **reddit-post-synthesizer** agent. Pass it:
- `trend_briefs`: the array of all returned briefs
- `output_path`: `drafts/<YYYY-MM-DD>.md` (use today's date)

The synthesizer reads the rest of CWD (writing-style, post-categories, rules cache) directly and writes the drafts file.

# Step 6 — Wrap up

Once the synthesizer finishes, show the user:

```
Daily run complete.

Drafts: drafts/<YYYY-MM-DD>.md
  Proposal 1: <category> → r/<sub> — "<title>"
  Proposal 2: <category> → r/<sub> — "<title>"
  Proposal 3: <category> → r/<sub> — "<title>"

Open the file, pick one (or all), edit if needed, post manually on Reddit.
When you do, run /reddit-add-post <url> so the engine learns from it.
```

# Quality bar

- Researcher dispatch must be parallel (single message, multiple tool calls). Sequential dispatch is the most common mistake — don't do it.
- If any researcher returns `ERROR: ...`, note it in the wrap-up but don't fail the whole run; the synthesizer can work with whatever briefs succeeded.
- Never invent post URLs in step 2 — only use what the user actually pasted.
- File writes only in CWD.
