# Placement report

Board: `input.kicad_pcb` (55 footprint blocks, 4-layer, no copper, outline present).
Verdict: **placement already clean at the P0 gate. No optimizer run. Board shipped unchanged.**

## P0 measurements (copper-free board, floor 0.1 mm from board netclass)

| instrument | result |
|---|---|
| `check_drc` (`--clearance 0.1 --clearance-margin 0`) | 0 violations |
| `check_assembly` | `buildable`, blocking 0, 0 pad pairs, 0 hole conflicts, 0 parts with pad copper off-board |
| `render_placement` `checklist.a_off_outline.pad_copper` | empty |
| `board_score` | blocking 39 = unrouted 39, broken 0, drc 0, assembly 0, undersized 0 |

`blocking 39` is the copper-free baseline: every net is unrouted because no routing exists yet. It is not a placement defect. `placement_blocked` is empty (no net is unroutable because of a part pose).

## Reported, not gated

- Courtyard pair J4 <-> J5: 1.181 mm2, depth 0.638 mm, front side. Marked COURTYARD-BLOCKING by census but report-only (no moved-vs-baseline, nothing moved). Bodies do not overlap (`b_body_overlap_pairs` empty); pads clear.
- Courtyard off outline: J2 by 1.07 mm, J4 by 0.12 mm. Pad copper is all inside the outline. Likely edge connectors (J2 has CC1/CC2, USB-C).
- crossings 511, hpwl 762.8 (informational).
- 7 rail-sharing decaps lie beyond the 5 mm search radius (worst 9.2 mm); decap grading skipped for them.
- No design brief, no intent. Floorplan ungraded.

## Decision

Non-negotiable 1: measured clean means STOP. A quench over a placement that passes risks making it worse. If J4/J5 courtyard contact or J2 overhang matters mechanically, declare it in a `.design-brief.json` / intent `overlap_waivers` or fix by hand with `place_pose.py`. Not changed here because no instrument shows it harming routing or buildability.

## Artifacts

- `final.kicad_pcb` + `final.kicad_pro` (+ `.kicad_prl`): byte copy of input via `copy_board.py`.
- `ledger.jsonl`: one entry, stop condition 1.
- `wk/`: drc0.json, assembly0.json, render0.json/png, score0.json, intent0.json, venv.
- Movie: not produced. No lap moved a part, so a film would show one static frame.

## Environment note

System Python lacked numpy/scipy/shapely/Pillow. A venv was created inside `wk/venv` to avoid touching files outside the work dir.
