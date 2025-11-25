# VSDBABySoC - VLSI System Design Baby SoC

## Project Overview

This repository documents the complete journey of designing and implementing VSDBABySoC (VLSI System Design Baby System on Chip) from RTL to GDSII.

---

## Table of Contents

1. [Introduction](#introduction)
2. [Design Specifications](#design-specifications)
3. [Week-wise Progress](#week-wise-progress)
4. [Tools Used](#tools-used)
5. [Results and Observations](#results-and-observations)
6. [Challenges and Solutions](#challenges-and-solutions)
7. [Conclusion](#conclusion)
8. [References](#references)

---

## Introduction

VSDBABySoC is a small RISC-V based System on Chip designed as part of the VLSI System Design course. The SoC integrates:
- RISC-V processor core
- Phase-Locked Loop (PLL)
- Digital-to-Analog Converter (DAC)

This project covers the complete IC design flow from RTL design to final GDSII layout generation.

---

## Design Specifications

### Architecture Components

**RISC-V Core:**
- Instruction Set Architecture: RV32I
- Pipeline stages: [Specify if applicable]
- Clock frequency: [Specify]

**PLL (Phase-Locked Loop):**
- Input frequency: [Specify]
- Output frequency: [Specify]
- Purpose: Clock generation and frequency multiplication

**DAC (Digital-to-Analog Converter):**
- Resolution: [Specify bits]
- Conversion rate: [Specify]

### Design Constraints
- Technology node: [Specify - e.g., 130nm, 180nm]
- Supply voltage: [Specify]
- Target clock frequency: [Specify]

---

## Week-wise Progress

### Week 1: Introduction and Setup
- Understanding VSDBABySoC architecture
- Tool installation and environment setup
- RTL code review and understanding

### Week 2: RTL Simulation
- Functional verification of individual modules
- Integration of RISC-V core, PLL, and DAC
- Testbench development and simulation

### Week 3: Synthesis
- RTL to gate-level netlist conversion
- Logic optimization
- Technology mapping to standard cells
- Synthesis reports analysis (area, timing, power)

### Week 4: Floorplanning
- Die area estimation
- Core and IO placement planning
- Power grid planning
- Pin placement

### Week 5: Placement
- Standard cell placement
- Placement optimization
- Congestion analysis
- Timing-driven placement

### Week 6: Clock Tree Synthesis (CTS)
- Clock tree building
- Clock skew minimization
- Clock buffer insertion
- Post-CTS timing analysis

### Week 7: Routing
- Global routing
- Detailed routing
- Design Rule Check (DRC) violations fixing
- Antenna effect mitigation

### Week 8: Post-Layout Analysis
- Static Timing Analysis (STA)
- Power analysis
- Signal integrity checks
- SPEF extraction and back-annotation

### Week 9: Final Documentation
- Complete flow documentation
- Results compilation
- Final repository preparation

---

## Tools Used

### EDA Tools
- **Synthesis:** Yosys / Design Compiler
- **Simulation:** Icarus Verilog / ModelSim
- **Physical Design:** OpenLane / Innovus / ICC2
- **Static Timing Analysis:** OpenSTA / PrimeTime
- **Layout Viewer:** Magic / KLayout
- **Waveform Viewer:** GTKWave

### Technology Library
- Process: [Specify PDK - e.g., SkyWater 130nm]
- Standard cell library: [Specify]

---

## Results and Observations

### Synthesis Results
- Total cell area: [Specify]
- Cell count: [Specify]
- Critical path delay: [Specify]

### Timing Analysis
- Setup slack: [Specify]
- Hold slack: [Specify]
- Maximum frequency achieved: [Specify]

### Power Analysis
- Total power consumption: [Specify]
- Static power: [Specify]
- Dynamic power: [Specify]

### Physical Design Metrics
- Die area: [Specify]
- Core utilization: [Specify]%
- Routing congestion: [Describe]
- DRC violations: [Specify - should be 0]

---

## Challenges and Solutions

### Challenge 1: [Timing Violations]
**Problem:** [Describe the timing issue encountered]

**Solution:** [Describe how you resolved it - e.g., buffer insertion, cell sizing, etc.]

### Challenge 2: [Routing Congestion]
**Problem:** [Describe congestion issues]

**Solution:** [Describe optimization techniques used]

### Challenge 3: [Power/Ground Network]
**Problem:** [Describe any IR drop or EM issues]

**Solution:** [Describe how power network was optimized]

---

## Unique Contributions

### Custom Scripts/Modifications
[If you created any custom scripts, TCL files, or modified LEF/LIB files, document them here]

**What was changed:**
- [Describe modification]

**Why the change was needed:**
- [Explain the reasoning]

**Impact on design:**
- [Explain how it improved your flow]

---

## Conclusion

This project provided hands-on experience with the complete ASIC design flow. Key learnings include:
- Understanding of RTL to GDSII flow
- Physical design challenges and optimization techniques
- Timing closure methodologies
- Importance of design planning and constraints

The final design successfully meets the specified requirements with [mention key achievements].

---

## Future Improvements

- [Potential optimization 1]
- [Potential optimization 2]
- [Additional features that could be added]

---

## References

1. VSD IAT Course Material
2. OpenLane Documentation: https://openlane.readthedocs.io/
3. SkyWater PDK Documentation: https://skywater-pdk.readthedocs.io/
4. RISC-V Specifications: https://riscv.org/specifications/

---

## Author

**Name:** [Your Name]  
**Institution:** [Your Institution]  
**Course:** VLSI System Design  
**Batch:** [Your Batch]  

---

## Acknowledgments

Thanks to VLSI System Design (VSD) team for the comprehensive course and guidance throughout this project.

---

## Repository Structure

```
vsdbabysoc/
├── README.md
├── rtl/
│   ├── rvmyth.v
│   ├── pll.v
│   ├── dac.v
│   └── vsdbabysoc.v
├── scripts/
│   ├── synthesis.tcl
│   └── sta.tcl
├── constraints/
│   └── timing.sdc
├── reports/
│   ├── synthesis/
│   ├── placement/
│   ├── routing/
│   └── sta/
└── docs/
    └── additional_notes.md
```

---

**Last Updated:** November 2025