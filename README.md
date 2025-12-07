# India RISC-V Chip Tapeout Journey
A Complete Documentation of My End-to-End Experience in the Open-Source Silicon Program

This repository documents my entire learning and development journey as part of the India RISC-V Chip Tapeout Program, an initiative to democratize silicon design using open-source EDA tools and RISC-V architecture.

The goal of this journey was to understand, build, verify, and tapeout a RISC-V based SoC using tools like OpenLane, OpenROAD, Magic, KLayout, Yosys, and the Skywater 130nm PDK—along with hands-on experience debugging real-world issues encountered during SoC design.

## 📌 Table of Contents

- Program Overview
- Tools & Technologies Used
- Design Flow Summary
- RTL Design and Understanding
- Key Debugging Issues & How I Fixed Them
- Major Learnings & Reflections
- Acknowledgements

## ⭐ Program Overview

The India RISC-V Tapeout Program is a national-level, open-source VLSI training and tapeout initiative. The program allows students and engineers with zero fabrication access to build their own silicon chips.

The program covers:

1. RISC-V ISA basics
2. Fabrication and PDK understanding
3. SoC integration
4. Custom RTL modifications
5. Physical Design using OpenLane
6. STA & signoff
7. Final Tapeout submission

## 🛠 Tools & Technologies Used

1. Open-Source Tools
2. Yosys – Synthesis
3. OpenROAD – Entire PnR stack
4. OpenLane – Automated RTL → GDS flow
5. Magic VLSI – DRC, Layout debug
6. KLayout – GDS visualization
7. Netgen – LVS
8. OpenROAD – Routing
9. OpenSTA – Static Timing Analysis
10. PDK - SkyWater Sky130 – open-source 130nm PDK

## 🧩 Design Flow Summary

The entire RTL → GDS flow followed the OpenLane methodology:

RTL Design & Analysis → Synthesis (Yosys + ABC) → Floorplanning → Placement → Clock Tree Synthesis → Routing → Parasitic Extraction (SPEF) → Static Timing Analysis → Physical Verification (DRC/LVS) → Generation of final GDSII

## 🔧 RTL Design and Understanding

My primary design was VSDBabySoC.

Key modules:

- DAC (avsddac)
- PLL (avsdpll)

Debugging tasks:

- Verilog hierarchy
- Missing .v file paths
- Module integration
- Clock/reset signal propagation


## 🚧 Key Debugging Issues & Fixes
1. No liberty libraries found
Fixed by: Adding correct paths in config.tcl

2. Missing source RTL file
Solved by fixing:VERILOG_FILES path

3. Off-grid pin errors
Solved by: Editing LEF,Snapping pins to grid

4. STA failing due to missing corners 
Fixed by: Adding ff, ss, tt libs

5. Floorplan earlier failure
Solved by: Correct macro placement configuration

## 🎓 Major Learnings & Reflections

This journey taught me:

✔ Full RTL-to-GDS flow

✔ Deep STA understanding

✔ Reading & modifying LEF/GDS

✔ Real-world SoC design constraints

✔ Open-source ASIC toolchain mastery

### Most importantly:
I learned how industry-grade chip design actually happens from scratch.

## 🤝 Acknowledgements

I express my gratitude to:

- VLSI System Design (VSD)
- Open-Source Hardware Community
- Mentors & Contributors of OpenLane/OpenROAD
- SkyWater PDK

## ❤️ A Special Thank You to Kunal Ghosh

I would like to express my heartfelt gratitude to Mr. Kunal Ghosh, Co-Founder of VLSI System Design (VSD).
Thank you, Sir, for giving students like me the rare opportunity to design, verify, and tapeout a real silicon chip using open-source tools—something once unimaginable.

Your vision has transformed India’s semiconductor ecosystem and empowered thousands of students to dream bigger, learn fearlessly, and achieve milestones earlier reserved only for industry experts.

This tapeout journey taught me not just chip design, but discipline, consistency, debugging confidence, and a strong problem-solving mindset—skills I will carry throughout my engineering career.

Thank you for trusting students, guiding us patiently, and making VLSI education accessible to all.
I am truly honored to be part of this initiative.
Thank you, Sir, for inspiring us and lighting the path forward.

---
