# RISC-V_SOC_Week7
VSDBabySoC Physical Design Flow – Sky130HD (OpenROAD)

This repository contains the steps, logs, assets, and observations from completing the full RTL-to-GDS flow for VSDBabySoC using OpenROAD-flow-scripts with the Sky130HD PDK.

## 📌 1. Deliverables
1.1 Layout assets / Images

✔️ Floorplan view
![floorplan](assets/floorplan.jpg)

✔️ Placement view
![placement](assets/placement.jpg)

✔️ CTS view
![cts](assets/cts.jpg)

✔️ Routed design (with congestion issue noted)
![routing](assets/routing.jpg)

✔️ Terminal output showing SPEF generation



## 📌 2. Commands Used (Step-by-Step)
🔹 1. Run the full flow
```
cd OpenROAD-flow-scripts/flow
make DESIGN_CONFIG=designs/sky130hd/VSDBabySoC/config.mk
```

🔹 2. View the design in OpenROAD GUI

Start GUI:

```
openroad -gui
```

Load LEF files:

```
read_lef platforms/sky130hd/lef/sky130_fd_sc_hd.tlef
read_lef platforms/sky130hd/lef/sky130_fd_sc_hd_merged.lef
read_lef platforms/sky130hd/lef/sky130io_fill.lef
```

Load any stage’s layout:

Floorplan

```
read_db results/sky130hd/VSDBabySoC/base/2_floorplan.odb
```

Placement

```
read_db results/sky130hd/VSDBabySoC/base/3_5_place_dp.odb
```

CTS

```
read_db results/sky130hd/VSDBabySoC/base/4_1_cts.odb
```

Global Route (failed due to congestion)

```
read_db results/sky130hd/VSDBabySoC/base/5_1_grt-failed.odb
```

## 📌 3. Observations + Challenges
🔸 Floorplanning

Core utilization was applied correctly.

Power grid generation successful.

🔸 Placement

Global and detailed placement executed without issues.

No major density violations.

🔸 Clock Tree Synthesis

CTS completed successfully.

Buffers inserted as expected.

Skew and insertion delay within expected range.

🔸 Routing

⚠️ Routing did not complete successfully

Global routing encountered congestion at multiple regions.

Final overflow reported (example):

```
[ERROR GRT-0116] Global routing finished with congestion.
```

The flow generated:

5_1_grt-failed.odb


Since congestion persisted, detailed routing could not complete cleanly.

This has been documented as part of the verification notes.

## 📌 4. SPEF Generation

The flow automatically generates a SPEF file after routing (if routing is successful).

Typical log:

Writing parasitics SPEF...


Command used internally by OpenROAD:

```
estimate_parasitics -global_routing
write_spef results/sky130hd/VSDBabySoC/base/VSDBabySoC.spef
```

## 📌 5. What is SPEF and Why It Matters

SPEF (Standard Parasitic Exchange Format) represents:

- Wire resistances
- Wire capacitances
- Coupling capacitances between nets
- Parasitic delay contributions

✔ Vital for Accurate STA (Static Timing Analysis)

- Without SPEF, STA uses ideal wires → unrealistic timing.
- With SPEF, STA uses extracted RC parasitics, giving:
- Accurate delay
- Better slack estimation
- Correct setup/hold analysis
- Realistic clock skew calculations
- Final sign-off timing metrics

Thus, SPEF is essential for tape-out-quality design verification.

## 📌 6. Verification Note

✔ RTL synthesis completed successfully

✔ Floorplan, placement, and CTS successful

❌ Routing failed due to congestion

✔ Congestion reports generated

✔ All intermediate ODB stages saved

✔ Design can be debugged by inspecting congestion maps in GUI

Therefore:

➡️ The VSDBabySoC layout is not fully complete because routing overflow prevented detailed routing from finishing.

This is documented as part of the project deliverables.
