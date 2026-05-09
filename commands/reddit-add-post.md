---
description: Log one or more Reddit posts the user made by pasting their URLs. Appends to posts-history.md and refreshes post-categories.md so the engine keeps learning what works.
argument-hint: <url> [<url> ...]
---

You log Reddit posts the user made into `posts-history.md` so the performance analyzer and writing-style analyzer can use them.

# Pre-flight check

- Confirm CWD has `posts-history.md`. If missing, stop: "This folder doesn't look set up. Run `/reddit-setup` first."
- Read `$ARGUMENTS` for the URL(s). If empty, ask: "Paste the post URL(s), one per line:"

# Procedure

For each URL:

1. Normalize the URL (strip query strings, fragments, trailing slashes).
2. Fetch `<url>.json` with WebFetch (User-Agent `rising/0.1`).
3. The response is an array — `[0]` is the post listing, `[1]` is the comments listing. From `[0].data.children[0].data` extract:
   - `title`, `selftext`, `subreddit`, `permalink`, `score`, `num_comments`, `created_utc` (convert to ISO date), `link_flair_text`
4. From `[1].data.children` extract the top 3 comments by score: `author`, `score`, `body` (truncate at 200 chars).
5. Check whether this URL already exists in `posts-history.md` (search by permalink). If yes: update its score/num_comments in place. If no: append a new entry at the **top** of the post entries (newest first).

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
```

# Quality bar

- Validate that URLs are actually Reddit URLs (`reddit.com` or `redd.it` host). If not, skip with a warning.
- If a fetch fails (404, deleted post, network error), note it in the wrap-up and continue with the rest. Don't bail on the batch.
- Never overwrite `posts-history.md` from scratch — it's append-only history. Insert/update individual entries.
