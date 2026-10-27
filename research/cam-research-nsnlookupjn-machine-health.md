# CAM research: predicting machine failure and bad work, and ordering maintenance

Research for [#1621](https://github.com/tangency-labs/solidcraft/issues/1621) (Research Request
for Plugin: CAM). This file covers one part of that request: **using the machine monitor (loads,
torque, vibration and temperatures recorded at the machine and aligned to the G-code) to predict
machine failure and bad work before they happen, and to order maintenance checks**.

It is the "watch the machine over months" side of **Self-tuning CAM Operations** (Jason's name).
Self-tuning asks *"can this cut go harder?"* from one run to the next. This file asks *"is this
machine, hold or coolant getting worse, and what should someone check before parts are
scrapped?"* from the same data, over weeks and months.

It builds on a set of files from the same author and does not repeat them:

| Topic | File | What this file takes from it |
|---|---|---|
| System overview | [cam-research-nsnlookupjn-physics-system.md](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-vgqo3g/research/cam-research-nsnlookupjn-physics-system.md) | the per-segment predictions, the closed loop, "feedback updates the shop instance" |
| In-process inspection and SolidCraft Edge | [cam-research-nsnlookupjn-in-process-inspection.md](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-y2w06n/research/cam-research-nsnlookupjn-in-process-inspection.md) | **SolidCraft Edge (the Raspberry Pi computer at the machine), its sensors and interfaces**, alignment of data to G-code blocks and CAM segments, the per-part record, self-tuning |
| Machine library | [cam-research-nsnlookupjn-machine-library.md](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-machine-library/research/cam-research-nsnlookupjn-machine-library.md) | spindle torque/power curves, warm-up routine and cold derating, signals each control publishes, accuracy and wear held by the shop instance |
| Workholding | [cam-research-nsnlookupjn-workholding.md](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-workholding/research/cam-research-nsnlookupjn-workholding.md) | workholding-side trends (worn vise screw, jaws), slip and lift detection |
| Cutting physics | [cam-research-nsnlookupjn-cutting-physics.md](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-cutting-physics/research/cam-research-nsnlookupjn-cutting-physics.md) | predicted force, torque and power per segment; Taylor tool life |
| Tool library | [cam-research-nsnlookupjn-tool-library.md](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-tool-library/research/cam-research-nsnlookupjn-tool-library.md) | tool and holder records where wear history lives |
| CNC monitoring and control interfaces | [cam-research-nsnlookupjn-machine-interfaces.md](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-fz0g0g/research/cam-research-nsnlookupjn-machine-interfaces.md) | which controls publish spindle torque and axis load and how fast, how to read them, the safety model for writing to the control, SolidCraft at the machine on a mini PC |
| Stock integrity | [cam-research-nsnlookupjn-stock-integrity.md](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-stock-integrity/research/cam-research-nsnlookupjn-stock-integrity.md) | wall and part modes, which must not be mistaken for machine faults |

**Ownership (so nothing is said twice):** the SolidCraft Edge hardware, how it connects to each
control, and how samples are lined up with the program stay in the in-process inspection file
(the sensor kit itself is in its section [A commercial retrofit kit: upgrade any machine](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-y2w06n/research/cam-research-nsnlookupjn-in-process-inspection.md#a-commercial-retrofit-kit-upgrade-any-machine)).
The machine model and its published-signal list stay in the machine library. This file owns
**what the recorded history means for health**: failure modes and their signatures, baselines,
thresholds and trends, predicting bad work, **tool condition (chipped, broken and worn tools,
setup errors, when to stop and re-measure)**, coolant condition, and turning a finding into a
maintenance work order. Reading and writing the control is in the interfaces note.

**Name (Jason's decision):** the Raspberry Pi computer at the machine is called **SolidCraft
Edge**, because it sits on the edge of CAM and brings the machine and its automation back into
the CAM for closed-loop process improvement. It is a working name: using "SolidCraft" on a
product needs the maintainers' agreement (see the inspection file's notes on offering the
device). The device itself is defined in the inspection file's section
[SolidCraft Edge: a Raspberry Pi at the machine](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-y2w06n/research/cam-research-nsnlookupjn-in-process-inspection.md#solidcraft-edge-a-raspberry-pi-at-the-machine).

**What SolidCraft Edge does, in Jason's description, and what this file uses it for.** Jason
describes it as an edge computer on a Raspberry Pi 5, programmed in Rust, with a monitor,
keyboard, mouse and printer, built for reliability, low latency and safe continuous operation.
Its five functions, mapped to this file:

| Function (Jason's spec) | Used here for |
|---|---|
| 1. **Data acquisition** from the CNC (spindle speed, position, temperature) and physical sensors over serial, Modbus or Ethernet machine interfaces | the signals behind every finding: spindle and axis load, vibration, temperatures, coolant sensors, the fingerprint cycle |
| 2. **Machine communication**: commands and status requests to the CNC | status reads; a stop request at the next safe point or a tool-check flag, **only** where the interfaces note's [safety model](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-fz0g0g/research/cam-research-nsnlookupjn-machine-interfaces.md#writing-to-the-control-the-safety-model) allows it; never feed, speed or offset changes |
| 3. **Data transmission to the CAM station** (REST API, MQTT or WebSocket) | per-segment residuals, trends and findings sent to SolidCraft, where the machine library's shop instance and the CAM react (deratings, self-tuning hold, tool histories) |
| 4. **Visual display**: real-time dashboard of machine state, alerts and production metrics | the machine's health page at the machine: open findings, alerts graded note/watch/act/stop, tool flags, coolant state, next planned checks |
| 5. **Report printing**: production logs, maintenance summaries, quality checks | printed maintenance tickets and maintenance summaries, alongside the part travellers the inspection file prints |

The operator uses the keyboard and mouse to acknowledge alerts, enter hand checks (coolant
refractometer readings, tests done) and close maintenance tasks with what was found and done.

Licence: MIT OR Apache-2.0, the same as SolidCraft (see the Licence section of CONTRIBUTING.md).

## About this research

- **Author:** Jason ([@nsnlookupjn](https://github.com/nsnlookupjn)). Idea and direction by Jason;
  source research and writing done with an AI assistant (Claude) in October 2026.
- **How it was produced:** web searches of public sources (standards bodies' catalogue entries
  and public previews, the OPC Foundation's online reference, a NIST publication record, trade
  articles, the PHM Society's data challenge page, metalworking-fluid trade articles), each claim
  checked against the cited page where possible. No commercial CAD/CAM program was driven or
  scripted, no GPL/LGPL/AGPL source was read, and no text is copied beyond short quotes.
  Standards are described from their catalogue entries and secondary summaries, not their paid
  text, so exact clause content should be checked by whoever implements it. Worked examples are
  computed for this file from textbook formulas.
- **SolidCraft facts** were checked in the repository at commit `10b5cc5` (October 2026). There
  is no CAM code on `main` yet; see #1153 (2.5D CAM RFC) and PR #4158 (2.5D CAM core).
- **Units:** millimetres, micrometres (µm), radians; vibration velocity in mm/s RMS as the
  standards quote it; frequency in Hz; spindle speed in rev/min where a standard quotes it.
- **Marking:** "(unverified)" marks a claim not confirmed against a source. **Fact**,
  **opinion** and **marketing** are labelled where they mix.

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

> **[TODO Jason: personal experience]** How maintenance works in your shop today: what is on a
> calendar (lube, way oil, filters, coolant), what is done only when something breaks, which
> failures cost you the most (spindle rebuild, ballscrew, tool changer, coolant pump, crash
> damage), whether anyone trends anything, how you noticed the last spindle going bad, and how
> you check and top up coolant. Label this as personal experience.

## Summary

1. **CAM knows what the load should be. That is the key advantage.** A generic condition
   monitor sees a spindle load of 62 % and cannot tell whether the cut is heavy or the machine is
   sick. SolidCraft's physics system predicts the torque, power and force of every CAM segment.
   The **ratio of measured to predicted** (the *residual*) removes the job from the signal, so
   every cut of every job becomes a health test of the machine, the tool and the hold. Opinion:
   this is the most useful thing SolidCraft can add over add-on monitoring products, and it costs
   little once the prediction and the alignment exist.
2. **Three kinds of test, from cheap to rich.** (a) A **fingerprint cycle** with no cutting: a
   fixed routine of spindle speeds and axis moves, folded into the warm-up routine the machine
   library already requires before full torque, so it runs every day at no extra cost.
   (b) **Repeated cuts** of the same job and segment, compared run against run. (c) **Physics
   residuals** on any cut, compared against prediction. Each catches different faults.
3. **What fails, and what it looks like.** Spindle bearings (vibration rising, then bearing
   defect frequencies, then temperature), ballscrews and guides (axis load and reversal error
   rising, axial natural frequency falling, per NIST work), thermal drift (probe readings of a
   reference against temperature), tool wear (power rising along a tool's life), workholding
   (covered in its own file), coolant (concentration, temperature, pH; Jason's addition) and
   lubrication (way-oil consumption, alarms). A failure-mode table below maps each to signals,
   physical cause and the check to order.
4. **Thresholds have three layers.** Absolute limits from standards where they exist (ISO 20816-3
   zones for general machines; ISO/TR 17243 for machine tool spindles, measured on the housing);
   **relative to the machine's own baseline** (the more sensitive and more useful one); and
   **statistical trend tests** (EWMA, CUSUM) that catch slow drift before any fixed limit.
   A trend that is rising also gives a **time-to-limit estimate**, which is what lets a
   maintenance check be ordered for the next convenient slot instead of after a breakdown.
5. **Bad work is predicted from inspection trends, then explained by the machine data.** The
   in-process inspection file records every probed and gauged characteristic. Control charts on
   those (and on the size of wear-offset corrections) show a process drifting toward a tolerance
   limit before a part is out. The aligned machine data then says *why*: a size drifting with
   spindle temperature is thermal; a wall going thin as load rises is tool wear; a finish getting
   worse with vibration at a bearing frequency is the spindle.
6. **Coolant is a machine component (Jason's addition).** Concentration (refractometer),
   temperature, pH and level drift slowly and silently, and they change tool life, surface
   finish, thermal size and corrosion. Measured inline by sensors on SolidCraft Edge, they belong
   in the same health record and the same trend tests.
7. **The output is a maintenance order, not a dashboard.** Each finding names the asset, the
   likely cause, the evidence (with links to the G-code line and the trend), the check to make,
   how urgent it is and an estimate of time left. It is sent to whatever the shop uses: a
   computerised maintenance management system (CMMS) by its API, email, a printed ticket from the
   SolidCraft Edge, or the machine's own OPC UA `MaintenancePrognosisType` where a control supports it.
8. **The CAM reacts to health.** A machine whose shop instance carries an open finding (for
   example "spindle vibration zone C") gets derated limits in the machine library, the
   conservative-to-aggressive slider cannot push past them, and self-tuning stops pushing on
   that machine until the check is closed. Opinion: closing the loop this way is what turns
   monitoring into fewer scrapped parts.
9. **Be modest about prediction.** Fact: commercial and academic predictive maintenance relies
   on long histories and labelled failures, which a single shop rarely has. Opinion: start with
   **baselines, trends and plain rules** (explainable, little data), add learned models only
   when a fleet of shops chooses to pool labelled histories, and never let a model hide its
   evidence.
10. **Tools: three layers, and stop and re-measure when in doubt (Jason's point).** The
    control's own load monitor reacts in about a tenth of a second, and SolidCraft can give it
    the right limit per tool from the predicted load instead of a guess. SolidCraft Edge, comparing
    spindle torque, axis load and vibration with the prediction, spots a missing tool, a chipped
    tooth (once-per-revolution vibration) and wear trends, and catches setup errors on the first
    part (stock size, offsets, wrong material). The tool setter and probe confirm. When the
    evidence says a tool may be damaged, the program's own check cycle measures it at the next
    safe point and decides: continue, correct, sister tool, or stop; then the features that tool
    cut are checked.
11. **Shop infrastructure (Jason's addition).** Power quality (dips, imbalance, harmonics),
    compressed air (pressure, moisture, oil) and shop climate (temperature, gradients,
    humidity, floor vibration) affect every machine at once: thermal size, contouring
    accuracy, clamp force, spindle bearing life and probing. Logged on the same clock as
    everything else, they become causes the monitor can name, facilities work orders, and
    inputs that tighten the CAM's limits.

## Fundamentals

### Words used in this file

| Term | Meaning here |
|---|---|
| **Condition monitoring** | measuring a machine's state while it works, to find faults early (ISO 17359 frames the general programme; catalogue) |
| **Diagnosis** | naming the fault and its likely cause from the measured symptoms (ISO 13379 series; catalogue, unverified scope details) |
| **Prognosis** | estimating how long until the fault reaches a limit: the *remaining useful life* (RUL) (ISO 13381-1:2015; catalogue) |
| **Predictive maintenance** | maintenance scheduled from the measured condition and its trend, instead of a fixed calendar (*preventive*) or after failure (*corrective*); terms as used in EN 13306 (unverified wording) |
| **P-F interval** | the time between when a developing fault can first be detected (P, potential failure) and when it fails (F). The monitor is only worth having if it detects within the P-F interval with time left to plan the repair (reliability-engineering convention) |
| **Baseline / fingerprint** | the healthy reference for one machine, recorded under fixed conditions |
| **Residual** | measured value divided by (or minus) the value predicted for the same conditions |
| **Finding** | one detected change, with evidence, a likely cause, urgency and a recommended check |
| **Work order** | the request to a person to do a maintenance task, tracked to completion |

### The processing chain (ISO 13374)

ISO 13374 (condition monitoring and diagnostics of machines: data processing, communication and
presentation) describes an open chain of processing blocks, realised as the MIMOSA OSA-CBM
architecture (fact, from the catalogue entries and the VTT summary; the clause text was not
read). As commonly described the blocks are: data acquisition, data manipulation (filtering,
spectra, features), state detection (compare with baseline and limits), health assessment
(diagnosis), prognostic assessment (time left) and advisory generation (what to do).

Mapping to the SolidCraft pieces (opinion):

| ISO 13374 block | Where it lives |
|---|---|
| Data acquisition | SolidCraft Edge (in-process inspection file): control signals, added accelerometer and temperature sensors, coolant sensors |
| Data manipulation | SolidCraft Edge: per-segment features (RMS, peak, band energies, dominant frequency, mean load, temperature rise) |
| State detection | this file: baselines, residuals against the physics prediction, thresholds |
| Health assessment | this file: failure-mode table, cause ranking |
| Prognostic assessment | this file: trends and time-to-limit |
| Advisory generation | this file: the finding and the maintenance work order; and the machine library: derated limits until the check is closed |

Following an open processing model matters for one practical reason: a shop that already runs a
monitoring or maintenance system can take SolidCraft's findings at any block, rather than
replacing what it has.

### Why the physics prediction changes condition monitoring

A conventional monitor watches a signal on a machine whose work changes all day. Its limits must
be wide enough not to alarm on a heavy roughing cut, which makes it blind to a 15 % rise in
friction. Some monitors learn a profile per program to get round this, but only for repeated
jobs, and each new program starts blind.

SolidCraft's CAM predicts, for each segment, the spindle torque and power from the cutting
physics (Kienzle or edge-force coefficients, engagement), the feed forces per axis, the expected
vibration regime (stable or near a chatter boundary), and the expected cycle time from the
machine library's acceleration and jerk. With samples aligned to segments (inspection file), the
monitor can work with:

- **Process residual** `r_P = P_measured / P_predicted` per segment (and the same for axis load).
  For a healthy machine with good material and tool data it sits near a shop-specific constant.
  A rise across *all jobs* points at the machine (friction, bearings, drive); a rise across *one
  tool's life* points at tool wear; a rise across *one material lot* points at the material
  (harder lot, wrong alloy); a rise at *one direction of one hold* points at the workholding.
- **Idle residual**: spindle power at a given speed with no cut, and axis load at a given feed
  with no cut, against the machine's own baseline. These isolate the machine from the process
  entirely.
- **Vibration residual**: vibration in bands *away from* the tooth-passing frequency and its
  harmonics. Tooth passing is expected; energy elsewhere is either chatter (near a known mode) or
  the machine itself (bearing frequencies, imbalance at 1× spindle speed, gear mesh).

Opinion: separating "the job is different" from "the machine is different" is what makes trends
trustworthy on a jobbing shop's mix of work, and only a CAM that predicts loads can do it.

### Three kinds of test

| Test | How | Catches | Cost |
|---|---|---|---|
| **Fingerprint cycle** | a fixed routine with no cutting: spindle steps through set speeds (hold each for a few seconds), each axis runs set moves at set feeds in both directions, optionally a probe touch on a reference artefact on the table | spindle bearing and balance condition (vibration at each speed, idle power), axis friction and drive condition (axis load at each feed, reversal behaviour), thermal growth (reference probing), lubrication | none if folded into the warm-up routine the machine library already requires before full torque |
| **Repeated cuts** | the same program, segment and tool, run after run | everything that changes run to run, without needing the physics to be exact | free for production work |
| **Physics residuals** | any cut against its prediction | slow machine-wide drift on jobbing work; tool wear; material lot changes | free once prediction and alignment exist |

**Folding the fingerprint into the warm-up (opinion).** The machine library already stores, per
machine model, the builder's warm-up routine as steps of spindle speed and duration, and holds
the machine *cold* until it has run. Adding fixed axis moves and recording vibration and load at
each step turns the daily warm-up into the daily health check. Each run is then the same test
under the same conditions (only temperature differs, and that is recorded), which is exactly what
a baseline needs. The same idea appears in some machine builders' own offerings as a periodic
"diagnostic cycle" (unverified per builder).

## Accuracy

What limits how well this can work (fact where cited, otherwise opinion):

- **Polled control data is slow** (about 100–1,000 ms per sample, per the inspection file).
  Enough for load trends and residuals per segment; not enough for bearing frequencies or
  chatter, which need the added accelerometer sampled at kHz by SolidCraft Edge.
- **Sensor position dominates.** A housing-mounted accelerometer far from the front bearing sees
  a weak, filtered version of the bearing signal. ISO/TR 17243-2 and -3 evaluate spindle
  vibration **measured on the spindle housing** (non-rotating parts), so mount the sensor where
  those reports place it, and never move it without starting a new baseline (opinion).
- **Physics predictions have error.** Kienzle-type force predictions are often quoted within
  roughly 10–20 % with good coefficients (unverified as a general figure). That is why trends use
  the *residual's change over time* on one machine, not its absolute value: a constant model bias
  cancels out.
- **Temperature changes everything.** Viscosity of lubricants and coolant, bearing preload and
  axis friction all move with temperature. Baselines must be compared at matching thermal state
  (the warm-up gate helps) or corrected for temperature.
- **Alignment errors** smear segment features. The inspection file's alignment scheme (post
  writes segment IDs, events correct clock drift) is a precondition; with time-only alignment
  the residual approach falls back to per-operation averages.
- **False alarms cost trust.** A monitor that cries wolf gets unplugged. Opinion: require
  persistence (several runs or days) before a maintenance-level finding; alarm immediately only
  on safety-relevant events (crash-level spikes, sudden vibration jumps, overload).
- **Little labelled data.** One shop sees few failures of each kind. Generic datasets help
  method development (the PHM Society 2010 milling challenge: six 6 mm ball-nose carbide cutters,
  dynamometer, three-axis accelerometer and acoustic emission at 50 kHz, wear measured after each
  cut; no data licence stated on the page), but a model trained on one lab machine does not
  transfer to another machine without re-baselining (opinion, widely reported in the literature,
  unverified as a general rule).

## Programs

What exists today (described from public pages; no product was used or driven for this file):

- **Machine-builder and control monitoring.** Several builders and control makers offer spindle
  and axis monitoring, some with built-in sensors and periodic diagnostic cycles (marketing; which
  builder offers what is unverified per model and changes often). Patents describe spindle
  monitoring devices built into machine tools (for example US 12011793 B2, "Main spindle
  monitoring device…", and EP 2023224 A1, a control detecting spindle abnormality), which shows
  the idea is mainstream on the machine side.
- **Spindle makers.** Some motor-spindle makers fit vibration and temperature sensors and log
  overloads and crashes inside the spindle (unverified per maker).
- **Tool condition monitoring products.** Add-on systems watch spindle power or force to detect
  tool wear and breakage per program (marketing; names omitted here because this file did not
  verify their current offers).
- **Generic industrial condition monitoring and CMMS vendors.** Wireless vibration sensors with
  cloud analytics and CMMS work-order integration are widely sold for pumps, fans and motors
  (several vendor blogs in Sources; marketing). They are not machine-tool specific and do not know
  the cutting load.
- **Periodic geometric checks.** Ballbar tests (ISO 230-4 circular tests) take about ten to
  fifteen minutes and diagnose backlash, reversal spikes, squareness, servo mismatch, scale
  mismatch, cyclic error and stick-slip; trending them lets repairs be scheduled instead of
  waiting for a breakdown (MoldMaking Technology article). Shops in that article test annually,
  after a crash or move, and at the first sign of nonconforming parts.
- **Standards-based machine data.** OPC UA for Machine Tools (OPC 40501-1) exposes errors and
  warnings via alarms and conditions and has a **prognosis list**: `MaintenancePrognosisType`
  (predicted time plus the maintenance activity), tool change, utility change (coolant, filter)
  and others (OPC Foundation reference, section 8.6). MTConnect exposes CONDITION items (normal,
  warning, fault) alongside SAMPLE and EVENT data (machine library file).

**The gap (opinion):** none of these combine the *predicted* load of each move with the measured
one, the inspection results of the part that move cut, and the maintenance order. Builder systems
know the machine but not the CAM's intent; tool monitors learn one program at a time; CMMS knows
the work order but not the signal.

## Complaints and requests

Common complaints in public trade articles and forums (summarised; opinion where marked):

- **"It ran fine until it didn't."** Spindle failures are often noticed only when finish or size
  goes bad or the spindle is noisy, by which time a rebuild is needed and parts have been scrapped
  (common account in trade press; unverified as typical).
- **Calendar maintenance wastes time and still misses things.** The ballbar article notes that
  trend data can cut time spent on routine preventive maintenance because technicians often need
  fewer adjustments than scheduled.
- **Too many alarms.** Generic monitors with fixed thresholds alarm on heavy cuts (opinion; a
  common reason given for monitoring projects being abandoned, unverified).
- **Data without action.** Dashboards nobody watches. Requests in vendor and plant blogs centre on
  automatic work orders in the CMMS rather than more charts.
- **Coolant neglected.** Coolant is checked irregularly; low concentration and dirty coolant
  cause rust, short tool life and poor finish, and high concentration causes dermatitis (MSC
  article: a refractometer correction factor is needed for the true concentration; running too
  rich is linked to most dermatitis cases in smaller shops).

## What users need

### Failure modes and their signatures

Fact where cited; the signal mapping is opinion based on general condition-monitoring practice.

| Asset / failure | Physical cause | Earliest signs (monitor) | Later signs | Check to order |
|---|---|---|---|---|
| **Spindle bearings** (wear, lubrication loss, contamination, preload loss) | rolling contact damage; grease degradation; coolant ingress | rising high-frequency energy and envelope at bearing defect frequencies (BPFO, BPFI, BSF, FTF) in the fingerprint cycle; idle power rising at speed | broadband vibration in ISO/TR 17243 terms rising; front-bearing temperature rising; finish worse; noise | spindle vibration survey at fingerprint speeds; spindle runout (ISO 230-7) and drawbar force check; call spindle service |
| **Spindle imbalance / tool holder** | dirty taper, damaged holder, imbalanced tool assembly | 1× spindle speed component rising, but only with certain tools | finish marks; taper fretting | clean and inspect taper; check holder balance (ISO 21940-11 / 16084 in the cutting physics file) |
| **Drawbar / clamping** | spring stack fatigue | (unverified via signals) more 1× and 2× vibration; tool pull-out events | taper wear | measure drawbar force with a gauge (builder procedure) |
| **Ballscrew and nut** | preload loss, ball wear, contamination | **axial natural frequency falling** (NIST: early rigidity loss precedes backlash, and the axial natural frequency is an early indicator); axis load at fixed feed changing | backlash and reversal spikes in ballbar and probing; size error that depends on approach direction | ballbar test (ISO 230-4); backlash check; lubrication check |
| **Linear guides / ways** | lubrication starvation, wear, damaged wipers | axis load at fixed feed rising (friction); stick-slip at low feed | straightness and squareness change | way-lube system check; wiper inspection; ballbar |
| **Axis drives and couplings** | coupling slip, motor bearing, encoder | load ripple at a fixed feed; following error (where published) | servo alarms | builder diagnostic; coupling torque check |
| **Thermal drift** | spindle and structure growth; failing chiller | probe drift of a reference artefact vs spindle temperature changes shape compared with baseline | size drifting through a shift (inspection file SPC) | chiller and spindle cooling check; thermal compensation check (ISO 230-3) |
| **Tool wear** (not a maintenance item, but the commonest drift) | flank and crater wear, chipping | `r_P` rising along a tool's cut time; vibration rising | size and finish drift; breakage | change tool earlier; correct tool-life record in the tool library; see [Tool condition](#tool-condition-chipped-broken-and-worn-tools-jasons-point) |
| **Workholding** | worn vise screw, jaw plates, hydraulic leaks | covered in the workholding file: torque needed to pass rising, slip events | part movement | the workholding file's checks |
| **Coolant** (Jason's addition) | concentration drift, contamination, bacteria, temperature | concentration and temperature trends; pH drop | rust, odour, short tool life, poor finish, skin complaints | coolant test and correction (below) |
| **Lubrication systems** | empty reservoir, blocked line | way-oil consumption off its normal rate (if a level sensor exists); control lube alarms | guide and screw wear | lube system check |
| **Tool changer** (mention only) | worn gripper, misaligned pocket | tool-change time rising; changer alarms counted | dropped tools | builder service |

**Worked example: bearing defect frequencies (computed for this file).** For an illustrative
angular-contact spindle bearing with `Z = 20` balls, ball diameter `Bd = 11 mm`, pitch diameter
`Pd = 90 mm`, contact angle `α = 15°`, at 6,000 rev/min (shaft frequency `fr = 100 Hz`), with
`ρ = (Bd / Pd) · cos α = 0.118`:

| Frequency | Formula | Value |
|---|---|---|
| outer race (BPFO) | `Z/2 · fr · (1 − ρ)` | 882 Hz |
| inner race (BPFI) | `Z/2 · fr · (1 + ρ)` | 1,118 Hz |
| ball spin (BSF) | `Pd/(2·Bd) · fr · (1 − ρ²)` | 403 Hz |
| cage (FTF) | `fr/2 · (1 − ρ)` | 44 Hz |

A 3-flute cutter at the same speed has a tooth-passing frequency of 300 Hz, harmonics at 600,
900, 1,200 Hz. So **during a cut**, the 3rd harmonic (900 Hz) sits close to BPFO (882 Hz) and the
4th (1,200 Hz) near BPFI: bearing signs are easily confused with cutting vibration. That is the
practical reason for doing bearing diagnosis in the **fingerprint cycle with no cut** (opinion),
and for envelope analysis, which looks for impacts repeating at those rates rather than plain
tones. The bearing geometry comes from the bearing maker's data for the spindle (a field for the
machine library's shop instance, since spindles are often rebuilt with different bearings).

### Baselines

**What a baseline is (opinion).** For each machine (shop instance in the machine library), each
sensor position and each fingerprint step: the mean and spread of each feature over the first
healthy runs (for example 20 warm-up cycles), with the date, the spindle hours, the sensor
mounting and the software version. A baseline is invalidated by: a sensor moved or replaced; a
spindle rebuilt or replaced; a ballscrew replaced; a control parameter change affecting gains; a
crash. The monitor then starts a new baseline and keeps the old one for comparison ("after the
rebuild, vibration at 8,000 rev/min is 40 % of what it was before the failure").

**Per job and segment (opinion).** For repeated jobs, keep a baseline per program revision,
segment and tool type, built from the first good runs (good = parts passed inspection). A new
program revision starts a new segment baseline but inherits machine-level residual history.

**After every maintenance action (opinion).** Closing a work order records what was done; the
monitor takes the next fingerprint runs as the new reference and checks that the finding's
signal really went back toward normal. A check that did not change the signal is evidence that
the cause was something else, which is useful diagnosis in itself.

### Thresholds and trends

Three layers, all of which can raise a finding (opinion on structure; standards cited where
used):

**1. Absolute limits from standards (where they apply).**

- **ISO 20816-3** (replaced ISO 10816-3 for industrial machines over 15 kW at 120–30,000
  rev/min; the zone boundaries did not change) sorts broadband vibration velocity (RMS, 10–1,000
  Hz) into zones A (new), B (acceptable long-term), C (restricted operation) and D (damage). Its
  boundaries are, in mm/s RMS: group 2 (15–300 kW) rigid 1.4 / 2.8 / 4.5, flexible 2.3 / 4.5 /
  7.1; group 1 (over 300 kW) rigid 2.3 / 4.5 / 7.1, flexible 3.5 / 7.1 / 11.0 (Fabrico summary).
  Useful for the machine's pumps, chillers and hydraulic units; **not** written for spindles.
- **ISO/TR 17243** is the series for **machine tool spindles**: evaluating spindle vibration
  measured on the spindle housing, for testing, periodic verification and **continuous
  monitoring**. Part 2 (2017) covers direct-driven and belt-driven spindles with rolling
  bearings at 600–30,000 rev/min; Part 3 (2020) covers gear-driven spindles at 600–12,000
  rev/min. Both exclude FFT and envelope methods (an annex introduces other bearing techniques)
  and refer rotation accuracy to ISO 230-7 (ISO and SCC catalogue entries). Its limit values were
  not available from public pages; whoever implements this should read the report. Opinion:
  SolidCraft should use these reports' measurement positions and zone values as the absolute
  layer for spindles, cited and not copied.

**2. Relative to the machine's own baseline.** For each feature, alert at a set multiple of
the baseline value. Opinion: relative limits catch faults far earlier than absolute ones because
a healthy precision spindle runs well below any general zone boundary. Starting values to be
tuned by experience, for example "watch" at 1.5× baseline RMS and "act" at 2.5× for a band, and a
new frequency peak more than a set dB above the baseline spectrum (no source; to be agreed).

**3. Statistical trend tests.** For residuals and features that drift slowly:

- **EWMA** (exponentially weighted moving average) chart: smooths noise and flags a small
  persistent shift. Worked example (computed): a segment's process residual has baseline mean
  1.00 and run-to-run spread σ = 0.04; with weight λ = 0.2 and a 3-sigma control limit, the
  steady-state limit is `1.00 + 3 · 0.04 · √(λ / (2 − λ)) = 1.00 + 0.04`. A persistent rise of
  5 % in friction-plus-cutting load crosses it within a few runs, while a single noisy run of
  +8 % does not.
- **CUSUM**: accumulates small deviations; good for "something changed on this date".
- **Control-chart rules** (Shewhart with run rules; ISO 7870 series for control charts,
  unverified part numbers) on inspection data, below.

**Time to limit (prognosis).** When a feature's trend is rising steadily, fit a line (or an
exponential, which suits bearing damage better in its later stage, opinion) to the recent
history in machine hours and project when it reaches the "act" limit. Report it as a range
("between 120 and 300 spindle hours") and as calendar time at the machine's current utilisation.
Opinion: an honest range is more useful than a falsely precise RUL, and it is enough to order
the check for a planned stop.

**Persistence and urgency (opinion):**

| Level | Rule | Action |
|---|---|---|
| **Note** | a feature past "watch" once | logged on the machine's health page; no work order |
| **Watch** | past "watch" on several consecutive fingerprints, or an EWMA signal | work order "inspect" at the next planned stop; self-tuning stops pushing harder on this machine |
| **Act** | past "act", or time-to-limit shorter than the next planned stop | work order "repair/inspect" with priority; machine library limits derated; operator told at job start |
| **Stop** | sudden step change (crash-like), zone D on a spindle, temperature runaway | alarm at once through SolidCraft Edge's alarm output, which the program checks at a safe point (inspection file); machine flagged not fit for critical work |

### Predicting bad work

Bad work is a quality problem before it is a machine problem. The inspection file records every
characteristic per part; this file adds trend detection and cause ranking (opinion):

1. **Charts per characteristic.** For each probed or gauged feature: the measured value against
   its tolerance, a control chart (individuals/moving range or EWMA), and the capability index
   (Cpk). A characteristic **trending toward a limit** is flagged before it leaves tolerance,
   with an estimate of how many parts remain ("at this rate, bore Ø20 H7 reaches the upper
   limit in about 35 parts").
2. **Charts on corrections.** The in-cycle compensation in the control keeps sizes in tolerance
   by adjusting wear offsets. So the **size of the corrections** is the hidden trend: a part can
   be perfect while the machine needs ever-larger offsets to keep it so. Rising corrections on
   one tool = tool wear; on all tools in one axis = thermal or axis wear (opinion).
3. **Cause ranking from aligned machine data.** For a drifting characteristic, the monitor looks
   at the segments that cut that feature (CAM knows which; the inspection file maps them) and
   correlates the drift with: spindle and structure temperature (thermal), the process residual
   of that tool (tool wear), vibration bands (chatter or bearing), clamp torque and slip events
   (workholding), coolant concentration and temperature, and material lot. It shows the best
   correlations with their evidence, not a single verdict. Opinion: correlation is a hint, so
   the finding says "likely" and names the check that would confirm it.
4. **Surface finish.** Finish is rarely measured on the machine. Between bench measurements, the
   predicted roughness from feed and step-over plus vibration level in the tooth-passing band is
   used as a proxy (inspection file); a rise in vibration on finishing segments then predicts a
   finish problem. Correlation per shop needed (unverified how well).
5. **Pre-run risk score (opinion).** Before a critical job starts, the setup sheet shows the
   health of everything the job needs: machine findings open, spindle hours since the last
   fingerprint, tool remaining life, coolant state, warm-up done. A job with tight tolerances is
   steered away from a machine with an open thermal or spindle finding, or the sheet asks for a
   first-part check.

### Tool condition: chipped, broken and worn tools (Jason's point)

**Jason's idea:** read spindle torque and axis load in real time (through SolidCraft Edge) to
detect or predict tool failure, setup problems and generally bad cutting; detect chipped,
broken and worn tools from the machine's loads and vibration as well as probing and the tool
setter, and **if there is a possibility, stop and re-measure the tool**.

Which controls publish spindle torque and axis load, at what rate, how to read them, and how
anything may write back to the control (feed hold, stop requests) is in the companion note on
CNC monitoring and control interfaces:
[What each control maker offers](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-fz0g0g/research/cam-research-nsnlookupjn-machine-interfaces.md#what-each-control-maker-offers),
[Signal rates: what "real time" really means](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-fz0g0g/research/cam-research-nsnlookupjn-machine-interfaces.md#signal-rates-what-real-time-really-means)
and [Writing to the control: the safety model](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-fz0g0g/research/cam-research-nsnlookupjn-machine-interfaces.md#writing-to-the-control-the-safety-model).
Running SolidCraft itself beside the machine is in its section
[SolidCraft at the machine on a mini PC](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-fz0g0g/research/cam-research-nsnlookupjn-machine-interfaces.md#solidcraft-at-the-machine-on-a-mini-pc).
This section owns the **detection logic and the decision**: what each tool fault looks like,
which layer catches it, and when to stop and re-measure.

#### Three layers, three speeds

A broken tool does its damage in milliseconds to seconds; wear takes hours. No single device
covers both (opinion, following from the signal rates in the interfaces note):

| Layer | Runs where | Reacts in | Catches | Acts by |
|---|---|---|---|---|
| **1. The control's own load monitor** | inside the CNC | about 0.1 s (Haas autofeed steps every 0.1 s, per a copy of Haas's tool-monitoring guide; other controls unverified) | overload, a jammed or broken tool that spikes the load, a crash-like hit | alarm, feed hold or automatic feed reduction (Haas Setting 84: Alarm, Feedhold, Beep, Autofeed; limits per tool) or the maker's equivalent (Fanuc and others offer abnormal-load detection options; unverified per control) |
| **2. SolidCraft Edge** | the Pi computer beside the machine | polled control data: a few hundred ms to about 1 s; its own accelerometer: per spindle revolution, reduced on the Edge | missing load where load was predicted (tool gone), slow load rise (wear), one tooth doing more work than the others (chipping), chatter, setup errors | a finding; a stop request **at the next safe point** through the input the interfaces note's safety model allows (for example "stop after this tool"); never a feed or offset change on its own |
| **3. Measure the tool** | tool setter (contact or laser) and spindle probe, run by the program | seconds, between operations | broken, chipped or shortened tool for certain; length and radius change from wear; features cut by the tool out of size | the macro decides: continue, update the wear offset, switch to a sister tool, or stop for the operator |

**What SolidCraft adds to layer 1 (opinion; the biggest cheap win).** Shops leave control load
limits off or set them by guesswork because the right value depends on the cut. SolidCraft's CAM
predicts the spindle and axis load of every segment, so the post-processor can **write a load
limit per tool** (from the highest predicted load for that tool in the program, plus the
slider's margin, capped by the tool and spindle limits) into the control's tool table or as
program settings, and choose the action. Where a control allows the limit to change inside the
program (by macro variable or a command; unverified per control), the post can lower it for
light finishing segments, where a single per-tool limit set for roughing is far too high to
notice anything. The fast reaction stays inside the control, where it is deterministic; the
CAM only supplies better numbers. How the limits get into each control is in the interfaces note's section
[Posting limits into the control](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-fz0g0g/research/cam-research-nsnlookupjn-machine-interfaces.md#posting-limits-into-the-control).

**Why SolidCraft Edge cannot be the breakage guard.** At 1,200 mm/min the tool moves 10 mm in a
0.5 s poll (computed). A broken end mill can be dragged that far before the Edge even sees the
sample, then the request still has to reach a safe point. So the Edge detects, records and
stops the *next* damage (a missing tool going into the next feature, a whole batch cut with a
chipped tool); the control's monitor and the tool setter stop the immediate damage.

#### What each fault looks like

All signals are compared with the per-segment prediction (the residual `r_P` from
[Why the physics prediction changes condition monitoring](#why-the-physics-prediction-changes-condition-monitoring)),
which is what lets the monitor tell an unusual cut from an unusual tool.

| Fault | Spindle torque / power | Axis load | Vibration (SolidCraft Edge accelerometer) | Tool setter / probing | Confidence from load alone |
|---|---|---|---|---|---|
| **Broken tool** (gone or snapped short) | a spike or step, then load near idle where cutting was predicted (`r_P` ≈ 0 on following segments) | feed-direction load near zero | burst at the break, then no tooth-passing content | length short or beam not broken: certain | high when predicted load is well above idle; low for small tools (below) |
| **Chipped edge** (one tooth damaged) | small rise in mean load; often hidden at polled rates | small rise | **once-per-revolution component grows** relative to tooth passing, because one tooth now cuts more or less than the others (same signature as runout) | laser setter radius or profile check may see it (unverified for small chips); probed finish and size worse | low; needs the accelerometer |
| **Worn tool** (flank wear) | `r_P` rises slowly over the tool's cut time; sharper rise near end of life (rule of thumb, unverified as general) | feed and radial loads rise (rubbing) | broadband level rises; finish proxy worse | length or radius smaller by tens of µm (laser setter measures radius); probed walls thicker as deflection rises; wear-offset corrections growing | medium; best from trends across the tool's life |
| **Built-up edge / chip packing** (aluminium, deep slots) | erratic rises that clear and return | erratic | broadband, irregular | finish and size scatter | medium |
| **Chatter** | mean load may barely change | — | a strong peak **not** at tooth passing or its harmonics, near a known tool or machine mode (cutting physics, machine library) | finish marks | none from load; needs vibration |
| **Wrong tool loaded** | residual pattern consistently off for that tool's segments (bigger tool: higher; smaller: lower) | same | tooth-passing frequency differs if the flute count differs | length/diameter mismatch with the tool record: certain | medium |

**Worked example: seeing a chipped tooth (computed for this file).** A 4-flute end mill at
8,000 rev/min has a tooth-passing frequency of `8000 · 4 / 60 = 533 Hz` and a once-per-revolution
frequency of 133 Hz. With all four teeth sharp and little runout, the vibration is dominated by
533 Hz and its harmonics. A chipped tooth makes one tooth cut differently every revolution, so
energy grows at 133 Hz and its multiples that are not multiples of 533 Hz (266, 400 Hz). The
box tracks the ratio of once-per-revolution to tooth-passing energy per segment against the same
tool's history: a step change mid-batch is a chipping event; a value high from the first cut is
runout from the holder (cutting physics file). Spindle load polled at 1 Hz cannot show any of
this.

**Small tools are the hard case.** A 1 mm drill or end mill may need a few watts of cutting power
against a spindle that idles at hundreds of watts (opinion; idle power varies by machine), so
its break hardly changes the spindle load. For those tools the reliable checks are the tool
setter (laser setters are made to check tools under a millimetre for breakage, per Renishaw's
TRS2 page) and acoustic or vibration sensing close to the cut. The CAM knows which tools are
too small for load monitoring (predicted load below a set fraction of the spindle's idle
power) and should plan a tool-setter check after them instead.

#### Setup problems and bad cutting

The same residuals catch setup errors on the first part, often within the first operation
(opinion; the timings come from the CAM's predicted engagement):

| Problem | What the monitor sees | Check |
|---|---|---|
| **Stock larger or smaller than modelled, or wrong work offset** | load starts earlier or later than the predicted engagement on entry moves; the shift in time × feed gives the size error (computed example: load starts 0.25 s early at 1,200 mm/min → about 5 mm more stock than modelled) | probe the stock before roughing (inspection file) |
| **Wrong tool length offset** | Z plunges and ramps start cutting early or late; floors cut by that tool are deep or shallow | re-measure the tool on the setter |
| **Part not seated, slipping, lifting** | load drops or steps unexpectedly in one direction (workholding file) | stop at a safe point; re-clamp; probe the datum |
| **Wrong material or a hard lot** | residual off by a similar factor for every tool from the first cuts | check the material certificate; update `kc1.1` for the lot (cutting physics file) |
| **Coolant off or misdirected** | residual and spindle temperature rising together; finish worse | coolant check (coolant section below) |
| **Operator overriding feed or speed** | actual feed or speed differs from programmed (published by most controls) | not a fault; recorded so residuals are computed at the real feed |
| **Air cutting** (program too conservative or stock already removed) | idle-level load on segments planned as cuts | CAM: refine the in-process stock model; self-tuning candidate |

#### Stop and re-measure: the decision

**Jason's rule: if there is a possibility, stop and re-measure the tool.** The question is how to
make that cheap enough that it happens every time it should, and rare enough that the machine
keeps cutting. Opinion:

1. **Each tool gets a check policy** in the program, chosen by the CAM from the tool and the risk:
   - *always after use*: small drills and taps, tools whose breakage the load cannot see, and the
     last tool before a costly feature or a finished surface;
   - *on suspicion*: most tools; checked when the control's monitor or SolidCraft Edge raises a flag;
   - *by count*: every N parts or N minutes of cut, where wear is the main risk.
2. **Suspicion comes from evidence, graded:**

   | Evidence | Example | Response |
   |---|---|---|
   | Certain | the control's load monitor tripped; tool setter beam not broken | stop (control's own action); operator |
   | Strong | `r_P` near zero on segments predicted to cut; vibration burst followed by silence | request a stop at the next safe point; tool-setter check before the next tool touches the part |
   | Moderate | once-per-revolution energy stepped up; `r_P` up by more than its EWMA limit; probed size jumped | tool-setter length and radius check at the next tool change; probe the features this tool cut |
   | Weak | slow wear trend inside limits | no stop; change the tool earlier at the next planned point; adjust the tool-life count |

3. **The check is a cycle the post already wrote.** Opinion: the program carries, after each
   tool (or at chosen safe points), a short conditional call: *if the check flag for this tool is
   set, go to the tool setter, measure length (and radius on a laser setter), compare with the
   stored values and the limits, then continue, update the wear offset, call a sister tool, or
   stop with a message*. The flag is set by the control's own monitor, by a macro variable the
   SolidCraft Edge may write **only where the interfaces note's safety model allows it**, or by the
   operator. The decision itself stays in the control's macro, the same principle as in-cycle
   compensation in the inspection file.
4. **Then check what the tool already cut.** A tool that measured short or chipped may have
   left features out of size. The CAM knows which features that tool cut since its last good
   measurement, so the program probes those (or the part record marks them for bench
   inspection) before the part moves on.
5. **Record everything.** Each flag, check result and decision goes into the part record and the
   tool's history in the tool library (shop instance): tool life used, how it ended (worn,
   chipped, broken), and the load trend before it. That history is what lets the CAM predict
   the next failure of the same tool type and set the *by count* interval from evidence instead
   of guesswork.

**Predicting tool failure, not just detecting it (opinion).** The Taylor tool-life estimate in
the cutting physics file gives the starting expected life; the wear trend of the current tool
(residual rise per minute of cut, setter radius loss, probed deflection) updates it as the tool
works; the shop's history of the same tool type in the same material gives the spread. Together
they give a remaining-life estimate with a range, and the CAM can schedule the tool change or the
re-measure at a convenient safe point *before* the predicted end of life, rather than at a fixed
count. The datasets in the Accuracy section (PHM 2010 milling) are the kind of data used to
develop such estimators; a shop's own history is what calibrates them.

### Coolant condition (Jason's addition)

Jason's idea (other thread): the SolidCraft Edge kit could include machine sensors (vibration,
temperature) and **measure coolant refraction (concentration) and temperature**, "to really
upgrade any machine while optimizing production and quality". The hardware, sensor choice,
costs and selling paths are in the inspection file's section [A commercial retrofit kit: upgrade any machine](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-y2w06n/research/cam-research-nsnlookupjn-in-process-inspection.md#a-commercial-retrofit-kit-upgrade-any-machine); here is how
those signals feed failure and bad-work prediction.

**What to measure and why (fact where cited):**

| Parameter | How | Why it matters |
|---|---|---|
| **Concentration** | refractometer (hand or inline); reading × the fluid's correction factor = true concentration (MSC: a reading of 6 % with a factor of 2 is 12 %). Inline process refractometers exist (for example ATAGO's inline Brix meters; suitability for emulsions unverified) | concentration is the most important coolant test (STLE webinar summary). Too lean: corrosion, bacteria, short tool life (STLE example: correcting low concentration reduced tooling cost, bacteria and tramp oil). Too rich: dermatitis, foaming, residue (MSC) |
| **Temperature** | a probe in the sump or return | coolant temperature changes the part's and the machine's temperature, so it drives size (thermal growth of the part and fixture); warm sumps grow bacteria faster (opinion, general microbiology) |
| **pH** | probe (needs calibration and cleaning) or test strip | typically kept at 8.5–10.0 to prevent ferrous corrosion; drift signals wrong concentration, biological growth or contamination (STLE) |
| **Conductivity** | probe | large change warns of emulsion instability and hard water (STLE) |
| **Level / top-up** | level sensor, or log of top-ups | evaporation and carry-off change concentration; top-up mix differs by material (MSC: about 2 % for heat-producing alloys, 3 % for aluminium) |
| Tramp oil, bacteria | centrifuge, dip slides, ATP tests (STLE) | not practical inline (opinion); stay as logged manual tests |

**How coolant feeds the predictions (opinion):**

- **As a cause in bad-work ranking.** A size drift that follows coolant temperature through the
  day (part growth) is fixed by coolant cooling or by probing after the part settles, not by the
  machine. A finish or tool-life change that starts when concentration drops points at coolant.
- **As a modifier of tool life.** The tool library's Taylor tool-life values assume the coolant
  is in range. When concentration is out of range, expected tool life is shortened (factor to
  be learned per shop; no source), and tool wear alarms come earlier.
- **As its own maintenance item.** Findings like "concentration below range for 3 days",
  "pH falling for a week", "sump temperature above set point" order a coolant check or
  top-up, with the amount of concentrate to add computed from sump volume and the measured
  concentration (simple mixing arithmetic; opinion that showing the amount saves errors).
- **As a corrosion risk on the part record.** Parts that sat wet in lean coolant get a note in
  their traveller (corrosion risk, especially cast iron and steel).

**Caution (opinion).** Inline refractometers read the oil phase of the emulsion; contamination
by tramp oil or other fluids raises the reading and hides a lean mix. A periodic titration or
hand refractometer check, logged as a manual test, keeps the inline sensor honest.

### Shop infrastructure: power, air and climate (Jason's addition)

**Jason's idea:** track infrastructure problems such as **power quality, air quality and
temperature**, and how they affect the whole physics system. A machine can be healthy and still
make bad parts or fail early because what feeds it is not. These causes are invisible to the
machine's own alarms until something trips, and they usually hit **every machine in the shop at
once**, which is itself the strongest clue (opinion).

How conditions enter the limits: the [CAM physics system](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-vgqo3g/research/cam-research-nsnlookupjn-physics-system.md#the-shared-setup)
carries the setup the solver plans against, and the [machine library](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-machine-library/research/cam-research-nsnlookupjn-machine-library.md#accuracy-thermal-growth-and-wear)
holds thermal growth and the [warm-up gate](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-machine-library/research/cam-research-nsnlookupjn-machine-library.md#warm-up-before-maximum-torque-jasons-requirement).
This section proposes that the shop's measured conditions become one more input to that setup
(a *shop conditions* record per site and per machine), so the same plan is judged against the
conditions it will actually run in. The sensor hardware belongs to the inspection file's
[A commercial retrofit kit: upgrade any machine](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-y2w06n/research/cam-research-nsnlookupjn-in-process-inspection.md#a-commercial-retrofit-kit-upgrade-any-machine);
the list below is what this file needs from it.

#### What each condition does to the physics

| Condition | Physical effect | Shows up as | Who it hurts most |
|---|---|---|---|
| **Voltage sags (dips)** | drive DC bus and control supply drop for a few cycles | control shutdown and re-home, or worse, a machine that rides through but contours badly: NIST tests (Damazo, Donmez, Soons et al., 2003) found 40 % five-cycle sags degraded contouring accuracy before the machine faulted, so a part can be out of tolerance with no alarm | finishing passes, long unattended runs |
| **Swells, transients** (switching of big loads, lightning) | stress on drive electronics and power supplies | drive and power-supply failures over time (opinion, general electrical practice; unverified rate) | drives, spindle amplifiers |
| **Harmonics** (from variable-speed drives on the same supply) | extra heating in motors and transformers; possible interference with encoders and sensors | warmer motors, nuisance faults (opinion, unverified per machine) | spindle and axis motors, sensitive measuring circuits |
| **Phase imbalance, low or high supply voltage** | unequal phase currents, motor heating; less torque headroom at the top of the spindle curve (opinion) | spindle load higher than predicted on the same cut; overload alarms near the curve limit | heavy roughing at the spindle limit |
| **Compressed air pressure low or unstable** | tool changer, drawbar release, pallet and fixture clamps, spindle air seal and probe/tool-setter air blast all rely on it (per machine, unverified) | slow or failed tool changes; fixtures that clamp below their rated force (workholding file); dirty probe or setter readings | pneumatic workholding, tool changes, probing |
| **Wet or oily compressed air** | water and oil carried into spindle air seals, valves and air blasts | spindle bearing contamination over months (a cause in the failure-mode table); corroded valves; contaminated probe and tool-setter optics; rust on parts | spindle bearings, laser tool setters, finished steel parts |
| **Shop air: dust, oil mist** | dirt in electrical cabinets and filters; mist from coolant | cabinet overheating, filter clogging, sticky surfaces (opinion); also a health issue for people, outside this file | control cabinets, chillers |
| **Ambient temperature (level and change)** | the machine structure, the part and the measuring scales all grow or shrink; ISO 230-3 has a dedicated *environmental temperature variation error* (ETVE) test for this | size drift through the day; first parts after the weekend different from Friday's; probing errors versus a part that is not at 20 °C | tight-tolerance work, large parts, aluminium |
| **Temperature gradients** (sun on one side, doors, heaters, the floor colder than the air) | the machine bends rather than just grows | squareness and position errors that change with time of day (opinion) | large machines, 5-axis |
| **Humidity and condensation** | condensation in cabinets and on cold parts when warm humid air meets cold surfaces | electrical faults, corrosion of parts and fixtures; coolant water balance (opinion) | electronics, cast iron |
| **Floor vibration** (nearby presses, forklifts, other machines) | adds to the tool-to-part vibration | finish marks unrelated to the program; probing scatter; a vibration feature in the fingerprint that comes and goes with the neighbour's work (opinion) | finishing, probing, grinding |

**Reference temperature (fact).** Dimensions on a drawing are defined at **20 °C** (ISO 1). A
part measured warm is not wrong, it is just not at reference conditions: a 300 mm steel part
3 K above 20 °C is about **10 µm longer**, aluminium about **21 µm** (computed with 11.5 and
23 µm/(m·K)). A 1 K front-to-back gradient through a 400 mm deep, 600 mm tall steel column tips
its top by about **5 µm** (computed, simple bending estimate). On a ±0.01 mm feature both are the
whole tolerance. This is why temperature appears here as a cause of bad work, not just as a
comfort figure.

#### Signatures: telling the shop from the machine

The key test (opinion): **does the change appear on several machines at the same time?**

| Pattern | Likely cause |
|---|---|
| a step in idle power, spindle temperature or residual on **every** machine at once | supply voltage or ambient temperature |
| control resets, drive faults or tool-change failures clustered at the same time across machines | a sag or an air pressure drop (check the logged supply and air data at that moment) |
| size drift that follows ambient temperature (or the clock) on all machines doing tight work | shop climate; compare with each machine's own thermal model |
| only one machine affected | the machine, its tooling or its local supply (one phase, one air drop, a door near it) |
| problems that start when a neighbouring machine or the compressor starts | shared supply, shared air, or floor vibration |

Because every record from SolidCraft Edge already carries a UTC timestamp (inspection file's
alignment keys), shop conditions logged with the same clock line up with every segment, every
probe result and every alarm with no extra work.

#### Read the machine's own sensors first (Jason's point)

Many machines already measure their environment: thermal compensation sensors on the spindle,
structure and sometimes ambient air; spindle and coolant temperatures; air pressure switches or
transducers; and, on some controls, supply voltage and power monitoring (which ones varies by
builder, model and options; unverified per machine). The rule (opinion): **SolidCraft Edge reads
what the control already publishes first, and external sensors only fill the gaps.** The
[machine library](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-machine-library/research/cam-research-nsnlookupjn-machine-library.md#signals-the-machine-publishes-for-the-closed-loop)
already lists, per machine model and control version, each published signal with its unit,
meaning and update rate; environment signals go in the same list (for example "ambient
temperature, °C, from the thermal compensation sensor, 1 s"). How to read them per control is in
the interfaces note's
[What each control maker offers](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-fz0g0g/research/cam-research-nsnlookupjn-machine-interfaces.md#what-each-control-maker-offers).
When the shop sets up a machine, SolidCraft compares the conditions this file wants with what the
entry says the machine publishes, and shows the shortlist of external sensors still needed for
that machine (often only the shop-level ones: supply quality, dryer dew point, a reference
ambient sensor away from the machine). Built-in sensors have two further advantages: they are
already calibrated into the builder's thermal compensation, and they need no wiring. A pressure
*switch* only says above or below a threshold, so an inlet transducer is still worth adding where
pneumatic clamp force matters (opinion).

#### Sensors SolidCraft Edge would add where the machine lacks them (for the inspection file's kit)

Opinion, cheapest useful set first; only for signals the machine does not already publish.
Hardware choice, cost and wiring stay in the inspection file:

| Measure | Sensor type | Rate | Where |
|---|---|---|---|
| Supply voltage per phase, dips and swells, imbalance, harmonics (THD) | a three-phase power-quality meter with Modbus or Ethernet output, ideally measuring to IEC 61000-4-30 (class S or A; the standard defines the measurement methods, unverified which class a given meter meets); or clip-on current transformers for current only | event capture for dips; 1 s to 10 min summaries | machine's main supply, inside the cabinet (by an electrician) |
| Compressed air pressure | 4–20 mA or I²C pressure transducer | 1–10 Hz (catches drops during tool changes) | at the machine's air inlet, after its regulator |
| Compressed air dew point (moisture) | dew-point sensor | minutes | air inlet; shop-level sensor on the dryer outlet |
| Compressed air oil and particles | not practical inline for a small kit (opinion); log the dryer and filter service and periodic tests to **ISO 8573-1** classes (particles, water as pressure dew point, oil; written as e.g. Class 1.2.1) | manual, by schedule | — |
| Ambient temperature | several cheap digital sensors: near the spindle head height, at the table, near the floor, and one away from the machine | 1 per minute | on and around the machine |
| Machine structure temperature | stick-on sensors on the column and bed (optional) | 1 per minute | structure |
| Humidity | combined temperature/humidity sensor | 1 per minute | in the shop and inside the electrical cabinet |
| Shop air particulates (PM2.5/PM10) | low-cost particle sensor (optional) | minutes | near the cabinet air intake |
| Floor vibration | the kit's accelerometer on the floor or base, or a second one | fingerprint and idle periods | machine base |

**One per shop or one per machine?** Opinion: supply and compressed air quality are often shared,
so one shop-level SolidCraft Edge (or a shared sensor) can serve many machines, while air
pressure at the inlet, ambient temperature at the machine and the cabinet humidity are worth
having per machine.

#### Findings and maintenance orders

Shop-condition findings go through the same levels and work orders as machine findings
([From finding to maintenance order](#from-finding-to-maintenance-order)), but to a different
owner: **facilities** rather than the machine's maintenance (opinion). Examples:

- "Voltage dips below 80 % on L2 three times this week, all during the compressor start; two
  coincide with control resets on VF2-01 and VF2-02" → electrician: check supply and compressor
  starter; consider ride-through or UPS for the controls.
- "Air pressure at VF2-01 inlet falls to 5.2 bar during tool changes" → check the drop, regulator
  and receiver size; the workholding file's pneumatic clamp force is derated until fixed.
- "Dryer outlet dew point rising for 10 days" → service the dryer before water reaches spindle
  air seals.
- "Shop temperature swung 6 K on Monday morning; bores cut before 08:30 measured 8 µm smaller on
  the CMM" → heating schedule; first-part check after cold starts.
- "Cabinet humidity above 70 % with the door seal open" → close and check the cabinet cooler.

#### How the CAM reacts to shop conditions

Opinion, following the same rule as machine health (conditions can tighten limits, never loosen
the hard ones):

- **Thermal state in the setup.** Measured ambient and structure temperatures, alongside the
  machine library's thermal model and warm-up state, decide whether the tightest finishing
  passes and in-process probing should wait, and whether probed values are corrected to 20 °C
  using the part material's expansion coefficient.
- **Air-dependent limits.** Pneumatic clamp force from the workholding library is computed at
  the **measured** inlet pressure, not the nominal one; if pressure is below the fixture's
  minimum, the job is blocked like any other hard workholding limit.
- **Power-dependent warnings.** A shop with a recent record of dips gets a setup-sheet warning
  for long unattended jobs, and the program can add re-probing after any control reset (a reset
  mid-cycle makes the in-process stock state uncertain).
- **Residuals cleaned of shop effects.** Supply voltage and ambient temperature are recorded
  with every segment, so trends in the machine's own residuals can be corrected for them
  instead of blaming the machine for the weather.
- **Scheduling (opinion).** Tight-tolerance finishing is planned for the thermally stable part
  of the day if the shop's temperature history shows a pattern.

### From finding to maintenance order

**What a finding carries (opinion):**

```json
{
  "finding": "F-2026-0142",
  "asset": "VF2-01/spindle",
  "level": "watch",
  "detected": "2026-10-26T06:10:00Z",
  "signal": "fingerprint step 8000 rpm: envelope energy at BPFO band",
  "evidence": {
    "baseline": 0.12, "current": 0.31, "unit": "g (envelope RMS)",
    "trend": "rising 11 % per 100 spindle hours since 2026-09-14",
    "links": ["trend chart", "fingerprint spectra", "last 3 jobs: segments with vibration residual > 1.5"]
  },
  "likely_cause": [
    {"cause": "front bearing outer race wear", "confidence": "medium"},
    {"cause": "contamination of bearing grease", "confidence": "low"}
  ],
  "time_to_act": {"spindle_hours": [180, 420], "calendar": "about 4 to 9 weeks at current use"},
  "check": "spindle vibration survey at fingerprint speeds; inspect front bearing temperature; consult spindle service",
  "cam_effect": "derate spindle P and M to 80 % above 8000 rpm; no self-tuning increases on this machine",
  "work_order": {"system": "shop CMMS", "id": null, "state": "to send"}
}
```

**Where it goes (opinion; the shop chooses):**

- **A CMMS through its API.** Most maintenance systems accept work orders by REST API or webhook
  (vendor guides in Sources; marketing). SolidCraft (or the SolidCraft Edge service) should send a
  plain, documented record and keep the CMMS's ID; it should not become a CMMS.
- **OPC UA prognosis.** Where the machine has an OPC UA server per OPC 40501-1, the machine's own
  `MaintenancePrognosisType` entries (predicted time, activity) can be read and shown next to
  SolidCraft's findings, so builder-scheduled maintenance and condition-based findings sit on one
  list. Writing SolidCraft's own prognoses into a control's server is not something the standard
  invites (opinion, unverified); keep them in SolidCraft's record and the CMMS.
- **No CMMS.** An alert on SolidCraft Edge's dashboard at the machine, email, a message to a
  chat channel, or a printed maintenance ticket from SolidCraft Edge's printer (the same printer
  that prints part travellers), with a QR code back to the evidence. A weekly printed
  **maintenance summary** per machine (open findings, trends, work done, next checks) suits shops
  that run maintenance from a clipboard (opinion).
- **The machine's health page**, in SolidCraft on the CAM station and on SolidCraft Edge's
  dashboard at the machine: open findings, trend charts, the last fingerprint, maintenance done,
  and the next planned checks; the shop instance in the machine library holds them, and
  SolidCraft Edge keeps a local copy so the page works when the network is down (opinion).

**Closing the loop.** The person who does the work closes the order with what they found and
did (bearing replaced, lube line cleared, coolant corrected, nothing found). The monitor checks
the next fingerprints, and the finding closes only when the signal agrees. "Nothing found" with
an unchanged signal escalates to a second, different check (opinion).

**Calendar maintenance still matters (opinion).** Lubrication, filter changes, coolant tank
cleaning and annual geometry checks keep their calendars or hour counters. The monitor can
adjust intervals (stretch them when trends are flat, as the ballbar article suggests; shorten
them when they are not), but it should not remove the builder's schedule.

### How CAM reacts to machine health

This is where SolidCraft differs from a standalone monitor (opinion):

- **Derating.** An open finding can attach a derating to the shop instance in the machine library
  (spindle power and torque above a speed, axis thrust, maximum acceleration). The limits engine
  in the CAM physics system takes the minimum as always, so the binding limit shown to the
  programmer reads "spindle derated: open finding F-2026-0142". The slider cannot lift it.
- **Self-tuning holds.** While a machine has a *watch* or *act* finding, self-tuning proposes no
  increases on it, and the next tuning baseline after the repair starts fresh.
- **Job routing.** The setup sheet warns when a tight-tolerance or long-running job is assigned to
  a machine with open findings, and suggests another instance of a comparable machine if the shop
  has one.
- **Programs as tests.** Because every segment has a predicted load, the CAM can add a short
  "health check" segment at the start of a program (a fingerprint subset, a light cut on the
  stock top at known engagement) when the shop wants more evidence, the same way it adds probing
  cycles. Opinion: optional, off by default, and shown in the cycle time.

### Data model sketch (opinion)

Per shop instance of a machine (machine library), additions only:

| Field | Content |
|---|---|
| `sensors[]` | type, position (named point on the machine model), axis orientation, sample rate, mounting date |
| `bearings[]` | spindle bearing geometry (Z, Bd, Pd, α) per position, with source and date (spindles get rebuilt) |
| `fingerprint` | the routine (spindle speeds, axis moves, reference probing), versioned |
| `baselines[]` | per feature and step: mean, spread, valid from/to, reason for a new baseline |
| `findings[]` | as above, with state (open, watch, act, closed) and work order link |
| `maintenance[]` | what was done, when, by whom, at what spindle hours |
| `deratings[]` | limits reduced by open findings |
| `coolant` | fluid, correction factor, target range, sump volume, sensor list |

Per job and segment: residual history keyed by the same program, block and segment keys the
inspection file defines.

### Privacy, trust and sharing (opinion)

- **Local first.** Health data stays on SolidCraft Edge and the shop's own storage; nothing goes to a
  vendor cloud unless the shop chooses (same principle as the inspection file).
- **Shared learning is opt-in and labelled.** A shop may contribute anonymised trend histories and
  confirmed failure labels ("bearing replaced at 6,200 spindle hours, BPFO envelope 4× baseline")
  to a shared machine-library entry, labelled as measured, to give other owners of the same model
  better starting limits. This is the same "shop instance → shared catalogue only by choice"
  rule as the other files.
- **Explain every finding.** No finding without its evidence, its trend and the rule that fired.
  Opinion: an unexplained "health score 62" is worse than nothing in a shop.
- **Not a safety system.** The monitor is advisory. Machine safety (guards, overload trips, crash
  detection inside the drive) stays with the machine builder and the control.

### Phased plan

Fits the CAM physics system's phases; each step is useful on its own (opinion):

| Step | Adds | Needs | Useful because |
|---|---|---|---|
| H0 | health fields in the machine library's shop instance; maintenance log | machine library | a place for maintenance history from day one |
| H1 | fingerprint folded into the warm-up; idle power, axis load and vibration per step; baselines; relative thresholds | SolidCraft Edge with accelerometer; warm-up gate | catches spindle and axis degradation with no CAM prediction at all |
| H2 | findings and work orders (CMMS API, email, printed ticket); closing the loop after maintenance | H1 | turns data into action |
| H3 | per-segment residuals and EWMA trends on production cuts | physics predictions per segment; G-code alignment | machine-wide drift and tool wear on jobbing work |
| H4 | bad-work prediction: control charts on characteristics and corrections, cause ranking | in-process inspection records | catches drift before a part is out of tolerance |
| H3b | tool condition: per-tool load limits written by the post into the control's monitor; tool check cycles with check policies; missing-tool, chipping and setup-error detection; tool histories | post-processor; tool setter; SolidCraft Edge accelerometer for chipping | fewer broken tools carried into the next feature, setup errors caught on the first part |
| H5b | shop conditions: supply power quality, air pressure and dew point, ambient temperature and humidity; facilities findings; measured air pressure in clamp force; temperature correction of probing | sensor kit (inspection file); workholding library | causes shared by all machines are named instead of blamed on one |
| H5 | coolant sensors and coolant findings | sensor kit (inspection file) | removes a silent cause of bad work |
| H6 | CAM reacts: deratings, self-tuning hold, job routing | limits engine | the monitor changes what the CAM does |
| H7 | opt-in shared histories per machine model; learned models where pooled data allows | many shops | better starting limits for everyone |

### Fit with SolidCraft

- **Everything is a command** (AGENTS.md). Opinion: health would appear as commands such as
  `machine.health.findings`, `machine.health.fingerprint.record`, `machine.maintenance.log`
  (illustrative ids following the `<area>.<thing>[.<variant>]` rule), so the UI, CLI, scripts
  and the control channel reach the same functions.
- **Never crash.** Sensor data is input-derived and hostile: finite checks, caps on sample
  counts and spectrum sizes, `Result` everywhere (AGENTS.md rules). Missing or corrupted health
  data must never stop a CAM job from being planned; it only removes the health-based deratings
  and says so.
- **Rust only, wasm-safe below the apps.** Feature extraction (RMS, spectra, envelope, EWMA) is
  ordinary numeric code that fits the engine layers. SolidCraft Edge is programmed in Rust
  (Jason's spec), so the same feature, residual and trend code can be shared crates used by
  both SolidCraft and SolidCraft Edge, with the device-specific parts (drivers, display,
  printing) in SolidCraft Edge alone (opinion).

## Glossary

| Term | Meaning |
|---|---|
| BPFO / BPFI | ball pass frequency, outer / inner race: the rate at which rolling elements pass a point on the race |
| BSF | ball spin frequency |
| FTF | fundamental train (cage) frequency |
| CMMS | computerised maintenance management system: tracks assets, work orders and maintenance history |
| CUSUM | cumulative sum control chart, sensitive to small persistent shifts |
| Cpk | process capability index relative to the nearer tolerance limit |
| Envelope analysis | demodulating high-frequency vibration to find repetition rates of impacts, used for rolling bearings |
| EWMA | exponentially weighted moving average chart |
| Check policy | when a tool is measured on the tool setter: always after use, on suspicion, or by count |
| Chipping | loss of a small piece of a cutting edge; one tooth then cuts differently from the others |
| Fingerprint cycle | a fixed no-cut routine run regularly to compare a machine with its own baseline |
| P-F interval | time from first detectable sign (P) to functional failure (F) |
| Residual | measured divided by predicted, for the same segment or condition |
| RUL | remaining useful life |
| Zone A–D | ISO 20816 vibration severity zones: new, acceptable long-term, restricted, damage |

## Open questions

1. Should SolidCraft hold maintenance history at all, or only findings with a link to the shop's
   CMMS? (Opinion: a small log in the machine shop instance, because many small shops have no
   CMMS.)
2. Where do the spindle absolute limits come from in practice: ISO/TR 17243 values, the spindle
   maker's values per model, or both? Is it acceptable to store builder values in a shared
   machine-library entry (they are facts, but the source document's terms may restrict
   copying)?
3. How long a baseline is enough? A fixed number of warm-up runs, or until the spread settles?
4. Should the fingerprint add time to the builder's warm-up, or only record what the warm-up
   already does?
5. Which controls let a program change the load-monitor limit per segment, and which let an
   outside device set a check flag safely? (Interfaces note.)
6. Is a printed maintenance ticket from SolidCraft Edge useful, or does everything go to phones?
7. Which coolant sensors are robust enough inline (refractometer fouling, pH probe drift) for a
   small-shop kit? Needs hands-on testing.
8. How should SolidCraft treat a finding on a machine owned by someone else (a subcontractor's
   machine running SolidCraft programs)?

> **[TODO Jason: personal experience]** Which of these findings would have saved you money in
> the last few years, and which would you ignore as noise?

## Sources

### Standards (catalogue entries and public previews only)

- ISO 13374-1:2003, Condition monitoring and diagnostics of machines: data processing,
  communication and presentation, Part 1 general guidelines (ANSI preview; SAI Global and AFNOR
  catalogue): https://webstore.ansi.org/preview-pages/ISO/preview_ISO+13374-1-2003.pdf ,
  https://infostore.saiglobal.com/en-us/standards/iso-13374-1-2003-586699_saig_iso_iso_1343762/ ,
  https://www.boutique.afnor.org/standard/nf-iso-13374-1/condition-monitoring-and-diagnostics-of-machines-data-processing-communication-and-presentation-part-1-general-guidelines/article/698492/fa107785
- ISO 13374-2:2007 (ANSI preview): https://webstore.ansi.org/preview-pages/ISO/preview_ISO+13374-2-2007.pdf
- ISO 13381-1:2015, Prognostics, general guidelines (SAI Global):
  https://infostore.saiglobal.com/en-us/standards/ISO-13381-1-2015-588598_SAIG_ISO_ISO_1348231/
- ISO/TR 17243-2:2017, Machine tool spindles: evaluation of spindle vibrations by measurements on
  non-rotating parts, direct- and belt-driven spindles (ISO): https://www.iso.org/standard/66432.html
- ISO/TR 17243-3:2020, gear-driven spindles 600–12,000 r/min (SCC; BSI; SNV):
  https://scc-ccn.ca/standardsdb/standards/8175472 ,
  https://knowledge.bsigroup.com/products/machine-tool-spindles-evaluation-of-machine-tool-spindle-vibrations-by-measurements-on-spindle-housing-gear-driven-spindles-with-rolling-bearings-operating-at-speeds-between-600-r-min-and-12-000-r-min ,
  https://connect.snv.ch/en/iso-tr-17243-3-2020
- ISO 20816-3 and ISO 10816-3 zone values (secondary summary, Fabrico):
  https://www.fabrico.io/blog/iso-20816-vibration-severity-zones/ ,
  https://www.fabrico.io/blog/iso-10816-3-vibration-severity/
- Also named, catalogue knowledge only (not opened for this file): ISO 17359 (general
  guidelines), ISO 13379 (diagnostics), ISO 230-3 (thermal), ISO 230-4 (circular tests), ISO 230-7
  (rotation axes), ISO 7870 (control charts), EN 13306 (maintenance terminology).

### OPC UA

- OPC 40501-1 Machine Tools, use cases: https://reference.opcfoundation.org/specs/OPC-40501-1/5
- OPC 40501-1 Machine Tools, 8.6 Notification (PrognosisListType, MaintenancePrognosisType):
  https://reference.opcfoundation.org/specs/OPC-40501-1/8.6
- OPC 40001-1 Machinery Basic Building Blocks: https://opcfoundation.org/developer-tools/documents/view/213

### Papers, data and research

- Damazo, Donmez, Soons et al., Investigating power quality solutions for computer numerical
  control machine tools (NIST, 2003; voltage sag tests on a CNC machine):
  https://www.academia.edu/108907821/Investigating_Power_Quality_Solutions_ForComputer_Numerical_Control_Machine_Tools
- ISO 230-3:2020, thermal effects, including the environmental temperature variation error test
  (ISO catalogue): https://committee.iso.org/standard/73291.html
- ISO 8573-1 compressed air purity classes (distributor summary):
  https://jhfoster.com/wp-content/uploads/2025/01/ISO8573-1_CompressedAirQualityStandard_2025.pdf

- Han, Miller, Vogl, Chen, Jia, ball screw degradation and axial natural frequency as an early
  indicator of rigidity loss, Manufacturing Letters 41 (2024) (NIST record):
  https://www.nist.gov/node/1876136
- MIMOSA for condition-based maintenance (VTT): https://cris.vtt.fi/en/publications/mimosa-for-condition-based-maintenance/
- PHM Society 2010 data challenge (milling tool wear):
  https://phmsociety.org/competition/phm/10 ; IEEE DataPort copy:
  https://ieee-dataport.org/documents/2010-phm-society-conference-data-challenge
- Spindle monitoring patents: https://patents.google.com/patent/US12011793 ,
  https://data.epo.org/gpi/EP2023224A1

### Trade press and vendor pages (marketing where they describe products)

- Haas tool load monitoring and autofeed, Setting 84 (secondary copy of Haas's guide):
  https://www.mfgrobots.com/Article/equipment/cnc/46178.html ; Haas tool life management:
  https://www.haas.co.uk/?p=1266
- Renishaw TRS2 non-contact broken tool detection (checks tools under a millimetre):
  https://www.renishaw.com/trs2 ; NC4 non-contact tool setter (CTE article):
  https://ctemag.com/?p=110714
- Cho, Han, Chi, Yoo, "A tool breakage detection system using load signals of spindle motors in
  CNC machines", ICUFN 2016, pp. 160–163, doi:10.1109/ICUFN.2016.7537007:
  https://pure.seoultech.ac.kr/en/publications/a-tool-breakage-detection-system-using-load-signals-of-spindle-mo/
- Toshiba Kikai, "Numerical controlling unit having tool-breakage detecting function",
  US 6859680 B2 (expired): https://patents.google.com/patent/US6859680B2

- The ins and outs of ballbar calibration (MoldMaking Technology):
  https://www.moldmakingtechnology.com/articles/the-ins-and-outs-of-ballbar-calibration
- Renishaw QC20-W ballbar resources: https://renishaw.com/resourcecentre/en/download/qc20-w--74856
- Metalworking fluid hacks (MSC Industrial):
  https://www.mscdirect.com/knowledge-center/articles/hacks-to-optimize-metalworking-fluids
- Condition monitoring of metalworking fluids (STLE TLT webinar summary, January 2019):
  https://www.stle.org/files/TLTArchives/2019/01_January/Webinars.aspx
- ATAGO inline refractometer (product page via reseller):
  https://mreport.co.th/en/products/metrology/383-ATAGO-Refractometer-Brix-Meter-Inline
- CMMS and condition-monitoring integration (vendor blogs, marketing):
  https://www.getmaintainx.com/fr/blog/cmms-integration-guide ,
  https://oxmaint.com/blog/post/predictive-maintenance-sensor-data-to-cmms-work-order-integration ,
  https://endpoint.augury.com/en/articles/11998212-cmms-eam-integration-guide
- Vibration sensors for CNC predictive maintenance (PatSnap blog):
  https://www.patsnap.com/resources/blog/rd-blog/vibration-sensors-for-cnc-predictive-maintenance-patsnap-eureka/
