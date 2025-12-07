# RISC-V SOC Week 8
# Post route STA - VSDBabySoC

## Last routing ended in Congestion

⚠️ Routing did not complete successfully

Global routing encountered congestion at multiple regions.

Final overflow reported (example):

```
[ERROR GRT-0116] Global routing finished with congestion.
```

## Cause for Congestion 
Your design had pins placed on a metal layer that the router could not physically reach, due to:
- Macro pin OBS layers
- Wrong LEF pin definitions
- Access blockage
- Incorrect pin layer in the IP or hard macro
The avsdac.lef file is changed to overcome the routing congestion problem.
![routing](assets/routing_crct.jpg)

## Final GDSII outputs
- Floorplan
  ![routing](assets/floorplan.png)
- Placement
  ![routing](assets/placement.png)
- Clock Tree Synthesis
  ![routing](assets/cts.png)
- Routing
  ![routing](assets/routing.png)

---

## SPEF generation 
- Creating Post-Route DEF File

To generate a post-route DEF file from the OpenROAD database (.odb):

```
cd ~/OpenROAD-flow-scripts
source env.sh
cd flow
openroad
```
Load the post-route database and write the DEF:
```
read_db /home/madank/OpenROAD-flow-scripts/flow/results/sky130hd/vsdbabysoc/base/5_route.odb
write_def /home/madank/OpenROAD-flow-scripts/flow/results/sky130hd/vsdbabysoc/base/5_route.def
```
- Generating Post-Route SPEF File
Follow the steps below to generate the SPEF file:
1. Launch OpenROAD
```
cd ~/OpenROAD-flow-scripts
source env.sh
cd flow/
openroad
```
2. Load the required LEF files, timing libraries and post route def
```
read_lef /home/madank/OpenROAD-flow-scripts/flow/designs/sky130hd/vsdbabysoc/lef/sky130hd.lef
read_lef /home/madank/OpenROAD-flow-scripts/flow/designs/sky130hd/vsdbabysoc/lef/avsdpll.lef
read_lef /home/madank/OpenROAD-flow-scripts/flow/designs/sky130hd/vsdbabysoc/lef/avsddac.lef
read_liberty /home/madank/OpenROAD-flow-scripts/flow/platforms/sky130hd/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_def /home/madank/OpenROAD-flow-scripts/flow/results/sky130hd/vsdbabysoc/base/5_route.def
```
3. Set up the parasitic extraction models
Create the external-resources/ directory and clone open_pdks inside it:
```
mkdir external-resources
cd external-resources/
git clone https://github.com/RTimothyEdwards/open_pdks.git
```
Define the extraction model:
```
define_process_corner -ext_model_index 0 /home/madank/OpenROAD-flow-scripts/external-resources/open_pdks/sky130/openlane/rules.openrcx.sky130A.nom.calibre
```
6. Extract parasitics
```
extract_parasitics -ext_model_file /home/madank/OpenROAD-flow-scripts/external-resources/open_pdks/sky130/openlane/rules.openrcx.sky130A.nom.calibre
```
7. Write the SPEF file
```
write_spef /home/madank/OpenROAD-flow-scripts/flow/designs/sky130hd/vsdbabysoc/vsdbabysoc.spef
```
8. Export the post-route Verilog netlist
```
write_verilog /home/madank/OpenROAD-flow-scripts/flow/designs/sky130hd/vsdbabysoc/vsdbabysoc_post_place.v
```
![routing](assets/Pasted_image.png)
