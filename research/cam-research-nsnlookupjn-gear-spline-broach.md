# CAM research: gears, splines and broached shapes, from model to cut to gauge

Research for [#1621](https://github.com/tangency-labs/solidcraft/issues/1621) (Research Request
for Plugin: CAM). This file covers one part of that request: **toothed and keyed features (gears,
involute and straight-sided splines, keyways, broached polygons such as hex sockets), how a CAD
program should model them parametrically, which processes make them (hobbing, shaping, power
skiving, gear milling on 5-axis machines, form milling, push/pull broaching, stroke broaching on
mills and lathes, rotary broaching, wire EDM), the physics of each process, how the parts are
inspected, and how all of this plugs into the tool, holder and machine libraries and simulation
proposed in the companion files.**

Companion files (same author, same fork):

- [Cutting tool physics](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-cutting-physics/research/cam-research-nsnlookupjn-cutting-physics.md):
  the Kienzle model, `kc1.1`/`mc`, tool life and the conservative-to-aggressive slider. This file
  reuses that model for broach teeth and stroke-shaping edges and does not re-derive it.
- [CAM Physics System](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-vgqo3g/research/cam-research-nsnlookupjn-physics-system.md):
  the umbrella (one Setup, one limits engine, one slider, "explain every number"). Every process
  here is meant to be another client of that engine, not a separate calculator.
- [In-process inspection](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-y2w06n/research/cam-research-nsnlookupjn-in-process-inspection.md):
  the closed loop. "Inspection" below gives the gear and spline measurements that loop needs.
- [Tool holding](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-4aoliy/research/cam-research-nsnlookupjn-tool-holding.md),
  section "Non-rotating tools: orientation and alignment": owns the holder side of keyway and
  slot broaching tools and rotary broach holders (spindle orient and lock, edge alignment, centre
  height, twist, holder stiffness, centring). This file owns the process side (tooth load, rise
  per tooth, stroke force, pass plans, rotary broach feed and thrust) and hands that file a force
  vector and the alignment tolerances each process needs.
- [Workholding](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-workholding/research/cam-research-nsnlookupjn-workholding.md)
  and [machine library](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-machine-library/research/cam-research-nsnlookupjn-machine-library.md):
  receive the force vectors and machine capability requirements listed in "Library and
  simulation integration".

Licence: MIT OR Apache-2.0, the same as SolidCraft (see the Licence section of CONTRIBUTING.md).

## About this research

- **Author:** Jason ([@nsnlookupjn](https://github.com/nsnlookupjn)). Topic and direction by
  Jason ("What about gear cutting?", "What about splines and broached slots"); source research and
  writing done with an AI assistant (Claude) in October 2026.
- **How it was produced:** web searches of public sources (trade press such as Gear Technology,
  Gear Solutions and Production Machining, university theses, standards bodies' catalogue
  entries, engineering reference pages). No commercial CAD/CAM program was driven or scripted, no
  GPL/LGPL/AGPL source was read (several open gear generators exist as OpenSCAD or Python
  libraries; their code was not opened), and no text is copied beyond short quotes. Standards are
  described from their public catalogue entries and secondary summaries, not their paid text, so
  every table value here must be checked against the purchased standard before it ships.
- **Units:** millimetres, newtons, watts, radians, unless a source is quoted in other units.
  Inch-based standards (ANSI B92.1, ANSI B17.1) are described in their own units.
- **Marking:** "(unverified)" marks a claim not confirmed against a source. "(derived)" marks a
  formula or number worked out here from geometry or from the companion files' models, not quoted.
  "(textbook)" marks classical gear geometry that appears in every gear text and is not tied to
  one source. **Opinion** is labelled.

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

> **[TODO Jason: gear and broaching experience]** Which gears, splines and keyways you make or buy (module
> or DP range, internal or external, materials, batch sizes), on which machines (gear hobber,
> shaper, 5-axis, lathe with live tooling, broach press, wire EDM), and how they are inspected
> today (over pins, span micrometer, spline gauges, gear lab). Where it goes wrong: undercut,
> wrong pin size, broach breakage, keyways off centre, spline gauges not going on. Label this as
> personal experience.

## Summary

1. **Gears, splines, keyways and broached polygons are one family**: features defined by a
   standard designation and a handful of numbers (module or diametral pitch, tooth count,
   pressure angle, profile shift, fit class), whose exact shape depends on *how they are made*.
   The root fillet of a hobbed gear is the trochoid the hob tip sweeps, not an arc a CAD program
   chooses; a keyway's end is the runout of a broach, an end mill or a shaping tool. A CAD model
   that ignores the process is a picture, not a manufacturable definition.
2. **SolidCraft already has the pieces to start.** It has a `joint.gear` and `joint.rack_pinion`
   for motion (ratio from radii or expressions such as `m*z1/2`), a tapped-hole and thread
   catalogue that turns a designation into geometry plus metadata (`docs/thread-catalogue.md`),
   cosmetic thread markings that keep big models light (`docs/cosmetic-threads.md`), and its
   plugin design already uses `"gear_generator"` as its example plugin id (`docs/plugins.md`).
   This file proposes **Gear, Spline, Keyway and Broached Profile features built the same way as
   the thread catalogue**: a designation, stored intent metadata, generated exact geometry, an
   optional cosmetic display, and the metadata CAM and inspection read back.
3. **Generate the profile by simulating the cutter, not by drawing the involute.** The
   rack-generation method (roll ISO 53 basic rack or a shaper cutter around the blank and keep
   the envelope) gives the true involute, the true trochoidal root, correct undercut on low tooth
   counts (below about 17 teeth at 20° without profile shift (textbook)) and correct profile
   shift, all from one algorithm. The same envelope code is the core of gear-process
   simulation in CAM.
4. **Processes, in the order a job shop meets them:**
   - *Hobbing* (continuous generating with a worm-like hob): the default for external spur and
     helical gears and splines with run-out room.
   - *Shaping* (a reciprocating pinion-shaped cutter): internal gears and gears next to
     shoulders.
   - *Power skiving* (a crossed-axis cutter, continuous): internal gears and splines
     "multiple times faster" than shaping; reported 6 to 12 times shaping's productivity and 1.6
     to 3.3 times hobbing's
     ([Stadtfeld, Gear Technology 2014](https://ik.imagekit.io/agmamedia/gt/issues/0114x/power-skiving.pdf)).
   - *Gear milling on 5-axis machines* (InvoMilling-style disk cutters with standard inserts,
     where "the tool path, not the cutter, ... generate[s] the desired gear profile"
     ([Modern Machine Shop](https://mmsonline.com/articles/changing-the-landscape-of-gear-production))):
     small and medium batches on general-purpose machines. This is the process a general CAM
     program can own end to end.
   - *Form milling with indexing*: one tooth space at a time with a profile cutter; simple,
     slow, approximate unless the cutter matches the tooth count.
   - *Push/pull broaching* on a press: high-volume internal splines and keyways; CAM's job is
     broach selection and checks, not toolpaths.
   - *Stroke broaching (keyway shaping) on mills and lathes*: a non-rotating tool stroked along
     the bore with the spindle locked. A single edge, a Kienzle force per stroke, many strokes.
   - *Rotary (wobble) broaching*: hex and Torx sockets and small splines on a lathe or mill, with
     the tool set at 1° to the axis
     ([Production Machining, "Rotary Broaching 101"](https://www.productionmachining.com/articles/rotary-broaching-101)).
   - *Wire EDM*: any profile, prototypes and hardened parts.
5. **The physics reuses one model.** Every one of these is a set of edges removing chips of known
   thickness. The Kienzle and edge-force models in the cutting physics file give the force per
   tooth; what differs is how many teeth are in cut at once and how the chip thickness varies.
   For broaching that is a sum over teeth in the cut, with chip/gullet capacity and tooth
   strength as extra limits
   ([Köktürk, Sabanci University 2004](https://research.sabanciuniv.edu/id/eprint/8226/1/kokturkutku.pdf)).
   Worked examples below give a keyway broach at about 5.8 kN and a single stroke-shaping edge
   at about 1.3 kN (derived).
6. **Inspection is designed in, not bolted on.** For every gear and spline the model can compute
   the nominal over-pins (or over-balls) dimension and the span (base tangent length), the
   numbers a machinist checks at the machine. Worked example: module 2, 24 teeth, 20°, 3.5 mm
   pins gives M = 52.951 mm; the 3-tooth span is W = 15.433 mm (derived). Splines also need the
   *effective* (functional, gauge) size as well as the *actual* (measured) size.
7. **Quality grades are a known trap.** AGMA 2000's Q numbers rise with accuracy; ISO 1328-1:2013
   classes 1 to 11 and AGMA 2015's A-classes fall with accuracy (higher number = coarser). A
   model and drawing must store the system with the grade, never a bare number.
8. **What users need, in short:** catalogue-driven Gear/Spline/Keyway/Broached Profile features
   that store intent, generated true geometry with cosmetic display for speed, gear data tables
   on drawings, design-for-process checks (runout grooves, pilot holes, cutter overtravel), CAM
   that proposes a process from the feature, the machine and the batch size, and inspection
   numbers computed from the same data.

## Fundamentals

### Involute geometry in one page (textbook)

A gear tooth flank is an involute of the **base circle**: the curve traced by the end of a taut
string unwound from it. Two involutes in mesh keep a constant ratio and a constant line of
action even when the centre distance moves slightly, which is why almost all power gears and
most splines use it.

For module `m` (mm), tooth count `z`, pressure angle `α` (radians), helix angle `β` (0 for spur):

| Quantity | Formula |
|---|---|
| Reference (pitch) diameter | `d = m·z / cos β` (transverse; `m·z` for spur) |
| Base diameter | `db = d·cos αt`, where `tan αt = tan α / cos β` |
| Tip diameter (no shift) | `da = d + 2·ha*·m`, with `ha* = 1` for ISO 53 |
| Root diameter (no shift) | `df = d − 2·hf*·m`, with `hf* = 1.25` for ISO 53 profiles A to C |
| Circular pitch | `p = π·m` |
| Involute function | `inv φ = tan φ − φ` |
| Tooth thickness at reference circle | `s = m·(π/2 + 2·x·tan α)` (profile shift `x`) |
| Minimum teeth without undercut | `z_min = 2·ha* / sin² α` ≈ 17.1 at 20° (derived) |

Inch practice uses **diametral pitch** `P` (teeth per inch of pitch diameter), with
`m = 25.4 / P`. Splines to ANSI B92.1 use a **fractional pitch** such as 16/32: the numerator is
the diametral pitch that sets the pitch diameter, the denominator the "stub pitch" that sets the
tooth depth, `Ps = 2P`
([Engineers Edge, ANSI B92.1 summary](https://www.engineersedge.com/gears/involute_spline_13649.htm)).

**Profile shift** `x` moves the generating rack away from (positive) or toward (negative) the
blank. It cures undercut on small pinions, balances tooth strength between gear and pinion, and
sets an exact centre distance. It is the parameter most often lost when a gear moves between
programs, because the outside diameter alone does not reveal it.

### Basic rack (ISO 53)

ISO 53 defines the tooth proportions of a straight-sided rack from which all generated gears
follow, as multiples of module, all with α = 20° and addendum `1.00 m`
([drivetrainhub notebook, Basic Rack](https://nbviewer.org/github/drivetrainhub/notebooks-gears/blob/master/tooling/Chapter%25201%2520-%2520Basic%2520Rack.ipynb);
[ISO 53:1998 catalogue entry](https://scc-ccn.ca/standardsdb/standards/8110707)):

| Profile | Dedendum `hf*` | Root radius `ρF*` | Typical use (unverified) |
|---|---|---|---|
| A | 1.25 | 0.38 | high-torque gears |
| B | 1.25 | 0.30 | general |
| C | 1.25 | 0.25 | general, deeper root clearance for hobs |
| D | 1.40 | 0.39 | gears that will be ground after heat treatment |

The rack is also the tool shape for hobs and for the generating motion of gear milling, so the
same table configures the CAD generator and the CAM tool.

### Gear types in scope

Spur and helical cylindrical gears (external and internal), racks, and their tool-generated
roots. Out of scope for a first pass, named so the data model leaves room: bevel and hypoid
gears (different standards and machines), worms and worm wheels, face gears, non-circular
gears, cycloidal profiles (clocks, some reducers).

### Splines

Splines transmit torque between a shaft and a hub. Three geometric families matter:

- **Involute splines**, the majority. Pressure angles 30°, 37.5° and 45°; modules 0.5 to 10 for
  30° and 37.5°, 0.25 to 2.5 for 45° in ISO 4156; flat or fillet root
  ([Roymech, ISO 4156 summary](https://roymech.org/Useful_Tables/Keyways/Splines_Inv.html)).
  - **ISO 4156** (metric module, side fit): designation example
    `INT/EXT 32z × 2,5m × 30R × 5H/5f ISO 4156` (32 teeth, module 2.5, 30° fillet root, tolerance
    class 5, fit H internal / f external). External fit deviations d, e, f, h; tolerance classes
    4 to 7.
  - **ANSI B92.1** (inch, fractional pitch): 30° flat root side fit, 30° flat root major diameter
    fit, 30° and 37.5° fillet root side fit, 45° fillet root side fit. Basic formulas: pitch
    diameter `D = N/P`, base diameter `Db = D·cos φ`, minimum effective space width `π/(2P)` for
    30°
    ([Engineers Edge](https://www.engineersedge.com/gears/involute_spline_13649.htm)).
  - **DIN 5480** ("Involute splines based on reference diameters"): the nominal size is the
    reference (approximately outside) diameter, so a shaft of 40 mm takes a W 40 spline whose
    profile shift is chosen to make the diameters come out round
    ([DIN 5480-2 catalogue entry](https://www.dinmedia.de/en/standard/din-5480-2/394645667)).
    Designation shape `W 40 × 2 × 18` (reference diameter × module × teeth) (unverified exact
    form). Common in European gearboxes and motor shafts.
- **Straight-sided (parallel-sided) splines**: ISO 14 light and medium series with 6, 8 or 10
  splines, SAE J499 in inch practice (unverified series detail). Centred on the minor or major
  diameter or on the sides.
- **Serrations**: fine-pitch triangular teeth (DIN 5481) for clamped connections such as levers
  on shafts (unverified).

**Fit vocabulary** that the model must hold: *side fit* (the flanks centre the hub, the common,
economical case) versus *major* or *minor diameter fit* (a diameter centres it, more accurate
but more costly); *effective* space width or tooth thickness (the size a perfect mating part
"feels", which includes index, profile and lead errors; checked with composite GO gauges) versus
*actual* (a single space measured with pins; checked with sector NO-GO gauges)
([Roymech](https://roymech.org/Useful_Tables/Keyways/Splines_Inv.html)).

### Keys and keyways

- **Parallel keys:** ANSI B17.1 (inch, square and rectangular keys sized by shaft diameter
  range) and DIN 6885-1 / ISO 773 (metric; form A round ends, form B square ends) (unverified
  form letters). Hub keyway width tolerance classes such as JS9 (normal), P9 (tight) and D10/C11
  (sliding) set the fit
  ([Production Machining, "Broaching on CNC Machines"](https://www.productionmachining.com/articles/broaching-on-cnc-machines)
  lists JS9, P9 and C11 keyways as broachable on CNC machines).
- **Woodruff keys:** ASME B17.2 (half-moon keys in a milled pocket) (unverified edition).
- A key table is a lookup: shaft diameter → key width × height, shaft keyseat depth `t1`, hub
  keyway depth `t2`, corner radii. This belongs in the same catalogue mechanism as the tapped
  hole table.

### Broached polygons and sockets

Internal hexes (screw sockets, wrench drives), Torx-style lobular sockets, squares, double-D
and polygon shaft-hub profiles (DIN 32711 P3G and DIN 32712 P4C (unverified numbers)). These
are usually made by rotary broaching (small, blind) or push broaching (through, high volume),
and their design must include the pilot hole that broaching requires.

## Making them: the processes

For each process: what moves, what it can make, what limits it, and what CAM has to compute.

### Hobbing

**Kinematics (textbook).** The hob is a worm with gashes that form cutting teeth. Hob and blank
rotate in a fixed ratio (an electronic gearbox on modern machines): one hob revolution advances
the blank by `g / z` revolutions, where `g` is the number of hob starts. The hob is swivelled
so its thread is parallel to the tooth: swivel angle `η = β ∓ γ0` (hob lead angle `γ0`; minus
for same hand). The hob feeds axially across the face width (axial feed `fa` in mm per
workpiece revolution); a helical gear adds a differential rotation to the blank.

**What it makes.** External spur and helical gears, splines, sprockets, serrations, worm wheels.
It cannot make internal gears and needs runout room at the end of the face (the hob's radius
sweeps past the last tooth), so gears tight against a shoulder go to shaping or skiving.

**Physics.** Each hob tooth cuts a small chip whose thickness varies along its edge; simulated
chips have a "three-flank" geometry, the leading flank in the protuberance region wears first
because its effective clearance is only about 2°, and real hobbing life was about 30 to 40 %
of single-tooth fly-cutter tests
([Klocke and Winkel, Gear Technology 2004](https://ik.imagekit.io/agmamedia/gt/issues/0104x/klocke.pdf)).
The same paper reports carbide hobs costing 2 to 3 times HSS, an edge radius optimum around 10 to
15 µm, and tool life given in metres of tooth cut per tooth. That last point matters for the
library: **hob life is naturally a "metres of gear face per edge" or "parts per sharpening"
number**, not minutes.

There is no closed-form chip thickness for hobbing; industrial practice uses empirical formulas
(Hoffmeister's maximum chip thickness is the classical one) or a penetration simulation (the WZL
"Sparta" approach in the Klocke paper). **Opinion:** SolidCraft should compute it by simulation
(the same envelope code as the CAD generator, sampled per hob tooth), and use Hoffmeister-style
empirical caps only as a cross-check once someone can source the formula from a public paper.

**Cycle time (derived).** Axial travel = face width `b` + approach `La` + overrun. For a spur
gear the approach is the geometric entry of the hob, `La ≈ √(h·(da0 − h))`, with `h` the tooth
depth and `da0` the hob outside diameter. Time `t = (b + La + overrun) / (fa · n0 · g / z)`.

*Worked example (derived):* module 2, z = 24, b = 20 mm, h = 4.5 mm, carbide hob Ø70 mm at
150 m/min (n0 ≈ 682 rpm), 1 start, fa = 2 mm/rev, 2 mm overrun:
`La ≈ √(4.5 × 65.5) ≈ 17.2 mm`, workpiece speed ≈ 682/24 ≈ 28.4 rpm,
`t ≈ 39.2 / (2 × 28.4) ≈ 0.69 min`. The approach is almost as long as the face: for narrow gears,
**approach dominates hobbing time**, which is why the library needs hob diameter and the CAM
needs to minimise approach.

### Shaping

**Kinematics (textbook).** A cutter shaped like a gear (a pinion with relief) reciprocates along
the work axis while cutter and work rotate in mesh. The cutter lifts off ("relief" or
"backoff") on the return stroke so it does not rub. Helical gears need a helical guide matching
the cutter helix (mechanical machines) or programmed rotation (CNC).

**What it makes.** Internal gears and splines, gears next to shoulders (the cutter only needs a
small overtravel groove), cluster gears, sector gears.

**Limits.** Speed is limited by the reciprocating mass (strokes per minute), only half of each
cycle cuts, and the cutter's own tooth count must be checked against the work for interference
and for the ability to cut the internal gear without trimming tips (the number of cutter teeth
versus internal gear teeth) (textbook).

**What CAM computes.** Stroke length = face width + approach + overtravel; strokes per minute
from cutting speed; radial infeed per pass; rotary feed per stroke; the **overtravel groove**
needed in the part (a design-for-process check that goes back to CAD).

### Power skiving

**Kinematics.** Cutter axis and work axis cross at a shaft angle `Σ`; both rotate in a fixed
ratio and the cutter feeds along the work axis. The cutting speed comes from the difference of
the two surface velocities, so "the shaft angle is the main factor in generating sufficient
cutting speed"
([Stadtfeld, Gear Technology 2014](https://ik.imagekit.io/agmamedia/gt/issues/0114x/power-skiving.pdf)).
For a spur gear the cutting speed is roughly the work surface speed times `tan Σ` (derived,
approximate): a Ø100 mm work gear at 1000 rpm with `Σ` = 20° gives about 114 m/min.

**What it makes.** Internal and external gears and splines, especially internal ring gears and
splines that would otherwise be shaped or broached; needs little runout.

**Data from the same source:** HSS cutters about 100 m/min wet, carbide stick blades about
300 m/min; "under-critical speed" skiving at 150 to 200 m/min with higher feed improved tool
life; carbide top rake above 4° chipped; dry skiving drew about 15 % less spindle power than
wet; machines need direct drives and a stiff electronic gearbox.

**Opinion:** skiving is where a general-purpose 5-axis or mill-turn machine with synchronised
spindles competes with dedicated gear machines, so it is the generating process most likely to
appear in a general CAM's machine library. Its interference check (cutter body versus work,
especially internal gears) is pure geometry SolidCraft can do.

### Gear milling on 5-axis and mill-turn machines

Two approaches exist:

1. **Generating with a disk cutter (InvoMilling-style).** A disk cutter with standard carbide
   inserts cuts the slot, then the root and protuberance, then turn-mills the lower and upper
   flank, so that one cutter covers a whole module range; the CoroMill 161 covers modules 2 to 4
   and other cutters in the series go to module 12
   ([Modern Machine Shop](https://mmsonline.com/articles/changing-the-landscape-of-gear-production)).
   The CAM, not the cutter, carries the gear geometry, which is why the vendor ships it as
   CAD/CAM software
   ([Gear Technology, InvoMilling software](https://www.geartechnology.com/sandvik-coromant-invomilling-cad-cam-software-offers-additional-five-axis-functionality)).
2. **Flank milling or point milling with end mills and ball mills** along the flank surface
   (any 5-axis CAM can do this for large modules, prototypes, bevel and non-standard gears).
   Scallop height and flank accuracy are set by stepover and tool radius; slow, but needs no
   special tool.

**Why it matters for SolidCraft:** this is the only gear process that is fully a toolpath
problem. A SolidCraft gear operation can use the Gear feature's stored intent (module, z, α,
β, x, root form) to generate the slot, root and flank passes directly, check them against the
cutting physics model per pass, and verify them in material-removal simulation.

### Form milling with indexing

A disk cutter shaped like one tooth space is fed across the blank, the blank indexes one
pitch, repeat. Classical inch practice uses a set of eight involute cutters per diametral pitch,
each covering a range of tooth counts (the "#1 to #8" cutter system) (textbook, unverified
range boundaries), so the profile is exact only at one tooth count in the range. On a CNC with a
4th axis this is the cheapest way to make a spur gear; the CAM must know the cutter's design
tooth count and warn when the error for the actual `z` exceeds the requested quality class.

### Push and pull broaching

**What it is.** A long tool with a series of teeth, each a little higher than the one before
(the **rise per tooth**), is pushed or pulled once through a bore or along a surface. Roughing,
semi-finishing and finishing sections; the finishing teeth have very small rise and set the
size. Internal splines, keyways, hexes, squares, and external profiles in high volume.

**What the CAM does.** No toolpath, but four jobs: pick a broach (or flag a custom one), check
the press force and stroke, check the part (pilot bore, length, wall stiffness, through or
relieved end), and produce the setup sheet. For keyways, standard keyway broaches work in
several passes with **shims** behind the broach in a guide bushing; each shim raises the next
pass (unverified shim-set detail).

**Physics.** From Köktürk's thesis
([Sabanci University 2004](https://research.sabanciuniv.edu/id/eprint/8226/1/kokturkutku.pdf)):

- Force per tooth is the edge-force model: `Ft = Ktc·b·t + Kte·b`, `Ff = Kfc·b·t + Kfe·b`
  (width `b`, uncut chip thickness `t` = rise per tooth).
- **Total force** is the sum over the teeth in contact, `m = ceil(w / p)` for cut length `w` and
  pitch `p`; for oblique teeth the entry and exit are partial, which smooths the force.
- **Chip space:** the chip curled into each gullet must fit, chip volume / gullet volume ≤ 0.35.
- Other limits: tooth stress (from an FE-fitted equation), power `P = Ft,total · v`, total tool
  length, maximum pitch.
- A later thesis optimises the broach to **level the cutting force** between sections because
  uneven forces cause chipping, tooth breakage and poor surface
  ([Ebrahimi Araghizad, Sabanci University 2018](https://research.sabanciuniv.edu/id/eprint/36614/)).

Typical rise per tooth from a public reference, roughing / finishing, inches: high-tensile
steel 0.0015 to 0.002 / 0.0005; medium-tensile steel 0.0025 to 0.005 / 0.0005; soft cast iron
0.006 to 0.010 / 0.0005; wrought aluminium 0.005 to 0.010 / 0.0010
([Engineers Edge, broaching](https://engineersedge.com/manufacturing/broaching_types.htm)).
In millimetres: about 0.04 to 0.05, 0.06 to 0.13, 0.15 to 0.25 and 0.13 to 0.25 roughing, and
0.013 to 0.025 finishing (derived conversion).

**Worked example (derived): keyway broach in medium carbon steel.** Hub keyway 8 mm wide
(DIN 6885 size for a shaft of roughly 22 to 30 mm, hub depth about 3.3 mm (unverified)), hub
length 40 mm, rise 0.06 mm, pitch 12 mm, using `kc1.1 = 1500 N/mm²` and `mc = 0.25` for a
medium carbon steel (the companion tool library table gives values of this order):

- `kc = 1500 × 0.06^−0.25 ≈ 3030 N/mm²`
- Force per tooth `Fc = kc · b · h ≈ 3030 × 8 × 0.06 ≈ 1450 N`
- Teeth in cut `ceil(40 / 12) = 4`, so cutting force ≈ **5.8 kN**, before any allowance for
  dulling (a factor of 1.2 to 1.5 is common practice (unverified)).
- At 6 m/min (0.1 m/s), power ≈ 0.58 kW.
- Cutting teeth needed for 3.3 mm: about 55, or about 660 mm of broach at 12 mm pitch: which is
  exactly why keyway broaches are short and used in passes with shims.
- Chip space per tooth: chip area 0.06 × 40 = 2.4 mm² per unit width; the gullet needs at least
  2.4 / 0.35 ≈ 6.9 mm² of section, which a 12 mm pitch tooth easily provides; at 0.15 mm rise
  in a 100 mm hub it would need 43 mm², which a 12 mm pitch does not, so **gullet capacity, not
  force, sets the rise for long hubs**.

The pitch also controls force ripple: with exactly 3.33 teeth in cut, the count alternates
between 3 and 4, so the force jumps by a quarter every few millimetres. Choosing a pitch that
keeps the teeth-in-cut count constant (or nearly) is a design rule a check can enforce.

### Stroke broaching (keyway shaping) on mills and lathes

**What it is.** A single-point slotting tool or an insert in a holder, not rotating, stroked
along the bore with the spindle oriented and locked (machining centre: Z strokes; lathe: Z
strokes with the C axis or main spindle locked and X infeed). After each cutting stroke the tool
retracts radially for the back stroke and infeeds for the next
([Production Machining, "Broaching on CNC Machines"](https://www.productionmachining.com/articles/broaching-on-cnc-machines)).

Process rules from that source: infeed per stroke equals depth of cut and falls as tensile
strength rises; raise the tool 0.4 mm off the groove on the back stroke; the groove end needs a
runout or undercut; for groove lengths over 4 × D internal coolant helps; broach "at the 12
o'clock position" for chip removal; even slight tool twist shifts centre height (holder issues
go to the tool holding file). The source also prints a force formula with a "broaching factor"
of 1.18 and a chip angle correction of 0.79 for steel, which reduces to the Kienzle form with
correction factors (reproduced in the source with what look like typographical errors, so not
copied here).

**Worked example (derived): same 8 mm × 3.3 mm keyway, stroked.** Infeed 0.05 mm per stroke,
full width cut: `kc ≈ 1500 × 0.05^−0.25 ≈ 3170 N/mm²`, force per stroke
`≈ 3170 × 8 × 0.05 ≈ 1.27 kN` along the stroke, plus a radial (feed) force pushing the tool out
of the cut, of the order of 30 to 50 % of that (unverified ratio for zero-rake slotting tools).
Strokes ≈ 3.3 / 0.05 = 66, plus two or three spring passes. **This force vector (≈1.3 kN axial,
≈0.4 to 0.6 kN radial, at the edge's offset from the spindle axis) is what the tool holding file
uses to check the spindle orient lock and holder stiffness**, and what the workholding file uses
to check that the part cannot be pushed axially in the chuck.

**Physics limits a CAM must check per stroke:** edge force (tool strength), holder and spindle
lock moment, part pushed in the workholding, chip evacuation (blind keyways need the runout
groove), and stroke speed limited by the machine's Z acceleration over a short stroke.

### Rotary (wobble) broaching

**What it is.** A broach shaped like the final socket is held in a holder that tilts it 1° to
the work axis; the tool is free to rotate in the holder, so friction makes it turn with the work
and only one corner of the profile cuts at a time, in a nibbling motion
([Production Machining, "Rotary Broaching 101"](https://www.productionmachining.com/articles/rotary-broaching-101)).
On a machining centre the holder spins and the part stays still; on a lathe the part spins.

Process data from the same source:

- **Pilot hole:** about 1 % over the hex across-flats (less for free-cutting, more for hard
  material), chamfered 60 to 90°, drilled **1.3 to 1.5 × the profile length** deep to hold the
  chips; oversize tolerances tabulated by diameter (for example 0.0025 to 0.005 in for 0.236 to
  0.394 in holes in mild steel).
- **Speed** 1,500 to 3,000 rpm; **feed** in mild steel 0.0012 to 0.0024 in/rev (about 0.03 to
  0.06 mm/rev), up to two or three times as machinability and thrust allow, but not more than
  0.03 × profile diameter per rev.
- **Centring** within 0.0008 in TIR (about 0.02 mm); poor centring gives spiralling, oversize and
  short tool life. Reversing the spindle halfway can reduce spiralling.
- **Force:** about 80 % less than form punching at the optimum feed (vendor claim).
- **Materials:** free-cutting materials up to about 850 N/mm² tensile broach without much tool
  life loss; titanium, cobalt and high-temperature alloys are harder.

**Thrust (derived, rough).** With the 1° tilt, the cutting edge touches only a short arc of
the profile at any moment, so the axial thrust is roughly `kc` × (the area cut per revolution
at the contact arc) and is much smaller than `kc` × (whole profile area ring × feed). Without a
public validated model, **opinion:** SolidCraft should give an upper bound (full ring area × feed
× `kc`), state it as an upper bound, and cap feed by the source's 0.03 × D rule and the machine's
thrust, rather than invent precision.

### Wire EDM

Any 2D profile, internal or external, in hardened material, with no cutting force and no tool
wear in the usual sense. Used for prototype gears and splines, internal splines in hardened
parts, and broach and gauge making. The CAM needs wire diameter, spark gap offset, number of skim
passes and corner strategy (unverified detail; wire EDM CAM deserves its own research topic). For
gears it is attractive in SolidCraft because the toolpath is just the generated profile offset,
which the Gear feature already has.

### Finishing processes (named only)

Shaving, honing, profile and generating grinding after heat treatment. They depend on the stock
allowance left by the cutting process (ISO 53 profile D exists for this), so the gear feature
must store **"finished" versus "pre-grind" geometry** even if SolidCraft never generates grinding
paths.

### Process selection (what CAM can propose)

| Feature | Low volume, general machines | Medium volume | High volume | Hardened or prototype |
|---|---|---|---|---|
| External spur/helical gear, free ends | 5-axis gear milling; form milling + 4th axis | hobbing | hobbing | wire EDM (spur), grinding |
| External gear next to a shoulder | 5-axis gear milling | shaping, skiving | shaping, skiving | wire EDM (spur only) |
| Internal gear or spline | stroke shaping on a mill (spur only); wire EDM | shaping, skiving | push/pull broach, skiving | wire EDM |
| Keyway in a hub | stroke broaching on mill/lathe; keyway broach + shims on a press | same | push broach | wire EDM |
| Hex/Torx socket, blind | rotary broach | rotary broach | rotary broach, punch | wire EDM, sinker EDM |
| External spline on a shaft | form milling + 4th axis; 5-axis | hobbing | hobbing, rolling | grinding |

(Compiled from the sources above; **opinion** on the boundaries.) The CAM can make this a
proposal, not a rule: read the feature (internal/external, distance to shoulders, through or
blind), the machine library (does a machine have synchronised spindles, C axis, spindle lock,
Y axis), the tool library (which hobs, broaches, keyway inserts exist), and the batch size, and
list the options with estimated cycle time and the binding limit, in the CAM Physics System's
"explain every number" style.

## Modelling them in CAD

### Proposed features

1. **Gear** (cylindrical): designation fields `module` or `diametral_pitch`, `teeth`,
   `pressure_angle`, `helix_angle` and `hand`, `profile_shift`, `face_width`,
   `basic_rack` (ISO 53 A/B/C/D or custom `ha*`, `hf*`, `ρF*`), `generating_tool`
   (rack/hob, or shaper cutter with its own tooth count and shift), `tooth_thickness_allowance`
   or backlash allowance, `tip_chamfer`/`tip_relief`, `accuracy` (system + grade, for example
   `ISO 1328-1:2013 class 7`), internal or external. Output: the toothed solid or a cut into an
   existing body.
2. **Gear pair** (an assembly relationship): computes working pressure angle and centre
   distance from both gears' shifts, sets the `joint.gear` ratio from `z1/z2` exactly instead of
   from radii, and checks contact ratio and tip interference.
3. **Spline**: catalogue designations (ISO 4156, ANSI B92.1, DIN 5480, ISO 14 straight-sided),
   parsed the way the Hole command parses `1/4-20 UNC`; internal or external; fit class; root
   form; effective and actual limits stored for inspection.
4. **Keyway / keyseat**: shaft diameter → key from ANSI B17.1 or DIN 6885 table; shaft or hub;
   end form (milled round end, sled runner, broached through, stroke-broached into a runout
   groove); width tolerance class (JS9, P9, D10/C11).
5. **Broached profile**: hex, Torx-style lobular, square, double-D, polygon (P3G/P4C), with the
   **pilot hole** as a child feature whose size and depth come from the process rules above.

All five follow the thread catalogue pattern already in SolidCraft: a verified subset of each
standard, stated as a subset, internal units mm, inch sizes converted exactly
(`docs/thread-catalogue.md` does this for threads).

### Generating the geometry

**Opinion, with reasons:** generate tooth flanks by **envelope of the generating tool**, not by
drawing an involute and adding a fillet arc.

- The involute flank and the trochoidal root both fall out of one algorithm: sweep the tool
  profile (ISO 53 rack, or a shaper cutter) through the generating motion, intersect with the
  transverse plane, keep the region never touched.
- Undercut, profile shift, protuberance (pre-grind) and tip chamfer come for free, and match
  what the shop will produce.
- The same code is the CAM's 2D generating simulation for hobbing, shaping and skiving (one
  algorithm, two uses, consistent with the CAM Physics System's "one engine" rule).

Geometry representation: the involute is not a NURBS curve, so the B-rep holds a fitted spline
within a stated tolerance (for example 0.1 µm, far below ISO 1328 class 1 profile tolerances)
(derived). Store the **analytic intent** (the designation fields) as the source of truth and the
fitted geometry as derived, so inspection and CAM never measure the approximation. Helical teeth
are swept along a helix (a helicoid flank); helical B-rep surfaces should be checked against the
kernel's sweep accuracy.

**Performance:** a 100-tooth gear has several hundred faces. For assemblies, a cosmetic display
(pitch and root cylinders plus a tooth pattern drawn on them, like the existing cosmetic thread
bands) keeps the viewport light, with true teeth generated for export, mass properties, drawing
detail views and CAM.

### Drawings

Gear drawings carry a **gear data table** (module, teeth, pressure angle, helix and hand,
profile shift, accuracy grade with system, tooth thickness or over-pins dimension with pin size,
span with number of teeth, mating gear) rather than dimensioning the involute
(unverified reference to ISO 1340/ ANSI Y14.7.1 drawing conventions). Splines carry their
designation and fit. These tables can be generated from the stored intent, the same way the
Hole command produces thread callouts.

### Design-for-process checks (CAD side)

From the processes above, checks a design can run before it reaches CAM:

| Check | Rule (source) |
|---|---|
| Undercut | `z < z_min` without positive shift (textbook) |
| Hob runout | free face length beyond the gear ≥ hob approach `√(h·(da0 − h))` (derived) |
| Shaping overtravel | groove beyond the face for cutter overtravel (textbook) |
| Stroke-broached keyway, blind | runout groove or undercut at the end (Production Machining) |
| Rotary-broached socket | pilot hole about 1 % over across-flats, depth 1.3 to 1.5 × profile length, 60 to 90° chamfer (Production Machining) |
| Push broach | through bore, or a relief at the exit; hub length within broach and press stroke (textbook) |
| Spline fit | internal and external fit classes compatible; effective limits stored (ISO 4156 via Roymech) |
| Quality grade | grade stored with its system; warn when grade and process do not match (opinion) |

## Physics summary and interfaces

All processes are clients of the companion cutting physics model:

| Process | Force per edge | Edges in cut | Extra limits | Output to other files |
|---|---|---|---|---|
| Hobbing | Kienzle on simulated chip | several hob teeth, varying | hob life in metres per tooth; machine synchronisation | radial and tangential force on the work → workholding (arbor/collet) |
| Shaping | Kienzle per stroke | 1 to a few cutter teeth | strokes/min, relief, interference | axial and radial force → workholding; stroke speed → machine library |
| Skiving | Kienzle on simulated chip | several blades | shaft angle, blade top rake ≤ ~4° carbide (Stadtfeld) | force on synchronised spindles → machine library |
| 5-axis gear milling | Kienzle per insert, standard milling model | as milling | same as milling; profile from toolpath | standard milling outputs |
| Form milling | as milling | as milling | profile error vs cutter design `z` | standard |
| Push/pull broach | edge-force model per tooth (Köktürk) | `ceil(w/p)` | gullet ≤ 0.35, tooth stress, press force and stroke | total pull/push force → press and fixture |
| Stroke broaching | Kienzle per stroke | 1 | infeed by tensile strength, back-stroke lift 0.4 mm | **force vector + alignment tolerances → tool holding file**; axial push → workholding |
| Rotary broaching | upper bound only (opinion) | 1 corner at a time | feed ≤ 0.03 × D/rev, centring ≤ 0.02 mm TIR | thrust → machine library; centring → tool holding |
| Wire EDM | none (no contact) | n/a | wire, gap, skims | none |

Every limit above is a "link" in the CAM Physics System's min-over-links solver, and the
conservative-to-aggressive slider moves the soft margins (rise per tooth, infeed per stroke,
hob feed) while hard limits (press capacity, spindle lock torque, gullet capacity, centring
tolerance) do not move.

## Inspection

### Numbers the model can compute for every gear (textbook)

**Over pins (or balls).** Pin diameter `dp` chosen so the pin touches near the reference circle
(about 1.7 × module for 20° spur gears (unverified), then rounded to an available pin). For an
external spur gear:

```
inv φ = inv α + dp / db − π / (2z) + 2·x·tan α / z
M = db / cos φ + dp                       (even z)
M = db·cos(π / (2z)) / cos φ + dp         (odd z)
```

(The shift term follows from the tooth thickness formula above; derived.) Pins sit in opposite
spaces for even counts and in the nearest-to-opposite spaces for odd counts
([KHK, over-pin measurement](https://www.khkgears.us/media/1276/how-to-calculate-the-nominal-values-of-over-pin-or-ball-measurement-of-teeth-for-various-types-of-gearing.pdf)).
Internal gears and splines use the same idea between pins, and helical gears use balls with a
transverse correction (not reproduced here).

*Worked example (derived):* m = 2, z = 24, α = 20°, x = 0, dp = 3.5 mm: `db = 45.105`,
`inv α = 0.014904`, `inv φ = 0.027051`, `φ = 24.20°`, **M = 52.951 mm**. The same gear with
25 teeth: M = 54.854 mm.

**Span (base tangent length)** over `k` teeth, measured with disc micrometers, independent of
the outside diameter:

```
k ≈ z·α/π + 0.5   (rounded)
W = m·cos α·[π·(k − 0.5) + z·inv α] + 2·x·m·sin α
```

*Worked example (derived):* same gear, k = 3, **W = 15.433 mm**
([excelcalcs, base tangent length](https://excelcalcs.com/calcs/repository/Machines/Gears/Base-Tangent-Length-Calculation-for-Helical-Gears)
and [MESYS helical gear calculation](https://www.mesys.ch/calc/helicalgear.fcgi?lang=en)
cover the helical form).

Tooth thickness tolerance converts to over-pins and span tolerance; the model should output
both nominal and limits, with pin size, on the drawing and in the inspection plan.

### Gear lab and shop checks

- **Analytical** (CNC gear measuring machine or CMM with gear software): profile (root to tip),
  helix/lead, pitch (single and cumulative), runout
  ([Gear Solutions, gear inspection](https://gearsolutions.com/?p=24201)).
- **Double-flank composite**: roll against a spring-loaded master with no backlash; reports
  total and tooth-to-tooth radial composite deviation; catches size, runout and nicks but cannot
  separate flank errors (same source; ISO 1328-2 covers radial composite tolerances,
  [ISO 1328-2:2020 entry](https://www.boutique.afnor.org/en-gb/standard/iso-132822020/cylindrical-gears-iso-system-of-flank-tolerance-classification-part-2-defin/xs134034/329713)).
- **Single-flank**: roll at working centre distance, measure transmission error (same source).
- **Splines**: composite GO gauge for effective size, sector NO-GO gauge or pins for actual size.
- **Keyways**: width (gauge block or GO/NO-GO), depth from the opposite side of the bore,
  symmetry to the bore axis.

### On-machine and in-process (link to the in-process inspection file)

- Probing two opposite tooth spaces with a ball stylus of diameter `dp` gives an on-machine
  equivalent of the over-pins dimension; the M formula above becomes the nominal for a probe
  cycle (derived). The in-process inspection file's closed loop can then adjust radial infeed
  for the next part or the finishing pass.
- Keyway position and width can be probed directly; spline effective size cannot (it needs a
  composite gauge or a full analytical check), so the loop must say what it can and cannot see.
- Broach wear shows up as growing pull force at constant rise: a force or spindle-load trend per
  part is the cheapest wear sensor (opinion; ties to the machine health file).

## Library and simulation integration

### Tool library fields (additions to the companion tool library schema)

| Tool type | Fields |
|---|---|
| Hob | module/DP, pressure angle, basic rack (incl. protuberance, pre-grind), starts `g`, gashes, outside diameter `da0`, usable length, hand, lead angle `γ0`, accuracy class (DIN 3968 AA/A/B (unverified)), substrate and coating, life in metres per tooth |
| Shaper cutter | module, α, teeth `z0`, profile shift `x0`, type (disk, shank, hub), helix and hand, length of stroke allowed |
| Skiving cutter | module, α, teeth or blades, helix, rake and relief, solid or stick-blade |
| Gear milling disk cutter | insert geometry, module range covered, max slot depth, profile root radius |
| Involute form cutter | module/DP, α, design tooth-count range (for example "#3: 35 to 54 teeth" (unverified)) |
| Push/pull broach | profile, sections with rise per tooth, pitch, tooth count per section, gullet depth/area, total length, max pull or push force, guide bushing and shim set (keyway) |
| Keyway/slot insert | width, max depth, reach, minimum bore, rake, number of edges |
| Rotary broach | form and size, across flats/corners, length of cut, holder wobble angle |
| Wire | diameter, material, spark gap per pass |

### Holder library (owned by the tool holding file)

Keyway holders, rotary broach holders (wobble angle, centring adjustment) and their alignment
tolerances, with the process force vector from this file as the load case.

### Machine library capabilities

Synchronised axes or electronic gearbox (hobbing/skiving/gear-milling cycles), C axis indexing
resolution, spindle orient and lock holding torque (stroke broaching), Z axis acceleration and
max stroke rate, Y axis on lathes, max axial thrust (rotary broaching, drilling), broach press
capacity and stroke. These go into the machine library file's capability list, so the process
proposal can say "this machine cannot skive: no synchronised second spindle".

### Simulation

- **2D generating simulation** in the transverse plane for spur gears, extended slice by slice
  for helical: the envelope engine from CAD, run with the CAM's actual tool and feeds, gives the
  produced profile, root, feed scallops ("generating flats") and undercut, and compares them to
  the accuracy class (opinion: cheaper and more accurate than general voxel/dexel simulation for
  gears).
- **General material removal** (dexel or similar) for gear milling, stroke broaching and form
  milling, which are ordinary toolpaths.
- **Collision and interference**: hob or shaper or skiving cutter body versus shoulders,
  cutter-to-internal-gear tip interference, keyway tool body in small bores, rotary broach holder
  versus part.
- **Force trace**: per-tooth (broach) or per-stroke force over the cut, so the user sees the
  force ripple the pitch causes and the binding limit, in the CAM Physics System's
  explain-every-number display.

## Standards and references

Described from public catalogue entries and secondary summaries only.

| Standard | Covers |
|---|---|
| ISO 53:1998 | basic rack profiles A to D for cylindrical gears |
| ISO 54 | modules (preferred series) (unverified number) |
| ISO 1328-1:2013 | flank tolerance classes 1 to 11 (pitch, profile, helix) ([catalogue](https://www.sis.se/en/produkter/mechanical-systems-and-components-for-general-use/gears/iso132812013/)) |
| ISO 1328-2:2020 | radial composite and runout tolerances |
| AGMA 2015-1 / AGMA 2000 | US accuracy systems (A-classes, falling with accuracy; Q numbers, rising with accuracy) (unverified edition detail) |
| ISO 6336 / AGMA 2001 | load capacity (strength and pitting); a gear *design* topic, not CAM (named only) |
| ISO 4156-1..3 | metric involute splines, side fit |
| ANSI B92.1 (SAE) | inch involute splines, fractional pitch |
| DIN 5480-1..2 | involute splines on reference diameters |
| ISO 14 | straight-sided splines (unverified series detail) |
| DIN 5481 | serrations (unverified) |
| ANSI B17.1, ASME B17.2 | keys and keyseats; Woodruff keys |
| DIN 6885-1, ISO 773 | parallel keys (metric) |
| DIN 32711, DIN 32712 | polygon shaft-hub profiles P3G, P4C (unverified) |
| DIN 3968 | hob tolerances (unverified) |

## Accuracy

- **Approximate profiles.** Gears drawn with circular arcs instead of involutes, or with an
  arbitrary root arc, look right and are wrong; they mesh badly in motion simulation and do not
  match any cutter. The generating approach removes this class of error (opinion based on the
  geometry).
- **Fitted spline tolerance.** The B-rep can only approximate an involute; the stored analytic
  intent must win for inspection and CAM (derived).
- **Profile shift lost on exchange.** STEP files carry geometry, not intent; a gear imported
  from STEP must be recognised (fit `db`, `z`, `x` from the faces) or treated as plain geometry.
  Feature recognition is possible because `db` and `z` are recoverable from any two flanks
  (derived).
- **Grade systems reversed.** AGMA Q numbers versus ISO and AGMA A-classes (see Summary).
- **Inch/metric.** DP to module is exact (`m = 25.4/P`); an ANSI B92.1 spline is **not** an
  ISO 4156 spline with the same pitch diameter; tooth depth (stub pitch) and fits differ.
- **Over-pins sensitivity.** M changes about 2 to 3 times faster than tooth thickness (derived
  from the formulas above: `dM/ds = cos α / sin φ` for even counts, about 2.3 in the worked
  example), so a 10 µm thickness tolerance is roughly a 23 µm over-pins tolerance, which is why
  pin size must be stated with the number.
- **Process numbers here are starting points.** Rise per tooth, infeed per stroke and rotary
  broach feeds are published starting ranges, to be calibrated by the in-process loop.

## Programs

From public pages only; no program was run.

- **CAD gear and spline generators** are common as add-ins or design accelerators in commercial
  CAD; one documents ANSI spline tolerance dialogs
  ([Autodesk Inventor help, ANSI spline tolerances](https://help.autodesk.com/cloudhelp/2014/ENU/Inventor/files/GUID-3610CA28-270F-461E-97D9-D71D22EA4F5E.htm)).
  How faithfully they generate roots varies and is not stated on public pages (unverified).
- **Gear calculation software** (for example MESYS's public online helical gear calculator) does
  the geometry, measurement dimensions and strength, but not CAD or CAM
  ([MESYS](https://www.mesys.ch/calc/helicalgear.fcgi?lang=en)).
- **Tool-maker CAM**: Sandvik Coromant ships InvoMilling as CAD/CAM software for its gear milling
  method on 5-axis machines
  ([Gear Solutions](https://gearsolutions.com/?p=32799)).
- **Machine-builder software**: dedicated hobbing, shaping and skiving machines carry their own
  technology software (settings, infeed strategy, interference checks), per Stadtfeld's article.
  General CAM rarely covers generating processes (unverified, **opinion**: this is the gap).
- **Open source:** several gear generators exist as OpenSCAD or Python libraries (for example
  [python-geargen](https://github.com/rocketcollider/python-geargen),
  [involute gear collection](https://github.com/koppi/involute-gear-collection)); their licences
  vary and their code was not read for this file. A SolidCraft implementation should start from
  the textbook equations here, not from any of them.

## Complaints and requests

Patterns reported in forums and articles (unverified as a survey; **opinion** on weight):

- Generated gears with approximate profiles that do not mesh correctly in motion studies or do
  not match the shop's hob.
- No way to state profile shift, or it is lost on STEP export.
- Heavy models: a few large gears slow an assembly.
- Drawings without a gear data table, so the machinist recomputes over-pins and picks a pin.
- Splines treated as gears: no effective/actual limits, no fit classes.
- Keyways drawn by hand with the wrong table or no tolerance class.
- CAM cannot make gears on general machines without vendor software; broach and keyway
  operations are "manual macro" territory with no force check.

## What users need

In priority order (**opinion**):

1. **Catalogue features that store intent**: Gear, Spline, Keyway, Broached Profile, built like
   the existing thread catalogue, with exact designations and verified subsets.
2. **Generated true geometry** by envelope of the generating tool, with a cosmetic display
   option for assemblies.
3. **Gear pair** relationship that drives `joint.gear` from tooth counts and checks centre
   distance, contact ratio and interference.
4. **Drawing data tables** and **inspection numbers** (over-pins with pin size, span, spline
   effective/actual limits) generated from the intent.
5. **Design-for-process checks** (undercut, runout, overtravel, pilot hole, through or relieved
   end).
6. **CAM process proposal** from feature + machine + tools + batch, with cycle time and the
   binding limit explained.
7. **CAM operations** in this order of difficulty: stroke broaching and rotary broaching (simple
   motion, high shop value), form milling with a 4th axis, 5-axis gear milling, then
   synchronised processes (hobbing, skiving, shaping) when machine support exists.
8. **Physics checks** for each through the shared limits engine, and a force trace for broaching.
9. **Closed loop**: probe-based over-pins, wear trend from force, feeding the in-process
   inspection file's loop.

## Glossary

- **Base circle**: the circle an involute unwinds from; `db = d·cos α`.
- **Backlash**: free play between mating teeth, set by tooth thickness allowances and centre
  distance.
- **Broach**: a multi-tooth tool whose teeth rise one after another; one pass makes the shape.
- **Diametral pitch (DP)**: teeth per inch of pitch diameter; `m = 25.4/DP`.
- **Effective size (splines)**: the size a perfect mating part sees, including form errors.
- **Envelope (generating)**: the region a moving tool never touches; it is the generated tooth.
- **Gullet**: the space in front of a broach tooth that carries the chip.
- **Hob**: a gashed worm-shaped generating cutter.
- **Involute**: the flank curve of most gears and splines.
- **Module (m)**: reference diameter divided by tooth count, in mm.
- **Over pins (M)**: the dimension across pins placed in opposite tooth spaces.
- **Profile shift (x)**: offset of the generating rack, in modules.
- **Rise per tooth**: the step in height between successive broach teeth; the chip thickness.
- **Rotary (wobble) broaching**: broaching with a tool tilted about 1° so one corner cuts at a
  time.
- **Skiving (power skiving)**: continuous generating with crossed tool and work axes.
- **Span (base tangent length, W)**: the dimension across `k` teeth measured with disc
  micrometers.
- **Stroke broaching / keyway shaping**: reciprocating a non-rotating single edge along a bore on
  a CNC machine.
- **Trochoid (root)**: the curve the tool tip traces in the root; it forms the root fillet.

## Open questions

1. Should the Gear feature be a native feature or a plugin (the plugin design already names
   `"gear_generator"`)? **Opinion:** a native feature for spur/helical gears and splines, because
   CAM, inspection, drawings and joints all need its intent; plugins for exotic types.
2. Which subsets of ISO 4156, ANSI B92.1 and DIN 5480 should ship first, and who verifies them
   against the purchased standards?
3. How should a gear imported from STEP be recognised (fit `db`, `z`, `x`), and how confident
   must the fit be before it is promoted to a Gear feature?
4. What fitted-curve tolerance should the involute use, and does the kernel's helical sweep meet
   it?
5. Which machines in the machine library support synchronised hobbing or skiving cycles, and
   how is that described in a vendor-neutral way?
6. Is there a public, validated rotary broaching thrust model? Without one, the upper bound and
   feed cap are the honest answer.
7. Should broach design (choosing rise, pitch and sections for a custom broach) be in scope, or
   only checking catalogue broaches?
8. What is a reasonable public source for hob chip thickness formulas (Hoffmeister), so a check
   can be added beside the simulation?

## Sources

All accessed 2026-10-26.

- Stadtfeld, H. J., "Power Skiving of Cylindrical Gears on Different Machine Platforms", Gear
  Technology, Jan/Feb 2014. https://ik.imagekit.io/agmamedia/gt/issues/0114x/power-skiving.pdf
- Klocke, F. and Winkel, O., "New Potentials in Carbide Hobbing", Gear Technology, Jan/Feb 2004.
  https://ik.imagekit.io/agmamedia/gt/issues/0104x/klocke.pdf
- Köktürk, U., *Optimization of Broaching Tool Design*, M.Sc. thesis, Sabanci University, 2004.
  https://research.sabanciuniv.edu/id/eprint/8226/1/kokturkutku.pdf
- Ebrahimi Araghizad, A., *Geometrical optimization of the broaching tools by leveling of the
  cutting forces*, M.Sc. thesis, Sabanci University, 2018.
  https://research.sabanciuniv.edu/id/eprint/36614/
- Production Machining, "Rotary Broaching 101".
  https://www.productionmachining.com/articles/rotary-broaching-101
- Production Machining, "Broaching on CNC Machines".
  https://www.productionmachining.com/articles/broaching-on-cnc-machines
- Modern Machine Shop, "Changing the Landscape of Gear Production".
  https://mmsonline.com/articles/changing-the-landscape-of-gear-production
- Gear Technology, "Sandvik Coromant InvoMilling CAD/CAM software offers additional five-axis
  functionality".
  https://www.geartechnology.com/sandvik-coromant-invomilling-cad-cam-software-offers-additional-five-axis-functionality
- Gear Solutions, Sandvik Coromant software for five-axis machining centres.
  https://gearsolutions.com/?p=32799
- Gear Solutions, gear inspection methods. https://gearsolutions.com/?p=24201
- KHK Gears, "How to calculate the nominal values of over pin or ball measurement".
  https://www.khkgears.us/media/1276/how-to-calculate-the-nominal-values-of-over-pin-or-ball-measurement-of-teeth-for-various-types-of-gearing.pdf
- Engineers Edge, involute spline (ANSI B92.1) formulas.
  https://www.engineersedge.com/gears/involute_spline_13649.htm
- Engineers Edge, broaching types and depth of cut per tooth.
  https://engineersedge.com/manufacturing/broaching_types.htm
- Roymech, involute splines (ISO 4156). https://roymech.org/Useful_Tables/Keyways/Splines_Inv.html
- Glinsky, C., drivetrainhub gear notebooks, "Basic Rack".
  https://nbviewer.org/github/drivetrainhub/notebooks-gears/blob/master/tooling/Chapter%25201%2520-%2520Basic%2520Rack.ipynb
- ISO 53:1998 catalogue entry. https://scc-ccn.ca/standardsdb/standards/8110707
- ISO 1328-1:2013 catalogue entry.
  https://www.sis.se/en/produkter/mechanical-systems-and-components-for-general-use/gears/iso132812013/
- ISO 1328-2:2020 catalogue entry.
  https://www.boutique.afnor.org/en-gb/standard/iso-132822020/cylindrical-gears-iso-system-of-flank-tolerance-classification-part-2-defin/xs134034/329713
- DIN 5480-2 catalogue entry. https://www.dinmedia.de/en/standard/din-5480-2/394645667
- Autodesk Inventor help, Tolerances (for ANSI standard) dialog box.
  https://help.autodesk.com/cloudhelp/2014/ENU/Inventor/files/GUID-3610CA28-270F-461E-97D9-D71D22EA4F5E.htm
- MESYS, helical gear calculation. https://www.mesys.ch/calc/helicalgear.fcgi?lang=en
- excelcalcs, base tangent length calculation for helical gears.
  https://excelcalcs.com/calcs/repository/Machines/Gears/Base-Tangent-Length-Calculation-for-Helical-Gears
- SolidCraft repository: `docs/thread-catalogue.md`, `docs/cosmetic-threads.md`,
  `docs/plugins.md`, `skills/solidcraft-mechanisms/SKILL.md` (gear and rack joints).
