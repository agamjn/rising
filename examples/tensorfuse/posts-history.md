# Posts History — u/agamjain_tf

_Append-only log. Newest first. Updated by /reddit-setup, /reddit-add-post, and engagement refresh in /reddit-daily._

## 2026-05-06 — r/devops — "Scaling GPU pods to zero — what actually worked"
- URL: https://reddit.com/r/devops/comments/abc123/scaling_gpu_pods_to_zero/
- Score: 412 | Comments: 67
- Flair: Discussion

We tried KEDA for autoscaling first and it took us three weeks to admit it
was the wrong abstraction. Sharing what we ended up doing instead.

The actual problem with KEDA + GPU nodes is that node provisioning happens
out-of-band from pod scheduling. Karpenter helps but adds its own surface
area...

(full post body — 540 words about Knative + custom controller approach)

**Top comments:**
- (87) u/sre_grumpy: This matches our experience. Curious what your p99 cold start ended up at?
- (54) u/k8s_dad: Have you tried virtual-kubelet? Different approach but same goal...
- (41) u/ml_infra: Saved. We're about to start this exact migration.

---

## 2026-05-02 — r/LocalLLaMA — "Notes on cold-start latency for self-hosted LLMs"
- URL: https://reddit.com/r/LocalLLaMA/comments/def456/cold_start_notes/
- Score: 287 | Comments: 41
- Flair: Discussion

Honestly the cold start problem isn't solved. We got it from 90s to 12s
and that's still 12s longer than anyone wants.

Quick notes on what moved the needle for Llama-3-8B on A10G:

1. Tensor parallelism eats startup time. For 8B, single GPU + bigger batch
   beats 2x GPU with TP every time below ~50 RPS...

(full body — 380 words)

**Top comments:**
- (52) u/quant_bro: 12s on 8B is impressive actually. What's your loader?
- (38) u/llm_chad: We use vLLM and see ~25s for similar setup. Curious about your stack.
- (29) u/inference_eng: Have you looked at lazyllm-style on-demand layer loading?

---

## 2026-04-28 — r/MachineLearning — "[D] Why are we still building bespoke inference servers in 2026"
- URL: https://reddit.com/r/MachineLearning/comments/ghi789/bespoke_inference/
- Score: 156 | Comments: 89
- Flair: Discussion

Maybe a hot take but: every infra team I talk to is building roughly the
same vLLM-or-TGI + Kubernetes + autoscaling stack. We're all rediscovering
the same gotchas.

What's the moat that's preventing a real shared abstraction? My theory:
the assumption that "production inference" means tightly-coupled to
business logic, when actually 80% of teams just want a deploy command...

(full body — 620 words)

**Top comments:**
- (44) u/researcher_x: I disagree — the customization IS the value...
- (31) u/mlops_consult: You're describing what BentoML thought it was building 4 years ago.
- (28) u/infra_dad: This but applied to embeddings too.

---

## 2026-04-22 — r/devops — "What's your team using for GPU observability in 2026?"
- URL: https://reddit.com/r/devops/comments/jkl012/gpu_observability/
- Score: 98 | Comments: 54
- Flair: Question

Genuinely asking. We've been on a homegrown nvidia-smi + Prometheus exporter
setup for a year and it's getting hairy at our scale (~80 GPUs across 4
clusters). DCGM Exporter looks promising but the cardinality story scares me.

What are folks using?

**Top comments:**
- (33) u/observability_pro: NVIDIA NIM for monitoring even if not for inference...
- (22) u/prometheus_guy: DCGM works but yeah cardinality. We pre-aggregate at the node level.
- (19) u/ops_lead: Datadog GPU monitoring is fine if you're already paying.

---

## 2026-04-15 — r/MachineLearning — "[P] Comparing fine-tune frameworks for Llama-3 — Axolotl vs LLaMA-Factory vs raw HF"
- URL: https://reddit.com/r/MachineLearning/comments/mno345/finetune_frameworks/
- Score: 234 | Comments: 38
- Flair: Project

Spent the last month comparing three approaches for fine-tuning Llama-3-8B
on a domain-specific corpus. Here's what we found:

(table comparing memory, throughput, ease-of-use)

**Top comments:**
- (41) u/ax_user: Helpful. The Axolotl docs really do hide the good stuff.
- (28) u/finetune_fan: What batch size at what GPU?

---

## 2026-04-08 — r/kubernetes — "RBAC for GPU node pools — sharing what worked"
- URL: https://reddit.com/r/kubernetes/comments/pqr678/gpu_rbac/
- Score: 67 | Comments: 19
- Flair: Discussion

Short post. We had multiple teams sharing GPU nodes and ran into the
classic noisy neighbor + unfair scheduling. Here's the RBAC + ResourceQuota
combo that ended up working...

(short body — 240 words)

**Top comments:**
- (15) u/k8s_jedi: Why not just use namespace-level quotas alone?
- (11) u/sre_pro: Saving for later, thanks.

---

## 2026-03-30 — r/programming — "What 'serverless' means once you have GPU workloads"
- URL: https://reddit.com/r/programming/comments/stu901/serverless_gpu/
- Score: 412 | Comments: 78
- Flair: none

(broader-audience post — 450 words walking through why GPU "serverless" is a
different beast than CPU serverless, and what abstractions break down)

**Top comments:**
- (62) u/grumpy_dev: The "scale to zero" assumption falls apart with 30s cold starts.
- (44) u/cloud_skeptic: Good post but you didn't mention that the economics are completely different too.
- (29) u/lambda_user: Curious where this leaves the FaaS-for-ML category.

---

_(40 more entries truncated for brevity in this example file — your real `posts-history.md` will contain everything)_
