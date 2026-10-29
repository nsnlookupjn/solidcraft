# CAM research: a shared machine library, or "what can this machine actually push?"

Research for [#1621](https://github.com/tangency-labs/solidcraft/issues/1621) (Research Request
for Plugin: CAM). This file covers one part of that request: **a shared, physics-based library
of machine tools** (for example a Haas VF-2), describing what each machine can do in every axis
(thrust, stiffness, speed, acceleration, spindle torque and power against speed, dynamics,
kinematics, accuracy) and linking each entry to a 3D model of the machine for simulation.

It is the last piece of a set of files from the same author that together describe the
machining process as **one physical system**:

| Topic | File |
|---|---|
| System overview (shared setup object, solver loop, one slider) | [cam-research-nsnlookupjn-physics-system.md](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-vgqo3g/research/cam-research-nsnlookupjn-physics-system.md) |
| Shared tool library | [cam-research-nsnlookupjn-tool-library.md](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-tool-library/research/cam-research-nsnlookupjn-tool-library.md) |
| Cutting tool physics | [cam-research-nsnlookupjn-cutting-physics.md](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-cutting-physics/research/cam-research-nsnlookupjn-cutting-physics.md) |
| Workholding physics and library | [cam-research-nsnlookupjn-workholding.md](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-workholding/research/cam-research-nsnlookupjn-workholding.md) |
| Stock structural integrity during the cycle | [cam-research-nsnlookupjn-stock-integrity.md](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-stock-integrity/research/cam-research-nsnlookupjn-stock-integrity.md) |
| **Machine library (this file)** | cam-research-nsnlookupjn-machine-library.md |

The system overview owns the parts shared by all topics. This file owns the machine model and
its interface rows with the other files.

Licence: MIT OR Apache-2.0, the same as SolidCraft (see the Licence section of CONTRIBUTING.md).

## About this research

- **Author:** Jason ([@nsnlookupjn](https://github.com/nsnlookupjn)). Idea and direction by Jason;
  source research and writing done with an AI assistant (Claude) in October 2026.
- **How it was produced:** web searches of public sources (published papers and their
  abstracts, standards' public catalogue entries, CAM vendors' public help pages and blogs,
  machine dealers' spec listings, trade press, user forums), each claim checked against the
  cited page where possible. No commercial CAD/CAM program was driven or scripted, no
  GPL/LGPL/AGPL source was read (LinuxCNC is cited only through its public documentation and
  forum, never its code), and no text is copied beyond short quotes. Standards are described
  from their public catalogue entries, not their paid text. Worked examples are textbook
  mechanics computed for this file.
- **Units:** millimetres, newtons, newton-metres, watts, hertz, radians unless a source is quoted
  in other units (then converted: 1 lbf = 4.448 N, 1 ft·lbf = 1.356 N·m, 1 hp = 0.746 kW).
- **Marking:** "(unverified)" marks a claim not confirmed against a source. **Fact**,
  **opinion** and **marketing** are labelled where they mix. The Haas VF-2 is used as the
  running example because Jason named it; its numbers come from a dealer's listing of a 2018
  machine, not from Haas (whose pages could not be fetched), so they are (unverified) against
  the builder and vary by year and option.

**Author's experience (personal experience, not a cited source):**

- **Shop floor since 1987:** a machine shop apprenticeship from 1987 to 1991, starting on manual
  machines, then writing G-code at the control on early CNC machines (Milacron and similar, with
  no way to save a program).
- **CAD/CAM over about 35 years:** most major packages. In automotive work: Unisurf, Siemens,
  Sketchpad and NX, with programs stored on punch cards, then tape, then floppy disks; later
  CATIA, AutoCAD and Pro/Engineer. Currently fluent in Fusion, Mastercam, SOLIDWORKS and NX.
- **Engineering and software:** BTEC HND in Electrical and Mechanical Engineering, MEng, PhD in
  Materials Science, certified welder. Programming since childhood (machine code, BASIC, C++,
  C#, assembler, web and database technologies). Owns engineering and software companies.
- **Current work:** engineering and manufacturing spare parts for defence and aerospace,
  including flight-safety components.

> **[TODO Jason: personal experience]** The machines you run or have run, which of their limits
> you actually meet (spindle stalling or overloading at low RPM, axis overload alarms, chatter
> that moves with table position, corners that slow the cycle), and how you learnt each
> machine's limits. Label this as personal experience.

## Summary

1. **The missing element.** The tool library says what the cutter can take, the workholding
   library says what the hold can take, and the stock integrity work says what the part can
   take as it is cut away. None of them says **what the machine can deliver or resist**. The
   machine is both the source of the force (spindle torque, axis thrust) and one of the springs
   that deflect under it (Jason's observation: "the only missing element is the machine
   library").
2. **What a machine entry holds.** Per machine model: axis travels, kinematics (which axis
   carries what), per-axis **maximum thrust**, **rapid and cutting feed, acceleration and jerk**,
   **static stiffness at the tool point** per direction, the **dominant structural modes**,
   **spindle torque and power against speed** (per gear or winding range, continuous and
   short-time ratings), maximum spindle speed and taper, table load, rotary-axis torque and
   **brake/clamp holding torque**, accuracy and repeatability, and a link to a **3D model** of
   the machine with its moving parts named after its axes.
3. **The spindle curve is a hard limit at every speed.** Torque and power available depend on
   spindle speed. Below the base speed torque is flat and power falls with speed; above it power
   is roughly flat and torque falls (Smith, CTE 2012). A cut that is fine at 3000 rpm can stall
   at 600 rpm. Every cut step, and every speed change suggested for chatter, must satisfy
   `Mc ≤ M_spindle(n)` and `Pc ≤ P_spindle(n)` for the chosen gear range and duty rating
   (Jason's requirement).
4. **The machine is a spring in the chain.** The tool tip moves relative to the part by the sum
   of the deflections of tool, holder, spindle, machine axes, fixture and stock (springs in
   series). A measured 3-axis CNC milling machine was about 15 to 25 N/µm at the tool
   (Røsjordet and Hovland 2019), similar to a slender end mill, so the machine can be half of the
   total deflection. The weakest link sets the feed.
5. **The machine sets real feed and cycle time.** CAM estimates cycle time from path length and
   programmed feed; the control slows down for corners and short segments according to the
   machine's acceleration, jerk and tolerance settings. On a published aerostructure part CAM
   under-predicted cycle time by 36 to 62 % (Ward et al. 2021). A machine library with
   acceleration and jerk lets SolidCraft predict the feed that will really happen, which also
   changes chip load and force.
6. **A shared library, with measured shop instances.** Like the tool library, a machine model
   is described once, collaboratively, from the builder's public specs (catalogue layer), and
   each shop records its own instance (options, measured stiffness, tap-test FRFs, ballbar
   results, wear) on top. Missing or estimated data pulls the conservative-to-aggressive slider
   toward conservative.
7. **Warm up before full torque (Jason's requirement).** A cold spindle has unevenly spread
   lubricant and has not reached its thermal size (Okuma 2020). The machine entry carries its
   warm-up routine and an idle time after which it must run again; until it has, CAM derates
   spindle torque, power and speed, and the slider cannot lift that derating.
8. **Lock a machine to its materials (Jason's requirement).** Many shops have a machine that
   only ever cuts 6061 aluminium, and tool libraries kept for aluminium only. A machine (and a
   tool library) can be locked to one or several materials, or to a range; rarely to all.
   CAM then offers only the machines and tools approved for the part's material.
9. **3D models are the hard part legally.** Users have asked builders for machine models for
   at least 15 years; they are often withheld, paid or only STL (eMastercam forum, 2011). A
   shared library should hold specifications (facts), kinematics and openly licensed or
   community-built simplified models, and point to builder models rather than copy them.

## Fundamentals

### Where the machine sits in the physical system

Jason's description of the system: as the cutter moves through three dimensions it pushes and
pulls the part in different directions against the workholding; the toolpath adjusts in real
time to keep the best tool load in each direction, within what the hold can take; and as stock
is machined away the stock weakens and can deform, so the order of cuts must balance stock
integrity with tool force and workholding force. The machine closes the loop:

```text
          spindle torque/power(n) ── drives ──┐
                                             v
  machine axes ── thrust ──> tool + holder ──> cut ──> stock/part ──> workholding ──> table
      ^                                                                              |
      └──────────────────────── reaction force flows back through the frame ─────────┘
```

Two separate roles:

- **Source (capacity).** The spindle must supply the cutting torque and power at the chosen
  speed; each feed axis must supply the thrust that pushes the tool into the cut (plus the
  forces to accelerate the moving masses and overcome friction). If either is short, the
  spindle slows or stalls, or the axis faults with an overload or following error.
- **Spring (compliance).** The cutting force reacts through the whole structure: tool, holder,
  spindle bearings, head, column, saddle, table, fixture and part. Each part deflects. The tool
  tip's position error is the sum, and the vibration of the whole loop decides chatter.

### Spindle torque and power against speed

- **Physics.** Power `P = M · ω = M · 2π n / 60` (W, N·m, rpm). Cutting power from the cutting
  physics file is `Pc = Fc · vc`, cutting torque `Mc = Fc · D / 2`.
- **Curve shape (fact).** Two common spindle designs: constant torque over the speed range (power
  grows about in proportion to speed) or constant torque up to a point and then roughly
  constant power with falling torque. Lower-speed spindles usually have more torque and reach
  peak power at a lower speed. "There is little power available at low spindle speeds"; short
  overloads are absorbed by the spindle's inertia, but sustained power above the curve stalls
  it (Smith, *Cutting Tool Engineering*, 2012).
- **Duty ratings (fact).** Spindle motors are rated for continuous duty (**S1**) and for
  intermittent duty such as **S6-40 %** (loaded 4 minutes in each 10). Example: 7 kW S1 and
  10.5 kW S6-40 %; the higher figure is only available if heavy cutting takes no more than about
  40 % of the time (CADEM). The brochure number is often the short-time one (opinion, common
  experience; unverified per builder).
- **Gear and winding ranges.** Some spindles have a gearbox or a motor with switchable windings
  (wye/delta), giving two or more curves; the high-torque range has a lower maximum speed.
  The range change costs time and may not be allowed during a cut (unverified per control).
- **What the library must store (Jason's requirement: respect the torque curves).** Per spindle
  and per gear/winding range: a table of `(n, M_S1(n), P_S1(n), M_S6(n), P_S6(n))` points with
  the duty-cycle definition, the minimum and maximum speed of the range, and how the control
  selects the range. The solver interpolates linearly between points (curves from brochures are
  piecewise straight in torque or power, so this is close).
- **How CAM uses it.** For each cut step at speed `n`: `Mc ≤ s_M · M_spindle(n)` and
  `Pc ≤ s_P · P_spindle(n)`, where `s` is the slider's fraction (cutting physics file: 60 % /
  80 % / 95 %). The duty rating used depends on the predicted cutting time fraction over a
  10-minute window: S6 only while the predicted load pattern fits the rating, S1 otherwise.
  When the chatter or tool-life logic proposes a new speed, the torque and power checks run
  again at the new speed before it is accepted. The curve is a **hard limit**: the slider moves
  the margin, never the curve itself.

### Feed axes: thrust, speed, acceleration and jerk

- **Thrust (fact).** For a ball-screw axis, force from motor torque is
  `F = 2π · M_motor · i · η / p` (gear ratio `i`, efficiency `η`, screw lead `p`) (Kollmorgen).
  Builders publish the result as "maximum thrust" per axis. Example (dealer listing, 2018
  VF-2): X and Y 8.87 kN, Z 13.7 kN (unverified against Haas).
- **What uses up thrust.** Cutting force component along the axis + inertial force
  `m_moving · a` + friction + gravity on a vertical axis (Z carries the head; a counterbalance
  may take part of it). For milling, the feed-direction component is usually well below the
  rated thrust; for **drilling, tapping, plunging and face milling with large cutters**, the Z
  thrust can matter (opinion; see worked example).
- **Speeds.** Rapid traverse and maximum cutting feed per axis (VF-2 listing: 35.6 m/min
  rapid, 21.2 m/min max cutting feed).
- **Acceleration and jerk.** The control limits each axis's acceleration and jerk (rate of
  change of acceleration). Look-ahead and corner blending slow the tool at corners and on short
  segments; the slowdown depends on the path tolerance. CAM programs that assume the programmed
  feed is held under-predict cycle time: by up to 55 % on test pockets and 36 to 62 % on an
  aerostructure part, with real corner feeds as low as 4 % of programmed (Ward, Sencer, Jones,
  Ozturk, arXiv 2102.02062, 2021). Their model with machine dynamics predicted within about 5 %.
- **Why this matters for the physics (opinion).** If the real feed drops at a corner, chip load
  drops (rubbing, more wear) and the force direction changes as the tool turns. A physics CAM
  that assumes programmed feed will be wrong exactly where engagement is highest (corners).
  The machine library's acceleration and jerk let the solver use the feed that will really
  happen.
- **Control settings.** Open controls expose these as configuration values: LinuxCNC's INI has
  per-joint and per-axis `MAX_VELOCITY` and `MAX_ACCELERATION`, and jerk-limited (S-curve)
  planning has been under development (LinuxCNC forum). Closed controls have equivalent
  parameters, often visible to the user only as a "smoothing" or "high-speed machining" mode.

### The control: block processing, look-ahead and accuracy modes

*Jason: "tolerances can drive the toolpath required accuracy making speedier cuts possible on
non critical areas. Features like G187 and High-Speed Machining (HSM code look forward) come
into play here. Arc fitting vs interpolated points to meet tolerances. Consider the line
processing speed of the machine and try to stay away from processing limits so machine is
always smooth."*

The tolerance budget and the choice of CAM tolerance per face are in the
[in-process inspection file](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-y2w06n/research/cam-research-nsnlookupjn-in-process-inspection.md) (section "Tolerance-driven toolpath accuracy"). This file
holds the **machine and control facts** that choice depends on:

- **Block processing rate.** A control executes a limited number of blocks per second and reads
  a limited number of blocks ahead. Demand is `feed (mm/min) / 60 / segment length (mm)` blocks
  per second: 6000 mm/min on 0.05 mm segments needs 2000 blocks/s, on 0.2 mm segments 500
  blocks/s. Above the control's rate it slows, hesitates and marks the part. Rates and look-ahead
  depth vary widely by control and option and are rarely published (unverified per model), so
  the entry stores a published or **measured** value with its source.
- **Smoothness margin.** CAM keeps block demand below a fraction of the rate, by default 70 %
  (Jason: stay away from the processing limit), and the slider does not raise it above the rate.
  Segments over the margin are fixed by arc or spline fitting first, then merging points within
  the face's allowed tolerance, and only then by lowering feed (inspection file).
- **Drip-feed (DNC) link speed.** A program streamed over a serial line has a second limit: at
  115,200 baud and 10 bits per character about 11,520 characters/s, roughly 380 blocks/s at 30
  characters per block before handshake pauses (arithmetic, inspection file). Network transfer
  and memory size decide whether a program can be loaded whole instead.
- **Accuracy / high-speed modes.** Which smoothing modes the control has and their parameters:
  Haas **G187** P1/P2/P3 (rough, medium, finish; default from Setting 191, optional E corner
  limit, unverified detail); Heidenhain **Cycle 32 TOLERANCE** (path tolerance T, HSC mode 0
  accuracy / 1 speed, rotary tolerance TA); Fanuc AI contour / high-speed high-precision modes
  (G05.1 and related, option-dependent, unverified); Siemens **CYCLE832** (unverified details).
  The post emits the mode chosen from the tolerance budget at each operation.
- **Arcs and splines.** Whether the control accepts G2/G3 arcs in each plane (XY, XZ, YZ; usually
  only principal planes, not in simultaneous 5-axis moves), helical arcs, and spline/NURBS input
  (option-dependent, unverified per control). CAM arc-fits by default where the planes allow it
  and uses splines only where the entry says they exist.
- **Link to real feed.** Block rate, look-ahead and the smoothing mode, together with axis
  acceleration and jerk above, decide the feed the control really runs (Ward et al. 2021), which
  is what the cutting physics should use.

### Rotary axes, tables and the workholding chain

- **Rotary table torque and brake.** A 4th or 5th axis has a drive torque (to index under load)
  and a **brake or clamp** holding torque (for cutting with the axis locked). Example: a Haas
  HRT210 listed with 160 ft·lbf (217 N·m) drive torque and 200 ft·lbf (271 N·m) pneumatic brake
  (dealer listing, unverified). The workholding file treats the brake as part of the hold:
  the cutting moment about the rotary axis must stay under both the vise's limit and the brake's
  limit, with the safety factor.
- **Trunnions are softer.** Measured stiffness of a 2-axis trunnion table was 7.4 to 68.8 N/µm
  depending on direction and whether brakes were engaged; the combined 5-axis chain was
  estimated at 5.3 to 16.9 N/µm (Røsjordet and Hovland 2019). Locking the brake during
  positional (3+2) work makes the chain stiffer.
- **Table load limit.** Maximum mass on the table (VF-2 listing: 680 kg). Fixture + vise + stock
  must fit, and the moving mass reduces the acceleration available on that axis (opinion).
- **Mounting pose.** Where the hold sits on the table, pallet or trunnion (position and
  orientation in machine coordinates). The workholding file needs it to express cutting forces
  in the hold's frame; the machine model supplies the frames, the setup supplies the pose.

### Static stiffness and the series chain

- **Measured values (fact).** On a 3-axis CNC milling machine, LVDT measurements gave about
  14.8 to 15.8 N/µm in X, 18.4 to 21.1 N/µm in Y and 24.7 N/µm in Z; an industrial robot was
  0.25 to 1.1 N/µm (Røsjordet and Hovland, *Modeling, Identification and Control* 40(1), 2019).
  The same paper found laser trackers and dial indicators not accurate enough to measure a stiff
  CNC; an LVDT with a force sensor was the most promising method.
- **Series springs.** Everything between tool tip and part deflects under the same force, so
  compliances add: `1/k_total = 1/k_tool + 1/k_holder + 1/k_spindle + 1/k_machine + 1/k_fixture
  + 1/k_stock` per direction. The deflection is `δ = F / k_total`. The weakest link dominates
  (agreed interface with the stock integrity file).
- **Varies with position.** Machine stiffness and dynamics change with axis positions (a table
  at the end of its travel, a long Z extension) (position-dependent dynamics literature, e.g.
  MM Science Journal 2017 state-space models). Some studies on machine tools assume no change
  within the work volume while robots change strongly with pose (Swan et al. 2024). Opinion:
  store a single value first, and an optional stiffness map over the work envelope later.
- **Standards.** ISO 230-1 (geometric accuracy under no-load or quasi-static conditions) is the
  test code family for machine tools; static stiffness tests are defined in machine-type
  standards (unverified which part covers compliance; the paid text was not read).

### Dynamics: tool-point FRF and chatter

- **What it is.** The frequency response function (FRF) at the tool point describes how much
  the tool tip moves per newton at each frequency. From it a **stability lobe diagram** shows
  which spindle speed and depth combinations chatter (Schmitz, *Modern Machine Shop*, 2026).
- **Measuring it.** A **tap test**: an instrumented hammer excites the tool, an accelerometer
  measures the response. The result belongs to that tool + holder + spindle + machine
  combination; a different stick-out gives a different FRF.
- **Receptance coupling (RCSA)** splits the problem: measure the spindle-machine once (tap test
  on a standard artefact, inverse RCSA), model the tool and holder as beams, and couple them
  analytically to predict the tool-point FRF for any tool, quickly (Schmitz et al., *Precision
  Engineering* 79, 2023 review; first applied by Schmitz and Donaldson 2000). **This is the
  natural split for a library:** the machine entry (shop instance) stores the spindle-machine
  receptances; the tool library stores tool and holder geometry; SolidCraft couples them.
- **Accelerometer mass** changes the measured FRF on small tools; corrections exist (Özşahin,
  Ertürk, Özgüven, Budak, *IJMTM* 50(6), 2010).

### Kinematics and the 3D model

- **Kinematic chain.** Which axes move the tool and which move the part, in what order, with
  what limits: e.g. VF-2 is X and Y on the table, Z on the head; a trunnion adds A then C on
  the table. CAM needs this to compute axis positions from tool positions (inverse kinematics),
  to check travel limits and singularities, and to know which axis carries which mass.
- **How CAM programs describe it (fact).** Fusion's machine definition holds kinematics,
  capabilities, multi-axis settings, a linked post and an optional linked 3D model; Machine
  Builder links the model's parts to the axes they move with and sets the tool attach point at
  the spindle face (Autodesk help and blog). VERICUT has shipped a "showroom" of machine and
  control models including Haas, DMG, Hermle, Makino, Mazak, Okuma since 2006 (Digital
  Engineering 2006).
- **Open formats.** STEP AP242 can carry kinematic links, pairs (revolute, prismatic, screw...),
  motion ranges and mechanisms; the CAx-IF recommended practice v1.3 (2025-10-20) covers its XML
  form, aimed at automotive and aerospace, not machine tools (no claim of fit; opinion: usable
  for axes and limits). ISO/TS 14649-201:2011 (STEP-NC) defines **machine tool data for
  cutting processes** for milling machines, machining centres, lathes and multi-tasking
  machines, for programming, process planning, simulation and machining-time and energy
  estimates (ISO catalogue). SolidCraft already has mechanism and joint work for its render
  assets (repository `skills/solidcraft-mechanisms/`), which may be reusable for axis
  definitions (opinion; not checked in depth).
- **Getting models.** Users have long asked builders for machine models for simulation: some
  builders supply STEP, some only STL, some charge, some need it negotiated before purchase,
  and many users model their own machine (eMastercam forum, 2011-2012). Third-party sites host
  user-made models of popular machines (e.g. 3dcadbrowser "Haas VF-2" listing), licence often
  unclear.

### Warm-up before maximum torque (Jason's requirement)

- **Why (fact).** When a spindle sits idle its lubricant settles; "the first revolutions of the
  spindle may be dry or unevenly lubricated", which wears bearings, and a cold spindle is
  smaller than a warm one, so the first parts differ from later ones. Okuma recommends "just a
  few minutes" of warm-up, run from the control's calendar timer before the shift, and a
  routine set up with the distributor for each machine (Okuma blog, Heeg, 2020-12-14). The
  article gives no speeds, durations or idle threshold; those come from each builder's manual
  (unverified per builder).
- **What the library stores (opinion).** Per machine model: the builder's warm-up routine as
  steps of `(spindle speed, duration)` plus axis motion if any; the idle time after which the
  routine is needed again (builder value, or a shop setting); and the **cold derating**: the
  fraction of `M_spindle(n)`, `P_spindle(n)` and `n_max` allowed before warm-up is complete
  (default value to be agreed, e.g. 50 % torque and power and the routine's first-step speed,
  (opinion, no source)). The shop instance records when the routine last ran (from the operator
  or MTConnect spindle data).
- **How CAM uses it.** The setup carries a machine state: *cold*, *warming*, *warm*. While not
  warm, the spindle limits are derated, the slider cannot raise them, and the job sheet tells the
  operator to run the warm-up first (or the post adds a warm-up call at the start of the program
  if the shop allows it). Heavy roughing and the tightest finishing tolerances both wait for
  *warm*: the first for the bearings, the second for thermal size. The workholding and system
  files use the same gate.

### Signals the machine publishes (for the closed loop)

Jason's idea (workholding thread): an **edge box at the machine** feeds back vibration, loads and
torques in real time, so the plan can be checked and corrected against what really happens.
What can be measured depends on the control, so each machine entry lists its signals.

- **MTConnect.** Supported natively or through an agent/adapter on many controls (Haas, Mazak,
  DMG Mori, Okuma; Fanuc, Siemens and Heidenhain through an agent) (IronFlock docs). Data items
  are SAMPLE (continuous: position, spindle speed, feed rate, power, temperature), EVENT (mode,
  program, tool change) and CONDITION (normal, warning, fault). Whether a given machine publishes
  spindle or axis load must be checked in its `/probe` response. The agent serves current values
  and a buffered stream over HTTP; sample rates are set by the adapter (not documented in the
  sources read; unverified).
- **OPC UA for Machine Tools (OPC 40501-1, umati).** An information model for monitoring and job
  management: machine and channel state, spindles, tools and tool life (OPC Foundation reference,
  v1.02). Whether spindle load is in its spindle monitoring type was not confirmed.
- **Maker interfaces.** Fanuc FOCAS exposes, among others, `cnc_rdspload` (serial spindle load)
  and `cnc_rdspeed` (actual feed and spindle speed) (Cybus connector docs). Haas has its own data
  collection methods in its control manuals (page could not be fetched; unverified).
- **What the controls do not give.** Control interfaces are polled, typically at tens to
  hundreds of milliseconds (opinion, unverified per control), which is enough for load and power
  trends but far too slow for chatter: tooth-passing frequencies are hundreds of hertz to kilohertz,
  so **vibration needs an added accelerometer** (or microphone) sampled at kilohertz rates in the
  edge box (opinion, from `f_tp = n · z / 60`: 10000 rpm × 3 teeth = 500 Hz).
- **What the library stores (opinion).** Per machine model and control version: protocol(s);
  each published signal (spindle load, spindle power, axis load per axis, actual feed, actual
  speed, temperatures, alarms) with unit, meaning (e.g. "% of S1 rating") and typical update
  rate; and per shop instance the endpoint and any added sensors (type, position, sample rate).
  The closed loop uses only signals the entry says exist, and labels everything else as
  predicted, not measured.
- **Aligning data with the G-code (Jason's requirement, "Self-tuning CAM Operations").** A
  measured spike is only useful if it can be placed on the toolpath: which block, which tool,
  which position, which planned force. So each signal entry also says whether the control
  publishes, **with the same timestamp**, the executing program, block or line number and the
  machine position, and at what rate. MTConnect has EVENT items for the program and the block
  being executed and SAMPLE items for axis positions, each observation timestamped by the agent
  (MTConnect model; whether a given machine fills the line-number item is (unverified) per
  control). Where the control's rate is too slow, the edge box interpolates position between
  samples using the planned path and the actual feed, and an added sensor's high-rate stream is
  aligned to the control's slower one by timestamp (opinion). The library records, per control,
  the best alignment it allows: *per block*, *per position sample*, or *per time only*.
- **Uses.** Calibrating `kc` and the torque curve from measured spindle power; detecting chatter
  from the vibration spectrum near `f_tp`; checking that axis load stays under thrust; logging the
  measured load per operation into the part traveller (in-process inspection file, unverified link).

### Accuracy, thermal growth and wear

- **Positioning accuracy and repeatability** of linear and rotary axes are tested to ISO 230-2
  and, for machining centres, ISO 10791-4 (catalogue entries). Circular (ballbar) tests are
  ISO 230-4 (unverified, catalogue not opened).
- **Thermal growth.** Spindle and structure grow as they warm, moving the tool by tens of
  micrometres; ISO 230-3 covers thermal effects (catalogue; magnitudes vary by machine,
  unverified).
- **Wear.** Ball screws, guides and spindle bearings wear: backlash and lost stiffness show up as
  worse ballbar plots and lower measured stiffness. Opinion: the shop instance, not the model,
  holds these, with a date.

### Worked examples (computed for this file)

All with the 2018 VF-2 dealer-listing figures: 22.4 kW, 122 N·m peak torque, 8100 rpm, X/Y
thrust 8.87 kN, Z thrust 13.7 kN. The listing gives peak torque "at 2000 rpm"; constant power
22.4 kW meets 122 N·m at `n = 22400 · 60 / (2π · 122) ≈ 1750 rpm`, so the listing's torque and
power are probably different duty ratings (unverified). Assume for the example: 122 N·m flat up
to 1750 rpm, 22.4 kW flat above.

**1. A face mill at low speed is torque-limited, not power-limited.** 63 mm face mill in steel
at `vc = 120 m/min`: `n = 120000 / (π · 63) ≈ 606 rpm`. Available torque 122 N·m, available power
`122 · 2π · 606 / 60 ≈ 7.7 kW`, a third of the brochure 22.4 kW. With the balanced slider (80 %)
the cut may use 6.2 kW. A programmer who sizes the cut from "22 kW" overloads the spindle at this
speed. At `vc = 250 m/min` (`n ≈ 1260 rpm`) available power is 16 kW. The torque curve, not the
headline power, sets the cut.

**2. Z thrust while drilling.** 20 mm drill in steel, feed 0.25 mm/rev, `kc ≈ 2000 N/mm²`.
A textbook estimate of feed force is roughly `Ff ≈ 0.5 · kc · (D/2) · f · sin κ` with point
half-angle `κ = 59°`: `0.5 · 2000 · 10 · 0.25 · 0.857 ≈ 2.1 kN` (order of magnitude only; real
drills vary with web and point geometry). That is 16 % of the 13.7 kN Z thrust: fine. A 50 mm
U-drill or heavy plunge roughing can reach the axis limit; the check costs nothing.

**3. How much of the deflection is the machine?** 12 mm solid carbide end mill, 36 mm stick-out,
modelled as a cantilever with effective diameter `0.8 · D = 9.6 mm`, `E = 600 GPa`:
`I = π · 9.6⁴ / 64 ≈ 417 mm⁴`, `k_tool = 3 E I / L³ = 3 · 600000 · 417 / 36³ ≈ 16 N/µm`.
Machine `k ≈ 15 N/µm` (Røsjordet and Hovland, X), vise and part `≈ 50 N/µm` (assumed). Series:
`k_total = 1 / (1/16 + 1/15 + 1/50) ≈ 6.7 N/µm`. A 200 N radial force deflects the tool tip
`200 / 6.7 ≈ 30 µm`, of which the machine is `200/15 ≈ 13 µm`, **45 %**. A CAM that models only
the tool would predict 12 µm and leave a finishing pass out of tolerance.

**4. Rotary brake.** Cutting force 1500 N, 80 mm from the A-axis: moment 120 N·m. HRT210 brake
271 N·m (unverified); with a safety factor of 2 the allowed moment is 135 N·m, so the cut is
just inside. A deeper cut at a larger radius needs a lower load or a different setup.

## Accuracy

- **Builder specs are maxima, not guarantees.** Maximum thrust, torque and power are peak
  values (often short-time). Real available values depend on duty, voltage, temperature,
  machine age and options. Use the S1 curve unless the duty pattern supports S6.
- **Dealer listings are second-hand data.** The VF-2 figures here come from a dealer, and vary
  by year and option (e.g. 8100 rpm vs 10000 / 12000 rpm spindles, CT40 vs BT40). The library
  must record model year range, options and source for every number.
- **Stiffness is rarely published.** Builders seldom give tool-point stiffness. Measured values
  in papers are for specific machines (Røsjordet and Hovland did not test a VF-2). Default
  values by machine class must be labelled as estimates.
- **Dynamics are specific.** An FRF holds for one spindle, one holder, one stick-out and often
  one axis position; RCSA reduces measurements but still needs a measured spindle.
- **Cycle-time and feed prediction** with acceleration and jerk is within about 5 % in research
  (Ward et al. 2021) but needs the control's interpolator behaviour, which closed controls do not
  publish.
- **Accuracy of the chain.** Combining tool, machine, fixture and stock stiffness in series is a
  linear, quasi-static estimate; it misses joint non-linearity, preload and damping (opinion).

## Programs

- **Fusion (Autodesk):** machine definitions with kinematics, capabilities, multi-axis settings,
  a linked post processor and optional linked 3D model; a machine library with a vendor library
  folder and "My Machines" stored locally or in the cloud; Machine Builder to link model parts to
  axes (Autodesk help, blog). The public pages reviewed do not mention spindle curves, axis
  thrust or stiffness in the definition (absence not confirmed).
- **VERICUT (CGTech):** machine and control simulation with a showroom of builder machines since
  version 6.0 (2006); separate feed optimisation (see the cutting physics file).
- **Mastercam, CAMplete, CIMCO and others:** machine simulation from user or builder models
  (eMastercam forum threads); details not reviewed.
- **CloudNC (CAM Assist):** has a "machine properties" help page; its content could not be
  fetched, so what it stores is (unverified).
- **Open standards:** ISO/TS 14649-201 machine tool data; STEP AP242 kinematics; MTConnect
  describes devices, components (axes, spindle, rotary), data items such as load, position and
  feed overrides, and a Specifications configuration with maximum, minimum and nominal limits
  (MTConnect model browser). Opinion: MTConnect is the natural way to **read back** spindle and
  axis load to calibrate the library.
- **Open-source controls:** LinuxCNC (configuration documented publicly; code not read).

## Complaints and requests

- **Machine models are hard to get.** "It sure would be nice if machine tool manufacturers would
  supply basic 3d models of their machines" (eMastercam forum, 2011); builders slow or unwilling,
  models paid or STL only, advice to negotiate them before the purchase order. Users proposed a
  shared folder of machine models by manufacturer.
- **CAM cycle times are wrong.** CAM's estimate ignores the control's corner slowdowns and gets
  worse as tolerance tightens (Ward et al. 2021). Trade press on "closing the loop between CAM
  and the shop floor" points the same way (IMAGINiT, not reviewed in depth).
- **Spindle stalls at low speed.** The CTE article advises programmers to check programmed
  removal rate against the power curve because there is little power at low speed (Smith 2012).
- **Chatter that changes with setup.** Tap testing is recommended because each tool, holder and
  machine combination differs (Schmitz, MMS 2026; MSC article).
- **Novices cannot see the limits** (Jason's observation, shared with the tool and workholding
  files): a novice does not know that the machine's 30 hp is not available at 600 rpm, or that a
  trunnion is much softer than the table.

> **[TODO Jason: personal experience]** Complaints you have met about machine limits: spindle
> overloads, axis alarms, corner slowdowns, chatter that depends on where the part sits.

## What users need

### A shared (cloud) machine library: two layers, like the tool library

| Layer | Who writes it | Example contents |
|---|---|---|
| **Catalogue: machine model** | community, from builders' public specs; builders could contribute | Haas VF-2 (2018–, 8100 rpm option): travels, kinematics, spindle curves per range, thrust, feeds, acceleration if published, table load, taper, accuracy spec, 3D model link |
| **Shop instance** | the shop | serial or name, options fitted, control version, rotary add-ons, measured stiffness, tap-test or RCSA spindle receptances, ballbar results, measured acceleration/jerk (from a trace), wear notes, MTConnect endpoint |

Each field records its **source** (builder spec, dealer listing, paper, shop measurement), its
**quality** (published, estimated, measured) and a date. Estimated or missing values pull the
slider toward conservative for the limits that use them (the rule in the cutting physics file).

### Locking a machine to one or several materials (Jason's requirement)

*Jason: "In the machine library and tool library be able to lock to one or several materials.
For example many people have a machine that only cuts aluminum 6061 and many people have a
library of tools for only aluminum. It would be possible for a machine or tool library to be
good for a range of materials, but rarely all materials."*

- **Why (opinion, common shop practice; unverified as a survey).** Shops dedicate machines to a
  material to avoid cross-contamination of chips and coolant (aluminium chips in a steel
  machine's coolant, or the reverse; titanium and magnesium fire risk; medical or aerospace
  material segregation rules), because a high-speed, low-torque spindle suits aluminium but not
  steel, and because the coolant concentration, way covers and chip conveyor are set up for one
  chip type. The physics already hint at it: the VF-2 example above only has its full torque
  below about 1750 rpm, which steel needs and aluminium rarely does.
- **Where the lock lives.** On the **shop instance**, not the catalogue model: a VF-2 can cut
  steel, but *this* VF-2 is the aluminium machine. The catalogue model may carry a builder's
  *suitability* note (e.g. a high-speed router "for non-ferrous and plastics"), which is a
  default for the instance, not a lock.
- **Proposed field (opinion, same shape as the tool library's).** `material_scope`: a list of
  entries, each either an ISO 513 group or subgroup (e.g. N, wrought aluminium) or a specific
  grade (e.g. 6061-T6), linked to SolidCraft's physical material cards where one exists, plus a
  mode:
  - `locked`: only these materials. CAM refuses others and says why ("Machine 3 is locked to
    6061-T6, 7075-T6; part material is 304 stainless").
  - `preferred`: others allowed with a warning, logged in the job.
  - empty: no restriction.
- **Combining scopes.** For a job, the allowed set is the intersection of the machine's scope,
  the tool library's scope (tool library file) and, if set, the workholding's scope (e.g. soft
  jaws machined for one part family). If the intersection is empty, CAM says which item excludes
  the material instead of failing silently. When several machines qualify, the ranking in "Using
  the library in the negotiation" picks among them.
- **Effect on the physics.** A material-locked machine can carry **measured, material-specific
  data** with more confidence: measured `kc` for 6061 from its own spindle-load history,
  stability lobes tap-tested with its usual aluminium tools, a warm-up and coolant setup tuned to
  that work. Opinion: data measured on a locked machine is tagged with the material it was
  measured in and is not reused for other materials without a warning.
- **Changing a lock.** A permission, like the tool library's locked fixed load: only a named
  role changes it, the change is logged with a date and reason (e.g. "machine cleaned, coolant
  changed, now steel"), and measured material-specific data stays tagged to its material.

### Machine entry schema (opinion)

Which calculation needs each field: **P** power/torque, **T** thrust, **K** stiffness and
deflection, **C** chatter, **V** feed/cycle time, **G** geometry/collision, **W** workholding,
**H** hard limit.

| Group | Field | Unit | Used by |
|---|---|---|---|
| Identity | builder, model, year range, options, control, source, licence of each part | | |
| Material scope | catalogue: builder suitability note; shop instance: `material_scope` (ISO 513 groups and/or grades) with mode `locked` / `preferred`, editors, change log; machine group membership (shared with the tool library) | – | job filter |
| Kinematics | ordered chain of axes from bed to tool and bed to table: type (linear/rotary), direction, travel min/max, home, which component moves | mm, rad | G V |
| Spindle (per gear/winding range) | `n_min`, `n_max`; table of `(n, M_S1, P_S1, M_S6, P_S6)`; duty definition (e.g. S6-40 % of 10 min); range-change rule | rpm, N·m, W | P H |
| Spindle | taper/interface, max tool mass and length, orientation, coolant (through-spindle pressure) | | G H |
| Feed axes (each) | max thrust `F_axis_max`, rapid, max cutting feed, max acceleration, max jerk, moving mass, counterbalance | N, mm/min, mm/s², mm/s³, kg | T V H |
| Rotary axes (each) | drive torque, **brake/clamp holding torque**, max speed, table load, accuracy | N·m, rad/s, kg | W H |
| Table / pallet | size, slots, max load, mounting frames (for the hold's pose) | mm, kg | W G |
| Static stiffness | `k_X`, `k_Y`, `k_Z` at the tool point (optional map over position); trunnion stiffness with and without brakes | N/µm | K |
| Dynamics | dominant modes per direction `(f_n, ζ, k)`; spindle-machine receptances for RCSA (shop instance) | Hz, –, N/µm | C |
| Warm-up | routine steps `(n, duration)`, idle time before it is needed again, cold derating of `M`, `P`, `n_max`; shop instance: last warm-up time | rpm, s, – | P H |
| Accuracy | positioning accuracy and repeatability per axis (ISO 230-2 / 10791-4 style), ballbar result, thermal drift | µm | H |
| Signals | protocols (MTConnect, OPC UA, maker API); per signal: name, unit, meaning, update rate; block/line number and position published with it (alignment level); shop: endpoint, added sensors and sample rate | –, Hz | closed loop |
| Control | block processing rate (published or measured, with source), look-ahead depth, smoothness margin (default 70 %), DNC link speed and program memory, accuracy/HSM modes with parameters (G187 P1–P3 + E, Cycle 32 T/HSC/TA, CYCLE832, G05.1…), arcs per plane (XY/XZ/YZ, helical), spline/NURBS support, corner behaviour if known | blocks/s, –, baud | V H |
| 3D model | link to model (STEP/STL), part-to-axis mapping, tool attach point, table frame, licence | | G |

### Capability model: which features each machine has, and how we know (Jason's requirement)

*Jason: "The machine library in SolidCraft will need to determine which features and
capabilities each machine has."*

Two VF-2s are not the same machine: one has a 2001 classic control and no probe, another a
2023 NGC with MTConnect, wireless probing and the 8M relay option. What CAM may post and what
Edge may read depends on the **combination** of machine model, control, control software
version and options fitted. The tables of real machines, controls, software versions and their
monitoring and control features live in the
[Edge machine integration research](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-i8m3ue/research/cam-research-nsnlookupjn-edge-machine-integration.md), section
[machine tables](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-i8m3ue/research/cam-research-nsnlookupjn-edge-machine-integration.md#machine-tables-controls-versions-features-and-user-base); its section
[every factory feature, every cut](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-i8m3ue/research/cam-research-nsnlookupjn-edge-machine-integration.md#every-factory-feature-every-cut) lists each control's
smoothing, adaptive feed, chatter, thermal and probing features as seed flags for the groups
below. This section defines the model they fill, and does not repeat them.

**Key.** A capability set belongs to `(machine model, control family and generation, software
version range, options fitted)`. The catalogue holds the sets for each published combination;
the shop instance points at one and adds what is unique to that machine.

**Three layers, from general to individual (Jason: "We must be granular through manufacturer
libraries to extract the most performance from every individual machine configuration and
version"; opinion on the structure):**

1. **Maker library**: per machine model, control version and option, from the maker's published
   data (or contributed by the maker): spindle curves per spindle option, thrust, feeds,
   smoothing modes per software version, interfaces per version.
2. **Configuration**: one buildable combination of model + control version + options fitted
   (e.g. VF-2, NGC 100.20, 12k spindle, 8M relays, wireless probing, HRT210). It is assembled
   from the maker library entries and is what the catalogue lists.
3. **Individual machine**: one serial number in one shop: its configuration, its discovered and
   verified capability flags, and its own **measured physics** learned by Edge (real spindle
   power curve, stiffness, modes, block rate, warm-up behaviour, wear), plus its material scope.

CAM plans from **the most specific layer that has a value**, field by field: a measured value on
this machine beats the configuration's, which beats the maker's generic model value, which beats
a class default. Each value shows which layer it came from, and values from the general layers
count as less certain (pulling toward conservative), so the more a machine is measured the more
performance CAM can safely take from it. This refines the two layers above (catalogue = layers 1
and 2, shop instance = layer 3).

**Capability groups (opinion; each one a list of named flags or parameters):**

| Group | Examples of what is recorded | Who uses it |
|---|---|---|
| Data interfaces | MTConnect (port, version), OPC UA / umati, Haas Ethernet Q commands (`?Q` list, variable read), DPRNT over network or serial, FOCAS, serial DNC, status relays, user relays | Edge: which drivers to start, read-only |
| Monitoring signals | spindle load, axis loads, actual feed and speed, block/line number, positions, temperatures, and their rates (see "Signals the machine publishes") | Edge and closed loop: what can be measured and how well it aligns with the G-code |
| Motion and smoothing | HSM / smoothing modes and parameters (G187, Cycle 32, CYCLE832, G05.1), block rate, look-ahead, arcs per plane, helical, splines, TCPC / 5-axis modes (see "The control") | CAM tolerance and post |
| Load monitor and autofeed | in-control tool load monitor, reaction choices (alarm, feed hold, autofeed: Haas Setting 84), **whether per-tool limits can be set from the program** (unverified for Haas, open question in the integration file) | post: write per-tool load limits from CAM Physics |
| Probing | spindle probe and tool setter fitted, probe cycles available (maker or Renishaw), result variables, DPRNT of results | inspection and setup |
| Machine options | 4th/5th axis model, through-spindle coolant and pressure, chip conveyor, high-speed spindle option, rigid tapping, program memory, network | CAM operations and limits |
| Physics data | everything in "Machine entry schema" above | feed optimiser |

**How each capability is known (provenance), from strongest to weakest:**

1. **Discovered** from the control by Edge: e.g. Haas NGC `?Q100`/`?Q101`/`?Q102` give serial,
   software version and model; MTConnect `/probe` lists the device's components and data items;
   OPC UA servers can be browsed; FOCAS `cnc_sysinfo` gives series, version and axis count
   (integration file; Cybus FOCAS docs). Discovery confirms interfaces and identity well, options
   less well.
2. **Verified by test**: a short test proves a feature works on this machine (e.g. a G187 block
   accepted, a DPRNT line received, a probe cycle returning a value, the integration file's
   "fingerprint" air cut).
3. **Declared by the maker or catalogue**: published spec or software release notes for that
   version (e.g. Haas recommending software 100.20.000.1200 or later for MTConnect, from the
   integration file).
4. **Declared by the user**: options the shop says are fitted.
5. **Unknown**: treated as absent.

Each flag stores its provenance and date. When a discovered value contradicts a declared one
(e.g. a software update), discovery wins and the change is logged.

**How CAM and Edge use it (opinion):**

- **The post emits only what the machine has.** No G187 on a control without it, no arcs in a
  plane it cannot do, no probe cycle without a probe, no DPRNT markers without a channel to
  receive them; a fallback is chosen instead (points within tolerance, manual check, time-only
  alignment), and the job sheet says so.
- **Capabilities gate features, not just code.** Per-tool load limits from CAM Physics are
  written into the program only where the control can take them; otherwise they go to the setup
  sheet for the operator. Tolerance-driven smoothing is only offered where modes exist.
- **Unknown or declared-only capabilities pull toward conservative**, the same rule as missing
  physics data; a feature the program depends on (e.g. a probe cycle that sets a work offset)
  must be at least *verified* before it runs unattended.
- **Edge picks its drivers from the discovered identity and version**, and stays read-only
  (integration file).
- **Machine choice**: capabilities join material scope and physics in ranking machines for a job
  ("needs probing and 4th axis: only Machine 2 qualifies").

### Interface with the other files

Names agreed with the workholding and stock integrity threads (and consistent with the cutting
physics file's "Machine" row).

| Direction | Quantity | Unit | Notes |
|---|---|---|---|
| Machine → cutting physics / feed optimiser | `M_spindle(n)`, `P_spindle(n)` per gear range and duty rating | N·m, W | **hard limit at every cut step and every proposed speed change** |
| Machine → feed optimiser | machine state (cold / warming / warm) and cold derating | – | gates full torque, power and speed; slider cannot lift it |
| Machine → feed optimiser | `n_max`, max cutting feed per axis | rpm, mm/min | hard limits |
| Machine → feed optimiser | `F_axis_max` per axis (minus inertial and gravity share) | N | thrust limit |
| Machine → feed optimiser | max acceleration and jerk per axis, control tolerance mode | mm/s², mm/s³ | real feed along the path |
| Machine → stock integrity / workholding | `k_X`, `k_Y`, `k_Z` at tool point (and table), optional position map | N/µm | series compliance chain |
| Machine → cutting physics (chatter) | modes `(f_n, ζ, k)` or spindle receptances for RCSA | Hz, –, N/µm | tool-point FRF with the tool library's tool and holder |
| Machine → workholding | rotary brake/clamp torque, table load, mounting frames | N·m, kg, frame | brake is part of the hold |
| Machine ↔ tool library / workholding | `material_scope` and machine group; job's allowed set is the intersection | – | refuses or warns, naming the item that excludes the material |
| Machine → CAM tolerance / post (inspection file) | block rate and margin, DNC speed, accuracy modes and parameters, arc planes and spline support | blocks/s, baud | block demand checked along every path; mode chosen per operation from the tolerance budget |
| Machine ↔ Edge integration file | capability set per (model, control, version, options) with provenance; tables of real controls come from the integration file | – | post emits only supported features; Edge picks drivers |
| Machine → simulation | kinematics, travel limits, 3D model and axis mapping | mm, rad | collision and travel checks |
| Cutting physics → machine check | `Mc`, `Pc`, `Fx`, `Fy`, `Fz`, `n` per step | N·m, W, N, rpm | compared with the limits above |
| Machine → closed loop / edge box | list of signals the control publishes, with units and rates; added sensors | –, Hz | what can be measured on this machine |
| MTConnect / OPC UA / edge box (shop) → library | measured spindle load, axis load, actual feed, vibration | %, mm/min, m/s² | calibrates the shop instance |

The feed for each step is set by the smallest of the limits:
`min(tool, holder, spindle(n), machine axes, workholding, stock)`, as in the system overview.

### Using the library in the negotiation

1. **Before cutting (planning):** for each operation, the solver checks spindle torque and power
   at the chosen speed, axis thrust, and deflection through the whole chain; it picks the speed
   where `M_spindle(n)` and stability both allow the most removal (often not the vendor's
   middle `vc`).
2. **Along the path (per step):** with the real feed from acceleration and jerk, the force vector
   in machine coordinates, the stock's current stiffness and the hold's capacity envelope, the
   solver adjusts feed and engagement so that no limit is exceeded in any direction (Jason's
   real-time negotiation idea, detailed in the workholding file).
3. **Cut order:** stiffness of the machine is fixed but the stock's falls as it is cut; the stock
   integrity file plans the order; the machine supplies its part of the chain.
4. **Choosing a machine:** with several machines in a shop, the same checks rank them for a job
   ("VF-2: spindle-power limited at 600 rpm, 14 min; a 50-taper machine: 9 min") (opinion).
5. **Explaining the limit:** every result names the limiting item in plain words, e.g.
   "limited by spindle torque at 606 rpm (122 N·m available, 80 % used)".

### Fit with SolidCraft

- Everything is a command (AGENTS.md): machine entries would be read and assigned through
  commands such as `cam.machine.assign` (illustrative id), so scripts, the CLI and agents use the
  same checks.
- Units: SolidCraft is mm and radians internally; the library should store SI with mm (N, N·m,
  W, rad/s, mm/s²) and convert brochure units on entry.
- Model links: SolidCraft's existing mechanism/joint work and STEP import could show the machine
  model; the kinematic chain could be expressed with AP242 kinematic pairs (opinion).
- No CAM is merged in SolidCraft yet (open RFC #1153 and PR #4158 at the time of writing; the
  tool library file has the details).

### Where time is lost

Typing machine specs into each CAM program; building machine models for simulation; discovering
spindle or axis limits by overloads; trial cuts to find stable speeds; cycle time estimates that
miss by a third or more.

## Export control, sanctions and government use

*Added 2026-10-29 from the [government use research](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-qk5riz/research/gov-research-nsnlookupjn-government-use.md). Opinion and policy proposals, not legal advice.*

**Our three rules:**

- **No one goes to jail.** Our team has no in-house counsel. So wherever a rule is unclear we take the most conservative reading and refuse rather than warn. Questions that need an outside export-control lawyer are marked as such, and free official help (BIS, DDTC, OFAC) is the first call.
- **Help users comply.** SolidCraft stops and explains in plain language, using rule packs for each jurisdiction that the administrator picks. It flags conflicts between countries' laws, never resolves them, and never gives legal advice.
- **Prevention first.** Nothing in this file that is hosted, shared, paid or networked ships until its compliance gates are in place and checked: region blocks, screening, ITAR flag handling, government-mode behaviour, and an outside lawyer's sign-off for any hosted or paid service.

**What in this file crosses a border:** a shared machine library and capability models contributed by users and makers.

- **Hosted parts block sanctioned regions and screen people.** Any service this file proposes that the project would run blocks comprehensively sanctioned regions, plus Russia and Belarus (US, EU and UK rules restrict CAD/CAM software and related services there), and screens accounts, payers, contributors and marketplace parties against the US Consolidated Screening List and the UK, EU and UN lists.
- **No support there.** No support, consulting or custom work for users in sanctioned regions or for listed parties.
- **Government mode.** At the Controlled level (CUI and allied equivalents) online parts are off or point to the organisation's own mirror; at the Isolated level they arrive only as signed packs on media; a design flagged Export controlled (ITAR/EAR or national equivalent) never leaves through them.
- **Only generic data is shared.** Nothing derived from a controlled design (geometry, toolpaths, part or drawing numbers, test or inspection results) goes into a shared library, report or issue. Tool data stays shareable through the field allow-list in the high-security research.
- **Code stays public.** Every build the project ships is public, which keeps it "published" software outside the EAR. The rules are tracked by the standing export and sanctions watch.
- **Checked for controlled data (2026-10-29):** this file contains no controlled technical data. Its examples are public sources, generic values or made-up stand-ins, and the author's experience is stated in general terms.

## Glossary

- **Machine model / shop instance:** the catalogue description of a type of machine / one real
  machine in a shop with its options and measurements.
- **Base speed:** spindle speed where constant torque gives way to constant power.
- **S1 / S6-40 %:** continuous rating / rating for 40 % load in each 10-minute period.
- **Material scope / lock:** the materials a machine or library is approved for; *locked* refuses
  others, *preferred* only warns.
- **Gear or winding range:** a spindle mode with its own torque/power curve and speed range.
- **Thrust:** the maximum force an axis drive can push with.
- **Warm-up routine:** a builder-defined spindle (and axis) run before cutting, to spread
  lubricant and reach thermal size.
- **Jerk:** rate of change of acceleration; limiting it smooths motion but slows corners.
- **Look-ahead:** the control reading ahead in the program to plan speed through corners.
- **Static stiffness (`k`):** force per unit deflection, N/µm.
- **Compliance:** 1/stiffness; compliances of parts in series add.
- **FRF:** frequency response function; displacement per unit force against frequency.
- **Tap test:** measuring an FRF with an instrumented hammer and an accelerometer.
- **RCSA:** receptance coupling substructure analysis; joins component FRFs to predict the
  tool-point FRF.
- **Stability lobe diagram:** chart of stable and chatter-prone speed/depth combinations.
- **Kinematic chain:** the ordered axes from bed to tool and from bed to part.
- **Ballbar:** a circular test of a machine's contouring accuracy.

## Open questions

1. Should a material lock also follow from the coolant and chip setup (e.g. "coolant: aluminium
   only"), or always be set by hand?

2. Will builders contribute catalogue entries, or should the community transcribe public specs?
   How are conflicting sources (builder vs dealer vs measurement) shown?
3. What are safe default stiffness and mode values per machine class (small VMC, 40-taper VMC,
   50-taper, trunnion 5-axis, router) when nothing is measured, and who reviews them?
4. Can 3D machine models be shared at all? Which builders allow redistribution? Should the
   library ship simplified community models (envelope boxes per moving part) by default?
5. Which kinematic description should SolidCraft use: STEP AP242 kinematics, ISO 14649-201, or
   its own, with import from the others?
6. What cold derating is reasonable when the builder gives none, and should SolidCraft's post
   insert the builder's warm-up routine automatically when the machine has been idle?
7. How should the duty rating be chosen automatically: from the predicted load over a 10-minute
   window, or always S1 unless the user opts in?
8. Is an optional stiffness/FRF map over the work envelope worth its measurement effort for
   typical VMCs?
9. Can MTConnect load data refine the shop instance automatically, and how is a bad reading
   (e.g. a dull tool) kept from corrupting the library?

## Sources

All accessed 2026-10-26.

### Standards (catalogue entries only)

- ISO/TS 14649-201:2011, Data model for CNCs, Part 201: Machine tool data for cutting
  processes. [iso.org](https://www.iso.org/standard/60042.html)
- ISO 230-1:2012, Test code for machine tools, Part 1: Geometric accuracy of machines operating
  under no-load or quasi-static conditions. [iso.org](https://www.iso.org/standard/46449.html)
- ISO 10791-4:1998, Test conditions for machining centres, Part 4: Accuracy and repeatability of
  positioning of linear and rotary axes. [AFNOR](https://www.boutique.afnor.org/en-gb/standard/iso-1079141998/test-conditions-for-machining-centres-part-4-accuracy-and-repeatability-of-/xs010933/106489)
- ISO 10303-105 Kinematics (integrated resource). [STEP Tools index](https://downloads.steptools.com/stds/smrl/data/resource_docs/kinematics/sys/resdocindex.htm)
- CAx-IF / JT-IF, Recommended Practices for AP242 XML Kinematics v1.3, 2025-10-20.
  [mbx-if.org](https://www.mbx-if.org/home/wp-content/uploads/2025/10/rec_prac_ap242xml_kinematics_v1.3.pdf)
- MTConnect model browser (Specifications, components).
  [model.mtconnect.org](https://model.mtconnect.org/Version2.3/DeviceInformationModel/Configurations/Specifications/Specification/)

### Papers and books

- Røsjordet, J.F. and Hovland, G. (2019). Methods for Experimentally Determining Stiffness of a
  Multi-Axis Machining Centre. *Modeling, Identification and Control* 40(1), 11–25.
  [PDF](https://www.mic-journal.no/PDF/2019/MIC-2019-1-2.pdf)
- Schmitz, T., Betters, E., Budak, E., Yüksel, E., Park, S., Altintas, Y. (2023). Review and
  status of tool tip frequency response function prediction using receptance coupling.
  *Precision Engineering* 79, 60–77. [PDF](https://mtrc.utk.edu/wp-content/uploads/sites/45/2022/10/RCSA-review.pdf)
- Ward, R., Sencer, B., Jones, B., Ozturk, E. (2021). Accurate prediction of machining feedrate
  and cycle times considering interpolator dynamics. [arXiv 2102.02062](https://arxiv.org/pdf/2102.02062)
- Swan, R., Penney, J., Corson, G., Nazario, J., Schmitz, T. (2024). Robot milling surface
  location error. *CIRP JMST* 49, 203–215. [PDF](https://par.nsf.gov/servlets/purl/10543730)
- Position-dependent response simulation of machine tool using state space models. *MM Science
  Journal*, December 2017. [mmscience.eu](https://www.mmscience.eu/journal/issues/december-2017/articles/position-dependent-response-simulation-of-machine-tool-using-state-space-models)
- Olvera, D. et al. (2012). Analysis of the tool tip radial stiffness of turn-milling centers.
  *IJAMT* 60, 883–891. [ORNL record](https://impact.ornl.gov/en/publications/analysis-of-the-tool-tip-radial-stiffness-of-turn-milling-centers/)
- Mass loading effect of accelerometers on tool point FRF (2010). *IJMTM* 50(6), 585–589.
  [Sabancı](https://research.sabanciuniv.edu/12777)
- Hedlind, M. (2010). Manufacturing resource modelling (STEP-NC/ISO 10303-105 slides), KTH.
  [PDF](https://steptools.com/stds/stepnc/2010_nist/presentations/Manufacturing_resource_modelling_2010-06-17.pdf)

### Vendor and program pages

- Autodesk, Machines for manufacturing (Fusion help).
  [help.autodesk.com](https://help.autodesk.com/cloudhelp/ENU/Fusion-CAM/files/MFG-MACHINES.htm)
- Autodesk, Learn how Machine Builder works in Autodesk Fusion (blog).
  [autodesk.com](https://www.autodesk.com/products/fusion-360/blog/learn-how-machine-builder-works-in-autodesk-fusion/)
- Kollmorgen, Calculating linear force and torque, ballscrew.
  [kollmorgen.com](https://www.kollmorgen.com/en-us/developer-network/calculating-linear-force-and-torque-ballscrew)
- CADEM, S1 S6 motor ratings. [cadem.com](https://cadem.com/s1-s6-motor-ratings-spindle-motor/)
- CloudNC, How to edit machine properties (not fetched). [atlassian](https://cloudnc.atlassian.net/wiki/x/MQDqAw)
- Haas VF-2 (2018) dealer listing (specs used in examples).
  [machinehub](https://blumberg.machinehub.com/listing/haas-vf-2-cnc-3-axis-vertical-machining-center/)
- Haas HRT210 dealer listing (torque and brake figures).
  [surplusrecord](https://surplusrecord.com/listing/8-27-haas-hrt-210-brushless-4th-axis-cnc-rotary-table-160-ft-lb-torque-200-ft-lb-brake-6-t-slots-pneumatic-brake-in-crate-2022-never-used-1133844/)

### Trade press

- Heeg, C. (2020-12-14). The benefits of a proper spindle warm-up routine. Okuma blog.
  [okuma.com](https://www.okuma.com/blog/the-benefits-of-a-proper-spindle-warm-up-routine)

- Smith, S. (2012). Spindle power and torque limitations. *Cutting Tool Engineering*.
  [ctemag.com](https://ctemag.com/?p=96749)
- Schmitz, T. (2026). How to stabilize machining with tap testing. *Modern Machine Shop*.
  [mmsonline.com](https://mmsonline.com/articles/how-to-stabilize-machining-with-tap-testing)
- VERICUT 6.0 includes showroom of popular machines and controls (2006). *Digital Engineering*.
  [digitalengineering247.com](https://www.digitalengineering247.com/article/vericut-6-0-includes-showroom-of-popular-machines-and-controls)

- IronFlock, MTConnect collector docs. [ironflock.com](https://ironflock.com/en/docs/factory-data-extraction/mtconnect-collector/)
- OPC Foundation, OPC 40501-1 Machine Tools: Monitoring and Job Management (v1.02).
  [reference.opcfoundation.org](https://reference.opcfoundation.org/MachineTool/v102/docs/1?n=1)
- Cybus, FOCAS connector docs. [docs.cybus.io](https://docs.cybus.io/connectors/shop-floor-connectors/focas.md)

### Community sources

- eMastercam forum, Machine tool models (2011–2012).
  [emastercam.com](https://www.emastercam.com/forums/topic/60611-machine-tool-models/)
- LinuxCNC forum, a new finite-jerk S-curve trajectory planner for testing.
  [forum.linuxcnc.org](https://forum.linuxcnc.org/29-forum-announcements/58181-a-new-finite-jerk-s-curve-trajectory-planner-for-testing)
- 3dcadbrowser, Haas VF-2 CNC rotary mill CAD model (user-made, licence unclear).
  [3dcadbrowser.com](https://www.3dcadbrowser.com/3d-model/haas-vf-2-cnc-rotary-mill)
