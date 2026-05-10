---
description: Log one or more Reddit posts the user made by pasting their URLs. Appends to posts-history.md and refreshes post-categories.md so the engine keeps learning what works.
argument-hint: <url> [<url> ...]
---

You log Reddit posts the user made into `posts-history.md` so the performance analyzer and writing-style analyzer can use them.

# Pre-flight check

- Confirm CWD has `posts-history.md` AND `.rising/config.json`. If either is missing, stop: "This folder doesn't look set up. Run `/reddit-setup` first."
- Read `$ARGUMENTS` for the URL(s). If empty, ask: "Paste the post URL(s), one per line:"

Capture the User-Agent for all Reddit calls:

```bash
USERNAME=$(jq -r .reddit_username .rising/config.json)
UA="script:com.github.agamjn.rising:v0.1.0 (by /u/${USERNAME})"
```

(**Never use WebFetch for Reddit URLs.** Reddit rejects requests without a conformant User-Agent; only Bash + curl with `-A "$UA"` works reliably.)

# Procedure

For each URL, in parallel (single message, multiple Bash tool calls):

1. Normalize the URL (strip query strings, fragments, trailing slashes). Validate that the host is `reddit.com` or `redd.it` — skip with a warning otherwise.
2. Derive a stable post id from the URL (the segment after `/comments/`).
3. Curl it:

```bash
curl -sS --fail-with-body -A "$UA" --connect-timeout 10 --max-time 30 \
  "<normalized-url>.json" \
  -o ".rising/cache/post-${POST_ID}.json"
```

After the parallel fetches finish, for each cached file:

4. Parse `[0].data.children[0].data` for: `title`, `selftext`, `subreddit`, `permalink`, `score`, `num_comments`, `created_utc` (convert to ISO date), `link_flair_text`.
5. Parse `[1].data.children` for the top 3 comments by score: `author`, `score`, `body` (truncate at 200 chars).
6. Check whether this URL already exists in `posts-history.md` (search by permalink). If yes: update its score/num_comments in place. If no: insert a new entry at the **top** of the post entries (newest first).

Entry format (same as setup):

```markdown
## <ISO-date> — r/<sub> — "<title>"
- URL: https://reddit.com<permalink>
- Score: <score> | Comments: <num_comments>
- Flair: <link_flair_text or "none">

<selftext or "[link post: <url>]">

**Top comments:**
- (<score>) <username>: <truncated body>
- ...

---
```

# After logging

If at least one post was logged or updated, launch the **reddit-performance-analyzer** agent to refresh `post-categories.md`.

# Wrap up

```
Logged N post(s). Updated M existing.
post-categories.md refreshed.
<If any URLs failed to fetch:>
Skipped (curl failed): <url1>, <url2>
```

# Quality bar

- **Never use WebFetch.** Always Bash + curl with `-A "$UA"`.
- If a fetch fails (404, deleted post, network error), note it in the wrap-up and continue with the rest. Don't bail on the batch.
- Validate URLs are actually Reddit URLs before fetching.
- Never overwrite `posts-history.md` from scratch — it's append-only history. Insert/update individual entries.
