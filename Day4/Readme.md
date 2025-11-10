# Week 6 Day 4: Clock Tree Synthesis and Timing Analysis

## Lab 4 (Part A): Pre-CTS Setup

### Commands (To be updated)
```bash
# Commands for pre-CTS configuration will be added here
# - LEF/DEF modifications
# - Custom cell integration
# - Timing constraints setup
```

## Theory: Power Aware Clock Tree Synthesis (CTS)

### Overview
Power-aware CTS aims to minimize power consumption while meeting timing requirements in clock distribution networks.

### Key Concepts

#### 1. Clock Power Components
- **Dynamic Power**: Switching power of clock buffers and nets
```
  P_dynamic = α × C × V² × f
```
  - α: Switching activity factor
  - C: Load capacitance
  - V: Supply voltage
  - f: Clock frequency

- **Short Circuit Power**: During transitions
- **Leakage Power**: Static power in clock buffers

#### 2. Power Optimization Techniques

##### Clock Gating
- **Purpose**: Disable clock to inactive logic blocks
- **Types**:
  - Latch-based clock gating
  - Flip-flop-based clock gating
- **Benefits**: Reduces dynamic power by 20-70%

##### Multi-Vt Cell Usage
- **HVT (High Vt)**: Low leakage, slower (for non-critical paths)
- **SVT (Standard Vt)**: Balanced performance and leakage
- **LVT (Low Vt)**: High speed, high leakage (for critical paths)

##### Buffer Sizing
- Right-size buffers based on fanout
- Avoid over-buffering
- Use minimum drive strength meeting timing

##### Logic Restructuring
- Combine clock domains where possible
- Reduce clock tree depth
- Minimize wire length

### 3. Power-Aware CTS Constraints
```tcl
# Maximum transition time
set_max_transition 0.5 [get_clocks CLK]

# Maximum capacitance
set_max_capacitance 0.3 [get_ports CLK]

# Clock gating setup
set_clock_gating_style -sequential_cell latch \
                       -minimum_bitwidth 3 \
                       -control_point before

# Power optimization
set_clock_tree_options -target_skew 0.3 \
                       -target_early_delay 0.2
```

## Lab 4 (Part B): CTS Execution

### Commands (To be updated)
```tcl
# Commands for running CTS will be added here
# run_cts
# CTS report generation
# Post-CTS timing analysis
```

## Theory: Clock Tree Routing and Buffering using H-Tree Algorithm

### H-Tree Algorithm

#### Characteristics
- **Hierarchical structure**: Resembles letter 'H'
- **Symmetric distribution**: Equal path length to all endpoints
- **Recursive partitioning**: Divides clock domain into quadrants

#### Advantages
1. **Zero Skew**: Equal wire length to all sinks
2. **Predictable**: Simple geometric structure
3. **Low power**: Minimizes total wire length
4. **Scalable**: Works for large designs

#### Limitations
1. **Blockage issues**: Difficult with placement obstacles
2. **Routing congestion**: May create hotspots
3. **Non-uniform loading**: Real designs have varying loads
4. **Limited flexibility**: Rigid structure

### H-Tree Construction Process

#### Step 1: Find the Center
- Calculate geometric center of all clock sinks
- Place root buffer at center point

#### Step 2: First Level Division
```
                |
    -----+-----+-----+-----
         |           |
```
- Divide region into 4 quadrants
- Route from center to quadrant centers

#### Step 3: Recursive Division
```
      |       |       |       |
   ---+---  --+--  ---+---  --+--
      |       |       |       |
```
- Continue H-pattern at each level
- Stop when reaching individual flip-flops

#### Step 4: Buffer Insertion
- Insert buffers at each branch point
- Maintain equal delay on all branches
- Size buffers based on fanout

### Buffer Insertion Strategy

#### Purpose of Buffers
1. **Drive strength**: Overcome wire resistance
2. **Transition time**: Maintain sharp edges
3. **Skew control**: Balance delays
4. **Load isolation**: Separate clock domains

#### Buffer Placement Rules
```
Max Distance = √(R_wire × C_wire × t_max)
```
- R_wire: Wire resistance per unit length
- C_wire: Wire capacitance per unit length
- t_max: Maximum allowable transition time

## Theory: Clock Tree Synthesis - Problems and Solutions

### Problem 1: Clock Skew

#### Issue
- Different arrival times at flip-flops
- Causes setup/hold violations
- Limits maximum frequency

#### Solution: H-Tree with Repeaters
```
Source Buffer → Level 1 Buffers → Level 2 Buffers → Sinks
     |               |                 |
  Equal path length ensures zero skew
```

**Implementation:**
- Insert repeaters at regular intervals
- Match delays on all branches
- Use dummy loads for balancing

### Problem 2: Clock Latency

#### Issue
- Long delay from source to sinks
- Reduces effective cycle time
- Impacts performance

#### Solution: Strategic Buffer Placement
- **Minimize levels**: Reduce tree depth
- **Increase drive strength**: Use larger buffers
- **Reduce wire length**: Optimize topology
- **Use low-Vt cells**: For clock path

### Problem 3: Signal Integrity

#### Issue
- Crosstalk from neighboring nets
- Voltage drop on clock lines
- Transition time degradation

#### Solution: Repeater Insertion
```
Long Wire → [Buffer] → Segment → [Buffer] → Segment → Sink
```
**Benefits:**
- Regenerates signal strength
- Reduces transition time
- Isolates noise
- Improves slew rate

### Problem 4: Power Consumption

#### Issue
- Clock network consumes 30-40% of total power
- Always switching at full frequency
- Large buffers draw significant current

#### Solution: Power-Aware Techniques
```tcl
# Use clock gating
insert_clock_gating -global

# Multi-Vt optimization
set_clock_tree_references -library slow_HVT.lib

# Gate sizing
optimize_clock_tree -power
```

### Problem 5: Process Variation

#### Issue
- Manufacturing variations cause delay mismatches
- OCV (On-Chip Variation) affects skew
- Corner analysis needed

#### Solution: Useful Skew and Margin
```
Setup Check: T_clk ≥ T_logic + T_setup + T_skew + T_margin
Hold Check:  T_hold ≤ T_skew - T_cq - T_logic - T_margin
```
- **Useful skew**: Intentional skew to improve timing
- **Guard-banding**: Add margin for variation
- **Derating factors**: Account for OCV

## Lab 4 (Part C): Post-CTS Optimization

### Commands (To be updated)
```tcl
# Commands for post-CTS optimization will be added here
# - Fixing timing violations
# - Buffer optimization
# - Clock skew analysis
```

## Theory: Timing Analysis with Real Clocks

### Ideal vs Real Clock

#### Ideal Clock
- Zero skew
- Zero latency
- Perfect duty cycle
- No jitter

#### Real Clock
- Non-zero skew
- Finite latency
- Duty cycle variation
- Jitter and noise

### Setup Timing with Real Clock
```
Launch FF         Capture FF
    |                 |
    CLK1             CLK2
    |                 |
    +--[Logic Path]---+
```

#### Setup Equation
```
T_clk + T_skew ≥ T_cq + T_logic + T_setup + T_uncertainty
```

Where:
- **T_clk**: Clock period
- **T_skew**: Clock skew (can be positive or negative)
- **T_cq**: Clock-to-Q delay of launch flip-flop
- **T_logic**: Combinational logic delay
- **T_setup**: Setup time of capture flip-flop
- **T_uncertainty**: Clock jitter + margin

#### With Real Clock (Pessimistic)
```
T_clk + min(skew) ≥ T_cq + T_logic + T_setup + T_uncertainty + max(skew)
```

### Hold Timing with Real Clock

#### Hold Equation
```
T_cq + T_logic ≥ T_hold + |T_skew| - T_uncertainty
```

#### With Real Clock (Pessimistic)
```
T_cq + T_logic ≥ T_hold + max(|skew|) + T_uncertainty
```

### Clock Uncertainty Components

#### 1. Jitter
- **Cycle-to-cycle jitter**: Variation between adjacent cycles
- **Period jitter**: Variation from ideal period
- **Phase jitter**: Long-term phase drift
```tcl
set_clock_uncertainty -setup 0.5 [get_clocks CLK]
set_clock_uncertainty -hold 0.3 [get_clocks CLK]
```

#### 2. Clock Skew
- **Useful skew**: Helps setup, hurts hold
- **Harmful skew**: Hurts both setup and hold
```
Positive Skew: Capture clock arrives late
Negative Skew: Capture clock arrives early
```

### Multi-Corner Multi-Mode (MCMM) Analysis

#### Process Corners
- **SS (Slow-Slow)**: Worst setup, best hold
- **FF (Fast-Fast)**: Worst hold, best setup
- **TT (Typical-Typical)**: Nominal
- **SF (Slow-Fast)**: Mixed
- **FS (Fast-Slow)**: Mixed

#### Voltage and Temperature
```
Setup Critical: SS corner, Low voltage, High temp
Hold Critical:  FF corner, High voltage, Low temp
```

### Common Path Pessimism Removal (CPPR)

#### Concept
- Clock paths share common buffers
- Same variation affects both launch and capture
- Should not double-count

#### CPPR Credit
```
CPPR = Common clock path delay variation
Actual Skew = Calculated Skew - CPPR
```
```tcl
set_timing_derate -early 0.95
set_timing_derate -late 1.05
set_timing_derate -clock -early 0.97
set_timing_derate -clock -late 1.03
```

## Lab 4 (Part D): Timing Analysis and Reports

### Commands (To be updated)
```tcl
# Commands for timing analysis will be added here
# report_timing -path_type full_clock
# report_clock_timing -type skew
# report_clock_timing -type latency
```

### Expected Timing Reports

#### Clock Skew Report
```
Clock: CLK
Period: 10.000
Latency: 2.500

Sink               Latency    Skew
----------------------------------------
FF1/CLK            2.450     -0.050
FF2/CLK            2.500      0.000
FF3/CLK            2.550      0.050
----------------------------------------
Max Skew: 0.100
```

#### Setup Timing Report
```
Startpoint: FF1 (rising edge-triggered flip-flop clocked by CLK)
Endpoint: FF2 (rising edge-triggered flip-flop clocked by CLK)
Path Group: CLK
Path Type: max

Point                          Incr       Path
--------------------------------------------------------
clock CLK (rise edge)         0.000      0.000
clock network delay (ideal)   2.450      2.450
FF1/CLK (FF_X1)              0.000      2.450 r
FF1/Q (FF_X1)                0.850      3.300 r
logic_path                    4.200      7.500 r
FF2/D (FF_X1)                0.000      7.500 r
data arrival time                        7.500

clock CLK (rise edge)        10.000     10.000
clock network delay           2.500     12.500
FF2/CLK (FF_X1)              0.000     12.500 r
clock uncertainty            -0.500     12.000
FF2 (setup)                  -0.200     11.800
data required time                      11.800
--------------------------------------------------------
slack (MET)                              4.300
```

## Key Performance Metrics

### Clock Tree Quality
- **Skew**: < 5% of clock period (target)
- **Latency**: < 20% of clock period
- **Transition time**: < 10% of clock period
- **Max capacitance**: Per library limits
- **Max fanout**: Typically 16-32

### Power Metrics
```
Clock Power = Number of Buffers × Buffer Power + Wire Power
Target: < 30% of total power
```

## Useful TCL Commands
```tcl
# Report clock tree
report_clock_tree -summary

# Report clock skew
report_clock_timing -type skew

# Report clock latency
report_clock_timing -type latency

# Report timing with clock
report_timing -path_type full_clock \
              -delay max \
              -max_paths 10

# Check hold timing
report_timing -path_type full_clock \
              -delay min \
              -max_paths 10
```

## Key Takeaways
- Real clocks have skew, latency, and uncertainty
- H-tree provides symmetric zero-skew distribution
- Buffer insertion critical for signal integrity
- Power-aware CTS reduces clock network power
- MCMM analysis ensures design works across corners
- CPPR removes pessimism in timing analysis

## Notes
- Always verify CTS results in multiple corners
- Check both setup and hold timing post-CTS
- Monitor transition times and capacitance violations
- Use clock gating for power reduction
- Consider useful skew for timing optimization

## References
- Clock Tree Synthesis for VLSI Circuits
- OpenLane CTS Documentation
- Sky130 Clock Tree Guidelines
- Timing Analysis Fundamentals
