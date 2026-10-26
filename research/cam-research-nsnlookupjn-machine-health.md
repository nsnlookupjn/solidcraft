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
| In-process inspection and the edge box | [cam-research-nsnlookupjn-in-process-inspection.md](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-y2w06n/research/cam-research-nsnlookupjn-in-process-inspection.md) | **the Raspberry Pi box, its sensors and interfaces**, alignment of data to G-code blocks and CAM segments, the per-part record, self-tuning |
| Machine library | [cam-research-nsnlookupjn-machine-library.md](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-machine-library/research/cam-research-nsnlookupjn-machine-library.md) | spindle torque/power curves, warm-up routine and cold derating, signals each control publishes, accuracy and wear held by the shop instance |
| Workholding | [cam-research-nsnlookupjn-workholding.md](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-workholding/research/cam-research-nsnlookupjn-workholding.md) | workholding-side trends (worn vise screw, jaws), slip and lift detection |
| Cutting physics | [cam-research-nsnlookupjn-cutting-physics.md](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-cutting-physics/research/cam-research-nsnlookupjn-cutting-physics.md) | predicted force, torque and power per segment; Taylor tool life |
| Tool library | [cam-research-nsnlookupjn-tool-library.md](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-tool-library/research/cam-research-nsnlookupjn-tool-library.md) | tool and holder records where wear history lives |
| Stock integrity | [cam-research-nsnlookupjn-stock-integrity.md](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-stock-integrity/research/cam-research-nsnlookupjn-stock-integrity.md) | wall and part modes, which must not be mistaken for machine faults |

**Ownership (so nothing is said twice):** the edge box hardware, how it connects to each
control, and how samples are lined up with the program stay in the in-process inspection file
(the sensor kit itself is in its section [A commercial retrofit kit: upgrade any machine](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-y2w06n/research/cam-research-nsnlookupjn-in-process-inspection.md#a-commercial-retrofit-kit-upgrade-any-machine)).
The machine model and its published-signal list stay in the machine library. This file owns
**what the recorded history means for health**: failure modes and their signatures, baselines,
thresholds and trends, predicting bad work, coolant condition, and turning a finding into a
maintenance work order.

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
   finish, thermal size and corrosion. Measured inline by sensors on the edge box, they belong
   in the same health record and the same trend tests.
7. **The output is a maintenance order, not a dashboard.** Each finding names the asset, the
   likely cause, the evidence (with links to the G-code line and the trend), the check to make,
   how urgent it is and an estimate of time left. It is sent to whatever the shop uses: a
   computerised maintenance management system (CMMS) by its API, email, a printed ticket from the
   edge box, or the machine's own OPC UA `MaintenancePrognosisType` where a control supports it.
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
| Data acquisition | the edge box (in-process inspection file): control signals, added accelerometer and temperature sensors, coolant sensors |
| Data manipulation | the edge box: per-segment features (RMS, peak, band energies, dominant frequency, mean load, temperature rise) |
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
  chatter, which need the added accelerometer sampled at kHz by the edge box.
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
| **Tool wear** (not a maintenance item, but the commonest drift) | flank and crater wear, chipping | `r_P` rising along a tool's cut time; vibration rising | size and finish drift; breakage | change tool earlier; correct tool-life record in the tool library |
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
| **Stop** | sudden step change (crash-like), zone D on a spindle, temperature runaway | alarm at once through the edge box's alarm output, which the program checks at a safe point (inspection file); machine flagged not fit for critical work |

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

### Coolant condition (Jason's addition)

Jason's idea (other thread): the edge box kit could include machine sensors (vibration,
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
  (vendor guides in Sources; marketing). SolidCraft (or the edge box service) should send a
  plain, documented record and keep the CMMS's ID; it should not become a CMMS.
- **OPC UA prognosis.** Where the machine has an OPC UA server per OPC 40501-1, the machine's own
  `MaintenancePrognosisType` entries (predicted time, activity) can be read and shown next to
  SolidCraft's findings, so builder-scheduled maintenance and condition-based findings sit on one
  list. Writing SolidCraft's own prognoses into a control's server is not something the standard
  invites (opinion, unverified); keep them in SolidCraft's record and the CMMS.
- **No CMMS.** Email, a message to a chat channel, or a printed maintenance ticket from the edge
  box's label printer (the same printer that prints part travellers), with a QR code back to the
  evidence.
- **The machine's health page in SolidCraft**: open findings, trend charts, the last
  fingerprint, maintenance done, and the next planned checks; the shop instance in the machine
  library holds them.

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

- **Local first.** Health data stays on the edge box and the shop's own storage; nothing goes to a
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
| H1 | fingerprint folded into the warm-up; idle power, axis load and vibration per step; baselines; relative thresholds | edge box with accelerometer; warm-up gate | catches spindle and axis degradation with no CAM prediction at all |
| H2 | findings and work orders (CMMS API, email, printed ticket); closing the loop after maintenance | H1 | turns data into action |
| H3 | per-segment residuals and EWMA trends on production cuts | physics predictions per segment; G-code alignment | machine-wide drift and tool wear on jobbing work |
| H4 | bad-work prediction: control charts on characteristics and corrections, cause ranking | in-process inspection records | catches drift before a part is out of tolerance |
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
  ordinary numeric code that fits the engine layers; the edge box service is separate.

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
5. Is a printed maintenance ticket from the edge box useful, or does everything go to phones?
6. Which coolant sensors are robust enough inline (refractometer fouling, pH probe drift) for a
   small-shop kit? Needs hands-on testing.
7. How should SolidCraft treat a finding on a machine owned by someone else (a subcontractor's
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
