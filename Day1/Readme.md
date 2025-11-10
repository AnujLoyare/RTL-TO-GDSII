# Week 6 Day 1: Introduction to QFN-48 Package, Chips, Pads, Core, Die and IPs

## Overview
This session covers the fundamentals of QFN-48 (Quad Flat No-lead 48-pin) packaging and introduces key semiconductor concepts including chips, pads, core, die, and intellectual property (IP) blocks.

## Topics Covered
- QFN-48 Package architecture
- Chip structure and components
- Pads and their functionality
- Core design principles
- Die characteristics
- IP blocks integration

## Lab 1: OpenLane Flow Setup and Synthesis

### Prerequisites
- OpenLane installed
- PDK_ROOT environment variable configured
- Docker installed and running

### Steps

#### 1. Navigate to OpenLane Directory
```bash
cd openlane
```

#### 2. Launch OpenLane Docker Container
```bash
sudo docker run -it \
    -v $(pwd):/openLANE_flow \
    -v $PDK_ROOT:$PDK_ROOT \
    -e PDK_ROOT=$PDK_ROOT \
    -u $(id -u $USER):$(id -g $USER) \
    efabless/openlane:rc2 /bin/bash
```

#### 3. Verify Installation
```bash
ls
```

#### 4. Start Interactive Flow
```tcl
./flow.tcl -interactive
```

#### 5. Load OpenLane Package
```tcl
package require openlane 0.9
```

#### 6. Prepare Design
```tcl
prep -design picorv32a
```

#### 7. Run Synthesis
```tcl
run_synthesis
```

## Expected Outcomes
- Successfully set up OpenLane environment
- Complete synthesis of picorv32a design
- Generate synthesis reports and statistics

## Notes
- Ensure all environment variables are properly set before running commands
- Check Docker permissions if you encounter access issues
- Synthesis may take several minutes depending on system resources

## References
- OpenLane Documentation
- QFN Package Specifications
- picorv32a Design Reference
