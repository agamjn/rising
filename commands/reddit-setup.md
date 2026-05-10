---
description: One-time setup wizard for the Reddit content engine. Gathers product brief, Reddit username, and goals; analyzes the user's writing style; discovers target subreddits; caches sub rules; classifies past posts by performance.
argument-hint: (interactive — no args)
---

You are the setup wizard for the **rising** plugin. Walk the user through a one-time setup that produces all the config files for their per-product Reddit folder.

# Pre-flight check

Confirm CWD is appropriate before doing anything destructive. Run:
- `pwd` to show the user where you are
- `ls -la` to show what's already here

If `product.md` already exists, the user has already set up here. Ask: "Setup files already exist in this folder. Re-run setup (overwrites everything) or cancel?" Wait for an explicit answer.

# Step 1 — Product brief

Ask the user: "What's your product? Paste a website URL (I'll fetch it) or describe it in a paragraph."

- If URL: WebFetch it, extract the actual product description (not nav/footer cruft).
- If text: take it as-is.

Write `product.md` containing the brief — keep it tight, 100-200 words, focused on what the product does + who it's for.

# Step 2 — Goal

Ask: "What do you want out of Reddit? (e.g., 'increase brand presence in the ML/infra community', 'drive signups from devops engineers', 'build karma so I can post in stricter subs', 'recruit beta testers')"

Write `goal.md` with their answer + a brief expansion of what success looks like for that goal.

# Step 3 — Reddit username + profile-visibility caveat

**This step has a critical caveat. Surface it BEFORE asking for the username:**

> "I need to fetch your past Reddit posts to learn your writing style and figure out what's worked for you. Reddit's public JSON API only returns submissions for **public** profiles. If yours is hidden:
>
> 1. Go to https://www.reddit.com/settings → **Profile** tab
> 2. Toggle **ON**: 'Show active communities' and 'Make my profile visible to search engines'
> 3. Save. Come back here.
> 4. After this setup finishes, you can flip both toggles **OFF** again — from then on you'll log new posts manually with `/reddit-add-post <url>` so we keep growing your history without your profile being public.
>
> Ready? Paste your Reddit username (without the `u/`):"

Wait for the username.

# Step 4 — Persist config and seed cache directory

Run this Bash command to write `.rising/config.json` and prepare the cache:

```bash
mkdir -p .rising/cache
cat > .rising/config.json <<JSON
{
  "reddit_username": "<username>"
}
JSON

# Add .rising/ to .gitignore (creates if missing, appends if missing the line)
touch .gitignore
grep -qxF '.rising/' .gitignore || echo '.rising/' >> .gitignore
```

Substitute `<username>` with the value the user gave.

# Step 5 — Pull post history (Bash + curl)

Reddit's API rejects requests without a conformant User-Agent. **Always use Bash + curl, never WebFetch, for Reddit endpoints.**

Run this Bash command:

```bash
USERNAME=$(jq -r .reddit_username .rising/config.json)
UA="script:com.github.agamjn.rising:v0.1.0 (by /u/${USERNAME})"
curl -sS --fail-with-body \
  -A "$UA" \
  --connect-timeout 10 --max-time 30 \
  "https://www.reddit.com/user/${USERNAME}/submitted.json?limit=100" \
  -o ".rising/cache/${USERNAME}-submissions.json"
```

If the curl exits non-zero or the resulting JSON has no `data.children` (empty list), tell the user their profile is likely hidden — point them back to the toggle steps and wait for them to retry.

Read the saved JSON and parse each post. For each post in `data.children[].data`, extract: `title`, `selftext`, `subreddit`, `permalink`, `created_utc` (convert to ISO date), `score`, `num_comments`, `link_flair_text`. Also pull the top 3 comments by score for context — for each post, fetch its comments via:

```bash
curl -sS --fail-with-body -A "$UA" --connect-timeout 10 --max-time 30 \
  "https://www.reddit.com<permalink>.json?limit=100" \
  -o ".rising/cache/post-<id>.json"
```

Issue these per-post comment fetches in parallel — multiple Bash tool calls in a single message. (Up to ~20 at a time is fine; Reddit's unauthenticated rate limit is ~60/min.)

Write `posts-history.md` as the canonical append-only log. Format each entry as:

```markdown
## <ISO-date> — r/<sub> — "<title>"
- URL: https://reddit.com<permalink>
- Score: <score> | Comments: <num_comments>
- Flair: <link_flair_text or "none">

<selftext (or "[link post: <url>]" for link posts)>

**Top comments:**
- (<score>) <username>: <truncated 200 chars>
- ...

---
```

Order: newest first. This file is the source of truth for every other analysis.

# Step 6 — Brainstorm subreddit candidates and validate them (orchestrator does this, NOT the discoverer agent)

You (the orchestrator) brainstorm an initial list of ~10-15 subreddit candidates based on the product brief and goal. Think across:
- Where the target *user* hangs out (posting candidates)
- Where competitors get discussed or compete (non-posting candidates)
- Adjacent communities and industry trend signals (non-posting candidates)

For each candidate, fan out parallel curls (single message, multiple Bash tool calls) to validate:

```bash
USERNAME=$(jq -r .reddit_username .rising/config.json)
UA="script:com.github.agamjn.rising:v0.1.0 (by /u/${USERNAME})"
SUB=<candidate>

curl -sS --fail-with-body -A "$UA" --connect-timeout 10 --max-time 30 \
  "https://www.reddit.com/r/${SUB}/about.json" \
  -o ".rising/cache/${SUB}-about.json"

curl -sS --fail-with-body -A "$UA" --connect-timeout 10 --max-time 30 \
  "https://www.reddit.com/r/${SUB}/about/rules.json" \
  -o ".rising/cache/${SUB}-rules.json"
```

Filter the candidates:
- **Drop** if the about.json fetch returned 404 or the body indicates the sub doesn't exist
- **Drop** if `subscribers` < 1000
- **Drop** if `subreddit_type` is `private` or `restricted`

For the surviving candidates, build the `validated_candidates` array with: `name`, `subscribers`, `over18`, `subreddit_type`, `public_description`, `description`, `submit_text`, `submission_type`.

# Step 7 — Style analysis + subreddit curation (parallel agents)

Dispatch two agents in parallel — single message, two Agent tool calls:

1. **reddit-style-analyzer** — input: `corpus_source: posts-history.md`, `username: <from config>`. It produces `writing-style.md`.

2. **reddit-subreddit-discoverer** — inputs: contents of `product.md`, contents of `goal.md`, the `validated_candidates` array you built. It produces `subreddits.md`.

# Step 8 — Cache rules into per-sub markdown files

After the discoverer finishes and `subreddits.md` exists, parse it for the chosen **posting** subs. For each posting sub, run:

```bash
mkdir -p "subreddits/${SUB}"
jq -r '.rules[] | "## \(.short_name)\n\n\(.description)\n"' \
  ".rising/cache/${SUB}-rules.json" > "subreddits/${SUB}/rules.md"
```

This writes the rules verbatim from the cached JSON into the per-sub markdown the daily run consumes.

# Step 9 — Performance analysis

Launch the **reddit-performance-analyzer** agent. It reads `posts-history.md` + `product.md`, classifies each post into conversational/soft-promo/promotional, computes per-category stats, and writes `post-categories.md`.

# Step 10 — Wrap up

Summarize what was generated:

```
Setup complete. Files in <CWD>:
  product.md          — your product brief
  goal.md             — what you want from Reddit
  writing-style.md    — your voice, distilled (top traits: ...)
  posts-history.md    — N posts logged
  post-categories.md  — performance breakdown (conversational: X avg ↑, soft-promo: Y, promotional: Z)
  subreddits.md       — N posting subs + M non-posting subs
  subreddits/         — cached rules per posting sub
  .rising/            — config + raw JSON cache (gitignored)

Reminder: you can hide your Reddit profile again now. Use /reddit-add-post <url>
to log future posts so the engine keeps learning.

Next: run /reddit-daily to get your first 3 post proposals.
Optional: run /reddit-schedule "weekdays at 9am" to automate it.
```

# Quality bar

- Every step requires the user's actual input — don't fabricate the product brief, goal, or username.
- **Never use WebFetch for Reddit URLs.** Reddit blocks generic User-Agents; only Bash + curl with the explicit `-A "$UA"` header reliably works.
- If any curl fails (non-zero exit), surface the error to the user with the URL and exit code. Don't silently fabricate content.
- All file writes happen in CWD. Never write outside CWD.
- WebFetch is fine for **non-Reddit** URLs (the product website in Step 1).
