# Introduction {#sec:intro}

Agricultural robots must reach semantically specified goals (e.g., "third palm tree from the left") in dynamic, repetitive fields, yet deploying a navigation stack for a new crop or site still requires weeks to months of data collection, labeling, and manual tuning [@bac2014harvesting]. In our deployments this process takes ${\sim}$`<!-- -->`{=html}3 months. This bottleneck severely limits the scalability of agricultural automation.

Traditional navigation workflows further compound this delay by requiring a full 3D reconstruction or map before waypoint planning. A typical stack collects dense RGB-D data, performs SLAM or multi-view reconstruction, builds a 3D representation, and then tunes cost maps and planners for each site [@campos2021orbslam3; @pfrommer2019tagslam]. In outdoor agriculture, repetitive vegetation, lighting changes, and dynamic obstacles make mapping brittle and increase the time spent on calibration and hand-tuning. These pipelines are also ill-suited to semantic constraints ("avoid the irrigation ditch" or "stop at the third tree"), which require additional hand-coded logic.

Natural language is uniquely suited for commanding agricultural robots because it supports *compositional, constraint-aware* instructions (e.g., "go to the third palm tree from the left while avoiding the water channel"), enables *dynamic re-tasking*, and when paired with open-vocabulary VLMs provides *zero-shot generalization* across crops and sites. Our design exploits two insights from field deployments: (i) start-point conditioning anchors the robot in repetitive rows where SLAM drifts; (ii) BEV projection makes traversable structure explicit, reducing VLM spatial hallucinations.

Existing outdoor agricultural navigation systems are constrained by (i) *limited generalizability* across crops and spatial abstractions, (ii) *excessive site-specific fine-tuning* for new deployments, (iii) *narrow NL command diversity* (typically single-target ROI only), and (iv) *limited edge deployability* of large VLMs. WaypointGen addresses these jointly: Qwen 3.6 MoE [@qwen2026qwen36] generates executable Code-as-Policies [@liang2023code] for 2D spatial reasoning over normalized BEV centroids of open-world ROI masks, enabling phrase grounding for NL-defined abstractions (rows, paths, gaps); a semi-supervised pipeline expands 5 labeled {RGBD, NL command, MPPI trajectory} pairs into 500 training pairs per NL template via in-context learning; users add viewpoint-dependent, temporal, and spatial NL constraints at inference time without retraining; and MoE distillation for Jetson-class edge deployment is targeted as future work. Together this yields inherently generalizable NL-conditioned trajectory selection that reduces deployment to a new agricultural environment from months to 1--2 weeks.

This paper introduces WaypointGen, a system that generates navigation waypoints directly in 2D image space from natural language commands and synchronized RGB-D observations (Sec. [3.1](#sec:problem){reference-type="ref" reference="sec:problem"}). Our key insight is that image-space waypoint generation followed by lightweight 3D projection is more robust than full 3D reconstruction in outdoor agriculture. The main contributions are:

![**Egocentric waypoint generation.** Given the NL command *"Drive to the 3rd tall palm tree,"* WaypointGen identifies the target via SAM3 instance segmentation (blue mask, labeled `det_008_tree`, with crosshair) and generates 7 MPPI candidate trajectories (colored curves) in the egocentric camera frame. The VLM selects the trajectory that best satisfies the NL constraint. This figure depicts the on-robot *execution* layer; the NL$\to$area-equation *planning* layer that produces the target region is what we evaluate quantitatively (Sec. [4](#sec:experiments){reference-type="ref" reference="sec:experiments"}).](figures/fig_hero_egocentric_mppi.pdf){#fig:hero_egocentric width="\\columnwidth"}

**Contributions.** (i) A map-free, NL constraint-aware 2D waypoint generation pipeline using a single-frame egocentric BEV projection with Code-as-Policies [@liang2023code] spatial reasoning, reducing deployment from 3 months to 1--2 weeks. (ii) A semi-supervised dataset pipeline that expands 5 labeled {RGBD, NL, trajectory} pairs per template into 500 training pairs via in-context learning using SAM3, DAM, MPPI, and Qwen3.6-35B-A3B (256 experts, 3B active), with ablations quantifying the trajectory-selection strategy. (iii) Evaluation on 200 GT-annotated commands spanning 10 citrus orchards across four detail levels (D1--D4), achieving 99% structural pass rate (198/200 verified) and 81.9% compositional constraint satisfaction via multi-round VLM inference.

# Related Work {#sec:related}

## Vision-Language Models for Spatial Reasoning

Recent work demonstrates that VLMs benefit from explicit geometric signal injection for spatial tasks. SpatialVLM [@chen2024spatialvlm] introduces a spatial reasoning module that grounds metric distance and directional queries before VLM decoding, enabling quantitative spatial judgments from language. However, SpatialVLM operates on single-image depth estimation and does not model multi-level spatial hierarchies. SpatialStack [@zhang2026spatialstack] extends this line with multi-level geometry-language fusion across scene hierarchies, achieving state-of-the-art 3D spatial reasoning in indoor and outdoor scenes by injecting geometric features at multiple transformer layers. RoboPoint [@yuan2025robopoint] predicts spatial affordance points directly in the camera frame from language descriptions, achieving precise action-point predictions without costmaps but targeting indoor tabletop settings and single-object goals. VGGT [@wang2025vggt] demonstrates feed-forward 3D geometry estimation from multi-view images, providing the geometric backbone that downstream VLM reasoning can exploit for scene understanding. These works demonstrate that VLMs benefit from explicit geometric signal injection, but none address compositional NL constraints across a multi-level spatial hierarchy---the core challenge for agricultural navigation where commands span orchard-level coverage down to individual tree pickup.

## Language-Conditioned Navigation

SayCan [@ichter2023saycan] grounds language commands in robotic affordances via value functions over a fixed skill library, enabling long-horizon task execution but requiring pre-defined primitives that must be manually specified for each environment. NaVILA [@cheng2025navila] drives legged-robot navigation with a hierarchical VLM policy that emits mid-level language actions executed by a low-level controller, rather than explicit metric waypoints. VLM-GroNav [@elnoor2025vlmgronav] grounds outdoor navigation in physical terrain properties and has its VLM *select* waypoints from numbered candidate markers overlaid on aerial imagery to optimize traversability, while BehAV [@weerakoon2025behav] encodes behavioral rules into a cost map and outputs velocity commands; neither conditions waypoints on compositional natural-language spatial constraints. AgriVLN [@zhao2025agrivln] is the closest to our domain, adapting vision-and-language navigation to agriculture, but a VLM selects low-level discrete actions (forward/rotate/stop) frame-by-frame in a continuous environment rather than emitting metric BEV waypoint sequences. CoNVOI [@sathyamoorthy2024convoi] uses zero-shot VLM scene classification and selects numbered free-space markers to construct a metric reference path, but does not generate waypoints from compositional NL constraints. Prior language-conditioned navigation systems either require prior maps or topological graphs from earlier traversal [@huang2023vlmaps; @shah2023lmnav] or per-environment training; WaypointGen removes both requirements through in-context learning over single-frame BEV projections.

## Compositional Spatial Grounding

Compositional language grounding remains challenging because instruction-following accuracy degrades as spatial abstraction increases. The foundational study by Lachmy et al. [@lachmy2022draw] establishes an 8-level NL abstraction taxonomy for spatial instruction following in a drawing domain, demonstrating that seq2seq models collapse on composed objects without explicit algebra emission---an inverse accuracy-abstraction correlation. This finding directly motivates our four-level evaluation hierarchy (D1--D4), which tests whether VLMs exhibit the same degradation when generating navigation waypoints under compositional spatial constraints. ConceptGraphs [@gu2024conceptgraphs] constructs open-vocabulary 3D scene graphs from posed RGB-D streams, enabling language-grounded object retrieval across rooms but lacking trajectory generation or compositional constraint reasoning. MapGPT [@chen2024mapgpt] uses hierarchical map-based prompting with adaptive path planning for vision-and-language navigation, demonstrating that structured spatial representations improve VLM navigation decisions in indoor settings. Goal-conditioned navigation models [@shah2023gnm; @shah2023vint; @sridhar2024nomad; @wang2025genie] achieve impressive zero-shot transfer across environments but are conditioned on goal images or GPS goals rather than compositional language, preventing users from specifying constraints such as ordinal selection or avoidance zones. WaypointGen uniquely combines compositional NL constraints, prior-map-free operation (single-frame BEV only), open-vocabulary detection via SAM3, edge deployment, and outdoor agricultural evaluation across four spatial abstraction levels.

## Navigation in Agricultural Environments

Classical agricultural navigation relies on color thresholding and hard-coded depth rules that require extensive per-site tuning and lack semantic understanding [@bac2014harvesting]. Recent learned approaches---self-supervised traversability maps [@gasparino2024wayfaster] and semantic keypoint tracking (CropFollow++ [@sivakumar2024cropfollowpp])---improve robustness but still use task-agnostic costmaps and cannot incorporate language directives such as ordinality or avoidance constraints. VLM-guided outdoor navigation methods such as BehAV [@weerakoon2025behav] (behavioral rules to velocity commands) and VLM-GroNav [@elnoor2025vlmgronav] (marker-selected waypoints for terrain traversability) integrate behavioral or proprioceptive grounding but do not condition waypoints on compositional NL directives such as ordinality or avoidance constraints. CoNVOI [@sathyamoorthy2024convoi] uses zero-shot VLM classification to build a metric reference path for context-aware navigation. Visual aliasing in crop rows causes SLAM drift [@cuaran2024undercanopy], motivating alternative representations.

## Waypoint Generation and Planning

Conventional waypoint methods rely on manual teaching or map-based planning (SLAM + A\*/RRT) [@campos2021orbslam3; @pfrommer2019tagslam], which require per-site tuning and cannot handle semantic goals. End-to-end learned policies bypass mapping but must be retrained per environment. RoboPoint [@yuan2025robopoint] predicts action points in the camera frame, achieving precise plans without costmaps, but targets indoor settings. IG-PRM [@bao2025igprm] converts language instructions into cost maps for path planning, while GOAT [@chang2024goat] handles multi-modal goal navigation with semantic mapping and VLFM [@yokoyama2024vlfm] enables zero-shot language-conditioned navigation via frontier maps. Our approach generates waypoints in image/BEV space, operating outdoors on edge hardware with a modular pipeline integrating detection, segmentation, and depth projection.

::: table*
:::

Table [\[tab:related_comparison\]](#tab:related_comparison){reference-type="ref" reference="tab:related_comparison"} (with the per-capability matrix, Table [\[tab:capability_matrix\]](#tab:capability_matrix){reference-type="ref" reference="tab:capability_matrix"} in Appendix [7](#app:extra){reference-type="ref" reference="app:extra"}) shows that WaypointGen uniquely combines compositional language understanding, map-free operation, open-vocabulary perception, edge deployment, and outdoor agricultural evaluation: prior map-free systems (LM-Nav, SayCan) rely on topological graphs from prior traversal or fixed skill libraries, action-point methods (RoboPoint, PIVOT) target tabletop or general navigation rather than outdoor agricultural BEV with multi-level NL abstractions, and goal-conditioned models (GNM, ViNT, NoMaD, GeNIE) are conditioned on goal images or GPS goals rather than compositional language. WaypointGen bridges these gaps, synthesizing constraint-aware waypoint sequences directly from language and onboard RGB-D sensing on lightweight edge hardware.

# Technical Approach {#sec:approach}

## Problem Formulation {#sec:problem}

We formalize language-conditioned 2D waypoint generation as follows. A waypoint is an ordered 2D point in the image plane whose 3D projection yields a feasible base pose for the robot. Let $\mathcal{L}$ denote a natural language navigation command (e.g., "go to the third palm tree from the left while avoiding the water channel"), $\{I_k^{\text{RGB}}, I_k^{\text{D}}\}_{k=1}^{K}$ a set of $K$ synchronized RGB-D image pairs from on-board cameras, $\{K_k\}$ the corresponding camera intrinsic matrices, and $\mathcal{E}$ the set of entities in the scene.

**Goal.** Given $(\mathcal{L}, \{I_k^{\text{RGB}}, I_k^{\text{D}}\}, \{K_k\})$, find a 2D waypoint sequence $W = \{(x_i, y_i)\}_{i=1}^{N}$ in image coordinates of a selected camera $k^*$ such that:

1.  $W_N$ lies within the bounding box of the target entity specified by $\mathcal{L}$;

2.  No waypoint $W_i$ lies within the bounding box of any avoidance entity specified by $\mathcal{L}$;

3.  All waypoints lie within regions whose traversability score $\tau \geq \tau_{\min}$.

**Objective.** Minimize the Average Displacement Error (ADE) between predicted waypoints and expert-annotated ground truth $W^*$: $$\text{ADE}(W, W^*) = \frac{1}{N}\sum_{i=1}^{N} \| W_i - W^*_i \|_2$$ while maximizing end-to-end command execution success rate $\mathcal{S}$ (defined in Sec. [4.1](#sec:setup){reference-type="ref" reference="sec:setup"}).

The 2D waypoints are then projected to 3D for execution: $P_{3D} = K_{k^*}^{-1} [x, y, 1]^T D_{k^*}(x,y)$.

## System Architecture Overview

WaypointGen implements a seven-stage pipeline (Fig. [1](#fig:hero_egocentric){reference-type="ref" reference="fig:hero_egocentric"}; additional levels in Appendix [7](#app:extra){reference-type="ref" reference="app:extra"}) that maps a natural language command to executable 2D waypoints: instruction parsing, phrase grounding, open-world segmentation (SAM3 [@carion2025sam3]), dense captioning (DAM), BEV projection, trajectory planning (MPPI [@williams2017mppi]), and VLM-based trajectory selection (Qwen3.6), followed by lightweight 3D projection. The system operates on synchronized RGB-D streams ($640{\times}360$, depth filtered to 15 m) with AprilTag/TagSLAM [@pfrommer2019tagslam] or RTK GPS providing the shared coordinate frame.

## Pipeline Stages

Given a command $\mathcal{L}$, the system executes seven stages: (S1) an LLM parses $\mathcal{L}$ into a structured query $\mathcal{Q} = f_{\text{LLM}}(\mathcal{L})$ containing targets, avoidance constraints, and ordinals; (S2) phrase grounding expands the query into open-vocabulary detection prompts; (S3) SAM3 [@carion2025sam3] performs open-world segmentation producing instance masks $\{M_i, \ell_i\} = f_{\text{SAM3}}(I_{\text{RGB}})$; (S4) DAM generates dense captions per mask encoding identity and spatial context; (S5) a prompt bank of 336 NL commands is constructed and filtered via a three-stage Code-as-Policies pipeline; (S6) MPPI samples 50 unicycle trajectories on the BEV and selects the top-7 by depth-based cost; (S7) Qwen3.6-35B-A3B selects the trajectory best satisfying the NL constraints.

In parallel with S3, the depth map is projected to a $400{\times}400$ px BEV image at $0.05$ m/px resolution, making traversable structure explicit before waypoint synthesis. Start-point conditioning marks the robot's position in the image, providing local context crucial in repetitive crop rows.

::: algorithm
::: algorithmic
**Input:** RGB-D Image $I$, NL Command $C$, Intrinsic $K$ **Output:** Best BEV Trajectory $T^*$ $I_{\text{BEV}} \leftarrow \text{ProjectBEV}(I_{\text{RGB}}, I_{\text{D}}, K)$ $M \leftarrow \text{SAM3}(I_{\text{RGB}})$ $D \leftarrow \text{DAM}(M)$ $\mathcal{R}_{\text{target}} \leftarrow \text{Qwen3.6-35B-A3B}(C, D, M)$ $\text{CostMap} \leftarrow \text{DepthBEVCost}(I_{\text{BEV}}, M)$ $\mathcal{T}_{50} \leftarrow \text{MPPI}(I_{\text{BEV}}, \mathcal{R}_{\text{target}}, \text{CostMap})$ $\mathcal{T}_{7} \leftarrow \text{TopK}(\mathcal{T}_{50},\, K{=}7)$ $T^* \leftarrow \text{Qwen3.6-35B-A3B}(\mathcal{T}_{7}, C, D)$ **return** $T^*$
:::
:::

Generated 2D waypoints are projected to 3D via $P_{3D} = K^{-1} [x, y, 1]^T D(x,y)$, avoiding full 3D reconstruction.

## Semi-Supervised Labeling Pipeline {#sec:autolabel}

Traditional labeling requires 6 weeks of manual effort. Our automated pipeline (Fig. [1](#fig:hero_egocentric){reference-type="ref" reference="fig:hero_egocentric"}; additional levels in Appendix [7](#app:extra){reference-type="ref" reference="app:extra"}) reduces this to 1--2 weeks: RGB-D frames are projected to BEV, SAM3 generates open-world masks with DAM dense captions, a prompt bank of 336 NL commands is constructed and filtered via Code-as-Policies, and cost-aware MPPI selects top-7 trajectories for VLM-based NL-conditioned selection. The resulting *(NL command, trajectory, reasoning)* tuples enable future LoRA finetuning for edge deployment.

# Experimental Results {#sec:experiments}

**Scope of evaluation.** All metrics below characterize the *planning* layer: NL-to-area-equation generation over the L0--L3 hierarchy and the resulting waypoints' region containment. The on-robot execution layer (Alg. [\[alg:traj_gen\]](#alg:traj_gen){reference-type="ref" reference="alg:traj_gen"}, Fig. [1](#fig:hero_egocentric){reference-type="ref" reference="fig:hero_egocentric"}) is described qualitatively; its closed-loop evaluation is left to future work. We evaluate WaypointGen on compositional NL command execution across four spatial abstraction levels, measuring structural correctness and semantic constraint satisfaction on the 10-orchard citrus ground-truth dataset (200 pairs, D1--D4).

## Experimental Setup {#sec:setup}

**Ground-truth dataset.** We evaluate on the 10-orchard citrus dataset, comprising 200 NL navigation command pairs across four detail levels (D1 global, D2 multi-row, D3 row-segment, D4 single-tree). Each pair carries ground-truth area equations with concrete geometric parameters derived from a refined tree-reconstruction pipeline operating on satellite orthomosaic BEV images.

**In-context learning bank.** All evaluations use 6 ground-truth-verified SayPlan exemplars [@rana2023sayplan] covering D1--D4. Each exemplar was verified via leave-one-out multi-round inference.

**Platform.** Qwen3.6-35B-A3B (MoE, 3B active/token) served via ollama on RTX 5090 (32 GB GDDR7). Edge-class validation on NVIDIA Jetson Thor (122 GB unified).

## Multi-Round Inference Results

::: {#tab:multiround}
  Metric                                              Single-round   Multi-round (R1+R2+R3)
  -------------------------------------------------- -------------- ------------------------
  Structural output rate                                 96.6%             **100.0%**
  Structural pass rate                                   75.1%             **99.0%**
  Commands with $\geq$`<!-- -->`{=html}6 waypoints       75.1%             **98.5%**
  R1 direct success                                       ---                76.5%
  R2 targeted retry                                       ---                21.0%
  R3 deterministic fallback                               ---                 2.5%
  Errors                                                   0                   0
  Median time (s/cmd)                                     12.9                28.0

  : Multi-round inference results (200 GT-annotated command pairs, Qwen3.6-35B-A3B, RTX 5090).
:::

Multi-round inference eliminates the 23.5% missing-block failure mode: R2 carries R1's thinking trace and demands the missing blocks, recovering 21.0% of commands. R3 synthesizes waypoints along the area-equation primitive for the remaining 2.5%.

## 4-Level Compositional Constraint Satisfaction

::: {#tab:constraints}
  Level               Constraints   Satisfied      Rate
  ------------------ ------------- ----------- ------------
  D3 (Row-segment)        140          140      **100.0%**
  D4 (Single-tree)        120          114        95.0%
  D2 (Multi-row)          150          112        74.7%
  D1 (Global)             169          108        63.9%
  **Overall**           **579**      **474**    **81.9%**

  : Constraint satisfaction by abstraction level (200 pairs, semantic scoring). An independent semantic LLM-judge re-scoring on all GT constraints yields 422/713 = 59.2% overall (D1 43.8%, D2 55.7%, D3 73.8%, D4 91.1%); see supplementary for methodology comparison.
:::

D3 achieves **100%** (row-half + headland transitions); D4 achieves 95% (instance-level tree grounding). D1 global reasoning (63.9%) is hardest---requires `OrchardWing`/`quadrant` primitives under-represented in the 6-exemplar bank (Fig. [1](#fig:hero_egocentric){reference-type="ref" reference="fig:hero_egocentric"}; additional levels in Appendix [7](#app:extra){reference-type="ref" reference="app:extra"}).

## Reasoning Complexity and Hardware Generalization

Multi-hardware validation confirms architecture-independent generalization across workstation and edge classes: RTX 5090 (28.0 s/cmd median) achieves a 100% structural output rate (99.0% structural pass, 198/200), and NVIDIA Jetson Thor edge hardware (122 GB unified, ${\sim}69\,$s/cmd median) achieves a 99.5% (199/200) structural pass rate via multi-round inference---direct evidence of edge deployability.

## Geometric Fidelity and Ablation Studies

::: {#tab:geometric}
  Level                 $n$     Struct. %    Geo. %     Both %
  ------------------ --------- ----------- ---------- ----------
  D1 (Global)           50        96.0        80.0       78.0
  D2 (Multi-row)        50        100.0       44.0       44.0
  D3 (Row-segment)      50        100.0       28.0       28.0
  D4 (Single-tree)      50        100.0       6.0        6.0
  **Overall**         **200**   **99.0**    **39.5**   **39.0**

  : Geometric fidelity by detail level (200 pairs). Structural pass exceeds 96% at every level but geometric accuracy (waypoints inside GT region) degrades from D1 to D4.
:::

::: {#tab:ablation}
  Condition                                                  $n$     Struct. %
  ------------------------------------------------------- --------- -----------
  Zero-shot (no scaffold, no exemplars)                      400        0.0
  Scaffold only ($k{=}0$, no exemplars)                      200        0.0
  $k{=}6$, different model (Qwen3.5-35B-A3B)                 200        0.5
  **Full system ($k{=}6$, hierarchy, Qwen3.6-35B-A3B)**    **200**   **99.0**

  : Ablation (matched-model, matched-$n{=}200$ on Qwen3.6-35B-A3B): the structured area-equation format does not emerge without the 6-shot scaffold, and the older-model control verifies model dependence. Intermediate-$k$ and no-hierarchy rows at matched $n$ are deferred to the extended version.
:::

Table [3](#tab:geometric){reference-type="ref" reference="tab:geometric"} separates structural correctness from geometric fidelity: finer-grained commands (D4 single-tree: 6% geometric) expose the VLM's lack of metric spatial grounding, motivating the proposed geometry encoder (Sec. [\[sec:geometry_encoder\]](#sec:geometry_encoder){reference-type="ref" reference="sec:geometry_encoder"}). Table [4](#tab:ablation){reference-type="ref" reference="tab:ablation"} isolates the key components at matched $n{=}200$ on Qwen3.6-35B-A3B: zero-shot and scaffold-only ($k{=}0$) both produce 0% structural pass, and an older-model control ($k{=}6$, Qwen3.5-35B-A3B) reaches only 0.5%, so the full 99% headline requires the 6-shot SayPlan scaffold and the Qwen3.6 model jointly. A matched-$n$ intermediate-$k$ and no-hierarchy sweep is deferred to the extended version.

# Discussion and Limitations {#sec:discussion}

**Why 2D BEV + multi-round works.** VLMs are trained on 2D images; image-space reasoning preserves spatial semantics that degrade under noisy 3D projection in agricultural environments (thin vegetation, repetitive rows, dynamic lighting). Our multi-round architecture exploits this: R1 generates area equations and predicates via thinking-mode inference over the BEV scene graph; R2 recovers structurally incomplete outputs by re-prompting with the model's own reasoning trace; R3 provides a deterministic geometric fallback. This three-round design achieves 100% structural output rate even when individual inference calls fail (24.9% of single-round attempts produce missing blocks).

The SayPlan [@rana2023sayplan] scaffold with 6 ground-truth-verified exemplars provides the compositional anchoring: the model learns from real orchard geometry (tree positions, row boundaries, headland extents) rather than synthetic templates, achieving 82% constraint satisfaction across four spatial abstraction levels.

**Limitations.** (1) *Latency*: multi-round inference requires 25--30 s/cmd on RTX 5090; mitigation via LoRA distillation to 7B targeting $<$`<!-- -->`{=html}2 s. (2) *Single-view*: planning from one BEV frame means occluded targets produce unreachable waypoints; multi-frame fusion is future work. (3) *Geometric fidelity*: 100% structural correctness does not guarantee region containment (39.5% overall); we propose a geometry encoder (Sec. [\[sec:geometry_encoder\]](#sec:geometry_encoder){reference-type="ref" reference="sec:geometry_encoder"}) and a training-free inference-time activation-steering complement to address this, both left to future work. (4) *Edge deployment*: requires ${\geq}32$ GB VRAM; 4-bit NF4 quantization targets Jetson-class hardware.

**Future work: a proposed geometry-aware fusion layer.** []{#sec:geometry_encoder label="sec:geometry_encoder"} To close the geometric-fidelity gap, we are developing (but do not evaluate here) a lightweight geometry encoder (${\sim}4$M params) inspired by SpatialStack [@zhang2026spatialstack] that injects spatial structure into the language backbone before its LoRA-adapted layers by fusing BEV patch embeddings, SAM3 mask centroids, and scene-graph node coordinates (with L0--L3 level embeddings) via geometry-biased cross-attention. Whether such an encoder, trained on the semi-supervised triples, measurably improves region containment is left to an extension of this work, alongside the inference-time steering complement noted above.

**Generative AI disclosure.** An AI coding assistant was used for experimental orchestration, script generation, and editorial assistance. All scientific claims, experimental results, and architectural decisions are authored and verified by the human researchers. VLM inference for waypoint generation uses Qwen3.6-35B-A3B (Alibaba/Qwen team) served via ollama.

**Reproducibility.** The full code, the semi-supervised dataset (200 GT-annotated command pairs plus the in-context distillation set), and an interactive project page will be released publicly upon acceptance; an anonymized repository and data archive will be provided to reviewers during the discussion period upon request.

# Conclusion {#sec:conclusions}

We presented WaypointGen, a language-conditioned system that compiles compositional NL navigation commands into executable area equations over a four-level spatial hierarchy (L0--L3) using multi-round VLM inference. Across 200 evaluation pairs spanning 10 citrus orchards (D1--D4), it raises the structural output rate from 75.1% (single-round) to 100% (99.0% structural pass) and attains 81.9% compositional constraint satisfaction, with the R1 thinking $\to$ R2 retry $\to$ R3 fallback design eliminating structural failures across RTX 5090 and NVIDIA Jetson Thor edge hardware (99.5%). This reduces deployment from months to 1--2 weeks using only 6 ground-truth-verified in-context examples. Future work targets the geometry encoder for metric fidelity and LoRA distillation for edge latency.

# Per-Capability Comparison and Qualitative Examples {#app:extra}

Table [\[tab:capability_matrix\]](#tab:capability_matrix){reference-type="ref" reference="tab:capability_matrix"} gives the per-capability breakdown that complements the summary comparison (Table [\[tab:related_comparison\]](#tab:related_comparison){reference-type="ref" reference="tab:related_comparison"}) in the main text, and Figure [\[fig:citrus_bev_traj\]](#fig:citrus_bev_traj){reference-type="ref" reference="fig:citrus_bev_traj"} shows one qualitative example per abstraction level (D1--D4).

::: table*
**Columns:** *Compos. NL* = compositional language with ordinals, avoidance, spatial relations; *Map-Free* = no prior map or exploration; *Open-Vocab* = open-vocabulary object detection; *Edge Deploy* = runs on Jetson-class hardware; *Outdoor Ag.* = tested in agricultural outdoor environments; *Waypoints* = outputs explicit waypoint sequences; *Constraints* = handles NL-defined avoidance constraints.
:::

::: figure*
![D1: *"Cover the south-east quadrant of the orchard while staying inside the access-lane network and avoiding row 1."* 3 simultaneous constraints: quadrant + lane + avoidance.](figures/selected/D1_SE_quadrant_coverage.png){width="\\textwidth"}

![D2: *"Survey only the odd-numbered rows from north to south, finishing at the southern headland."* Boustrophedon pattern through row subset.](figures/selected/D2_odd_rows_survey.png){width="\\textwidth"}

![D3: *"Continue along row 5 (skip its westernmost tree, which is mis-labelled) until you reach the eastern headland."* Inline avoidance constraint.](figures/selected/D3_skip_mislabelled_tree.png){width="\\textwidth"}

![D4: *"Goal: reach the third tree counted from the east edge of row 11, where a missing-tree slot was flagged earlier today."* Ordinal + contextual grounding.](figures/selected/D4_single_tree_row11.png){width="\\textwidth"}
:::
