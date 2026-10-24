# led_toggle_using_clock_final_project_vlsi_traning
video link:
https://drive.google.com/file/d/1VZYWMEbZ5kQ1SHntgWdYNatYdDmwr_lX/view?usp=drivesdk

# LED Blinker: Complete RTL-to-GDSII Flow

### Clock-driven LED toggle design on SkyWater sky130hd using open-source tools

| | |
|---|---|
| **Name** |P.HARSHA VARDHAN REDDY|
| **Technology** | SkyWater sky130, `sky130_fd_sc_hd` |
| **Top module** | `led_blinker` |
| **Flow** | RTL, Simulation, Synthesis, Floorplan, Power Planning, Placement, CTS, Routing, Parasitic Extraction, STA, Power, DRC, LVS, GDSII |
| **Final status** | GDSII generated, DRC clean, LVS match |

</div>

---

## Table of contents

1. Introduction
2. Design specification
3. Tools and technology
4. RTL design
5. Functional simulation
6. Logic synthesis
7. Floorplanning
8. Power planning
9. Placement
10. Clock tree synthesis
11. Routing
12. Parasitic extraction
13. Static timing analysis
14. Power analysis
15. GDSII generation
16. Physical verification: DRC
17. Physical verification: LVS
18. Results summary
19. Problems faced and solutions
20. Repository layout
21. How to reproduce
22. Conclusion and future work

---

## 1. Introduction

This project takes a small sequential design, `led_blinker`, through the complete digital physical-design flow, from RTL to a verified GDSII layout, using open-source tools and the SkyWater 130 nm process. All stages were completed and the final layout passed DRC and LVS.

## 2. Design specification

| Item | Value |
|---|---|
| Top module | `led_blinker` |
| Inputs | `clk` (clock), `rst` (reset) |
| Output | `led` |
| Internal state | 2-bit counter `count[1:0]`, 3 flip-flops in the synthesized netlist |
| Function | A clock-driven counter toggles the LED output |
| Clock period / frequency | FILL |
| Supply voltage | FILL (sky130hd nominal 1.8 V) |
| Target corner | FILL |

## 3. Tools and technology

| Tool | Purpose |
|---|---|
| Icarus Verilog / GTKWave (edit if different) | RTL simulation and waveforms |
| Yosys (edit if different) | Logic synthesis |
| OpenROAD / OpenROAD-flow-scripts | Floorplan, placement, CTS, routing, STA, parasitics |
| KLayout 0.28 | GDS viewing, DRC, LVS |
| SkyWater sky130 PDK, `sky130_fd_sc_hd` | Technology files and standard cells |
| Oracle VirtualBox VM | Development environment |

## 4. RTL design

Verilog source for `led_blinker` is in `rtl/`. It contains a clocked counter with reset, and the LED output is derived from the counter state.

## 5. Functional simulation

The testbench in `tb/` applies a clock and reset and checks that the LED toggles as expected. Simulation was run before synthesis to confirm functional correctness.

| Item | Result |
|---|---|
| Testbench | `tb/` |
| Simulator | FILL |
| Result | FILL (pass, waveform attached) |

## 6. Logic synthesis

The RTL was mapped to `sky130_fd_sc_hd` standard cells under the constraints in `constraints/`. Outputs are in `build/`, with schematics in `schematics/`.

| Cell | Count |
|---|---|
| sky130_fd_sc_hd__a21oi_1 | 2 |
| sky130_fd_sc_hd__and3_1 | 1 |
| sky130_fd_sc_hd__dfxtp_1 | 3 |
| sky130_fd_sc_hd__nor2_1 | 1 |
| sky130_fd_sc_hd__nor3_1 | 1 |
| sky130_fd_sc_hd__o21a_1 | 1 |

9 logic cells before CTS. Synthesis area: FILL.

## 7. Floorplanning

Script: `pnr/stage1_floorplan_place.tcl`. The die and core area, utilization and aspect ratio were set, and I/O pins for `clk`, `rst` and `led` were placed.

| Item | Value |
|---|---|
| Die area | FILL |
| Core area | FILL |
| Utilization | FILL |
| Output | `pnr/results/1_floorplan.odb` |

## 8. Power planning

A power distribution network (PDN) was created: met1 follow-pin rails for the standard cells, and vertical met4 stripes for VDD and VSS, connected with vias. 4 tap cells were inserted to tie the wells. Output: `pnr/results/2_power.odb`.

## 9. Placement

Global placement followed by detailed placement legalised the cells onto the row grid. Outputs: `3_global_place.odb` and `4_detail_place.odb`.

## 10. Clock tree synthesis

Script: `pnr/stage2_cts.tcl`. CTS built a buffered clock tree from `clk` to the flip-flops, adding 4 clock cells: 3 `clkbuf_16` and 1 `clkbuf_4`. Output: `5_cts.odb`.

| Item | Value |
|---|---|
| Clock buffers inserted | 4 |
| Clock skew | FILL |

## 11. Routing

Script: `pnr/stage3_route.tcl`. Global routing (`6_global_route.odb`) was followed by detailed routing (`7_detail_route.odb`). 24 fill cells were added at the end.

| Item | Value |
|---|---|
| Routing DRC violations | FILL |
| Total wirelength | FILL |

## 12. Parasitic extraction

Script: `pnr/stage4_corner.tcl`. Parasitics were extracted from the routed design and written to `pnr/results/led_blinker.spef`.

## 13. Static timing analysis

Setup and hold timing were checked using the extracted parasitics.

| Check | Worst slack | Result |
|---|---|---|
| Setup | FILL | FILL |
| Hold | FILL | FILL |

## 14. Power analysis

| Component | Power |
|---|---|
| Internal | FILL |
| Switching | FILL |
| Leakage | FILL |
| Total | FILL |

## 15. GDSII generation

The final database `pnr/results/8_final.odb` was exported to `gds/led_blinker.gds`, along with `led_blinker_final.def` and `led_blinker_final.v`.

## 16. Physical verification: DRC

KLayout DRC on the final GDSII reported **no violations**.

## 17. Physical verification: LVS

KLayout LVS compared the layout extracted from the GDSII with a reference netlist written from the OpenROAD database and combined with the platform standard-cell CDL. Result: `Congratulations! Netlists match.` All signal nets (`clk`, `led`, `rst`, internal) and all 13 cells paired.

The platform LVS deck and the delivered `led_blinker.gds` were not modified. Clean LVS needed three reconciliation steps on working copies:

| Problem found | Cause | Fix |
|---|---|---|
| Power nets unnamed in layout | The PDN stripes carry no text labels, so the deck's VNB-to-VSS join could not run | `label_power.py` adds `VDD` and `VSS` labels on met4 (layer 71/5) in a copy of the GDS |
| Extra nets in the reference | Fill cells have no devices and are never extracted from the layout | Fill cells removed from the reference netlist |
| 4 CTS cells with unconnected power pins, plus 4 tap cells with no devices | Reference netlist exported from the ODB | `fix_ref.py` ties the clock cells' power pins to `VDD`/`VSS` using the platform CDL pin order, and drops the tap cells |

## 18. Results summary

| Stage | Status |
|---|---|
| RTL and simulation | Done |
| Synthesis | Done |
| Floorplan and power planning | Done |
| Placement | Done |
| CTS | Done (4 clock cells) |
| Routing | Done |
| Parasitic extraction and STA | Done |
| GDSII | Done |
| DRC | Clean |
| LVS | Match |

## 19. Problems faced and solutions

- **LVS mismatch on power nets.** Cause: unlabelled PDN stripes and a reference netlist exported with unconnected clock-cell power pins. Fixed with labelled and repaired working copies (see section 17).
- **Fill and tap cells in the reference.** These contain no devices and were excluded from the comparison.
- **Debugging method.** Net names and pin counts per net were compared between the extracted and reference netlists, which showed that the power nets had 26 pins in the layout and 22 in the reference.

## 20. Repository layout

```
LED_TOGGLE_PROJECT/
├── README.md
├── rtl/                    RTL source
├── tb/                     testbench
├── constraints/            design constraints
├── schematics/             schematics
├── build/                  synthesis outputs
├── pnr/
│   ├── stage1_floorplan_place.tcl
│   ├── stage2_cts.tcl
│   ├── stage3_route.tcl
│   ├── stage4_corner.tcl
│   ├── save_images.tcl
│   ├── images/             layout images
│   ├── reports/            PnR reports
│   └── results/            ODB per stage, final DEF, netlist, SPEF
├── gds/
│   ├── led_blinker.gds             final GDSII
│   ├── led_blinker_lvs2.gds        copy with VDD/VSS labels (LVS only)
│   ├── led_blinker_nofill2.cdl     repaired reference netlist
│   ├── led_blinker_full3.cdl       platform CDL + reference netlist
│   ├── lvs_report6.lvsdb           LVS report (match)
│   ├── led_blinker_extracted6.cir  netlist extracted from layout
│   ├── lvs6.log
│   ├── label_power.py              adds VDD/VSS labels on met4
│   ├── fix_ref.py                  repairs the reference netlist
│   └── write_cdl2.tcl              writes CDL from the ODB
└── reports/                Word and PDF reports
```

## 21. How to reproduce

PnR stages, in order:

```bash
cd ~/LED_TOGGLE_PROJECT/pnr
openroad stage1_floorplan_place.tcl
openroad stage2_cts.tcl
openroad stage3_route.tcl
openroad stage4_corner.tcl
```

(Edit the commands if your stage scripts are launched differently.)

LVS:

```bash
W=~/LED_TOGGLE_PROJECT/gds
PLAT=~/OpenROAD-flow-scripts-master/flow/platforms/sky130hd

klayout -b -r $W/label_power.py
openroad -no_splash -exit $W/write_cdl2.tcl
python3 $W/fix_ref.py
cat $PLAT/cdl/sky130hd.cdl $W/led_blinker_nofill2.cdl > $W/led_blinker_full3.cdl

klayout -b \
  -rd in_gds=$W/led_blinker_lvs2.gds \
  -rd report_file=$W/lvs_report6.lvsdb \
  -rd cdl_file=$W/led_blinker_full3.cdl \
  -rd target_netlist=$W/led_blinker_extracted6.cir \
  -rd top_cell=led_blinker \
  -r $PLAT/lvs/sky130hd.lylvs
```

Expected final message: `Congratulations! Netlists match.`

## 22. Conclusion and future work

The `led_blinker` design completed the full RTL-to-GDSII flow on sky130hd with a DRC-clean layout and a layout-versus-schematic match, and the LVS reconciliation steps are documented for reproducibility.

Possible extensions: a larger counter or programmable blink rate, multi-corner timing sign-off, and automating the LVS power-net labelling inside the flow.

P.HARSHA VARDHAN REDDY
24EG104B20
