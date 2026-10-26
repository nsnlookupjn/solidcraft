# CAM research: part certification by in-process inspection

Research for [#1621](https://github.com/tangency-labs/solidcraft/issues/1621) (Research Request
for Plugin: CAM). This file covers one part of that request: **measuring parts while they are
still on the machine (probing) or beside it (hand gauges), correcting the process in cycle when a
feature drifts, carrying tolerances from the design into that loop, and producing a per-part
quality record that travels with the part through every setup**. It ends with an idea for a small
shop-floor device (a Raspberry Pi) that collects the measurements and prints the traveller.

It touches design-side work as much as CAM: the tolerances have to exist in the model before
anything downstream can check them. Those parts are marked **(design side)** so maintainers can
route them elsewhere if they prefer.

It pairs with earlier files from the same author:

- [Shared cloud tool library](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-tool-library/research/cam-research-nsnlookupjn-tool-library.md):
  tool records, where tool wear offsets and probe records would live.
- [Cutting tool physics](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-cutting-physics/research/cam-research-nsnlookupjn-cutting-physics.md):
  predicted forces and deflection, which explain much of the error that probing finds.
- [Workholding physics](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-workholding/research/cam-research-nsnlookupjn-workholding.md):
  clamp distortion and part springback, which decide whether an in-clamp measurement can be
  trusted once the part is released.
- [Machine library](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-machine-library/research/cam-research-nsnlookupjn-machine-library.md):
  spindle torque/power curves, axis thrust, stiffness and modes per machine; the shop instance
  that measured loads and vibration (below) would calibrate.
- [CAM physics system](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-vgqo3g/research/cam-research-nsnlookupjn-physics-system.md):
  how the libraries fit together, and its "closed loop: measuring what was predicted" section,
  which this file's edge device feeds.

Licence: MIT OR Apache-2.0, the same as SolidCraft (see the Licence section of CONTRIBUTING.md).

## About this research

- **Author:** Jason ([@nsnlookupjn](https://github.com/nsnlookupjn)). Idea and direction by Jason;
  source research and writing done with an AI assistant (Claude) in October 2026.
- **How it was produced:** web searches of public sources (machine-builder and probe-maker public
  pages, standards bodies' catalogue entries and free previews, NIST and CAx-IF public documents,
  data-collection vendors' public guides), each claim checked against the cited page where
  possible. No commercial CAD/CAM program was driven or scripted, no GPL/LGPL/AGPL source was read,
  and no text is copied beyond short quotes. Standards are described from their public catalogue
  entries and secondary summaries, not their paid text, so exact clause content should be checked
  by whoever implements it. Worked examples are computed for this file from tabulated ISO 286
  values and textbook formulas.
- **SolidCraft facts** were checked in the repository at commit `10b5cc5` (October 2026).
- **Units:** millimetres and micrometres (µm) for size, radians for angles unless a drawing
  convention is quoted in degrees.
- **Marking:** "(unverified)" marks a claim not confirmed against a source. **Fact**,
  **opinion** and **marketing** are labelled where they mix.

> **[TODO Jason: personal experience]** How you inspect today: which features you probe, which
> you gauge by hand, how you decide when to adjust a wear offset, what paperwork travels with a
> part (and how often it gets lost or wet), whether customers accept on-machine measurements, and
> any part that passed on the machine and failed at final inspection. Label this as personal
> experience.

## Summary

1. **The loop is old; the connections are missing.** Machinists already probe bores and faces
   with a spindle probe, compare the result with a tolerance written into a macro, nudge a wear
   offset, and write numbers on a paper traveller. Every one of those steps is done by hand or by
   a hand-written macro. The tolerance is typed in again at each step (drawing → CAM → probe
   macro → inspection sheet), and each retyping is a chance for a mistake.
2. **Tolerances must live in the model (design side).** SolidCraft dimensions today are exact
   values; Thread does not yet have tolerance classes (docs/roadmap-detail.md, "This does not add
   tolerance classes"); STEP export is AP242 geometry, names and colours without PMI
   (crates/kernel/src/step_out.rs). The first step is to let a dimension, a hole, a thread or a
   face carry a **tolerance** (±, limits, an ISO 286 fit such as H7, an ISO 965 thread class such
   as 6H, a geometric tolerance with datums, a surface texture requirement), with a general
   tolerance (ISO 2768) for everything else. Everything later in this file reads from that.
3. **CAM can then generate the inspection, not just the toolpath.** Once a feature knows its
   limits, CAM can (a) cut to the middle of the tolerance instead of the nominal, (b) insert a
   probing cycle after the semi-finish or finish pass with the limits filled in, and (c) write
   the decision logic: inside the band → carry on; drifting → correct the wear offset in cycle;
   outside → stop, re-cut if material remains, or flag the part as rejected.
4. **Compensation belongs in the machine, records belong outside it.** The correction (update a
   wear offset, re-run the finish pass) must happen in the control's own macro, at machine speed,
   with no network in the loop. An external device only **records** what the machine measured and
   decided. This keeps the safety-critical part where machine builders already support it.
5. **One record per part, many setups.** Each part gets a serial number and a record. Each setup
   (Op 10 on machine A, Op 20 on machine B, a hand-gauge check at the bench) appends its
   measurements to the same record. A printed traveller carries a 2D code (DataMatrix) that
   points at the record, so the next station can load it by scanning.
6. **The report is the product.** At the end, the record becomes a per-part quality report:
   every characteristic, its limits, the measured value, how it was measured, pass/fail, and the
   decision. Formats exist: QIF Results (ISO 23952) for machines, AS9102 Form 3 for aerospace
   first articles, PPAP dimensional results for automotive.
7. **Honest limit: on-machine measurement is not independent.** A machine that cut a bore
   oversize because of a thermal or geometric error can measure it with the same error. On-machine
   probing is excellent for **process control** and good evidence for **acceptance** only after
   it has been correlated against an independent instrument (a CMM or calibrated gauge). The
   report must say how each value was measured, and the decision rule must account for
   measurement uncertainty (ISO 14253-1).
8. **The Raspberry Pi edge device is feasible** with off-the-shelf parts, for reading results the
   machine already produces: by serial output (DPRNT, on many Fanuc-style and Haas controls), by
   network (Fanuc FOCAS, Haas machine data collection, MTConnect, OPC UA where the control offers
   it), or by operator entry for hand gauges. It cannot read a probe signal directly; the probe
   talks to the control. Selling it is a business decision for maintainers and a hardware vendor;
   an open reference design plus "works with SolidCraft" vendors is the lowest-risk path. See
   [The edge device](#the-edge-device-a-raspberry-pi-at-the-machine).
9. **The same box can watch the cut, not just the part (Jason's addition).** Many controls
   publish spindle load, axis (servo) loads and sometimes torque; an add-on accelerometer gives
   vibration. Logged against the toolpath, these are the measurements the physics models predict,
   so they can **calibrate** the machine, tool and material libraries for that shop. They are
   for monitoring and learning; corrections still happen in the control. Every sample is aligned
   with the G-code block, the CAM segment and the timeline, so an event can be found exactly on
   the program, the toolpath and the part. See
   [Machine monitoring](#machine-monitoring-loads-torque-and-vibration).
10. **Self-tuning CAM operations (Jason's name for it).** Put the two loops together: the
    machine data says how much margin the cut had (load, torque, vibration), and the inspection
    results say whether accuracy and surface finish held. CAM then raises the feeds and depths of
    the next run where both have margin, and backs off where either does not. Inspection is the
    guard that lets the shop **go harder without losing accuracy or finish**. See
    [Self-tuning CAM operations](#self-tuning-cam-operations).
11. **Close the loop back to the design (Jason's vision).** Existing CAD/CAM inspection tools
    (Fusion's is the closest public example) can probe and record, but results mostly stop at a
    report. The goal here is a complete, well-designed inspection system whose results and machine
    data flow **back into the design during manufacturing**. It would suggest tolerance, feature
    and process changes that improve performance, time, cost and quality, and route each change
    through a controlled revision and release. See
    [From inspection back to the design](#from-inspection-back-to-the-design).

## Fundamentals

### Words that get mixed up

- **Dimension** (nominal): the ideal value in the model, e.g. a bore Ø20.
- **Tolerance:** the permitted variation, written as limits (Ø20.000–20.021), ± values, a fit
  code (Ø20 H7) or a geometric tolerance (position Ø0.05 to datums A, B, C).
- **Characteristic:** anything on the drawing or model that has a requirement and can be checked:
  a size, a position, a flatness, a surface roughness, a thread, a note ("break all edges").
  Inspection reports are organised by characteristic, each with a number (the "balloon").
- **Measurement:** the value found, with the instrument and its uncertainty.
- **Inspection:** measurement plus a decision against the tolerance.
- **Compensation:** changing the process (usually a tool wear offset or a work offset) so the next
  cut lands closer to the target.
- **Certification / acceptance:** a signed statement that the part meets the requirements. It
  rests on the inspection record, the material and process certificates, and who approved it.

### Tolerances in the model (design side)

**Size tolerances, ISO 286 fits.** ISO 286-1/-2 defines standard tolerance grades (IT01 to IT18)
and fundamental deviations (letters: capital for holes, lower case for shafts). A fit is a pair,
e.g. **H7/g6**. Worked example, Ø20 (the 18–30 mm range in the ISO 286 tables):

| Feature | Code | IT value | Deviations | Limits |
|---|---|---|---|---|
| Hole | H7 | 21 µm | 0 / +21 µm | 20.000 – 20.021 |
| Shaft | g6 | 13 µm | −7 / −20 µm | 19.980 – 19.993 |
| Fit | H7/g6 | | | clearance 0.007 – 0.041 |

A modeller that knows the code can show the limits, the fit type (clearance, transition,
interference) and the clearance range without the user opening a table. **Fact** (ISO 286
tables; values widely reproduced).

**General tolerances, ISO 2768.** For dimensions without an individual tolerance, a title-block
note such as "ISO 2768-m" sets defaults. ISO 2768-1 classes f (fine), m (medium), c (coarse),
v (very coarse); for example linear 6–30 mm is ±0.1 (f) or ±0.2 (m), and 30–120 mm is ±0.15 (f)
or ±0.3 (m). ISO 2768-2 covers general geometric tolerances (H, K, L). **Fact**, but note ISO is
moving general specifications to ISO 22081, and ISO 2768-2 is withdrawn in some catalogues
(unverified; check before implementing).

**Evidence that general tolerances should be a per-process default (from Jason).** Jason points
out that Xometry, an on-demand manufacturing marketplace, standardises on ISO 2768 for general
tolerances, metals at ISO 2768 medium or fine and plastics at medium. Checked against Xometry's
public pages:

- Xometry's US manufacturing standards page: features of size and location "+/- .005" (metals)
  or +/- .010 (plastics and composites) following ISO 2768 unless otherwise specified".
- A Xometry Pro article on standard tolerances: "All other dimensions without specific
  tolerances will follow the general ISO 2768-medium tolerances"; it recommends ISO 2768 medium
  (with ISO 286 IT8 or coarser) for plastic parts and offers fine as an option.
- Not found on the public pages: a rule that metals default to *fine*. Medium is the stated
  default, fine is available on request (so "medium/fine" is best read as "medium, fine when
  asked").

Observation (computed for this file): the inch default and the ISO class are not the same
thing. ±0.005" is ±0.127 mm. ISO 2768-m allows ±0.2 mm for 6–30 mm and ±0.3 mm for 30–120 mm,
and ISO 2768-f allows ±0.1 mm and ±0.15 mm. So for most machined sizes the ±0.005" figure is
tighter than medium and close to fine. A modeller should store the rule exactly as the supplier
states it, not translate it into the nearest ISO class.

Jason also summarised Xometry's defaults by process. The table below puts his summary next to
what Xometry's public US standards page says (accessed 2026-10-26, inch values as published):

| Process | Jason's summary | Xometry public page | Standard that governs it in general |
|---|---|---|---|
| CNC machining | ±0.005" (±0.127 mm), ISO 2768 m/f | ±.005" metals, ±.010" plastics and composites, "following ISO 2768"; 125 µin Ra or better; form and angularity by part length | ISO 2768-1/-2 |
| Sheet metal | ±0.010" flat, ±0.030" bends, ISO 2768-m | ±.010" feature to feature on one surface; ±.015" edge or hole to bend; ±.030" across bends and edge to edge on formed parts; bend angle ±1° | ISO 2768-m is common for sheet metal (Xometry's ISO 2768 page) |
| Injection moulding | ±0.005" + ±0.002"/in, DIN 16742 | mould machining ±.005" plus ±.002"/in shrink allowance; repeatability under ±.004" | DIN 16742 (tolerances for plastic mouldings); the page itself does not cite it |
| Die casting | ±0.005" + ±0.002"/in, ISO 8062 | mould machining ±.010" plus ±.001"/in shrink; repeatability under ±.004" | ISO 8062-3 (casting tolerance grades); the page itself does not cite it |
| SLS / MJF | ±0.3 % with a ±0.012" floor | SLS nylon ±.015" or ±.002"/in, whichever is greater; MJF prototype bands by size (e.g. under 30 mm ±0.70 mm), tighter for engineer-reviewed production | process-specific; no general ISO class |
| DMLS / FDM | ±0.1 % to ±0.5 % | DMLS ±.005" first inch + ±.002" each additional inch; FDM ± one layer first inch + ±.002"/in; SLA ±.005" XY / ±.010" Z first inch + ±.002"/in | process-specific |

Differences between the two columns are expected: Jason's table may come from another Xometry
region or page, and the published numbers change. Either way the point stands: **the default
general tolerance depends on the process and material**, and it comes in a few rule shapes.

**Proposal: general-tolerance profiles (opinion).**

- A design (or each body) carries a **process**: CNC metal, CNC plastic, sheet metal, injection
  moulding, die casting, SLS, MJF, DMLS, FDM, SLA and so on.
- Each process has a **general-tolerance profile** that gives every untoleranced dimension its
  limits. Rule shapes seen above: an ISO 2768 class; a fixed ± value; "± first inch plus ± per
  additional inch"; "± percentage, with a minimum"; "whichever is greater" of two rules; separate
  XY and Z values for layer processes; separate form and angularity by part length.
- SolidCraft ships profiles for the **standards** (ISO 2768 f/m/c/v, ISO 8062 grades, DIN 16742
  groups). Users add **supplier profiles** as data they enter or import (for example a
  supplier's published standards), labelled with source and date. The project does not need to
  bless any supplier.
- The profile feeds the rest of this file. Drawings and PMI show the general-tolerance note. The
  inspection plan checks untoleranced features against the profile. A warning appears when an
  explicit tolerance is tighter than the process can hold (for example ±0.05 mm on an SLS part).
- For additive processes the profile ties into the
  [additive process physics](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-j72y6h/research/3d-printing-research-nsnlookupjn-process-physics.md)
  research. That file covers why each process holds the accuracy it does (shrinkage, layer
  height, melt pool), and could predict a tolerance band per feature rather than one number per
  process.

**Geometric tolerances (GD&T / GPS).** Two families describe the same ideas with different rules:

- **ISO GPS**: ISO 1101 (geometric tolerancing: form, orientation, location, run-out), ISO 5459
  (datums), ISO 8015 (fundamental rules, including the principle that each requirement is
  independent unless stated), ISO 14405-1 (size, including the envelope requirement Ⓔ),
  ISO 2692 (maximum material requirement Ⓜ).
- **ASME Y14.5-2018** (US): the same symbols mostly, but Rule #1 (size controls form, the
  envelope principle) is the default, where ISO's default is independence.

A modeller has to know which system a design uses, because the same symbol can mean a different
zone. **Fact** (summarised from public overviews; the standards are paid documents).

What a model needs to hold, per tolerance: the toleranced feature (a face, a pattern of holes,
an axis), the type (14 geometric characteristics plus size), the zone (value, shape: Ø, spherical,
two planes), modifiers (Ⓜ, Ⓛ, Ⓔ, projected zone, free state, "common zone", "all around"), and
the datum reference frame (A | B | C, with their own modifiers). **Fact.**

**Threads.** ISO 965-1 metric threads use tolerance classes such as **6H** (internal) and
**6g** (external): a grade number for the pitch-diameter tolerance and a letter for the
position. Threads are normally accepted with **go/no-go gauges** (ISO 1502): the go gauge must
screw in fully, the no-go must not enter more than about two turns (unverified: the exact
no-go acceptance wording is in ISO 1502). UNC/UNF threads use classes 2B/3B and 2A/3A
(ASME B1.1). SolidCraft's thread catalogue currently stores designation and pitch but not class
(docs/thread-catalogue.md: "thread classes, handedness and sealing behavior are outside this
tranche").

**Threads follow a named standard (from Jason).** Jason notes that Xometry requires all internal
and external threads to follow ISO metric or ANSI/ASME unified standards. Xometry's public
standards page says threads "will be fully formed and cut to specified size and class indicated
in provided drawings", and its standard threads chart lists UNC/UNF sizes. The exact "must adhere
strictly" wording was not found on the public pages checked (unverified). For 3D-printed parts
the same page warns that modelled threads "may have limited functionality as printed" and
recommends tapping or threaded inserts. What this means for the model (opinion):

- Every thread carries its **system** (ISO metric: ISO 261/965; unified: ASME B1.1) and its
  **class** explicitly, not just a designation.
- When the user gives no class, use the usual general-purpose defaults and show them: 6H/6g for
  metric and 2B/2A for unified (common practice; check against the standards before implementing).
- The class drives the inspection plan (which go/no-go gauges), the CAM thread-milling target
  (pitch diameter in the middle of the class band), and the report.
- On additive processes, a modelled thread prompts "tap after printing or use an insert" and
  becomes a post-process step on the traveller.

**Surface texture.** A face can carry a roughness requirement, e.g. Ra 1.6 µm, with lay
direction and machining allowance. Standards changed recently: **ISO 21920-1/-2/-3 (2021)
replaced ISO 1302, ISO 4287, ISO 4288 and ISO 13565-2/-3**, withdrawn at the end of 2021; old
drawings remain valid under the old standards, and ISO 21920 adds a new drawing symbol. One
practical change: ISO 21920-2 computes most parameters once over the evaluation length, where
ISO 4287 averaged over (by default) five sampling lengths; Rz, Rp and Rv are still taken per
section (Digital Surf). **Fact.** In the US, ASME B46.1 covers the same ground.

Machining can **predict** roughness, which matters for CAM:

- Turning, feed *f* per rev, nose radius *r*: theoretical peak-to-valley **Rz ≈ f²/(8r)**,
  **Ra ≈ f²/(32r)**. For f = 0.15 mm/rev, r = 0.8 mm: Rz ≈ 3.5 µm, Ra ≈ 0.9 µm. Real values are
  higher (built-up edge, vibration, wear). **Fact** (textbook geometry).
- Ball-end or bull-nose finishing with step-over *s*: scallop height **h ≈ s²/(8R)**.
  R = 3 mm, s = 0.3 mm → h ≈ 3.75 µm. **Fact** (geometry).

So a face tagged "Ra 1.6" can tell CAM the maximum feed or step-over it may use, and the cutting
physics file can check that the force at that feed is acceptable.

**Model-based definition (MBD) and PMI.** Product and Manufacturing Information (PMI) is the set
of tolerances, datums, finishes and notes attached to a 3D model. Two kinds:

- **Presentation PMI:** the symbols as graphics, for people to read.
- **Semantic PMI:** the meaning as data (this face, this zone, these datums), for software to use.

Only semantic PMI can drive CAM and inspection automatically. The exchange formats:

- **STEP AP242** (ISO 10303-242) carries semantic and presentation PMI; the CAx-IF (now MBx-IF)
  publishes free *Recommended Practices for the Representation and Presentation of PMI* (v4.x,
  2024) describing exactly how to write it. **Fact.**
- **QIF** (ISO 23952:2020, Quality Information Framework) is an XML format in eight parts:
  Overview, Library, **MBD**, **Plans**, **Resources**, **Rules**, **Results**, **Statistics**
  (qifstandards.org). It covers the whole quality chain: the model with PMI, the measurement plan,
  the results per part and the statistics over many parts. **Fact.**
- **DMIS** (ISO 22093:2011): a programming language for measuring machines (CMMs), older than
  QIF and still common on CMMs. **Fact.**
- **NIST's STEP File Analyzer and Viewer** is free US-government software that reports and checks
  semantic PMI in AP242 files and has a large PMI test-case set; useful as an independent check
  for an exporter. **Fact** (NIST page; licence should be checked before any reuse beyond running
  it as a separate program).

### In-process probing

**Hardware.** A spindle (touch-trigger) probe sits in the tool changer like a tool. When the
stylus touches the part it sends a trigger (by optical, radio or cable) to an interface, which
switches the control's **skip** input. The control latches the axis positions at that instant.
A **tool setter** on the table does the reverse: tools touch it to measure length, diameter and
breakage. Lathes use arm-mounted tool setters and turret probes. **Fact.**

**Software: the measuring cycles.** Probe makers and machine builders supply macro packages that
run on the control. Renishaw's public page lists the basic set for machining centres: single
surface X/Y/Z, web/pocket, bore/boss, external corner, single point and two-point diameter, plus
rib/groove and Z datum on lathes; "Tool and work offsets can be updated automatically based on
measured dimensions", and results can be printed after each cycle. A more comprehensive package
(Inspection Plus) adds more cycles. Haas sells WIPS (wireless spindle probe, tool setter and
guided templates) with in-process inspection, work-offset and tool-length setting, and broken-tool
detection. **Fact** (Renishaw, Haas UK pages).

**How a cycle works (generic).** At G-code level the pattern is the same on most controls:

1. Move to a safe point near the feature.
2. Protected positioning move (stop if the probe triggers unexpectedly: collision protection).
3. Skip move (G31 on Fanuc-style controls; G38.x on LinuxCNC-style controls) toward the surface;
   the control latches the trigger position into system variables.
4. Back off, repeat slowly for accuracy, repeat on other points.
5. Compute the result (centre and diameter from three or four points for a bore; a plane from
   three points), correct for the stylus ball radius and the calibrated trigger offset.
6. Compare with the tolerance; set flags/alarms; optionally update an offset.
7. Optionally output the result (print, file, variable range).

**Fact** for the pattern; exact variable numbers differ per control and probe package (check the
control's manual).

**Compensation in cycle.** The useful corrections:

| Feature measured | Typical correction | Offset changed |
|---|---|---|
| Bore/boss diameter (interpolated with an end mill) | cutter radius/diameter wear | tool diameter (radius) wear |
| Bore diameter (boring bar) | boring head adjustment (manual or automatic heads) | tool diameter wear on lathes; head adjustment on mills (unverified for automatic heads) |
| Face height / hole depth | Z length | tool length wear |
| Feature position | work offset or a local shift | work offset (setup), not per tool |
| Turned diameter | X wear | lathe X wear |
| Thread pitch diameter (thread milling) | cutter radius wear | tool diameter wear |

A good correction has the same parts in every package (opinion, drawn from how the probe-maker
packages are described publicly):

- **Target**: usually the middle of the tolerance, not the nominal.
- **Dead band**: do not correct when the error is smaller than the measurement noise.
- **Feedback fraction**: apply only part of the error (e.g. 50 %) so the loop does not chase
  noise; or average over the last N parts.
- **Correction limit**: refuse a correction larger than a set value; a big error means a
  broken tool, a wrong tool or a loose part, not wear. Stop and alarm instead.
- **Experience value**: a fixed bias for a known systematic difference between the probed value
  and the true value (e.g. tool deflection that the probe does not see).
- **Re-cut**: if the feature is undersize (for a bore) and material remains, re-run the finish
  pass with the corrected offset and measure again; cap the number of re-cuts.

**Worked example: bore Ø20 H7, end mill helical interpolation.**

- Limits 20.000 – 20.021 → target 20.0105.
- Probe uncertainty on this machine, from a correlation study: say U = 3 µm (assumed for the
  example).
- Acceptance zone per ISO 14253-1 (conformance proven only inside the tolerance reduced by U at
  each end): 20.003 – 20.018.
- Part 1 measures **20.004**. Inside the acceptance zone, so it passes, but 6.5 µm below target.
- Correction: the bore is small, so the tool must cut larger: reduce the compensated tool
  diameter by 6.5 µm (or radius by 3.25 µm). With 50 % feedback, apply −0.0033 to the diameter
  wear (−0.0016 to radius wear).
- Part 2 measures 20.008; next correction −0.0013; and so on toward the target.
- Had part 1 measured 20.001 (in tolerance, but outside the acceptance zone), the cycle would
  apply the correction, **re-cut**, re-measure, and record both values: the first as an
  in-process value, the second as the acceptance value.
- Had it measured 19.940 (60 µm small), the correction limit (say 20 µm) is exceeded: stop,
  alarm "check tool T12", record "held".

**Fact** for the arithmetic; the 3 µm, 50 % and 20 µm values are example settings.

### Manual gauging

Not everything can be probed: threads, small or deep holes, surface finish, features hidden by
fixtures, cosmetic notes. The usual instruments: bore gauges and air gauges for diameter,
depth micrometers and height gauges, plug and ring gauges for go/no-go, thread plug and ring
gauges, surface roughness testers (stylus profilometers), pin gauges for small holes. **Fact.**

Ways to get those values into the loop:

- **Operator prompt at the machine**: the program stops (M00/M01) and asks for a value or a
  pass/fail. Haas controls have an interactive user input M-code (M109) that writes the
  operator's answer into a macro variable (unverified detail; Haas code reference page for M109).
- **Digital gauges**: many calipers, micrometers and indicators have a data output (USB,
  Bluetooth, or a wired "SPC" port); a gauge interface turns a button press into a value on a
  computer (unverified; varies by maker and is often proprietary).
- **Tablet or edge device beside the machine**: the operator scans the traveller, the screen shows
  the characteristic to check with its limits, the operator enters or presses the gauge button.

A thread go/no-go result can drive compensation too: when the go gauge fails on a milled
internal thread, the cutter radius wear is reduced a small step and the thread pass re-run
(common shop practice; opinion that it should be automated with a cap on the number of steps).

### Accuracy of on-machine measurement

Sources of error (fact, general metrology):

- **Probe**: pre-travel and its variation with direction ("lobing"), stylus ball size,
  repeatability (touch-trigger probes are usually specified around 1 µm 2σ repeatability for
  premium models; unverified as a general figure).
- **Machine**: positioning error, squareness, backlash, scale errors. These are the same errors
  that made the part, so on-machine measurement can agree with a wrong part.
- **Temperature**: part, machine and fixture at different temperatures; a 100 mm aluminium part
  grows about 2.3 µm per °C (α ≈ 23 µm/m·°C).
- **Part condition**: coolant, chips and burrs under the stylus.
- **Clamping**: a part measured in the clamp can change shape when released (see the workholding
  file).

What makes the numbers trustworthy:

- **ISO 230-10** specifies tests of the measuring performance of probing systems on NC machine
  tools (current edition 2022). A machine-specific test gives the uncertainty to use in the
  decision rule. **Fact** (catalogue entry).
- **Correlation**: measure the same parts on the machine and on a CMM; the difference (bias) and
  scatter tell you whether on-machine results can stand in for final inspection on that feature.
- **ISO 14253-1:2017** gives the decision rules: conformance is proven only when the measured
  value lies inside the tolerance reduced by the expanded uncertainty (guard band); a supplier
  and a customer can agree a different rule. **Fact** (catalogue entry); the report should state
  which rule was used.
- **Calibration**: the probe is calibrated on a ring gauge or sphere before use; the report
  should record when.

**Opinion:** the system should label each value with its *method* (on-machine probe, hand gauge,
CMM) and never present an on-machine value as final acceptance unless the user has recorded a
correlation for that feature type.

### The per-part record and report

What a per-part report needs to say (fact, from AS9102 and general practice):

- **Part identity**: part number, revision, serial or lot, material and heat/lot, purchase order.
- **Characteristics**: number (balloon), requirement (nominal and limits, or the GD&T frame, or
  the note), measured value(s), units, instrument and its ID, method, operator or machine,
  date and time, pass/fail.
- **Decision**: accepted, rejected, held, reworked, deviation approved; who decided.
- **Process evidence**: which program, which setup, which machine, which offsets were changed and
  by how much.

The formats:

- **AS9102 (Rev C, 2023) First Article Inspection** for aerospace: Form 1 part number
  accountability (part, drawing revision, reason for FAI), Form 2 product accountability
  (materials, special processes, functional tests with certificate numbers), Form 3
  characteristic accountability (every ballooned characteristic, with **actual values**, not
  "OK"). A partial ("delta") FAI covers only characteristics affected by a change. **Fact**
  (summary from a public guide; AS9102 itself is a paid SAE document).
- **PPAP** (AIAG Production Part Approval Process) for automotive: includes dimensional results
  on sample parts, a control plan, measurement system analysis and process capability studies.
  **Fact** (general knowledge of the AIAG manual; details unverified).
- **QIF Results** (ISO 23952 part 7) as the machine-readable form, and **QIF Statistics** across
  parts for SPC (Cp, Cpk, trends). **Fact.**

**Opinion:** generate QIF as the master record and render AS9102-style or customer-style PDFs
from it; do not invent another format.

**Selectable report levels: a marketplace model (from Jason).** Jason points out that Xometry
sells inspection as tiers: an optional Standard Inspection Report covering up to 20 critical
dimensions, then formal CMM dimensional reports and AS9102 First Article Inspection (FAIR).
Xometry's public inspection-options article for CNC and sheet metal confirms this and lists more
tiers:

| Xometry tier (public article) | What it covers | Documentation |
|---|---|---|
| Standard Inspection | material, features and finish against the quote, CAD or print; with a print, up to 20 critical dimensions | "Documentation will not be shipped with your parts" |
| Formal Inspection with Dimensional Report | all dimensions, attributes, locations and print notes, calibrated tools, sampled to ANSI/ASQ Z1.4 Level II with zero rejects | dimensional report with a ballooned print |
| CMM Inspection with Dimensional Report | same scope, measured on a CMM | dimensional report |
| First Article Inspection (AS9102 or equivalent) | sampled to Z1.4 Level II, 2.5 % AQL, full lot built first, capped at 100 inspection points | AS9102 forms |
| Source Inspection | audit at the supplier before shipment | per audit |
| Build and Hold First Article | one part or a small batch approved before the rest is made | per agreement |
| Custom | non-destructive testing, serialisation, custom sampling, customer datasheets | per agreement |

Not found on the public pages checked: the Standard tier being called a "Standard Inspection
*Report*". The article says that tier ships no documentation, so it is an inspection without a
report.

**What SolidCraft can take from it (opinion):** the per-part record is always kept. The *report
level* is a setting on the job that decides what gets measured and what gets printed:

1. **Critical only:** the characteristics flagged critical (or the top N), pass/fail plus
   values, on the traveller label. Fits on-machine probing and quick hand checks.
2. **Full dimensional:** every characteristic, including the ones covered by the general-tolerance
   profile, with method and instrument per value and a ballooned view generated from the model's
   PMI (no manual ballooning).
3. **CMM-backed:** the same, with the acceptance values coming from a CMM (imported as QIF
   Results or DMIS output). On-machine values stay as in-process evidence.
4. **First article (AS9102):** Forms 1–3 generated from the record, with Form 2 entries for
   material and process certificates attached to the record.
5. **Sampling:** for lots, a sampling plan (for example ANSI/ASQ Z1.4 or ISO 2859-1) picks which
   serials get the full level. The rest get the critical level, and the record says which plan
   was used.

Because the characteristics, limits and balloons all come from the model, moving up a level
costs measuring time but no extra paperwork. Today each step up a tier also means another round
of manual ballooning and form filling.

### The part traveller across setups

A traveller (router, job card) lists the operations in order and collects sign-offs and results
as the part moves. Problems with paper travellers (opinion, common shop complaints): they get lost,
wet, oily or illegible; values are copied by hand; the office learns of a problem late; the
traveller and the part can be separated.

A digital-plus-paper traveller (proposal):

- **One record per part (or lot)**, keyed by serial number, created at the first operation or
  at material issue.
- **Append-only**: each station adds an entry (operation, machine, program, measurements,
  offsets changed, operator, time). Nothing is overwritten; a correction is a new entry that
  points at the old one. A hash chain over the entries makes later tampering visible.
- **Printed traveller**: a label or sheet with the part number, serial, operation list, a
  summary of results so far and a **DataMatrix** that encodes the record's ID (and optionally a
  URL). Re-printed at each station so the paper always shows the latest state.
- **Offline first**: each station keeps a local copy and syncs when the network is available;
  a station with no network still prints and still accepts the part (the code identifies it).
- **Different machines, same record**: Op 10 on a mill, Op 20 on a lathe, a hand-gauge check on
  the bench and a CMM at final inspection all append to the same record. A setup on the same
  machine (flip the part, Op 20) is just another entry.
- **Datums across setups**: when Op 20 probes the part to set its work offset, the probed datum
  values are themselves recorded, so a later out-of-position feature can be traced to a setup
  error, not a tool error.

### Part serialisation and marking

Can the part carry its serial? Yes; the method depends on material, size and customer rules:

- **Engraving on the machine**: a small engraving tool or spot drill writes text in the same
  setup. Several controls can engrave a sequential serial number that increments each cycle
  (Haas G47 has a serial-number mode; unverified detail). Cheapest, no extra equipment.
- **DataMatrix on the machine**: a 2D code can be milled as a grid of dimples with a spot drill
  or a small ball end mill; readable codes need care with cell size and contrast (unverified as a
  reliable method; opinion that it suits larger cells, e.g. ≥ 0.5 mm, on flat faces).
- **Dot peen** (a stylus hammers dots): durable, good for DataMatrix on metal, common on castings
  and forgings.
- **Laser marking** (fibre laser): fast, high contrast, small codes; anneal marks on stainless
  avoid removing material.
- **Electro-chemical etch**, **vibro-etch**: low-cost, manual.
- **Tags or labels** where marking is not allowed on the part (bag-and-tag, wired tags).

Standards that apply when a customer requires them:

- **MIL-STD-130N** (US DoD): items over USD 5,000 (and some others) carry a Unique Item
  Identifier in an **ECC 200 DataMatrix** with ISO/IEC 15434 syntax, maximum 50 characters, which
  must verify at **grade B or better** (ISO/IEC 15415, AIM DPM-1-2006 or SAE AS9132). Construct 2
  (CAGE + part number + serial) is the common form. **Fact** (Wikipedia summary of the standard).
- **ISO/IEC TR 29158** (direct part mark quality guideline) is the usual method for grading
  marks made directly on parts. **Fact** (verifier makers' pages).
- **AS9132** (aerospace DataMatrix marking quality). **Fact.**

For SolidCraft this suggests (opinion): a "marking" feature in the model (a face, a location, the
code type, the content template such as `{part}-{rev}-{serial}`), which CAM turns into an engrave
toolpath and the edge device fills with the actual serial at run time.

## Accuracy

- **Of the ideas in this file:** the worked examples are exact arithmetic on standard tables.
  The 50 % feedback, dead band and correction limit are example settings; real values come from
  each shop's process and the probe package's own guidance.
- **Of on-machine measurement:** see [Accuracy of on-machine measurement](#accuracy-of-on-machine-measurement).
  Expect a few micrometres on a well-kept machine for size, more for position over long distances
  and with temperature changes (unverified as a general figure; ISO 230-10 testing gives the real
  number per machine).
- **Of standards descriptions:** ISO and SAE standards are paid documents; this file relies on
  catalogue entries, free previews and secondary summaries. Before implementing a rule (for
  example ISO 2768 values, ISO 286 tables, ISO 21920 defaults), check the standard text.

## Programs

Public descriptions only; no program was run for this file.

| Area | What exists (public descriptions) |
|---|---|
| Probe macro packages | Renishaw Inspection / Inspection Plus, Haas WIPS (VPS templates), Blum, Marposs, Heidenhain touch-probe cycles on TNC controls (unverified for Blum/Marposs/Heidenhain details). Run on the control; correct offsets; print results. |
| CAM with probing | Several commercial CAM programs output probing moves and cycle calls from the CAM setup (e.g. an Autodesk blog on using Fusion with a Renishaw spindle probe). Fusion's help pages describe Probe WCS, Probe Geometry, Inspect Surface, part alignment and manual inspection; see [From inspection back to the design](#from-inspection-back-to-the-design). Not run for this file. |
| PC-based on-machine inspection | Probe makers sell PC software that plans and reports probing on the machine from the CAD model (unverified details). |
| CMM software | Reads PMI or QIF, runs DMIS, writes QIF/PDF reports. |
| FAI/ballooning software | Balloons a drawing or model and produces AS9102 forms (multiple vendors). |
| Machine monitoring | MTConnect agents, Fanuc FOCAS-based collectors, Haas machine data collection; commercial monitoring boxes (MachineMetrics, Chatter) connect to Haas over Ethernet (Setting 143, port 5051 in Chatter's guide). |
| Open/free | NIST STEP File Analyzer and Viewer (PMI checking); QIF schemas published by the Digital Metrology Standards Consortium (licence to check); LinuxCNC's G38.x probing (behaviour described from public docs only; LinuxCNC is GPL, so no code reading). An open-source MTConnect agent for Haas exists on GitHub but is **GPL-2.0**, so it is out of bounds for SolidCraft under its rules. |

**SolidCraft today (repository, October 2026):**

- No CAM merged; proposals #1153 and #1621 (ROADMAP.md).
- Sketch and feature dimensions are exact values; no tolerance fields.
- Thread: designations and pitches; no tolerance classes or left-hand (docs/roadmap-detail.md,
  docs/thread-catalogue.md).
- STEP export: AP242 schema with product structure, assemblies, names and colours; no PMI
  (crates/kernel/src/step_out.rs).
- Drawings: in progress on the `drawings` branch (RFC #860); annotations deferred.
- Measure tool and section analysis exist in the app.
- Plugins: designed as sandboxed WebAssembly with one command surface (docs/plugins.md, #1221);
  a natural home for a CAM probing plugin.

## Complaints and requests

Common themes in public forums and vendor material (opinion where marked; no single source
proves frequency):

- **Writing probing macros is hard** and error-prone; operators copy old programs and edit
  variables by hand (forum threads asking how to update wear from a bore measurement, e.g. on the
  Tormach and eMastercam forums).
- **Tolerances are typed three or four times**: on the drawing, in CAM, in the probe macro, on the
  inspection sheet (opinion, consistent with the "characteristic accountability" problem AS9102
  addresses).
- **Results stay in the machine**: probe results print to a serial port nobody reads, or vanish
  when the next cycle overwrites the variables (opinion; data-collection guides note that probing
  results are only collectable if the macro writes them to a known variable range).
- **Paper travellers get lost or ruined**, and results are copied by hand into spreadsheets
  (opinion; common shop complaint).
- **Customers do not accept on-machine results** without correlation (opinion; follows from
  ISO 14253-1 and AS9102 practice).
- **Machine connectivity is fragmented**: each control family has its own interface, some need a
  paid option (Fanuc FOCAS needs the option enabled; parameter and option bits per the data
  collection guide).

## What users need

### Design side: tolerances as first-class data

Proposal for SolidCraft (opinion; command ids are our own and illustrative):

1. **Tolerance on any dimension or feature parameter**: symmetric (±), bilateral (+a/−b),
   limits (min/max), fit code (ISO 286, e.g. `H7`), thread class (`6H`, `2B`). Shown in the
   sketch and in feature dialogs; stored with the parameter so it follows edits.
2. **General tolerance per design**: ISO 2768-f/m/c/v (and a user table), applied to every
   dimension without its own tolerance, so every characteristic has limits.
3. **Datums and geometric tolerances on faces and features**: picked in the model, stored by
   persistent face names (SolidCraft already keeps feature identity through edits), shown as
   3D annotations.
4. **Surface texture per face**: Ra/Rz (or ISO 21920 parameters) and lay, with a default for the
   design.
5. **Tolerance checks while modelling**: a fit's clearance range, a simple 1D stack-up
   (worst case and RSS) between two picked faces, warnings for unachievable tolerances for a chosen
   process (opinion: a process-capability table per machine type, e.g. "milling, IT7 typical").
6. **Output**: drawings show them (when the Drawings workspace lands); STEP AP242 semantic PMI
   (following the MBx-IF recommended practices); QIF MBD.

In SolidCraft terms each of these is a command (everything is a command), stored as a new
pending design-format file, with layering: data in `doc`, exchange in `io`, UI in `ui-egui`
(opinion on placement; maintainers decide).

### CAM side: from tolerance to toolpath to probe cycle

1. **Machine to the target**: the finishing toolpath aims at the tolerance middle (or a
   user-chosen bias, e.g. "bores to the low side for a press fit").
2. **Inspection plan generated from the model**: for each toleranced feature, propose how it is
   checked (probe on this machine, hand gauge, CMM later) with the cycle type (bore, boss, web,
   pocket, surface, angle) and the point pattern. The user accepts or edits; a probe is a tool in
   the tool library (with its stylus length, ball size and calibration date).
3. **Post-processor writes the cycle calls** for the chosen probe package (or plain skip moves
   and macro logic for controls without one), with the limits, dead band, feedback fraction,
   correction limit, re-cut count and the offset to update, all filled from the model.
4. **Decision logic in the program**: pass / correct / re-cut / stop, entirely in the control.
5. **Output of every result** to a known place: DPRNT line, file, or a variable block, in a
   documented format the edge device can parse. Proposed line format (opinion):
   `SC1,<serial>,<op>,<char#>,<value>,<lo>,<hi>,<decision>,<offset#>,<delta>`.
6. **Feed back into the model of the process**: measured error per tool feeds the cutting
   physics (deflection estimates, tool wear rate) and the tool library's tool-life records
   (cross-reference: tool library and cutting physics files).

### Shop side: the per-part record

1. Serial assigned (or read from the marking) at the first operation.
2. Every station appends results; hand gauges by prompt.
3. Report at the end: QIF Results + PDF; AS9102 Form 3 when it is a first article.
4. SPC across parts: trends per characteristic, Cpk, early warning when a feature drifts.
5. Non-conformance: a rejected part's record links to the disposition (scrap, rework, use as is
   with concession).

### The edge device: a Raspberry Pi at the machine

**What Jason proposes:** a small box that plugs into any machine with probing, connects to the
network, stores reports in the cloud or on a corporate server, and prints a report locally to
travel with the part.

**What it can and cannot read (fact, from the interface descriptions):**

| Path | Works with | Reads | Writes | Notes |
|---|---|---|---|---|
| **Serial (RS-232) capture of DPRNT** | Many Fanuc-style controls and Haas (unverified per model) | whatever the program prints | nothing | Most universal; needs a USB-serial adapter and a matching serial setup; the program must print the results. |
| **Haas Ethernet machine data collection** | Haas NGC | status; macro variables by Q600 (unverified details) | limited (unverified) | Setting 143 = 5051 per Chatter's Haas NGC guide. |
| **Fanuc FOCAS2** | Fanuc with the Ethernet/FOCAS option | macro variables, offsets, status, alarms | macro variables, tool and work offsets (cnc_wrmacro, cnc_wrtofs) | Library on Linux ARM exists (libfwlib32); option must be enabled; licence of the library to check. |
| **MTConnect** | Controls with an agent/adapter (built into some) | what the adapter exposes | nothing (read-only) | Probe results only if the adapter maps the variables. |
| **OPC UA** (OPC 40501 Machine Tools) | Controls with an OPC UA server (Okuma OSP; Siemens, Heidenhain options; unverified per model) | per server | per server | Standard information model for monitoring and job management. |
| **Network file share** | Most modern controls | files the program writes (unverified) | program files | Simplest on controls that can write to a share. |
| **Operator entry** | All | hand-gauge values | nothing | Touch screen, keyboard, barcode scanner, gauge interface. |
| Probe signal directly | none | — | — | The trigger only means something together with axis positions latched inside the control; the Pi cannot do this. |

**Design principle (opinion, strongly held):** the Pi **never writes compensation** to the
machine by default. Correction happens in the control's macro (fast, deterministic, works when
the network is down). Optional "write-back" (e.g. push a corrected wear offset computed from a
CMM result into FOCAS) is a separate, explicit, logged feature with limits, off by default.

**Hardware sketch (opinion):**

- **Compute**: a Raspberry Pi 5 or a Compute Module 4/5 on an industrial carrier. For a product,
  an industrial Pi such as the Kunbus Revolution Pi (CM4/CM5, DIN rail, 24 V, −25 to +55 °C,
  CE/UKCA, UL-certified, EN 61131-2 and EMC to IEC 61000-6-2/-4, from about EUR 270–420) saves
  most of the compliance work. eMMC storage, not SD cards (SD cards fail under power cuts).
- **I/O**: USB-serial (RS-232, with isolation) for DPRNT; Ethernet to the control (second port to
  the plant network, keeping the machine network separate); USB barcode/DataMatrix scanner;
  optional digital gauge interface.
- **Printer**: a thermal label printer (ZPL-speaking printers work under CUPS on Linux) for
  traveller labels with the DataMatrix, and/or an ordinary office printer on the network for a
  full-page report.
- **Screen**: small touch screen or the operator's tablet through a local web page.
- **Power and enclosure**: 24 V DC from the machine cabinet's supply (common in machine
  cabinets) or a wall adapter; IP-rated enclosure away from coolant; real-time clock with battery
  so timestamps survive power loss.
- **Bill of materials** (rough, opinion): DIY Pi 5 + case + RS-232 + scanner + label printer
  around USD 300–600; industrial version USD 700–1,200 (unverified; prices move).

**Software sketch (opinion):**

- Linux image with a small service: parsers per control family (DPRNT line format above, FOCAS,
  Haas data collection, MTConnect, OPC UA), the part-record store (append-only, hash-chained),
  a QIF Results writer, the label/report renderer and a sync client.
- Storage targets: local first; then a corporate share (SMB), an S3-compatible bucket, or a
  quality system's API; the user chooses. No vendor cloud required.
- Security: signed updates, no inbound ports on the plant side, machine network isolated, reports
  signed (so a customer can check a report was not edited).
- Written in Rust if it is to share code with SolidCraft (e.g. the QIF and record model);
  application code in SolidCraft is Rust only (AGENTS.md).

**Feasibility (opinion):**

- **Technically**: yes. Every piece exists today: Pi-class industrial computers, serial capture,
  FOCAS on Linux ARM, CUPS label printing, QIF schemas. The hard part is the long tail of control
  variants and the per-shop setup (serial settings, variable ranges, which output the program
  makes), which is support work, not invention.
- **As a quality record**: it is only as good as its traceability: calibrated probes, recorded
  correlation, documented software. Aerospace customers (AS9100) expect software used for product
  acceptance to be validated (unverified; check the customer's requirements). That is a real cost
  for a product but fine for shop-internal process control.
- **Liability**: a device that marks parts "accepted" carries risk if it is wrong. Wording such as
  "records the machine's measurements and decisions" is safer than "certifies".

**Offering it through SolidCraft channels (opinion; the maintainers' decision):**

SolidCraft is an open-source project under MIT OR Apache-2.0 with a README sponsor section and
store listings (Snap Store) but no shop. Paths, from least to most commitment:

1. **Open reference design**: SolidCraft publishes the record format, the DPRNT line format and a
   reference image/BOM in a separate repository; anyone builds their own. No hardware sales, no
   liability for hardware.
2. **"Works with SolidCraft" listing**: third-party vendors (an industrial-Pi maker, a machine
   dealer, Jason or another business) sell ready boxes; SolidCraft's docs list compatible devices
   that pass a public conformance test (reading a sample DPRNT stream, writing valid QIF).
3. **Official kit with a partner**: a hardware partner builds and certifies (CE/FCC/UKCA); the
   Raspberry Pi Integrator Programme helps small companies through compliance testing, and the
   "Powered by Raspberry Pi" mark is available to approved products. Revenue could fund the
   project, but needs a legal entity, warranty and support.

#### A commercial retrofit kit: "upgrade any machine"

**Jason's idea:** commercialise a Pi package that also includes machine sensors (vibration,
temperature) and coolant sensors (concentration by refraction, coolant temperature, and so on),
to upgrade any machine while optimising production and quality. The kit is the hardware for
[self-tuning CAM operations](#self-tuning-cam-operations) and for the per-part record; using the
same signals to **predict failures and schedule maintenance** is covered by the companion
predictive-maintenance research (topic "Predict machine failure and order maintenance").

**Sensor options (prices are rough single-unit street prices; unverified, they move):**

| Measures | Sensor | Rough cost | Why it matters | Notes |
|---|---|---|---|---|
| Vibration (chatter, imbalance, bearing condition) | MEMS accelerometer (low cost) or IEPE industrial accelerometer with an ADC board | USD 30–100 (MEMS); 250–650 (IEPE + ADC) | stability check per segment; self-tuning margin | magnetic or stud mount on spindle housing; sampled at kHz on the box |
| Spindle and axis power | current transformers on the drive supply, or a Modbus power meter | USD 20–80 per CT; 100–300 meter | load measured independently of the control; works on old machines without data output | **electrical work inside the cabinet must be done by a qualified person**; clamp-on CTs avoid breaking circuits |
| Temperatures | RTD or thermocouple probes with converter boards; IR sensor for spindle nose | USD 15–60 each | thermal growth and warm-up (machine library); part temperature at probing | 4–8 points: spindle housing, column, table, ambient, coolant |
| Coolant temperature | RTD in the sump or return line | USD 20–60 | viscosity, part temperature, thermal growth | cheap and reliable |
| Coolant level | ultrasonic or float sensor | USD 20–60 | top-up alerts; evaporation raises concentration | |
| Coolant concentration | (a) operator reading with a handheld digital refractometer, entered by scan; (b) sump-side automatic unit; (c) inline process refractometer | (a) USD 100–300; (b) maker's price, not published in the sources found; (c) USD 12,000+ per point (data-collection guide) | concentration drift affects tool life, finish and corrosion | reading in °Brix × the coolant's refractometer factor = % concentration; tramp oil and temperature disturb the reading (unverified magnitude); a DIY inline refractometer is not a realistic first product (opinion) |
| Coolant pH / conductivity | electrode probes with transmitters | USD 50–500 | ageing and bacterial growth | electrodes foul in oily fluid and need cleaning (data-collection guide) |
| Coolant and air pressure | 4–20 mA pressure transmitters | USD 30–100 each + ADC | through-spindle coolant failures, low air for tool changes | |

**Kit tiers (opinion, rough totals including the box from the edge-device section):**

- **Inspect** (records, traveller, printer, scanner, serial capture): about USD 400–1,200.
- **Monitor** (+ accelerometer, 4–8 temperatures, power CTs, coolant temperature and level):
  + USD 300–1,000.
- **Coolant** (+ automatic concentration unit or inline refractometer, pH/conductivity):
  + USD 1,500 to well over 12,000, depending on whether a sump-side unit or a process
  refractometer is used.

**Existing products to compete or partner with (public descriptions only):**

- **Caron Engineering TMAC**: tool monitoring and adaptive control with a scaling power sensor;
  compares load to limits, adjusts feeds and speeds in real time, stops on breakage, records every
  cut, and (v3.0) labels cuts by part feature; "universal interface" to nearly any CNC.
  **Caron DTect-IT** detects machine irregularities with USB vibration sensors.
- **Liquidtool Sensor and Manager**: magnet-mounted next to the sump, pumps a filtered sample past
  a built-in refractometer and thermometer, networked, with app entry of pH, nitrite and hardness.
- **Inline process refractometers** (e.g. Anton Paar L-Rix 2100/3100) and process sensors from
  instrument makers (e.g. Vaisala lists metalworking applications): accurate, expensive, built for
  process plants.
- **Machine monitoring services** (MachineMetrics, Chatter and others): control data plus
  optional sensors, cloud dashboards, subscription pricing (unverified details).
- **Industrial Pi hardware** (Revolution Pi and others): a base to build on rather than a
  competitor.

**Where an open kit could stand out (opinion):** the incumbents each cover one piece (tool load,
vibration, coolant, dashboards). None of the public descriptions reviewed ties the readings to
**CAM's predictions per toolpath segment** and to **per-part inspection results** in an open
format. That link, and an open data format with no mandatory cloud, is the differentiator,
not the sensors themselves.

**Practicalities for a product (opinion):**

- **Never connect to safety circuits** (e-stop, door interlocks). Alarm outputs to the control
  only through an input the machine builder documents for that purpose.
- **Compliance**: the assembled kit needs its own EMC and electrical-safety conformity (CE/UKCA,
  FCC); starting from a pre-certified industrial Pi and pre-certified sensors reduces the work but
  does not remove it. The Raspberry Pi Integrator Programme exists to help small companies with
  this testing.
- **Installation and support** dominate cost: mounting, wiring, per-control setup, calibration.
  A dealer or integrator network fits better than mail order for the Monitor and Coolant tiers.
- **Calibration records** for anything used in acceptance (probes, gauges, temperature sensors
  used to correct measurements).

**Through SolidCraft channels:** the same three paths as above apply. The open part (record
format, segment-aligned data format, conformance tests, reference BOM) belongs with SolidCraft;
the kit itself is best sold by a separate business (Jason's or a partner's) listed as
"works with SolidCraft", subject to the maintainers' permission to use the name.

Someone outside the core team (for example Jason) could start at path 2 without asking anyone
except the trademark question: using the "SolidCraft" name on a product needs the maintainers'
permission (opinion; check the project's trademark position).

### Machine monitoring: loads, torque and vibration

**Jason's idea:** the Raspberry Pi box could also feed back machine vibration, loads and torques
in real time, because some machines already output them. This closes the loop described in the
[CAM physics system](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-vgqo3g/research/cam-research-nsnlookupjn-physics-system.md)
file ("The closed loop: measuring what was predicted") and the "MTConnect (shop) → library" row
of the [machine library](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-machine-library/research/cam-research-nsnlookupjn-machine-library.md)
interface table.

**What the machine can give (fact where cited, otherwise unverified per control):**

| Signal | Typical source | Rate the box can get | Compared with |
|---|---|---|---|
| Spindle load (% of rated) | control load meter via FOCAS, MTConnect, OPC UA, Haas data collection | polled, about 100–1,000 ms (the data-collection guide's figures) | predicted cutting power `Pc` against the spindle curve at that speed (machine library) |
| Axis / servo loads | FOCAS servo load meters; MTConnect `Load` on axes (unverified per adapter) | polled, same | predicted feed forces `Fx`, `Fy`, `Fz` against axis thrust (machine library) |
| Spindle torque | some controls report it, or derive from load % × rated torque at that speed (opinion) | polled | predicted `Mc` (cutting physics) |
| Actual feed and speed, override % | all of the above | polled | the planned feed per segment (is the operator overriding it?) |
| Vibration | **add-on** accelerometer on the spindle housing or table, read by the Pi itself; a few controls and spindles have built-in sensors (unverified) | sampled locally, kHz | tooth-passing frequency `n·z/60` and its harmonics vs chatter frequencies near the machine and tool modes (cutting physics, machine library) |
| Temperature | add-on sensors, or control thermal compensation values (unverified) | seconds | machine library's thermal growth and warm-up |

Two practical limits (opinion, following from the rates above):

- **Polled control data is slow.** At 100–1,000 ms per sample, spindle and axis loads show the
  average load of a cut, not each tooth. That is enough to calibrate specific cutting energy
  (`kc1.1`) and check power and thrust margins, which is what the libraries need.
- **Vibration needs its own sensor.** Chatter lives at hundreds to thousands of hertz, so it must
  be sampled at the sensor (a MEMS or IEPE accelerometer and an ADC on the Pi) and reduced on the
  box to a few numbers per segment (RMS, dominant frequency, whether it is a tooth-passing harmonic
  or not). Raw waveforms stay local unless asked for.

**What the box does with them (opinion):**

1. **Time-align to the G-code and the timeline** (Jason's requirement; see
   [Aligning machine data with the G-code](#aligning-machine-data-with-the-g-code-and-the-timeline)).
   Every sample and event carries a timestamp, the program, the block (line) and the CAM segment,
   so an exact event can be found on the program, on the toolpath in 3D and in the part record.
2. **Compare with the prediction.** For each segment: predicted vs measured power, thrust,
   vibration level. Store the ratio, not just the value.
3. **Calibrate the shop instance**, never the shared catalogue: update this machine's effective
   power and thrust margins, this material lot's `kc1.1`, this tool's wear trend, this setup's
   chatter-free depth. The physics system file states the same rule: feedback updates the shop
   instance; contributing back to the shared library is the shop's choice, labelled as measured.
4. **Feed the part record.** A part cut with a chatter event, an overload or a broken-tool alarm
   gets that event in its traveller next to the dimensional results, so an inspector knows which
   features to look at.
5. **Show it.** A per-part and per-toolpath chart (planned vs measured) is the easiest way for a
   novice to see where the program was too timid or too aggressive, which ties into the
   conservative-to-aggressive slider in the cutting physics file.

**Staging (matches the CAM physics system file's closed loop):** first **record and compare**
(this section), then **detect and alarm** (chatter onset, overload, broken tool flagged to the
operator and the part record), and only later **feed adjustment**, and then only through the
control maker's own adaptive-control interface, not from the box.

**Corrections stay in the control.** The box monitors and learns; it does not change feeds,
speeds or offsets while the machine runs. Real-time feed adjustment from load (adaptive control)
exists as control options and commercial products, runs inside or beside the control with
deterministic timing, and is safety-relevant. A Linux Pi on a network is not a real-time
controller. If a shop wants the box to act, the safe forms are (a) a recommendation shown to the
operator or the CAM programmer for the next run, and (b) an alarm output (a dry contact into a
control input that the program checks at a safe point, e.g. "stop after this tool"), designed
and wired by the machine's integrator. Both are opinions, and (b) needs the machine builder's
approval.

#### Aligning machine data with the G-code and the timeline

**Jason's requirement:** the machine data should be aligned with the G-code and the timeline so
exact events can be located. A load spike is only useful if you can say "block N4520, T7, second
finishing pass on the pocket wall, 41.2 s into Op 20, part SN000123".

**Keys every record carries (opinion):** wall-clock time (UTC, with the box's clock synchronised
by NTP, or PTP where the plant has it), machine, program name and revision, **block or line
number**, **CAM segment ID**, operation, tool number, machine position, active work offset,
spindle speed, feed and override, and the part serial. The physics predictions are indexed by the
same keys, so measured and predicted values line up row by row (the same scheme as the
[CAM physics system](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-vgqo3g/research/cam-research-nsnlookupjn-physics-system.md)
file's self-tuning loop). Clock drift between the box and the control is corrected at known
events: program start, each tool change and each reported block number.

**Where the program position comes from:**

- **Read from the control.** MTConnect defines data items for the program, the current block and
  the line number (unverified per adapter which ones are filled); Fanuc FOCAS can read the
  executing sequence (N) number (unverified exact function); OPC UA machine-tool servers expose the
  running program per the companion specification (unverified detail). Polled values carry the
  poll delay (100–1,000 ms), so they locate a segment, not a single tooth.
- **Written by the post-processor (more reliable, opinion).** CAM numbers its segments and the
  post writes the segment ID into a macro variable (or emits an N number) at the start of each
  segment; the box reads that variable, or the program prints a short marker line (DPRNT) at key
  points such as tool changes, probe cycles and the start of each finishing pass. The same ID
  exists in CAM's simulation, so measured data lines up with predicted data segment by segment.
  Markers should be sparse: output statements can interrupt look-ahead on some controls and leave
  dwell marks (unverified per control; test before using inside a finishing pass).
- **Recovered from the signals.** Spindle start/stop, tool changes and the start of a cut show as
  steps in load and vibration. Matching those steps against the CAM's predicted times corrects
  clock offsets and poll delays (cross-correlation; opinion that this tightens alignment to well
  under one poll interval for events with a clear edge).

**What alignment gives the user (opinion):**

- **Timeline view**: one time axis with the program's blocks, the predicted load and the measured
  load and vibration; click an event to jump to the G-code line.
- **3D view**: the event marked on the toolpath and on the part's face (the same place a probe
  result is shown), so "chatter here" and "this wall measured 0.03 thin" appear side by side.
- **Part record**: events (alarms, feed holds, override changes, chatter onset, overload, tool
  breakage, probe results, offsets changed) become traveller entries with their block, segment
  and time, next to the dimensional results.
- **Replay**: re-running the CAM simulation with the measured data overlaid, which is how a
  programmer learns where a program was too timid or too aggressive.

**Hardware addition (opinion):** an accelerometer kit (a magnetic-mount IEPE accelerometer and a
24-bit ADC board, or a MEMS sensor for a low-cost version) adds roughly USD 50–400 to the box
(unverified; prices move). Mount position and cable routing away from coolant matter more than
sensor grade for chatter detection.

### From inspection back to the design

**Jason's vision:** the inspection features in Fusion (automatic probing, manual measuring and
gauging) are a wonderful idea, but they were never fully developed. They rely on fully manual
entry or on a machine connector that was always clunky. SolidCraft should have a beautiful,
comprehensive inspection system that **drives back into the design during manufacturing**, so
the design gets iteratively better in performance, time, cost and quality.

#### What Fusion offers today (public help pages only; Fusion was not run for this file)

- **Probe WCS:** probing cycles locate the stock and update the setup's work offset.
- **Probe Geometry** (Manufacturing Extension): probing cycles or a probe path check holes,
  bosses, walls, channels and corners in process.
- **Inspect Surface** (Manufacturing Extension): a probe path over free-form surfaces, with an
  in- or out-of-tolerance result per point.
- **Part alignment / live part alignment** (Manufacturing Extension): computes the part's real
  position from probing to compensate for imprecise fixturing of castings and forgings.
- **Manual inspection:** an inspection plan in Fusion where measurements from calipers,
  micrometers and height gauges are entered by hand.
- **Getting results back:** either import a results file the machine writes, or use a live
  connection through a separate Machine Connector application installed for that machine.
  Results can be saved as a report.

**Gaps (Jason's experience as a user, and what the public sources show):**

- **Connection.** Results come back by file import or a per-machine connector app. Public forum
  thread titles show the friction, for example "Live Inspection Fusion 360 and my Machine wont
  connect", "Surface Inspection DEPRINT Error on Haas NGC", "Fanuc Inspection Connector",
  "INSPECTION RESULTS PROBLEM", and a Tormach forum asking whether machine connect is supported
  at all. Thread titles only; the Autodesk forum pages could not be fetched for details.
- **Paywall split.** Most probing inspection needs the paid Manufacturing Extension, while manual
  inspection does not, so many shops have half of the loop (fact, per the help page).
- **Results stop at a report.** The public pages describe recording and reporting. They do not
  describe feeding results into the next part's offsets, a per-serial record across setups, SPC,
  or changes back to the design (gap as far as the public documentation shows; Fusion may do more
  than its overview pages say).
- **Tolerances.** The inspection checks against tolerances set on the operation, not semantic
  PMI that the design owns (unverified; based on the help pages' description of the strategies).

#### UX goals: what "beautiful and comprehensive" means (opinion)

1. **One inspection plan generated from the model.** Every characteristic gets a balloon number,
   limits and a method (probe, gauge, CMM) automatically. The user reviews and edits; nothing is
   typed twice.
2. **No connector setup.** Results arrive through whichever path the machine already has (DPRNT,
   network, file share, operator entry on the edge box). Adding a machine is choosing its control
   family from a list.
3. **Results on the model.** Each feature is coloured by its result (centred, drifting, out),
   with the trend across serials one click away. A surface inspection shows a deviation map, not
   a table.
4. **Manual gauging as fast as probing.** The screen shows the next characteristic to check with
   its limits and a picture of the feature. A gauge button press or a go/no-go tap records it, and
   the traveller updates.
5. **Every setup, every machine, one record.** Scan the traveller and the right plan for this
   operation appears.
6. **Explanations, not just numbers.** "Bore 7 drifting +4 µm per 10 parts, matching T12 wear;
   offset corrected 3 times" beats a red cell.
7. **Reports as a by-product** at the chosen [report level](#the-per-part-record-and-report), with
   no manual ballooning.
8. **Works with no extension, subscription or cloud.** SolidCraft is open source; the record
   format is open.

#### The design-feedback loop (opinion)

Inspection results and machine data (aligned to segments, as above) answer four questions per
feature. Each answer can become a **suggestion** on the design or the process:

| Evidence | Suggests | Improves |
|---|---|---|
| A feature is always centred with a small spread (high Cpk) | keep the tolerance (it may be functional) and use the margin: faster settings (self-tuning), fewer checks (lower report level or sampling), or a cheaper process | time, cost |
| A feature drifts or scatters near a limit (low Cpk) | tighten the process (finish pass, tool, hold), or **propose a wider tolerance** if the function allows it (checked against the fit or stack-up that set it) | quality, cost |
| A feature is always out of tolerance until compensated | change the **nominal** in CAM (or the design if the feature is a fit) to the measured bias, or change the feature (deeper relief, larger corner radius, thicker wall) | quality, time |
| A thin wall chatters or deflects (vibration, probed thickness) | thicker wall, rib, different hold, different order of operations | quality, time |
| A feature takes most of the cycle time (machine data per segment) | larger internal radius (bigger tool), shallower pocket, a standard hole size or thread, a draft | time, cost |
| Measured loads far below limits on all segments | more aggressive defaults for this material and machine (shop instance) | time |
| Rejects cluster on one feature across machines | design review: is the tolerance achievable for the process (general-tolerance profile)? | cost, quality |
| Performance tests (from the user or FEA) show margin | reduce material or relax a tolerance; or, without margin, add it back | performance, cost |

Rules for the loop (opinion):

- **Suggestions, never silent edits.** The design changes only by a person accepting a proposal.
- **Every accepted change is a new design revision** with the evidence attached (the
  characteristics, serials, machine data and charts that justified it). It goes through the
  revision and release workflow in the
  [compliance research](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-ymp2ma/research/compliance-research-nsnlookupjn-itar-quality.md) (section "2. Revision and release workflow"), so travellers and
  inspection records always point at one exact released revision. A change that affects fit,
  form or function triggers a partial (delta) FAI on the changed characteristics only. That file
  owns the workflow, signatures and nonconformance/corrective action (section "8"); this file
  supplies the evidence.
- **Process-only changes** (feeds, offsets, tool choice, report level) stay outside the design
  revision but are still versioned with the CAM program.
- **Cost and time are tracked per feature.** Machine time per segment, inspection time per
  characteristic, scrap and rework per feature give a cost per feature. A proposal can then show
  "this change saves 2 min 40 s and USD 6 per part" next to "Cpk 0.9 → 1.6".
- **The loop runs during production, not after.** The suggestions appear while the lot is still
  running, so the next lot (or the next revision) benefits at once. This is what makes the design
  "iteratively better" rather than reviewed once a year.

### Self-tuning CAM operations

**Jason's idea:** call it *self-tuning CAM operations*: the live machine data from the edge box,
aligned to the G-code, tied into in-process inspection, "so that we can go harder without
affecting accuracy or surface finish". It is the closed loop of the
[CAM physics system](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-vgqo3g/research/cam-research-nsnlookupjn-physics-system.md)
("The closed loop: measuring what was predicted"; phase 6 of its plan) with inspection added as
the quality gate.

**Two kinds of evidence, one decision (opinion):**

| Evidence | From | Says |
|---|---|---|
| Process margin | spindle/axis load vs the machine library's curves, torque vs the tool library's limits, vibration vs the stability prediction, per CAM segment | "this segment could take more" or "this segment is near a limit" |
| Quality margin | probe results, hand-gauge results, surface roughness checks, per characteristic, mapped to the segments that cut each face | "this feature is centred with room to spare" or "this feature is drifting toward a limit" |

A segment may be pushed harder only when **both** say there is margin. Either one alone is not
enough: a cut can run at low load and still leave a bad finish (chatter, deflection), and a part
can measure well while the spindle is at its limit.

**How a cycle of self-tuning works (opinion):**

1. **Run** the program as planned (from the physics system at the slider's current setting).
2. **Record** loads, vibration and events aligned to segments; **inspect** the characteristics
   in process and at the bench.
3. **Map** each characteristic back to the segments that made it (the CAM knows which toolpath
   cut which face; the part record knows the result). Roughing segments feed only stock left for
   finishing, so their quality evidence is the finishing pass's correction size and stability.
4. **Propose** changes for the next part or batch: raise feed or depth on segments with process
   margin whose features have quality margin; lower them where vibration rose or a feature
   drifted; leave finishing passes alone until roughing gains are taken (the large time savings are
   in roughing; finishing is bounded by deflection and the roughness target).
5. **Check** each proposal against the physics limits again (tool, holder, spindle, axes,
   workholding, stock); the physics models are the safety net, never relaxed by the data.
6. **Apply** between parts, never mid-cut: either the programmer approves, or the shop allows
   automatic steps inside set bounds (e.g. at most +10 % per step, one change per segment group
   per batch). The conservative-to-aggressive slider sets those bounds.
7. **Watch** the next parts; if a characteristic's capability (Cpk) falls or a guard band is
   approached, roll back to the last good settings automatically and flag it.

**What "without affecting accuracy or finish" means in numbers (opinion):** keep each
characteristic's measured spread inside its ISO 14253-1 acceptance zone with a set capability
(for example Cpk ≥ 1.33, a common customer minimum, unverified as universal), and keep measured
or predicted roughness below the requirement with a margin. Roughness is rarely measured on the
machine, so sample it at the bench (profilometer) every N parts, and use the predicted value from
feed/step-over plus the vibration level as a proxy in between (the proxy needs correlation per
shop; unverified how well vibration predicts Ra).

**Why inspection makes "harder" safe:** today a programmer stays conservative because they
cannot see the margin. With measured margin per feature and per segment, the program can be
pushed exactly where the evidence allows, and pulled back automatically when it no longer does.
The per-part record also proves to a customer that the faster program still produced
conforming parts.

**Where it stays careful (opinion):** first articles and any characteristic flagged critical
are never tuned without a person approving; a change of material lot, tool brand or machine
starts a new tuning history (the shop instance in the libraries, not the shared catalogue); and
corrections inside a run remain the control's in-cycle compensation described earlier.

### Data model sketch

Illustrative only (opinion), one characteristic as it might be stored and exchanged:

```json
{
  "char": 7,
  "feature": "face:Hole1/bore",
  "kind": "size.diameter",
  "nominal": 20.0,
  "spec": "H7",
  "lower": 20.000,
  "upper": 20.021,
  "target": 20.0105,
  "method": ["probe.bore@op10", "gauge.bore@final"],
  "results": [
    {"serial": "SN000123", "op": 10, "machine": "VF2-01", "value": 20.004,
     "u": 0.003, "decision": "pass", "offset": "D12", "delta": -0.0033,
     "time": "2026-10-26T15:30:00Z"}
  ]
}
```

In QIF terms: `kind`/`spec`/limits are a characteristic definition and nominal (QIF MBD/Plans);
`results` are measured characteristics (QIF Results).

## Glossary

- **AP242**: STEP application protocol for managed model-based 3D engineering; carries PMI.
- **AS9102**: aerospace standard for First Article Inspection (Forms 1–3).
- **Balloon / characteristic number**: the numbered tag for each requirement on a drawing.
- **Cpk**: process capability index; how centred and narrow the process is relative to the
  tolerance.
- **DataMatrix (ECC 200)**: square 2D code, readable when small, used for part marking.
- **Datum**: a theoretically exact feature (plane, axis, point) from which tolerances are
  measured.
- **DMIS**: CMM programming language (ISO 22093).
- **DPRNT**: G-code macro statement that prints text and variables to a port or file.
- **FAI**: First Article Inspection.
- **FOCAS**: Fanuc's library for reading and writing CNC data over Ethernet.
- **GD&T / GPS**: geometric dimensioning and tolerancing (ASME) / geometrical product
  specifications (ISO).
- **Guard band**: margin inside the tolerance to account for measurement uncertainty.
- **IT grade**: ISO 286 standard tolerance grade (IT7, IT6…).
- **MBD**: model-based definition; the 3D model with PMI is the authority, not a 2D drawing.
- **MTConnect**: read-only XML/HTTP standard for machine-tool data.
- **OPC UA**: industrial communication standard; OPC 40501 is its machine-tool information model.
- **PMI**: product and manufacturing information (tolerances, datums, finishes, notes).
- **PPAP**: automotive production part approval process.
- **QIF**: Quality Information Framework (ISO 23952), XML for the quality chain.
- **Skip (G31)**: a move that stops when an input (the probe) triggers and latches the position.
- **Traveller (router)**: the document that goes with a part through its operations.
- **UII / IUID**: unique item identifier (US DoD marking).
- **Wear offset**: a small correction added to a tool's geometry offset, used for compensation.

## Open questions

1. **Scope**: should tolerances/PMI be a design-side milestone of their own (they also serve
   drawings and STEP), with this CAM inspection work depending on it?
2. **ISO or ASME first?** Both, with a per-design setting, or one default by locale?
3. **Which probe cycles to target first**: the probe makers' packages (calls only) or our own
   generic skip-move macros (portable, but every control dialect differs)?
4. **Where should the per-part record live** in SolidCraft: inside the design, beside it as a
   job file, or only in an external system that reads QIF?
5. **Monitoring data ownership**: should measured loads and vibration ever leave the shop, and
   in what reduced form, if shops want to improve the shared libraries?
6. **How to label on-machine results**: block them from "final acceptance" unless a correlation
   is recorded, or just label the method?
7. **The edge device**: does the project want an official hardware path at all, or only an open
   data format that any device (Pi or otherwise) can implement?
8. **Trademark**: may third parties use "SolidCraft" in a product name ("for SolidCraft")?
9. **Jason's TODO**: personal shop experience, and which controls (Haas, Fanuc, others) his shop
   would test with first.

## Sources

Standards (catalogue entries and public summaries, not the paid texts):

- ISO 21920 vs ISO 4287 differences (Digital Surf): https://www.digitalsurf.com/blog/what-are-the-differences-between-iso-4287-and-iso-21920/
- ISO 21920 parameters guide (Digital Surf): https://guide.digitalsurf.com/en/guide-iso-21920-parameters.html
- ISO 21920-2:2021 catalogue: https://iteh.es/catalog/standards/iso/38ba69b0-cc1c-4e3d-9414-2bf60ffc98f8/iso-21920-2-2021
- ISO QIF (ISO 23952:2020) parts: https://qifstandards.org/iso-qif/
- NIST, QIF and MBE summit material: https://www.nist.gov/document/2024-mbe-summit-qif-campbell-qif-overview
- MBx-IF (CAx-IF) Recommended Practices for PMI, v4.1: https://www.mbx-if.org/home/wp-content/uploads/2024/06/rec_pracs_pmi_v41.pdf
- NIST STEP File Analyzer and Viewer: https://www.nist.gov/node/413991
- ISO 22093:2011 (DMIS) catalogue: https://scc-ccn.ca/standardsdb/standards/8151462
- ISO 230-10:2022 catalogue: https://iteh.es/catalog/standards/iso/5afaee71-d342-4ce1-a169-08acb8b06d50/iso-230-10-2022
- ISO 14253-1:2017 catalogue: https://www.evs.ee/en/iso-14253-1-2017
- OPC 40501-1 Machine Tools: https://reference.opcfoundation.org/MachineTool
- MIL-STD-130 summary: https://en.wikipedia.org/wiki/MIL-STD-130
- ISO/IEC TR 29158 verification (Keyence): https://www.keyence.com/ss/products/auto_id/codereader/code_verification/tr-29158.jsp
- AS9102 forms guide: https://gndctl.com/insights-library/how-to-fill-out-as9102-forms
- AS9102 FAI guidebook (Lockheed Martin supplier document): https://www.f35.com/content/dam/lockheed-martin/eo/documents/suppliers/mfc/mfc-quality-fai-guidebook.pdf

Probing and machines:

- Renishaw inspection software for machining centres: https://www.renishaw.com/en/inspection-software-for-machining-centres--6093
- Haas WIPS (UK): https://www.haas.co.uk/wips/
- Haas M109 interactive user input (code reference): https://www.haascnc.com/it/service/codes-settings.type=mcode.machine=mill.value=M109.html
- Haas macro programming information (variable ranges reserved for probes): https://www.haascnc.com/content/dam/haascnc/service/guides/how-to/lathe-work-probe-(lpro-r)---chc---installation---ad0027/macro_programming_information.pdf
- Autodesk blog, using a Renishaw spindle probe with CAM: https://www.autodesk.com/products/fusion-360/blog/top-tips-using-autodesk-fusion-with-renishaw-spindle-probe/
- Tormach forum, updating tool diameter after probing a bore: https://forums.tormach.com/t/a-way-to-update-the-tool-diameter-or-use-wear-to-update-the-tool-after-probing-a-bore/1345

Connectivity and the edge device:

- CNC data collection: FOCAS, MTConnect and NC protocols: https://industrialmonitordirect.com/blogs/knowledgebase/cnc-data-collection-deep-dive-focas-mtconnect-and-nc-protocols
- Fanuc FOCAS2 library: https://www.fanucamerica.com/products/cnc/cnc-software/machine-tool-builder-software/focas2-library
- Chatter, Haas NGC connection guide (Setting 143, port 5051): https://support.chatter.dev/docs/integration/haas-ngc
- MTConnect agent for Haas (GPL-2.0, listed only to note it is out of bounds): https://github.com/TrakHound/MTConnect-Agent-For-Haas
- Machine library (same author): https://github.com/nsnlookupjn/solidcraft/blob/research/cam-machine-library/research/cam-research-nsnlookupjn-machine-library.md
- CAM physics system (same author): https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-vgqo3g/research/cam-research-nsnlookupjn-physics-system.md
- Caron TMAC (Production Machining): https://www.productionmachining.com/products/carons-tmac-system-offers-real-time-tool-monitoring
- Caron DTect-IT (Modern Machine Shop): https://mmsonline.com/products/monitoring-system-detects-vibration-with-usb-sensors
- Liquidtool coolant sensor (Fabricating & Metalworking): https://fandmmag.com/2021/07/automated-monitoring-of-coolant-results-in-cleaner-machines
- CNC coolant monitoring sensors comparison: https://industrialmonitordirect.com/blogs/knowledgebase/cnc-coolant-monitoring-sensors-technical-comparison
- Anton Paar L-Rix inline refractometers: https://manufacturingchemist.com/anton-paar-launches-l-rix-2100-and-l
- Vaisala, metalworking and machining: https://www.vaisala.com/ko/node/82806
- Xometry, manufacturing standards (US; reference suggested by Jason): https://www.xometry.com/manufacturing-standards/
- Xometry community, CNC machining and sheet fabrication inspection options: https://community.xometry.com/kb/articles/673-cnc-machining-sheet-fabrication-inspection-options
- Xometry, ISO 2768 resource page: https://www.xometry.com/resources/certifications/iso-2768
- Xometry Pro, standard tolerances (ISO 2768, ISO 286, GD&T): https://xometry.pro/en/?p=16868
- Xometry community, standard threads chart: https://community.xometry.com/kb/articles/724
- Additive process physics (same author): https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-j72y6h/research/3d-printing-research-nsnlookupjn-process-physics.md
- Autodesk Fusion help, Inspection for manufacturing: https://help.autodesk.com/cloudhelp/ENU/Fusion-CAM/files/MFG-PROBING-OVERVIEW.htm
- Autodesk Fusion help, Workflow: inspecting a surface using probing: https://help.autodesk.com/cloudhelp/ENU/Fusion-CAM/files/MFG-HOW-TO-INSPECT-SURFACE-WORKFLOW.htm
- Autodesk forum thread titles (pages not fetched): https://forums.autodesk.com/t5/fusion-manufacture-forum/live-inspection-fusion-360-and-my-machine-wont-connect/m-p/12330730 ; https://forums.autodesk.com/t5/fusion-support-forum/surface-inspection-deprint-error-on-haas-ngc/td-p/9575587 ; https://forums.autodesk.com/t5/fusion-manufacture-forum/fanuc-inspection-connector/m-p/12806864 ; https://forums.autodesk.com/t5/fusion-support-forum/inspection-results-problem/m-p/9786577
- Tormach forum, Fusion machine connect support question: https://forums.tormach.com/t/fusion-360-machine-connect-supported-on-tormach/872
- Compliance research (same author; revision/release, FAI, NCR/CAPA): https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-ymp2ma/research/compliance-research-nsnlookupjn-itar-quality.md
- Revolution Pi products: https://revolutionpi.com/en/products
- Raspberry Pi Integrator Programme and Powered by Raspberry Pi: https://www.raspberrypi.com/for-industry/integrator-programme/
- Zebra CUPS driver configuration: https://support.zebra.com/cpws/docs/cups/cups_driver1_4_configure.pdf

SolidCraft repository (commit `10b5cc5`): ROADMAP.md, docs/roadmap-detail.md,
docs/thread-catalogue.md, docs/plugins.md, crates/kernel/src/step_out.rs, AGENTS.md.
