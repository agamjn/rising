# Writing Style — u/agamjain_tf

_Derived from 47 posts across 12 subreddits. Last refreshed 2026-05-09._

## Voice and tone

Direct, technical, slightly self-deprecating. Speaks as a peer engineer, not a
marketer. Comfortable saying "I don't know" or "we got this wrong initially."

> "We tried KEDA for autoscaling first and it took us three weeks to admit it
> was the wrong abstraction. Sharing what we ended up doing instead."
> — _r/devops, "Scaling GPU pods to zero — what actually worked"_

> "Honestly the cold start problem isn't solved. We got it from 90s to 12s
> and that's still 12s longer than anyone wants."
> — _r/MachineLearning, "Notes on cold-start latency for self-hosted LLMs"_

## Sentence rhythm

Mix of medium sentences (15-25 words) and short punchy ones for emphasis. Uses
em-dashes sparingly — only when actually needed for asides, not as a tic.
Occasional one-line paragraphs to land a point.

## Opening patterns

Two dominant patterns:
1. **Lead with the specific result, then explain.** "Got cold start from 90s
   to 12s. Here's what we changed." (Used in 18 of 47 posts.)
2. **Open with the failed approach.** "We tried X. It didn't work. Here's why
   we ended up with Y." (Used in 11 posts; consistently the highest-engagement
   opener.)

Almost never opens with a question. When questions are used, they come after
2-3 lines of setup.

## Closing patterns

Closes with either:
- A specific question to the community ("Anyone running this at higher scale
  than 50 RPS? Curious if the patterns hold."), or
- A blunt summary of the takeaway, no CTA. Never closes with "thoughts?" or
  "let me know what you think" — finds these performative.

## Vocabulary signature

Words/phrases that show up repeatedly:
- "honestly" (used to soften critiques)
- "the actual problem"
- "we got this wrong"
- "in production"
- "at our scale" (uses this to be specific about claims)
- "tradeoff"
- "abstraction"
- "lift"
- "hairy" (for complex problems)
- "didn't pan out"
- "ended up with"
- "in our case"

## Words/phrases to avoid

These NEVER appear in the corpus and would read as inauthentic:
- "delve", "dive deep", "unpack"
- "in conclusion", "to summarize"
- "leverage" (uses "use")
- "robust", "seamless", "cutting-edge"
- "it's important to note"
- Em-dash used as a stylistic flourish rather than for grammatical aside
- Exclamation marks (zero in 47 posts)
- "🚀" or any emoji (zero in 47 posts)

## Structural patterns

- **Length:** technical posts run 300-700 words. Discussion posts 100-250.
- **Formatting:** uses code blocks for actual code (never decorative). Uses
  numbered lists for sequenced reasoning. Avoids bullet lists for prose —
  prefers paragraphs.
- **Headings:** uses ## headings only in posts >500 words. Otherwise just
  paragraph breaks.
- **Links:** sparingly, and never to own product unless directly relevant.

## Subreddit-specific shifts

- **r/MachineLearning:** more formal, more citations, fewer asides.
- **r/devops, r/kubernetes:** more conversational, more "we tried X" stories.
- **r/LocalLLaMA:** assumes deep context, jumps straight into specifics.
- **r/programming:** broadens the framing to be accessible to non-ML engineers.
