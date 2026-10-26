# CAM research: a Raspberry Pi that drives the cell, robot part handling and tool setting

Research for [#1621](https://github.com/tangency-labs/solidcraft/issues/1621) (Research Request
for Plugin: CAM). This file covers one part of that request: **using the small edge computer at
the machine (a Raspberry Pi) for more than collecting measurements: coordinating robot part
loading and unloading, and carrying tool-setting data from the presetter and the on-machine tool
setter into the machine**, and how SolidCraft CAM could plan such a cell before anything moves.

The single most important finding is a boundary, not a feature: **the Pi can supervise, sequence
and record a cell, but it must never be part of the safety chain and should not be part of the
real-time handshake between the robot and the machine.** Those belong to certified safety
hardware and to the robot and machine controls. Everything the Pi does sits above them.

It builds on earlier files from the same author and does not repeat them:

- [In-process inspection](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-y2w06n/research/cam-research-nsnlookupjn-in-process-inspection.md):
  **owns the Pi box** (hardware, interfaces to the control, the "how could it be offered"
  question) and its rule "**the Pi records, the control corrects**". This file extends the same
  box and keeps the same rule.
- [Shared cloud tool library](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-tool-library/research/cam-research-nsnlookupjn-tool-library.md):
  tool assemblies, the shop layer with measured offsets, and **tool tags** (RFID to DIN 69873,
  DataMatrix, printed label holders). Tool setting here is the path from that record into the
  machine.
- [Machine library](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-machine-library/research/cam-research-nsnlookupjn-machine-library.md):
  machine entries, kinematics and 3D model, the signals each control publishes, warm-up. A robot
  is added here as another kind of entry.
- [Workholding](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-workholding/research/cam-research-nsnlookupjn-workholding.md):
  vises, clamps, clamping force. A robot gripper is workholding that moves.
- [Stock integrity](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-stock-integrity/research/cam-research-nsnlookupjn-stock-integrity.md):
  thin walls after early operations; the robot gripper squeezes the part too.
- [CAM physics system](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-vgqo3g/research/cam-research-nsnlookupjn-physics-system.md):
  the shared setup, the one limits engine and the "physics in the simulation before G-code" gate,
  which this file extends to the robot.

Licence: MIT OR Apache-2.0, the same as SolidCraft (see the Licence section of CONTRIBUTING.md).

## About this research

- **Author:** Jason ([@nsnlookupjn](https://github.com/nsnlookupjn)). Idea and direction by Jason
  ("a Pi system to drive other functions such as robot part handling and tool setting"); source
  research and writing done with an AI assistant (Claude) in October 2026.
- **How it was produced:** web searches of public sources (standards bodies' and trade
  associations' public pages, the MTConnect public specification, the OPC Foundation's public
  reference, robot and machine builders' public product pages, probe makers' product pages),
  each claim checked against the cited page where possible. No commercial CAD/CAM program was
  driven or scripted, no GPL/LGPL/AGPL source was read, and no text is copied beyond short
  quotes. Safety standards are described from public summaries, not their paid text: **anyone
  building a cell must work from the standards themselves and a proper risk assessment**, not
  from this file.
- **SolidCraft facts** were checked in the repository at commit `10b5cc5` (October 2026).
- **Units:** millimetres, newtons, seconds, radians, unless a source is quoted.
- **Marking:** "(unverified)" marks a claim not confirmed against a source. **Fact**,
  **opinion** and **marketing** are labelled where they mix. Several manufacturer pages
  (notably Haas's online manuals) could not be fetched for this research; claims that depend on
  them are marked.

> **[TODO Jason: personal experience]** Which machines you would automate first, whether you
> have run a robot or a parts loader (Haas APL, a cobot cart, a bar feeder), how tools get from
> the presetter (if any) into your Haas controls today, how often an offset gets typed wrong, and
> what you would trust the Pi to do unattended. Label this as personal experience.

## Summary

1. **Three layers, and the Pi lives only in the top one.** A machine-tending cell has a
   **safety layer** (fences, interlocked doors, area scanners, E-stops, the robot's own safety
   controller; certified to ISO 13849-1 / IEC 62061 and ISO 10218-2), a **real-time handshake
   layer** (robot and machine telling each other "door open", "part clamped", "cycle start",
   in milliseconds, over discrete I/O, M-codes, a fieldbus or a vendor robot interface), and a
   **supervision layer** (which job, which program, how many parts, which tools, what to record).
   The Pi belongs in the third. Opinion, strongly held: a Linux single-board computer is not a
   safety-rated device and should not sit in the second layer either, because a crash, an SD
   card failure or an update would stop or confuse the cell mid-move.
2. **The handshake already has an open standard.** MTConnect Part 5 "Interfaces" defines
   request/response pairs between two pieces of equipment, including **MaterialLoad,
   MaterialUnload, OpenDoor, CloseDoor, OpenChuck, CloseChuck** and **PartChange**, with
   states READY → ACTIVE → COMPLETE (or FAIL) on each side
   ([MTConnect Part 5 v2.2](https://docs.mtconnect.org/MBSD_MTConnect_Part_5_2-2-0.pdf)).
   It is the natural *record* of what the cell did, and where both machines support it, a
   vendor-neutral handshake. It says nothing about safety.
3. **Machine makers already sell the robot integration.** Haas Robot Package 3 is a 6-axis,
   50 kg robot set up from the Haas control's robot interface, with CE fencing and fence
   interlock included ([Race Engine Technology summary of Haas](https://www.raceenginetechnology.com/Suppliers/haas-robot-package-3-whats-new));
   FANUC QSSR runs a robot from M-codes in the CNC program over one Ethernet cable, with robot
   jog and a production monitor on the CNC screen
   ([FANUC](https://fanuc.co.jp/en/product/robot/application/machining/qssrforcnc.html)).
   The Pi should **work alongside these, not replace them**: they own the motion and the
   handshake; the Pi gives the job, records what happened and feeds SolidCraft.
4. **Tool setting is a data problem first.** The presetter (or the on-machine tool setter)
   measures the tool; the measured length and diameter must reach the right offset register,
   for the right tool, in the right machine, without retyping. Paths exist: RFID or DataMatrix
   on the holder, presetter post-processors, DNC, and G-code `G10` blocks that load offsets
   ([Haas UK, G10](https://www.haas.co.uk/?p=1287)). MTConnect's **CuttingTool** asset already
   carries measured assembly values with ISO 13399 codes (OAL, DC, LF…), tool life, pot number
   and status ([MTConnect Part 4.1](https://docs.mtconnect.org/MTC_Part4_1_CuttingTools_1_4_0.pdf)).
5. **The Pi's job in tool setting: identity, staging and verification.** Scan the tool tag, look
   the assembly up in the tool library, check it is the tool the CAM program expects (number,
   stick-out, length window), **stage** the offsets for the control, and record the on-machine
   check against the presetter value. Writing offsets directly is a separate, explicit, logged
   step, off by default (the same rule as the inspection file's compensation write-back).
6. **SolidCraft's part: plan the cell before it runs.** CAM already decides setups and
   workholding; with a robot it can also decide **where the gripper takes the part in each state,
   whether the grip holds it, whether the robot reaches and clears the machine, how much tool
   life the batch needs, and what the handshake looks like in the program**. SolidCraft already
   exports assemblies as URDF (`docs/urdf.md`), the format ROS 2 and MoveIt read, so a gripper
   designed in SolidCraft can go straight into a robot simulation.
7. **The request:** treat a robot as another machine-library entry and a gripper as another
   workholding entry; add a "cell" to the shared setup; extend the simulation gate to the robot;
   and let the post processor emit a declared handshake (M-codes or MTConnect Interfaces) from a
   per-machine mapping, never hard-coded numbers.

## Fundamentals

### What a machine-tending cell is made of

| Element | What it does | Who controls it |
|---|---|---|
| Machine tool | cuts; owns spindle, axes, door (if automatic), vise or chuck (if powered), coolant, probe and tool setter | CNC |
| Robot or loader | moves parts between stations; may also flip, blow off, mark or present parts to a gauge | robot controller (with its safety controller) |
| Gripper | holds the part; may have sensors (closed, part present, width) | robot controller I/O |
| Stations | raw-part table or drawers, finished-part rack, regrip stand, blow-off, gauge or CMM, reject chute | robot (positions), sensors on I/O |
| Safeguards | fence, interlocked gate, area scanner or light curtain, E-stops, mode selector | safety controller / safety PLC |
| Cell coordination | job order, program select, counts, tool life, alarms, records | today: robot controller, a PLC or the CNC; proposed: the Pi supervises |

Fact: the hardwired version needs at minimum load command / load complete and unload command /
unload complete signals each way, plus E-stop and door interlock wiring and often a
part-present sensor; the cycle is "CNC reaches a load point → M-code → robot acts → completion
signal → CNC continues" (summary from a trade knowledge-base page,
[Industrial Monitor Direct](https://industrialmonitordirect.com/blogs/knowledgebase/fanuc-robot-haas-cnc-integration-methods);
that page's example uses `G04 P0` as a "wait", which it rightly flags as doubtful; check the
machine's own manual, (unverified)).

### Three layers of control

1. **Safety layer.** Stops the hazard when a person comes near: interlocked gate, scanner field,
   E-stop, safe speed and position limits on the robot. It must be designed to a required
   performance level (ISO 13849-1, PL a–e) or SIL (IEC 62061) from a risk assessment. Robot
   makers provide safety functions inside the robot controller (e.g. FANUC Dual Check Safety,
   which Haas's robot-package manual has a setup chapter for; page not fetched, (unverified)).
   **No general-purpose computer in this layer.**
2. **Handshake layer.** Coordinates two controllers that each move things: the robot must not
   enter while the spindle runs or the door is closed; the machine must not start before the
   robot has left and the vise has closed. Deterministic, local, survives network loss. Today:
   discrete I/O, M-codes, fieldbus (EtherNet/IP, PROFINET), or a vendor interface (Haas Robot
   Package, FANUC QSSR, VersaBuilt's Robot2CNC kit for UR cobots, which selects CNC programs from
   the teach pendant, cycle-starts the CNC and checks the program finished
   ([Universal Robots+](https://www.universal-robots.com/plus/products/versabuilt-robotics/robot2cnc-haas-legacy/))).
3. **Supervision layer.** Slow, rich and networked: which job is next, which program and robot
   recipe go with it, how many raw parts are left, whether the tools have life for the batch,
   what each part's record says, when to call a person. Seconds, not milliseconds. **This is the
   Pi's layer.**

### Why the Pi stays out of the safety and handshake layers

- **Certification (fact).** Safety functions must be built from components whose reliability is
  known (PL/SIL, diagnostic coverage, common-cause measures). A Raspberry Pi has none of this;
  even industrial Pi products such as the Revolution Pi are sold with EN 61131-2 and EMC
  conformity, not as safety controllers (inspection file, hardware sketch). A maker of a
  Pi-based PLC writes that its software interlocks "complement, never replace, hardware
  protection" and keeps a hardwired E-stop chain
  ([Industrial Shields](https://www.industrialshields.com/plc-solution-library/raspberry-plc-safety-interlocks-emergency-stop)).
- **Timing (worked example, opinion on the numbers).** Guard placement uses the approach-speed
  formula of ISO 13855, `S = K · T + C`, where `K` is the body approach speed (about
  1,600–2,000 mm/s for walking, per public summaries; (unverified) exact values), `T` the whole
  stopping time and `C` an intrusion allowance. Every 50 ms of extra response time adds
  `1600 × 0.05 = 80 mm` to the minimum distance. Linux without real-time tuning can stall for
  tens of milliseconds (opinion); a safety function must have a guaranteed `T`.
- **Robustness (opinion).** If the Pi is only supervising and it fails, the cell finishes the
  current part and stops at the next job boundary. If the Pi were in the handshake and failed,
  the cell could stop with the robot inside the machine and the door open.
- **Cybersecurity (fact).** ISO 10218-2:2025 adds cybersecurity requirements related to robot
  safety ([A3 FAQ](https://www.automate.org/robotics/blogs/updated-iso-10218-faq)). A
  network-connected computer that could command robot motion widens what must be assessed.
  Keeping the Pi to supervision, behind the robot's and machine's own "remote allowed" modes,
  keeps that surface small.

### Robot safety standards in 2025–2026 (from public summaries)

- **ISO 10218-1:2025** (robot makers) and **ISO 10218-2:2025** (integrators and applications)
  were published in March 2025, replacing the 2011 editions. Part 2 absorbs the collaborative
  application requirements that were in **ISO/TS 15066:2016**; functional safety requirements
  are now explicit; Part 2 shifts emphasis from "robot system" to "**robot application**",
  including workpieces, task programs and supporting machinery; and the words "collaborative
  robot" are replaced by "**collaborative application**", because only a use can be shown to
  be collaborative ([A3 FAQ](https://www.automate.org/robotics/blogs/updated-iso-10218-faq)).
  Consequence for marketing (opinion): "a cobot needs no fence" is not a property of the robot;
  a machine-tending application with sharp parts, a vise and a door may still need guarding.
- **ISO 13849-1** (safety-related parts of control systems, PL) and **IEC 62061** (SIL) are how
  each safety function is rated; **ISO 13855** (positioning of protective equipment),
  **ISO 13857** (reach distances), **ISO 14119** (interlocks) and **ISO 14120** (guards) are the
  usual companions (catalogue knowledge; editions to check).
- **ISO 16090-1:2022** covers safety of milling machines and machining centres, including
  machines fitted with tool magazines, workpiece handling mechanisms, powered clamping and
  power-operated doors, and workpiece loading/unloading devices "when they form an integral
  part of the machine" ([EVS catalogue](https://www.evs.ee/en/evs-en-iso-16090-1-2022)).
  Whether an add-on robot is covered there or only by ISO 10218-2 was not settled from the
  summary (unverified); in practice both apply to the combined cell (opinion).
- **US:** ANSI/A3 R15.06 adopts ISO 10218; a 2025 edition aligned with the 2025 ISO parts is
  reported (unverified).

### The handshake: what exists today

| Method | How it works | Notes |
|---|---|---|
| **Discrete I/O + M-codes** | CNC sets an output (M-code), robot replies on an input; the program waits on it | Universal; M-code numbers and wiring vary by machine and integrator (Industrial Monitor Direct says "assignments vary by integrator"). Haas user M-codes with M-FIN wait and output relays exist (Haas manual pages not fetched; numbers (unverified)). |
| **Fieldbus** | EtherNet/IP, PROFINET, DeviceNet I/O between robot and CNC/PLC | On Haas, reported as options to order with the machine (Industrial Monitor Direct; (unverified)). |
| **Vendor robot interface** | Robot set up and jogged from the CNC screen; robot routines called from the part program | Haas Robot Package (APL mode with an `M299` at the end of the program loads the first part and runs the cycle, per the Haas summary); FANUC QSSR (M-code calls, one Ethernet cable, guided load/unload programs on the CNC). |
| **Robot-side CNC driver** | Robot program selects and starts CNC programs over Ethernet | VersaBuilt Robot2CNC for UR + Haas (above). |
| **MTConnect Part 5 Interfaces** | Each side publishes request and response states the other reads | Vendor-neutral; needs both sides to be MTConnect agents and clients. Covers MaterialHandler, Door, Chuck and BarFeeder interfaces. |
| **SRCI (Standard Robot Command Interface)** | A PLC calls 115+ standardised robot functions on any compliant robot controller; client-server with remote procedure calls, robot controller as PROFINET IO-device | From PROFIBUS & PROFINET International ([profinet.com](https://www.profinet.com/profinet-explained/srci-standard-robot-command-interface)); a UR listing exists ([UR marketplace](https://www.universal-robots.com/marketplace/products/01tP40000071NjXIAU)). Aimed at PLCs, not at a Linux supervisor. |
| **OPC UA** | OPC 40010 Robotics (VDMA) describes robots for monitoring; OPC 40501-1 Machine Tools covers monitoring and job management, tool data and tool life, production plan facets ([OPC Foundation reference v1.02](https://reference.opcfoundation.org/MachineTool/v102/docs/1?n=1); [OPC Foundation on VDMA robotics](https://opcfoundation.org/news/press-releases/vdma-releases-opc-ua-companion-specifications-robotics-machine-vision/)) | Good supervision interfaces; neither is a motion handshake. |
| **ROS 2** | Robot drivers (FANUC's official ROS 2 driver: ros2_control, MoveIt, 1 ms control on CRX and other models, [FANUC America](https://fanucamerica.com/solutions/ros-2); UR has one too) | Excellent for planning and simulation; not a safety system; machine-tool side has no standard ROS 2 interface (opinion). |

**MTConnect Interfaces in a bit more detail (fact, Part 5 v2.2).** Coordination happens by each
side publishing its own state, because "the base MTConnect method provides no direct way for one
piece of equipment to change the state of" another. The requester sets its request to ACTIVE;
the responder sets its response to ACTIVE, does the job, sets COMPLETE; the requester returns to
READY, then the responder does. Either may go to FAIL; both then reset partial actions and return
to READY, or NOT_READY if they cannot recover. An `InterfaceState` of DISABLED forces
NOT_READY. Part 5 has no safety content.

Opinion: this is the right **vocabulary** for SolidCraft to plan and record a cell in, even when
the real handshake is wires and M-codes: "MaterialLoad requested at block N210, ACTIVE 3.2 s,
COMPLETE" reads the same whatever the wiring.

### Tool setting: where the numbers come from

| Source | Measures | Typical accuracy | Where the number goes |
|---|---|---|---|
| **Off-machine presetter** (Zoller, Speroni, Haimer, Kelch…) | length, diameter, radius, run-out, edge condition, under magnification, before the tool reaches the machine | microns (vendor claims; unverified per model) | label/RFID, post-processor or DNC file, network to the control (Zoller: RFID, post-processor through DNC, higher-level systems, or a DataMatrix label read at the control, [SIAMS news](https://www.siams.ch/news-en/Data-Transfer-Securely-to-the-Machine-at-Top-Speed/6475)) |
| **Contact tool setter in the machine** (Renishaw OTS, RTS, TS27R, TS34, LTS; APC arm on lathes; Haas WIPS tool setter) | length and diameter (3D touch-trigger setters); length only (LTS); turning, grooving, threading and boring tools (APC); broken tool on all | 1.00 µm 2σ (OTS, RTS, TS27R, TS34); 0.75 µm (LTS); 1.50 µm (APC) ([Renishaw contact tool setters](https://www.renishaw.com/ots)) | written to the offset by the control's macro (Renishaw macros, below; Haas WIPS sets tool length offsets and detects broken tools, [Haas UK](https://www.haas.co.uk/wips/); WIPS hardware is Renishaw's, [Renishaw](https://www.renishaw.com/en/39930.aspx)) |
| **Non-contact laser setter** (e.g. Renishaw NC4) | length, diameter, radius of ball and radius tools, broken or chipped edges, swarf on the tool, spindle thermal growth; run-out with an advanced package | NC4: 1.0 µm 2σ repeatability; NC4+ Blue F100 "down to ±0.1 µm 2σ" ([Renishaw](https://www.renishaw.com/en/nc4-non-contact-tool-setter--6099)) | the setter's interface sends a relay signal to the CNC; Renishaw's macros compute and write offsets (same page) |
| **Manual** (touch-off, height gauge) | length | tens of microns (opinion) | typed in: the error the rest of this section removes |

**Renishaw as the reference (Jason's pointer).** Jason names Renishaw as the gold standard for
on-machine probing and tool setting; the [in-process inspection file's "Reference products for
on-machine probing"](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-y2w06n/research/cam-research-nsnlookupjn-in-process-inspection.md#reference-products-for-on-machine-probing)
section covers Renishaw's whole stack (Inspection Plus macros, GoProbe, the Set and Inspect GUI,
Reporter, 5-axis checks) and is not repeated here. What matters for tool setting:

- **Transmission and interface (fact).** OTS is infrared optical, RTS is radio (2.4 GHz
  frequency hopping), TS27R, TS34, LTS and APC are hardwired; every contact setter needs a
  separately sold interface to the control ([Renishaw](https://www.renishaw.com/ots)). The NC4
  laser's NCi-6 interface gives the control a solid-state relay signal
  ([Renishaw NC4](https://www.renishaw.com/en/nc4-non-contact-tool-setter--6099)). Either way
  the measurement happens **inside the control**: the probe only triggers, the control latches
  the axis position, the macro computes. That is why the Pi can record results but cannot
  measure tools itself (same reason as in the inspection file).
- **What the software covers (fact).** Renishaw's tool-setting software sets length and/or
  diameter for single and multi-point tools on machining centres and static and powered tools on
  turning centres, by manual or programmed positioning; contact setters reseat to within 1 µm
  after triggering and support broken-tool detection; non-contact adds radial and linear
  profile checks and cutting-edge condition monitoring
  ([Renishaw tool-setting software](https://www.renishaw.com/en/tool-setting-software--6251)).
  The Set and Inspect GUI also covers contact and non-contact tool setting (inspection file).
- **What a macro call looks like (fact, from a 2002 Renishaw manual for Haas VF controls,
  H-2000-6223-00-A, hosted by Haas; newer packages may number things differently,
  (unverified)).** `O9851` manual length setting and length calibration against a reference
  arbor; `O9852` diameter setting and stylus XY calibration; `O9853` automatic setting:
  `G65 P9853 B1 T5` sets the length of tool 5 (`B2` diameter, `B3` both); with `H0.05` it is a
  broken-tool check at ±0.05 mm, which either raises alarm 1099 BROKEN TOOL and stops, or, with
  `M` naming a flag register, writes 1 (broken) or 0 (good) there and lets the program decide;
  tool offsets are not changed by the broken-tool check
  ([Renishaw/Haas manual PDF](https://www.haascnc.com/content/dam/haascnc/en/service/manual/probe/english---renishaw-contact-tool-setting---programming-manual---2002.pdf)).

**Fit with the Pi and with CAM (opinion).** CAM should emit **calls to the shop's existing
cycle library** (Renishaw, WIPS, the control maker's own), never its own probing arithmetic, the
same position the inspection file takes for part probing. Concretely: a `B1` length setting the
first time each tool is used, and an `H…` broken-tool check after the risky tools (small drills,
taps, long-reach end mills), using the **flag form** so the program can send the part to the
reject station instead of stopping the whole lights-out cell, and print the flag (DPRNT) for the
Pi to record. The tolerance `H` comes from CAM (for example, a fraction of the finishing stock
for that tool), so it is explained like every other number.

**How offsets are written in G-code (fact for Haas mills).** `G10 L10 G90 P5 R250.` sets tool 5's
length geometry, `L11` (or `L1`) its length wear, `L12` diameter geometry, `L13` diameter wear
([Haas UK, programmable offsets](https://www.haas.co.uk/?p=1287)). A presetter or a Pi can
therefore deliver offsets as an **ordinary program** the operator loads and runs, which every
control understands and which leaves a trace. Fanuc-family controls use the same `G10 L10–L13`
form for their tool-offset memory types (unverified per memory type), and FOCAS can write offsets
directly (`cnc_wrtofs`, inspection file).

**Machine-side vs. presetter value (fact and opinion).** The two measure different things: the
presetter measures the assembly cold, against its own reference; the machine measures it in the
spindle, warm, with spindle growth and the setter's own trigger point. Good practice (opinion):
take the presetter value as the setting and use the machine setter as a **check** (broken tool,
wrong tool, wrong stick-out, gross error), with a declared tolerance window and a recorded
difference. Worked example (computed for this file): presetter `L = 125.430 mm`, machine setter
`L = 125.452 mm`, difference `0.022 mm`; with a window of `0.030 mm` the tool is accepted and both
numbers go in the record; at `0.300 mm` the cycle stops, which is what catches a tool seated on a
chip or set at the wrong stick-out.

**What the shared formats already carry (fact).** MTConnect separates **CuttingToolArchetype**
(catalogue: nominal values, no measurements) from **CuttingTool** (the real assembly on the
floor) with `CutterStatus` (NEW, AVAILABLE, MEASURED, USED, EXPIRED, BROKEN, …),
`ReconditionCount`, `ToolLife` (MINUTES, PART_COUNT or WEAR, with warning and limit),
`ProgramToolNumber`, `ProgramToolGroup` (sister tools), `Location` (POT, STATION or CRIB),
`ProcessSpindleSpeed`, `ProcessFeedRate`, and assembly **Measurements** named with ISO 13399
codes: OverallToolLength (OAL), CuttingDiameterMax (DC), FunctionalLength (LF), ProtrudingLength
(LPR), BodyDiameterMax (BDX), UsableLengthMax (LUX), Weight (WT, grams) and others
([MTConnect Part 4.1 v1.4](https://docs.mtconnect.org/MTC_Part4_1_CuttingTools_1_4_0.pdf)). The
tool-library file's split (shared catalogue vs. shop instance) is the same split; opinion: use
these names and codes so a SolidCraft tool record exports to an MTConnect CuttingTool without a
mapping table.

### Gripping the part (worked example)

A friction grip must hold `m (g + a)` with margin: `F > m (g + a) / μ × S`, with `F` the sum of
the finger forces, `μ` the static friction between finger and part and `S` a safety factor raised
for strong acceleration or possible impact ([Robotiq](https://blog.robotiq.com/bid/69524/robotic-end-effectors-payload-vs-grip-force)).
For a 2 kg steel part, `a = 5 m/s²`, an assumed `μ = 0.2` and `S = 2` (both assumptions, not
sourced values): `F > 2 × (9.81 + 5) / 0.2 × 2 = 296 N` total, about 148 N per finger.
Two things CAM knows that the integrator usually guesses (opinion):

- **The grip faces.** After op 1 the part may no longer have the parallel faces the raw stock
  had; the gripper must take it by faces that exist in that state, and those faces should be
  machined, not a sawn surface that varies.
- **Whether the part survives the grip.** A thin-walled part after roughing can deform under
  300 N across a 2 mm wall; that is the [stock integrity](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-stock-integrity/research/cam-research-nsnlookupjn-stock-integrity.md)
  file's question with a different load. Shaped (form-fitting) fingers, designed from the part
  model, trade friction for geometry and lower the needed force.

### Lights-out arithmetic (worked examples)

**Spindle utilisation.** A 12 min cutting cycle with a 45 s robot exchange (door, unclamp,
swap, clamp, door) gives `12 / 12.75 = 94 %`; with a 3 min operator exchange, `12 / 15 = 80 %`.
Over 8 hours: `480 / 12.75 ≈ 37` parts vs `480 / 15 = 32`. (Illustrative numbers, computed for
this file.)

**Tool budget for an unattended batch.** If the finishing end mill cuts 3.2 min per part and
its predicted life (from the [cutting physics](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-cutting-physics/research/cam-research-nsnlookupjn-cutting-physics.md)
file's tool-life model, corrected by the shop's history) is 45 min, one tool does
`floor(45 / 3.2) = 14` parts. 37 parts need `ceil(37 / 14) = 3` tools: the main tool plus two
sisters in the same `ProgramToolGroup`. If the magazine has no room for two sisters, the batch
must stop at part 28 or the feed must change. **This is a CAM decision made before the shift,
not an alarm at 2 a.m.**

## Accuracy

- **Robot repeatability is not part accuracy.** The robot only has to put the part into the
  workholding well enough for the vise or chuck to locate it; the vise locates it. A part that
  is not seated (chip under it, robot placed it 0.5 mm high) is the main accuracy risk.
  Air-sense or part-seated checks, then a probe cycle, catch it (opinion; see in-process
  inspection file for the probe side).
- **Tool offsets are accurate only if they belong to the tool in the spindle.** The classic
  failure is a correct number on the wrong tool number (opinion). Identity (tag scan, pocket
  record, `ProgramToolNumber`) matters more than another micron of presetter accuracy.
- **Presetter vs. machine.** Expect a systematic difference between the two (temperature,
  reference, trigger point); measure it per machine and keep it in the machine's shop record,
  rather than discovering it on the first part (opinion).
- **Laser setters** see coolant drops and swarf; NC4's own literature lists "burrs or attached
  swarf" among what it detects, which also means the tool should be clean before measuring
  (Renishaw page above).
- **Every number in the record says how it was measured** (presetter, contact setter, laser,
  manual), the same rule as the inspection file's "label by method".

## Programs

Described from public product pages; no program was run or scripted.

- **Machine-maker robot packages:** Haas Robot Package (set up and jogged from the Haas
  control; APL mode; fence and interlock included); FANUC QSSR on FANUC CNCs (robot from
  M-codes, guided setup, jog and monitor on the CNC screen). Both keep the handshake inside the
  maker's world. Marketing claims of setup time were not checked.
- **Cobot add-ons:** VersaBuilt Robot2CNC for UR + Haas (select program, cycle start, check
  completion from the teach pendant). Others exist (unverified, not researched).
- **Presetter software:** Zoller (»pilot« software, tool management, transfer by RFID,
  post-processor/DNC, DataMatrix), Speroni, Haimer, Kelch; each has post-processors per control
  (unverified per vendor). Data stays in each vendor's database unless exported.
- **Probe-maker macros:** Renishaw tool-setting macros for OTS/NC4; Haas WIPS templates (VPS).
- **Open source:** ROS 2, MoveIt 2 and ros2_control for robot planning and simulation; the
  official FANUC ROS 2 driver; the MTConnect C++ agent (open source, Apache-2.0 per the project
  (unverified)); OPC UA stacks in many languages. **Nothing open plans the whole cell from the
  part model** (opinion, from the search).

## Complaints and requests

Not researched as deeply as the other files; the points below come from the sources above and
are labelled.

- "Integration is custom every time": M-code numbers, I/O maps and handshake timing differ per
  integrator (Industrial Monitor Direct). Opinion: that is what a declared per-machine mapping
  in the machine library fixes.
- Fieldbus options on some machines must be ordered with the machine and cannot easily be added
  later (Industrial Monitor Direct, about Haas; (unverified)).
- Retyping presetter values into the control (opinion, widely reported in trade press; not
  sourced here); presetter vendors sell RFID, DataMatrix and post-processors precisely to avoid
  it.
- Cobot marketing vs. safety reality: the 2025 standard's own move from "collaborative robot"
  to "collaborative application" (A3 FAQ) answers the common surprise that a cobot cell still
  needs guarding.

> **[TODO Jason: personal experience]** Your own complaints: what has gone wrong with automation
> you have seen, and what a small shop would need before trusting a robot overnight.

## What users need

### The Pi's role in the cell (opinion)

The same box as the [in-process inspection](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-y2w06n/research/cam-research-nsnlookupjn-in-process-inspection.md)
file, with more jobs, all in the supervision layer:

| Function | What the Pi does | What it never does |
|---|---|---|
| Safety | Reads safety status if a non-safe status output is provided (gate open, scanner muted, E-stop) and records it | Is wired into any safety circuit; bypasses, mutes or resets a safety function |
| Robot ↔ machine handshake | Records each MTConnect Interface transition, or the I/O and M-code events, with time and block number | Drives the handshake signals |
| Job sequencing | Holds the queue (job, program, robot recipe, quantity, traveller) and, through the robot's or control's documented remote interface, selects the next job **while the cell is in automatic and idle** | Starts motion outside the robot's and machine's own remote-allowed mode |
| Raw-part and finished-part counts | Counts from the handshake; warns before the table runs out | Moves conveyors or drawers inside the guarded space |
| Tool identity | Reads tag scans at the machine and in the presetter room; matches them to the tool library and the program's tool list | Trusts a tag that does not match the program |
| Tool offsets | **Stages** them (G10 program, offset file, or presetter output) for the control; records the on-machine check | Writes them silently (see tiers below) |
| Tool life | Reads tool life from the control (OPC 40501 ToolLife, MTConnect `ToolLife`, FOCAS), compares with the batch's tool budget | Changes tool life counters |
| Part record | Appends handling events (loaded at, unloaded at, gauge station, reject chute) to the part's record and traveller | — |
| Alerts | Calls a person (stack light via a non-safety output, message, label) | — |

**Write tiers (opinion), the same ladder for every function the Pi could act on:**

1. **Record** (default): read and log only.
2. **Advise**: show a recommendation ("tool 12 has life for 9 more parts; batch needs 14").
3. **Stage**: prepare the action for a person or the control to commit (a `G10` offset program
   in the control's folder; a job queued on the robot's pendant). This is where tool setting
   and job sequencing should start.
4. **Commit**: perform the write through the documented interface, only when the shop turns it
   on per machine and per function, only in the documented states (cycle not running, machine
   in automatic, robot at home), with limits (e.g. an offset change larger than 0.3 mm is
   refused), read-back verification, and a log entry. **Off by default.**

This is the inspection file's "Pi records, control corrects" rule widened: corrections inside a
cycle stay in the control's macro; job and setup data can be staged by the Pi and committed by
the control or a person.

### Tool setting flow (opinion)

1. **In the presetter room:** the assembly is built from the tool library's record (tool,
   holder, stick-out from CAM's collision check). The presetter measures it; the measured values
   go into the shop instance of the record (not into the tag). The tag (RFID ID or DataMatrix
   label in the DIN 69873 position, tool-library file) carries only the assembly's ID.
2. **At the machine:** the operator or the robot loads the tool; the Pi scans the tag (handheld
   scanner, or a reader at the spindle on machines with RFID) and checks: is this assembly in
   the program's tool list, at this tool number, with stick-out ≥ the CAM minimum, length in the
   CAM window?
3. **Stage:** the Pi writes a `G10` block set (or the control's offset file) for exactly the
   tools that changed, with the assembly IDs in comments, and puts it where the control reads
   programs. The operator runs it (tier 3), or the shop has enabled commit (tier 4).
4. **Verify:** the first use of the tool in the program calls the machine's tool-setter cycle
   (generated by CAM, see below); the control macro compares with the staged value and the
   declared window, alarms if outside, and prints the result (DPRNT or equivalent); the Pi
   records both numbers.
5. **Life:** as the batch runs, tool life comes back from the control; the Pi compares it with
   the CAM tool budget and warns early.

### Robot part handling flow (opinion)

1. CAM plans the cell (next section) and emits: the NC program with handshake blocks, a **cell
   recipe** (stations, grip poses in fixture frames, grip width and force, part states, regrip
   steps), the tool list with tool budget, and the inspection plan.
2. The integrator's robot program is a **template** that reads the recipe (positions and
   widths as registers or variables), not a program SolidCraft writes in each robot maker's
   language. Opinion: generating TP, KRL, RAPID or URScript is a later, per-maker post-processor
   job, and the safety-relevant parts (zones, speeds) stay in the robot's safety configuration,
   set by the integrator.
3. The Pi queues the job; the robot and CNC run it through their own handshake; the Pi records
   every MaterialLoad/MaterialUnload and attaches it to each part's record (serial from the
   marking step in the inspection file).
4. Probing after load (inspection file) catches a badly seated part before cutting.

### How SolidCraft CAM could plan the cell

**Libraries (fits the one-library pattern of the physics-system file):**

- **Robot entry in the machine library:** model, reach, payload vs. wrist offset, joint limits,
  repeatability, mounting, controller and its interfaces (MTConnect Interfaces, vendor CNC
  interface, SRCI, ROS 2 driver), and a kinematic model. SolidCraft already **exports URDF**
  from an assembly with joints (`file.export` with `format: "urdf"`, `docs/urdf.md`); URDF
  **import** is listed there as not done yet. Importing robot URDFs (published openly by many
  robot makers and ROS packages; licence per package) would let the same simulation show the
  robot (opinion).
- **Gripper entry in the workholding library:** type (parallel, centric, vacuum, magnetic),
  stroke, force range, finger interface, finger geometry (often designed in SolidCraft for the
  part), sensors.
- **Cell instance in the shop layer:** which robot, which machine(s), station frames (raw table,
  finished rack, regrip stand, blow-off, gauge), and the machine's **handshake mapping**: which
  M-code or MTConnect Interface means MaterialLoad, OpenDoor, vise close, and so on, as data.

**Planning steps (opinion):**

1. **Part states.** From the setups (raw → after op 1 → after op 2 → finished), CAM knows the
   geometry at every handling step, as the stock-integrity file already tracks for cutting.
2. **Grip choice.** Per state, candidate faces for the chosen gripper type; check opposing
   parallel faces or a form-fit; compute required force (above); check the part against that
   force (stock integrity); check the fingers clear the vise jaws when placing.
3. **Reach and clearance.** With the robot's kinematics and the machine's 3D model (machine
   library): can the robot reach each station and the vise through the door opening, with the
   spindle at its declared load position and the tool in the spindle? This is the physics-system
   file's simulation gate extended: **no release of a cell recipe whose robot path collides or
   is out of reach.**
4. **Handshake in the program.** The post processor writes the load/unload points and the
   machine's declared handshake codes from the cell instance, never fixed numbers in the post.
   Tool-setter check cycles go in at first use of each tool.
5. **Batch readiness.** Before an unattended run, one check lists: tool budget per tool group
   (above), raw parts available, warm-up done (machine-library file's warm-up gate), probe and
   tool-setter cycles present, inspection plan present. A missing item blocks "lights-out
   ready" but not manual running.
6. **Explain every number** (physics-system file): the grip force, the tool count and the cycle
   time each show their inputs.

**Fit with SolidCraft's rules (from AGENTS.md):** everything is a command, so this would be
commands such as `cam.cell.check` or `cam.cell.recipe.export` (illustrative ids, our own dotted
form); programmatic calls never open dialogs; the Pi software, if written to share the record
model, would be Rust like the rest of SolidCraft; and nothing below the UI may depend on robot
or machine connections, so live connections belong in a separate service, not in the CAD crates
(opinion).

### Data model sketch (opinion)

```text
Cell (shop layer)
  id, name
  machine: MachineInstanceRef            # machine library
  robot: RobotInstanceRef                # machine library, kind = robot
  stations[]: { id, kind: raw|finished|regrip|blowoff|gauge|reject, frame: Pose }
  handshake: { protocol: mtconnect_interfaces|io_mcode|vendor|srci,
               map: { MaterialLoad: "...", MaterialUnload: "...", OpenDoor: "...",
                      CloseDoor: "...", ClampClose: "...", ClampOpen: "..." } }
  safety: { risk_assessment_ref, integrator, notes }   # recorded, never computed by SolidCraft

CellRecipe (per job, generated by CAM)
  part_states[]: { id, setup_ref, mass_kg }
  grips[]: { state, gripper_ref, faces[], width_mm, force_n, force_required_n, explained }
  moves[]: { from_station, to_station, state }
  tool_budget[]: { program_tool_group, parts_per_tool, tools_needed, tools_loaded }
  readiness: { warmup, raw_count, probes, toolsetter_checks, inspection_plan }

CellEvent (recorded by the Pi, appended to the part record)
  time, part_serial, interface: MaterialLoad|..., state: ACTIVE|COMPLETE|FAIL, block, source
ToolSetEvent
  time, assembly_id, tool_number, presetter: {L, D}, machine: {L, D, method}, window, result
```

## Glossary

- **APL**: Haas automatic parts loader; also the name of the Haas robot package's mode that
  starts the cycle with a load.
- **Cell recipe**: (this file) the CAM-generated data a robot template reads: stations, grips,
  moves.
- **Collaborative application**: ISO 10218-2:2025's term for a robot use where people and robot
  share space by design; replaces "collaborative robot".
- **CuttingTool / CuttingToolArchetype**: MTConnect assets for a real tool assembly and its
  catalogue description.
- **DCS**: FANUC Dual Check Safety, safety-rated position and speed monitoring in the robot
  controller.
- **G10**: G-code that writes offsets from a program; `L10–L13` select tool length/diameter
  geometry/wear on Haas mills.
- **M-FIN**: the "M-code finished" signal a control waits for before continuing.
- **MTConnect Interfaces**: Part 5 of MTConnect; request/response states for coordinating two
  pieces of equipment.
- **PL / SIL**: performance level (ISO 13849-1) / safety integrity level (IEC 62061).
- **Presetter**: off-machine instrument that measures tool assemblies.
- **QSSR**: FANUC's "Quick & Simple Startup of Robotization" for running a robot from the CNC.
- **SRCI**: Standard Robot Command Interface, PI's vendor-neutral PLC-to-robot command set.
- **Tool setter**: on-machine device (contact or laser) that measures tools in the spindle.

## Open questions

1. Should SolidCraft model robots and grippers at all, or only export what an integrator needs
   (part states, grip faces, URDF of the fixture and fingers)? The first is bigger but gives the
   collision gate; the second is a smaller first step.
2. Is MTConnect Part 5 the right internal vocabulary for handshakes, even when a machine only has
   wires and M-codes? (This file says yes.)
3. Where does URDF import sit in the roadmap? `docs/urdf.md` lists it as not done; robot
   simulation in CAM would need it.
4. How much of the Pi software belongs in the SolidCraft project? The inspection file's open
   question about an official hardware path applies here too, with a bigger liability question:
   a box that can select robot jobs must come with clear documentation that it is not a safety
   device.
5. Which presetter export formats are actually used in small shops (G10 programs, vendor files,
   DNC)? Jason's experience and other shops' replies would settle this.
6. Should tool-life budgets use the cutting-physics model, the shop's measured history, or both
   (physics prior, history update)?
7. Does any control on the market accept MTConnect Interfaces as a *responder* today, or is it
   mostly a record? (Not confirmed in this research.)

## Sources

### Standards and specifications (public pages only)

- A3, "Updated ISO 10218: Answers to Frequently Asked Questions", 2025-03-20:
  <https://www.automate.org/robotics/blogs/updated-iso-10218-faq>
- MTConnect Institute, MTConnect Standard Part 5 Interfaces, v2.2:
  <https://docs.mtconnect.org/MBSD_MTConnect_Part_5_2-2-0.pdf>
- MTConnect Institute, Part 4.1 Cutting Tools, v1.4.0:
  <https://docs.mtconnect.org/MTC_Part4_1_CuttingTools_1_4_0.pdf>
- OPC Foundation, OPC 40501-1 Machine Tools, Monitoring and Job Management, v1.02:
  <https://reference.opcfoundation.org/MachineTool/v102/docs/1?n=1>
- OPC Foundation, "VDMA releases OPC UA Companion Specifications for Robotics and Machine
  Vision": <https://opcfoundation.org/news/press-releases/vdma-releases-opc-ua-companion-specifications-robotics-machine-vision/>
- PROFIBUS & PROFINET International, SRCI:
  <https://www.profinet.com/profinet-explained/srci-standard-robot-command-interface>
- EVS catalogue entry, EN ISO 16090-1:2022: <https://www.evs.ee/en/evs-en-iso-16090-1-2022>
- ISO 13849-1, IEC 62061, ISO 13855, ISO 13857, ISO 14119, ISO 14120, ISO/TS 15066:2016,
  ANSI/A3 R15.06: catalogue knowledge only; not read.

### Vendor and product pages

- FANUC, QSSR for CNC: <https://fanuc.co.jp/en/product/robot/application/machining/qssrforcnc.html>
- FANUC America, ROS 2 driver: <https://fanucamerica.com/solutions/ros-2>
- Race Engine Technology, "Haas Robot Package 3: what's new" (Haas product text):
  <https://www.raceenginetechnology.com/Suppliers/haas-robot-package-3-whats-new>
- Haas online manual, Robot Package 1 and DCS setup (could not be fetched):
  <https://haascnc.com/service/online-manuals/haas-robot-package-1/introduction.html>
- Haas UK, programmable offsets (G10): <https://www.haas.co.uk/?p=1287>
- Haas UK, WIPS: <https://www.haas.co.uk/wips/>; Renishaw, Haas WIPS:
  <https://www.renishaw.com/en/39930.aspx>
- Renishaw, NC4 non-contact tool setter: <https://www.renishaw.com/en/nc4-non-contact-tool-setter--6099>
- Renishaw, contact tool setters (OTS, RTS, TS27R, TS34, LTS, APC): <https://www.renishaw.com/ots>
- Renishaw, tool-setting software: <https://www.renishaw.com/en/tool-setting-software--6251>
- Renishaw, "Contact tool setting for machining centres for Haas VF series controls",
  programming manual H-2000-6223-00-A, February 2002 (hosted by Haas):
  <https://www.haascnc.com/content/dam/haascnc/en/service/manual/probe/english---renishaw-contact-tool-setting---programming-manual---2002.pdf>
- Universal Robots+, VersaBuilt Robot2CNC Haas:
  <https://www.universal-robots.com/plus/products/versabuilt-robotics/robot2cnc-haas-legacy/>
- Universal Robots marketplace, SRCI:
  <https://www.universal-robots.com/marketplace/products/01tP40000071NjXIAU>
- SIAMS news on ZOLLER data transfer:
  <https://www.siams.ch/news-en/Data-Transfer-Securely-to-the-Machine-at-Top-Speed/6475>
- Robotiq, "Robotic end effectors: payload vs. grip force":
  <https://blog.robotiq.com/bid/69524/robotic-end-effectors-payload-vs-grip-force>
- Industrial Shields, Raspberry PLC safety interlocks:
  <https://www.industrialshields.com/plc-solution-library/raspberry-plc-safety-interlocks-emergency-stop>

### Community and knowledge-base sources

- Industrial Monitor Direct, "Fanuc Robot HAAS CNC Integration Methods" (secondary; used only
  for the general shape of hardwired integration):
  <https://industrialmonitordirect.com/blogs/knowledgebase/fanuc-robot-haas-cnc-integration-methods>

### SolidCraft

- `docs/urdf.md` and `crates/engine/src/cmd/file.rs` (URDF export), AGENTS.md (commands,
  layering, Rust only), ROADMAP.md (no CAM yet), at commit `10b5cc5`.
