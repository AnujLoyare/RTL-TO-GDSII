# Day 4: Static Behavior Evaluation - CMOS Inverter Robustness

## Table of Contents
1. [Introduction](#introduction)
2. [Static Behavior Evaluation](#static-behavior-evaluation)
3. [CMOS Inverter Robustness](#cmos-inverter-robustness)
4. [Lab Session](#lab-session)
5. [Conclusions](#conclusions)

---

## Introduction

Day 4 focuses on **static behavior evaluation** of the CMOS inverter, specifically analyzing robustness through two critical parameters:

1. **Switching Threshold (Vm)**
2. **Noise Margins (NMH & NML)**

---

## Static Behavior Evaluation

### What is Static Behavior?

Static behavior refers to the DC (steady-state) characteristics of a circuit when inputs are held constant. Key parameters include:
- Voltage Transfer Characteristic (VTC)
- Logic thresholds
- Noise margins
- DC operating points

---

## CMOS Inverter Robustness

### 1. Switching Threshold (Vm)

**Definition:**
The switching threshold (Vm) is the input voltage at which Vout = Vin.

**Characteristics:**
- At Vm, both NMOS and PMOS are in saturation
- Both transistors conduct equal currents
- Maximum gain point on VTC curve
- Ideally located near VDD/2 for balanced operation

**Importance:**
- Determines logic level transition point
- Affects noise margins symmetry
- Impacts propagation delays
- Critical for reliable digital operation

---

### 2. Noise Margins (NMH & NML)

**Definition:**
Noise margin is the maximum noise voltage that can be tolerated without causing logic errors.

#### **NML (Noise Margin Low)**
```
NML = VIL - VOL
```

Where:
- **VIL**: Maximum input voltage recognized as LOW
- **VOL**: Minimum output voltage when input is HIGH

**Measures immunity to noise when input is LOW**

---

#### **NMH (Noise Margin High)**
```
NMH = VOH - VIH
```

Where:
- **VOH**: Maximum output voltage when input is LOW  
- **VIH**: Minimum input voltage recognized as HIGH

**Measures immunity to noise when input is HIGH**

---

### Robustness Metrics

**Good Noise Margins:**
- NM > 30% of VDD → Acceptable
- NM > 40% of VDD → Excellent
- NM < 20% of VDD → Poor (redesign needed)

**Robustness Factors:**
- Device sizing (Wp/Wn ratio)
- Supply voltage (VDD)
- Process variations (PVT corners)
- Temperature effects

---

## Lab Session

### Experiment: Noise Margin Analysis

**Objective:** 
Analyze CMOS inverter robustness by extracting switching threshold and noise margins from VTC.

---

### Step 1: Navigate to Design Directory

```bash
# 1. Change to your circuit design directory
cd ~/sky130_circuit_design_workshop/design
```

---

### Step 2: Verify Netlist

```bash
# 2. List the files to confirm the netlist is there
ls
```

---

### Step 3: Open Netlist

```bash
gvim day4_inv_noisemargin_wp1_wn036.spice
```

---

### Step 4: Run Simulation

```bash
# 3. Run the simulation using ngspice
ngspice day4_inv_noisemargin_wp1_wn036.spice
```

---

### Step 5: Plot VTC

```bash
plot out vs in
```

---

## Conclusions

### Key Findings from Day 4:

1. **Static Behavior Evaluation:**
   - Analyzed DC characteristics through VTC
   - Extracted critical voltage levels (VOH, VOL, VIH, VIL)
   - Measured switching threshold (Vm)

2. **Switching Threshold (Vm):**
   - Identified the point where Vout = Vin
   - Observed its location relative to VDD/2
   - Understood its impact on circuit robustness

3. **Noise Margins:**
   - Calculated NML = VIL - VOL
   - Calculated NMH = VOH - VIH
   - Evaluated noise immunity for both logic levels

4. **CMOS Inverter Robustness:**
   - Device sizing affects Vm and noise margins
   - Trade-off between NMH and NML based on Wp/Wn ratio
   - Robustness ensures reliable operation under PVT variations

5. **Connection to STA:**
   - Noise margins impact timing margins
   - Switching threshold affects delay calculations
   - Robustness parameters critical for timing closure
   - Must verify across all process corners

---

**Author**: [Anuj Loyare]  
**Date**: October 19, 2025  
**Workshop**: SKY130 CMOS Circuit Design - Day 4
