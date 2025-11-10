# Week 6 Day 5: Routing Algorithms and Physical Design Completion

## Theory: Introduction to Maze Routing - Lee's Algorithm

### Overview
Lee's Algorithm (also known as Lee-Moore Algorithm) is a grid-based maze routing algorithm that guarantees finding the shortest path between two points if a path exists.

### Routing Fundamentals

#### What is Routing?
- **Purpose**: Create physical connections between pins
- **Goal**: Minimize wire length, vias, and congestion
- **Constraint**: Obey DRC rules

#### Routing Types
1. **Global Routing**: Coarse routing, defines general paths
2. **Detailed Routing**: Fine routing, actual wire tracks
3. **Track Assignment**: Assign nets to specific metal layers and tracks

### Lee's Algorithm Steps

#### Step 1: Grid Initialization
```
Source (S) ──────► Target (T)

Grid Representation:
┌─┬─┬─┬─┬─┐
│ │ │ │ │ │
├─┼─┼─┼─┼─┤
│ │S│ │ │ │
├─┼─┼─┼─┼─┤
│ │ │ │T│ │
└─┴─┴─┴─┴─┘
```

#### Step 2: Wave Expansion
- Label source as 0
- Label adjacent cells as 1
- Continue wave expansion until target reached
```
Wave 0:    Wave 1:    Wave 2:    Wave 3:
┌─┬─┬─┐    ┌─┬─┬─┐    ┌─┬─┬─┐    ┌─┬─┬─┐
│ │ │ │    │ │1│ │    │2│1│2│    │2│1│2│
├─┼─┼─┤    ├─┼─┼─┤    ├─┼─┼─┤    ├─┼─┼─┤
│ │0│ │ →  │1│0│1│ →  │1│0│1│ →  │1│0│1│
├─┼─┼─┤    ├─┼─┼─┤    ├─┼─┼─┤    ├─┼─┼─┤
│ │ │T│    │ │1│T│    │2│1│T│    │2│1│2│
└─┴─┴─┘    └─┴─┴─┘    └─┴─┴─┘    └─┴─┴─┘
Target reached at Wave 2!
```

#### Step 3: Backtrace
- Start from target
- Move to neighbor with label (n-1)
- Continue until reaching source
```
Path Trace:
┌─┬─┬─┐
│ │1│ │
├─┼─┼─┤
│1│0│1│
├─┼─┼─┤    Path: S → → ↓ → T
│ │1│T│
└─┴─┴─┘
```

### Advantages of Lee's Algorithm
1. **Guaranteed solution**: Always finds path if exists
2. **Shortest path**: Finds minimum Manhattan distance
3. **Simple implementation**: Easy to understand and code
4. **Complete coverage**: Explores all possibilities

### Disadvantages of Lee's Algorithm
1. **Memory intensive**: Requires large grid storage
2. **Time consuming**: Slow for large designs
3. **No layer consideration**: Single layer algorithm
4. **Sequential processing**: Routes one net at a time

### Algorithm Complexity
```
Time Complexity: O(n × m)
Space Complexity: O(n × m)

where n × m = grid dimensions
```

### Lee's Algorithm Implementation
```python
def lee_routing(grid, source, target):
    # Wave expansion
    queue = [(source, 0)]
    visited = {source: 0}
    
    while queue:
        (x, y), dist = queue.pop(0)
        
        if (x, y) == target:
            return backtrace(visited, source, target)
        
        # Check 4 neighbors
        for dx, dy in [(0,1), (1,0), (0,-1), (-1,0)]:
            nx, ny = x + dx, y + dy
            
            if is_valid(nx, ny) and (nx, ny) not in visited:
                visited[(nx, ny)] = dist + 1
                queue.append(((nx, ny), dist + 1))
    
    return None  # No path found
```

## Theory: Design Rule Check (DRC)

### What is DRC?
Design Rule Check verifies that the physical layout meets manufacturing constraints defined by the foundry.

### Common DRC Rules

#### 1. Minimum Width Rule
```
Metal Width ≥ Minimum Width

Violation:           Correct:
   ▄▄                  ████
   ▄▄  (too thin)      ████
   ▄▄                  ████
```

#### 2. Minimum Spacing Rule
```
Spacing between metals ≥ Minimum Spacing

Violation:           Correct:
████ ████            ████    ████
████ ████  (close)   ████    ████
```

#### 3. Minimum Pitch Rule
```
Pitch = Width + Spacing

Pitch ≥ Minimum Pitch

├──Width──┤
████████████
           ├─Spacing─┤
                     ████████████
├────────Pitch────────┤
```

#### 4. Via Rules
```
- Via must be completely inside metal
- Via spacing rules
- Via enclosure rules

Violation:           Correct:
████                 ████████
 ▓▓  (via outside)    ▓▓▓▓
████                 ████████
```

#### 5. Minimum Area Rule
```
Metal Area ≥ Minimum Area

Violation:           Correct:
██                   ████████
██  (small area)     ████████
```

### DRC Categories

#### 1. Intra-layer DRC
- Rules within same metal layer
- Width, spacing, area checks

#### 2. Inter-layer DRC
- Rules between different layers
- Via enclosure, overlap checks

#### 3. Antenna Rules
- Prevents charge accumulation during fabrication
- Limits metal area connected to gate
```
Antenna Ratio = Metal Area / Gate Area
Must be < Maximum Antenna Ratio
```

### Common DRC Violations

#### Short Circuit
```
Two different nets touching:

Net1: ████████
            ████████ :Net2
         ↑
      Short!
```

#### Spacing Violation
```
Insufficient spacing:

████████  ████████
       ↑↑
    < Min Spacing
```

#### Width Violation
```
Wire too narrow:

████████
██  ← Too thin
████████
```

#### Enclosure Violation
```
Via not properly enclosed:

████████
  ▓▓    ← Via extends beyond metal
████████
```

## Lab 5 (Part A): Pre-Routing Setup

### Commands (To be updated)
```tcl
# Commands for pre-routing configuration will be added here
# - Setting routing layers
# - Defining routing constraints
# - Configuring routing strategy
```

### Routing Configuration Parameters
```tcl
# Routing layers
set ::env(RT_MIN_LAYER) met1
set ::env(RT_MAX_LAYER) met5

# Routing strategy
set ::env(ROUTING_STRATEGY) 0

# Global routing adjustments
set ::env(GLB_RT_ADJUSTMENT) 0.15
```

## Theory: TritonRoute - Detailed Router

### Overview
TritonRoute is an open-source detailed router used in OpenLane that performs track assignment and generates final routing.

### TritonRoute Features

#### 1. Honors Pre-routed Constraints
- Respects power/ground networks
- Avoids macro placements
- Maintains preplaced cell connections

#### 2. Routing Strategy
- **Intra-layer routing**: Parallel routing within layer
- **Inter-layer routing**: Via-based layer transitions
- **MILP-based optimization**: Mixed Integer Linear Programming

#### 3. Connectivity Access Points
- **Access Point (AP)**: On-grid point where router connects to pins
- **Access Point Cluster (APC)**: Group of APs for same net

### TritonRoute Algorithm Flow
```
┌─────────────────────────┐
│   Initial Route Search  │
│   (Lee-based algorithm) │
└───────────┬─────────────┘
            │
            ▼
┌─────────────────────────┐
│   Track Assignment      │
│   (Assign to routing    │
│    tracks)              │
└───────────┬─────────────┘
            │
            ▼
┌─────────────────────────┐
│   Detailed Routing      │
│   (Generate actual      │
│    geometries)          │
└───────────┬─────────────┘
            │
            ▼
┌─────────────────────────┐
│   DRC Fixing            │
│   (Resolve violations)  │
└─────────────────────────┘
```

### TritonRoute Methodology

#### Panel Routing
- Divides chip into panels
- Routes each panel independently
- Stitches panels together
```
┌─────┬─────┬─────┐
│Panel│Panel│Panel│
│  1  │  2  │  3  │
├─────┼─────┼─────┤
│Panel│Panel│Panel│
│  4  │  5  │  6  │
└─────┴─────┴─────┘
```

#### Routing Layers

##### Preferred Direction
- **M1 (Metal 1)**: Horizontal preferred
- **M2 (Metal 2)**: Vertical preferred
- **M3 (Metal 3)**: Horizontal preferred
- **M4 (Metal 4)**: Vertical preferred
- **M5+ (Higher metals)**: Alternate H/V
```
M3: ═══════════  (Horizontal)
        ║
M2:     ║        (Vertical)
        ║
M1: ═══════════  (Horizontal)
```

### Routing Constraints

#### 1. Track Specifications
```
Layer M1:
- Pitch: 0.46 µm
- Width: 0.14 µm
- Spacing: 0.32 µm
- Offset: 0.23 µm
```

#### 2. Via Specifications
```
Via12 (M1 to M2):
- Size: 0.15 µm × 0.15 µm
- Enclosure M1: 0.055 µm
- Enclosure M2: 0.055 µm
- Spacing: 0.17 µm
```

## Theory: Routing Topology Algorithms

### 1. Steiner Tree Algorithm

#### Concept
- Connects multiple pins with minimum total wire length
- Allows branching points (Steiner points) not at pins
```
Spanning Tree:        Steiner Tree:
    A                     A
    │                     │
    B                     S ← Steiner point
    │                    ╱ ╲
    C                   B   C

Length: 2L           Length: 1.4L (shorter!)
```

#### Types
- **Minimum Spanning Tree (MST)**: No additional points
- **Rectilinear Steiner Tree (RST)**: Manhattan geometry
- **Minimum Steiner Tree (MST)**: Optimal but NP-hard

### 2. A* Algorithm

#### Features
- Heuristic-based pathfinding
- Faster than Lee's for point-to-point routing
- Uses cost function: f(n) = g(n) + h(n)
```
f(n) = Total cost
g(n) = Cost from start to n
h(n) = Heuristic estimate from n to goal
```

#### Advantages over Lee's
- **Directed search**: Explores promising paths first
- **Faster**: Reduces search space
- **Flexible**: Can incorporate various costs

### 3. Maze Runner with Cost

#### Enhanced Lee's Algorithm
```
Cost = Base Distance + Bend Cost + Via Cost + Congestion Cost

Example costs:
- Unit distance: 1
- Bend: +2
- Via: +5
- Congested region: +3
```

### 4. Pattern Routing

#### Concept
- Uses predefined routing patterns
- Fast for regular structures
- Common in memory arrays
```
Standard Cell Pattern:
VDD ═══════════════════
     │  │  │  │  │  │
Cell ├──┼──┼──┼──┼──┤
     │  │  │  │  │  │
VSS ═══════════════════
```

## Lab 5 (Part B): Detailed Routing Execution

### Run Global Routing
```tcl
run_routing
```

### TritonRoute Execution
```bash
# TritonRoute automatically invoked by run_routing
# Reads:
# - LEF files (technology and cells)
# - DEF file (placed and CTS design)
# - Guides file (from global router)

# Generates:
# - Routed DEF file
# - DRC report
# - Wire length report
```

### View Routing in Magic
```bash
magic -T ~/pdks/sky130A/libs.tech/magic/sky130A.tech \
      lef read ../../tmp/merged.lef \
      def read picorv32a.def &
```

### Magic Commands for Routing Verification
```tcl
# In Magic console

# Select a net
select net <net_name>

# Highlight selected net
highlight

# Check DRC
drc check

# Show DRC errors
drc why

# Get detailed info
what

# Measure distance
box

# Zoom to fit
zoom fit
```

### Routing Reports

#### Wire Length Report
```
Layer          Wire Length (µm)    Vias
------------------------------------------------
met1           125,432             15,234
met2           98,765              12,456
met3           45,678              5,678
met4           23,456              2,345
met5           12,345              1,234
------------------------------------------------
Total          305,676             36,947
```

#### DRC Violations Report
```
Violation Type              Count    Layer
------------------------------------------------
Minimum Width              0         -
Minimum Spacing            2         met2
Via Enclosure             1         via2
Short                     0         -
Antenna                   3         met3
------------------------------------------------
Total Violations: 6
```

### Post-Routing Optimization

#### Fix DRC Violations
```tcl
# Detailed routing with optimization
set ::env(ROUTING_OPT_ITERS) 15

# Re-run routing
run_routing
```

#### Timing-Driven Routing
```tcl
# Enable timing optimization
set ::env(GLB_RT_TIMING_DRIVEN) 1

# Set timing weights
set ::env(GLB_RT_EST_CONGESTION) 0
```

## Routing Metrics and Quality Checks

### 1. Wire Length
```
Total Wire Length = Σ (length of all nets)
Target: Minimize while meeting timing
```

### 2. Via Count
```
Via Count = Number of layer transitions
Target: Minimize for reliability
```

### 3. Congestion
```
Congestion = (Used tracks / Available tracks)
Target: < 80% for successful routing
```

### 4. DRC Violations
```
Target: Zero violations
Must fix before tapeout
```

### 5. Antenna Violations
```
Antenna Ratio = (Metal area during fab) / (Gate area)
Fix: Add diodes or route splitting
```

## Complete OpenLane Routing Flow
```tcl
# Start from placed and CTS design

# 1. Generate power distribution network (if not done)
gen_pdn

# 2. Run global routing
run_routing

# 3. Check for violations
check_design_violations

# 4. Optimization iterations (if needed)
set ::env(ROUTING_OPT_ITERS) 10
run_routing

# 5. Generate final reports
run_magic
run_magic_spice_export
run_magic_drc
run_lvs
run_antenna_check
```

## Troubleshooting Common Issues

### Issue 1: Routing Congestion
**Symptoms**: Router cannot complete all connections

**Solutions**:
```tcl
# Reduce utilization
set ::env(FP_CORE_UTIL) 40

# Increase routing layers
set ::env(RT_MAX_LAYER) met5

# Adjust global routing
set ::env(GLB_RT_ADJUSTMENT) 0.2
```

### Issue 2: Antenna Violations
**Symptoms**: Antenna ratio exceeds limits

**Solutions**:
```tcl
# Enable antenna diode insertion
set ::env(DIODE_INSERTION_STRATEGY) 3

# Run antenna check
run_antenna_check
```

### Issue 3: DRC Violations
**Symptoms**: Design rule violations after routing

**Solutions**:
```tcl
# Increase routing iterations
set ::env(ROUTING_OPT_ITERS) 20

# Enable detailed DRC fixing
run_magic_drc
```

## Key Takeaways
- Lee's algorithm guarantees shortest path but is memory/time intensive
- DRC violations must be resolved before fabrication
- TritonRoute handles detailed routing with multiple optimization passes
- Routing topology affects wire length, timing, and power
- Multiple routing strategies exist for different design requirements
- Post-routing verification is critical for tapeout

## Important Metrics
- **Wire length**: Target minimum for better performance
- **Via count**: Fewer vias = better reliability
- **DRC violations**: Must be zero
- **Antenna ratio**: Must meet foundry limits
- **Congestion**: Keep below 80% for successful routing

## Notes
- Always verify routing in multiple corners
- Check both setup and hold timing after routing
- Review DRC reports carefully
- Antenna violations can cause yield loss
- Use Magic for visual inspection of routes

## References
- Lee's Maze Routing Algorithm - Classic Paper
- TritonRoute Documentation
- Sky130 PDK DRC Manual
- OpenLane Routing Guide
- VLSI Physical Design Automation
