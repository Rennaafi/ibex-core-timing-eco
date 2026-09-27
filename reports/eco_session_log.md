# Interactive ECO Session — Closing ibex_core's Setup Violation

This documents an interactive timing-closure exercise performed on the finished
`ibex_core` route (baseline: `clk_period = 2.2 ns`, WNS -15.5 ps / -0.02 ns,
240 setup violations, 0 hold violations — see `baseline_6_finish_curated.rpt`).
Done inside OpenROAD's interactive GUI/Tcl console via
`make DESIGN_CONFIG=./designs/nangate45/ibex/config.mk gui_final`.

## 1. Diagnosing the critical path

```tcl
report_checks -path_delay max -to {gen_regfile_ff.register_file_i.rf_reg[948]$_DFFE_PN0P_/D} -fields {slew cap input}
```

Worst path: `if_stage_i.instr_rdata_id_o[15]` → ~30 logic gates → register file
write port `rf_reg[948]/D`. Slack: **-0.02 ns (VIOLATED)**.

Reading the Cap/Slew columns against sibling cells of the same type revealed
one clearly undersized driver:

```
Cap    Slew   Delay   Cell
2.90   0.01   0.06    _17633_/Z (MUX2_X1)
32.48  0.04   0.10    _17634_/Z (MUX2_X1)   <- ~11x the load of its neighbor, still smallest drive strength
3.34   0.01   0.07    _18610_/Z (MUX2_X1)   <- inherits the degraded edge
```

## 2. Manual fix (cell resize ECO)

```tcl
replace_cell _17634_ MUX2_X2
```

Result: slack improved **-0.02 → -0.01 ns**, but the critical path moved to a
sibling startpoint (`instr_rdata_id_o[20]`) converging on the same shared
OR-tree tail (`_19479_` → `_21568_`). Checked whether that shared tail had
more sizing headroom:

```bash
grep -A2 "cell (OR4" .../NangateOpenCellLibrary_typical.lib | grep "cell ("
# OR4_X1, OR4_X2, OR4_X4 only — already at X4, no bigger variant exists
```

Conclusion: manual cell-by-cell resizing had hit its structural ceiling.

## 3. Automated ECO — `repair_timing`

```tcl
repair_timing -setup
```

```
[INFO RSZ-0094] Found 240 endpoints with setup violations.
[INFO RSZ-0099] Repairing 240 out of 240 (100.00%) violating endpoints...
   Iter | Resized | Pin Swaps |   WNS   |  Viol Endpts
------------------------------------------------------
      0 |       0 |         0 |  -0.015 |      240
    250 |       8 |         2 |   0.005 |      240
  final |       8 |         2 |   0.005 |        0
[INFO RSZ-0051] Resized 8 instances: 8 up, 0 up match, 0 down, 0 VT
[INFO RSZ-0043] Swapped pins on 2 instances.
```

**Result: all 240 setup violations closed. WNS -15.5 ps → +5 ps.**

## 4. Attempted full physical closure (legalize + reroute)

```tcl
remove_fillers
detailed_placement
check_placement -verbose      # silent = legal, 0 violations
```

```
Placement Analysis
total displacement    5.7 u
max displacement      3.9 u
delta HPWL             0 %
```

Legalization succeeded cleanly. Re-routing did not:

```tcl
global_route
detailed_route
```

```
[ERROR DRT-0206] checkConnectivity error.
Error: checkConnectivity break, net _10529_ ... pin not visited
Error: checkConnectivity break, net _04573_ ... pin not visited
Error: checkConnectivity break, net ex_block_i.alu_i.adder_in_b[24] ... pin not visited
```

**Diagnosis:** a from-scratch, whole-chip `global_route`/`detailed_route` run
interactively after legalization is not equivalent to a proper *incremental*
ECO reroute — it produced genuinely disconnected nets (a real chip-breaking
result), not just DRC noise. Real ECO flows use incremental/localized
rerouting specifically to avoid this. Session was deliberately stopped here
rather than push further into router internals; the baseline `.odb`/`.spef`
on disk was never overwritten with this broken state.

## Honest scope statement

The **STA-level ECO (steps 1–3) is fully verified and reproducible.** Step 4
(full physical re-route + re-extraction for a true signoff-clean GDS) was
attempted, hit a real and instructive limitation, and is left as documented
future work — see README for the specific next step (`detailed_route
-incremental`, or driving the reroute through ORFS's own route-stage script
rather than bare interactive Tcl calls).
