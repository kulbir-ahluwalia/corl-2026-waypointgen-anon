# Evaluation Benchmark Details

The benchmark comprises **200 ground-truth-annotated command pairs** over 10 citrus orchards, organized as **20 command types $\times$ 4 detail levels** (D1 global coverage, D2 multi-row patterns, D3 row segments, D4 single-tree pickup), 10 pairs per type. Command types span single-tree (pickup, inspection, replanting, via-corridor, landmark-skip), row-segment (row-then-headland, headland-then-row-half, row-central-segment, lane-then-row, row-with-inline-avoid), multi-row (every-other-tree, alternating-rows-zigzag, odd-rows-only, row-pair-corridor-transition, headland-strip-sweep), and global (quadrant-coverage, perimeter-loop, centre-then-quadrant, northern-half-then-southern-half, multi-constraint-avoidance).

# Area-Equation Primitives

Commands compile to compositional area equations over a four-level hierarchy (L0 orchard $\to$ L1 row $\to$ L2 segment $\to$ L3 tree). Primitives: `Row(n)`, `TreePoint(x,y,r)`, `Headland(name)`, `Quadrant(dir)`, `Corridor(width_m)`, `OrchardWing`, combined with the `&` (intersection) operator and predicates (avoidance, ordering, containment, proximity). Example (D4): `Row(3).corridor(width_m=2.0) & TreePoint(tree_3_6.x, tree_3_6.y, r=1.0)`.

# Multi-Round Inference Protocol

R1: thinking-mode inference emits the area equation + predicate set grounded in the BEV scene graph (`num_predict`=2048). R2: if structurally incomplete, re-prompt carrying R1's reasoning trace, demanding the missing blocks. R3: deterministic geometric synthesis along the area-equation primitive as fallback. This eliminates the 24.9% missing-block failure mode of single-round inference.

# Per-Level Results

  Level                 $n$     Struct. %    Geo. %     Both %
  ------------------ --------- ----------- ---------- ----------
  D1 (Global)           50        96.0        80.0       78.0
  D2 (Multi-row)        50        100.0       44.0       44.0
  D3 (Row-segment)      50        100.0       28.0       28.0
  D4 (Single-tree)      50        100.0       6.0        6.0
  **Overall**         **200**   **99.0**    **39.5**   **39.0**

# 3-Stage Semi-Supervised Distillation

Stage 1: 6 human-verified in-context exemplars per command type seed generation of 200 candidate trajectories. Stage 2: human-in-the-loop filtering retains the top-quality subset as weak-supervision signal. Stage 3: the in-context pool is expanded with verified successes and generation is scaled toward a fine-tuning corpus for a compact edge student. Round-2 generation produced 391 valid area-equation samples across the 20 types on workstation- and edge-class hardware.

# Hardware

Validated on three platforms: RTX 5090 (32 GB, 25.5 s/cmd), NVIDIA GB10 unified memory (128 GB, 290 s/cmd), and NVIDIA Jetson Thor (122 GB unified, $\sim$`<!-- -->`{=html}66 s/cmd, 99.5% structural) --- evidence of architecture-independent, edge-deployable inference.

# Limitations of Supplementary Scope

All evaluation is open-loop (generated area equations scored against ground-truth regions); closed-loop on-robot execution and a learned geometry encoder for metric fidelity are future work.
