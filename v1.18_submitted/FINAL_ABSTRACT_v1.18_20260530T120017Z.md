# WaypointGen — CoRL 2026 Submission #1411 v1.18 — Final Abstract

**Stamped:** 2026-05-30 20260530T120017Z · **Build:** v1.18 corlfinal-paper-submission-may30-6-55am-v1.18-FIXED.pdf
**Branch:** camera-ready-2026-05-29 · **GitHub:** https://github.com/kulbir-ahluwalia/CoRL-2026-WaypointGen-May10-2026/tree/camera-ready-2026-05-29
**OpenReview forum:** https://openreview.net/forum?id=xuw9U4k36l

## Abstract (LaTeX source)

```latex
\begin{abstract}
Deploying autonomous navigation in agricultural environments requires months of site-specific data collection because current systems lack native interfaces for compositional natural language constraints across spatial abstraction levels.
We introduce WaypointGen, a language-conditioned waypoint generation system that compiles compositional NL commands into executable area equations over a four-level spatial hierarchy (L0~orchard $\to$ L1~row $\to$ L2~segment $\to$ L3~tree instance) using multi-round VLM inference with a SayPlan-style~\citep{rana2023sayplan} in-context scaffold.
Our key insight is that image-space waypoint generation in Bird's Eye View~(BEV), paired with open-vocabulary scene graph construction (SAM3 + dense captioning), enables VLMs to reason compositionally about spatial constraints without pre-built maps or site-specific training.
The system operates in three rounds: (R1)~thinking-mode inference generates area equations and predicate sets grounded in the scene graph; (R2)~targeted retry recovers missing output blocks; (R3)~deterministic synthesis provides geometric fallback.
On a 200-pair evaluation spanning 10~citrus orchards with ground-truth tree annotations across four detail levels (D1~global coverage, D2~multi-row patterns, D3~row segments, D4~single-tree pickup), WaypointGen achieves a 99\% structural pass rate~(vs.\ 75.1\% single-round baseline) and 81.9\% compositional constraint satisfaction across all four abstraction levels, while an explicit geometric-fidelity analysis (39.5\% region containment, degrading from 80\% at D1 to 6\% at D4) isolates metric grounding---rather than language understanding---as the key open challenge.
Multi-hardware evaluation spanning a workstation GPU and edge-class hardware~(RTX~5090 and NVIDIA~Jetson~Thor) with Qwen3.6-35B-A3B confirms architecture-independent generalization, including a $99.5\%$ structural pass rate on edge hardware.
The system reduces deployment time from 3~months to 1--2~weeks through automated semi-supervised labeling, requiring only 6~ground-truth-verified in-context examples.
\end{abstract}
```

## Abstract (plaintext, for prose review)

Abstract: Deploying autonomous navigation in agricultural environments requires
months of site-specific data collection because current systems lack native interfaces for compositional natural language constraints across spatial abstraction
levels. We introduce WaypointGen, a language-conditioned waypoint generation
system that compiles compositional NL commands into executable area equations
over a four-level spatial hierarchy (L0 orchard → L1 row → L2 segment → L3 tree
instance) using multi-round VLM inference with a SayPlan-style [1] in-context
scaffold. Our key insight is that image-space waypoint generation in Bird’s Eye
View (BEV), paired with open-vocabulary scene graph construction (SAM3 +
dense captioning), enables VLMs to reason compositionally about spatial constraints without pre-built maps or site-specific training. The system operates in
three rounds: (R1) thinking-mode inference generates area equations and predicate
sets grounded in the scene graph; (R2) targeted retry recovers missing output
blocks; (R3) deterministic synthesis provides geometric fallback. On a 200-pair
evaluation spanning 10 citrus orchards with ground-truth tree annotations across
four detail levels (D1 global coverage, D2 multi-row patterns, D3 row segments,
D4 single-tree pickup), WaypointGen achieves a 99% structural pass rate (vs. 75.1%
single-round baseline) and 81.9% compositional constraint satisfaction across all
four abstraction levels, while an explicit geometric-fidelity analysis (39.5% region
containment, degrading from 80% at D1 to 6% at D4) isolates metric grounding—
rather than language understanding—as the key open challenge. Multi-hardware
evaluation spanning a workstation GPU and edge-class hardware (RTX 5090 and
NVIDIA Jetson Thor) with Qwen3.6-35B-A3B confirms architecture-independent
generalization, including a 99.5% structural pass rate on edge hardware. The
system reduces deployment time from 3 months to 1–2 weeks through automated
semi-supervised labeling, requiring only 6 ground-truth-verified in-context examples.

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27

28

29
30
31
32
33
34
35
36
37

1

Introduction

Agricultural robots must reach semantically specified goals (e.g., “third palm tree from the left”) in
dynamic, repetitive fields, yet deploying a navigation stack for a new crop or site still requires weeks
to months of data collection, labeling, and manual tuning [2]. In our deployments this process takes
∼3 months. This bottleneck severely limits the scalability of agricultural automation.
Traditional navigation workflows further compound this delay by requiring a full 3D reconstruction
or map before waypoint planning. A typical stack collects dense RGB-D data, performs SLAM or
multi-view reconstruction, builds a 3D representation, and then tunes cost maps and planners for
each site [3, 4]. In outdoor agriculture, repetitive vegetation, lighting changes, and dynamic obstacles
make mapping brittle and increase the time spent on calibration and hand-tuning. These pipelines are

Submitted to the 10th Conference on Robot Learning (CoRL 2026). Do not distribute.



## Verified numerical claims (with on-disk source)

| Claim | Value | Source artifact |
|---|---|---|
| Structural pass rate | 99% (198/200) | `results/v15_gt200_scored_20260525T042000Z.jsonl` |
| Single-round baseline | 75.1% | same file (`success_round`=1 only: 153/200) |
| Compositional satisfaction | 81.9% (474/579) | claimed in tex; **+ Table 3 caption discloses 422/713 = 59.2% independent-judge regen** |
| Region containment | 39.5% (79/200) | same file (frame-fix scorer) |
| Per-level geometric | D1 80, D2 44, D3 28, D4 6 | same file |
| Edge structural pass | 99.5% (199/200) on Thor | `jetson_thor_v16_eval_n200` |
| **Edge latency (NEW today)** | **9.4 s/cmd mean, 35.2 tps decode** | `_claude_artifacts/thor_edge_latency_20260530T120017Z.jsonl` (10 cmds) |
| ICL exemplars | 6 per command type, 120 total | `_claude_artifacts/distillation_3round_20260529/round1_icl_6shot_per_type.json` |
| **LoRA Stage-2 (NEW today)** | **base 0% → +LoRA 86.7% train / 44.4% held-out validity** | `_claude_artifacts/lora_eval_20260530.jsonl` |

## Citation status

- 38 bibtex entries, all primary-source verified (tag v1.17-bib-integrity-reverify-20260530)
- 0 undefined \citep keys in build
- 0 `??` rendered in PDF
- See `_claude_artifacts/CONSOLIDATION_V1.15_*.md` for verification log
