# Week 6 Day 3: Cell Design Flow and CMOS Fabrication Process

## Theory: Cell Design Flow

### 1. Inputs
- **Process Design Kits (PDKs)**
  - DRC and LVS rules
  - SPICE models
  - Library and user-defined specifications
- **Technology files**
  - Layer information
  - Device models
- **User specifications**
  - Supply voltage
  - Drive strength requirements
  - Pin locations

### 2. Design Steps

#### Circuit Design
- Implement boolean function using transistor networks
- Determine W/L ratios for NMOS and PMOS
- Meet timing and power specifications

#### Layout Design
- Implement circuit function using PMOS and NMOS transistors
- Create stick diagrams and Euler paths
- Draw actual layout using Magic/layout tools
- Follow DRC rules

#### Characterization
- Extract parasitics from layout
- Generate timing, noise, and power models
- Create liberty (.lib) files

### 3. Outputs
- **CDL (Circuit Description Language)** - Netlist
- **GDSII** - Layout file
- **LEF (Library Exchange Format)** - Abstract view
- **SPICE extracted netlist** - With parasitics
- **Timing, noise, and power .lib files**

## Theory: Characterization Flow

### Timing Characterization Parameters

#### 1. Timing Threshold Definitions
- **slew_low_rise_thr**: Typically 20% of VDD (lower threshold for rising edge)
- **slew_high_rise_thr**: Typically 80% of VDD (upper threshold for rising edge)
- **slew_low_fall_thr**: Typically 20% of VDD (lower threshold for falling edge)
- **slew_high_fall_thr**: Typically 80% of VDD (upper threshold for falling edge)
- **in_rise_thr**: Input threshold for rising edge (50% of VDD)
- **in_fall_thr**: Input threshold for falling edge (50% of VDD)
- **out_rise_thr**: Output threshold for rising edge (50% of VDD)
- **out_fall_thr**: Output threshold for falling edge (50% of VDD)

#### 2. Propagation Delay
```
Propagation Delay = Time(out_thr) - Time(in_thr)
```
- **Rise delay**: Time for output to reach 50% when input falls to 50%
- **Fall delay**: Time for output to reach 50% when input rises to 50%
- Should always be positive value
- Negative delay indicates timing violation

#### 3. Transition Time (Slew Rate)
```
Rise Transition Time = Time(slew_high_rise_thr) - Time(slew_low_rise_thr)
Fall Transition Time = Time(slew_high_fall_thr) - Time(slew_low_fall_thr)
```
- Measures how fast signal transitions between logic levels
- Affects power consumption and timing

## Lab 3 (Part A): Advanced Floorplan Configuration

### Modify I/O Pin Mode
```tcl
set ::env(FP_IO_MODE) 2
```

### Run Floorplan with New Configuration
```tcl
run_floorplan
```

### I/O Mode Options
- **Mode 1**: Random equidistant placement
- **Mode 2**: Custom pin configuration

## Theory: SPICE Simulation for VTC

### SPICE Deck Components
1. **Component connectivity** - Netlist description
2. **Component values** - W/L ratios, load capacitance
3. **Identify nodes** - Input, output, supply, ground
4. **Name nodes** - Assign names to all nodes
5. **Simulation commands** - DC sweep, transient analysis
6. **Model descriptions** - Technology parameters

### Model Description
- Define PMOS and NMOS models
- Include technology parameters (VTO, KP, LAMBDA, etc.)
- Specify process corners (TT, FF, SS, FS, SF)

## Lab 3 (Part B): Clone Custom Standard Cell Design

### Clone VSDSTDCELL Repository
```bash
git clone https://github.com/nickson-jose/vsdstdcelldesign.git
```

### Navigate to Directory
```bash
ls
cd vsdstdcelldesign
```

### Repository Contents
- Custom inverter layout
- SPICE netlists
- Sky130 tech files
- Characterization scripts

## Theory: 16-Mask CMOS Fabrication Process

### 1. Selecting a Substrate
- P-type substrate (Boron doped)
- Doping level: ~10^15 cm^-3
- High resistivity: ~5-50 Ω-cm
- Orientation: <100>

### 2. Creating Active Region for Transistors
- **Oxidation**: Grow ~40nm SiO2 on substrate
- **Deposition**: Deposit ~80nm Si3N4 layer
- **Photolithography**: Define active regions using Mask 1
- **Etch**: Remove Si3N4 using plasma etching
- **LOCOS**: Grow field oxide (~200nm) - Local Oxidation of Silicon
- **Strip**: Remove remaining Si3N4

### 3. N-well and P-well Formation
- **Mask 2**: Photoresist for N-well
- **Ion implantation**: Phosphorus (10^13 cm^-2, 200 keV)
- **Mask 3**: Photoresist for P-well
- **Ion implantation**: Boron (10^13 cm^-2, 200 keV)
- **Drive-in diffusion**: High temperature (~1100°C) for deep well formation
- **Twin-tub process**: Forms both wells simultaneously

### 4. Formation of Gate
- **Threshold voltage adjustment**:
  - Mask 4: Boron implant for N-well (PMOS VT control)
  - Mask 5: Arsenic/Phosphorus implant for P-well (NMOS VT control)
- **Gate oxide**: Grow high-quality thin oxide (~10nm)
- **Polysilicon deposition**: ~400nm poly-Si layer
- **Mask 6**: Define gate patterns
- **Gate etch**: Pattern polysilicon gates

### 5. Lightly Doped Drain (LDD) Formation
- **Purpose**: Prevent hot electron effect and short channel effects
- **Mask 7**: P- implant for NMOS (Phosphorus, low dose)
- **Mask 8**: N- implant for PMOS (Boron, low dose)
- **Spacer formation**:
  - Deposit Si3N4 or SiO2 (~100nm)
  - Anisotropic plasma etch to form sidewall spacers

### 6. Source and Drain Formation
- **Mask 9**: N+ implant for NMOS (Arsenic, high dose ~10^15 cm^-2)
- **Mask 10**: P+ implant for PMOS (Boron, high dose)
- **Annealing**: High temperature (~1000°C) to activate dopants and repair damage

### 7. Steps to Form Contacts and Interconnects (Local)
- **Remove oxide**: Etch thin screen oxide using HF solution
- **Deposit Ti**: Titanium sputtering for low-resistance contacts
- **Mask 11**: Define contact regions
- **RCA cleaning**: Remove organic and ionic contaminants
- **TiN deposition**: Barrier layer to prevent metal diffusion
- **Blanket tungsten**: CVD tungsten deposition to fill contacts
- **CMP (Chemical Mechanical Polishing)**: Planarize surface

### 8. Higher Level Metal Formation
- **Mask 12**: First metal layer (Aluminum/Copper)
  - Deposit metal (~500nm)
  - Pattern and etch
- **ILD (Inter-Layer Dielectric)**: Deposit SiO2
- **CMP**: Planarize
- **Mask 13**: Via 1 formation
- **Mask 14**: Second metal layer
- **Mask 15**: Via 2 formation
- **Mask 16**: Top metal layer and pad formation
- **Passivation**: Final protective layer (Si3N4)

### Summary of 16 Masks
1. Active region definition
2. N-well formation
3. P-well formation
4. N-well VT adjust (PMOS)
5. P-well VT adjust (NMOS)
6. Poly gate definition
7. LDD N-type (NMOS)
8. LDD P-type (PMOS)
9. N+ Source/Drain (NMOS)
10. P+ Source/Drain (PMOS)
11. Contact holes
12. Metal 1 layer
13. Via 1
14. Metal 2 layer
15. Via 2
16. Metal 3/Pad layer

## Lab 3 (Part C): Custom Cell Characterization

### Commands (To be updated)
```bash
# Additional commands will be added here for:
# - Opening custom inverter layout in Magic
# - Extracting SPICE netlist
# - Running SPICE simulations
# - Characterizing timing parameters
```

## Key Takeaways
- Cell design requires careful consideration of timing, power, and area
- SPICE simulation validates electrical characteristics
- 16-mask CMOS process enables complex IC fabrication
- LDD formation is critical for reliability
- Multiple metal layers enable complex routing

## Notes
- Standard cell height must be multiple of track pitch
- Width must be odd multiples of track pitch
- Input/output ports must align with routing tracks
- Power and ground rails must match standard cell specifications

## References
- CMOS VLSI Design by Weste and Harris
- Sky130 PDK Process Documentation
- OpenLane Cell Design Guidelines
- SPICE Simulation Manual
