---
name: reddit-writing-style
description: Use when analyzing a corpus of a user's Reddit posts to extract their writing voice, tone, structural patterns, and vocabulary into a reusable style guide. Invoked during /reddit-setup and re-invokable standalone if the user wants to refresh their style file after writing more posts.
---

# Reddit Writing Style Analysis

Extract a user's authentic Reddit voice from their post history so future generated content matches how they actually write.

## Inputs

A corpus of posts. Each post has at minimum: `title`, `body` (selftext), `subreddit`, `score`, `num_comments`. Posts may come from `posts-history.md` in the current working directory (the canonical local source) or be passed inline.

## What to extract

Produce a `writing-style.md` file with these sections, each backed by direct quotes/examples from the corpus (cite the post by title, never paraphrase the example itself):

1. **Voice and tone** — formal vs casual, dry vs warm, confident vs hedged. Quote 2-3 lines that exemplify it.
2. **Sentence rhythm** — average sentence length, use of fragments, comma-heavy or short-staccato. Note any signature mannerisms (em-dashes, parentheticals, ALL CAPS for emphasis, etc.).
3. **Opening patterns** — how the user typically starts a post. (e.g., "I always open with a one-line setup question", or "I lead with a strong claim then justify").
4. **Closing patterns** — how the user wraps up. (CTA? Open question? Just stops?).
5. **Vocabulary signature** — words/phrases the user reaches for repeatedly. List 8-15 specific examples actually pulled from the corpus.
6. **Words/phrases to avoid** — any LLM-tells the user clearly never uses ("delve", "in conclusion", "it's important to note", em-dash overuse, etc.) — base on what's *absent* from their corpus, not generic advice.
7. **Structural patterns** — paragraph length, use of bullet lists vs prose, headings, line breaks, links, code blocks. Note differences across categories if visible.
8. **Subreddit-specific tone shifts** — does the user write differently in r/programming vs r/AskReddit? Capture if so.

## How to extract

- Read every post in the corpus before generalizing. Do not summarize from the first 5 and call it done.
- For each section, your claim must be defended by 2+ concrete examples from the corpus. If you cannot find 2 examples, omit the claim.
- Prefer specificity over generality. "Uses lowercase even at sentence start in casual subs (see post 'why your monorepo sucks')" beats "informal tone".
- Avoid generic style-guide platitudes ("clear and concise"). Every point must be something a different writer might *not* do.

## Output format

Write directly to `writing-style.md` in CWD. Markdown, no frontmatter, top-level title `# Writing Style — <username>`. Each section as `## ...`. Examples as blockquotes with the source post title in italics underneath.

## When the corpus is too small

If the corpus has <5 posts, output a `writing-style.md` that explicitly says so at the top:
> **Caveat:** Style guide derived from only N posts; treat patterns as tentative and refresh after the user has more history.

Then do the best analysis you can on what's there, but keep claims hedged.
