## WEEK 6 

### Day 1 — Inception, packages, RISC-V and the move from software to hardware

#### Sky130 Day 1 - Inception of open-source EDA, OpenLANE and Sky130 PDK
Concept: history and motivation behind open-source EDA: reproducibility, lowering barrier-to-entry for academia/industry, and community-driven PDKs like SkyWater’s sky130. OpenLANE is a proven open-source RTL-to-GDS flow that chains synthesis, placement, routing, and signoff tools. Sky130 PDK exposes layer rules, LEF/DEF models and SPICE models that let tools target a real process.
Why it matters: enables students and startups to fabricate real silicon at low cost and learn full-flow digital design.
Practical notes: emphasize licensing (Apache/other), reproducible Docker images, and how OpenLANE orchestrates magic, ngspice, yosys, OpenROAD, TritonCTS, TritonRoute.

#### How to talk to computers
Concept: abstraction layers: from human ideas → algorithms → high-level languages → compilers → assembly → machine code → hardware signals. In ASIC-land: algorithm → RTL (Verilog/VHDL) → synthesis → gates → placement & routing → masks → silicon.
Why it matters: designers must express intent in RTL while being aware of hardware realities (timing, area, power).
Practical notes: give examples of a C loop translated to RTL and then to gate-level timing implications.

#### Introduction to QFN-48 Package, chip, pads, core, die and IPs
Concept: package vs die vs chip — die = silicon piece; chip = packaged die + bond/wirebonds or bumps; QFN-48 = Quad Flat No-lead 48-pin package. Pads are I/O bond-pad locations on die; core = area containing logic (standard-cell core) vs I/O ring and filler; IPs = pre-designed blocks (UART, memory, CPU).
Why it matters: packaging affects I/O count, thermal behavior and board-level integration. Pads & I/O spacing determine padframe floorplan; QFN has exposed thermal pad considerations.
Practical notes: show die floorplan with I/O ring, metal pads, and a core area; mention padframe generation and pad bonding.

#### Introduction to RISC-V
Concept: open ISA (instruction set architecture) with modular base (RV32I/RV64I) and extensions (M, A, F, D, C, etc.). RISC-V enables research & custom extensions easily.
Why it matters: lowers entry-barrier for processor design, widely adopted in academia/industry, great for tapeout projects (Shakti, Rocket, PicoRV32).
Practical notes: talk about privileged ISA, toolchain (GCC/LLVM), simulators (Spike/qemu), and linking to SoC integration.

#### From Software Applications to Hardware
Concept: mapping software functionality to hardware building blocks: identify compute kernels, data movement, memory hierarchy, and timing/latency constraints. Decide what stays in software, what becomes hardware IP or accelerator.
Why it matters: efficient SoC design depends on correct partitioning: hardware accelerators reduce latency/power for critical kernels.
Practical notes: show example — FFT or encryption: profiling → hotspot → RTL accelerator → integration.

#### SoC design and OpenLANE (overview & flow)

#### Introduction to all components of open-source digital ASIC design
Concept: typical components: RTL (Verilog), testbench, synthesis (Yosys/ABC), liberty (.lib) timing models, standard-cell LEF/tech LEF, floorplanning, placement, CTS, routing (OpenROAD/TritonRoute), LVS/DRC (calibre-like checks; magic for layout), SPICE (ngspice) for analog verification, timing signoff (OpenSTA).
Why it matters: each component is a dependency; missing/wrong models break flow.
Practical notes: maintain clear project structure: src/, synth/, layout/, db/, reports/.

#### Simplified RTL2GDS flow
Concept: high-level steps: RTL → synthesis → netlist (verilog/gate-level) → floorplan → placement → CTS (clock tree) → routing → extraction → LVS / parasitic extraction → SPICE/timing signoff → GDSII.
Why it matters: students must understand the sequence and handoffs (e.g., what data flows through DEF/LEF).
Practical notes: emphasize iterative nature: fix timing, congestion, re-run synth/placement.

#### Introduction to OpenLANE and Strive chipsets
Concept: OpenLANE is a set of scripts/containers that perform RTL2GDS using OpenROAD and friends. Strive is a set of pre-built chip examples & flows (or a target platform) that demonstrates how to stitch IPs and run flows (explain depending on your reference — treat “Strive” as example reference design).
Why it matters: OpenLANE standardizes many tedious tasks; Strive or example chipsets provide templates.
Practical notes: show flow.tcl orchestration and designs/ structure in OpenLANE.

#### Introduction to OpenLANE detailed ASIC design flow
Concept: deep-dive on steps OpenLANE automates (synth, floorplan, place, CTS, route, post-route optimization, signoff).
Why it matters: helps debug flow failures and customize steps.
Practical notes: illustrate how to modify runconfigs: clocks, constraints (.sdc), cell libraries, and physical constraints.

#### Get familiar with open-source EDA tools & project structure

#### OpenLANE Directory structure in detail
Concept: typical folders: flow/ (scripts), designs/ (project-specific), pdks/ (PDK files), tools/ (binaries/containers), runs/ (results) and configs/ (run config).
Why it matters: knowing where reports, logs, GDS, DEF/LEF, SDC and power/timing reports live is essential for debugging.
Practical notes: show where to find final.gds, final.spef, post_route.timing.rpt.

#### Design Preparation Step
Concept: pre-flight operations: linting RTL, verifying TOP module, creating constraints (SDC), packaging cell LEFs, generating synopsys .lib, creating config.tcl for OpenLANE, and verifying IP LEFs and MIL.
Why it matters: most runs fail due to missing constraints or mismatched cell names/LEFs.
Practical notes: include checklist: clock definitions, pin mapping, floorplan area, power rails, and I/O pads.

#### Review files after design prep and run synthesis
Concept: check synthesized netlist, synth.rpt, cell usage, area, estimated timing, and warnings. Verify correlation between RTL and synthesized cells.
Why it matters: synthesis is the first tangible mapping to real gates; catching issues here is cheaper.
Practical notes: open synth.v, .sdc, and report_timing.txt; confirm no X-propagation, illegal constructs, or unsupported attributes.

#### Steps to characterize synthesis results
Concept: quantify area, worst-case timing paths, slack distribution, cell utilization, register count, and inferred latches. Use synthesis reports to create constraints and refactor RTL.
Why it matters: characterization informs tradeoffs (e.g., pipeline vs area).
Practical notes: suggest metrics: total cell area (µm²), FMax estimation, register-to-logic ratio, and critical path breakdown.

### Day 2 — Floorplan, library cells and placement strategies

#### Utilization factor and aspect ratio
Concept: utilization = fraction of standard-cell area used vs core area; aspect ratio = core width:height shape. Both influence routing congestion and timing.
Why it matters: too high utilization → routing impossible; too low → wasted area and longer interconnects. Aspect ratio affects wirelength and cell placement efficiency.
Practical notes: recommend util ~60–75% for initial runs; tune shape to fit pins/power rails.

#### Concept of pre-placed cells
Concept: pre-placing macros/IPs (memories, hard IP, I/O) before standard-cell placement to ensure correct routing channels and power planning.
Why it matters: pre-placed large macros influence placement of standard cells and routing congestion.
Practical notes: set obstruction regions and fixed placements in DEF; reserve routing channels for macros.

#### De-coupling capacitors
Concept: on-die or package decoupling capacitors supply instantaneous current to blocks during switching, stabilizing supply voltage and reducing IR drop and high-frequency noise.
Why it matters: helps noise/ground bounce; crucial for clock/power integrity.
Practical notes: in advanced designs, add on-chip decaps or plan package caps; model with PDN simulation.

#### Power planning
Concept: design of power distribution network (PDN): power straps, grid, rails, via stitching, and retention of low-impedance supply. Must account for IR drop, transient currents, and reliability.
Why it matters: improper power planning causes timing variation, functional failure, and reliability issues.
Practical notes: plan coarse global straps then fine distribution to each row of standard cells; run IR-drop analysis.

#### Pin placement and logical cell placement blockage
Concept: physical pin location choices influence routing and timing; blockages (obstructions) prevent placement in certain areas, e.g., for I/O or analog.
Why it matters: poor pin placement increases routing demand and critical net length.
Practical notes: use automatic pin-placement heuristics but review pins near timing-critical blocks; add placement-blocks for keepout zones.

#### Steps to run floorplan using OpenLANE
Concept: flow: set core size/aspect ratio, define die/pitch, set utilization, add macros and pblock constraints, generate floorplan and run initial legalization.
Why it matters: floorplan anchors placement & power.
Practical notes: demonstrate openlane runsteps: floorplan stage, set config.json params (core_margin, die_area), then inspect init_floorplan.def.

#### Review floorplan files and steps to view floorplan
Concept: floorplan artifacts: DEF/LEF, floorplan.tcl, pblock definitions, and placement blockage files. Tools to view: Magic, klayout, or OpenROAD GUI.
Why it matters: visual inspection helps catch overlapping placements, missing keepouts, or odd aspect ratios.
Practical notes: show how to open DEF in Magic and highlight macros, rails and blockages.

#### Review floorplan layout in Magic
Concept: Magic is a layout editor and viewer; load the technology file (sky130.tech) and view LEF/DEF overlay to confirm physical coordinates and layer mapping.
Why it matters: allows detailed inspection and easy debug of DRC issues that will appear later.
Practical notes: walk through load lef, load def, and drc commands in Magic with sky130.

#### Library binding, placement optimization & placement tools

#### Netlist binding and initial place design
Concept: binding: tying logical netlist cells to physical standard-cell names; initial placement places cells in core respecting pblocks/keepouts.
Why it matters: ensures correct mapping of logical design to real cells used by placer and router.
Practical notes: check for unmatched cell names, black-box cells, or missing LEFs.

#### Optimize placement using estimated wire-length and capacitance
Concept: placement optimization tries to minimize estimated wirelength (HPWL) and critical net delay; estimated capacitance helps identify long nets.
Why it matters: better placement reduces timing violations and routing congestion.
Practical notes: show cost-functions used by placer and how to weight timing vs area.

#### Final placement optimization
Concept: after initial legalization, run timing-driven placement passes, cell spreading, rebuffering, and legalization to meet timing and congestion goals.
Why it matters: final placement dramatically affects post-route timing.
Practical notes: use incremental placement to fix hotspots rather than global rip-up.

#### Need for libraries and characterization
Concept: standard cell libraries provide area, LEF, and .lib timing models describing delay vs input slews and output loads. Characterization produces .lib from transistor-level simulation.
Why it matters: accurate timing requires good characterization; cell delays drive STA results.
Practical notes: show steps: netlist for cell → run SPICE corners → generateliberty via characterization tools (ngspice + custom scripts).

#### Congestion aware placement using RePlAce
Concept: RePlAce is a global placer that balances density and connectivity; congestion-aware strategies place cells to minimize routing overflow.
Why it matters: reduces routing iterations and improves routability.
Practical notes: show common tunables: density target and smoothing.

#### Cell design and characterization flows

#### Inputs for cell design flow
Concept: inputs: schematic/netlist, layout rules (PDK), target drive strengths, timing goals, transistor sizes, and parasitic constraints. Also require test vectors for functional verification.
Why it matters: these inputs drive both schematic sizing and layout choices.
Practical notes: define target cell height (row height), pin locations, and rail positions up front.

#### Circuit design step
Concept: choose topology (static CMOS), size transistors for drive strength and edge rates, balance rise/fall, and ensure noise margins. Run DC/AC and transient SPICE simulations.
Why it matters: determines performance, power and area tradeoffs of the cell.
Practical notes: demonstrate sizing heuristics: increase W/L for stronger drive but watch capacitance.

#### Layout design step
Concept: create layout in Magic or other layout editor: adhere to cell height/grid, place transistors, poly gates, diffusion taps, contacts, metal routing, and create well/substrate connections. Ensure DRC compliance.
Why it matters: layout defines parasitics and manufacturability.
Practical notes: keep pin locations predictable and consistent for automatic placement; follow PDK-defined routing tracks.

#### Typical characterization flow
Concept: extract parasitics (RC) from layout (SPEF/SPICE) → run SPICE timing simulations across PVT corners → generate Liberty .lib timing and power tables for use by STA and synthesis.
Why it matters: library cells must carry corner-specific timing/power for accurate signoff.
Practical notes: explain corners (TT, SS, FF, FS, SF) and voltage/temperature vectors.

#### Timing basics (general)

#### Timing threshold definitions
Concept: thresholds include setup time, hold time, contamination delay, clock-to-Q delay, propagation delay — defined for sequential and combinational elements.
Why it matters: essential to compute if flip-flops capture correct data on clock edges.
Practical notes: link threshold concept to STA equations Tclk >= TclktoQ + Tcomb + Tsetup + Tskew.

#### Propagation delay and transition time
Concept: propagation delay = time between input change and output reaching steady logic threshold; transition time (slew) = time for signal to move between defined voltage thresholds (e.g., 10% → 90%). Both affect timing and crosstalk sensitivity.
Why it matters: slew influences cell delay (non-linear) and timing constraints; slow edges enlarge coupling effects.
Practical notes: show how .lib cells have delay tables indexed by input slew and output load.

### Day 3 — Layout, ngspice characterization, CMOS inverter labs

#### IO placer revision
Concept: iterate padframe/pin placements to ensure shortest routes, minimal crossovers, and accommodate package constraints. Place IO macros and connect to rails.
Why it matters: poor IO placement ruins board routing and worsens timing.
Practical notes: map package pins → die pads → PCB netlist.

#### SPICE deck creation for CMOS inverter
Concept: SPICE deck = transistor netlist, device models (sky130 spice models), voltage sources, stimuli, and measurement probes. For inverter include transistor sizes and load.
Why it matters: base for functional and timing characterization.
Practical notes: include .include sky130.lib.spice, .option, and .tran plus .measure statements to capture delay and switching points.

#### SPICE simulation lab for CMOS inverter
Concept: run DC sweep, transient switching, and param sweeps for load/sizing/temperature. Measure VOH/VOL, rise/fall times, propagation delays, and dynamic power.
Why it matters: validates cell behavior before layout extraction.
Practical notes: show how to extract switching threshold (VM), and plot input vs output.

#### Switching Threshold Vm
Concept: Vm (or V_M) is the input voltage at which the inverter output equals the input (Vout = Vin), roughly the switching point and indicates noise margin and sizing balance.
Why it matters: tells if inverter is balanced (Vm ≈ Vdd/2 for symmetric).
Practical notes: measure from DC transfer curve; adjust Wp/Wn to shift Vm.

#### Static and dynamic simulation of CMOS inverter
Concept: static (DC) covers logic levels, leakage; dynamic (transient) covers switching, propagation delay, power during transitions. Both necessary for full characterization.
Why it matters: static shows logic correctness; dynamic shows performance and power.
Practical notes: dynamic power ∝ C·V²·f; include parasitic loads to be realistic.

#### Lab steps to git clone vsdstdcelldesign
Concept: show repo clone, inspect directory, required submodules or PDK links, and run included testbenches & scripts.
Why it matters: students can reproduce the inverter design pipeline.
Practical notes: check README for PDK instructions; advise using container/docker environment.

#### Inception of layout — CMOS fabrication process (conceptual steps)

#### Create Active regions
Concept: define diffusion/active areas (where transistors form) via masks that protect regions for doping/implant.
Why it matters: active zones determine where source/drain will be formed.

#### Formation of N-well and P-well
Concept: wells isolate p-type and n-type regions to form PMOS/NMOS bodies; well implant and masking define their boundaries.
Why it matters: correct well ties critical for threshold control and body effects.

#### Formation of gate terminal
Concept: gate poly deposition and patterning create MOS transistor gates; gate length controls channel behavior.
Why it matters: lithography limits minimum gate length, affecting performance.

#### Lightly doped drain (LDD) formation
Concept: LDD reduces hot-carrier effects near gate by using lighter doping near gate edges then heavier doping further out.
Why it matters: improves reliability and reduces short-channel effects.

#### Source – drain formation
Concept: heavier implants form low-resistance source/drain regions, followed by contacts and silicidation for reduced contact resistance.
Why it matters: important for drive strength and series resistance.

#### Local interconnect formation
Concept: local metal or poly connections within cell rows (M0 or local interconnect) link transistors to create gates and nets.
Why it matters: first metal levels are precious for dense routing; correct local interconnect keeps cells compact.

#### Higher level metal formation
Concept: multiple metal routing layers provide global interconnect, power, and signal routing; vias connect layers.
Why it matters: determines routing stack and via reliability.

#### Lab introduction to Sky130 basic layers layout and LEF using inverter
Concept: map logical layers to PDK layers, understand layer names (e.g., poly, diff, metal1) and create LEF describing geometry and pins for the standard cell.
Why it matters: LEF is required for place & route to know pin geometry and macro shapes.
Practical notes: show how to generate .lef from Magic layout.

#### Lab steps to create std cell layout and extract spice netlist
Concept: create cell in Magic using sky130.tech, run extract to generate netlist with parasitic estimates (RC), produce .spice for LVS/extraction.
Why it matters: extracted netlist used for post-layout timing/power and LVS comparison.
Practical notes: run cif2spice or magic extract and then run ngspice.

#### Sky130 Tech File Labs (PDK specific exercises)

#### Lab steps to create final SPICE deck using Sky130 tech
Concept: combine extracted netlist, PDK device models, include corners, and stimulus to generate full SPICE deck for cell or block.
Why it matters: ensures sims use same models as fabrication.
Practical notes: include lightning checks like mismatched model names and correct .include paths.

#### Lab steps to characterize inverter using sky130 model files
Concept: run corners and parameter sweeps (load, temp, voltage) using sky130 models; produce timing & power numbers.
Why it matters: yields accurate .lib.
Practical notes: script simulations to run TT/SS/FF corners; capture times with .measure.

#### Lab introduction to Magic tool options and DRC rules
Concept: Magic has DRC rules derived from PDK; learn rule sets, viewing options, and commands to check/repair violations.
Why it matters: prevents fabrication fails by catching geometric violations early.
Practical notes: use drc check, fix commands, and learn rule naming conventions.

#### Lab introduction to Sky130 PDKs and steps to download labs
Concept: SkyWater’s PDK includes tech files, SPICE models, layer maps, and LEF; often available via foundry channels or curated Git repos; follow licensing and access steps.
Why it matters: must have correct PDK to run layout and extraction.
Practical notes: use git clone of skywater-pdk (if available) and follow its README.

#### Lab introduction to Magic and steps to load Sky130 tech-rules
Concept: configure MAGICRC to point to sky130 tech files, then load the tech and run drc and extract with sky130 ruleset.
Why it matters: proper tech ensures layer mapping and DRC behavior is PDK-correct.

#### Lab exercise to fix poly.9 error in Sky130 tech-file
Concept: poly.9 likely refers to a specific DRC rule about poly spacing/overlap; lab requires diagnosing geometry causing violation and patching layout to satisfy rule.
Why it matters: teaches rule interpretation and layout fixes (e.g., adjust poly spacing or add implant).
Practical notes: show step: view violation highlight → change geometry → re-run DRC.

#### Lab exercise to implement poly resistor spacing to diff and tap
Concept: ensure poly resistors are isolated from diffusion/tap with required spacing and shields; sometimes need guard rings or diffusion implants.
Why it matters: avoids unintended junctions and preserves resistor properties.
Practical notes: add guard diffusion and contacts per PDK.

#### Lab challenge exercise to describe DRC error as geometrical construct
Concept: translate DRC code (rule ID) into geometry description (what layers, spacing, width and enclosure parameters failed).
Why it matters: students learn to read DRC logs and propose geometric fixes.
Practical notes: show mapping from rule to geometry and suggest minimal fix.

#### Lab challenge to find missing or incorrect rules and fix them
Concept: sometimes tech file mismatches or user-made stack-ups cause false DRCs. The lab is to compare PDK rules with tool config and correct maps.
Why it matters: avoids hours of pointless DRC failures.
Practical notes: maintain version control for tech files and document changes.

### Day 4 — Timing, delay tables and clock trees

#### Lab steps to convert grid info to track info
Concept: grid = geometric coordinates; tracks = discrete routing wires per metal layer. Conversion maps geometry to routing tracks usable by placer/router.
Why it matters: correct track definitions ensure routing uses intended tracks and respects spacing.
Practical notes: create tracks.lef or include track definitions in tech LEF.

#### Lab steps to convert Magic layout to std cell LEF
Concept: export cell boundary, pin shapes, and site definitions into LEF format from layout tool so placer knows pin locations and cell size.
Why it matters: LEF is required for physical tools to place and route cells.
Practical notes: use lef write commands or scripts to create accurate pin geometry.

#### Introduction to timing libs and steps to include new cell in synthesis
Concept: .lib files present timing arc data to synthesis and STA. To include new cell: add .lib, LEF and gate-level view in library catalog and rebuild synth database.
Why it matters: synthesis will infer gates correctly and estimate timing if libs are present.
Practical notes: update Yosys/abc mapping and ensure cell names match across LEF/Lib/Vendor names.

#### Introduction to delay tables
Concept: delay tables map cell delay as a function of input slew and output load (two-dimensional LUT). Used by STA and synthesis to estimate delays.
Why it matters: accurate delay tables produce accurate STA results.
Practical notes: show small example table and how to index it in STA.

#### Delay table usage Part 1
Concept: how STA queries delay for a particular input slew & output load to calculate arc delays; also how negative/positive slacks are computed.
Why it matters: designers manipulate cell sizing or buffer insertion based on that feedback.
Practical notes: mention interpolation between table points.

#### Delay table usage Part 2
Concept: effect of PVT corners on delay tables and how multiple libraries are used for corners; combining delay tables with parasitic data yields final timing.
Why it matters: ensures worst-case and best-case timing checks are valid.
Practical notes: show how to point STA to corner-specific .lib.

#### Lab steps to configure synthesis settings to fix slack and include vsdinv
Concept: change effort level, target-frequency, register balancing, and allow use of custom vsdinv (inverter cell) by adding to cell library and mapping. Re-run synthesis for improved slack.
Why it matters: synthesizer choices determine initial timing quality.
Practical notes: adjust set_max_area, pipeline insertion, and retiming options if supported.

#### Timing analysis with openSTA, clock jitter, ECOs & CTS

#### Setup timing analysis and introduction to flip-flop setup time
Concept: STA setup analysis checks data arrives before flip-flop setup window; setup time is required stable time before clock edge. Configure clocks, input delays, and timing libraries in OpenSTA.
Why it matters: failing setup means functional timing error at the given clock frequency.
Practical notes: show OpenSTA commands to read libs, read verilog/netlist, and set clocks.

#### Introduction to clock jitter and uncertainty
Concept: jitter = temporal variation of clock edge; uncertainty includes clock path skew and source jitter. STA uses uncertainty to tighten constraints (i.e., effective clock period smaller).
Why it matters: jitter reduces available timing margin and must be accounted for in worst-case checks.
Practical notes: show how to add set_clock_uncertainty or margin in SDC.

#### Lab steps to configure OpenSTA for post-synth timing analysis
Concept: load post-synthesis netlist, .lib files, SPEF parasitics if available, set clocks and generate timing reports (report_timing).
Why it matters: verifies synth stage meets timing before placement.
Practical notes: include read_spef or parasitic info when available for better accuracy.

#### Lab steps to optimize synthesis to reduce setup violations
Concept: strategies: increase optimization effort, re-balance logic, re-synthesize critical modules, add pipeline stages, or use faster cells.
Why it matters: fixes become progressively costlier later in flow.
Practical notes: show practical changes: adjust -opt flags, or constraint-specific clock regions.

#### Lab steps to do basic timing ECO
Concept: ECO (engineering change order) modifies netlist post-synthesis to fix timing without full resynthesis: change routing, add buffers, tweak constraints, or replace cells.
Why it matters: quick fixes to meet timing late in flow.
Practical notes: demonstrate inserting buffers on critical nets, or hand-modifying netlist with set_dont_touch.

#### Clock tree synthesis TritonCTS and signal integrity
Concept: TritonCTS builds buffered hierarchical H-tree or other topologies to distribute clocks with target skew and slew. Signal integrity concerns include shielding and avoiding crosstalk into clock nets.
Why it matters: clocks must be low-skew and robust; poor CTS causes timing failures.
Practical notes: show CTS constraints: root clocks, buffer choices, and insertion points.

#### Crosstalk and clock net shielding
Concept: crosstalk = coupling between signals, which can create delayed or spurious transitions. Shielding (routes, guard wires, using separate layers) reduces it. For clock nets, shielding reduces jitter and skew.
Why it matters: crosstalk can break timing reliability especially in dense routing.
Practical notes: use same-layer shielding or assign clock to middle metal layers with low coupling and add guard wires.

#### Lab steps to run CTS using TritonCTS
Concept: configure buffer libraries, clock root, insertion regions; run TritonCTS and inspect clock tree DEF output that shows buffers and net names.
Why it matters: practical experience with CTS tool & interpreting its outputs.
Practical notes: view CTS buffer tree in Magic or OpenROAD viewer to verify sinks and clock sinks pins.

#### Lab steps to verify CTS runs
Concept: run post-CTS STA: check clock tree skew, latency, and setup/hold margins. Verify clock insertion delay and ensure no hold issues introduced.
Why it matters: CTS affects both setup and hold; must re-run STA and adjust.
Practical notes: use report_clock -skew and re-run report_timing.

#### Setup timing analysis using real clocks
Concept: real clocks include insertion delay from root to flip-flop; STA uses actual clock nets and CTS results to compute true timing windows.
Why it matters: more accurate than ideal clocks — reveals missed paths due to insertion delay.
Practical notes: read post-CTS netlist and re-run STA with create_clock based on physical clock path.

#### Hold timing analysis using real clocks
Concept: hold checks ensure data doesn't change too soon after the capturing edge. CTS changes insertion delays and can introduce hold violations that must be fixed with buffers or delay inserts.
Why it matters: fixing hold issues typically done after CTS and can be localized.
Practical notes: use hold-focused optimization: increase delay on data path (buffers) or reduce clock skew where beneficial.

#### Lab steps to analyze timing with real clocks using OpenSTA
Concept: load timing libraries, post-CTS netlist, SPEF, then run report_timing -hold and report_timing -setup to get accurate violations and traces.
Why it matters: gives final timing picture before routing.
Practical notes: walk through read_spef, read_liberty, report_timing commands.

#### Lab steps to execute OpenSTA with right timing libraries and CTS assignment
Concept: ensure OpenSTA points to corner-specific .lib and the CTS-generated netlist/DEF to compute true timings.
Why it matters: incorrect library usage gives misleading slack values.
Practical notes: maintain mapping between cell names in .lib and netlist.

#### Lab steps to observe impact of bigger CTS buffers on setup and hold timing
Concept: larger buffers reduce clock tree insertion delay, might decrease skew but increase latency — affecting setup; can also change local skews influencing hold.
Why it matters: shows tradeoffs: faster buffers may help setup but worsen hold or increase power.
Practical notes: run param sweep with buffer sizes and plot resulting slacks.

### Day 5 — Routing, DRC, PDN and TritonRoute

#### Introduction to Maze Routing – Lee’s algorithm
Concept: Lee’s algorithm finds shortest path on grid by breadth-first search expanding wavefront until sink reached, then backtracks to build route. Guarantees a solution if one exists but can be slow and produce wide routes.
Why it matters: foundational for understanding grid-based routing and rip-up & reroute strategies.
Practical notes: mention computational complexity and memory usage.

#### Lee’s Algorithm conclusion
Concept: while robust, Lee’s algorithm is not scalable for full-chip routing; modern routers use heuristics, rip-up-and-reroute, and maze variants for speed.
Why it matters: relates to why tools like TritonRoute use more advanced techniques.

#### Design Rule Check (DRC)
Concept: DRC verifies layout geometry against foundry rules (spacing, width, enclosure, overlaps). It’s a gatekeeper for manufacturability.
Why it matters: failing DRC prevents tapeout; must be resolved before GDS submission.
Practical notes: use Magic/klayout to view errors and correct geometry.

#### Lab steps to build power distribution network
Concept: create hierarchical power strap design: global rails, stripes, and local cell-level connections; define via arrays and stitch points.
Why it matters: ensures low-impedance supply across die & avoids IR-drop problems.
Practical notes: show how to reserve routing layers for power and place wide metal straps for VDD/VSS.

#### Lab steps from power straps to std cell power
Concept: connect global straps to power pins of each standard cell via rails/tracks and ensure adequate via redundancy. Add decoupling placements if needed.
Why it matters: cell-level power connection must be robust and symmetric.
Practical notes: ensure power ring around core and even distribution across rows.

#### Basics of global and detail routing and configure TritonRoute
Concept: global routing decides which routing regions and layers nets will occupy (coarse), detail routing actually assigns exact tracks/wires and handles vias. TritonRoute can accept guides and LEF/DEF to route with constraints.
Why it matters: correct configuration reduces overflow and improves completion rate.
Practical notes: configure layer preferences, via rules, and guide files to help router.

#### TritonRoute feature 1 - Honors pre-processed route guides
Concept: TritonRoute can accept coarse guides (from global router or tools) and honor them during detailed routing to follow desired routing channels.
Why it matters: improves predictability and routability especially for constrained designs.
Practical notes: generate guides from global router or planner.

#### TritonRoute Feature2 & 3 - Inter-guide connectivity and intra- & inter-layer routing
Concept: handles connectivity between different guides and makes transitions across metal layers using vias while respecting design rules and layer-preference.
Why it matters: ensures multi-layer routing continuity and reduces need for manual intervention.
Practical notes: specify via types and metal layer stacks in config.

#### TritonRoute method to handle connectivity
Concept: incremental rip-up and reroute, conflict resolution, and maze-like local searches to stitch nets across blocked areas or around obstacles.
Why it matters: handles real-world congestion scenarios.
Practical notes: inspect route_summary and overflow logs to tune behavior.

#### Routing topology algorithm and final files list post-route
Concept: routers compute topologies (tree, Steiner) for multi-pin nets, generate final DEF/GDS, SPEF/RCX extracted parasitics, and routing reports. Final files include final.def, final.gds, final.spef, route.rpt, and drc/routing.rpt.
Why it matters: these artifacts are used for signoff (LVS/DRC/timing) and fabrication.
Practical notes: ensure to run extraction and SDF/SPEF generation for STA as final step.
