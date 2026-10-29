# CAM research: CNC milling best practices as rules CAM applies and simulation checks

Research for [#1621](https://github.com/tangency-labs/solidcraft/issues/1621) (Research Request
for Plugin: CAM). This file covers one part of that request: **shop-floor milling best practices
turned into rules that CAM applies automatically and that the simulation checks before G-code is
posted.** The practices covered are:

1. Coolant rules for carbide inserts (no intermittent or partial coolant on a hot edge).
2. Rolling into and out of the cut to keep tool stress low.
3. Lead-ins and lead-outs whose feed (and, where the machine allows, spindle speed) ramps
   smoothly instead of stepping.
4. Chip clearing cycles (air blast, chip fans, coolant flush) inserted between cuts where the
   program can predict that chips will be recut.
5. Predicting chip build-up in the simulation (in the cut, on the workpiece, in the machine and
   around the tool setter) and checking it with cameras.
6. Cleaning routines as standard CAM operations: washdown to the chip conveyor, part wash and
   dry, camera window cleaning.
7. Cleaning a tool before it goes back into the magazine.

It is a companion to these files in the same fork, and does not repeat them:

- [Cutting tool physics](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-cutting-physics/research/cam-research-nsnlookupjn-cutting-physics.md):
  chip thickness, forces, tool life, and the section "Coolant, air blast and chip clearing fans",
  which says *what* coolant does to tool life and why recut chips are a flag, not a slider
  factor. This file says *what CAM does about it*.
- [CAM Physics System](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-vgqo3g/research/cam-research-nsnlookupjn-physics-system.md):
  the shared setup, the limits engine, the slider and the simulation gate ("Physics in the
  simulation: the gate before G-code"). Every check in this file reports into that gate with the
  same *Stop / Warn / Info* severities.
- [In-process inspection](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-y2w06n/research/cam-research-nsnlookupjn-in-process-inspection.md):
  **SolidCraft Edge** (Jason's name for the Raspberry Pi computer at the machine, which sits on
  the edge of CAM and brings the machine and automation back into CAM for closed-loop process
  improvement) and Self-tuning CAM Operations. This file adds cameras and
  chip observations to that box.
- [Machine health](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-9w3gq9/research/cam-research-nsnlookupjn-machine-health.md):
  coolant concentration and temperature ("Coolant condition"), which this file does not repeat.
- [Machine library](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-machine-library/research/cam-research-nsnlookupjn-machine-library.md)
  and [cell automation](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-g08nas/research/cam-research-nsnlookupjn-cell-automation.md):
  the machine's fitted options and the tool setter, which the cleaning cycles here depend on.

Licence: MIT OR Apache-2.0, the same as SolidCraft (see the Licence section of CONTRIBUTING.md).

## About this research

- **Author:** Jason ([@nsnlookupjn](https://github.com/nsnlookupjn)). The practices and the
  requests (coolant rule for inserts, roll-in/roll-off, ramped feed and speed on lead-in and
  lead-out, chip clearing cycles, chip build-up in simulation and by camera, cleaning routines as
  standard operations, camera cleaning, tool self-cleaning) are Jason's; source research and
  writing done with an AI assistant (Claude) in October 2026.
- **How it was produced:** web searches of public sources (tool makers' and machine builders'
  public pages, trade press, a patent). No commercial CAD/CAM program was driven or scripted, no
  GPL/LGPL/AGPL source was read, and no text is copied beyond single-sentence quotes.
- **Units:** millimetres, seconds, revolutions per minute, newtons, unless a source is quoted in
  other units.
- **Marking:** "(unverified)" marks a claim not confirmed against a source. "(derived)" marks a
  number worked out here. **Opinion** is labelled. Machine codes are given only where a public
  page lists them; elsewhere the cycle is written in generic form for the post-processor to map.

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

> **[TODO Jason: personal experience]** Which machines and options you run (air jet, through-tool
> air, through-spindle coolant pressure, chip fan, wash gun, conveyor), how you clear chips today
> (operator with an air gun, M-codes added by hand, fan tool), where chips have cost you parts or
> tools (recut chips in pockets, scratched finished faces, a chip under the part on reload, a
> wrong tool length from a chip on the setter), and how you run coolant with indexable carbide.
> Label this as personal experience.

## Summary

1. **Most of these practices are well known but live in people's heads, not in CAM.** Tool makers
   publish the rules (roll into the cut, thick to thin, dry or constant coolant for carbide in
   interrupted cuts), machine builders sell the hardware (air jets, through-tool air, chip fans,
   wash-off tools), but the programmer adds the codes by hand and the simulation does not know
   chips exist. SolidCraft can make them defaults.
2. **Coolant on carbide is a timing rule, not a yes/no rule.** Thermal cracking comes from
   temperature swings: ISCAR lists "variations in coolant supply" and "intermittent machining"
   as causes and the remedy as "significantly increase coolant supply or turn it off"
   ([ISCAR](https://www.iscar.com/products.aspx/countryid/1/productid/12356)). A shop case in
   American Machinist found that running dry doubled tool life in an interrupted cut, because
   water "can increase the thermal cracking problem"
   ([American Machinist](https://www.americanmachinist.com/cutting-tools/overcoming-interrupted-cut-blues)).
   So the rule CAM applies is: for coated carbide inserts in interrupted steel or cast-iron
   milling, default to **dry or air**; if coolant is used, it is **full flood, on before entry,
   and never switched while the tool is cutting.** The simulation checks the coolant timeline
   move by move.
3. **Roll in, thick to thin.** Sandvik Coromant (quoted in SME's *Manufacturing Engineering*):
   rolling into the cut "will create a chip thickness on exit that is always zero", and if the
   tool must go straight in, "the feed reduced to 50 percent until the cutter is fully engaged"
   ([SME, 2021-10](https://www.sme.org/technologies/articles/2021/october/toolpath-tips-and-tricks-for-metal-cutting/)).
   These become default entry and exit rules with a check for thick exit chips.
4. **Ramped lead-ins and lead-outs (Jason's idea) are a feed profile, not a single entry feed.**
   CAM can compute the real chip thickness along the lead-in arc and set the feed block by block
   so the load rises smoothly to the cutting value, never dropping below the minimum chip
   thickness (rubbing). Changing spindle speed in the cut is possible only where the machine's
   spindle acceleration is known; the safer default is to set the speed before entry and ramp
   feed only.
5. **Chips can be simulated cheaply.** Every move already knows the volume it removes. Multiply by
   a bulk factor for the chip form, assign the chips to the region they land in (a closed pocket,
   an open face, the table, the vise, the tool setter), and subtract what each clearing method
   removes. Where chips accumulate above a threshold, CAM inserts a clearing cycle, and the
   simulation flags recutting, marring of finished faces and chips on datum and setter surfaces.
6. **Cleaning is a set of real operations with real codes.** Haas sells chip fans run from the
   spindle (5,000 to 12,000 rpm, started at 1,800 to 2,000 rpm and ramped up) and wash-off tools
   (start at 50 rpm, run at 100 rpm, with through-spindle coolant first) and ships a Table Wash
   template in its control ([Haas Tooling, fans](https://www.haastooling.com/c/cnc-clearing-fans);
   [Haas Tooling, wash-off tools](https://www.haastooling.com/c/parts-wash-off-tools-kits)).
   SolidCraft can make chip clear, table wash, part wash, part dry, camera clean, setter clean and
   tool clean standard operations, each with a machine-library capability, a time cost and an
   effect on the simulated chip state.
7. **Cameras need cleaning cycles too, and some products solve it with hardware.** Rotoclear's
   machine-interior camera looks through a "fast-rotating viewing window" that "flings droplets
   and solid particles outward"
   ([Foundry Planet](https://www.foundry-planet.com/d/seeing-is-understanding-new-camera-system-provides-optical-information-from-inside-the-machine-for-machining-process-optimization/)).
   A SolidCraft Edge camera without that window needs an air purge, a rinse-and-dry cycle, and an image
   check that says when the view is too dirty to trust.

## Fundamentals

### Why these are rules, not tips

Each practice below has three parts, and CAM needs all three (opinion):

| Part | What it is | Who owns it |
|---|---|---|
| **Rule** | the condition and the action ("interrupted cut + coated carbide insert + steel → dry or full constant flood") | this file; stored as data so a shop can change it |
| **Generator** | the toolpath or cycle that applies the rule (a roll-in arc, a feed ramp, a chip clear cycle) | CAM operations |
| **Check** | the simulation finding when a program breaks the rule, with severity and the move where it happens | the simulation gate in the CAM Physics System file |

A rule without a check is a default anyone can undo by hand-editing; a check without a generator
is a list of complaints the programmer must fix. So every rule here comes with both.

Severities are the gate's: **Stop** (would break or move something), **Warn** (shorter tool
life, worse finish, recut risk), **Info** (assumed data used). Shop rules in this file are
mostly **Warn**; the few **Stop** items are named.

### Coolant and carbide: why timing matters

Carbide is hard but has low toughness and moderate thermal conductivity, so a sharp temperature
change across the edge puts its surface in tension and cracks it; in milling every tooth heats in
the cut and cools in the air once per revolution, and coolant makes the cooling side of that
cycle much sharper (general metal-cutting knowledge; magnitudes not quantified here). The
cracks run **across** the edge ("perpendicular to the cutting edge", American Machinist), then
pieces break out: the chipping Jason wants to prevent.

Sources agree on the cause and the two cures:

- ISCAR's milling insert troubleshooting lists thermal cracking causes as "excessive variations
  in surface temperature, intermittent machining, or variations in coolant supply", with the
  remedy "significantly increase coolant supply or turn it off"
  ([ISCAR](https://www.iscar.com/products.aspx/countryid/1/productid/12356)).
- The American Machinist case (Seco coated insert, interrupted cut) ran dry and doubled tool life
  to 15 parts per corner, with the failure moving to normal flank wear
  ([American Machinist](https://www.americanmachinist.com/cutting-tools/overcoming-interrupted-cut-blues)).
- Sandvik Coromant has a page on the same question, "Dry milling or with cutting fluid"
  ([Sandvik Coromant](https://www.sandvik.coromant.com/en-us/knowledge/milling/dry-milling-or-with-cutting-fluid));
  its content did not load for this research, so it is listed as further reading, not quoted.

Where coolant is the right choice even with carbide (fact for direction; per-case data comes
from the tool maker's row):

- **Aluminium and other sticky materials:** lubrication and flushing stop chips welding to the
  edge; the cutting physics file notes ISCAR's point that too cool a zone causes built-up edge,
  so this is about lubrication and chip flushing, not cooling (opinion).
- **Stainless, titanium, heat-resistant alloys:** heat at the edge dominates; high-pressure
  through-tool coolant is normal practice (unverified per grade).
- **Drilling, tapping, deep slots:** chip evacuation dominates.
- **Solid carbide end mills:** many vendors publish wet and dry rows; follow the row.

So the rule is per tool, material group and operation, and the default is in the **tool's
cutting data row** (cutting physics file stores cutting data "per coolant mode"). The CAM rule
below is what happens when the row is silent.

### Roll in, thick to thin

In climb (down) milling the tooth enters at maximum chip thickness and leaves at zero; in
conventional (up) milling it is the reverse. Sandvik's "golden rule in milling is 'thick to
thin'" and "a thick chip on exit creates a sudden change in load from compressive to tensile",
which damages the carbide edge
([SME, 2021-10](https://www.sme.org/technologies/articles/2021/october/toolpath-tips-and-tricks-for-metal-cutting/)).

The same article gives three entry rules from Sandvik:

- **Roll into the cut** (an arc, turning in the direction that gives climb milling) "at 100
  percent feed rate"; this makes the exit chip thickness "always zero". In Inconel 718 this
  "extends tool life up to seven times longer, without reducing MRR".
- **If going straight in**, "the feed reduced to 50 percent until the cutter is fully engaged".
- **Engagement:** "ae = 70 percent × DC" or "25 percent or less" for the arc of engagement in
  face milling; the point is to avoid the cutter centre line sitting on the workpiece edge,
  where the entry is at the insert tip and the exit chip is thick (interpretation; the figure
  caption alone was read).

### Feed and speed through a lead-in (Jason's idea)

Most CAM systems let the programmer set one "entry feed" and one "exit feed" (unverified per
product), so the feed steps twice: lead-in feed to cutting feed, then cutting feed to lead-out
feed. Jason's point: the step itself is a load step. Ramp instead.

**What actually changes along a roll-in arc (derived).** For a cutter of radius `R` rolling into
a straight wall with a lead-in arc, the radial engagement `ae` grows from 0 to the cutting value,
and so does the engagement angle `φ`. The maximum chip thickness of a tooth is

`hex = fz · sin(φ)` for `ae < R` (radial chip thinning, as in the cutting physics file),

so at a fixed `fz` the chip starts thin and grows; the force grows with it. That is why rolling
in is gentle even at 100 % feed. Two problems remain:

1. **At the very start the chip is thinner than the edge can cut** (`hex < hmin`, the minimum
   chip thickness), so the edge rubs: heat, work hardening (stainless, titanium) and wear. A
   *higher* feed per tooth at first contact keeps `hex` at or above `hmin`, then the feed
   comes down to the cutting value as `φ` grows. This is chip-thinning compensation applied
   along the arc instead of once per operation.
2. **The rate of load rise** matters for an indexable cutter (each insert takes the full step of
   load the first time it enters). A smooth rise over several tooth passes is gentler than a
   rise over one (opinion; no source quantifies it).

**Feed profile (opinion on method):**

- Compute `φ(s)` along the lead-in arc from the in-process stock (the CAM Physics System's inner
  loop already does this per segment).
- Target chip thickness `h*(s)` rises smoothly from `max(hmin, h_entry)` to the cutting value
  `h_cut` over a length of at least `k` tooth passes (`k` a rule parameter, for example 5,
  unverified), with an S-curve (smooth start and end) rather than a straight ramp.
- Feed per tooth `fz(s) = h*(s) / sin(φ(s))`, capped by the force, power and deflection limits
  of the inner loop and by the machine's feed acceleration.
- Emit it as short blocks (one feed per block) or, on controls that support it, a feed that
  varies continuously along one arc (unverified per control). Look-ahead will smooth short
  blocks anyway; the blocks must be long enough not to starve the control's block rate (machine
  library field).
- Mirror it on the lead-out: keep `hex ≥ hmin` as the tool rolls out, then lift.

**Matching RPM (Jason's idea).** Ramping spindle speed with the feed keeps the feed per tooth
and the surface speed in step, which is attractive in theory, but three facts limit it:

- A spindle takes time to change speed; during acceleration the real speed and the commanded
  speed differ, so a feed computed from the commanded speed is wrong (fact in principle; the
  spindle acceleration figure belongs in the machine library and is often not published,
  unverified).
- Some controls wait for "spindle at speed" before the next feed move, which would make a pause
  in the cut: the worst case (unverified per control and parameter).
- Controls already vary speed in the cut for another reason: **spindle speed variation** (SSV),
  used against chatter, is offered on some controls and is an active research topic
  ([Politecnico di Milano, sinusoidal SSV](https://re.public.polimi.it/handle/11311/968962)).

So the default rule (opinion): **set the cutting speed before the lead-in starts (in air), ramp
feed only.** A machine whose library entry has a measured spindle acceleration and confirmed
no-wait behaviour may enable speed matching: the feed of each block is then computed from the
*predicted actual* speed at that point, not the commanded one, so feed per tooth is right even
while the spindle accelerates. The simulation checks that `fz` stays inside the tool's range
throughout.

### Chips: where they go and why they matter

A chip left in the machine can do four kinds of damage (fact for the kinds; frequencies are
not sourced here):

| Where the chip is | What goes wrong | Typical timing |
|---|---|---|
| In the cut (pocket floor, slot, hole) | Recut: the tooth hits a hardened, hot chip; edge chipping, poor finish, early wear. "A primary contributor to early onset tool wear" ([ETMM / Haas Automation Europe](https://www.etmm-online.com/optimum-cooling-for-a-better-finish-longer-tool-life-a-508424/?p=2)) | during roughing in closed features |
| On a finished face | Marring or scratching when a later tool, a coolant stream or a probe drags it; dents under clamps | finishing passes, second ops |
| On a datum or clamping face (vise jaw, fixed jaw, pallet, part rest) | Part not seated: size and position errors on the next part, or after a flip (the inspection file lists "chip under it" as the main seating risk) | part load, flip, re-clamp |
| On or near the tool setter, probe or camera | Wrong tool length or false broken-tool alarm; probe triggers on a chip; camera cannot see | tool setting, probing, monitoring |

The machine-side facts that matter:

- Gravity helps a horizontal machining centre and hurts a vertical one: on a vertical spindle,
  chips in a closed pocket stay there until something lifts them out (general knowledge).
- Air from outside (an air jet, a fan) clears open surfaces but does not reach the bottom of a
  deep pocket; through-tool air or coolant does (cutting physics file; Haas recommends its
  through-tool air blast for "deep-hole pocketing" and says re-cutting chips "is bad"
  ([Haas, TAB](https://www.haascnc.com/productivity/chip-and-coolant-management/tab.html))).
- Chip **form** decides how much space chips take and how easily they move: short broken chips
  flow with coolant; long strings nest and wrap the tool. Conveyor makers classify chips this
  way ("short chips" under 2 in that "do not ball up", "long strings", "nested chips")
  ([Mayfran selection guide](https://www.mayfran.com/wp-content/uploads/2023/05/Conveyor-and-Filtration-Selection-Guide.pdf));
  ISO 3685 classifies chip forms (ribbon, tubular, spiral and others)
  ([PMPA](https://www.pmpa.org/?p=13351)).
- Loose chips occupy many times the volume of the metal they came from; the ratio depends on
  chip form and was not found in a public source for this file (unverified; a calibration value
  per material and chip form, see open questions).

## Accuracy

- **Coolant rules** rest on two public sources and a widely shared mechanism; they are
  directionally solid but the size of the effect varies by grade, coating and material. The tool
  maker's row overrides the default.
- **Entry rules** are Sandvik's as reported by SME; the 50 % straight-entry feed and the "ae 70 %
  or ≤ 25 %" figures are rules of thumb, not physics limits.
- **Lead-in feed profile** is derived from the standard chip thinning relation; `hmin` per edge
  and the number of tooth passes for the ramp are not published values here and need tool-maker
  data or testing (Self-tuning CAM Operations in the inspection file is the place to learn them).
- **Chip build-up** at the levels proposed here is a bookkeeping model, not a physical
  simulation of chip flight or coolant flow. It will be wrong in detail; its job is to say
  *where* and *when* chips are likely to matter, and to be corrected by cameras. Full chip and
  coolant flow simulation exists in research (CFD coupled with particle models, mostly for deep
  hole drilling, for example
  [arXiv 2508.02333](https://arxiv.org/pdf/2508.02333)) and is far too slow for a CAM check.
- **Camera chip detection** accuracy depends on lighting, coolant mist and window cleanliness;
  no accuracy figure is claimed.

## Programs

What exists today, from public pages only (no program was run for this file):

- **Machine builders** sell the hardware and some cycles: Haas through-tool air blast "activated
  by M-code" ([Haas, TAB](https://www.haascnc.com/productivity/chip-and-coolant-management/tab.html));
  Haas mill M-codes for coolant (M08/M09), programmable coolant spigot position (M34/M35), auto
  air jet (M83/M84), through-spindle coolant (M88/M89) and chip auger (M31/M33)
  ([summary of Haas mill M-codes, mfgrobots](https://www.mfgrobots.com/Article/equipment/cnc/46313.html);
  confirm against the Haas operator's manual for each control); Haas chip fans and wash-off
  tools with a built-in Table Wash template on NGC controls (see Summary).
- **Tool makers** sell spindle-mounted chip fans for unmanned cleaning: Lang Technik's Clean-Tec
  160 is held in a 20 mm holder, called "from the NC program as a final step", with wings that
  open with speed between 6,000 and 12,000 rpm
  ([Lang Technik](https://lang-technik.de/en/products/30160-clean-tec-160-chip-fan-shaft-o-20-mm)).
- **Tool setter makers** clean the tool before measuring: Renishaw's NC4 "built-in air blast
  efficiently removes machining debris and coolant from the tool before measurement", and the
  NC4+ Blue can "detect burrs or attached swarf"
  ([Renishaw NC4](https://www.renishaw.com/en/nc4-non-contact-tool-setter--6099)).
- **Machine builders and patents** clean tool interfaces at tool change with air: a patent
  describes air blown through the tool receiver that cleans the mating faces as the tool is
  inserted and then checks seating by pressure
  ([US4864714A](https://patents.google.com/patent/US4864714A/show.php)).
- **Camera makers** keep the view clear with a spinning window (Rotoclear, see Summary).
- **CAM programs** let the programmer pick a coolant mode per operation and set lead-in and
  lead-out geometry and an entry feed (common knowledge; details unverified per product). No
  public page found for this file describes a CAM program that predicts chip build-up or inserts
  chip clearing cycles automatically.

## Complaints and requests

(Paraphrased; widely reported in trade press and forums, not individually sourced here.)

- Chip clearing is added by hand: the programmer types air or coolant codes into the program or
  the operator stops the machine and uses an air gun, which defeats unattended running.
- Recut chips in deep pockets break tools, especially small end mills, and the CAM simulation
  shows a clean pocket.
- Coolant is switched on and off by habit (on for roughing, off for a probing move, on again)
  without thinking about the insert.
- Lead-ins are geometric only; the programmer guesses an entry feed.
- After an unattended run the part is covered in chips and coolant, so the robot or the next
  operator has to clean it first; chips on the vise ruin the next part's seating.
- Cameras inside the machine are blind within minutes from coolant mist and chips.

## What users need

### Rule set 1: coolant and carbide inserts (Jason's rule)

| # | Condition | Action CAM takes | Check (severity) |
|---|---|---|---|
| C1 | Indexable carbide (coated or not) in an **interrupted** milling cut (face, shoulder, slot entering and leaving the material per revolution) in ISO P or K material, and the tool's cutting data row has no wet row | Default coolant mode **dry**, with outside air or through-tool air for chip clearing if fitted | Coolant on in this case: **Warn** "thermal cracking risk", unless the row allows wet |
| C2 | Coolant chosen for a carbide tool (any reason) | **Flood** (or through-tool) at full flow, switched on **before** the lead-in starts, with a dwell long enough for flow to reach the tool (machine library field, for example 1 to 2 s, unverified) | Coolant switched **on or off while the tool is engaged**: **Warn** (Stop on a preference); coolant starting after the first contact: **Warn** |
| C3 | Same, but the nozzle cannot reach the edge all the time (spigot aimed for a shorter tool, pocket wall shading the stream, P-cool not programmed) | Use the programmable spigot position for the tool length, or through-tool coolant | Partial coolant: **Warn** "edge passes in and out of the stream" (geometric check of the nozzle line against tool and stock, level 2) |
| C4 | Ceramic, CBN or cermet inserts | Follow the tool row; if silent, dry (commonly stated, unverified per grade) | Coolant on without a wet row: **Warn** |
| C5 | Aluminium, stainless, titanium, HRSA; drilling, tapping, deep slots | Coolant per tool row (flushing and lubrication); through-tool where fitted | Dry in a deep closed feature: **Warn** "chip evacuation" (feeds chip model) |
| C6 | An operation switches between dry and wet compared to the previous one with the **same** tool | Insert a cooling or drying gap with the tool out of the cut, or keep the mode (opinion) | Mode change between consecutive cuts with a hot tool: **Info** |

Notes:

- These are rules for the case where the tool data is silent. The tool maker's row always wins
  (cutting physics file: "store life constants and cutting data per coolant mode").
- The coolant mode selects the cutting data row and the tool life constants, so changing the mode
  changes feeds; the user sees one choice, not two settings to keep in step.
- "Never switch coolant while engaged" is the simplest rule a post-processor can enforce: the
  coolant code moves to the retract or the clearance move before the lead-in.

### Rule set 2: entry, exit and engagement (roll-in and roll-off)

| # | Condition | Action | Check |
|---|---|---|---|
| E1 | Milling with carbide | Climb milling by default (thick to thin); conventional only by choice (thin walls, hard skin, backlash machines) | Conventional cut with an indexable carbide tool: **Info** |
| E2 | Entry from outside the stock | **Roll in** on a tangential arc that turns in the climb direction, arc radius at least the tool radius (opinion) | Straight entry at full feed into a full-width or ≥ 50 % engagement: **Warn**; with the 50 % feed rule applied: no finding |
| E3 | Entry that cannot roll (no room for an arc) | Straight entry with feed at 50 % until fully engaged (Sandvik), or a ramped profile (rule set 3) | — |
| E4 | Exit | Roll off on an arc so the exit chip thins to zero; avoid exiting with the cutter centre on the edge | Exit chip thickness above a fraction of `hex` at an edge (computed from the in-process stock): **Warn** "thick exit chip" |
| E5 | Face milling across an edge | Offset the cutter so that `ae` is about 70 % of diameter, not centred on the edge (Sandvik figure caption) | Cutter centre within a band around the workpiece edge on entry: **Info** |
| E6 | Corners and changes of engagement inside the cut | Arc toolpaths that keep engagement steady (adaptive clearing, cutting physics file); feed set per segment | Engagement jump above a limit (for example from 30 % to 100 % at an inside corner, unverified threshold): **Warn** with the move |
| E7 | Plunge into solid | Ramp or helix within the tool's ramp angle (tool library field) | Plunge with a non-centre-cutting tool: **Stop** |

### Rule set 3: ramped feed and matched speed on lead-in and lead-out (Jason's idea)

- **R1 (default on):** lead-in and lead-out feed is a **profile**, not a number. CAM computes it
  from the engagement along the lead (method in Fundamentals): start at a feed per tooth high
  enough to keep the chip at or above `hmin`, rise or fall smoothly to the cutting value over at
  least `k` tooth passes, capped by every limit the inner loop already checks.
- **R2:** no feed step larger than a set fraction (for example 20 %, unverified) between
  consecutive cutting blocks unless the engagement steps too (a step in engagement already steps
  the load; a feed step on top doubles it).
- **R3:** the spindle reaches cutting speed **before** the lead-in starts. The spindle speed
  command moves to the approach move, with a dwell from the machine's spindle acceleration if the
  control does not wait by itself.
- **R4 (opt-in per machine):** speed matching during the lead. Allowed only when the machine
  library has (a) a spindle acceleration figure (measured, for example from SolidCraft Edge, or from
  the builder), and (b) a confirmed "does not wait for spindle at speed during feed moves"
  behaviour. Feed per block is computed from the predicted actual speed.
- **Checks:** `fz` out of the tool's range anywhere in a lead (**Warn**); `hex < hmin` for more
  than a set number of tooth passes (**Warn** "rubbing", raised to a stronger warning for
  work-hardening materials); spindle speed change inside a cutting move on a machine without R4
  (**Warn**); feed step without engagement step (**Info**).
- **What the user sees (opinion):** one lead-in style ("Roll in, smooth") with no feed number to
  type, and in the simulation a small chart of chip thickness and load along the lead, so the
  smoothness is visible.

### Rule set 4: chip clearing cycles between cuts

**When CAM inserts a clearing cycle (opinion; thresholds are rule data, not constants):**

| Trigger | Why |
|---|---|
| The chip model says a closed region's chip load is over its threshold (below) | recutting |
| Before a **finishing** pass in a region where roughing left chips | recut chips scratch the finish; this is the cheapest clear to justify |
| Before **probing** a feature, and before **tool setting** | a chip under the stylus or on the setter is a measurement error |
| Before a **tool change** into a region with chips, when the next tool is small or fragile | small end mills break on chips first |
| Before **unclamping, flipping or reloading** | chips on datum and clamping faces |
| At the **end of the program** | wash and dry (rule set 5) |

**Cycle types (what each does in the machine; codes per machine library and post-processor):**

1. **Through-tool air sweep (in the pocket).** Retract above the floor, switch to through-tool
   air, sweep the region at a small height above the floor, spiralling **toward the open side or
   the drain direction** so chips are pushed out, not around; spindle at low speed or stopped per
   the tool maker (unverified). Best for deep closed pockets (Haas recommends through-tool air for
   deep pocketing).
2. **Outside air jet pass.** Auto air jet on (Haas M83, off M84), the tool passes over the region
   at clearance height in a pattern that moves chips toward the open side. Good for open faces
   and shallow features; weak in deep pockets.
3. **Coolant flush.** Through-spindle or flood coolant aimed into the region during a non-cutting
   sweep. Strong for heavy chips and tapped holes (Haas recommends through-spindle coolant to
   "pre-clean" chip piles and swarf in tapped holes). Not used on a hot carbide tool that is
   about to cut dry (rule C6).
4. **Chip fan pass.** Tool change to a fan, start the spindle low and ramp to the fan's operating
   speed (Haas: start at 1,800 to 2,000 rpm for 0.5 s, then ramp; long fan 5,000 to 8,000 rpm,
   short fan 6,000 to 12,000 rpm), pass over the table and part at the fan's offset, stop, return
   the fan. Best for the whole table and open fixtures; for the end-of-program clean, and between
   operations on unattended runs. Fans are held in the magazine like tools and carry their own
   speed limits in the tool library (minimum speed to open, maximum speed **Stop** limit).
5. **Wash-off tool pass.** A rotating wash-off tool at low speed (Haas: start at 50 rpm, run at
   100 rpm) with through-spindle coolant first, sweeping chips off fixtures and pallets.

**What CAM writes (generic form, for the post-processor to map; opinion):**

```
; CHIP_CLEAR  region=Pocket-3  method=THROUGH_TOOL_AIR  reason=chip_load 0.42 > 0.30
; tool already in spindle (T5, 10 mm end mill), cutting finished in Pocket-3
G0 Z<clearance>                 ; leave the cut
<coolant off>                   ; rule C2: coolant changes only out of the cut
<through-tool air on>           ; e.g. a machine-specific M-code from the machine library
<dwell for air to reach tool>   ; machine library: air delay
G1 Z<floor + 3 mm> F<sweep feed>
G1 <spiral outward toward open side, 70 % tool diameter stepover>
G0 Z<clearance>
<through-tool air off>
```

```
; CHIP_CLEAR  region=Table  method=CHIP_FAN  reason=end of roughing, unattended run
T<fan> M06
S1800 M03                       ; Haas fan instruction: start at 1,800-2,000 rpm for 0.5 s
G4 P0.5
S<fan operating speed>          ; ramp within the fan's range, never above its maximum
G0 <pass pattern at fan offset, moving chips toward the conveyor side>
M05
```

Every cycle is an operation in the CAM timeline with a time cost (shown in the cycle time), a
machine requirement (if the machine has no through-tool air, CAM picks the next method or warns),
and an **effect on the chip model** (below), so the simulation knows the region is clear.

### Predicting chip build-up in the simulation (Jason's idea)

**Level 0: chip bookkeeping (opinion; cheap enough to run on every move).**

1. **Chips made.** Each move removes a known volume `ΔV` from the in-process stock (the
   simulation already computes it). Loose chip volume `Vc = B · ΔV`, where `B` is a bulk factor
   for the material and chip form (a calibration value; see open questions). Chip form comes
   from the tool and cutting data (chip breaker, chip thickness), defaulting to "short" for
   milling.
2. **Where they go.** Each move is classified by the in-process stock around it:
   - **closed** (pocket, slot or hole below the surrounding stock, no open side within reach):
     chips stay in that region;
   - **open** (open face, open-sided pocket, on a horizontal spindle where chips fall clear):
     a fraction leaves the part and lands on the table or fixture below and around it;
   - a fraction is thrown toward the machine enclosure (opinion; one number per machine).
   Regions are features of the in-process stock (each closed pocket is one region), the fixture's
   surfaces, the table zones, and named zones from the machine library: **tool setter**, probe
   rest, camera field of view, vise datum faces, conveyor or drain.
3. **What removes them.** Each region has an evacuation rate per clearing mode active during the
   move: through-tool coolant (high), flood (medium in open, low in deep closed), through-tool air
   (medium), outside air jet (low in closed), fan (high on open surfaces, none in closed), gravity
   (high on horizontal spindles). Cleaning cycles remove a set fraction from the regions they
   sweep. Rates are rule data, starting from conservative guesses and corrected by cameras.
4. **State.** Each region carries chip volume `C(t)` and a fill ratio `C / V_free`, where
   `V_free` is the free volume below the tool's working height in that region.
5. **Findings.**
   - Fill ratio above the **recut threshold** while a tool cuts there: **Warn** "recutting
     chips", with the move; CAM's fix: insert a clearing cycle before it (rule set 4).
   - Chips on a **finished face** (a face whose final pass is done) when a later operation moves
     over or near it, or when the part is unclamped: **Warn** "marring risk".
   - Chips in the **tool setter** or **probe** zone at a setting or probing move: **Warn**, fix:
     setter clean before measuring.
   - Chips on **datum or clamping faces** at unclamp, flip or reload: **Warn**, fix: datum clean
     cycle (air jet over jaws and rests).
   - Chips in the **camera field of view** when an image is to be taken: **Info**.
   - Total loose chips in the machine above the chip tray or conveyor capacity for an unattended
     run: **Warn**, fix: conveyor run or washdown mid-program.

**Level 1: a coarse grid (opinion).** The table and part split into a voxel grid (for example
5 mm cells); chips land in cells with a spread around the tool, and flushing moves them downhill
toward drains along the machine's floor slope (machine library). This shows *where* on a face
chips lie, so the marring check can name the face, and makes camera comparison per cell possible.

**Level 2: physics (research only).** Particle and CFD models of chip flight and coolant flow
(see Accuracy). Not for the CAM check; useful offline to tune the level 0 and 1 rates for a
machine type.

**Monitoring with cameras (Jason's idea).** SolidCraft Edge, the Raspberry Pi computer at the machine (inspection file), gets one
or more cameras inside the enclosure:

- **What it checks:** chip coverage per zone (table, vise jaws, tool setter, part faces), chips
  wrapped on the tool after a cut, coolant level in a pocket, part clean after the wash.
- **How (opinion):** images taken at program points CAM chooses (after a clearing cycle, before a
  setting move, at the end), with spindle stopped and flood off so mist does not blur the view;
  compared with a reference image of the clean zone taken at setup; coverage per zone as a
  number. A learned segmentation model can come later; image differencing against a clean
  reference is enough to start (unverified on real coolant scenes).
- **What it does with the answer:** if a zone is dirty when the model says clean, repeat the
  cleaning cycle once (the program carries a conditional branch through a macro variable
  SolidCraft Edge can set, on controls that allow writes; otherwise SolidCraft Edge raises an alarm the operator sees), and
  log the miss so the evacuation rates for that machine are corrected. This is Self-tuning CAM
  Operations applied to chips.
- **Safety principle (same as the inspection file):** SolidCraft Edge never moves the machine. It can
  set a flag the program reads at a defined point, or stop and alert; the cycle that runs is one
  CAM already wrote and the simulation already checked.

### Rule set 5: cleaning routines as standard operations (Jason's idea)

These become built-in CAM operations, available in every setup and added automatically by the
end-of-program and unattended-run rules. Each has a machine requirement and an effect on the
chip model.

| Operation | What it does | Needs (machine library) | Default trigger |
|---|---|---|---|
| **Chip clear** | rule set 4 cycles in a named region | air jet, through-tool air, TSC, fan or wash tool | chip model, before finishing, probing, setting |
| **Datum clean** | air jet or fan over vise jaws, rests, pallet locating faces | air jet or fan | before reload or flip |
| **Setter clean** | air blast on the setter, then on the tool (setters with built-in air, such as Renishaw NC4, do the tool side) | setter air; air jet | before each tool setting move |
| **Table washdown** | flood or through-spindle coolant sweep that pushes chips **toward the conveyor or chip trough**, then a fan or wash tool pass | coolant, conveyor position | end of program; mid-program when the chip model says tray is filling |
| **Conveyor run** | run the chip conveyor or auger (Haas M31/M33) for a time after washdown, then stop | conveyor | after washdown |
| **Part wash** | through-tool coolant into holes and pockets (tapped holes first), then flood over faces | TSC or flood | end of program, before probing finished features |
| **Part dry** | through-tool air into holes, air jet over faces, fan pass; drip dwell | air jet, TAB, fan | after part wash; before robot unload (cell automation) |
| **Camera clean** | air pulse on the camera window before each image; rinse and air dry after washdown | camera air line or nozzle; coolant | before imaging; after washdown |
| **Tool clean** | rule set 6 | per method | before a tool goes back to the magazine |

**End-of-program sequence (opinion, the default for unattended runs):**

1. Tool clean of the last tool, tool to magazine.
2. Part wash (holes first, then faces).
3. Table washdown toward the conveyor, conveyor run.
4. Part dry (through-tool air into holes, air jet, fan pass), drip dwell.
5. Camera clean, then camera check: part clean, jaws clean, setter clean.
6. If the check fails, repeat steps 2 to 5 once; if it fails again, stop and alert, do not
   unload (cell automation: a robot must not pick a wet, chip-covered part into the next
   machine).

The time each step costs is in the cycle time, and the user can shorten or switch off steps per
job; switching off the end clean on an unattended run gives a **Warn** naming what is left dirty.

### Rule set 6: cleaning a tool before it goes back (Jason's request)

**Why (fact for the risks; frequency unverified):** chips and coolant carried back on the tool
fall into the magazine pocket and onto the next holder's taper, and a chip on a taper or flange
face causes runout and a wrong length; aluminium chips welded to flutes cut badly next time; a
wet through-coolant tool drips on the part and the setter.

**Methods (from public sources and common practice):**

| Method | How | Limits | Source / status |
|---|---|---|---|
| **Coolant off, air purge through the tool** | switch through-spindle coolant off, through-tool air on for a short time to empty the coolant passage | needs through-tool air or a purge function | Haas TAB uses the TSC plumbing ([Haas](https://www.haascnc.com/productivity/chip-and-coolant-management/tab.html)); purge timing unverified |
| **Spin-off** | with coolant off and the tool clear of the part, run the spindle at a set speed for 1 to 3 s to throw coolant and loose chips (opinion on timing) | below the tool's `RPMX`, the holder's balance rating and the machine's maximum; never for fans or wash tools outside their range; large indexable cutters limited by their own rating (the slider never moves these limits) | common practice (unverified) |
| **Outside air blast during spin** | air jet on during the spin-off | air jet aimed at the tool length | Haas M83/M84 |
| **Wash then air** | through-tool or flood rinse to free stuck chips (aluminium), then air and spin-off to dry | adds time; not for a hot carbide tool about to cool fast (rule C6 spirit: the tool is out of the cut, so thermal shock is less of a concern, unverified) | opinion |
| **Taper and face cleaning at tool change** | air through the spindle or receiver during the tool change cleans the mating faces; some designs then check seating by air pressure | machine feature, not CAM | patent [US4864714A](https://patents.google.com/patent/US4864714A/show.php); per-machine availability unverified |
| **Check before storing** | laser setter or camera checks for attached swarf, wrapped strings, broken edge | setter or camera fitted | Renishaw NC4+ Blue "detect burrs or attached swarf" |

**Rule (opinion):** the tool clean is a short default sequence attached to every tool change:
coolant off → air purge (if through-coolant tool) → spin-off with air jet at a safe speed → (if a
setter or camera can see) swarf check. A wrapped string that spin-off cannot clear is a **Stop**
for unattended runs: the program should not put a tool wrapped in chips back in the magazine.

**Simulation check:** the spin-off speed is checked against tool, holder and machine limits like
any spindle speed (**Stop** above any of them); the spin-off position is checked for collisions;
the time is added to the cycle.

### What the machine library needs

Additions to the machine library file's schema (opinion):

- **Coolant and air options fitted:** flood, programmable spigot (and how it is aimed), shower or
  washdown, through-spindle coolant and its pressure, through-tool air, outside air jet, MQL, wash
  gun; each with its on/off codes, delay before it reaches the tool, and whether it may change
  during a move.
- **Chip handling:** conveyor or auger position and direction, chip tray capacity, floor slope or
  drain points, spindle orientation (vertical or horizontal).
- **Zones:** tool setter position and its air blast; probe rest; camera positions, fields of view
  and cleaning means; vise or pallet datum faces (from the workholding setup).
- **Spindle behaviour:** acceleration (rpm per s by range), and whether the control waits for
  "at speed" in feed moves (for rule R4).
- **Cleaning tools in the magazine:** fans and wash-off tools as tools with minimum and maximum
  speed, start speed and ramp, and pass offset.

### Where this fits in SolidCraft (opinion)

- **Rules are data.** Each rule set is a table of conditions and actions stored with the CAM
  library, editable per shop, versioned, and shown with its source. A shop that runs flood on
  inserts on purpose turns C1 off once, with the reason recorded (the same recorded-approval
  pattern as the CAM Physics System gate).
- **Generators are operations.** Roll-in, ramped feed and cleaning cycles are operation options
  or operations in the CAM timeline, each a command with input and output schemas (the repo's
  command rule), so an agent or a script can add a chip clear exactly as a person does.
- **Checks are simulation findings.** The chip model runs inside the same simulation as the
  physics checks; its findings use the gate's severities and appear on the setup sheet.
- **Slider:** rules here are mostly **Warn** and do not move with the conservative-to-aggressive
  slider, except thresholds that are about tool life (recut threshold, ramp length), which the
  slider may loosen. Speed limits on spin-off and fans never move.

### Phased plan (opinion)

1. **Coolant timeline and entry rules:** C1–C6 and E1–E7 as checks on any program, plus the
   post-processor rule that coolant changes only out of the cut. Small, no new physics.
2. **Ramped lead-in and lead-out feed:** uses the per-segment engagement the physics system
   already needs.
3. **Cleaning operations:** chip clear, washdown, part wash and dry, tool clean, setter clean as
   operations with machine-library requirements and post-processor codes; end-of-program default.
4. **Chip model level 0:** bookkeeping per region, findings, automatic chip clear insertion.
5. **Cameras on SolidCraft Edge:** reference images, coverage per zone, repeat-once branch, rate
   correction (self-tuning).
6. **Chip model level 1 and speed matching (R4)** where machines report what they need.

## Export control, sanctions and government use

*Added 2026-10-29 from the [government use research](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-qk5riz/research/gov-research-nsnlookupjn-government-use.md). Opinion and policy proposals, not legal advice.*

**Our three rules:**

- **No one goes to jail.** Our team has no in-house counsel. So wherever a rule is unclear we take the most conservative reading and refuse rather than warn. Questions that need an outside export-control lawyer are marked as such, and free official help (BIS, DDTC, OFAC) is the first call.
- **Help users comply.** SolidCraft stops and explains in plain language, using rule packs for each jurisdiction that the administrator picks. It flags conflicts between countries' laws, never resolves them, and never gives legal advice.
- **Prevention first.** Nothing in this file that is hosted, shared, paid or networked ships until its compliance gates are in place and checked: region blocks, screening, ITAR flag handling, government-mode behaviour, and an outside lawyer's sign-off for any hosted or paid service.

**What in this file crosses a border:** local milling strategies and checks.

- **No hosted service is proposed here.** If one is added later, it blocks comprehensively sanctioned regions, plus Russia and Belarus, and screens accounts and payers against the US Consolidated Screening List and the UK, EU and UN lists.
- **No support there.** No support, consulting or custom work for users in sanctioned regions or for listed parties.
- **Government mode.** At the Controlled level (CUI and allied equivalents) online parts are off or point to the organisation's own mirror; at the Isolated level they arrive only as signed packs on media; a design flagged Export controlled (ITAR/EAR or national equivalent) never leaves through them.
- **Only generic data is shared.** Nothing derived from a controlled design (geometry, toolpaths, part or drawing numbers, test or inspection results) goes into a shared library, report or issue. Tool data stays shareable through the field allow-list in the high-security research.
- **Code stays public.** Every build the project ships is public, which keeps it "published" software outside the EAR. The rules are tracked by the standing export and sanctions watch.
- **Checked for controlled data (2026-10-29):** this file contains no controlled technical data. Its examples are public sources, generic values or made-up stand-ins, and the author's experience is stated in general terms.

## Glossary

- **Climb (down) milling:** the tooth enters thick and leaves thin; the default with carbide.
- **Conventional (up) milling:** the tooth enters thin and leaves thick.
- **Roll in / roll off:** entering and leaving the cut along an arc so engagement grows and falls
  smoothly and the exit chip is thin.
- **`hex`:** maximum chip thickness of a tooth; **`hmin`:** the thinnest chip the edge can cut
  without rubbing.
- **Thermal cracking:** cracks across a carbide edge from repeated heating and cooling, which
  lead to chipping.
- **Recutting:** a tooth cutting a chip already made, instead of fresh material.
- **TSC / TAB:** through-spindle coolant / through-tool air blast.
- **Air jet:** an outside nozzle near the spindle blowing air at the tool and cut.
- **Chip fan:** a fan held in a tool holder and run in the spindle to blow chips off the table.
- **Wash-off tool:** a slow-turning tool used with coolant to sweep chips off fixtures.
- **Bulk factor `B`:** loose chip volume divided by the solid volume it came from.
- **SSV:** spindle speed variation, a control function that varies speed in the cut against
  chatter.

## Open questions

1. **Bulk factor by chip form.** Is there a public table of loose chip volume per solid volume
   for common chip forms (short broken, curled, long strings) in steel and aluminium? Conveyor and
   chip processing makers likely have it; none was found for this file. Without it, `B` starts as
   a shop calibration value.
2. **Minimum chip thickness `hmin` per edge.** Tool makers publish minimum feeds per tooth for
   some tools; is there a usable general rule (for example a fraction of the edge radius) for
   when data is missing?
3. **Spindle acceleration and "at speed" waiting** on common controls (Haas NGC, Fanuc, Siemens,
   Heidenhain): which publish it, which parameters control the wait? This decides whether rule R4
   is practical.
4. **Through-tool air codes and purge** on Haas and other controls: the Haas TAB page says "by
   M-code" without naming it; the operator's manual should be checked before a post-processor
   uses one.
5. **Camera placement and cleaning:** is an air purge plus rinse-and-dry cycle enough for a
   low-cost camera, or does a spinning window (as on the Rotoclear product) become necessary in
   flood machines?
6. **Maintainers:** should cleaning operations live in the CAM timeline as operations (visible,
   editable, timed) or as post-processor options? This file recommends operations, so the
   simulation can see them.

## Sources

Accessed 2026-10-26 unless noted.

### Tool makers and trade press

- ISCAR, milling inserts wear problems and solutions:
  <https://www.iscar.com/products.aspx/countryid/1/productid/12356>
- American Machinist, "Overcoming interrupted cut blues":
  <https://www.americanmachinist.com/cutting-tools/overcoming-interrupted-cut-blues>
- SME *Manufacturing Engineering*, "Toolpath tips and tricks for metal cutting", October 2021
  (Sandvik Coromant guidance):
  <https://www.sme.org/technologies/articles/2021/october/toolpath-tips-and-tricks-for-metal-cutting/>
- Sandvik Coromant, "Dry milling or with cutting fluid" (further reading; content not loaded):
  <https://www.sandvik.coromant.com/en-us/knowledge/milling/dry-milling-or-with-cutting-fluid>
- ETMM / Haas Automation Europe, "Optimum cooling for a better finish, longer tool life",
  2015-10-20: <https://www.etmm-online.com/optimum-cooling-for-a-better-finish-longer-tool-life-a-508424/?p=2>
- PMPA, chips and ISO 3685 chip forms: <https://www.pmpa.org/?p=13351>
- Mayfran, Conveyor and Filtration Selection Guide:
  <https://www.mayfran.com/wp-content/uploads/2023/05/Conveyor-and-Filtration-Selection-Guide.pdf>

### Machine and accessory makers

- Haas Automation, Through-Tool Air Blast (TAB):
  <https://www.haascnc.com/productivity/chip-and-coolant-management/tab.html>
- Haas Tooling, CNC clearing fans: <https://www.haastooling.com/c/cnc-clearing-fans>
- Haas Tooling, parts wash-off tools and Table Wash template:
  <https://www.haastooling.com/c/parts-wash-off-tools-kits>
- Haas mill M-code summary (third-party; check against the Haas manual):
  <https://www.mfgrobots.com/Article/equipment/cnc/46313.html>
- Lang Technik, Clean-Tec 160 chip fan:
  <https://lang-technik.de/en/products/30160-clean-tec-160-chip-fan-shaft-o-20-mm>
- Renishaw, NC4 non-contact tool setter:
  <https://www.renishaw.com/en/nc4-non-contact-tool-setter--6099>
- Foundry Planet, Rotoclear camera system:
  <https://www.foundry-planet.com/d/seeing-is-understanding-new-camera-system-provides-optical-information-from-inside-the-machine-for-machining-process-optimization/>
- US patent 4864714A, tool changing device with cleaning:
  <https://patents.google.com/patent/US4864714A/show.php>

### Research

- Politecnico di Milano, analytical optimisation of sinusoidal spindle speed variation in
  milling: <https://re.public.polimi.it/handle/11311/968962>
- Numerical and experimental evaluation of chip evacuation and lubricant flow in ejector deep hole
  drilling (example of chip and coolant flow simulation): <https://arxiv.org/pdf/2508.02333>
