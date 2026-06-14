# S.1 Independent Compositional Re-Scoring {#s.1-independent-compositional-re-scoring .unnumbered}

Independent semantic LLM-judge re-score on all GT constraints: 422/713 = 59.2% overall (D1 43.8%, D2 55.7%, D3 73.8%, D4 91.1%). Different constraint enumeration than main Table 3 (713 vs 579); both methodologies reported for transparency.

# S.2 Stage-2 LoRA Adapter {#s.2-stage-2-lora-adapter .unnumbered}

Base Qwen2.5-VL-7B = 0% area-equation validity (train + held-out, n=15+18); + Stage-2 LoRA = 86.7% train / 44.4% held-out / 63.6% overall. Validates the learned-component design. Caveats: validity = grammar well-formedness (not metric grounding); judge accepts LoRA-invented predicate variants; weakest on $\geq$`<!-- -->`{=html}3 simultaneous constraints.

# S.3 Edge Latency on NVIDIA Jetson Thor {#s.3-edge-latency-on-nvidia-jetson-thor .unnumbered}

n=10 cmds spanning L0-L3 on Qwen3.6-35B-A3B: mean 9.4 s/cmd (med 8.0), decode 35.2 tps, output 256 tokens avg, CV $<$`<!-- -->`{=html}0.05 (decode-bound on 122GB unified-memory edge). The full per-command breakdown is given in Table [1](#tab:thor){reference-type="ref" reference="tab:thor"}.

::: {#tab:thor}
       \#      command                                     wall (s)   tokens    decode (tok/s)
  ------------ ------------------------------------------ ---------- --------- ----------------
       0       Drive down row 3 corridor                    22.15       256          35.2
       1       Traverse all even rows from north end         7.98       256          35.3
       2       Stop at the third tree from the gate          8.05       256          35.2
       3       Avoid the missing tree at row 2 column 4      7.98       256          35.3
       4       Go to the southwest quadrant                  8.04       256          35.2
       5       Enter the headland and circle clockwise       8.10       256          35.3
       6       Pick up at tree (3, 5)                        8.04       256          35.2
       7       Skip every other row starting from row 1      8.04       256          35.2
       8       Stay 2m from the trunk on the east side       8.03       256          35.2
       9       Inspect the gap between rows 4 and 5          8.03       256          35.2
    **mean**                                               **9.44**   **256**      **35.2**
   **median**                                              **8.04**     --         **35.2**

  : Per-command edge inference latency on the NVIDIA Jetson Thor (122 GB unified memory, Qwen3.6-35B-A3B), $n=10$ commands spanning detail levels L0--L3. Wall = end-to-end wall-clock; tokens = output tokens; decode = decode throughput.
:::

# S.4 Integrity Reconciliation {#s.4-integrity-reconciliation .unnumbered}

Every headline metric was independently re-derived from the on-disk scored evaluation log (`v15_gt200_scored`, $n=200$ pairs). Table [2](#tab:integrity){reference-type="ref" reference="tab:integrity"} lists the authoritative values; the per-pair recompute in Table [3](#tab:perpair){reference-type="ref" reference="tab:perpair"} reproduces them exactly. Where two enumeration methodologies exist (compositional satisfaction), both are reported (main paper vs. independent re-judge in S.1) for full transparency.

::: {#tab:integrity}
  metric                                                 authoritative value
  --------------------------------------------------- --------------------------
  Structural output rate                                   200/200 = 100.0%
  Structural pass rate                                     198/200 = 99.0%
  $\geq$`<!-- -->`{=html}6 waypoints                       197/200 = 98.5%
  Geometric pass (overall)                                  79/200 = 39.5%
  Geometric pass D1 / D2 / D3 / D4                       80% / 44% / 28% / 6%
  Structural pass D1 / D2 / D3 / D4                    96% / 100% / 100% / 100%
  Round-1 success (single-round baseline)                  153/200 = 76.5%
  Success-round distribution {1,2,3}                         153 / 42 / 5
  RTX 5090 latency (median / mean)                        28.0 / 28.8 s/cmd
  Jetson Thor edge structural                              199/200 = 99.5%
  Compositional satisfaction (independent re-judge)        422/713 = 59.2%

  : Integrity reconciliation: every headline metric re-derived from the on-disk scored evaluation log (`v15_gt200_scored`, $n=200$). Per-pair recompute (Table [3](#tab:perpair){reference-type="ref" reference="tab:perpair"}) agrees with these authoritative values.
:::

# S.5 Per-Pair Evaluation Table {#s.5-per-pair-evaluation-table .unnumbered}

Table [3](#tab:perpair){reference-type="ref" reference="tab:perpair"} reports every one of the 200 ground-truth NL$\to$waypoint pairs (20 command categories $\times$ 4 detail levels D1--D4), grouped by detail level with a per-level subtotal. Structural pass measures whether a well-formed area-equation and waypoint sequence were produced; geometric pass measures whether the generated waypoints fall inside the target region. The structural--geometric gap (99.0% vs. 39.5%) and its sharp dependence on target-region size (D1 80% $\to$ D4 6%) motivate the learned geometry encoder discussed in the main paper.

::: center
::: {#tab:perpair}
  image_id                   D    category                             SR     Struct      $n_{wp}$   $f_{in}$      Geo
  -------------------------- ---- ----------------------------------- ---- ------------- ---------- ---------- ------------
  Table  -- continued                                                                                          
  image_id                   D    category                             SR     Struct      $n_{wp}$   $f_{in}$      Geo
  *continued on next page*                                                                                     
  citrus_bev_sample_02       D1   centre_then_quadrant                 1                     6         0.50    
  citrus_bev_sample_03       D1   centre_then_quadrant                 1                     6         0.71    
  citrus_bev_sample_04       D1   centre_then_quadrant                 1                     6         0.17      $\times$
  citrus_bev_sample_05       D1   centre_then_quadrant                 1                     6         0.00      $\times$
  citrus_bev_sample_06       D1   centre_then_quadrant                 3                     6         0.50    
  citrus_bev_sample_07       D1   centre_then_quadrant                 1                     6         1.00    
  citrus_bev_sample_08       D1   centre_then_quadrant                 1                     6         0.00      $\times$
  citrus_bev_sample_09       D1   centre_then_quadrant                 2                     6         0.33      $\times$
  citrus_bev_sample_10       D1   centre_then_quadrant                 1                     6         0.17      $\times$
  citrus_bev_sample_11       D1   centre_then_quadrant                 2                     6         0.33      $\times$
  citrus_bev_sample_02       D1   multi_constraint_avoidance           1                     6         1.00    
  citrus_bev_sample_03       D1   multi_constraint_avoidance           1                     6         1.00    
  citrus_bev_sample_04       D1   multi_constraint_avoidance           1                     6         1.00    
  citrus_bev_sample_05       D1   multi_constraint_avoidance           2                     6         1.00    
  citrus_bev_sample_06       D1   multi_constraint_avoidance           2                     6         1.00    
  citrus_bev_sample_07       D1   multi_constraint_avoidance           1                     6         1.00    
  citrus_bev_sample_08       D1   multi_constraint_avoidance           1                     6         1.00    
  citrus_bev_sample_09       D1   multi_constraint_avoidance           1                     6         1.00    
  citrus_bev_sample_10       D1   multi_constraint_avoidance           1                     6         0.67    
  citrus_bev_sample_11       D1   multi_constraint_avoidance           1                     6         1.00    
  citrus_bev_sample_02       D1   northern_half_then_southern_half     1                     6         0.78    
  citrus_bev_sample_03       D1   northern_half_then_southern_half     2                     6         1.00    
  citrus_bev_sample_04       D1   northern_half_then_southern_half     1                     6         0.67    
  citrus_bev_sample_05       D1   northern_half_then_southern_half     1                     6         1.00    
  citrus_bev_sample_06       D1   northern_half_then_southern_half     1                     6         0.69    
  citrus_bev_sample_07       D1   northern_half_then_southern_half     1                     6         1.00    
  citrus_bev_sample_08       D1   northern_half_then_southern_half     2                     6         1.00    
  citrus_bev_sample_09       D1   northern_half_then_southern_half     2                     6         1.00    
  citrus_bev_sample_10       D1   northern_half_then_southern_half     1                     6         1.00    
  citrus_bev_sample_11       D1   northern_half_then_southern_half     3     $\times$        6         0.00      $\times$
  citrus_bev_sample_02       D1   perimeter_loop                       1                     6         1.00    
  citrus_bev_sample_03       D1   perimeter_loop                       1                     6         1.00    
  citrus_bev_sample_04       D1   perimeter_loop                       2                     6         1.00    
  citrus_bev_sample_05       D1   perimeter_loop                       1                     6         1.00    
  citrus_bev_sample_06       D1   perimeter_loop                       2                     6         1.00    
  citrus_bev_sample_07       D1   perimeter_loop                       2                     6         1.00    
  citrus_bev_sample_08       D1   perimeter_loop                       2                     6         1.00    
  citrus_bev_sample_09       D1   perimeter_loop                       1                     6         1.00    
  citrus_bev_sample_10       D1   perimeter_loop                       1                     6         1.00    
  citrus_bev_sample_11       D1   perimeter_loop                       2                     6         1.00    
  citrus_bev_sample_02       D1   quadrant_coverage                    1                     6         1.00    
  citrus_bev_sample_03       D1   quadrant_coverage                    1                     6         1.00    
  citrus_bev_sample_04       D1   quadrant_coverage                    2                     6         0.33      $\times$
  citrus_bev_sample_05       D1   quadrant_coverage                    1                     6         0.20      $\times$
  citrus_bev_sample_06       D1   quadrant_coverage                    2                     6         1.00    
  citrus_bev_sample_07       D1   quadrant_coverage                    1                     6         1.00    
  citrus_bev_sample_08       D1   quadrant_coverage                    1     $\times$        6         0.50    
  citrus_bev_sample_09       D1   quadrant_coverage                    2                     6         1.00    
  citrus_bev_sample_10       D1   quadrant_coverage                    1                     6         0.33      $\times$
  citrus_bev_sample_11       D1   quadrant_coverage                    2                     6         0.50    
  **D1 subtotal** ($n=50$)                                                   **48/50**                          **40/50**
  citrus_bev_sample_02       D2   alternating_rows_zig_zag             2                     6         0.33      $\times$
  citrus_bev_sample_03       D2   alternating_rows_zig_zag             1                     6         0.33      $\times$
  citrus_bev_sample_04       D2   alternating_rows_zig_zag             1                     6         0.00      $\times$
  citrus_bev_sample_05       D2   alternating_rows_zig_zag             1                     6         0.00      $\times$
  citrus_bev_sample_06       D2   alternating_rows_zig_zag             1                     6         1.00    
  citrus_bev_sample_07       D2   alternating_rows_zig_zag             1                     6         0.33      $\times$
  citrus_bev_sample_08       D2   alternating_rows_zig_zag             1                     4         0.00      $\times$
  citrus_bev_sample_09       D2   alternating_rows_zig_zag             1                     6         0.00      $\times$
  citrus_bev_sample_10       D2   alternating_rows_zig_zag             1                     6         0.67    
  citrus_bev_sample_11       D2   alternating_rows_zig_zag             1                     6         0.00      $\times$
  citrus_bev_sample_02       D2   every_other_tree_multi_row           1                     6         0.67    
  citrus_bev_sample_03       D2   every_other_tree_multi_row           2                     6         0.67    
  citrus_bev_sample_04       D2   every_other_tree_multi_row           2                     6         0.67    
  citrus_bev_sample_05       D2   every_other_tree_multi_row           2                     6         1.00    
  citrus_bev_sample_06       D2   every_other_tree_multi_row           1                     6         0.00      $\times$
  citrus_bev_sample_07       D2   every_other_tree_multi_row           2                     6         0.33      $\times$
  citrus_bev_sample_08       D2   every_other_tree_multi_row           1                     6         0.00      $\times$
  citrus_bev_sample_09       D2   every_other_tree_multi_row           1                     6         0.20      $\times$
  citrus_bev_sample_10       D2   every_other_tree_multi_row           3                     6         1.00    
  citrus_bev_sample_11       D2   every_other_tree_multi_row           1                     6         0.00      $\times$
  citrus_bev_sample_02       D2   headland_strip_sweep                 1                     6         0.00      $\times$
  citrus_bev_sample_03       D2   headland_strip_sweep                 1                     6         0.00      $\times$
  citrus_bev_sample_04       D2   headland_strip_sweep                 3                     6         0.00      $\times$
  citrus_bev_sample_05       D2   headland_strip_sweep                 1                     6         0.00      $\times$
  citrus_bev_sample_06       D2   headland_strip_sweep                 2                     6         0.00      $\times$
  citrus_bev_sample_07       D2   headland_strip_sweep                 1                     6         0.00      $\times$
  citrus_bev_sample_08       D2   headland_strip_sweep                 1                     6         0.00      $\times$
  citrus_bev_sample_09       D2   headland_strip_sweep                 1                     6         0.00      $\times$
  citrus_bev_sample_10       D2   headland_strip_sweep                 1                     6         0.00      $\times$
  citrus_bev_sample_11       D2   headland_strip_sweep                 2                     6         0.00      $\times$
  citrus_bev_sample_02       D2   odd_rows_only                        1                     6         1.00    
  citrus_bev_sample_03       D2   odd_rows_only                        1                     6         0.50    
  citrus_bev_sample_04       D2   odd_rows_only                        1                     6         0.75    
  citrus_bev_sample_05       D2   odd_rows_only                        1                     6         0.44      $\times$
  citrus_bev_sample_06       D2   odd_rows_only                        1                     6         0.75    
  citrus_bev_sample_07       D2   odd_rows_only                        1                     6         0.75    
  citrus_bev_sample_08       D2   odd_rows_only                        2                     6         0.33      $\times$
  citrus_bev_sample_09       D2   odd_rows_only                        1                     6         1.00    
  citrus_bev_sample_10       D2   odd_rows_only                        2                     6         1.00    
  citrus_bev_sample_11       D2   odd_rows_only                        2                     6         1.00    
  citrus_bev_sample_02       D2   row_pair_with_corridor_transition    1                     6         1.00    
  citrus_bev_sample_03       D2   row_pair_with_corridor_transition    1                     6         1.00    
  citrus_bev_sample_04       D2   row_pair_with_corridor_transition    1                     4         1.00    
  citrus_bev_sample_05       D2   row_pair_with_corridor_transition    1                     6         1.00    
  citrus_bev_sample_06       D2   row_pair_with_corridor_transition    1                     6         0.00      $\times$
  citrus_bev_sample_07       D2   row_pair_with_corridor_transition    1                     6         0.67    
  citrus_bev_sample_08       D2   row_pair_with_corridor_transition    1                     6         0.00      $\times$
  citrus_bev_sample_09       D2   row_pair_with_corridor_transition    1                     6         0.50    
  citrus_bev_sample_10       D2   row_pair_with_corridor_transition    1                     6         1.00    
  citrus_bev_sample_11       D2   row_pair_with_corridor_transition    1                     6         0.00      $\times$
  **D2 subtotal** ($n=50$)                                                   **50/50**                          **22/50**
  citrus_bev_sample_02       D3   headland_then_row_half               1                     6         0.33      $\times$
  citrus_bev_sample_03       D3   headland_then_row_half               2                     6         0.50    
  citrus_bev_sample_04       D3   headland_then_row_half               1                     6         0.33      $\times$
  citrus_bev_sample_05       D3   headland_then_row_half               1                     6         0.33      $\times$
  citrus_bev_sample_06       D3   headland_then_row_half               1                     6         0.00      $\times$
  citrus_bev_sample_07       D3   headland_then_row_half               2                     6         0.33      $\times$
  citrus_bev_sample_08       D3   headland_then_row_half               2                     6         0.33      $\times$
  citrus_bev_sample_09       D3   headland_then_row_half               2                     6         0.33      $\times$
  citrus_bev_sample_10       D3   headland_then_row_half               2                     6         0.50    
  citrus_bev_sample_11       D3   headland_then_row_half               1                     6         0.17      $\times$
  citrus_bev_sample_02       D3   lane_then_row                        1                     6         0.17      $\times$
  citrus_bev_sample_03       D3   lane_then_row                        2                     6         1.00    
  citrus_bev_sample_04       D3   lane_then_row                        1                     6         0.17      $\times$
  citrus_bev_sample_05       D3   lane_then_row                        1                     6         0.00      $\times$
  citrus_bev_sample_06       D3   lane_then_row                        1                     6         0.17      $\times$
  citrus_bev_sample_07       D3   lane_then_row                        1                     6         0.17      $\times$
  citrus_bev_sample_08       D3   lane_then_row                        1                     6         0.17      $\times$
  citrus_bev_sample_09       D3   lane_then_row                        1                     6         0.17      $\times$
  citrus_bev_sample_10       D3   lane_then_row                        1                     6         0.33      $\times$
  citrus_bev_sample_11       D3   lane_then_row                        1                     6         0.38      $\times$
  citrus_bev_sample_02       D3   row_central_segment                  1                     6         0.83    
  citrus_bev_sample_03       D3   row_central_segment                  1                     6         0.00      $\times$
  citrus_bev_sample_04       D3   row_central_segment                  1                     6         1.00    
  citrus_bev_sample_05       D3   row_central_segment                  1                     6         1.00    
  citrus_bev_sample_06       D3   row_central_segment                  2                     6         0.00      $\times$
  citrus_bev_sample_07       D3   row_central_segment                  1                     5         1.00    
  citrus_bev_sample_08       D3   row_central_segment                  1                     6         1.00    
  citrus_bev_sample_09       D3   row_central_segment                  1                     6         0.00      $\times$
  citrus_bev_sample_10       D3   row_central_segment                  1                     6         0.50    
  citrus_bev_sample_11       D3   row_central_segment                  1                     6         1.00    
  citrus_bev_sample_02       D3   row_then_headland                    1                     6         0.11      $\times$
  citrus_bev_sample_03       D3   row_then_headland                    1                     6         0.83    
  citrus_bev_sample_04       D3   row_then_headland                    1                     6         0.00      $\times$
  citrus_bev_sample_05       D3   row_then_headland                    1                     6         0.00      $\times$
  citrus_bev_sample_06       D3   row_then_headland                    1                     6         0.83    
  citrus_bev_sample_07       D3   row_then_headland                    1                     6         0.33      $\times$
  citrus_bev_sample_08       D3   row_then_headland                    1                     6         0.17      $\times$
  citrus_bev_sample_09       D3   row_then_headland                    1                     6         0.17      $\times$
  citrus_bev_sample_10       D3   row_then_headland                    1                     6         0.33      $\times$
  citrus_bev_sample_11       D3   row_then_headland                    1                     6         0.50    
  citrus_bev_sample_02       D3   row_with_inline_avoid                1                     6         0.17      $\times$
  citrus_bev_sample_03       D3   row_with_inline_avoid                1                     6         0.33      $\times$
  citrus_bev_sample_04       D3   row_with_inline_avoid                1                     6         0.00      $\times$
  citrus_bev_sample_05       D3   row_with_inline_avoid                1                     6         0.67    
  citrus_bev_sample_06       D3   row_with_inline_avoid                2                     6         0.00      $\times$
  citrus_bev_sample_07       D3   row_with_inline_avoid                1                     6         0.00      $\times$
  citrus_bev_sample_08       D3   row_with_inline_avoid                1                     6         0.33      $\times$
  citrus_bev_sample_09       D3   row_with_inline_avoid                1                     6         0.33      $\times$
  citrus_bev_sample_10       D3   row_with_inline_avoid                2                     6         0.00      $\times$
  citrus_bev_sample_11       D3   row_with_inline_avoid                1                     6         0.33      $\times$
  **D3 subtotal** ($n=50$)                                                   **50/50**                          **14/50**
  citrus_bev_sample_02       D4   single_tree_inspection               2                     6         0.00      $\times$
  citrus_bev_sample_03       D4   single_tree_inspection               1                     6         0.00      $\times$
  citrus_bev_sample_04       D4   single_tree_inspection               1                     6         0.00      $\times$
  citrus_bev_sample_05       D4   single_tree_inspection               1                     6         0.00      $\times$
  citrus_bev_sample_06       D4   single_tree_inspection               1                     6         0.00      $\times$
  citrus_bev_sample_07       D4   single_tree_inspection               1                     6         0.00      $\times$
  citrus_bev_sample_08       D4   single_tree_inspection               1                     6         0.00      $\times$
  citrus_bev_sample_09       D4   single_tree_inspection               1                     6         0.00      $\times$
  citrus_bev_sample_10       D4   single_tree_inspection               1                     6         0.00      $\times$
  citrus_bev_sample_11       D4   single_tree_inspection               1                     6         0.00      $\times$
  citrus_bev_sample_02       D4   single_tree_landmark_skip            2                     6         0.00      $\times$
  citrus_bev_sample_03       D4   single_tree_landmark_skip            1                     6         0.00      $\times$
  citrus_bev_sample_04       D4   single_tree_landmark_skip            1                     6         0.17      $\times$
  citrus_bev_sample_05       D4   single_tree_landmark_skip            1                     6         0.00      $\times$
  citrus_bev_sample_06       D4   single_tree_landmark_skip            1                     6         0.00      $\times$
  citrus_bev_sample_07       D4   single_tree_landmark_skip            2                     6         0.00      $\times$
  citrus_bev_sample_08       D4   single_tree_landmark_skip            1                     6         0.33      $\times$
  citrus_bev_sample_09       D4   single_tree_landmark_skip            1                     6         0.17      $\times$
  citrus_bev_sample_10       D4   single_tree_landmark_skip            1                     6         0.00      $\times$
  citrus_bev_sample_11       D4   single_tree_landmark_skip            1                     6         0.00      $\times$
  citrus_bev_sample_02       D4   single_tree_pickup                   2                     6         0.00      $\times$
  citrus_bev_sample_03       D4   single_tree_pickup                   1                     6         0.00      $\times$
  citrus_bev_sample_04       D4   single_tree_pickup                   1                     6         0.00      $\times$
  citrus_bev_sample_05       D4   single_tree_pickup                   1                     6         0.00      $\times$
  citrus_bev_sample_06       D4   single_tree_pickup                   1                     6         0.00      $\times$
  citrus_bev_sample_07       D4   single_tree_pickup                   1                     6         0.00      $\times$
  citrus_bev_sample_08       D4   single_tree_pickup                   2                     6         0.00      $\times$
  citrus_bev_sample_09       D4   single_tree_pickup                   1                     6         0.00      $\times$
  citrus_bev_sample_10       D4   single_tree_pickup                   1                     6         0.00      $\times$
  citrus_bev_sample_11       D4   single_tree_pickup                   2                     6         0.00      $\times$
  citrus_bev_sample_02       D4   single_tree_replanting               1                     6         0.00      $\times$
  citrus_bev_sample_03       D4   single_tree_replanting               1                     6         0.00      $\times$
  citrus_bev_sample_04       D4   single_tree_replanting               1                     6         0.00      $\times$
  citrus_bev_sample_05       D4   single_tree_replanting               1                     6         0.00      $\times$
  citrus_bev_sample_06       D4   single_tree_replanting               1                     6         0.00      $\times$
  citrus_bev_sample_07       D4   single_tree_replanting               1                     6         0.00      $\times$
  citrus_bev_sample_08       D4   single_tree_replanting               1                     6         0.00      $\times$
  citrus_bev_sample_09       D4   single_tree_replanting               2                     6         0.00      $\times$
  citrus_bev_sample_10       D4   single_tree_replanting               1                     6         0.17      $\times$
  citrus_bev_sample_11       D4   single_tree_replanting               1                     6         0.00      $\times$
  citrus_bev_sample_02       D4   single_tree_via_corridor             1                     6         0.50    
  citrus_bev_sample_03       D4   single_tree_via_corridor             1                     6         0.17      $\times$
  citrus_bev_sample_04       D4   single_tree_via_corridor             1                     6         0.00      $\times$
  citrus_bev_sample_05       D4   single_tree_via_corridor             1                     6         0.50    
  citrus_bev_sample_06       D4   single_tree_via_corridor             3                     6         0.00      $\times$
  citrus_bev_sample_07       D4   single_tree_via_corridor             1                     6         0.50    
  citrus_bev_sample_08       D4   single_tree_via_corridor             1                     6         0.33      $\times$
  citrus_bev_sample_09       D4   single_tree_via_corridor             1                     6         0.17      $\times$
  citrus_bev_sample_10       D4   single_tree_via_corridor             1                     6         0.09      $\times$
  citrus_bev_sample_11       D4   single_tree_via_corridor             1                     6         0.17      $\times$
  **D4 subtotal** ($n=50$)                                                   **50/50**                           **3/50**
  **Overall** ($n=200$)                                                     **198/200**                         **79/200**

  : Per-pair evaluation on the 200 ground-truth NL$\to$waypoint pairs (20 command categories $\times$ 4 detail levels D1--D4), grouped by detail level. SR = success round (multi-round inference, 1--3); Struct = structural pass; $n_{wp}$ = generated waypoints; $f_{in}$ = fraction of waypoints inside the target region; Geo = geometric pass.
:::
:::

# S.6 Reproducibility {#s.6-reproducibility .unnumbered}

Code + 200-pair eval set + 6-shot ICL bank + raw inference logs released on acceptance. Hardware: RTX 5090, NVIDIA GB10, NVIDIA Jetson Thor.
