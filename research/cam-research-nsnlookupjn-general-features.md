# CAM research: general feature requests from the shop floor

Research for [#1621](https://github.com/tangency-labs/solidcraft/issues/1621) (Research Request
for Plugin: CAM). This file collects **ten practical CAM features** that each solve a problem
machinists meet every week. They are smaller than the physics topics in the companion files, but
each one leans on the same libraries and the same simulation gate:

1. [Finish-first planning](#1-finish-first-planning): decide the finishing toolpaths first, then
   let adaptive roughing work out how to remove everything else.
2. [Angle heads in a standard tool changer](#2-angle-heads-in-a-standard-tool-changer): 90° and
   adjustable heads that turn a 3-axis mill into a "side-cutting" machine.
3. [Grinding on a 3- or 5-axis mill](#3-grinding-on-a-3--or-5-axis-mill).
4. [Burnishing](#4-burnishing): roller and ball burnishing for mirror-like bores and shafts
   without removing material.
5. [Flexible hones and abrasive brushes](#5-flexible-hones-and-abrasive-brushes): ball hones for
   bores, abrasive-filament brushes for surface finish and edge breaking, as real operations.
6. [Turning on a mill's 4th axis](#6-turning-on-a-mills-4th-axis).
7. [Stock-aware rapids](#7-stock-aware-rapids-instead-of-clearance-planes): use the 3D model of
   the remaining stock to move fast and close, instead of retracting to a fixed plane.
8. [Global arc fitting to G2/G3](#8-global-arc-fitting-to-g2g3): one switch to smooth a whole
   program into arcs, for smaller files, faster execution on older controls, and smoother motion.
9. [Serial numbers and live text fields in engraving](#9-serial-numbers-and-live-text-fields-in-engraving).
10. [Tool-setter placement and a safe length find](#10-tool-setter-placement-and-a-safe-length-find):
    put the setter where it costs least time in each setup, and measure a tool without the
    operator typing in a rough length.

Related files in the same fork (this file links to them instead of repeating them):

- [CAM Physics System overview](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-vgqo3g/research/cam-research-nsnlookupjn-physics-system.md):
  the shared setup, the "lowest limit wins" solver, the one conservative-to-aggressive slider,
  and the **simulation gate** (a program cannot be posted while a *Stop* finding is open). Every
  section below ends with what that gate checks for the feature.
- [Shared cloud tool library](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-tool-library/research/cam-research-nsnlookupjn-tool-library.md):
  tool, holder and assembly records. This file adds a few **new tool classes** (angle head,
  grinding wheel or mounted point, burnishing tool, flexible hone, abrasive brush, static turning
  tool for a mill) and the fields they need.
- [Cutting tool physics](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-cutting-physics/research/cam-research-nsnlookupjn-cutting-physics.md):
  cutting force and torque, used here for angle heads and 4th-axis turning.
- [Tool holding physics](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-4aoliy/research/cam-research-nsnlookupjn-tool-holding.md):
  holder speed, balance, clamping and coolant limits. Angle heads and static turning holders are
  holders too; their rules extend that file.
- [Machine library](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-machine-library/research/cam-research-nsnlookupjn-machine-library.md):
  axis speeds and accelerations, ATC limits, rotary axes, and the control's block processing time.
  This file asks for a few new machine fields (stop block, rotary turning rating, setter mounts).
- [Workholding physics](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-workholding/research/cam-research-nsnlookupjn-workholding.md)
  and [stock structural integrity](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-stock-integrity/research/cam-research-nsnlookupjn-stock-integrity.md):
  the in-process stock model and its uncertainty, which stock-aware rapids rely on.
- [In-process inspection](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-y2w06n/research/cam-research-nsnlookupjn-in-process-inspection.md):
  owns **part marking methods and standards** (engraving, DataMatrix, dot peen, laser,
  MIL-STD-130). Section 9 here adds only the live text fields and the serial counter.
- [Cell automation](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-g08nas/research/cam-research-nsnlookupjn-cell-automation.md):
  owns the **tool-setting data flow** (presetter, tags, `G10` offset programs, check windows).
  Section 10 here adds setter placement and the search move when no length is known.
- Two topics in progress in the same set: **threading** (a physics-based threading module) and
  **CNC milling best practices** (chips, cleaning, lead-ins and lead-outs). This file does not
  cover them.

Licence: MIT OR Apache-2.0, the same as SolidCraft (see the Licence section of CONTRIBUTING.md).

## About this research

- **Author:** Jason ([@nsnlookupjn](https://github.com/nsnlookupjn)). The ten feature ideas are
  Jason's; source research and writing done with an AI assistant (Claude) in October 2026.
- **How it was produced:** web searches of public sources (tool and machine makers' public
  manuals, data sheets and blogs, trade press, practitioner forums), each claim checked against
  the cited page where possible. No commercial CAD/CAM program was driven or scripted, no
  GPL/LGPL/AGPL source was read, and no text is copied beyond short quotes. The worked examples
  are arithmetic computed for this file; where they rest on assumed values, the assumptions are
  stated.
- **Units:** millimetres, newtons, newton-metres, rev/min, metres per second or per minute.
  Inch sources are converted (1 in = 25.4 mm, 1 µin = 0.0254 µm, 1 in/min = 25.4 mm/min,
  1 ft·lb = 1.356 N·m).
- **Marking:** "(unverified)" marks a claim not confirmed against a source. **Fact**,
  **opinion**, **estimate** and **marketing** are labelled where they mix.
- **Shop experience:** sections marked **TODO (Jason)** are where Jason will add his own
  experience. Where his experience is already included, it is labelled "personal experience".

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

**This research is not complete.** It is a starting point: the author will keep researching
and adding to it, and others are encouraged to do the same, correct it and contribute their own
findings and shop experience.

## Summary

| # | Feature | The idea in one line | Biggest risk the gate must catch | New library data |
|---|---|---|---|---|
| 1 | Finish-first planning | Fix finishing tools and paths first; roughing removes "everything the finishers do not need" | Rest material left that a finisher cannot take in one pass | Finisher's maximum load per pass (from cutting physics) |
| 2 | Angle heads | Treat the head as a holder with a gear ratio, a reverse rotation and a stop block | Speed or torque over the head's rating; head spinning with no stop block | Angle head record; machine stop block position |
| 3 | Grinding on a mill | Mounted points and plated wheels as tools, with wheel speed as a hard limit | Wheel over its marked speed; abrasive swarf in the machine | Wheel record (max rpm, bond, grit); machine "grinding allowed" flag |
| 4 | Burnishing | Cold-work the surface to a fine finish with rolls or a ball | Wrong pre-machined size, material too hard | Burnishing tool record (range, interference, hardness limit) |
| 5 | Hones and brushes | Flexible hones and abrasive brushes as programmed operations with real parameters | Brush over its rated speed; hone in a bore it does not fit | Hone and brush records (rated rpm, oversize, grit, trim) |
| 6 | 4th-axis turning | The rotary axis spins the part, a fixed tool in the spindle cuts | Rotary axis not rated for continuous turning; spindle not locked | Rotary "turning rating"; static turning tool record |
| 7 | Stock-aware rapids | Rapid just above the remaining stock, not up to a plane | Rapid into stock that is larger than modelled | Stock uncertainty; machine rapid behaviour (dog-leg) |
| 8 | Global arc fitting | One switch, per machine default, fits G2/G3 to the whole program | Fit error stacking on toolpath tolerance | Control block time, arc formats, plane support |
| 9 | Serial engraving | Text with live fields `{serial}`, `{date}`, `{time}`… filled by the control or the edge device | Duplicate or skipped serials; text too shallow on a curved face | Counter source; control's macro variables |
| 10 | Tool setter | CAM chooses the setter mount per setup; the length find uses a safe search speed | Tool side hitting the setter body; search faster than the stylus overtravel allows | Setter mounts, overtravel, latency; assembly expected length |

**The request (opinion):** none of these needs a new engine. Each is a new **tool class**, a new
**operation**, or a new **link/post option**, sitting on the tool, holder and machine libraries,
and each adds a few checks to the simulation gate. That makes them good early contributions once
the 2.5D CAM core ([PR #4158](https://github.com/tangency-labs/solidcraft/pull/4158)) lands.

## Fundamentals

### 1. Finish-first planning

**Jason's idea:** consider the approach CloudNC uses: finalise the finishing toolpaths first, then
let adaptive clearing work out the best way to clear the rough material.

**Prior art.**

- **CloudNC CAM Assist** (an AI add-on for several commercial CAM programs) released "Next Gen
  Roughing" in December 2024. Its blog says the update combines bulk and detailed roughing into
  one operation, uses automatically generated boundaries, and considers "roughing as an overall
  problem, rather than splitting it up into distinct stages"
  ([CloudNC blog, 15 Jan 2025](https://www.cloudnc.com/blog/next-gen-roughing-a-major-cam-assist-update)).
  The public page does not describe the order in which it plans finishing and roughing; the
  "finish first" description is Jason's reading of how the product behaves (unverified against
  public documentation). CloudNC also markets "complete up to 80% of your CAM program in minutes"
  (**marketing**, same page).
- **Experienced programmers already think backwards** (opinion): pick the finishing tool for the
  smallest radius and the tightest tolerance first, then choose the stock-to-leave and the rest
  roughing tools so that the finisher sees an even, light load. Most CAM programs make you build
  operations in the other order, from big tool to small.
- **Process planning literature** calls this backward or "goal-driven" planning (the final
  features define what every earlier step must leave) (unverified as a named method).

**How it would work in SolidCraft (opinion):**

1. The user (or a DFM pass, see the DFM check file in the same set) fixes the **finishing
   operations**: tools, step-overs, and the tolerance each face needs.
2. Each finishing operation declares its **acceptable incoming stock**: the minimum and maximum
   material it can take in one pass, from the cutting physics (radial engagement limit, the
   deflection limit of that tool at that stick-out). For example, a 6 mm end mill finishing a
   wall might accept 0.10 to 0.30 mm radially (example values, not a recommendation).
3. CAM builds the **target envelope**: the part offset by each finisher's accepted stock, face
   by face. Faces with different finishers get different offsets.
4. Adaptive roughing and rest roughing are generated **to that envelope**, choosing tools from
   the tool library from largest down, until the remaining stock everywhere is inside the
   finisher's window.
5. Any place where the remaining stock is still above a finisher's maximum (a corner a roughing
   tool cannot reach) becomes a **finding**: add a rest operation, use a smaller roughing tool,
   or accept a slower finishing pass there.

**Physics and limits.** The finisher's window is not one number. A thin wall wants a small, even
stock so that it does not spring; a long tool wants less radial load; a corner with a small
radius sees more engagement than a straight wall at the same offset (the engagement-angle
increase in corners is covered in the cutting physics file). The stock-integrity file gives the
rule for thin walls: leave stiffness where the finisher still needs it.

**Uses the libraries.** Tool library (finishing and roughing tools), cutting physics (engagement
and deflection limits per tool), stock integrity (thin-wall order).

**Simulation gate.** *Stop*: remaining stock at any finishing contact above the finisher's
maximum. *Warn*: stock below the finisher's minimum (rubbing, poor finish). *Info*: total time
saved compared with the user's own roughing.

**TODO (Jason):** how you plan finishers first today, and where CloudNC's result differed from
what you would have programmed.

### 2. Angle heads in a standard tool changer

**Jason's idea:** support new styles of milling tooling, such as 90° milling heads that fit into
standard tool changers.

**What they are (fact).** An angle head is a gearbox in a tool holder. The input shank sits in
the spindle taper; the output spindle points sideways (90°), at a fixed angle (for example 45°),
or at an adjustable angle (universal heads). The housing must not turn with the spindle, so a
**locating pin** on the head drops into a **stop block** fixed to the spindle nose. On the BIG
DAISHOWA heads the stop block must sit at a fixed pitch from the spindle centre, the "S
dimension": 65 mm for #40 tapers (BBT40/HSK-A63) and 80 or 110 mm for #50 tapers. The
orientation of the pin relative to the drive keys can be set anywhere within 360°, but the
spindle orientation (M19) must stop the drive keys at the position the machine maker specifies so
the ATC can load the head
([BIG DAISHOWA angle head manual](https://us.bigdaishowa.com/sites/default/files/documents/2018-02/manual-big_daishowa-angle_head.pdf)).

**Limits from one maker's manual (fact, same source):**

- Maximum speed mostly 6,000 rev/min at 1:1; some models 3,000 rev/min; one #50 model
  8,000 rev/min at a 1:2 step-up; tapping types use a 2:1 reduction at 1,000 to 2,000 rev/min.
  The manual advises against running near maximum speed continuously.
- Most models turn the **opposite way** to the machine spindle. Twin-output heads have one
  reverse and one forward output.
- Run-in: start gently and raise speed, feed and depth gradually. Maximum temperature rise
  under no-load running: room temperature + 35 °C. Stop if vibration or chatter appears.
- Coolant through the stop block: up to 1 MPa (10 bar) on standard heads; universal heads take no
  coolant through the body, and the supplied nozzle is limited to 0.3 MPa (3 bar).
- The machine's ATC weight and moment limits apply; the head must also clear the magazine, the
  ATC arm and neighbouring pots. Some machines cannot change these heads automatically.
- No numeric torque rating is given in that manual; torque ratings come from each head's
  catalogue page (unverified per model).

**How CAM should treat it (opinion):**

- An angle head is a **holder with a transform**: input axis, output axis and offset (the output
  tool tip is not on the spindle centreline), gear ratio `i`, direction reversal, and a list of
  allowed output orientations (fixed by the pin position). Output speed `n_out = n_spindle × i`;
  output torque `M_out ≤ M_spindle / i × η` (η: gear efficiency, unverified typical values), and
  the head's own torque and speed ratings are separate, lower limits.
- The post-processor must **reverse M03/M04** where the head reverses, and emit the spindle
  orientation required before the tool change.
- A 3-axis machine with a 90° head can cut **side features** (cross holes, undercut slots, faces
  inside a deep part) in one setup. The toolpath generator needs the tool axis to be "−X" or "+Y"
  instead of "+Z". This is the same tool-axis model a 3+2 machine needs, so it costs little if the
  kernel already has tool axes.
- **Warm-up gate** (from the machine library file): the head's run-in becomes a step in the
  setup sheet, and its duty limit ("not continuously near max rpm") becomes a *Warn* when an
  operation spends more than a set share of time above, for example, 80 % of rated speed
  (example threshold).

**Uses the libraries.** Holder library (new "angle head" class with ratio, reversal, ratings,
coolant limit, weight and moment); machine library (stop block fitted, S dimension, orientation
angle, ATC weight/moment limits, whether ATC of angle heads is allowed); cutting physics (torque
and force at the output).

**Simulation gate.** *Stop*: output speed over rating; machine has no stop block or the wrong S
dimension; head weight or moment over the ATC limit; coolant pressure over the head's limit
(including through-spindle coolant left on). *Stop*: collision of the head body, which is much
larger than a normal holder, with the part or fixtures. *Warn*: long time near maximum speed;
first use without run-in recorded.

### 3. Grinding on a 3- or 5-axis mill

**Jason's idea:** consider grinding in a 3-axis or 5-axis mill.

**Prior art (fact).** Combined grinding and machining centres exist: United Grinding's Mägerle
MFP 50 is sold as a five- or six-axis machine that can "mill, drill and grind challenging
workpieces in a single clamping"
([MMS product note](https://mmsonline.com/products/multitasking-grinding-machining-center-promotes-productivity)).
These are grinding machines first, with the sealing, filtration and wheel guarding that implies.
On ordinary machining centres, shops grind with **electroplated CBN or diamond mounted points**
and small wheels on shanks, and some do **jig-grinding-style** bore finishing by helical or
circular interpolation (practitioner discussion, e.g.
[eMastercam forum](https://www.emastercam.com/forums/topic/37140-jig-grinding-on-cnc-mill/);
details unverified).

**Physics and limits:**

- **Wheel speed is a hard limit.** Every bonded wheel is marked with a maximum operating speed;
  exceeding it can burst the wheel (fact in general; the marking rules are in safety standards such
  as ANSI B7.1 and EN 12413, described here from summaries, unverified). Worked example (computed
  for this file): a 50 mm wheel at 12,000 rev/min runs at `v = π × 0.050 × 12000 / 60 = 31.4 m/s`.
  If the wheel is marked 35 m/s, its maximum spindle speed is `n = 35 × 60 / (π × 0.050) =
  13,369 rev/min`. CAM must take the **lower** of the marked rpm and the computed one, and the
  slider must not be able to raise it.
- **Guarding.** A mill has no wheel guard. Small mounted points on shanks are the common
  compromise (low mass); larger wheels on a mill raise a safety question the program cannot
  answer alone (opinion: CAM should refuse wheels above a size set by the shop, with a recorded
  approval to override, the same pattern as the workholding override).
- **Abrasive swarf.** Grinding dust and fine chips are abrasive to way covers, wipers, ball
  screws and spindle seals, and it settles in the coolant tank. Shops that grind on mills report
  extra filtration and cleaning (opinion and practitioner reports, unverified as a figure).
- **Dressing.** Vitrified wheels need dressing (a diamond in the vice or a rotary dresser);
  plated CBN/diamond points are not dressed and are replaced when worn. Dressing changes the
  wheel diameter, so the tool radius offset must update after each dress.
- **Stiffness and runout.** Grinding takes microns per pass; spindle runout and thermal growth
  become the error budget. The in-process inspection file's probing and the machine library's
  warm-up rule apply.

**Uses the libraries.** Tool library (new "grinding wheel / mounted point" class: shape,
diameter, bond type, abrasive, grit, marked maximum speed in m/s and rev/min, dressable yes/no,
minimum diameter after dressing); machine library (**"grinding allowed" flag** set by the shop,
coolant filtration class, spindle runout); cutting physics (a grinding force model is a later
phase; the first phase can use conservative published depths of cut per pass).

**Simulation gate.** *Stop*: commanded rpm above the wheel's limit; machine not flagged for
grinding; wheel diameter below its minimum after dressing. *Warn*: no dress cycle before a finish
pass with a vitrified wheel; dry grinding.

**TODO (Jason):** what you grind on mills, with what wheels, and how you keep the machine clean.

### 4. Burnishing

**Jason's idea:** consider burnishing operations.

**What it is (fact).** Burnishing presses hardened rolls or a ball against a surface with enough
force to flatten the peaks into the valleys. No chips are produced. A Cutting Tool Engineering
article on roller burnishing gives these figures
([CTE, "Roll to finish"](https://www.ctemag.com/sites/www.ctemag.com/files/archive_pdf/0106-rolltofinish.pdf)):

- Finish typically **1 to 10 µin Ra (0.025 to 0.25 µm)**; one example reached 5 µin
  (0.13 µm).
- Best starting surface: bored or turned, **80 to 120 µin Ra (2.0 to 3.0 µm)**, uniform and
  free of tears. A rough but even surface burnishes better than a very smooth one.
- Tools adjust over about 0.040 in (1.0 mm) of diameter in 0.0001 in (2.5 µm) steps; size can be
  held within 0.0005 in (13 µm). Flaking means too much interference; the fix is a smaller
  setting.
- Workpiece hardness normally **not above 40 HRC**. Surface hardness rises 5 to 10 % to a depth
  of 0.010 to 0.030 in (0.25 to 0.76 mm).
- Speeds and feeds are "generally not critical". Use a light, low-viscosity oil, and filter the
  coolant so grit does not get rolled into the surface.
- Any rotating spindle can run the tool.

**Kinds of burnishing tool (fact in general; details unverified):** multi-roll tools for bores
and outside diameters (they collapse on retract); single-roll and ball tools for faces and
contours, including **hydrostatic ball tools** fed by a high-pressure pump, which can follow 3D
surfaces and are sold for harder materials; diamond-tipped burnishing tools for turning.
Ecoroll and Elliott Tool publish application examples
([Ecoroll guide sleeve example](https://www.ecoroll.de/fileadmin/user_upload/uploads/APPLICATION_EXAMPLE___Automotive___Guide_sleeve.pdf),
[Elliott OD roll manual](https://www.elliott-tool.com/wp-content/uploads/2023/12/OD-carbide-roll-standard-style-tech-manual.pdf)).

**Why it matters to CAM:** burnishing **changes the size** (by roughly the peak height, a few
microns) and depends on the size of the bore before it. So the **previous operation's size and
finish are inputs** to the burnishing operation, which is the finish-first idea of section 1 in
small: the burnishing tool states the incoming size window and finish window; CAM sets the
boring operation to hit them.

**Uses the libraries.** Tool library (new "burnishing tool" class: type, diameter range, current
setting, interference range, incoming finish window, maximum workpiece hardness, feed per rev per
roll, retract behaviour, pump pressure for hydrostatic tools); physical properties of the stock
(hardness, from the physical-properties file in this set); in-process inspection (measure the
bore before burnishing, and after).

**Simulation gate.** *Stop*: workpiece hardness above the tool's limit; the previous operation
leaves a bore outside the tool's incoming window. *Warn*: no bore measurement before burnishing
on a tight-tolerance bore; unfiltered coolant.

### 5. Flexible hones and abrasive brushes

**Jason's idea:** use Brush Research brushes as an operation to improve the surface finish of
parts, and ball hones for bores.

**Flexible hones (fact).** The Flex-Hone (Brush Research Manufacturing, "BRM") is a ball hone:
abrasive globules on flexible filaments. It is **always used oversize** to the bore; between two
standard sizes, choose the larger, for example a 1-1/8 in (28.6 mm) tool for a 1.093 in
(27.8 mm) bore
([BRM, how to select a Flex-Hone](https://www.brushresearch.com/pages/how-to-select-a-flex-hone%C2%AE)).
BRM gives these running figures
([BRM blog, speed and stroke rate](https://blog.brushresearch.com/flex-hone-tips-for-speed-and-stroke-rate)):

- **60 to 1,200 rev/min**, larger tools slower, smaller faster; higher speeds shorten tool life.
- Stroke rate about **100 to 120 in/min (2.5 to 3.0 m/min)** for larger tools, up to about
  **250 in/min (6.4 m/min)** for small ones.
- Crosshatch: 20° to 23° with a lower stroke rate, 45° with a higher one.
- Lubricate generously, and **withdraw the tool while it is still turning**.

The crosshatch angle follows from the ratio of stroke speed to surface speed. Worked example
(computed for this file): a 28.6 mm hone at 600 rev/min has a surface speed of
`π × 28.6 × 600 = 53.9 m/min`; at a stroke of 3.0 m/min the helix angle to the bore's
circumference is `atan(3.0 / 53.9) = 3.2°`, and the crosshatch included angle is twice that,
6.4°. To reach a 45° included angle (22.5° each way) the stroke must be
`53.9 × tan 22.5° = 22.3 m/min`, or the speed must fall to about 80 rev/min at 3.0 m/min. So
CAM can **compute** the stroke feed from the crosshatch angle the user asks for, instead of the
user guessing. (BRM's angle figures may be defined differently; unverified which convention
they use.)

What a flexible hone does **not** do (opinion, consistent with BRM's description of it as a
"soft cutting action"): it does not correct size, roundness or straightness much; it improves
finish, removes burrs at cross holes, and produces the plateau finish engine bores want. So it
sits after the operation that sets size.

**Abrasive-filament brushes (fact).** BRM's NamPower disc brushes deburr and finish faces and
edges. BRM recommends starting feeds of **30 to 80 in/min (0.76 to 2.0 m/min)**, settled by
trials; warns that "the maximum RPMs that are marked on a brushing tool may not be the optimum
working speed"; and says speed is reduced as interference (how far the filaments are pressed into
the part) increases, smaller brushes and coarser grits run faster, long trims flare at high
speed, and contoured surfaces run slower with more interference than flat ones
([BRM blog, NamPower speeds](https://blog.brushresearch.com/whats-the-right-speed-for-nampower-disc-brushes)).
Numeric speed tables are in BRM's NamPower resource guide (not reproduced here).

**How CAM should treat them (opinion):**

- **Brushing is a toolpath with interference, not depth.** The program places the brush face
  below the surface by the interference (for example 1 to 2 mm, example values) and passes over
  every edge to be broken. The generator is a face or contour pass with a different contact
  model: the brush "sees" the whole face and edges, so the path can be coarse.
- **Honing is a stroke cycle**: rapid to the top of the bore with the spindle on, a set number
  of strokes at the computed feed, then out while turning. On a mill this is a canned-cycle-like
  macro (or a G1 sequence), and must keep the spindle on during the exit.
- **Wear.** Brushes and hones wear shorter and softer, so interference must be re-set by tool
  length or by a few trial passes. A tool-life count per part belongs in the tool record.

**Uses the libraries.** Tool library (new classes "flexible hone" and "abrasive brush": diameter,
oversize rule, abrasive type, grit, filament trim length, marked maximum rpm, recommended range,
interference range, wear allowance); machine library (spindle minimum speed: many spindles do not
run smoothly at 60 rev/min, unverified per machine); tool holding (most of these tools have plain
shanks, so a collet or side-lock holder; balance is not usually the limit).

**Simulation gate.** *Stop*: commanded rpm above the brush's marked maximum; hone diameter
smaller than the bore (it would not cut) or far above the next standard size; exit move with the
spindle stopped. *Warn*: no lubricant or coolant on a hone; interference larger than the
brush's range.

**TODO (Jason):** which BRM tools you use, on which parts, and what speeds worked.

### 6. Turning on a mill's 4th axis

**Jason's idea:** consider turning operations on a 4th axis on mills.

**The idea (fact in general).** The rotary axis holds and spins the part like a lathe chuck. A
static turning tool, held in the mill spindle (locked) or in a block on the table, cuts as the
linear axes move. Some rotary tables offer lathe-style spindle noses so lathe chucks mount on them:
Haas lists A1-5 and A1-6 noses on some models and says most of its lathe workholding "mounts
directly" ([Haas rotaries and indexers guide](https://www.haascnc.com/content/dam/haascnc/assets/printdocuments/rotary-guide/2026-08-RotariesAndIndexers-Guide.pdf)).
Purpose-built **mill-turn tables** (direct-drive tables with turning speeds) exist on 5-axis
mill-turn machines from several makers (unverified per model).

**Physics and limits:**

- **Speed.** Normal rotary tables are slow. Haas's high-speed HRT200HS is listed at 1,000°/s,
  with 142 N·m spindle torque and 136 N·m brake torque (same guide). Worked example (computed for
  this file): 1,000°/s is `1000 / 360 × 60 = 167 rev/min`. An 80 mm aluminium bar at that speed
  has a surface speed of `π × 80 × 167 / 1000 = 42 m/min`, far below what carbide normally runs
  in aluminium (often several hundred m/min; estimate). It is enough for small steel and
  stainless parts at modest finish, or for large diameters.
- **Torque.** Cutting force `Fc = kc × ap × f`. Example (computed, with an assumed
  `kc = 2,000 N/mm²` for steel): `ap = 2 mm`, `f = 0.2 mm/rev` gives `Fc = 800 N`; at a radius of
  40 mm the torque is `800 × 0.040 = 32 N·m`, inside the 142 N·m figure above. But a worm-gear
  rotary may not be rated for **continuous** rotation under load (the guide does not state it for
  4th-axis tables); heat and wear in the worm set are the real limit (opinion).
- **The tool side.** A turning tool held in the mill spindle needs the spindle **oriented and
  held**. The force acts at the tool tip, which is usually offset from the spindle axis, so it
  becomes a torque on the spindle orientation servo: 800 N at a 20 mm offset is 16 N·m (computed).
  Whether the spindle can hold that, and whether the drawbar and taper like a side load with no
  rotation, are machine-specific (unverified per machine). Holding the tool in a block on the
  table, or using a holder designed for static use, avoids the question.
- **Centre height.** As on a lathe, the tool tip must be on the rotary axis centreline; an error
  changes the effective rake and leaves a pip at the centre on facing. The tool-holding file's
  non-rotating tools section covers centre height and orientation checks.
- **An alternative**: some tool makers offer **interpolation turning**, where the tool rotates
  with the spindle around a stationary part in circular interpolation, keeping the edge pointed
  at the centre. It suits short diameters and grooves on parts that cannot spin (unverified as to
  which makers and controls support it).

**Uses the libraries.** Machine library (new fields per rotary axis: "turning rated" yes/no,
maximum continuous rev/min, continuous torque, duty cycle, nose type, maximum part mass and
overhang with tailstock; and per spindle: whether orientation can hold a static tool, and the
holding torque); tool library (static turning tools: insert, holder, centre height, the offset
from the spindle axis); cutting physics (force and torque, as above); workholding (chuck grip on
a spinning part, with the centrifugal loss of grip at speed).

**Simulation gate.** *Stop*: rotary axis not rated for turning; speed or torque over the
continuous rating; tool held in an unlocked spindle; part mass or overhang above the axis limit
without tailstock. *Warn*: surface speed below the insert's minimum (poor finish, built-up edge);
centre height not verified.

### 7. Stock-aware rapids instead of clearance planes

**Jason's idea:** instead of clearance planes for moves, use a 3D model of the stock to move
rapidly and close to the stock, saving movement and time.

**Prior art (fact in general).** Most CAM programs separate **clearance**, **retract** and **feed**
heights, and offer linking options such as "minimum retract", "keep tool down" and "shortest
path" that stay closer to the part (for example
[Javelin on SOLIDWORKS CAM planes](https://www.javelin-tech.com/blog/2021/06/clearance-rapid-and-feed-planes-for-speed-and-safety-with-solidworks-cam/),
[Onshape CAM Studio tips](https://cad.onshape.com/help/Content/camtooltips.htm)). Some programs
check link moves against the in-process stock model (trade articles describe "stock-aware
toolpaths", details unverified). The request goes further: make the **in-process stock model
the default reference for every link and rapid**, so the planes become a fallback.

**Why it saves time (worked example, computed for this file).** Take 120 link moves in a part.
With a clearance plane 25 mm above the stock top, each link goes up 22 mm more and down 22 mm
more than a link that stays 3 mm above the local stock. Assume the Z axis rapids at 15 m/min
(250 mm/s) and accelerates at 3 m/s² (assumed values). A 22 mm move barely reaches full speed
(it needs `v² / a = 0.25² / 3 = 0.021 m = 21 mm` to accelerate and stop), so it takes about
`2 × √(d / a) = 2 × √(0.022 / 3) = 0.17 s`. Two such legs per link, 120 links:
`120 × 2 × 0.17 = 41 s` per part. Ignoring acceleration would have predicted only 21 s; short
moves are dominated by acceleration, so the machine library's acceleration values matter here.

**Physics and limits:**

- **The stock model is only as good as the stock.** Saw-cut bar varies in length, castings and
  forgings vary in shape, and the in-process model can be wrong after a broken tool or an
  operator edit. The stock-integrity file describes stock uncertainty; the link clearance must be
  the modelled stock plus that uncertainty plus a margin, and it must be larger where the stock
  was not probed.
- **G0 is not always a straight line.** On many controls a rapid moves each axis at its own
  maximum speed, so a diagonal rapid becomes a "dog-leg" (one axis finishes first) (fact for
  many controls; behaviour varies and some have a setting for straight-line rapids). The link
  must be checked against the **actual swept path** the machine will take, which the machine
  library can describe (straight-line or per-axis rapids, and each axis's rapid speed). Where the
  control dog-legs, CAM can use high-feed G1 for links near the stock instead.
- **The holder and the head count too.** A link close to the stock must clear the holder, the
  angle head body (section 2), fixtures and clamps, not just the cutter.
- **Re-entry.** Coming down close to the stock at rapid and then feeding the last few
  millimetres is normal; the feed-in distance must cover the stock uncertainty, otherwise the
  tool enters stock at rapid speed.

**Uses the libraries.** Machine library (rapid speeds and accelerations, rapid mode, a
"rapid near stock" margin set by the shop); stock and stock integrity (in-process model and its
uncertainty, probed or not); workholding (fixture and clamp solids); tool holding (holder
envelope).

**Simulation gate.** *Stop*: any rapid whose swept volume (cutter, holder, head) touches the
modelled stock **plus uncertainty**, fixtures or clamps. *Warn*: rapid within the margin of
unprobed stock. *Info*: time saved compared with plane-based linking, so the user sees the gain.

### 8. Global arc fitting to G2/G3

**Jason's idea:** a global option to smooth all of the code with G2/G3, to save program size,
speed execution on older controllers, and make machines happy.

**Prior art (fact).** Arc fitting is a standard post or toolpath option in most CAM programs and
in post-processing products ([ICAM NURBS and arc fitting](https://icam.com/nurbs-arc-fitting-solutions/)).
Modern Machine Shop notes that arc-based cuts can often be defined "with less than a tenth of the
program data" of point-to-point contouring, that dense blocks cause "stuttering cuts as controls
struggle to process dense point (block) clusters", and that standard circular interpolation is
limited to the XY, XZ and YZ planes on most machines
([MMS, "Machining in circles"](https://www.mmsonline.com/articles/machining-in-circles)). A
forum thread about a year-2000 NUM control shows the problem in practice: hundreds of tiny G1
segments (some 0.07 mm long) caused jerky motion and very low actual feed; the suggested fix was a
**smaller** original toolpath tolerance (0.01 or 0.005 mm) with about 0.1 mm smoothing so the post
could fit more arcs
([Autodesk forum](https://forums.autodesk.com/t5/fusion-manufacture-forum/excessive-micro-segments-causing-jerky-motion-on-older-cnc-num/td-p/14175829)).

**Physics (worked examples, computed for this file):**

- **Block time limits feed.** If a control needs `t_b` seconds to process a block, a segment of
  length `L` cannot run faster than `L / t_b`. With `L = 0.1 mm` and `t_b = 4 ms` (assumed for an
  older control), the limit is `0.1 / 0.004 = 25 mm/s = 1,500 mm/min`, whatever F says. One arc
  replacing fifty such segments (5 mm long) raises the limit to `5 / 0.004 = 1,250 mm/s`, so it
  stops being the binding limit.
- **Program size.** A chord of length `L` on a radius `R` deviates `h ≈ L² / (8R)` from the arc,
  so `L = √(8 R h)`. For `R = 10 mm` and `h = 0.005 mm`, `L = √(0.4) = 0.63 mm`. A quarter circle
  (15.7 mm of arc) needs 25 segments, or one G2 block.
- **Tolerance stacking.** The fitted arc can deviate from the toolpath by the fit tolerance
  `t_fit`, and the toolpath already deviates from the surface by its own tolerance `t_path`. The
  worst case is `t_path + t_fit`. A safe default is to split the user's surface tolerance between
  them (for example half each), which is why the forum advice to tighten the toolpath tolerance
  before fitting works.

**Pitfalls the option must handle (fact in general; details per control unverified):**

- Arcs only in the active plane (G17/G18/G19); 3D surfacing gains less, and helices need
  G2/G3 with a linear third axis. Some controls fit NURBS or smooth points themselves (look-ahead
  and smoothing modes such as Fanuc AI contour control); where those exist, short G1 may be
  better than many small arcs.
- **R versus I/J/K** format: R is ambiguous for arcs over 180° and loses precision on very large
  radii; most posts prefer I/J/K and split arcs at quadrants or at 180°.
- Controls check that the start and end radii match within a limit and alarm if not; rounding
  to the post's decimals can trip that check on large radii.
- Very large radii (near-straight lines) and very short arcs should stay as G1.
- Cutter compensation (G41/G42) and arcs interact; arcs smaller than the compensation radius on
  the inside are errors.

**How it should be offered (opinion).** A **machine-level default** (in the machine library:
control family, block time, look-ahead, arc planes, preferred format, smoothing mode available),
a **project switch** to apply it to every operation, and a per-operation override. The default
tolerance split follows the surface tolerance. The setup sheet shows the before and after block
counts and the estimated time difference.

**Uses the libraries.** Machine library (control block time, arc support, maximum arc radius,
decimals, smoothing modes); cutting physics (feed per tooth stays correct on arcs only if the
post adjusts feed for the tool centre radius on inside and outside arcs).

**Simulation gate.** The gate must simulate the **posted** program, not the toolpath before
fitting. *Stop*: fitted program deviates from the surface by more than the surface tolerance;
arc radius mismatch beyond the control's limit after rounding. *Warn*: feed limited by block time
in a region (with the achievable feed shown).

### 9. Serial numbers and live text fields in engraving

**Jason's idea:** be able to serialise parts with engraving, and add macro-type text fields such
as time and date for engraving.

**What already exists (fact).** Several controls can engrave text and serial numbers themselves.
Haas mills have **G47**: P0 engraves literal text, P1 engraves a **sequential serial number**;
the serial is held in a macro variable and incremented each cycle; E is the plunge feed, F the
engraving feed, I the rotation angle, and the XY start is the lower-left of the first character
(summary from [Plastic Ranger on G47](https://plasticranger.com/g47-cnc-code/); practitioners name
macro variable #599 as the serial store,
[CNC Arena thread](https://en.cncarena.com/forum/thread/174400-g47-macro-599-question/);
check the Haas manual, unverified). Fanuc-family custom macro controls expose the date and time
as system variables (#3011 date as YYYYMMDD, #3012 time as HHMMSS; unverified per control), and
shops write subprograms that engrave a number digit by digit
([LinuxCNC forum on sequential serial engraving](https://forum.linuxcnc.org/40-subroutines-and-ngcgui/33841-sequential-serial-number-engraving),
[eMastercam thread](https://www.emastercam.com/forums/topic/95341-need-help-with-sequential-engraving-macro/)).

**Marking methods and standards are in the inspection file.** The
[in-process inspection file](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-y2w06n/research/cam-research-nsnlookupjn-in-process-inspection.md)
section "Part serialisation and marking" covers engraving versus DataMatrix, dot peen and laser,
MIL-STD-130 and AS9132, and suggests a marking feature with a template such as
`{part}-{rev}-{serial}`. This section adds the **fields** and the **counter**.

**Proposed text fields (opinion):**

| Field | Example | Filled by |
|---|---|---|
| `{serial}` with width and padding, e.g. `{serial:05}` | `00042` | counter (see below) |
| `{date}` with format, e.g. `{date:YYMMDD}` | `261026` | control clock or edge device |
| `{time:HHmm}` | `2151` | control clock or edge device |
| `{part}`, `{rev}` | `FD-104`, `C` | design document |
| `{lot}`, `{order}`, `{heat}` | `L2610-3` | job record (edge device or operator) |
| `{machine}`, `{op}` | `VF2-01`, `OP20` | machine library, setup |
| `{cage}` | a CAGE code | shop profile (for MIL-STD-130 content, see inspection file) |

**Two ways to fill them (opinion):**

1. **On the control.** The post writes the fixed text as geometry and the live fields as calls
   to a control macro (G47 P1 on Haas, a digit-engraving subprogram elsewhere) reading the
   control's clock and counter variable. Works offline; limited to what the control can format.
2. **At the edge device.** The Pi box in the cell-automation file writes a small engraving
   program per part, or sets macro variables (`#500`-range common variables) before the cycle
   starts, from a **central counter**. This gives serials that are unique across machines,
   lot numbers from the job, and a record of which serial went to which part.

**Things the feature must get right (opinion):**

- **Uniqueness.** A per-machine counter gives duplicates when two machines run the same part.
  The counter should live in one place (the job record), and a serial is **consumed** when
  engraved, even if the part is later scrapped; the record says so.
- **Restart.** If the program is restarted mid-cycle, the counter must not advance twice. The
  increment happens after the engraving finishes, and the record keeps the last engraved value.
- **Depth on real surfaces.** Engraving 0.1 to 0.2 mm deep (example values) on a face that is
  0.05 mm out of flat gives uneven text. Probe the face first (inspection file) or use a
  spring-loaded engraving tool; on curved faces, wrap or project the text along the surface
  normal.
- **Fonts.** Engraving needs single-line ("stroke") fonts. SolidCraft must ship an original or
  openly licensed stroke font with an ATTRIBUTION.md row (ground rules).
- **Character set.** Some controls only engrave a limited character set; the gate must check
  that the text after filling the fields uses only allowed characters.

**Uses the libraries.** Tool library (engraving tool: tip angle, tip width, spring-loaded or
not, so the line width follows depth); machine library (control's engraving support, macro
support, date/time variables, allowed characters); cell automation (counter, lot, record).

**Simulation gate.** *Stop*: the longest possible filled text (for example the serial at its
maximum width) does not fit the marking area, or runs off the face; character not supported by
the control's engraving cycle. *Warn*: depth not verified on an unprobed face; per-machine
counter on a part released to several machines.

### 10. Tool-setter placement and a safe length find

**Jason's idea:** have the mill's tool setter position optimised by CAM for each setup for
maximum productivity, and the tool setter should not require the rough length of a tool to be
measured by the operator; use a safe move to find it.

**What the setter makers say (fact).** Renishaw's FAQ on contact tool setters asks how to measure
a tool without an approximate length and answers: "You can't, an approximate tool length is
always needed"
([Renishaw contact tool setter questions](https://www.renishaw.com/en/questions-on-renishaw-contact-tool-setters--15720)).
The reason is the stylus **overtravel**: the OTS allows ±3.5 mm in XY and **6 mm in +Z** beyond
the trigger point before damage, with a trigger force of 1.30 to 2.40 N and 1.00 µm 2σ
repeatability tested at 480 mm/min
([Renishaw OTS data sheet](https://renishaw.com/resourcecentre/en/download/data-sheet-ots--142125)).
The control must stop the axis within that overtravel after the trigger, so the fast approach
can only go as far as the expected length allows. Jason's request is to remove the operator's
rough measurement from that chain.

**Part A: a safe search instead of a typed rough length (opinion, with computed numbers).**

1. **First choice: the library knows the length.** The tool library's **assembly record**
   (holder gauge length + tool stick-out, from the tool-holding file and the cell-automation
   file's presetter flow) gives an expected length with a tolerance window. The cycle rapids to
   "expected + window" above the setter, then searches through the window at a safe speed. No
   typing.
2. **When nothing is known: search from the longest tool the machine can carry.** The machine
   library knows the maximum tool length the ATC allows. The cycle starts with the spindle Z
   high enough for that length and searches down at a speed `v` chosen so the machine stops
   inside half the stylus overtravel.

   Stopping distance after the trigger: `d = v × t_lat + v² / (2a)`, where `t_lat` is the total
   latency (probe signal, interface, skip input, servo response) and `a` the Z deceleration.
   With assumed values `t_lat = 10 ms` and `a = 2 m/s²` (conservative assumptions; real values
   belong in the machine library, measured once per machine):

   | Search speed | `v × t_lat` | `v² / 2a` | Stop distance | Inside 3 mm (half of 6 mm)? |
   |---|---|---|---|---|
   | 2 m/min (33 mm/s) | 0.33 mm | 0.28 mm | 0.61 mm | yes |
   | 5 m/min (83 mm/s) | 0.83 mm | 1.74 mm | 2.57 mm | yes, barely |
   | 6 m/min (100 mm/s) | 1.00 mm | 2.50 mm | 3.50 mm | no |

   A 250 mm search at 5 m/min takes 3 s, then the normal slow measuring touch follows. So a
   blind search is not slow; the time it adds is small next to an operator walking to the
   machine with a ruler (estimate).
3. **What the stylus cannot protect.** Overtravel protects only a straight-down touch on the
   stylus. The search must be exactly over the stylus, with the **tool diameter** known well
   enough that the tool tip, not the holder or the side of a large cutter, meets it; face mills
   wider than the stylus disc are measured off-centre on one insert. If the tool diameter is
   unknown too, a non-contact (laser) setter, which measures at speed by beam break, or a camera
   check at the Pi box, should come first (opinion; the cell-automation file lists NC4-class
   laser setters).
4. **Sanity windows.** The found length is compared with the library value (cell-automation
   file: accept within the window, stop outside it). A blind search with no library value
   records the found length as **"measured, no reference"** so the next use has a reference.
5. **Maker's rules win.** The setter's own macro may fix maximum approach speeds and the
   two-touch method; CAM supplies the start height and the window, the maker's cycle does the
   touch.

**Part B: placing the setter per setup (opinion, with an estimate).**

Many machines have one fixed setter position (often a table corner). Others have several mounts,
or a magnetic or quick-change base. CAM knows the fixtures, the stock, the travel limits and how
often each tool is checked (setting, broken-tool checks between operations), so it can choose:

- **Candidates:** the shop's declared mounts in the machine library (a position and a
  calibration record each), or free table positions on a quick-change base.
- **Constraints:** reachable by the longest and the widest tool at its search start height; not
  in the swept volume of any toolpath, fixture, rotary axis swing or pallet change; out of the
  main chip and coolant stream (a chip on the stylus gives a wrong length); not under the angle
  head's reach (section 2).
- **Cost:** total time per part of all setter visits: each visit's rapid from the last cutting
  position to the setter and on to the next start, using the machine library's speeds and
  accelerations (as in section 7).

Example (computed for this file, assumed values): 20 setter visits per part (one set at the
start of the batch is excluded; these are broken-tool checks); moving the setter from a corner
600 mm from the work to a mount 150 mm from it saves `2 × 450 = 900 mm` per visit; at 30 m/min
(0.5 m/s) that is about 1.8 s per visit (ignoring acceleration), or **36 s per part**. Over a
batch of 200 parts, two hours.

A moved setter must be **re-calibrated** before it is trusted (its trigger height and XY
centre), so CAM should only propose a move between batches, and the setup sheet says so.

**Uses the libraries.** Machine library (setter type, mounts with calibrated positions, stylus
overtravel, trigger latency, Z deceleration, maximum tool length and diameter); tool library and
tool holding (expected assembly length and diameter, which side of the tool touches); cell
automation (the presetter value, the check window, the record).

**Simulation gate.** *Stop*: search speed whose stopping distance exceeds the allowed share of
overtravel; search not centred over the stylus for the tool's diameter; setter inside a
toolpath's or fixture's swept volume. *Warn*: blind search with no tool diameter; setter in the
coolant stream; setter mount not calibrated since it was moved.

**Jason's shop experience (personal experience, 2026-10-26):** today offsets are loaded in one of
two ways: tools are set to a known length on a tool setter and the values are typed into the
machine by hand, or the machine's own tool setter measures length (and optionally diameter) with
its built-in macros. Both normally need a **rough tool length, measured with a tape measure**, to
save setting time. That tape-measure step is what Part A removes.

**TODO (Jason):** your setter (make, mount).

## Accuracy

- **Of the worked examples:** exact arithmetic on stated inputs. Inputs marked as assumed (latency,
  accelerations, `kc`, block time) are examples; real values come from the machine library
  (measured per machine) and the cutting physics file.
- **Of maker figures:** quoted from one maker each (BIG DAISHOWA for angle heads, BRM for hones
  and brushes, Renishaw OTS for the setter, Haas for rotary tables). Other makers' products
  differ; the library must carry each product's own values.
- **Of the CloudNC description:** the public blog confirms "roughing as an overall problem", not
  the finish-first order; that order is Jason's observation.
- **Of control behaviour:** dog-leg rapids, macro variables and arc checks vary by control and by
  parameter settings; each is marked unverified where it is not from the maker's manual.

## Programs

What the features would look like in SolidCraft's program structure (opinion):

| Feature | Kind of addition | Phase (from the CAM Physics System plan) |
|---|---|---|
| 1 Finish-first planning | planning mode over existing operations | after adaptive and rest roughing exist |
| 2 Angle heads | holder class + tool-axis per operation + post rule | with 3+2 tool axes |
| 3 Grinding | tool class + operation variants + machine flag | later (needs safety review) |
| 4 Burnishing | tool class + bore/OD operation | early (simple motion) |
| 5 Hones and brushes | tool classes + stroke cycle + face/edge pass | early |
| 6 4th-axis turning | operation type + rotary turning fields + static tool | with 4th-axis support |
| 7 Stock-aware rapids | linking option + gate check | with in-process stock model |
| 8 Arc fitting | post option + machine default | with the first post-processor |
| 9 Serial engraving | text fields + counter + post macro | with engraving |
| 10 Tool setter | machine setter data + setter cycle generator + placement solver | with probing support |

Every one is a **command** with input and output schemas (repository rule), so an agent can, for
example, ask "fit arcs on this program at 0.005 mm and report block counts" without a dialog.

## Complaints and requests

Patterns seen in public forums and articles (summaries, not quotes):

- Older controls stutter on dense short segments; users ask for better arc output and get told to
  change tolerances by trial (Autodesk forum thread above).
- Programmers spend time hand-editing roughing after the fact; CloudNC's update claims to reduce
  "the amount of manual editing" (CloudNC blog, **marketing**).
- Serial engraving is usually hand-written macros per control (LinuxCNC, eMastercam threads).
- Tool setters still need a rough length typed in (Renishaw FAQ above).
- Angle heads: machines that cannot change them automatically, interference in the magazine,
  and coolant ingress (BIG DAISHOWA manual's own cautions).

## What users need

1. Finishers that state what they need, and roughing that delivers it (section 1).
2. New tool classes in the shared library: angle head, grinding wheel, burnishing tool, flexible
   hone, abrasive brush, static turning tool (sections 2 to 6).
3. A rapid and link planner that trusts the stock model only as far as the stock is known
   (section 7).
4. One switch for arc output that is safe by default on each machine (section 8).
5. Engraving text with live fields and a shop-wide serial counter (section 9).
6. A tool setter that never needs a ruler, placed where it costs least time (section 10).
7. For every one: the reason behind each limit shown in plain words, and a gate that stops the
   unsafe cases.

## Glossary

- **Adaptive clearing:** roughing that keeps the tool's engagement constant, usually with
  trochoidal motion.
- **Angle head:** a geared tool holder whose output spindle is at an angle to the machine
  spindle; held still by a pin in a **stop block**.
- **Arc fitting:** replacing runs of short straight segments with circular arcs (G2/G3) within a
  tolerance.
- **Block processing time:** how long a control needs to read and prepare one line of a program.
- **Burnishing:** finishing by plastic deformation with rolls or a ball, without chips.
- **Crosshatch:** the criss-cross scratch pattern left by honing, described by its angle.
- **Dog-leg rapid:** a rapid where each axis moves at its own maximum speed, so the path is not
  straight.
- **Flexible hone (ball hone):** abrasive globules on flexible filaments, used oversize in a bore.
- **Interference (brushing, burnishing):** how far the tool is pressed into the surface beyond
  first contact.
- **In-process stock model:** the 3D shape of the material after each operation.
- **Overtravel (tool setter):** how far the stylus can move after triggering before damage.
- **Stroke font:** a font made of single lines, for engraving.

## Open questions

1. Does CloudNC publish anything about planning order? If not, can Jason describe what he sees
   in the product output (without driving it for this research)?
2. What finishing-load windows are safe defaults per tool size and material? (Cutting physics
   file owner.)
3. Which angle head makers publish torque ratings, and do any publish duty cycles?
4. What wheel size should be the default limit for grinding on a mill without a guard?
5. Is there an openly licensed source of burnishing interference values per material?
6. Which rotary tables are rated for continuous turning, and at what torque?
7. How should the in-process stock uncertainty be represented: a single offset, per face, or a
   probability band?
8. What is a measured trigger-to-stop latency on common controls? (A simple test the cell's Pi
   could run and store in the machine library.)
9. What default serial counter scope do shops want: per part number, per customer, per year?
10. **TODO (Jason):** which of the ten matters most in your shop, to set the order of work.

## Sources

- CloudNC, "Next Gen Roughing: a major CAM Assist update", 15 Jan 2025:
  https://www.cloudnc.com/blog/next-gen-roughing-a-major-cam-assist-update
- BIG DAISHOWA, angle head instruction manual:
  https://us.bigdaishowa.com/sites/default/files/documents/2018-02/manual-big_daishowa-angle_head.pdf
- Modern Machine Shop, multitasking grinding and machining centre (Mägerle MFP 50):
  https://mmsonline.com/products/multitasking-grinding-machining-center-promotes-productivity
- eMastercam forum, jig grinding on a CNC mill:
  https://www.emastercam.com/forums/topic/37140-jig-grinding-on-cnc-mill/
- Cutting Tool Engineering, "Roll to finish" (roller burnishing):
  https://www.ctemag.com/sites/www.ctemag.com/files/archive_pdf/0106-rolltofinish.pdf
- Ecoroll, application example (guide sleeve):
  https://www.ecoroll.de/fileadmin/user_upload/uploads/APPLICATION_EXAMPLE___Automotive___Guide_sleeve.pdf
- Elliott Tool, OD carbide roll technical manual:
  https://www.elliott-tool.com/wp-content/uploads/2023/12/OD-carbide-roll-standard-style-tech-manual.pdf
- Brush Research Manufacturing, how to select a Flex-Hone:
  https://www.brushresearch.com/pages/how-to-select-a-flex-hone%C2%AE
- Brush Research Manufacturing blog, Flex-Hone speed and stroke rate:
  https://blog.brushresearch.com/flex-hone-tips-for-speed-and-stroke-rate
- Brush Research Manufacturing blog, NamPower disc brush speeds:
  https://blog.brushresearch.com/whats-the-right-speed-for-nampower-disc-brushes
- Haas, Rotaries and Indexers guide (2026-08):
  https://www.haascnc.com/content/dam/haascnc/assets/printdocuments/rotary-guide/2026-08-RotariesAndIndexers-Guide.pdf
- Javelin, clearance, rapid and feed planes in SOLIDWORKS CAM:
  https://www.javelin-tech.com/blog/2021/06/clearance-rapid-and-feed-planes-for-speed-and-safety-with-solidworks-cam/
- Onshape, CAM Studio tool tips: https://cad.onshape.com/help/Content/camtooltips.htm
- Modern Machine Shop, "Machining in circles": https://www.mmsonline.com/articles/machining-in-circles
- ICAM, NURBS and arc fitting: https://icam.com/nurbs-arc-fitting-solutions/
- Autodesk forum, micro-segments on an older NUM control:
  https://forums.autodesk.com/t5/fusion-manufacture-forum/excessive-micro-segments-causing-jerky-motion-on-older-cnc-num/td-p/14175829
- Plastic Ranger, G47 CNC code: https://plasticranger.com/g47-cnc-code/
- CNC Arena forum, G47 macro #599 question:
  https://en.cncarena.com/forum/thread/174400-g47-macro-599-question/
- LinuxCNC forum, sequential serial number engraving:
  https://forum.linuxcnc.org/40-subroutines-and-ngcgui/33841-sequential-serial-number-engraving
- eMastercam forum, sequential engraving macro:
  https://www.emastercam.com/forums/topic/95341-need-help-with-sequential-engraving-macro/
- Renishaw, questions on contact tool setters:
  https://www.renishaw.com/en/questions-on-renishaw-contact-tool-setters--15720
- Renishaw, OTS data sheet: https://renishaw.com/resourcecentre/en/download/data-sheet-ots--142125
