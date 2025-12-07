# Project overview
This project introduces beginners to how a System on Chip (SoC) works using a small open-source design called VSDBabySoC, which is based on the RVMYTH RISC-V processor core. It explains the main components of SoCs, their importance, and how different digital and analog elements work together within a chip.

---
## Understanding SoC Basics  
A System on Chip combines all major functions, including the processor, memory, input/output ports, power control, and sometimes additional modules like Wi-Fi, onto a single chip. This integration makes devices:

- Smaller and more portable  
- Power-efficient  
- Faster and more reliable  
- Cost-effective to produce  

SoCs are used in phones, wearables, IoT devices, and smart appliances because they efficiently handle multiple tasks in limited space.

---
## Types of SoCs  
- Microcontroller-based SoC: For simple, low-power tasks (e.g., sensors, appliances).  
- Microprocessor-based SoC: For complex tasks that run operating systems (e.g., smartphones).  
- Application-Specific SoC: Built for specific, high-speed jobs like graphics or AI.  

---
## The VSDBabySoC Design  
VSDBabySoC is a small educational SoC built using Sky130 technology. It contains:

- RVMYTH (CPU): A simple RISC-V processor that executes instructions and controls data.  
- PLL (Phase-Locked Loop): Generates a stable internal clock so all components work in sync.  
- DAC (Digital-to-Analog Converter): Converts processed digital data into analog signals for audio or video output.  

### The SoC operates as follows:

1. The PLL produces a synchronized clock signal.  
2. The RVMYTH CPU processes data stored in its registers (like r17).  
3. The DAC converts this digital data into analog form, which can be output to devices like TVs or speakers.  

---
## PLL (Phase-Locked Loop)  
A PLL aligns an output signal with the phase and frequency of an input signal. It contains:

- A Phase Detector (compares signals)  
- A Loop Filter (smooths the signal)  
- A Voltage-Controlled Oscillator (adjusts frequency)  

PLLs are preferred over off-chip clocks because on-chip solutions reduce timing delays, jitter, and frequency mismatches that external sources may cause.

---
## DAC (Digital-to-Analog Converter)  
A DAC converts binary digital values into smooth analog voltages or currents. Types include:

- Weighted Resistor DAC: Uses different resistor values for each bit.  
- R-2R Ladder DAC: Uses repeating resistor pairs for easier scalability.  

In VSDBabySoC, a 10-bit DAC means it can represent 1024 levels of analog output, allowing for precise audio or visual signal generation.

---
In Summary  
VSDBabySoC shows how digital and analog building blocks come together in an SoC.

*Digital side:* RVMYTH processor and PLL ensure synchronized processing.  
*Analog side:* DAC creates real-world signals.  

This project helps beginners understand how modern chips blend computation, timing, and signal conversion to power everyday smart devices.

---

# BabySoC Functional Flow: From RTL to Gate-Level Verification  

This repository captures the design and verification journey of the **VSD BabySoC** –  a small but complete System-on-Chip (SoC) built using open-source tools.  
It integrates:  
- `rvmyth` → a RISC-V CPU core  
- `avsddac` → Digital-to-Analog Converter  
- `avsdpll` → Phase-Locked Loop  

The project demonstrates **RTL-to-Gate flow** with both **pre-synthesis** and **post-synthesis verification**.

---

## 📌 Objectives  

<!-- Why this project is useful -->
- Build a strong foundation in **SoC fundamentals**  
- Practice **functional modeling and verification** of BabySoC  
- Convert RTL to a **gate-level netlist** with the SkyWater 130nm library  
- Verify that **post-synthesis GLS = original RTL behavior**  

---

## 🔧 Tools & Setup  

<!-- Tools required -->
- **Yosys** → Logic synthesis  
- **Icarus Verilog** → Simulation  
- **GTKWave** → Waveform visualization  
- **Sandpiper** → Convert TL-Verilog to Verilog for `rvmyth`  

 Command for Sandpiper:  
```bash
pip3 install pyyaml click sandpiper-saas
sandpiper-saas -i ./src/module/*.tlv -o rvmyth.v --bestsv --noline -p verilog --outdir ./src/module/
```
This generates rvmyth.v, which is used in synthesis and simulation.
![rvmyth_core](assets/output_rvmyth_core.jpg)

---
## 🖥️ Workflow
### 1. RTL Simulation (Pre-Synthesis)
<!-- Pre-synthesis RTL check -->
Run RTL simulation with Icarus Verilog:

```bash
iverilog -o output/pre_synth_sim/pre_synth_sim.out -DPRE_SYNTH_SIM src/module/testbench.v -I src/include -I src/module
cd output/pre_synth_sim
./pre_synth_sim.out
gtkwave pre_synth_sim.vcd
```

✔ Confirms CPU writes to DAC & PLL locks properly.
![Presynthesis_simulation](assets/pre_synth_simulation.jpg)

### 2. Synthesis with Yosys
<!-- RTL -> Gate level -->
Analog IPs are black-boxed using stubs.

#### `src/module/avsddac_stub.v`

```verilog
// Stub module for the avsddac analog IP.
// This is a black box for synthesis. Do not add logic here.

module avsddac (
   output OUT,
   input [9:0] D,
   input VREFH,
   input VREFL
);

// Intentionally empty

endmodule
```

#### `src/module/avsdpll_stub.v`

```verilog
// Stub module for the avsdpll analog IP.
// This is a black box for synthesis. Do not add logic here.

module avsdpll (
   output reg  CLK,
   input  wire VCO_IN,
   input  wire ENb_CP,
   input  wire ENb_VCO,
   input  wire REF
);

// Intentionally empty

endmodule
```

#### Run synthesis:
```bash
read_liberty -lib src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_liberty -lib src/lib/avsddac.lib
read_liberty -lib src/lib/avsdpll.lib
read_verilog src/module/vsdbabysoc.v
read_verilog -I src/include src/module/rvmyth.v
read_verilog -I src/include src/module/clk_gate.v
read_verilog src/module/avsddac_stub.v
read_verilog src/module/avsdpll_stub.v
synth -top vsdbabysoc
dfflibmap -liberty src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
opt
abc -liberty src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
flatten
setundef -zero
clean -purge
rename -enumerate
write_verilog -noattr VSDBabySoC/src/module/vsdbabysoc_netlist.v
stat -liberty src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show vsdbabysoc
```
![Chip_statistics](assets/chip_statistics.jpg)

✔ Produces gate-level netlist → reportsVSDBabySoC/src/module/vsdbabysoc_netlist.v
![Yosys_simulation](assets/vsdbabysoc_yosys_show.jpg)

### 3. Gate-Level Simulation (GLS)
<!-- Post-synthesis verification -->
Simulate synthesized netlist:

```bash
iverilog -DFUNCTIONAL -DUNIT_DELAY=#1   -I src/gls_model   -o output/post_synth_sim/post_synth_sim.out   src/module/testbench.rvmyth.post-routing.v src/module/testbench.v  src/gls_model/primitives.v   src/gls_model/sky130_fd_sc_hd.v   output/synthesized/vsdbabysoc.synth.v   src/module/avsdpll.v   src/module/avsddac.v
cd output/post_synth_sim
./post_synth_sim.out
gtkwave dump.vcd
```

✔ GLS waveform = RTL waveform → functional equivalence verified.
![Postsynthesis_simulation](assets/post_synth_simulation.jpg)

---
## 📊 Results
✅ RTL simulation successful

✅ Synthesized netlist generated

✅ Identical waveforms in RTL & GLS

---
## 🚀 Conclusion
This project provides a complete RTL-to-GLS verification flow for BabySoC.
The results confirm that the design is functionally sound and ready for Physical Design (PnR).

---



