# Week 6 Day 2: Floorplanning, Placement and Routing

## Theory: Utilization Factor and Aspect Ratio

### Floorplanning Steps

#### 1. Define Width and Height of Core and Die
- **Core**: Area where the actual logic cells are placed
- **Die**: Total semiconductor area including core and I/O pads
- **Utilization Factor** = (Area occupied by netlist) / (Total core area)
- **Aspect Ratio** = Height / Width

#### 2. Define Location of Preplaced Cells
- Preplaced cells are manually positioned before automated placement
- Examples: Memory blocks, clock gating cells, complex IP blocks
- These cells are fixed and not moved during placement optimization

#### 3. Surround Preplaced Cells with Decoupling Capacitors
- Decoupling capacitors (Decaps) provide local charge reservoir
- Reduces voltage drop and noise in power supply
- Placed close to preplaced cells to ensure stable power delivery

#### 4. Power Planning
- Design power distribution network (PDN)
- Multiple VDD and VSS lines to reduce IR drop
- Power mesh/ring structure for uniform power distribution
- Ground bounce and voltage droop mitigation

#### 5. Pin Placement
- Strategic positioning of I/O pins
- Considers connectivity to reduce wire length
- Clock ports placed equidistant from clock consumers
- Organized to minimize signal routing congestion

#### 6. Logical Cell Placement Blockage
- Define blocked areas where standard cells cannot be placed
- Ensures no overlap with preplaced cells and routing channels
- Maintains proper spacing for power rails

## Lab 2 (Part A): Floorplan Execution

### Run Floorplan
```tcl
run_floorplan
```

### View Floorplan in Magic
```bash
magic -T ~/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef def read picorv32a.floorplan.def &
```

### Floorplan Output Files
- `picorv32a.floorplan.def` - Design Exchange Format file
- Contains die area, core area, and cell positions
- Shows power/ground rails and I/O pin locations

## Theory: Routing and Placement

### Placement Flow

#### 1. Bind Netlist with Physical Cells
- Map logical gates to actual physical cell layouts
- Select cells from standard cell library
- Consider drive strength and timing requirements

#### 2. Placement
- **Global Placement**: Rough positioning of cells
- **Detailed Placement**: Legal placement aligned to rows
- Optimizes wire length and timing

#### 3. Optimize Placement
- **Buffer Insertion**: Add buffers to meet timing constraints
- **Cell Sizing**: Adjust drive strength of cells
- **Timing-driven placement**: Minimize critical path delay
- Reduces congestion and improves routability

## Library Characterization and Modeling

### Standard Cell Library Components
- Timing models (setup/hold time, propagation delay)
- Power models (static and dynamic power)
- Physical models (layout, dimensions)
- Functional models (truth tables, Boolean functions)

## Lab 2 (Part B): Placement Execution

### Run Placement
```tcl
run_placement
```

### View Placement in Magic
```bash
magic -T ~/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef def read picorv32a.placement.def &
```

### Placement Output Files
- `picorv32a.placement.def` - DEF file with placed cells
- Shows detailed cell locations on floorplan
- Ready for clock tree synthesis and routing

## Key Observations
- Check utilization factor (typically 50-70% for good routability)
- Verify aspect ratio for balanced die dimensions
- Ensure no DRC violations in placement
- Review timing slack reports

## Magic Layout Viewer Commands
```
# Zoom in
z

# Zoom out
shift + z

# Select object
s

# View full layout
v

# Query selected object
what
```

## Expected Results
- Completed floorplan with proper die/core dimensions
- Preplaced cells positioned correctly
- Standard cells placed in legal positions
- No overlapping cells or placement violations

## Notes
- Floorplan determines overall chip performance
- Good power planning is critical for chip reliability
- Placement quality directly impacts routing success
- Use Magic for visual verification of each stage

## References
- OpenLane Floorplan Configuration
- Sky130 PDK Documentation
- Magic VLSI Layout Tool Guide
