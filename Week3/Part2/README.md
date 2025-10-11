# Week 3 Task: Post-Synthesis GLS & STA Fundamentals

## 🎯 Objective

The goal of this week's task is to understand and perform Gate-Level Simulation (GLS) after synthesis and to learn the fundamentals of Static Timing Analysis (STA) by performing practical experiments with OpenSTA. This document serves as a summary of the key concepts learned from the introductory STA course.

---

## 📚 Summary of Static Timing Analysis (STA) Fundamentals

**Static Timing Analysis (STA)** is a method of verifying the timing performance of a digital circuit without performing a dynamic simulation. It checks for timing violations by analyzing all possible paths in a design against the specified timing constraints.

### 1. The Core of STA: Timing Paths & Checks

A **timing path** is a route through the combinational logic of a design that starts and ends at a sequential element (like a flip-flop) or a primary port.

* **Startpoint:** Where the data is launched (e.g., clock pin of a flip-flop, input port).
* **Endpoint:** Where the data is captured (e.g., data pin of a flip-flop, output port).

For each path, STA calculates the following:
* **Arrival Time (AT):** The actual time it takes for a signal to travel from the startpoint to the endpoint.
* **Required Arrival Time (RAT):** The latest time the signal must arrive at the endpoint to be captured correctly.
* **Slack:** The difference between the RAT and the AT (`Slack = RAT - AT`).
    * **Positive Slack:** The timing path meets the requirement.
    * **Negative Slack:** The timing path has a violation.



---

### 2. Fundamental Timing Checks

#### **Setup Time Analysis**
* **Purpose:** Ensures that data arrives at the endpoint *before* the capture clock edge, allowing it to be stable when the clock triggers.
* **Violation:** Occurs when the data path is too slow, causing the signal to arrive too late. `AT > RAT`.
* **Analysis:** A check against the longest (max delay) path.

#### **Hold Time Analysis**
* **Purpose:** Ensures that data remains stable *after* the capture clock edge, preventing the new data from corrupting the currently captured data.
* **Violation:** Occurs when the data path is too fast, causing the next state's data to arrive too soon.
* **Analysis:** A check against the shortest (min delay) path.

---

### 3. Types of Timing Paths Analyzed

STA breaks down the design into several categories of paths for analysis:

1.  **Register-to-Register (reg2reg):** The most common path, starting from one flip-flop and ending at another.
2.  **Input-to-Register (in2reg):** Starts at a primary input port and ends at a flip-flop.
3.  **Register-to-Output (reg2out):** Starts at a flip-flop and ends at a primary output port.
4.  **Input-to-Output (in2out):** A purely combinational path from a primary input to a primary output.

---

### 4. Other Critical Checks & Considerations

Beyond basic setup and hold, STA performs a wide range of checks:

#### **I. Clock Analysis**
* **Clock Skew:** The difference in arrival time of the clock signal at two different flip-flops. Excessive skew can cause setup or hold violations.
* **Clock Jitter:** The variation of a clock edge from its ideal position over time. Jitter reduces the available time window for data to be stable, making timing closure more difficult. It's often visualized with an **eye diagram**.
* **Pulse Width:** Checks if the high and low periods of the clock are long enough for the sequential cells to operate correctly.



#### **II. Signal Integrity & Load Analysis**
* **Slew / Transition Time:** Measures how quickly a signal changes from low to high or high to low. Poor slew (slow transition) can increase gate delay and power consumption.
* **Load Analysis:** Checks the total capacitive load on a driving cell's output pin, which is determined by the `fanout` (number of gates it drives) and interconnect capacitance. High load increases the delay.

#### **III. Specialized Checks**
* **Clock Gating Checks:** Ensures that gated clocks are enabled and disabled at the correct times without causing glitches.
* **Recovery & Removal Checks:** These are checks for asynchronous reset signals.
    * **Recovery:** The minimum time an asynchronous reset must be de-asserted *before* the next active clock edge.
    * **Removal:** The minimum time an asynchronous reset must remain asserted *after* an active clock edge.
* **Latch-based timing:** Analyzes circuits with latches, considering concepts like **time borrowing**.
