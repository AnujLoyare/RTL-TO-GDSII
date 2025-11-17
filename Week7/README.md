# Week 7: BabySoC Physical Design & Post-Route SPEF Generation

## 📋 Table of Contents
1. [Introduction & Setup](#introduction--setup)
2. [Floorplanning](#1-babysoc-floorplanning)
3. [Placement](#2-placement)
4. [Clock Tree Synthesis](#3-clock-tree-synthesis)
5. [Routing](#4-routing)
6. [Post-Route SPEF Generation](#5-post-route-spef-generation)
7. [Verification & Reports](#6-verification--reports)
8. [Documentation Template](#7-documentation-template)

---

## Introduction & Setup

### Prerequisites
- OpenROAD installed and configured
- BabySoC synthesized netlist (`.v` file)
- PDK files (SKY130 or equivalent)
- Liberty timing files (`.lib`)
- LEF files for standard cells and macros

### Directory Structure
```
BabySoC_PD/
├── design/
│   ├── vsdbabysoc.synthesis.v    # Synthesized netlist
│   └── vsdbabysoc.sdc             # Timing constraints
├── lib/
│   ├── sky130_fd_sc_hd__tt_025C_1v80.lib
│   └── avsddac.lib, avsdpll.lib
├── lef/
│   ├── sky130_fd_sc_hd.tlef
│   ├── sky130_fd_sc_hd_merged.lef
│   └── avsddac.lef, avsdpll.lef
├── scripts/
│   └── openroad_flow.tcl
└── results/
    ├── floorplan/
    ├── placement/
    ├── cts/
    ├── routing/
    └── spef/
```

---

## 1. BabySoC Floorplanning

### Step 1.1: Launch OpenROAD
```tcl
# Start OpenROAD
openroad

# Or run with script
openroad -gui scripts/floorplan.tcl
```

### Step 1.2: Read Design Files
```tcl
# Read liberty files
read_liberty lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_liberty lib/avsddac.lib
read_liberty lib/avsdpll.lib

# Read LEF files
read_lef lef/sky130_fd_sc_hd.tlef
read_lef lef/sky130_fd_sc_hd_merged.lef
read_lef lef/avsddac.lef
read_lef lef/avsdpll.lef

# Read synthesized netlist
read_verilog design/vsdbabysoc.synthesis.v
link_design vsdbabysoc

# Read constraints
read_sdc design/vsdbabysoc.sdc
```

### Step 1.3: Initialize Floorplan
```tcl
# Initialize floorplan
# Syntax: initialize_floorplan -die_area {llx lly urx ury} \
#         -core_area {llx lly urx ury} \
#         -site unithd

# Example for a 1000um x 1000um die
initialize_floorplan \
    -die_area "0 0 1000 1000" \
    -core_area "10 10 990 990" \
    -site unithd

# Alternative: Use utilization-based approach
initialize_floorplan \
    -utilization 40 \
    -aspect_ratio 1.0 \
    -core_space 10 \
    -site unithd
```

### Step 1.4: Place Macros (DAC and PLL)
```tcl
# Manual macro placement
place_cell DAC_instance {200 200} R0
place_cell PLL_instance {700 200} R0

# Or use automatic macro placement
macro_placement \
    -halo {10 10} \
    -channel {20 20}
```

### Step 1.5: Create Power Distribution Network (PDN)
```tcl
# Add power rings
add_global_connection -net VDD -pin_pattern {^VDD$} -power
add_global_connection -net VSS -pin_pattern {^VSS$} -ground

# Define power grid
pdngen \
    -verbose \
    vsdbabysoc_pdn.cfg

# Or manual PDN specification
set_voltage_domain -power VDD -ground VSS

# Create power rings
add_pdn_ring \
    -grid stdcell \
    -layers {met5 met6} \
    -widths {5.0 5.0} \
    -spacings {2.0 2.0} \
    -core_offsets {10 10}

# Create power stripes
add_pdn_stripe \
    -layer met4 \
    -width 2.0 \
    -pitch 100.0 \
    -offset 50.0
```

### Step 1.6: Save Floorplan & Capture Screenshot
```tcl
# Write floorplan DEF
write_def results/floorplan/vsdbabysoc.floorplan.def

# Save floorplan image
save_image results/floorplan/floorplan_view.png

# In GUI mode: File > Save Image
```

**📸 Deliverable 1: Floorplan Screenshot**
- Capture the floorplan showing die area, core area, and macro placement

---

## 2. Placement

### Step 2.1: Global Placement
```tcl
# Run global placement
global_placement \
    -density 0.65 \
    -pad_left 2 \
    -pad_right 2

# Check placement legality
check_placement -verbose
```

### Step 2.2: Detailed Placement
```tcl
# Run detailed placement
detailed_placement

# Optimize placement
optimize_placement

# Check for overlaps
check_placement -verbose
```

### Step 2.3: Verify Placement
```tcl
# Generate placement report
report_design_area

# Check density
report_placement_utilization

# Timing report after placement
report_checks -path_delay max -format full_clock_expanded
```

### Step 2.4: Save Placement Results
```tcl
# Write placement DEF
write_def results/placement/vsdbabysoc.placement.def

# Save placement image
save_image results/placement/placement_view.png
```

**📸 Deliverable 2: Placement Screenshot**
- Show the placed standard cells and macros
- Highlight density distribution

---

## 3. Clock Tree Synthesis

### Step 3.1: Configure CTS
```tcl
# Set CTS parameters
set_wire_rc -clock -layer met3
set_clock_buffer_list {sky130_fd_sc_hd__clkbuf_1 \
                        sky130_fd_sc_hd__clkbuf_2 \
                        sky130_fd_sc_hd__clkbuf_4}

# Set clock root
set_propagated_clock [all_clocks]
```

### Step 3.2: Run CTS
```tcl
# Synthesize clock tree
clock_tree_synthesis \
    -buf_list {sky130_fd_sc_hd__clkbuf_1 \
               sky130_fd_sc_hd__clkbuf_2 \
               sky130_fd_sc_hd__clkbuf_4} \
    -root_buf sky130_fd_sc_hd__clkbuf_8 \
    -wire_unit 20

# Report clock skew
report_clock_skew
```

### Step 3.3: Save CTS Results
```tcl
write_def results/cts/vsdbabysoc.cts.def
save_image results/cts/cts_view.png
```

---

## 4. Routing

### Step 4.1: Global Routing
```tcl
# Set routing layers
set_global_routing_layer_adjustment met1-met5 0.5

# Run global routing
global_route \
    -guide_file results/routing/route.guide \
    -verbose

# Check routing congestion
report_routing_congestion
```

### Step 4.2: Detailed Routing
```tcl
# Run detailed routing
detailed_route \
    -output_drc results/routing/drc.rpt \
    -output_maze results/routing/maze.log \
    -verbose

# Or use TritonRoute
set_thread_count 4
detailed_route -verbose 1
```

### Step 4.3: Fix DRC Violations
```tcl
# Check for violations
check_drc -verbose

# If violations exist, run repair
repair_design -max_wire_length 1000

# Re-run detailed routing
detailed_route -verbose 1
```

### Step 4.4: Verify Routing
```tcl
# Check antenna violations
check_antennas

# Verify connectivity
check_connectivity

# Report routing statistics
report_routing_statistics
```

### Step 4.5: Save Routed Design
```tcl
# Write routed DEF
write_def results/routing/vsdbabysoc.routed.def

# Write final GDS (if needed)
write_gds results/routing/vsdbabysoc.gds

# Save routing image
save_image results/routing/routed_view.png
```

**📸 Deliverable 3: Routing Screenshot**
- Show the complete routed design
- Highlight routing layers used

---

## 5. Post-Route SPEF Generation

### 🎯 What is SPEF?

**Standard Parasitic Exchange Format (SPEF)** contains:
- **Resistance (R)**: Wire resistance causing IR drop
- **Capacitance (C)**: Coupling and ground capacitance
- **Coupling effects**: Between adjacent nets
- **Net connectivity**: Driver, loads, and parasitics

SPEF is **critical** for:
1. Accurate post-route Static Timing Analysis (STA)
2. IR drop analysis
3. Signal integrity verification
4. Power analysis

### Step 5.1: Extract Parasitics
```tcl
# Extract RC parasitics from routed design
extract_parasitics \
    -ext_model_file pdk/extmodel

# Or use external extractor
# RCX extraction
define_process_corner -ext_model_index 0
extract_rc

# Generate SPEF
write_spef results/spef/vsdbabysoc.spef
```

### Step 5.2: Verify SPEF File
```bash
# Check SPEF file exists
ls -lh results/spef/vsdbabysoc.spef

# View SPEF header
head -n 50 results/spef/vsdbabysoc.spef

# Count nets in SPEF
grep "^\*D_NET" results/spef/vsdbabysoc.spef | wc -l
```

### Step 5.3: SPEF File Structure Example
```spef
*SPEF "IEEE 1481-1998"
*DESIGN "vsdbabysoc"
*DATE "Sun Nov 16 2025"
*VENDOR "OpenROAD"
*PROGRAM "OpenSTA"
*VERSION "2.0"
*DIVIDER /
*DELIMITER :
*BUS_DELIMITER [ ]

*T_UNIT 1 PS
*C_UNIT 1 FF
*R_UNIT 1 OHM
*L_UNIT 1 HENRY

*D_NET net123 1.234
*CONN
*P clk_port I
*I inst1:A I
*I inst2:B O
*CAP
1 net123:1 0.145
2 net123:2 0.098
*RES
1 net123:1 net123:2 12.5
*END

// More nets...
```

### Step 5.4: Use SPEF in STA
```tcl
# Read SPEF into STA
read_spef results/spef/vsdbabysoc.spef

# Run post-route timing analysis
report_checks -path_delay max -format full_clock_expanded \
    -fields {input_pin slew cap} > results/spef/post_route_timing.rpt

# Check setup/hold
report_checks -path_delay min_max -format summary

# Report worst paths
report_worst_slack -max
report_worst_slack -min

# Total negative slack
report_tns

# Worst negative slack
report_wns
```

**📸 Deliverable 4: SPEF Generation Output**
- Terminal screenshot showing SPEF extraction
- First few lines of generated SPEF file
- Post-route timing report

---

## 6. Verification & Reports

### Generate Final Reports
```tcl
# Design statistics
report_design_area > results/reports/design_area.rpt
report_cell_usage > results/reports/cell_usage.rpt

# Timing reports
report_checks -path_delay max -format full_clock_expanded \
    > results/reports/setup_timing.rpt
report_checks -path_delay min -format full_clock_expanded \
    > results/reports/hold_timing.rpt

# Power report
report_power > results/reports/power.rpt

# DRC report
check_drc -verbose > results/reports/drc.rpt

# Antenna report
check_antennas > results/reports/antenna.rpt
```

### Key Metrics to Document
| Metric | Command | Expected Range |
|--------|---------|----------------|
| Total Cells | `report_design_area` | Varies by design |
| Core Utilization | `report_design_area` | 40-70% |
| Total Nets | `report_design_area` | ~cell count |
| WNS (Setup) | `report_wns -max` | > 0 (positive) |
| WNS (Hold) | `report_wns -min` | > 0 (positive) |
| Total Power | `report_power` | mW range |
| DRC Violations | `check_drc` | 0 |

---

## 7. Documentation Template

### GitHub Repository Structure
```
BabySoC_Week7/
├── README.md                    # Main documentation
├── images/
│   ├── floorplan_view.png
│   ├── placement_view.png
│   ├── routed_view.png
│   └── spef_generation.png
├── scripts/
│   ├── floorplan.tcl
│   ├── placement.tcl
│   ├── routing.tcl
│   └── complete_flow.tcl
├── reports/
│   ├── design_area.rpt
│   ├── timing.rpt
│   └── power.rpt
└── spef/
    └── vsdbabysoc.spef
```

### README.md Template
```markdown
# Week 7: BabySoC Physical Design Implementation

## Overview
Complete physical design implementation of BabySoC from floorplanning 
to post-route SPEF generation using OpenROAD.

## Design Specifications
- **Technology**: SKY130 130nm
- **Core Utilization**: 45%
- **Die Area**: 1000um x 1000um
- **Clock Period**: 10ns (100MHz)

## Flow Summary

### 1. Floorplanning
**Commands Used:**
```tcl
initialize_floorplan -utilization 45 -aspect_ratio 1.0
```

**Results:**
- Die Area: 1000µm x 1000µm
- Core Area: 980µm x 980µm
- Macro Placement: DAC at (200,200), PLL at (700,200)

![Floorplan](images/floorplan_view.png)

### 2. Placement
**Commands Used:**
```tcl
global_placement -density 0.65
detailed_placement
```

**Results:**
- Total Cells: XXXX
- Utilization: XX%
- Placement Legality: PASS

![Placement](images/placement_view.png)

### 3. Routing
**Commands Used:**
```tcl
global_route -verbose
detailed_route -verbose 1
```

**Results:**
- Total Nets Routed: XXXX
- DRC Violations: 0
- Routing Layers Used: met1-met5

![Routed Design](images/routed_view.png)

### 4. SPEF Generation
**Commands Used:**
```tcl
extract_parasitics
write_spef results/vsdbabysoc.spef
```

**SPEF Statistics:**
- Total Nets with Parasitics: XXXX
- File Size: XX MB
- Extraction Time: XX seconds

![SPEF Generation](images/spef_generation.png)

## What is SPEF and Why It Matters

SPEF (Standard Parasitic Exchange Format) captures the parasitic 
resistance and capacitance of interconnect wires after routing.

**Key Components:**
1. **Net Resistance**: Wire resistance causing voltage drop
2. **Capacitance**: Ground and coupling capacitance
3. **Topology**: RC network structure

**Critical for:**
- Accurate post-route timing analysis
- Signal integrity verification
- Power analysis and IR drop
- Final timing closure

## Timing Results

### Pre-Route vs Post-Route Comparison
| Metric | Pre-Route | Post-Route (with SPEF) |
|--------|-----------|------------------------|
| WNS (Setup) | +0.5ns | +0.2ns |
| TNS | 0.0ns | 0.0ns |
| WNS (Hold) | +0.1ns | +0.05ns |

Post-route timing degraded by ~0.3ns due to wire parasitics.

## Challenges & Solutions

### Challenge 1: High Placement Density
**Issue**: Initial placement had 80% density causing routing congestion
**Solution**: Reduced to 65% density, improved routability

### Challenge 2: DRC Violations After Routing
**Issue**: 47 spacing violations on met3
**Solution**: Ran detailed route with -repair option

### Challenge 3: SPEF Extraction Warnings
**Issue**: Some nets had missing RC data
**Solution**: Verified LEF files contained correct layer definitions

## Verification Checklist
- [x] Floorplan completed with proper boundaries
- [x] All macros placed without overlap
- [x] Placement legal and optimized
- [x] Routing completed with 0 DRC violations
- [x] SPEF extracted successfully
- [x] Post-route timing meets constraints
- [x] Power grid verified

## Key Learnings
1. Wire parasitics significantly impact timing (30% degradation observed)
2. SPEF is essential for accurate post-route STA
3. Placement density directly affects routing congestion
4. PDN planning must happen early in floorplan stage

## Files Included
- `scripts/`: All TCL scripts used
- `reports/`: Generated reports
- `spef/`: Post-route SPEF file
- `images/`: Layout screenshots

## References
- OpenROAD Documentation: https://openroad.readthedocs.io
- Task 13 Reference: [Link to repository]
```

---

## 🎯 Success Criteria

By the end of this task, you should have:

✅ **Complete Physical Design Flow**
- Floorplan with defined die/core areas
- Placed and optimized standard cells
- Fully routed design with 0 DRC violations

✅ **SPEF Generation**
- Valid SPEF file generated
- SPEF successfully read into STA tool
- Post-route timing analysis completed

✅ **Documentation**
- Screenshots at each stage
- Command log with explanations
- Analysis of parasitic impact on timing

✅ **Understanding**
- How each PD stage connects
- Why SPEF is critical for timing closure
- Real-world ASIC design flow

---

## 📚 Additional Resources

### OpenROAD Commands Quick Reference
```tcl
# Essential commands
initialize_floorplan    # Create floorplan
global_placement       # Coarse placement
detailed_placement     # Fine-grain placement
clock_tree_synthesis   # Build clock tree
global_route          # Coarse routing
detailed_route        # Fine routing
extract_parasitics    # Generate RC data
write_spef            # Export SPEF file
```

### Debugging Tips
1. **Placement fails**: Reduce utilization or increase die size
2. **Routing congestion**: Lower placement density
3. **DRC violations**: Check LEF file layer definitions
4. **SPEF errors**: Verify extraction model files exist
5. **Timing violations**: Optimize placement or adjust constraints

---

## 🚀 Next Steps (Week 8 Preview)

After completing physical design and SPEF generation:
1. Use SPEF in comprehensive STA analysis
2. Perform sign-off checks (DRC, LVS, Antenna)
3. Generate final GDS file for fabrication
4. Complete power analysis with parasitics
5. Document full RTL-to-GDS flow

**Good luck with your BabySoC physical design implementation!** 🎉
