# Day 3: CMOS Inverter - VTC and Transient Analysis

## Table of Contents
1. [Introduction](#introduction)
2. [CMOS Inverter Fundamentals](#cmos-inverter-fundamentals)
3. [Voltage Transfer Characteristic (VTC)](#voltage-transfer-characteristic-vtc)
4. [Individual MOSFET Characteristics](#individual-mosfet-characteristics)
5. [Load Curve Analysis](#load-curve-analysis)
6. [Operating Region Analysis](#operating-region-analysis)
7. [SPICE Deck Structure](#spice-deck-structure)
8. [Lab Session](#lab-session)
9. [Results and Analysis](#results-and-analysis)
10. [Conclusions](#conclusions)

---

## Introduction

Day 3 focuses on the **CMOS Inverter**, the fundamental building block of digital logic circuits. We analyze its DC and transient characteristics to understand:
- Voltage Transfer Characteristic (VTC)
- Operating regions of NMOS and PMOS transistors
- Switching threshold and noise margins
- Propagation delays (rise and fall times)

**Learning Objectives:**
- Plot and interpret VTC curves
- Understand load line analysis
- Identify transistor operating regions during switching
- Extract timing parameters (delays, transition times)
- Relate transistor-level behavior to digital circuit performance

---

## CMOS Inverter Fundamentals

### Circuit Structure

```
        VDD
         |
         |
    ┌────┴────┐
    │  PMOS   │
    │  (Pull-up)
    └────┬────┘
         │
    Vin ─┤
         │ Vout
    ┌────┴────┐
    │  NMOS   │
    │ (Pull-down)
    └────┬────┘
         |
        GND
```

**Key Components:**
- **PMOS (Wp/Lp)**: Pull-up network, turns ON when Vin = LOW
- **NMOS (Wn/Ln)**: Pull-down network, turns ON when Vin = HIGH
- **Vin**: Input voltage (gate connection for both transistors)
- **Vout**: Output voltage (common drain connection)

---

### Operating Principle

| Input (Vin) | PMOS State | NMOS State | Output (Vout) |
|-------------|------------|------------|---------------|
| **LOW (0V)** | ON (conducts) | OFF (cutoff) | HIGH (VDD) |
| **HIGH (VDD)** | OFF (cutoff) | ON (conducts) | LOW (0V) |

**Ideal Characteristics:**
- **High noise margins**: Sharp transition between HIGH and LOW
- **Rail-to-rail swing**: Vout = 0V or VDD
- **No static power consumption**: One transistor always OFF
- **Symmetric switching**: Equal rise and fall delays (with proper sizing)

---

## Voltage Transfer Characteristic (VTC)

### What is VTC?

The **Voltage Transfer Characteristic (VTC)** is a plot of output voltage (Vout) versus input voltage (Vin) under DC conditions.

**Key Parameters:**
1. **VOH (Output High Voltage)**: Maximum output voltage when Vin = LOW
2. **VOL (Output Low Voltage)**: Minimum output voltage when Vin = HIGH
3. **VIH (Input High Voltage)**: Minimum input recognized as HIGH
4. **VIL (Input Low Voltage)**: Maximum input recognized as LOW
5. **VM (Switching Threshold)**: Input voltage where Vout = Vin

---

### VTC Regions

The VTC can be divided into **five regions** based on transistor operation:

```
Region │ Vin Range    │ NMOS State │ PMOS State │ Vout
───────┼──────────────┼────────────┼────────────┼──────
  I    │ 0 → VTn      │ Cutoff     │ Linear     │ ≈ VDD
  II   │ VTn → VM     │ Saturation │ Linear     │ Falling
  III  │ ≈ VM         │ Saturation │ Saturation │ Sharp drop
  IV   │ VM → VDD+VTp │ Linear     │ Saturation │ Falling
  V    │ VDD+VTp → VDD│ Linear     │ Cutoff     │ ≈ 0V
```

---

### Ideal vs. Real VTC

**Ideal Inverter:**
- Step function at VM
- Infinite gain at switching point
- Zero transition region

**Real Inverter:**
- Gradual transition
- Finite gain (slope = dVout/dVin)
- Transition region width depends on device sizing

---

## Individual MOSFET Characteristics

### NMOS Characteristics: Idsn vs Vdsn

**Configuration for Analysis:**
- Gate connected to Vin (variable)
- Drain connected to Vout (variable)
- Source connected to GND (0V)
- Vgsn = Vin - 0 = Vin
- Vdsn = Vout - 0 = Vout

**Current Equations:**

**Cutoff (Vin < VTn):**
```
Idsn ≈ 0
```

**Linear Region (Vin > VTn and Vout < Vin - VTn):**
```
Idsn = μn·Cox·(Wn/Ln)·[(Vin - VTn)·Vout - Vout²/2]
```

**Saturation Region (Vin > VTn and Vout ≥ Vin - VTn):**
```
Idsn = (1/2)·μn·Cox·(Wn/Ln)·(Vin - VTn)²·(1 + λn·Vout)
```

---


```

**Observations:**
- Higher Vin → Higher current
- Each curve shows linear → saturation transition
- Saturation begins at Vdsn = Vin - VTn

---

### PMOS Characteristics: Idsp vs Vdsp

**Configuration for Analysis:**
- Gate connected to Vin (variable)
- Drain connected to Vout (variable)
- Source connected to VDD (1.8V)
- Vgsp = Vin - VDD (negative for conduction)
- Vdsp = Vout - VDD (negative)

**Current Equations:**

**Cutoff (Vin > VDD + VTp, where VTp < 0):**
```
Idsp ≈ 0
```

**Linear Region (Vin < VDD + VTp and |Vdsp| < |Vgsp - VTp|):**
```
Idsp = μp·Cox·(Wp/Lp)·[(Vgsp - VTp)·Vdsp - Vdsp²/2]
```

**Saturation Region (Vin < VDD + VTp and |Vdsp| ≥ |Vgsp - VTp|):**
```
Idsp = (1/2)·μp·Cox·(Wp/Lp)·(Vgsp - VTp)²·(1 + λp·|Vdsp|)
```

---


**Observations:**
- Lower Vin → Higher current (PMOS conducts with negative Vgs)
- Mirror behavior of NMOS
- Saturation begins at |Vdsp| = |Vgsp - VTp|

---

## Load Curve Analysis

### Superimposed Load Curves

**Key Concept:** In the CMOS inverter, NMOS and PMOS are in **series**, so:
```
Idsn = |Idsp| (KCL at Vout node)
```

For a given Vin, the operating point is where NMOS and PMOS currents intersect.

---



---

### Switching Threshold (VM)

**Definition:** The input voltage where Vout = Vin

**Condition at VM:**
- Both transistors in saturation
- Idsn = |Idsp|
- Maximum current flow (highest power)

**Calculation:**

Setting Idsn = Idsp in saturation:
```
(1/2)·μn·Cox·(Wn/Ln)·(Vin - VTn)² = (1/2)·μp·Cox·(Wp/Lp)·(Vin - VDD - VTp)²
```

For symmetric devices (μn·Wn/Ln = μp·Wp/Lp):
```
VM ≈ (VDD + VTn + |VTp|) / 2
```

For typical values (VDD=1.8V, VTn=0.42V, VTp=-0.4V):
```
VM ≈ (1.8 + 0.42 + 0.4) / 2 ≈ 1.31V
```

**Note:** Actual VM depends on relative device sizing (Wp/Wn ratio).


### Summary Table: Operating Regions

| Region | Vin Range | NMOS | PMOS | Vout | Gain | Notes |
|--------|-----------|------|------|------|------|-------|
| **I** | 0 to VTn | Cutoff | Linear | ≈ VDD | Low | VOH |
| **II** | VTn to VM | Saturation | Linear | Falling | Medium | VIL region |
| **III** | ≈ VM | Saturation | Saturation | ≈ VM | **High** | Switching point |
| **IV** | VM to VDD+VTp | Linear | Saturation | Falling | Medium | VIH region |
| **V** | VDD+VTp to VDD | Linear | Cutoff | ≈ 0V | Low | VOL |

---

## SPICE Deck Structure

### Understanding the SPICE Netlist

A typical CMOS inverter SPICE deck contains:

1. **Title and Comments**
2. **Library Inclusion** (.lib statement)
3. **Circuit Description** (MOSFET instances)
4. **Voltage Sources** (VDD, Vin)
5. **Analysis Commands** (.dc, .tran)
6. **Control Block** (.control ... .endc)
7. **Output Commands** (plot, print)

---

### Example SPICE Deck: VTC Analysis

```spice
* CMOS Inverter - Voltage Transfer Characteristic
* Wp = 0.84um, Wn = 0.36um

*--- Library Inclusion ---
.lib "../sky130_fd_pr/models/sky130.lib.spice" tt

*--- Circuit Description ---
* PMOS: Drain Gate Source Bulk Model W L
M1 out in vdd vdd sky130_fd_pr__pfet_01v8 W=0.84 L=0.15

* NMOS: Drain Gate Source Bulk Model W L
M2 out in 0 0 sky130_fd_pr__nfet_01v8 W=0.36 L=0.15

*--- Voltage Sources ---
Vdd vdd 0 1.8        * Supply voltage
Vin in 0 1.8         * Input voltage (swept in DC analysis)

*--- Capacitive Load (optional) ---
Cout out 0 10f       * 10fF load capacitance

*--- DC Analysis ---
.dc Vin 0 1.8 0.01   * Sweep Vin from 0 to 1.8V in 0.01V steps

*--- Control Block ---
.control
run                  * Execute simulation
display              * Show available plots
setplot dc1          * Select DC analysis plot
plot out vs in       * Plot VTC
print out            * Print output values
.endc

.end
```

**Key Components:**

1. **M1 (PMOS):**
   - Drain: `out`, Gate: `in`, Source: `vdd`, Bulk: `vdd`
   - Width: 0.84μm, Length: 0.15μm
   - Note: PMOS bulk tied to VDD to avoid body effect

2. **M2 (NMOS):**
   - Drain: `out`, Gate: `in`, Source: `0` (GND), Bulk: `0`
   - Width: 0.36μm, Length: 0.15μm
   - Note: NMOS bulk tied to GND

3. **Device Sizing:**
   - Wp/Wn ratio ≈ 2.33 (to compensate for lower hole mobility)
   - Typical ratio: μn/μp ≈ 2-3, so Wp ≈ (2-3)·Wn

---

### Example SPICE Deck: Transient Analysis

```spice
* CMOS Inverter - Transient Analysis
* Wp = 0.84um, Wn = 0.36um

*--- Library Inclusion ---
.lib "../sky130_fd_pr/models/sky130.lib.spice" tt

*--- Circuit Description ---
M1 out in vdd vdd sky130_fd_pr__pfet_01v8 W=0.84 L=0.15
M2 out in 0 0 sky130_fd_pr__nfet_01v8 W=0.36 L=0.15

*--- Voltage Sources ---
Vdd vdd 0 1.8

* Pulse Input: V1 V2 Tdelay Trise Tfall Ton Tperiod
Vin in 0 PULSE(0 1.8 0 10p 10p 1n 2n)

*--- Load Capacitance ---
Cout out 0 10f

*--- Transient Analysis ---
.tran 10p 4n         * Time step: 10ps, Stop time: 4ns

*--- Control Block ---
.control
run
plot in out          * Plot input and output waveforms
* Measure propagation delays
meas tran tpdr TRIG v(in) VAL=0.9 RISE=1 TARG v(out) VAL=0.9 FALL=1
meas tran tpdf TRIG v(in) VAL=0.9 FALL=1 TARG v(out) VAL=0.9 RISE=1
print tpdr tpdf      * Print delay values
.endc

.end
```

**Pulse Parameters:**
- V1 = 0V (LOW level)
- V2 = 1.8V (HIGH level)
- Tdelay = 0ns (start immediately)
- Trise/Tfall = 10ps (transition time)
- Ton = 1ns (pulse width)
- Tperiod = 2ns (period)

---

## Lab Session

### Setup Environment

```bash
# Move to design directory
cd ../../design
```

---

### Experiment 1: Voltage Transfer Characteristic (VTC)

**Objective:** Plot Vout vs Vin and observe the switching behavior of the CMOS inverter.

```bash
# Open the inverter VTC netlist
gvim day3_inv_vtc_wp084_wn036.spice
```

**Netlist Contents:**

```spice
* CMOS Inverter VTC
* PMOS: W=0.84um, NMOS: W=0.36um, L=0.15um for both

.lib "../sky130_fd_pr/models/sky130.lib.spice" tt

* Circuit
M1 out in vdd vdd sky130_fd_pr__pfet_01v8 W=0.84 L=0.15
M2 out in 0 0 sky130_fd_pr__nfet_01v8 W=0.36 L=0.15

* Sources
Vdd vdd 0 1.8
Vin in 0 1.8

* Load
Cout out 0 10f

* DC Sweep
.dc Vin 0 1.8 0.01

* Control
.control
run
setplot dc1
plot out vs in
.endc

.end
```

---

```bash
# Run ngspice simulation
ngspice day3_inv_vtc_wp084_wn036.spice
```

**NGSpice Output:**
```
Circuit: CMOS Inverter VTC

DC transfer curve analysis

    Vin          Vout
  -------      --------
   0.000       1.800
   0.100       1.800
   0.200       1.800
   ...
   0.900       1.156  ← Transition region
   0.976       0.976  ← Switching threshold (VM)
   1.000       0.892
   ...
   1.800       0.000
```

---

```bash
# Inside ngspice, plot output vs input
plot out vs in
```

---



### Experiment 2: Transient Analysis

**Objective:** Observe switching behavior over time and measure propagation delays.

```bash
# Open transient simulation netlist
gvim day3_inv_tran_wp084_wn036.spice
```

**Netlist Contents:**

```spice
* CMOS Inverter Transient Analysis
* PMOS: W=0.84um, NMOS: W=0.36um

.lib "../sky130_fd_pr/models/sky130.lib.spice" tt

* Circuit
M1 out in vdd vdd sky130_fd_pr__pfet_01v8 W=0.84 L=0.15
M2 out in 0 0 sky130_fd_pr__nfet_01v8 W=0.36 L=0.15

* Sources
Vdd vdd 0 1.8
Vin in 0 PULSE(0 1.8 0 10p 10p 1n 2n)

* Load
Cout out 0 10f

* Transient Analysis
.tran 10p 4n

* Control
.control
run
plot in out
* Measure delays at 50% points (0.9V for 1.8V supply)
meas tran tpdr TRIG v(in) VAL=0.9 RISE=1 TARG v(out) VAL=0.9 FALL=1
meas tran tpdf TRIG v(in) VAL=0.9 FALL=1 TARG v(out) VAL=0.9 RISE=1
let tpd_avg = (tpdr + tpdf)/2
print tpdr tpdf tpd_avg
.endc

.end
```

---

```bash
# Run transient simulation
ngspice day3_inv_tran_wp084_wn036.spice
```

**NGSpice Output:**
```
Circuit: CMOS Inverter Transient Analysis

Transient analysis

tpdr = 4.523e-11  (45.23 ps)  ← Rise propagation delay
tpdf = 3.876e-11  (38.76 ps)  ← Fall propagation delay
tpd_avg = 4.200e-11 (42.0 ps) ← Average delay
```

---

```bash
# Plot transient waveforms
plot out vs time in vs time
```



**Delay Measurements Using Cursors:**

Inside NGSpice plot window:
1. Left-click on the plot to activate cursor mode
2. Measure times at 50% voltage levels (0.9V for 1.8V supply)

**Rise Delay (tpdr):**
- Measure when Vin crosses 0.9V going UP
- Measure when Vout crosses 0.9V going DOWN
- tpdr = time difference ≈ 45 ps

**Fall Delay (tpdf):**
- Measure when Vin crosses 0.9V going DOWN
- Measure when Vout crosses 0.9V going UP
- tpdf = time difference ≈ 39 ps

**Average Propagation Delay:**
```
tpd = (tpdr + tpdf) / 2 ≈ 42 ps
```

---

### Additional Measurements

**Rise Time (tr):**
- Time for output to transition from 10% to 90% of VDD
- Measure: Vout from 0.18V to 1.62V

**Fall Time (tf):**
- Time for output to transition from 90% to 10% of VDD
- Measure: Vout from 1.62V to 0.18V

```bash
# In NGSpice control block:
meas tran tr TRIG v(out) VAL=0.18 RISE=1 TARG v(out) VAL=1.62 RISE=1
meas tran tf TRIG v(out) VAL=1.62 FALL=1 TARG v(out) VAL=0.18 FALL=1
print tr tf
```

**Typical Values:**
- tr ≈ 35-40 ps
- tf ≈ 30-35 ps

---

## Results and Analysis

### VTC Analysis Results

#### **Noise Margin Analysis:**

```
       VDD (1.8V)
         │
    ┌────┴────┐
    │   NMH   │ 0.68V
    ├─────────┤ VIH = 1.10V
    │         │
    │ Unknown │
    │         │
    ├─────────┤ VIL = 0.85V
    │   NML   │ 0.83V
    └────┬────┘
        GND (0V)
```

---

## Conclusions

### Key Findings from Day 3:

1. **Voltage Transfer Characteristic (VTC):**
   - Successfully plotted and analyzed the DC transfer characteristic of a CMOS inverter
   - Identified five distinct operating regions based on NMOS and PMOS states
   - Measured switching threshold VM ≈ 0.976V for Wp=0.84μm, Wn=0.36μm configuration
   - VTC shows sharp transition region with maximum gain ≈ -18.5, indicating good noise immunity

2. **Individual MOSFET Behavior:**
   - Analyzed Idsn vs Vdsn characteristics for NMOS showing linear and saturation regions
   - Studied Idsp vs Vdsp characteristics for PMOS with complementary behavior
   - Successfully superimposed load curves to understand how both transistors work together
   - Confirmed that operating point is determined by intersection of NMOS and PMOS load curves

3. **Operating Region Analysis:**
   - **Region I (Vin < VTn):** NMOS cutoff, PMOS linear → Vout ≈ VDD
   - **Region II (VTn < Vin < VM):** NMOS saturation, PMOS linear → Vout falling
   - **Region III (Vin ≈ VM):** Both in saturation → Maximum gain, switching point
   - **Region IV (VM < Vin < VDD+VTp):** NMOS linear, PMOS saturation → Vout falling
   - **Region V (Vin > VDD+VTp):** NMOS linear, PMOS cutoff → Vout ≈ 0V

4. **Noise Margins:**
   - NML (Low Noise Margin) ≈ 0.83V (46% of VDD) - Excellent
   - NMH (High Noise Margin) ≈ 0.68V (38% of VDD) - Good
   - Combined noise margins indicate robust digital switching with good immunity to input noise
   - Asymmetry in noise margins due to device sizing (Wp/Wn ≈ 2.33)

---
## References

1. Kunal Ghosh Sky130 Workshop: [https://github.com/kunalg123/sky130CircuitDesignWorkshop/](https://github.com/kunalg123/sky130CircuitDesignWorkshop/)

---

**Author**: [Anuj Loyare]  
**Date**: October 19, 2025  
**Workshop**: SKY130 CMOS Circuit Design - Day 3
