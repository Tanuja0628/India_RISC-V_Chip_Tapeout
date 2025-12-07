# Gate Level Simulation of BabySoC
Gate-Level Simulation (GLS) is a critical verification step that checks the correctness of a digital design after synthesis. Unlike RTL or behavioral simulations, which work at a higher abstraction, GLS operates on the actual gate-level netlist, ensuring the design meets both functional and timing requirements.

For BabySoC, which integrates multiple modules like the RISC-V processor, PLL, and DAC, GLS validates that these components interact correctly under real-world conditions.

---
# Why GLS is Important for BabySoC

## Timing Verification
- GLS uses Standard Delay Format (SDF) files to incorporate real gate delays.
- Ensures the SoC behaves as expected when subjected to physical timing constraints.

## Design Validation Post-Synthesis
- Confirms that the synthesized netlist maintains the intended logical behavior.
- Detects issues like glitches or potential metastability.

## Simulation Tools
- Netlist compilation and simulation: Icarus Verilog or similar.
- Waveform visualization: GTKWave.

---

# Step-by-Step GLS Execution for BabySoC
## Step 1: Load the Top-Level Design and Modules

Launch Yosys and load the main design and supporting modules:
```bash
yosys
read_verilog src/module/vsdbabysoc.v
read_verilog -I src/include src/module/rvmyth.v
read_verilog -I src/include src/module/clk_gate.v
```

## Step 2: Load Standard Cell Libraries

Include the necessary .lib files for synthesis:
```bash
read_liberty -lib src/lib/avsdpll.lib
read_liberty -lib src/lib/avsddac.lib
read_liberty -lib src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

## Step 3: Synthesize the Design

Target the top module:
```bash
synth -top vsdbabysoc
```

## Step 4: Map Flip-Flops to Standard Cells
```bash
dfflibmap -liberty src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

## Step 5: Optimize and Map Technology
```bash
opt
abc -liberty src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib -script +strash;scorr;ifraig;retime;{D};strash;dch,-f;map,-M,1,{D}
```

## Step 6: Clean-Up and Rename
```bash
flatten
setundef -zero
clean -purge
rename -enumerate
```

## Step 7: Check Design Statistics
```bash
stat
```

## Step 8: Export the Synthesized Netlist
```bash
write_verilog -noattr output/post_synth_sim/vsdbabysoc.synth.v
```
---

# Post-Synthesis Simulation

## Step 1: Compile the Testbench
```bash
iverilog -DFUNCTIONAL -DUNIT_DELAY=#1 \
  -I src/gls_model \
  -o output/post_synth_sim/post_synth_sim.out \
  src/module/testbench.v \
  src/gls_model/primitives.v \
  src/gls_model/sky130_fd_sc_hd.v \
  output/synthesized/vsdbabysoc.synth.v \
  src/module/avsdpll.v \
  src/module/avsddac.v
```

## Step 2: Navigate to Output Directory
```bash
cd output/post_synth_sim/
```

## Step 3: Run the Simulation
```bash
./post_synth_sim.out
```

## Step 4: View Waveforms
```bash
gtkwave post_synth_sim.vcd
```
---
![iverilog](assets/iverilog_sim.jpg) 
# Output of Post Synthesis Simulation

![Post_synthesis](assets/post_sim_result.jpg)

This workflow ensures that BabySoC behaves correctly after synthesis and all timing constraints are met.

---
# Installation of OpenSTA

## 1. Install CMake
```bash
sudo apt update
sudo apt install -y cmake
```
## 2. Install CUDD
```bash
git clone https://github.com/ivmai/cudd.git
cd cudd
autoreconf -i
./configure --prefix=/usr/local
make -j$(nproc)
sudo make install
```
## 3. Build OpenSTA
```bash
git clone https://github.com/The-OpenROAD-Project/OpenSTA.git
cd OpenSTA
mkdir build
cd build
cmake -DCUDD_DIR=/usr/local ..
make -j$(nproc)
```
For more detailed instructions, please refer to the [OpenSTA GitHub Repository](https://github.com/The-OpenROAD-Project/OpenSTA)

---

# The Timing Graphs of various synthesized netlists have been generated and submitted in the same folder

---
# Static Timing Analysis (STA) – Part I: Mandatory Timing Checks

## Introduction
Static Timing Analysis (STA) is essential for timing verification at each physical design step. This course addresses **signoff timing checks**.
It presents fundamentals of timing, practical STA techniques, and builds a foundation for further STA topics.
The course begins from the ground up and moves towards intermediate/advanced timing analysis.

---

## Principal Learning Outcomes
- Familiarize yourself with **timing path, arrival time (AAT), required arrival time (RAT), and slack**.  
- Check **setup and hold for flip-flops and latches**.  
- Examine **data, clock, slew, and load effects**.  
- Study **transistor-level timing** for flops and latches.  
- Perform **jitter analysis and factor variations** (etching, oxide thickness, etc.).  
- Obtain knowledge of **industry-standard STA practices**.

---  

## Fundamental Concepts
- **Timing Path:** Chain of logic cells from source to sink.
- **Arrival Time (AAT):** When the data arrives at a node.
- **Required Arrival Time (RAT):** When the data should arrive to satisfy timing.
- **Slack:** The difference between RAT and AAT; positive slack = timing satisfied, negative slack = violation.
- **Setup & Hold Checks:** Data is stable before/after clock edge.

---

## STA Steps Covered
1. Convert **logic gates and pins into nodes** for timing calculations.  
2. Compute **AAT, RAT, and slack** for all paths.  
3. Perform **GBA-PBA analysis** (Graphical vs. Path-based analysis).  
4. Transistor-level understanding of **flop/latch operation**.  
5. Compute **library setup times** and **clk-to-q delays**.  
6. **Jitter analysis** via eye diagrams.  
7. **Setup and hold timing** accounting for pessimism (OCV adjustments).
8. Examine **variation sources**: etching, oxide thickness, resistance, drain current.

---

## Practical Applications
- **Signoff timing verification** for actual designs.
- **Quality analysis** of the timing closure in industry processes.
- **Converting graphical STA reports** into textual information for design choices.
- Leads to **advanced STA and physical design optimization**.

---

## Prerequisites
- Familiarity with **physical design flow** is beneficial but not necessary.
- Familiarity with **flip-flops and basic sequential logic**.

---

## Conclusion
At the completion of this course, students can **analyze timing paths, detect violations, and execute STA with industry-standard quality**.
It lays the groundwork for **advanced timing analysis, signoff procedures, and optimization** in physical design.

---
