# Week 8: Post-Layout STA & Timing Analysis Across PVT Corners
## Complete Step-by-Step Guide with SPEF

---

## 📋 Table of Contents
1. [Overview & Objectives](#overview--objectives)
2. [Understanding PVT Corners](#understanding-pvt-corners)
3. [Setup & Prerequisites](#setup--prerequisites)
4. [Post-Route STA with SPEF](#post-route-sta-with-spef)
5. [Multi-Corner Timing Analysis](#multi-corner-timing-analysis)
6. [Week 3 vs Week 8 Comparison](#week-3-vs-week-8-comparison)
7. [Timing Graphs Generation](#timing-graphs-generation)
8. [Analysis & Documentation](#analysis--documentation)

---

## Overview & Objectives

### What You'll Accomplish
✅ Load post-route design with SPEF into OpenSTA  
✅ Run timing analysis across all PVT corners  
✅ Generate timing reports and graphs  
✅ Compare pre-route (Week 3) vs post-route (Week 8) timing  
✅ Understand impact of parasitic extraction on timing  

### Why This Matters
OpenLane's timing flow uses incremental timing analysis and SPEF extraction for better timing closure at various flow stages. This final step validates that your design meets timing requirements across all operating conditions before fabrication.

---

## Understanding PVT Corners

### What is PVT?

**PVT = Process, Voltage, Temperature**

| Factor | Impact | Range |
|--------|--------|-------|
| **Process (P)** | Transistor variations during manufacturing | Fast (F), Typical (T), Slow (S) |
| **Voltage (V)** | Supply voltage variations | 1.62V, 1.8V, 1.98V (±10%) |
| **Temperature (T)** | Operating temperature | -40°C, 25°C, 125°C |

### Common PVT Corners for SKY130

| Corner | Process | Voltage | Temp | Use Case | Speed |
|--------|---------|---------|------|----------|-------|
| **ff_n40C_1v95** | Fast-Fast | 1.95V | -40°C | Best case (hold) | Fastest ⚡ |
| **ff_100C_1v95** | Fast-Fast | 1.95V | 100°C | Fast, hot | Fast |
| **tt_025C_1v80** | Typical-Typical | 1.80V | 25°C | Nominal (typical) | Nominal |
| **ss_100C_1v60** | Slow-Slow | 1.60V | 100°C | Worst case (setup) | Slowest 🐌 |
| **ss_n40C_1v76** | Slow-Slow | 1.76V | -40°C | Slow, cold | Slow |

### Corner Analysis Strategy

```
Setup Timing (Max Delay):
  Check at SLOWEST corner: ss_100C_1v60
  → If this passes, faster corners will pass

Hold Timing (Min Delay):
  Check at FASTEST corner: ff_n40C_1v95
  → If this passes, slower corners will pass

Best Practice:
  Analyze ALL corners to catch corner-specific violations
```

---

## Setup & Prerequisites

### Required Files from Week 7

```bash
# Navigate to Week 7 results
cd ~/vlsi_design/week7/OpenLane/designs/vsdbabysoc/runs/week7_run1/

# Verify you have these files:
ls results/routing/vsdbabysoc.def           # Routed DEF
ls results/routing/vsdbabysoc.nom.spef      # SPEF file ← KEY FILE
ls results/synthesis/vsdbabysoc.v           # Gate-level netlist
ls tmp/merged.nom.lef                       # LEF file
```

### Library Files for All Corners

```bash
# Check available liberty files
ls ~/vlsi_design/week7/VSDBabySoC/lib/

# Expected files:
# sky130_fd_sc_hd__ff_100C_1v65.lib
# sky130_fd_sc_hd__ff_100C_1v95.lib
# sky130_fd_sc_hd__ff_n40C_1v56.lib
# sky130_fd_sc_hd__ff_n40C_1v95.lib
# sky130_fd_sc_hd__ss_100C_1v40.lib
# sky130_fd_sc_hd__ss_100C_1v60.lib
# sky130_fd_sc_hd__ss_n40C_1v28.lib
# sky130_fd_sc_hd__ss_n40C_1v76.lib
# sky130_fd_sc_hd__tt_025C_1v80.lib  ← Typical corner
# avsddac.lib
# avsdpll.lib
```

If library files are missing:
```bash
# Download from PDK
cd ~/vlsi_design/week7
cp -r ~/.volare/sky130A/libs.ref/sky130_fd_sc_hd/lib/*.lib \
      VSDBabySoC/lib/
```

### Create STA Working Directory

```bash
cd ~/vlsi_design
mkdir -p week8_sta
cd week8_sta

# Create subdirectories
mkdir -p scripts reports graphs data
```

---

## Post-Route STA with SPEF

### Step 1: Create OpenSTA Script for Single Corner

```bash
cd ~/vlsi_design/week8_sta/scripts
nano sta_post_route_typical.tcl
```

**Content:**
```tcl
# Post-Route STA Script for Typical Corner with SPEF
# Author: Your Name
# Date: November 2025

# Set design name
set design_name "vsdbabysoc"

# Define paths
set DESIGN_DIR "~/vlsi_design/week7/OpenLane/designs/vsdbabysoc/runs/week7_run1"
set LIB_DIR "~/vlsi_design/week7/VSDBabySoC/lib"

# Read liberty files (Typical corner)
puts "Reading liberty files for TT corner..."
read_liberty $LIB_DIR/sky130_fd_sc_hd__tt_025C_1v80.lib
read_liberty $LIB_DIR/avsddac.lib
read_liberty $LIB_DIR/avsdpll.lib

# Read gate-level netlist
puts "Reading gate-level netlist..."
read_verilog $DESIGN_DIR/results/synthesis/$design_name.v

# Link design
puts "Linking design..."
link_design $design_name

# Read SDC constraints
puts "Reading timing constraints..."
read_sdc $DESIGN_DIR/../../sdc/$design_name.sdc

# ============================================
# KEY STEP: Read SPEF for parasitic extraction
# ============================================
puts "Reading SPEF file (parasitic extraction)..."
read_spef $DESIGN_DIR/results/routing/$design_name.nom.spef

# Report design statistics
puts "\n=========================================="
puts "Design Statistics"
puts "=========================================="
report_design_area

# Setup timing analysis (max delay)
puts "\n=========================================="
puts "Setup Timing Analysis (Max Delay)"
puts "=========================================="
report_checks -path_delay max \
              -format full_clock_expanded \
              -fields {slew cap input_pin fanout} \
              -digits 4 \
              -nworst 10 \
              > ../reports/tt_setup_timing.rpt

# Hold timing analysis (min delay)
puts "\n=========================================="
puts "Hold Timing Analysis (Min Delay)"
puts "=========================================="
report_checks -path_delay min \
              -format full_clock_expanded \
              -fields {slew cap input_pin fanout} \
              -digits 4 \
              -nworst 10 \
              > ../reports/tt_hold_timing.rpt

# Summary reports
puts "\n=========================================="
puts "Timing Summary"
puts "=========================================="
report_worst_slack -max > ../reports/tt_wns_setup.rpt
report_worst_slack -min > ../reports/tt_wns_hold.rpt
report_tns -digits 4 > ../reports/tt_tns.rpt

# Check timing (combined)
report_checks -path_delay min_max \
              -format full_clock_expanded \
              -fields {slew cap input_pin} \
              -digits 4 \
              > ../reports/tt_timing_summary.rpt

# Power analysis
puts "\n=========================================="
puts "Power Analysis"
puts "=========================================="
report_power -digits 4 > ../reports/tt_power.rpt

# QoR summary
puts "\n=========================================="
puts "Quality of Results Summary"
puts "=========================================="
report_checks -path_delay min_max -format summary

puts "\n=========================================="
puts "Post-Route STA Complete (TT Corner)"
puts "=========================================="
```

**Save:** `Ctrl+O`, `Enter`, `Ctrl+X`

### Step 2: Run OpenSTA for Typical Corner

```bash
cd ~/vlsi_design/week8_sta

# Run STA
sta scripts/sta_post_route_typical.tcl | tee logs/sta_tt.log
```

**Expected Output:**
```
OpenSTA 2.x.x ...
Reading liberty files for TT corner...
Reading gate-level netlist...
Linking design...
Reading timing constraints...
Reading SPEF file (parasitic extraction)...

==========================================
Setup Timing Analysis (Max Delay)
==========================================
Startpoint: core/_9532_ (rising edge-triggered flip-flop clocked by CLK)
Endpoint: core/_10034_ (rising edge-triggered flip-flop clocked by CLK)
Path Group: CLK
Path Type: max

  Delay    Time   Description
---------------------------------------------------------
   0.00    0.00   clock CLK (rise edge)
   0.00    0.00   clock network delay (propagated)
   0.00    0.00 ^ core/_9532_/CLK (sky130_fd_sc_hd__dfxtp_1)
   0.45    0.45 ^ core/_9532_/Q (sky130_fd_sc_hd__dfxtp_1)
   0.23    0.68 v core/_8765_/A (sky130_fd_sc_hd__inv_2)
   ...
   8.95    8.95 ^ core/_10034_/D (sky130_fd_sc_hd__dfxtp_1)
  10.00   10.00   clock CLK (rise edge)
   0.00   10.00   clock network delay (propagated)
  -0.15    9.85   clock uncertainty
   0.00    9.85   core/_10034_/CLK (sky130_fd_sc_hd__dfxtp_1)
  -0.12    9.73   library setup time
           9.73   data required time
---------------------------------------------------------
           9.73   data required time
          -8.95   data arrival time
---------------------------------------------------------
           0.78   slack (MET)
```

### Step 3: Extract Key Metrics

```bash
cd ~/vlsi_design/week8_sta/reports

# Setup WNS (Worst Negative Slack)
grep "slack" tt_wns_setup.rpt

# Hold WNS
grep "slack" tt_wns_hold.rpt

# Total Negative Slack
grep "tns" tt_tns.rpt

# Create summary
cat > tt_summary.txt << EOF
Corner: TT (Typical, 25C, 1.8V)
================================
Setup WNS: $(grep "slack" tt_wns_setup.rpt | awk '{print $2}')
Hold WNS: $(grep "slack" tt_wns_hold.rpt | awk '{print $2}')
Total TNS: $(grep "tns" tt_tns.rpt | awk '{print $2}')
EOF

cat tt_summary.txt
```

---

## Multi-Corner Timing Analysis

### Step 4: Create Multi-Corner STA Script

```bash
cd ~/vlsi_design/week8_sta/scripts
nano sta_all_corners.tcl
```

**Content:**
```tcl
# Multi-Corner Post-Route STA Script
# Analyzes design across all PVT corners

set design_name "vsdbabysoc"
set DESIGN_DIR "~/vlsi_design/week7/OpenLane/designs/vsdbabysoc/runs/week7_run1"
set LIB_DIR "~/vlsi_design/week7/VSDBabySoC/lib"

# Define all corners to analyze
array set corners {
    ff_n40C_1v95  "Fast-Fast -40C 1.95V (Best case - Hold)"
    ff_100C_1v95  "Fast-Fast 100C 1.95V"
    tt_025C_1v80  "Typical-Typical 25C 1.80V (Nominal)"
    ss_100C_1v60  "Slow-Slow 100C 1.60V (Worst case - Setup)"
    ss_n40C_1v76  "Slow-Slow -40C 1.76V"
}

# Create results table
puts "=========================================="
puts "Multi-Corner Timing Analysis"
puts "=========================================="
puts "Corner\t\t\tSetup WNS\tHold WNS\tTNS"
puts "------------------------------------------"

# Loop through each corner
foreach {corner desc} [array get corners] {
    puts "\n=========================================="
    puts "Analyzing Corner: $corner"
    puts "Description: $desc"
    puts "=========================================="
    
    # Clear previous design
    clear
    
    # Read liberty for this corner
    read_liberty $LIB_DIR/sky130_fd_sc_hd__${corner}.lib
    read_liberty $LIB_DIR/avsddac.lib
    read_liberty $LIB_DIR/avsdpll.lib
    
    # Read netlist
    read_verilog $DESIGN_DIR/results/synthesis/$design_name.v
    link_design $design_name
    
    # Read constraints
    read_sdc $DESIGN_DIR/../../sdc/$design_name.sdc
    
    # Read SPEF
    read_spef $DESIGN_DIR/results/routing/$design_name.nom.spef
    
    # Setup timing
    report_checks -path_delay max \
                  -format full_clock_expanded \
                  -digits 4 \
                  -nworst 5 \
                  > ../reports/${corner}_setup.rpt
    
    # Hold timing
    report_checks -path_delay min \
                  -format full_clock_expanded \
                  -digits 4 \
                  -nworst 5 \
                  > ../reports/${corner}_hold.rpt
    
    # Get WNS and TNS
    set setup_wns [sta::worst_slack -max]
    set hold_wns [sta::worst_slack -min]
    set tns [sta::total_negative_slack -max]
    
    # Print summary
    puts "$corner\t$setup_wns\t$hold_wns\t$tns"
    
    # Save to file
    set fp [open "../reports/${corner}_summary.txt" w]
    puts $fp "Corner: $corner ($desc)"
    puts $fp "Setup WNS: $setup_wns ns"
    puts $fp "Hold WNS: $hold_wns ns"
    puts $fp "Total TNS: $tns ns"
    close $fp
}

puts "\n=========================================="
puts "Multi-Corner Analysis Complete"
puts "=========================================="
puts "Reports saved in reports/ directory"
```

**Save and exit**

### Step 5: Run Multi-Corner Analysis

```bash
cd ~/vlsi_design/week8_sta

# Run multi-corner STA
sta scripts/sta_all_corners.tcl | tee logs/sta_all_corners.log

# This will generate reports for each corner
# Estimated time: 10-15 minutes
```

### Step 6: Compile Results into Table

```bash
cd ~/vlsi_design/week8_sta
nano scripts/extract_results.sh
```

**Content:**
```bash
#!/bin/bash
# Extract timing results from all corner reports

echo "=========================================="
echo "Post-Route Timing Summary (Week 8)"
echo "=========================================="
echo ""
printf "%-20s %-15s %-15s %-15s\n" "Corner" "Setup WNS" "Hold WNS" "TNS"
echo "------------------------------------------------------------------"

for corner in ff_n40C_1v95 ff_100C_1v95 tt_025C_1v80 ss_100C_1v60 ss_n40C_1v76; do
    if [ -f "reports/${corner}_summary.txt" ]; then
        setup_wns=$(grep "Setup WNS" reports/${corner}_summary.txt | awk '{print $3}')
        hold_wns=$(grep "Hold WNS" reports/${corner}_summary.txt | awk '{print $3}')
        tns=$(grep "Total TNS" reports/${corner}_summary.txt | awk '{print $3}')
        
        printf "%-20s %-15s %-15s %-15s\n" "$corner" "$setup_wns" "$hold_wns" "$tns"
    fi
done
```

**Make executable and run:**
```bash
chmod +x scripts/extract_results.sh
./scripts/extract_results.sh | tee reports/week8_summary_table.txt
```

**Expected Output:**
```
==========================================
Post-Route Timing Summary (Week 8)
==========================================

Corner               Setup WNS       Hold WNS        TNS            
------------------------------------------------------------------
ff_n40C_1v95         1.234 ns        0.123 ns        0.000 ns       
ff_100C_1v95         0.987 ns        0.098 ns        0.000 ns       
tt_025C_1v80         0.456 ns        0.234 ns        0.000 ns       
ss_100C_1v60         0.123 ns        0.456 ns        0.000 ns       
ss_n40C_1v76         0.234 ns        0.398 ns        0.000 ns       
```

---

## Week 3 vs Week 8 Comparison

### Step 7: Retrieve Week 3 (Post-Synthesis) Results

```bash
# Navigate to Week 3 OpenSTA results
cd ~/vlsi_design/week3_synthesis

# If you have saved reports:
ls reports/sta_*

# Otherwise, re-run post-synthesis STA (without SPEF)
```

### Create Pre-Route STA Script (Week 3 Recreation)

```bash
cd ~/vlsi_design/week8_sta/scripts
nano sta_pre_route_typical.tcl
```

**Content:**
```tcl
# Pre-Route STA (Post-Synthesis) - NO SPEF
# This represents Week 3 timing

set design_name "vsdbabysoc"
set DESIGN_DIR "~/vlsi_design/week7/OpenLane/designs/vsdbabysoc/runs/week7_run1"
set LIB_DIR "~/vlsi_design/week7/VSDBabySoC/lib"

# Read liberty
read_liberty $LIB_DIR/sky130_fd_sc_hd__tt_025C_1v80.lib
read_liberty $LIB_DIR/avsddac.lib
read_liberty $LIB_DIR/avsdpll.lib

# Read synthesized netlist (pre-route)
read_verilog $DESIGN_DIR/results/synthesis/$design_name.v
link_design $design_name

# Read constraints
read_sdc $DESIGN_DIR/../../sdc/$design_name.sdc

# NO SPEF - uses wire load models (estimated)

# Setup timing
report_checks -path_delay max \
              -format full_clock_expanded \
              -digits 4 \
              > ../reports/pre_route_tt_setup.rpt

# Hold timing
report_checks -path_delay min \
              -format full_clock_expanded \
              -digits 4 \
              > ../reports/pre_route_tt_hold.rpt

# Summary
report_worst_slack -max > ../reports/pre_route_tt_wns_setup.rpt
report_worst_slack -min > ../reports/pre_route_tt_wns_hold.rpt
report_tns > ../reports/pre_route_tt_tns.rpt

puts "Pre-Route STA Complete (NO SPEF)"
```

**Run it:**
```bash
cd ~/vlsi_design/week8_sta
sta scripts/sta_pre_route_typical.tcl | tee logs/sta_pre_route.log
```

### Step 8: Create Comparison Table

```bash
cd ~/vlsi_design/week8_sta
nano scripts/compare_week3_week8.sh
```

**Content:**
```bash
#!/bin/bash
# Compare Week 3 (Pre-Route) vs Week 8 (Post-Route)

echo "=============================================="
echo "Week 3 vs Week 8 Timing Comparison"
echo "=============================================="
echo ""

# Function to extract slack
get_slack() {
    file=$1
    grep "slack" $file | awk '{print $2}' | head -1
}

# Typical corner comparison
echo "Typical Corner (TT, 25C, 1.8V)"
echo "----------------------------------------------"

pre_setup=$(get_slack reports/pre_route_tt_wns_setup.rpt)
post_setup=$(get_slack reports/tt_wns_setup.rpt)

pre_hold=$(get_slack reports/pre_route_tt_wns_hold.rpt)
post_hold=$(get_slack reports/tt_wns_hold.rpt)

printf "%-25s %-15s %-15s %-15s\n" "Metric" "Week 3" "Week 8" "Degradation"
echo "--------------------------------------------------------------"
printf "%-25s %-15s %-15s " "Setup WNS" "$pre_setup" "$post_setup"
degradation=$(echo "$pre_setup - $post_setup" | bc)
printf "%-15s\n" "$degradation ns"

printf "%-25s %-15s %-15s " "Hold WNS" "$pre_hold" "$post_hold"
degradation=$(echo "$pre_hold - $post_hold" | bc)
printf "%-15s\n" "$degradation ns"

echo ""
echo "Key Observations:"
echo "- Post-route timing includes real parasitic R and C from SPEF"
echo "- Typical degradation: 0.2-0.5ns due to interconnect delays"
echo "- Larger degradation indicates high routing congestion"
```

**Run it:**
```bash
chmod +x scripts/compare_week3_week8.sh
./scripts/compare_week3_week8.sh | tee reports/week3_vs_week8_comparison.txt
```

---

## Timing Graphs Generation

### Step 9: Create Timing Path Visualization Script

```bash
cd ~/vlsi_design/week8_sta/scripts
nano generate_timing_graphs.py
```

**Content:**
```python
#!/usr/bin/env python3
"""
Generate timing graphs from OpenSTA reports
Visualizes setup/hold paths across PVT corners
"""

import re
import matplotlib.pyplot as plt
import numpy as np

def parse_timing_report(filename):
    """Extract path delays from timing report"""
    delays = []
    stages = []
    
    with open(filename, 'r') as f:
        content = f.read()
        
        # Find timing path section
        path_match = re.search(r'Delay\s+Time\s+Description(.*?)slack', content, re.DOTALL)
        if path_match:
            lines = path_match.group(1).strip().split('\n')
            for line in lines:
                match = re.match(r'\s*([\d.]+)\s+([\d.]+)\s+(.+)', line)
                if match:
                    delay = float(match.group(1))
                    time = float(match.group(2))
                    desc = match.group(3).strip()
                    delays.append(time)
                    stages.append(desc[:30])  # Truncate description
    
    return stages, delays

def plot_timing_path(corner, path_type='setup'):
    """Generate timing path graph for a corner"""
    filename = f'../reports/{corner}_{path_type}.rpt'
    
    try:
        stages, delays = parse_timing_report(filename)
        
        if not stages:
            print(f"No timing data found in {filename}")
            return
        
        # Create plot
        fig, ax = plt.subplots(figsize=(14, 6))
        
        x = np.arange(len(stages))
        ax.plot(x, delays, marker='o', linewidth=2, markersize=8, color='blue')
        
        # Formatting
        ax.set_xlabel('Path Stages', fontsize=12)
        ax.set_ylabel('Cumulative Delay (ns)', fontsize=12)
        ax.set_title(f'{corner} - {path_type.title()} Timing Path', fontsize=14, fontweight='bold')
        ax.grid(True, alpha=0.3)
        ax.set_xticks(x[::max(1, len(x)//10)])  # Show every 10th label
        ax.set_xticklabels(stages[::max(1, len(x)//10)], rotation=45, ha='right', fontsize=8)
        
        plt.tight_layout()
        plt.savefig(f'../graphs/{corner}_{path_type}_path.png', dpi=300)
        plt.close()
        
        print(f"✓ Generated: {corner}_{path_type}_path.png")
        
    except FileNotFoundError:
        print(f"✗ File not found: {filename}")

def plot_corner_comparison():
    """Compare WNS across all corners"""
    corners = ['ff_n40C_1v95', 'ff_100C_1v95', 'tt_025C_1v80', 'ss_100C_1v60', 'ss_n40C_1v76']
    setup_wns = []
    hold_wns = []
    
    for corner in corners:
        try:
            with open(f'../reports/{corner}_summary.txt', 'r') as f:
                content = f.read()
                setup = float(re.search(r'Setup WNS: ([\d.-]+)', content).group(1))
                hold = float(re.search(r'Hold WNS: ([\d.-]+)', content).group(1))
                setup_wns.append(setup)
                hold_wns.append(hold)
        except:
            setup_wns.append(0)
            hold_wns.append(0)
    
    # Create comparison bar chart
    x = np.arange(len(corners))
    width = 0.35
    
    fig, ax = plt.subplots(figsize=(12, 6))
    bars1 = ax.bar(x - width/2, setup_wns, width, label='Setup WNS', color='skyblue')
    bars2 = ax.bar(x + width/2, hold_wns, width, label='Hold WNS', color='salmon')
    
    ax.set_xlabel('PVT Corners', fontsize=12)
    ax.set_ylabel('Worst Negative Slack (ns)', fontsize=12)
    ax.set_title('Post-Route Timing Across PVT Corners', fontsize=14, fontweight='bold')
    ax.set_xticks(x)
    ax.set_xticklabels(corners, rotation=30, ha='right')
    ax.legend()
    ax.grid(True, alpha=0.3, axis='y')
    ax.axhline(y=0, color='r', linestyle='--', linewidth=1, label='Zero Slack')
    
    # Add value labels on bars
    for bars in [bars1, bars2]:
        for bar in bars:
            height = bar.get_height()
            ax.text(bar.get_x() + bar.get_width()/2., height,
                   f'{height:.3f}',
                   ha='center', va='bottom', fontsize=8)
    
    plt.tight_layout()
    plt.savefig('../graphs/pvt_corner_comparison.png', dpi=300)
    plt.close()
    
    print("✓ Generated: pvt_corner_comparison.png")

def main():
    print("========================================")
    print("Generating Timing Graphs")
    print("========================================\n")
    
    # Generate path graphs for each corner
    corners = ['ff_n40C_1v95', 'ff_100C_1v95', 'tt_025C_1v80', 'ss_100C_1v60', 'ss_n40C_1v76']
    
    for corner in corners:
        plot_timing_path(corner, 'setup')
        plot_timing_path(corner, 'hold')
    
    # Generate comparison graph
    plot_corner_comparison()
    
    print("\n========================================")
    print("All graphs generated successfully!")
    print("Location: week8_sta/graphs/")
    print("========================================")

if __name__ == "__main__":
    main()
```

**Save and run:**
```bash
chmod +x scripts/generate_timing_graphs.py
python3 scripts/generate_timing_graphs.py
```

**Generated Graphs:**
- `tt_025C_1v80_setup_path.png`
- `tt_025C_1v80_hold_path.png`
- `ss_100C_1v60_setup_path.png` (worst case setup)
- `ff_n40C_1v95_hold_path.png` (worst case hold)
- `pvt_corner_comparison.png` (all corners)

---

## Analysis & Documentation

### Step 10: Write Analysis Document

```bash
cd ~/vlsi_design/week8_sta
nano ANALYSIS.md
```

**Content Template:**
```markdown
# Week 8: Post-Layout STA Analysis Report

## Executive Summary
Post-route static timing analysis of VSDBabySoC reveals:
- ✅ Timing closure achieved across all PVT corners
- 📉 0.XX ns average degradation from post-synthesis to post-route
- 🔍 Real parasitic extraction (SPEF) critical for accurate timing

## Design Information
| Parameter | Value |
|-----------|-------|
| Design | VSDBabySoC |
| Technology | SKY130 (130nm) |
| Clock Period | 10.0 ns (100 MHz) |
| Routed Nets | XXXX |
| SPEF File Size | X.XX MB |
| Analysis Date | November 2025 |

## PVT Corner Analysis Results

### Post-Route Timing (Week 8 - With SPEF)

| Corner | Process | Voltage | Temp | Setup WNS | Hold WNS | TNS |
|--------|---------|---------|------|-----------|----------|-----|
| ff_n40C_1v95 | Fast | 1.95V | -40°C | 1.234 ns | 0.123 ns | 0.000 |
| ff_100C_1v95 | Fast | 1.95V | 100°C | 0.987 ns | 0.098 ns | 0.000 |
| tt_025C_1v80 | Typical | 1.80V | 25°C | 0.456 ns | 0.234 ns | 0.000 |
| ss_100C_1v60 | Slow | 1.60V | 100°C | 0.123 ns ⚠️ | 0.456 ns | 0.000 |
| ss_n40C_1v76 | Slow | 1.76V | -40°C | 0.234 ns | 0.398 ns | 0.000 |

⚠️ **Critical Corner:** ss_100C_1v60 (Slowest - Worst Setup)

### Week 3 vs Week 8 Comparison

#### Typical Corner (tt_025C_1v80)

| Metric | Week 3 (Pre-Route) | Week 8 (Post-Route) | Δ Change |
|--------|--------------------|---------------------|----------|
| Setup WNS | 0.789 ns | 0.456 ns | **-0.333 ns** |
| Hold WNS | 0.345 ns | 0.234 ns | **-0.111 ns** |
| TNS | 0.000 ns | 0.000 ns | 0.000 ns |
| Critical Path Delay | 9.211 ns | 9.544 ns | **+0.333 ns** |

**Degradation Analysis:**
- Setup slack reduced by 0.333 ns (42% degradation)
- Hold slack reduced by 0.111 ns (32% degradation)
- Still meets timing (positive slack maintained)

#### All Corners Comparison

| Corner | WNS Week 3 | WNS Week 8 | Degradation | Status |
|--------|------------|------------|-------------|--------|
| ff_n40C_1v95 | 1.567 ns | 1.234 ns | 0.333 ns | ✅ PASS |
| ff_100C_1v95 | 1.320 ns | 0.987 ns | 0.333 ns | ✅ PASS |
| tt_025C_1v80 | 0.789 ns | 0.456 ns | 0.333 ns | ✅ PASS |
| ss_100C_1v60 | 0.456 ns | 0.123 ns | 0.333 ns | ✅ PASS |
| ss_n40C_1v76 | 0.567 ns | 0.234 ns | 0.333 ns | ✅ PASS |

**Average Degradation: 0.333 ns across all corners**

## Key Findings

### 1. Impact of SPEF on Timing

**Without SPEF (Week 3):**
- Uses wire load models (statistical estimates)
- No coupling capacitance
- No distributed RC networks
- Optimistic timing

**With SPEF (Week 8):**
- Real extracted parasitics from layout
- Includes coupling between nets
- Distributed RC pi-models
- Realistic timing (±5% of silicon)

### 2. Parasitic Components

From SPEF analysis:
```
Average Net Statistics:
- Wire Resistance: 12.5 Ω per net
- Ground Capacitance: 45.6 fF per net
- Coupling Capacitance: 23.4 fF per net
- Total Capacitance: 69.0 fF per net
```

### 3. Critical Path Analysis

**Most Critical Path (ss_100C_1v60):**
```
Startpoint: core/CPU_Xreg_value_a4[17][0]$_DFF_P_/CLK
Endpoint: core/CPU_Xreg_value_a4[8][31]$_DFF_P_/D

Path Stages: 15 gates
Gate Delay: 7.234 ns (73%)
Wire Delay: 2.666 ns (27%) ← SPEF contribution

Total: 9.900 ns
Required: 10.000 ns
Slack: 0.100 ns
```

**Observation:** Wire delays account for 27% of total path delay!

### 4. Corner-Specific Observations

#### Fast Corners (ff_n40C_1v95, ff_100C_1v95)
- Highest positive slack
- **Hold timing critical** at these corners
- Fast transistors + low wire resistance
- Minimum delays pose hold risk

#### Typical Corner (tt_025C_1v80)
- Represents nominal operating conditions
- Balanced setup/hold margins
- Design target: WNS > 0.2 ns

#### Slow Corners (ss_100C_1v60, ss_n40C_1v76)
- Lowest positive slack
- **Setup timing critical** at these corners
- Slow transistors + high wire resistance
- Maximum delays challenge clock period

### 5. Why Timing Degraded?

**Physical Effects from SPEF:**

1. **Wire Resistance (R)**
   - Increases with wire length
   - More pronounced in long global routes
   - Causes voltage drop and delay

2. **Wire Capacitance (C)**
   - Ground capacitance: Always present
   - Coupling capacitance: Between adjacent wires
   - Slows signal transitions (RC delay)

3. **Coupling Effects**
   - Crosstalk between parallel wires
   - Aggressor-victim interactions
   - Can speed up OR slow down signals

**Mathematical Impact:**
```
Pre-route delay  = Gate_delay + Estimated_wire_delay
Post-route delay = Gate_delay + (R × C) + Coupling_effects

Typical wire RC delay per 1mm:
R ≈ 0.5 Ω/μm × 1000 = 500 Ω
C ≈ 0.2 fF/μm × 1000 = 200 fF
RC = 500 × 200e-15 = 100 ps per mm
```

## Visualization Analysis

### Graph 1: PVT Corner Comparison
![PVT Comparison](graphs/pvt_corner_comparison.png)

**Insights:**
- Fast corners have highest WNS (best slack)
- Slow corners have lowest WNS (worst slack)
- All corners pass timing (positive slack)
- Voltage variation has larger impact than temperature

### Graph 2: Typical Corner Setup Path
![TT Setup Path](graphs/tt_025C_1v80_setup_path.png)

**Path Characteristics:**
- Starts at register clock
- Propagates through 15 combinational stages
- Ends at register data input
- Step increases show gate delays
- Slopes show wire delays (SPEF)

### Graph 3: Worst Corner Setup Path
![SS Setup Path](graphs/ss_100C_1v60_setup_path.png)

**Critical Observations:**
- Longer cumulative delay
- Steeper wire delay slopes
- Tight timing margin
- Potential optimization target

## SPEF File Analysis

### SPEF Statistics

```bash
File: vsdbabysoc.nom.spef
Size: 2.34 MB
Nets: 1,234
Capacitances: 5,678
Resistances: 4,567
```

### Sample SPEF Entry (Critical Net)

```spef
*D_NET core/CPU_src1_value_a3[4] 0.234567
*CONN
*P core/CPU_Xreg_value_a4[17][4]/Q O *C 123.456 234.567
*I core/_8765_/A I *C 345.678 456.789
*I core/_8766_/B I *C 567.890 678.901
*CAP
1 core/CPU_src1_value_a3[4]:1 4.567e-14
2 core/CPU_src1_value_a3[4]:2 6.789e-14
3 core/CPU_src1_value_a3[4]:3 3.456e-14
4 core/CPU_src1_value_a3[4]:1 core/CPU_src2_value_a3[4]:1 1.234e-14
*RES
1 core/CPU_src1_value_a3[4]:1 core/CPU_src1_value_a3[4]:2 12.34
2 core/CPU_src1_value_a3[4]:2 core/CPU_src1_value_a3[4]:3 8.76
*END
```

**Interpretation:**
- Total net capacitance: 0.234567 pF
- 3 segments with ground caps
- 1 coupling cap to adjacent net
- 2 resistive segments
- Forms distributed RC network

### Parasitic Network Model

```
Driver                    Load1        Load2
  Q ----[R1]----+----[R2]----+          +
                |            |          |
               [C1]         [C2]       [C3]
                |            |          |
               GND          GND        GND
                
      [Cc] - Coupling to adjacent net
```

## Timing Closure Recommendations

### For Future Tapeouts

1. **Setup Violations (if any):**
   - Reduce logic depth (gate count)
   - Upsize critical path cells
   - Add pipeline stages
   - Optimize placement for shorter wires

2. **Hold Violations (if any):**
   - Add delay buffers
   - Increase wire delay intentionally
   - Use minimum-sized cells on fast paths
   - Adjust CTS skew

3. **Parasitic Optimization:**
   - Minimize long interconnects
   - Use higher metal layers for clocks
   - Shield critical signals
   - Reduce coupling by spacing

4. **Multi-Corner Strategy:**
   - Fix setup at slowest corner (ss_100C_1v60)
   - Fix hold at fastest corner (ff_n40C_1v95)
   - Verify all corners pass

## Power Analysis (Bonus)

### Power Consumption Across Corners

| Corner | Total Power | Dynamic Power | Leakage Power |
|--------|-------------|---------------|---------------|
| ff_n40C_1v95 | 12.34 mW | 11.23 mW | 1.11 mW |
| tt_025C_1v80 | 8.56 mW | 7.89 mW | 0.67 mW |
| ss_100C_1v60 | 4.23 mW | 3.89 mW | 0.34 mW |

**Observation:** Fast corners consume more power due to higher voltage and faster switching.

## Conclusion

### Achievement Summary
✅ **Timing Closure:** All PVT corners meet timing requirements  
✅ **SPEF Integration:** Successfully incorporated parasitic extraction  
✅ **Multi-Corner Analysis:** Verified design across 5 process corners  
✅ **Comparison Complete:** Quantified pre-route vs post-route impact  

### Key Learnings

1. **SPEF is Essential:** Post-route timing analysis requires real parasitics for accuracy. Wire load models underestimate delays by 20-40%.

2. **Parasitics Matter:** Wire delays contributed 27% of critical path delay. Without SPEF, we would have over-optimistic timing estimates.

3. **Corner Coverage:** Analyzing all PVT corners is critical. Setup passes at slow corners, hold passes at fast corners - must verify both.

4. **Degradation is Normal:** 0.2-0.5ns degradation from synthesis to post-route is expected. Larger degradation indicates routing issues.

5. **Silicon Accuracy:** Post-route STA with SPEF achieves ±5% accuracy compared to silicon measurements.

### Design Quality Metrics

| Metric | Target | Achieved | Status |
|--------|--------|----------|--------|
| Setup WNS (worst) | > 0 ns | +0.123 ns | ✅ PASS |
| Hold WNS (worst) | > 0 ns | +0.098 ns | ✅ PASS |
| TNS | 0 ns | 0 ns | ✅ PASS |
| Max Freq | 100 MHz | ~101 MHz | ✅ PASS |
| All Corners Pass | Yes | Yes | ✅ PASS |

### Tape-Out Readiness

**This design is ready for fabrication** based on:
- ✅ Positive slack across all corners
- ✅ Zero timing violations
- ✅ Comprehensive SPEF-based analysis
- ✅ Multi-corner verification complete
- ✅ Power consumption acceptable

### Next Steps for Production

1. **Final Sign-Off Checks:**
   - LVS (Layout vs Schematic) verification
   - DRC (Design Rule Check) - all violations fixed
   - ERC (Electrical Rule Check)
   - Antenna rule violations check

2. **Manufacturing Prep:**
   - Generate final GDSII
   - Add fill cells and metal fill
   - Create chip finishing layers
   - Prepare mask data

3. **Documentation:**
   - Timing constraints document
   - Power delivery network specs
   - Test plan and vectors
   - Datasheet creation

## References

- OpenSTA Documentation: https://github.com/The-OpenROAD-Project/OpenSTA
- SPEF Format Specification: IEEE 1481-1999
- SKY130 PDK Documentation: https://skywater-pdk.readthedocs.io/
- OpenLane Flow Guide: https://openlane.readthedocs.io/

## Appendix A: Commands Summary

### Complete STA Flow
```bash
# Single corner analysis
sta scripts/sta_post_route_typical.tcl

# Multi-corner analysis
sta scripts/sta_all_corners.tcl

# Generate graphs
python3 scripts/generate_timing_graphs.py

# Compare Week 3 vs Week 8
./scripts/compare_week3_week8.sh
```

### Key OpenSTA Commands
```tcl
# Read SPEF (critical!)
read_spef vsdbabysoc.nom.spef

# Setup timing
report_checks -path_delay max

# Hold timing
report_checks -path_delay min

# Get WNS
report_worst_slack

# Get TNS
report_tns

# Full path with SPEF annotation
report_checks -path_delay max \
              -format full_clock_expanded \
              -fields {slew cap input_pin}
```

## Appendix B: File Structure

```
week8_sta/
├── scripts/
│   ├── sta_post_route_typical.tcl
│   ├── sta_pre_route_typical.tcl
│   ├── sta_all_corners.tcl
│   ├── generate_timing_graphs.py
│   ├── extract_results.sh
│   └── compare_week3_week8.sh
├── reports/
│   ├── tt_025C_1v80_setup.rpt
│   ├── tt_025C_1v80_hold.rpt
│   ├── ss_100C_1v60_setup.rpt
│   ├── ff_n40C_1v95_hold.rpt
│   ├── pre_route_tt_setup.rpt
│   └── week8_summary_table.txt
├── graphs/
│   ├── pvt_corner_comparison.png
│   ├── tt_025C_1v80_setup_path.png
│   ├── tt_025C_1v80_hold_path.png
│   ├── ss_100C_1v60_setup_path.png
│   └── ff_n40C_1v95_hold_path.png
├── logs/
│   ├── sta_tt.log
│   └── sta_all_corners.log
└── ANALYSIS.md
```

---

**Report Prepared By:** [Your Name]  
**Date:** November 2025  
**Design:** VSDBabySoC  
**Technology:** SkyWater SKY130 (130nm)  
**Status:** ✅ TIMING CLOSED - READY FOR TAPEOUT
```

**Save and exit**

---

## Step 11: Create GitHub Documentation

### Final Documentation Structure

```bash
cd ~/vlsi_design/week8_sta
mkdir -p Week8_Documentation
cd Week8_Documentation

# Create comprehensive README
nano README.md
```

**README.md Content:**
```markdown
# Week 8: VSDBabySoC Post-Layout STA Report

## 🎯 Overview
Complete post-layout Static Timing Analysis with SPEF annotation across all PVT corners, including comparison with pre-route timing from Week 3.

## 📊 Quick Results

### Timing Summary (Post-Route with SPEF)

| Corner | Setup WNS | Hold WNS | Status |
|--------|-----------|----------|--------|
| ff_n40C_1v95 | 1.234 ns | 0.123 ns | ✅ PASS |
| ff_100C_1v95 | 0.987 ns | 0.098 ns | ✅ PASS |
| tt_025C_1v80 | 0.456 ns | 0.234 ns | ✅ PASS |
| ss_100C_1v60 | 0.123 ns | 0.456 ns | ✅ PASS |
| ss_n40C_1v76 | 0.234 ns | 0.398 ns | ✅ PASS |

**Result:** ✅ All corners meet timing requirements

## 🔍 Key Findings

1. **Average Degradation:** 0.333 ns from pre-route to post-route
2. **Wire Delay Impact:** 27% of critical path delay from interconnect
3. **SPEF Contribution:** Real parasitics added 300-500ps to paths
4. **Worst Corner:** ss_100C_1v60 (slowest, hottest)
5. **Best Corner:** ff_n40C_1v95 (fastest, coldest)

## 📁 Repository Contents

```
Week8_Documentation/
├── README.md                          # This file
├── ANALYSIS.md                        # Detailed analysis report
├── images/
│   ├── pvt_corner_comparison.png      # All corners bar chart
│   ├── tt_setup_path.png              # Typical setup timing
│   ├── ss_setup_path.png              # Worst setup timing
│   └── ff_hold_path.png               # Worst hold timing
├── reports/
│   ├── week3_vs_week8_comparison.txt  # Pre vs Post comparison
│   ├── tt_025C_1v80_setup.rpt         # Detailed setup report
│   ├── ss_100C_1v60_setup.rpt         # Worst corner report
│   └── all_corners_summary.txt        # Summary table
├── scripts/
│   ├── sta_post_route_typical.tcl     # Single corner STA
│   ├── sta_all_corners.tcl            # Multi-corner STA
│   ├── generate_timing_graphs.py      # Graph generation
│   └── compare_week3_week8.sh         # Comparison script
└── data/
    └── spef_sample.txt                # Sample SPEF excerpt
```

## 🚀 How to Reproduce

### 1. Prerequisites
```bash
# OpenSTA installed
# Week 7 completed (SPEF generated)
# Python3 with matplotlib
```

### 2. Run Analysis
```bash
# Navigate to working directory
cd ~/vlsi_design/week8_sta

# Run single corner
sta scripts/sta_post_route_typical.tcl

# Run all corners
sta scripts/sta_all_corners.tcl

# Generate graphs
python3 scripts/generate_timing_graphs.py

# Compare Week 3 vs Week 8
./scripts/compare_week3_week8.sh
```

### 3. View Results
```bash
# View reports
ls -lh reports/

# View graphs
eog graphs/*.png

# Read analysis
cat ANALYSIS.md
```

## 📈 Visual Results

### PVT Corner Comparison
![PVT Corners](images/pvt_corner_comparison.png)

### Critical Path Analysis
![Setup Path](images/ss_setup_path.png)

### Week 3 vs Week 8
![Comparison](images/week3_vs_week8_comparison.png)

## 🔬 Technical Details

### SPEF Impact
- **File Size:** 2.34 MB
- **Nets Annotated:** 1,234
- **Avg Wire R:** 12.5 Ω per net
- **Avg Wire C:** 69.0 fF per net

### Critical Path (Worst Corner)
```
Path: core/CPU_Xreg_value[17][0]/CLK → core/CPU_Xreg_value[8][31]/D
Gates: 15 stages
Gate Delay: 7.234 ns (73%)
Wire Delay: 2.666 ns (27%)
Total: 9.900 ns
Slack: +0.100 ns
```

## ✅ Verification Checklist

- [x] SPEF loaded successfully
- [x] All 5 PVT corners analyzed
- [x] Setup timing met (all corners)
- [x] Hold timing met (all corners)
- [x] Week 3 comparison completed
- [x] Timing graphs generated
- [x] Documentation completed

## 📚 Key Learnings

### Why Timing Degraded
1. **Real Parasitic R&C:** SPEF includes actual layout parasitics
2. **Coupling Capacitance:** Inter-wire coupling slows transitions
3. **Distributed RC:** Wire modeled as RC network, not lumped
4. **Via Resistance:** Vertical connections add delay

### PVT Corner Insights
- **Fast corners:** Higher voltage, lower temp → faster, hold-critical
- **Slow corners:** Lower voltage, higher temp → slower, setup-critical
- **Typical corner:** Nominal conditions, balanced analysis

### SPEF Criticality
- Pre-route accuracy: ~70% (wire load models)
- Post-route accuracy: ~95% (extracted SPEF)
- Silicon correlation: ±5% with SPEF

## 🎓 References

- [OpenSTA Documentation](https://github.com/The-OpenROAD-Project/OpenSTA)
- [SPEF Format](https://en.wikipedia.org/wiki/Standard_Parasitic_Exchange_Format)
- [SKY130 PDK](https://skywater-pdk.readthedocs.io/)
- Week 7: Physical Design & SPEF Generation
- Week 3: Post-Synthesis Timing Analysis

## 👤 Author
**Name:** [Your Name]  
**Date:** November 2025  
**Course:** VLSI Physical Design  
**Task:** Week 8 - Post-Layout STA

---

**Status: ✅ COMPLETE - Design meets timing across all corners and is ready for tape-out**
```

**Save and exit**

---

## Step 12: Copy All Files to Documentation Folder

```bash
cd ~/vlsi_design/week8_sta

# Copy all reports
cp -r reports/ Week8_Documentation/

# Copy all graphs
cp -r graphs/ Week8_Documentation/images/

# Copy scripts
cp -r scripts/ Week8_Documentation/

# Copy analysis document
cp ANALYSIS.md Week8_Documentation/

# Copy sample SPEF (first 1000 lines)
head -1000 ~/vlsi_design/week7/OpenLane/designs/vsdbabysoc/runs/week7_run1/results/routing/vsdbabysoc.nom.spef \
     > Week8_Documentation/data/spef_sample.txt

# Create ZIP for submission
zip -r Week8_VSDBabySoC_STA_Analysis.zip Week8_Documentation/

echo "✅ Documentation package created: Week8_VSDBabySoC_STA_Analysis.zip"
```

---

## Step 13: Push to GitHub

```bash
cd ~/vlsi_design/week8_sta/Week8_Documentation

# Initialize Git (if not already)
git init

# Add all files
git add .

# Commit
git commit -m "Week 8: Complete Post-Layout STA with SPEF Analysis"

# Add remote (replace with your repo)
git remote add origin https://github.com/yourusername/VSDBabySoC_Week8.git

# Push
git push -u origin main
```

---

## 📸 Screenshots Deliverables Checklist

### Required Screenshots:

1. **Terminal Output - SPEF Loading**
   ```bash
   # Capture when running STA
   sta scripts/sta_post_route_typical.tcl
   # Screenshot showing "Reading SPEF file..." message
   ```

2. **Timing Report with SPEF Annotation**
   ```bash
   # Open report and capture
   cat reports/tt_025C_1v80_setup.rpt | less
   # Screenshot showing path with capacitance values
   ```

3. **All Corner Summary Table**
   ```bash
   ./scripts/extract_results.sh
   # Screenshot of the summary table
   ```

4. **PVT Corner Comparison Graph**
   ```bash
   eog graphs/pvt_corner_comparison.png
   # Screenshot of the bar chart
   ```

5. **Critical Path Timing Graph**
   ```bash
   eog graphs/ss_100C_1v60_setup_path.png
   # Screenshot showing worst corner path
   ```

6. **Week 3 vs Week 8 Comparison**
   ```bash
   cat reports/week3_vs_week8_comparison.txt
   # Screenshot of comparison table
   ```

---

## 🎯 Success Criteria Summary

By completing Week 8, you should have:

✅ **Complete STA Analysis**
- Post-route STA with SPEF for all corners
- Timing reports showing SPEF annotation
- All positive slack values

✅ **Multi-Corner Coverage**
- 5 PVT corners analyzed
- Setup timing verified at slow corners
- Hold timing verified at fast corners

✅ **Comparison Complete**
- Week 3 vs Week 8 metrics documented
- Degradation quantified and explained
- Root cause analysis completed

✅ **Visual Documentation**
- Timing path graphs generated
- Corner comparison charts created
- Professional presentation ready

✅ **Understanding Achieved**
- SPEF impact on timing understood
- PVT variations comprehended
- Physical effects on delay learned

---

## 🚀 Bonus: Advanced Analysis

### Clock Tree Analysis
```tcl
# Add to STA script for clock analysis
report_clock_skew
report_clock_latency
```

### Power Analysis with SPEF
```tcl
# Power analysis using SPEF
report_power -hier
```

### Crosstalk Analysis
```tcl
# Check coupling effects
report_noise
report_coupling_caps
```

---

## 📚 Additional Resources

### Understanding SPEF Format
- IEEE Standard 1481-1999
- SPEF format tutorial: [Link]
- Parasitic extraction guides

### PVT Corner Strategy
- Corner selection methodology
- Statistical timing analysis (SSTA)
- Monte Carlo timing analysis

### Timing Closure Techniques
- Path optimization strategies
- Buffer insertion algorithms
- Clock skew optimization

---

**End of Week 8 Guide**

🎉 **Congratulations!** You've completed the full ASIC design flow from RTL to post-layout timing verification!

**Complete Flow:**
- Week 1-2: RTL Design
- Week 3: Synthesis & Initial STA
- Week 4-6: Functional Verification
- Week 7: Physical Design & SPEF
- Week 8: Post-Layout STA ← YOU ARE HERE

**Next**: Tape-out preparation and sign-off checks!
