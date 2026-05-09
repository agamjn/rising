# rising

A personal Reddit content engine for founders, marketers, and devrel folks
who want to do Reddit *right* — not by spamming product blurbs, but by
researching what their target subs actually talk about and proposing posts
that fit the community.

The name comes from Reddit's "rising" sort — the posts on their way up. That's
what this plugin helps you write.

Reddit punishes self-promotion. So this plugin doesn't post for you. It does
the research that good Reddit content requires (writing style, sub trends,
performance history, sub rules) and proposes 3 posts a day that *you* then
review, edit, and post manually.

---

## What it does

**One-time setup** (`/reddit-setup`):
1. Asks for your product brief, Reddit username, and goal
2. Pulls your past Reddit posts and analyzes your writing style
3. Discovers ~10 subreddits split into "posting" (where to post) and
   "non-posting" (research only — competitors / trend signal)
4. Caches each posting sub's rules
5. Categorizes your past posts (conversational / soft-promo / promotional)
   and computes per-category engagement stats so the engine knows what's
   worked for you

**Daily run** (`/reddit-daily`):
1. Asks if you posted anything new (paste the URL — it appends to your
   history and refreshes performance analysis)
2. Refreshes engagement on your recent posts
3. Dispatches one research agent **in parallel** for every configured
   subreddit — each pulls top-of-week + hot, summarizes themes
4. Synthesizes 3 post proposals — title, body, target sub, category, and
   reasoning that cites which trend it's responding to and why this format
   has worked for you before
5. Saves to `drafts/YYYY-MM-DD.md`

**Other commands:**
- `/reddit-add-post <url>` — log a post you made (appends to history,
  refreshes performance)
- `/reddit-schedule "weekdays at 9am"` — automate the daily run

---

## Install

```
/plugin marketplace add agamjn/rising
/plugin install rising
```

---

## Quickstart (5 minutes)

```bash
# Make a folder for your product
mkdir -p ~/reddit/my-product
cd ~/reddit/my-product

# Set up
/reddit-setup
# (answers product URL, Reddit username, and goal interactively)

# Get your first batch of proposals
/reddit-daily

# (open drafts/<today>.md, pick a proposal, post on Reddit)
# (afterward, log it back so the engine learns)
/reddit-add-post https://reddit.com/r/<sub>/comments/<id>/<slug>/

# Optional: automate daily
/reddit-schedule "weekdays at 9am"
```

Want to see what files this produces? See [`examples/tensorfuse/`](examples/tensorfuse/)
for a fully-populated example product folder, including a real-looking
`drafts/2026-05-09.md`.

---

## ⚠️ Reddit profile visibility caveat

The setup pulls your posts via Reddit's public JSON API, which **only works
on public profiles**. Most users hide their profile by default.

**The recommended flow:**
1. Before running `/reddit-setup`, go to https://www.reddit.com/settings →
   Profile → toggle ON "Show active communities" and "Make my profile
   visible to search engines"
2. Run `/reddit-setup` — it pulls your full history and saves it to
   `posts-history.md` in your project folder
3. Hide your profile again (toggle OFF)
4. From then on, log new posts manually with `/reddit-add-post <url>` —
   the engine grows your history without your profile being public

Your `posts-history.md` is the source of truth after setup. Reddit's API
is only used for the initial pull and for fetching individual post URLs
you provide.

---

## Architecture (for the curious)

- **4 slash commands** as entry points
- **5 sub-agents** — most heavy lifting happens here, dispatched in parallel
  for fan-out work like per-subreddit research
- **1 skill** (`reddit-writing-style`) — reusable style-extraction workflow
- **No MCP server** — uses Claude Code's built-in `WebFetch` against Reddit's
  public `*.json` endpoints. No OAuth, no install steps, no auth tokens
- **Per-project data** — everything lives in your CWD. Want a separate
  product? Make a separate folder

The plugin doesn't post to Reddit. Ever. There is no Reddit write API
integration and no way to add one without modifying the plugin's source.

---

## Why no auto-posting?

Three reasons:

1. **Reddit ToS.** Automated posting via the API requires OAuth + a
   registered Reddit app, and even then the rules around automation are
   strict. Most accounts that try this get shadowbanned within weeks.
2. **Quality control.** The whole point of doing Reddit *right* is human
   judgment — picking the moment, tweaking the tone, responding to
   comments. Auto-posting throws away the part that makes the strategy
   work.
3. **Account safety.** A founder's Reddit account is a long-term asset.
   Letting an LLM post unsupervised is a fast path to a permaban for
   spam.

The engine does the 90% of work that's tedious (research, drafting). You
do the 10% that has to be human (review, post, engage with comments).

---

## Configuration files (in your CWD after setup)

| File | Purpose |
|---|---|
| `product.md` | Your product brief |
| `goal.md` | What you want from Reddit |
| `writing-style.md` | Your voice, distilled from past posts |
| `posts-history.md` | Append-only log of every post you've made |
| `post-categories.md` | Classification + per-category performance stats |
| `subreddits.md` | Posting subs + non-posting subs, with rationale |
| `subreddits/<sub>/rules.md` | Cached rules per posting sub |
| `drafts/YYYY-MM-DD.md` | Daily output — 3 proposals |

Edit any of these freely between runs. The engine uses whatever it finds.

---

## License

MIT. See [LICENSE](LICENSE).

---

## Contributing

Issues and PRs welcome. The plugin is opinionated — see the agent and
command files for the design choices. The goal is to keep it *simple*:
no MCP server, no auth, no DB, just markdown files and the public Reddit
API.
