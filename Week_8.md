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
read_db /home/tanuja/OpenROAD-flow-scripts/flow/results/sky130hd/vsdbabysoc/base/5_route.odb
write_def /home/tanuja/OpenROAD-flow-scripts/flow/results/sky130hd/vsdbabysoc/base/5_route.def
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
read_lef /home/tanuja/OpenROAD-flow-scripts/flow/designs/sky130hd/vsdbabysoc/lef/sky130hd.lef
read_lef /home/tanuja/OpenROAD-flow-scripts/flow/designs/sky130hd/vsdbabysoc/lef/avsdpll.lef
read_lef /home/tanuja/OpenROAD-flow-scripts/flow/designs/sky130hd/vsdbabysoc/lef/avsddac.lef
read_liberty /home/tanuja/OpenROAD-flow-scripts/flow/platforms/sky130hd/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_def /home/tanuja/OpenROAD-flow-scripts/flow/results/sky130hd/vsdbabysoc/base/5_route.def
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
define_process_corner -ext_model_index 0 /home/tanuja/OpenROAD-flow-scripts/external-resources/open_pdks/sky130/openlane/rules.openrcx.sky130A.nom.calibre
```
6. Extract parasitics
```
extract_parasitics -ext_model_file /home/tanuja/OpenROAD-flow-scripts/external-resources/open_pdks/sky130/openlane/rules.openrcx.sky130A.nom.calibre
```
7. Write the SPEF file
```
write_spef /home/tanuja/OpenROAD-flow-scripts/flow/designs/sky130hd/vsdbabysoc/vsdbabysoc.spef
```
8. Export the post-route Verilog netlist
```
write_verilog /home/tanuja/OpenROAD-flow-scripts/flow/designs/sky130hd/vsdbabysoc/vsdbabysoc_post_place.v
```
![routing](assets/Pasted_image.png)

---

## post route Static Timing Analysis

Post-route Static Timing Analysis (STA) is carried out after the routing stage to ensure that the chip still meets all timing requirements under different PVT (Process–Voltage–Temperature) corners. Since routing adds parasitic resistance and capacitance to the nets, these effects must be accounted for before finalizing the layout. Post-route STA checks timing on the fully extracted layout (GDSII) and confirms that it satisfies all constraints. During this analysis, key timing metrics such as Worst Negative Slack (WNS), Total Negative Slack (TNS), and setup/hold margins are reported.

### Setup

The post-route STA process is automated using a TCL script. In my flow, this script is named sta.tcl, and it drives the timing tool to perform all the necessary checks on the routed design.

```sta.tcl

# Define list of timing libraries (corners)

set list_of_lib_files(1) "sky130_fd_sc_hd__tt_025C_1v80.lib"
 # set list_of_lib_files(2) "sky130_fd_sc_hd__ff_100C_1v65.lib"
 # set list_of_lib_files(3) "sky130_fd_sc_hd__ff_100C_1v95.lib"
 # set list_of_lib_files(4) "sky130_fd_sc_hd__ff_n40C_1v56.lib"
 # set list_of_lib_files(5) "sky130_fd_sc_hd__ff_n40C_1v65.lib"
 # set list_of_lib_files(6) "sky130_fd_sc_hd__ff_n40C_1v76.lib"
 # set list_of_lib_files(7) "sky130_fd_sc_hd__ss_100C_1v40.lib"
 # set list_of_lib_files(8) "sky130_fd_sc_hd__ss_100C_1v60.lib"
 # set list_of_lib_files(9) "sky130_fd_sc_hd__ss_n40C_1v28.lib"
 # set list_of_lib_files(10) "sky130_fd_sc_hd__ss_n40C_1v35.lib"
set list_of_lib_files(2) "sky130_fd_sc_hd__ss_n40C_1v40.lib"
 # set list_of_lib_files(12) "sky130_fd_sc_hd__ss_n40C_1v44.lib"
 # set list_of_lib_files(13) "sky130_fd_sc_hd__ss_n40C_1v76.lib"
set list_of_lib_files(3) "sky130_fd_sc_hd__ff_n40C_1v95.lib"
#---------------------------------------------
#  Load design libraries 
#---------------------------------------------

 read_liberty /home/tanuja/OpenSTA/examples/timing_libs/avsdpll.lib
 read_liberty /home/tanuja/OpenSTA/examples/timing_libs/avsddac.lib

#---------------------------------------------
#  Loop through each .lib file (corner)
#---------------------------------------------

 for {set i 1} {$i <= [array size list_of_lib_files]} {incr i} {

# Read design and constraints

 read_liberty /home/tanuja/OpenSTA/examples/timing_libs/$list_of_lib_files($i)
 read_verilog /home/tanuja/OpenROAD-flow-scripts/flow/designs/sky130hd/vsdbabysoc/vsdbabysoc_post_place.v
 link_design vsdbabysoc
 current_design
 read_sdc /home/tanuja/OpenROAD-flow-scripts/flow/results/sky130hd/vsdbabysoc/base/4_cts.sdc
 read_spef /home/tanuja/OpenROAD-flow-scripts/flow/results/sky130hd/vsdbabysoc/base/6_final.spef

 # Perform timing checks
 check_setup -verbose

#-----------------------------------------
# Generate detailed reports
#-----------------------------------------

 report_checks -path_delay min_max -fields {nets cap slew input_pins fanout} -digits {4} > /home/tanuja/OpenROAD-flow-scripts/flow/results/sky130hd/vsdbabysoc/base/sta_output/route/min_max_$list_of_lib_files($i).txt

#---------------------------------------------------
# Save key metrics (WNS, TNS, Setup & Hold Slack )
#---------------------------------------------------

 exec echo "$list_of_lib_files($i)" >> /home/tanuja/OpenROAD-flow-scripts/flow/results/sky130hd/vsdbabysoc/base/sta_output/route/sta_worst_max_slack.txt
 report_worst_slack -max -digits {4} >> //home/tanuja/OpenROAD-flow-scripts/flow/results/sky130hd/vsdbabysoc/base/sta_output/route/sta_worst_max_slack.txt

 exec echo "$list_of_lib_files($i)" >> /home/tanuja/OpenROAD-flow-scripts/flow/results/sky130hd/vsdbabysoc/base/sta_output/route/sta_worst_min_slack.txt
 report_worst_slack -min -digits {4} >> /home/tanuja/OpenROAD-flow-scripts/flow/results/sky130hd/vsdbabysoc/base/sta_output/route/sta_worst_min_slack.txt

 exec echo "$list_of_lib_files($i)" >> /home/tanuja/OpenROAD-flow-scripts/flow/results/sky130hd/vsdbabysoc/base/sta_output/route/sta_tns.txt
 report_tns -digits {4} >> /home/tanuja/OpenROAD-flow-scripts/flow/results/sky130hd/vsdbabysoc/base/sta_output/route/sta_tns.txt

 exec echo "$list_of_lib_files($i)" >> /home/tanuja/OpenROAD-flow-scripts/flow/results/sky130hd/vsdbabysoc/base/sta_output/route/sta_wns.txt
 report_wns -digits {4} >> /home/tanuja/OpenROAD-flow-scripts/flow/results/sky130hd/vsdbabysoc/base/sta_output/route/sta_wns.txt
 }

```

This generates the following files

- min_max_sky130_fd_sc_hd__ff_n40C_1v95.lib.txt
- min_max_sky130_fd_sc_hd__ss_n40C_1v40.lib.txt
- min_max_sky130_fd_sc_hd__tt_025C_1v80.lib.txt
- sta_tns.txt
- sta_wns.txt
- sta_worst_max_slack.txt
- sta_worst_min_slack.txt
- timing_summary.csv
