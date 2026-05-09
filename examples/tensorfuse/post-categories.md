# Post Categories & Performance — u/agamjain_tf

_Last updated: 2026-05-09_
_Based on 47 posts._

## Summary table

| Category | Count | Avg upvotes | Median upvotes | Avg comments |
|---|---|---|---|---|
| Conversational | 31 | 198 | 156 | 42 |
| Soft-promo | 12 | 87 | 64 | 18 |
| Promotional | 4 | 24 | 19 | 6 |

**Headline:** Conversational outperforms promotional ~8x on upvotes and ~7x on
comments. Soft-promo lands in the middle. The synthesizer should default to
conversational unless there's a strong specific reason to go promotional.

## Conversational

**What's worked:**
- Specific technical war stories with numbers ("90s → 12s cold start"). The
  3 highest-performing posts all open this way.
- Posts in r/devops outperform same-content posts in r/programming 2:1 (more
  targeted audience, less generic-tech crowd).
- Asking specific operational questions ("What's your team using for GPU
  observability?") consistently lands 50+ comments — it's a way to spark
  discussion AND learn from the audience.

**What hasn't:**
- Generic "thoughts on X?" posts without concrete context.
- Posts that try to cover too much ground in one post — the meta-essay
  "[D] Why are we still building bespoke inference servers" did decently
  on comments (89) but lower on upvotes (156) than focused posts.

**Top 3 posts:**
1. r/devops — "Scaling GPU pods to zero — what actually worked" — 412↑ / 67 comments
2. r/programming — "What 'serverless' means once you have GPU workloads" — 412↑ / 78 comments
3. r/LocalLLaMA — "Notes on cold-start latency for self-hosted LLMs" — 287↑ / 41 comments

**Bottom 3:**
1. r/programming — "Some thoughts on inference architectures" — 23↑ / 4 comments
2. r/devops — "Devops corner: GPU edition" — 19↑ / 3 comments
3. r/programming — "ML infra musings" — 11↑ / 1 comment

## Soft-promo

**What's worked:**
- Comparison/benchmark posts where Tensorfuse is one option among 3-4 others
  ("Comparing fine-tune frameworks" was actually borderline soft-promo since
  we mentioned our deployment story). Reads as fair, not boosterism.
- Posts that share a *concrete operational pattern* we use, mentioning the
  product as the implementation rather than the subject.

**What hasn't:**
- Posts where the product mention feels grafted-on ("...and by the way we
  built a tool for this").

**Top 3 posts:**
1. r/MachineLearning — "[P] Comparing fine-tune frameworks for Llama-3" — 234↑ / 38 comments
2. r/MachineLearning — "[D] Why are we still building bespoke inference servers" — 156↑ / 89 comments
3. r/devops — "How we cut our GPU bill 60% with smarter autoscaling" — 134↑ / 22 comments

**Bottom 3:**
1. r/mlops — "What we learned shipping Tensorfuse to GA" — 41↑ / 8 comments
2. r/LocalLLaMA — "Tensorfuse + vLLM benchmarks" — 38↑ / 12 comments (numbers OK, framing felt vendor-y)
3. r/programming — "Building a serverless GPU platform" — 22↑ / 3 comments

## Promotional

_Only 4 posts. Treat as low-confidence._

**What's worked:**
- Nothing has worked well in this category. Best performer was 38↑.

**What hasn't:**
- Launch posts in product-tolerant subs still get downvoted to 25-30↑.
- Direct comparison posts where Tensorfuse is the headliner read as ads.

**All 4 posts:**
1. r/LocalLLaMA — "We launched Tensorfuse — serverless GPU for self-hosted LLMs" — 38↑ / 9 comments
2. r/devops — "Tensorfuse: deploy your model in one command" — 26↑ / 4 comments
3. r/MachineLearning — "[N] Tensorfuse 1.0 release" — 19↑ / 6 comments (likely auto-removed for "launch" keyword based on rules)
4. r/programming — "Show: Tensorfuse" — 12↑ / 4 comments

## Cross-category observations

- Posts on Tuesday/Wednesday/Thursday between 9am and noon ET outperform
  weekend posts roughly 3:1 on upvotes. Schedule daily for weekday mornings.
- Long-form (>500 words) outperforms short-form in r/MachineLearning and
  r/devops, but underperforms in r/programming. Match length to sub.
- Posts with specific numbers in the title ("412 RPS", "90s → 12s", "60%
  cheaper") get ~2x the click-through of posts with abstract titles.
- Account-level pattern: u/agamjain_tf has built credibility as a "war
  stories from the trenches" voice. Generic-take posts don't fit and
  underperform. Lean into the voice.
