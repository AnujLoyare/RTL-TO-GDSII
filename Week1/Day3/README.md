# Day 3: 🚀 Combinational and Sequential Optimization

Welcome to Day 3 of this workshop! Today we discuss optimization of combinational and sequential circuits, introducing techniques to enhance efficiency and performance.

---

## Table of Contents

- [1. Combinational Logic Optimization](#1-combinational-logic-optimization)
- [2. Sequential Logic Optimization](#2-sequential-logic-optimization)
- [3. Unused Output Optimization](#3-unused-output-optimization)

---

## Optimization Overview

### Combinational Logic Optimization
Combinational optimization involves techniques like constant propagation and Boolean logic optimization to reduce circuit complexity, improve performance, and minimize resource usage by simplifying logic expressions and removing redundant gates.

**Key Techniques:**
- **⚡ Constant Propagation:** Replaces variables with constant values during synthesis, simplifying logic and reducing circuit size
- ** Boolean Logic Optimization:** Uses Boolean algebra to minimize logic expressions and eliminate redundant gates

###  Sequential Logic Optimization
Sequential optimization focuses on optimizing circuits with memory elements through techniques like sequential constant propagation, state optimization, cloning, and retiming to improve timing performance and reduce power consumption.

**Key Techniques:**
- **State Optimization:** Reduces FSM states and optimizes encoding to minimize logic complexity
- **Cloning:** Duplicates logic cells to balance load and improve timing performance
- **Retiming:** Repositions registers to optimize clock period without changing functionality

---

## 1. Combinational Logic Optimization

### Lab 1: opt_check

```verilog
module opt_check (input a , input b , output y);
	assign y = a?b:0;
endmodule
```

**Explanation:** This is a 2-to-1 multiplexer where if `a` is true, `y` equals `b`; otherwise `y` is 0. This can be optimized to a simple AND gate (`y = a & b`).

**Before Optimization:**
![Before Optimization 1](/home/anuj-loyare/RTL-TO-GDSII/Week1/Day3/images/Before_Optimization_1.png)

**After Optimization:**
![After Optimization 1](/home/anuj-loyare/RTL-TO-GDSII/Week1/Day3/images/After_Optimization_1.png)

---

### Lab 2: opt_check2

```verilog
module opt_check2 (input a , input b , output y);
	assign y = a?1:b;
endmodule
```

**Explanation:** This multiplexer outputs `1` when `a` is true, otherwise outputs `b`. This can be optimized to `y = a | b` (OR gate).

**Before Optimization:**
![Before Optimization 2](/home/anuj-loyare/RTL-TO-GDSII/Week1/Day3/images/Before_Optimization_2.png)

**After Optimization:**
![After Optimization 2](/home/anuj-loyare/RTL-TO-GDSII/Week1/Day3/images/After_Optimization_2.png)

---

### Lab 3: opt_check3

```verilog
module opt_check3 (input a , input b , input c , output y);
	assign y = a?(c?b:0):0;
endmodule
```

**Explanation:** This nested multiplexer can be simplified to `y = a & c & b` (3-input AND gate).

**Before Optimization:**
![Before Optimization 3](/home/anuj-loyare/RTL-TO-GDSII/Week1/Day3/images/Before_Optimization_3.png)

**After Optimization:**
![After Optimization 3](/home/anuj-loyare/RTL-TO-GDSII/Week1/Day3/images/After_Optimization_3.png)

---

## 2. Sequential Logic Optimization

Sequential optimization involves optimizing circuits with flip-flops and latches by identifying sequential constants and removing redundant logic.

### Lab 1: dff_const1

```verilog
module dff_const1(input clk, input reset, output reg q);
always @(posedge clk, posedge reset)
begin
	if(reset)
		q <= 1'b0;
	else
		q <= 1'b1;
endmodule
```

**Explanation:** D flip-flop with asynchronous reset. When not in reset, it always loads `1`, but the flip-flop is still needed as the output depends on clock and reset timing.

**After Optimization:**
![DFF Constant 1](/home/anuj-loyare/RTL-TO-GDSII/Week1/Day3/images/After_optimization_DFF_1.png)

---

### Lab 2: dff_const2

```verilog
module dff_const2(input clk, input reset, output reg q);
always @(posedge clk, posedge reset)
begin
	if(reset)
		q <= 1'b1;
	else
		q <= 1'b1;
endmodule
```

**Explanation:** This flip-flop always outputs `1` regardless of clock or reset state, so it can be optimized to a constant `1` connection.

**After Optimization:**
![DFF Constant 2](/home/anuj-loyare/RTL-TO-GDSII/Week1/Day3/images/DFF_2.png)

---

## 3. Unused Output Optimization

Unused output optimization removes logic that doesn't affect any primary outputs, reducing area and power consumption by eliminating dead code.

### Lab: counter_opt

```verilog
module counter_opt (input clk , input reset , output q);
reg [2:0] count;
assign q = count[0];

always @(posedge clk ,posedge reset)
begin
	if(reset)
		count <= 3'b000;
	else
		count <= count + 1;
end
endmodule
```

**Explanation:** This is a 3-bit counter, but only the LSB (`count[0]`) is used as output. The synthesis tool can optimize this by removing the unused upper bits and implementing only a toggle flip-flop.

**Before Optimization:**
![Counter Before Optimization](/home/anuj-loyare/RTL-TO-GDSII/Week1/Day3/images/counter_opt.png)

**After Optimization:**
![Counter After Optimization](/home/anuj-loyare/RTL-TO-GDSII/Week1/Day3/images/After_counter_opt.png)

---

## Synthesis Commands

For all optimization labs, use the following commands:

```shell
# Read liberty file
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib

# Read verilog
read_verilog <module_name>.v

# Synthesize
synth -top <module_name>

# Clean and optimize
opt_clean -purge

# Map to library
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib

# Show schematic
show
```

---

## Summary

This day covered three main optimization techniques:

1. **Combinational Logic Optimization:** Simplifying logic expressions using constant propagation and Boolean optimization to reduce gate count and improve performance.

2. **Sequential Logic Optimization:** Optimizing flip-flops and sequential circuits by identifying constant values and removing redundant sequential elements.

3. **Unused Output Optimization:** Removing logic that doesn't contribute to primary outputs, significantly reducing circuit complexity when only partial outputs are used.

Each optimization technique demonstrates how synthesis tools can automatically improve circuit efficiency while maintaining functional correctness.
