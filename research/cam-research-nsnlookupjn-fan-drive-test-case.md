# CAM research: a fan drive assembly as an end-to-end test case

Research for [#1621](https://github.com/tangency-labs/solidcraft/issues/1621) (Research Request
for Plugin: CAM). This file proposes **one real-world product, a fan drive assembly, as a
shared test case** for the CAM, inspection, hardware and assembly ideas in my other research
files. A fan drive has almost every step a small manufacturer sees in one product: **castings,
milling, turning, purchased hardware, heat treatment, painting, and an assembly that is set with
shims to hold a tolerance**. If SolidCraft can carry this one product from model to a finished,
inspected, shimmed assembly, it can carry most small-shop work.

The file does not repeat the physics in the other topics. For each manufacturing stage it says
which topic the stage exercises, what SolidCraft would need, and how we would know it works. It
ends with a **staged validation plan**: small steps, each useful on its own, that build up to the
whole assembly.

Licence: MIT OR Apache-2.0, the same as SolidCraft (see the Licence section of CONTRIBUTING.md).

## About this research

- **Author:** Jason ([@nsnlookupjn](https://github.com/nsnlookupjn)). The idea of using the fan
  drive assembly as the test case is Jason's; the stage breakdown, the plan and the writing were
  done with an AI assistant (Claude) in October 2026.
- **What is and is not here:** the fan drive is a product Jason's company makes. **No proprietary
  information is in this file.** The part descriptions below are a *generic* fan drive, of the
  kind described in public trade literature, and every company-specific value is a
  **[TODO Jason]** placeholder. Section "A public stand-in design" proposes how the test case can
  be shared without releasing the real design.
- **How it was produced:** from public engineering knowledge and standards cited by name in
  Sources. No commercial CAD/CAM program was driven or scripted, no GPL/LGPL/AGPL source was read,
  and no text is copied beyond short quotes.
- **Units:** millimetres, newtons, newton-metres, degrees Celsius, HRC for hardness, µm for paint
  film thickness.
- **Marking:** "(unverified)" marks a claim not confirmed against a source. **Fact**, **opinion**
  and **Jason's idea** are labelled where they mix. The plan in "What users need" is opinion: a
  proposal for discussion, not a decision.

> **[TODO Jason: personal experience]** What a fan drive goes through on your floor in one or two
> sentences, and one problem that crossed departments: for example a casting that machined fine
> but moved after heat treatment, paint that got into a bore, or a shim stack that came out
> different on the line than on paper. Label this as personal experience.

## The related research files

| Topic | File | Fan drive stage that exercises it |
|---|---|---|
| CAM Physics System (summary) | [physics-system](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-vgqo3g/research/cam-research-nsnlookupjn-physics-system.md) | All machining stages |
| Shared cloud tool library | [tool-library](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-tool-library/research/cam-research-nsnlookupjn-tool-library.md) | Milling, turning |
| Cutting tool physics and the slider | [cutting-physics](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-cutting-physics/research/cam-research-nsnlookupjn-cutting-physics.md) | Milling, turning, cast skin |
| Workholding physics and library | [workholding](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-workholding/research/cam-research-nsnlookupjn-workholding.md) | First op on a casting, shaft between centres |
| Stock structural integrity | [stock-integrity](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-stock-integrity/research/cam-research-nsnlookupjn-stock-integrity.md) | Thin housing walls, cored casting |
| Machine library | [machine-library](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-machine-library/research/cam-research-nsnlookupjn-machine-library.md) | Mill and lathe |
| Machine health and failure prediction | [machine-health](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-9w3gq9/research/cam-research-nsnlookupjn-machine-health.md) | Long production runs |
| In-process inspection and part records | [in-process-inspection](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-y2w06n/research/cam-research-nsnlookupjn-in-process-inspection.md) | Every stage; the traveller |
| Cell automation | [cell-automation](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-g08nas/research/cam-research-nsnlookupjn-cell-automation.md) | Loading, tool setting |
| NSN and standard part library | [nsn-library](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-nc3c94/research/parts-research-nsnlookupjn-nsn-library.md) | Bearings, seals, fasteners |

## Summary

1. **One product, every step.** A fan drive takes a raw casting and bar stock through milling,
   turning, heat treatment, finish machining or grinding, painting with masking, and an assembly
   whose bearing setting is fixed by measuring and choosing shims. Each step changes the part, and
   each step's result is the next step's starting stock.
2. **Most of the hard problems are between the steps, not inside them.** Toolpaths for one
   setup are well understood. What goes wrong in a real shop is the hand-off: a casting that is
   not where the model says, heat-treat growth that eats the grind allowance, paint on a datum
   face, a stack-up that is right on paper and wrong on the bench. So the test case is mostly a
   test of **the states of the part and the records that travel with it**.
3. **SolidCraft would need five things it does not have yet** (opinion): (a) a part that has
   **named manufacturing states** (as-cast, rough-machined, heat-treated, finished, painted), each
   a body that can be stock for the next step; (b) **non-machining process steps** (heat treat,
   paint, wash, inspect) in the same process route as the CAM setups; (c) **tolerances on the
   model** that the CAM, the inspection plan and the assembly all read; (d) **tolerance stack-up
   and shim selection** in assemblies; (e) a **per-part record (traveller)** that collects
   measurements across all steps and feeds the shim calculation.
4. **The plan is staged** so each stage is useful alone: model the parts, then one milling setup
   on the casting, then turning the shaft, then the process route with heat treat and paint, then
   the assembly with stack-up and shims, then closing the loop with inspection, then the whole
   product end to end.
5. **It can be public.** A generic stand-in fan drive, modelled from scratch with made-up but
   realistic dimensions, could live in the SolidCraft repository as an open test fixture. The
   real design stays private and is used only to check that the stand-in is realistic.

## Fundamentals

### What a fan drive is (generic)

**Fact (general engineering knowledge).** A fan drive carries a cooling fan on a rotating hub,
driven from an engine or motor by a belt, gears or a direct shaft. Industrial and vehicle fan
drives commonly have:

- a **housing or bracket**, usually a **casting** (grey or ductile iron, or aluminium), with a
  machined mounting face, bolt holes, and one or two bearing bores;
- a **shaft or spindle**, **turned** from bar steel, often **heat treated** and ground on the
  bearing journals and seal surfaces;
- a **hub or pulley**, cast or turned, fitted to the shaft by a key, spline or taper;
- **rolling bearings**, very often a pair of **tapered roller bearings** or angular-contact
  bearings whose axial play or preload must be set at assembly;
- **seals**, a **retaining nut or end cap**, **spacers** and a **shim pack** that sets the
  bearing endplay or preload;
- **fasteners**, plugs, a grease fitting or breather;
- **paint** on the outside, with machined fits and threads masked.

> **[TODO Jason]** Which of these your fan drive has, in generic words only: drive type (belt,
> gear, direct), housing material family (for example "ductile iron casting"), bearing type, how
> endplay or preload is set (shims under a cap, a spacer ground to length, a nut torqued to a
> rolling-torque target), and roughly how many parts make one assembly. No part numbers, customer
> names or drawings.

### The route of each part (generic)

| Part | Typical route (opinion, generic) | States the model needs |
|---|---|---|
| Housing (casting) | Cast at a foundry → inspect casting → mill first datum face from cast datum targets → mill and bore → drill and tap → wash → paint with masks → final inspect | as-cast, op 10, op 20, painted |
| Shaft | Saw bar → turn between centres or in a chuck, leaving grind stock on journals → mill keyway → heat treat → straighten if needed → grind journals → thread → inspect | bar, turned, heat-treated, ground |
| Hub or pulley | Cast or saw → turn bore and faces → broach or mill keyway → balance → paint | as-cast or bar, turned, painted |
| Spacers, caps | Turn or buy | turned |
| Shims | Buy as a set of standard thicknesses or cut from shim stock | catalogue item |
| Bearings, seals, fasteners | Buy | catalogue item |
| Assembly | Press bearings → fit shaft → measure endplay without shims → choose shim pack → assemble → torque fasteners → check rolling torque or endplay → paint touch-up → final test | as-built, with records |

> **[TODO Jason]** Correct this route to match your shop's, in generic steps. In particular: do
> you machine before or after heat treatment, do you grind, do you paint before or after
> assembly, and is balancing done.

### Castings as stock

**Fact.** A casting is not where the nominal model says it is. ISO 8062-3 sets **casting
dimensional tolerance grades (DCTG)** and **required machining allowance grades (RMAG)** by process
and material, so a sand-cast iron part has both a size scatter of millimetres and a machining
allowance added on the faces to be cut. A cast surface also has a hard, abrasive skin, sand
inclusions and sometimes porosity just under it.

What this means for the software (opinion):

- The **stock is the casting model**, not a block. It has draft, fillets, cored holes, and an
  allowance on every machined face, and it is **uncertain**: each face can be anywhere inside its
  tolerance band.
- The **first operation locates on cast datum targets** (three, two, one points on cast faces,
  as in ASME Y14.5 datum targets) and machines the datum faces that every later operation uses.
  The workholding topic's hold must be checked against the *worst* cast position, not the nominal.
- The **first cut meets the skin** at a depth that changes around the part, so the cutting
  physics topic's force model must treat engagement as a range, and the slider should start
  conservative on the first pass.
- **Probing the casting before cutting** (in-process inspection topic) turns the uncertainty into
  a measured position, so the program can shift or reject the part before it scraps a tool.

### Heat treatment as a process step

**Fact.** Steel shafts are commonly through-hardened (quench and temper), induction hardened on
journals, or carburised (case hardened). Each changes size and shape: parts **grow or shrink**
by a small amount and long slender parts **bow**. This is why bearing journals are usually turned
with **grind stock** left on, hardened, then ground to size. Carburising needs **masking** of areas
that must stay soft (often threads), and hardness is checked afterwards (Rockwell, ISO 6508).

What this means for the software (opinion):

- Heat treatment is a **process step with no toolpath** but with effects the model must carry:
  a target hardness and depth per zone, a **size-change and distortion allowance**, masked zones,
  and an inspection (hardness, runout) afterwards.
- The **turned state must be modelled with grind stock** on the journals, larger than the
  finished state by an allowance that covers heat-treat distortion. If the measured runout after
  heat treatment is larger than the grind stock can clean up, the part is scrap or needs
  straightening; the record should catch that **before** the part reaches the grinder.
- The **cutting physics for the finish step changes**: hardened steel at 50 to 60 HRC is ground
  or hard turned, not cut with the roughing data.

> **[TODO Jason]** Generic heat treatment only: which kind (through, induction, carburise, none),
> rough target hardness range, and whether it is done in house or outside.

### Paint as a process step

**Fact.** Protective paint systems are specified by standards such as ISO 12944 (corrosion
protection of steel structures by paint systems), with a **dry film thickness** typically in the
range of tens to a few hundred µm. That thickness is larger than many fit tolerances, so bores,
journals, mating faces, threads and earth points are **masked** with plugs, caps or tape.

What this means for the software (opinion):

- Paint is a **process step on faces**: which faces are painted, which are masked, film thickness,
  and whether the paint is applied before or after assembly.
- A **masked face is a constraint the model can check**: every face with a fit tolerance or a
  datum must be masked or machined after painting. A rule that flags "painted face carries a
  tolerance tighter than the film thickness" would catch a real class of mistakes.
- The **painted state** is what the assembly and the customer see; it is the body that should go
  into the assembly's clearance checks for exterior parts.

### Assembly with shims and tolerances

**Fact.** The axial play or preload of a pair of tapered roller bearings depends on the sum of
many part lengths: the housing bore shoulders, the bearing widths, spacers, the shaft shoulders,
the cap. A **tolerance stack-up** adds these up:

- **Worst case:** the assembly tolerance is the sum of every part's tolerance. Safe, often too
  tight to make.
- **Statistical (root sum of squares, RSS):** the assembly tolerance is the square root of the sum
  of squares of part tolerances, which assumes independent, centred, normal scatter. Looser, with a
  small risk of out-of-tolerance assemblies.
- **Adjust at assembly:** when neither is affordable, one part is made **adjustable**: a **shim
  pack**, a spacer ground to measured length, or a nut set to a measured rolling torque. Bearing
  makers' public application guides describe these setting methods.

**A worked example, with made-up numbers (illustrative only).** Five axial parts each have a
tolerance of ±0.05 mm. Worst case the gap varies by ±0.25 mm; RSS by ±0.11 mm. The bearing needs
an endplay of 0.05 to 0.10 mm, a band of 0.05 mm, so neither works and a shim is needed. At
assembly the gap is measured with no shims: say 0.83 mm. The target is the middle of the band,
0.075 mm, so the shim pack should be 0.83 − 0.075 = 0.755 mm. From a set of 0.05, 0.10, 0.25 and
0.50 mm shims, 0.50 + 0.25 = 0.75 mm gives an endplay of 0.08 mm, inside the band, with two shims.
The software would do exactly this: compute the needed thickness from the measurement, choose
the fewest shims from the stocked sizes, and record which shims went into which serial number.

> **[TODO Jason]** Generic only: how endplay or preload is set (shims, ground spacer, torque),
> whether it is measured as endplay with a dial indicator or as rolling torque, what shim sizes
> you stock (generic), and how often a stack does not come out on the first try.

### The per-part record (traveller)

**Fact (general practice).** Many shops send a **traveller** (route sheet) with each part or
batch: the steps, sign-offs and measured values. For aerospace and defence work, first-article
inspection under AS9102 and part traceability are contractual. The in-process inspection topic
proposes that SolidCraft keep this record per part.

In the fan drive, the record is not just paperwork: **the assembly step reads it.** The bore
shoulder depth measured at op 20 and the shaft shoulder length measured after grinding can
predict the shim pack before the parts reach the bench. If the prediction and the bench
measurement disagree, one of the measurements or one of the parts is wrong.

## Accuracy

- The generic fan drive and route are common knowledge from public trade and textbook sources;
  they are **not** Jason's company's design and do not describe it. Where a TODO is filled in,
  that section becomes personal experience and is labelled.
- The stack-up example uses made-up numbers chosen to be easy to follow. Real bearing endplay and
  preload values come from the bearing maker for the specific bearing and application.
- Standards are cited by name and scope only; their contents are paywalled and no text is copied.
- "Five things SolidCraft would need" is based on reading SolidCraft's ROADMAP.md and searching
  its issues in October 2026: assemblies, joints and motion exist; fastener stacks are planned
  ([#2072](https://github.com/tangency-labs/solidcraft/issues/2072)); 2.5D CAM is an RFC
  ([#1153](https://github.com/tangency-labs/solidcraft/issues/1153)); no issue for tolerance
  stack-up, GD&T on the model, turning, or non-machining process steps was found (searches may
  have missed some).

## Programs

**Fact (general).** Commercial tools already do parts of this, each in its own program: CAM
programs accept a casting model as stock; separate tolerance analysis programs do 1D and 3D
stack-ups; quality systems hold travellers and first-article reports; ERP and MES systems hold
the process route with outside operations such as heat treat and paint. The gap is that **they are
separate programs with separate data**, so the casting allowance, the heat-treat growth, the
masked faces and the measured shoulder depths are re-typed by people between them. No commercial
program was driven for this research.

## Complaints and requests

Common complaints from shop practice (general, public forum and trade knowledge; specific
examples are **[TODO Jason]**):

- The CAM stock is a block, so the first-op program on a casting is written for a part that does
  not exist, and the first part off a new casting lot is a test.
- Heat-treat distortion is found at the grinder, after all the machining value has been added.
- Paint in bores and on mating faces is found at assembly.
- Stack-ups are done once on paper at design time; when parts drift, nobody reruns them, and the
  assembler finds out by needing a shim size that is not in stock.
- Shim choices and measured values are written on paper and never reach the design or the
  supplier.

## What users need

### What SolidCraft would need (opinion)

| Need | What it is | Which stage first needs it | Related topic or issue |
|---|---|---|---|
| Manufacturing states of a part | Named bodies (as-cast, op 10, heat-treated, finished, painted) derived from the design body, each with allowance or growth, each usable as CAM stock | 1 | stock-integrity, #1153 |
| Uncertain stock | A stock body with a tolerance band per face (from ISO 8062-3 grades or measured), so hold and cut checks use the worst case | 2 | workholding, cutting-physics |
| Datum targets and probing | Cast datum targets on the model; a probing step that locates the real casting before the first cut | 2 | in-process-inspection |
| Turning | Lathe setups, turning tools in the shared library, chuck and centre workholding | 3 | tool-library, workholding, machine-library |
| Non-machining process steps | Heat treat, paint, wash, outside operation, inspect: steps in the same route as CAM setups, with their own fields and effects on the part state | 4 | in-process-inspection |
| Face-level process attributes | Masked faces, painted faces, hardness zones; a check that flags a toleranced face that is painted or a hard zone that is cut with soft-steel data | 4 | cutting-physics |
| Tolerances on the model | Size and geometric tolerances stored on faces and features, read by CAM (finish allowance), inspection (what to measure) and assembly (stack-up) | 5 | drawings milestone |
| Tolerance stack-up | Pick a chain of mates in an assembly, get worst-case and RSS results, name the largest contributor | 5 | assemblies milestone |
| Adjustable members and shim selection | Mark a part as a shim pack or select-fit spacer; compute required thickness from a target and measurements; choose from stocked sizes | 5 | nsn-library (shim stock as catalogue items) |
| Standard hardware | Bearings, seals, fasteners and shims as catalogue parts with their own tolerances, placed in stacks | 5 | nsn-library, #2072, #787 |
| Per-part record | Traveller with every measured value across all steps, per serial number; assembly reads it | 6 | in-process-inspection |

### A public stand-in design

**Opinion (a proposal for Jason to confirm).** The real fan drive cannot be published. A
**stand-in fan drive** could be modelled from scratch in SolidCraft with made-up but realistic
dimensions, generic materials and catalogue-style bearings, and released under SolidCraft's
licence as a test fixture (for example as a design in the repository's test data, with an
ATTRIBUTION row). It should:

- have the same *kinds* of features as the real one: a cast housing with draft and machining
  allowance, a stepped shaft with journals and a keyway, a hub, a tapered roller bearing pair, a
  shim pack, a cap and fasteners;
- be sized so a hobby or small-shop machine can make the housing and shaft (useful for anyone who
  wants to check results on real metal);
- carry the full route, tolerances and an example traveller, with made-up measured values that
  produce a known shim answer;
- be **reviewed by Jason** against the real product for realism only, so nothing proprietary
  leaks into it.

> **[TODO Jason]** Roughly what size class your fan drive is (for example "fits in two hands",
> "fan diameter around a metre"), so the stand-in is of a realistic size. Generic words only.

### The staged validation plan

Each stage has: what it exercises, what SolidCraft must do, and a **pass check** that someone can
run. Stages can be done in order or picked up separately; each is useful alone.

**Stage 0: describe the stand-in.** Write the stand-in's part list, route and generic materials
as a short spec (this file plus the TODOs). *Pass:* Jason confirms the route and features are
realistic and nothing in it is proprietary.

**Stage 1: model the parts and their states.** Model the housing as the finished part and derive
the as-cast body (draft, fillets, allowance on machined faces), and the shaft as finished with
a turned state that has grind stock on the journals. *Exercises:* modelling, draft, shell and
offset features; the "manufacturing states" need. *Pass:* both states regenerate from the design
when a dimension changes, and the allowance follows.

**Stage 2: first milling setup on the casting.** Use the as-cast body as stock, locate on datum
targets, machine the mounting face and datum features with 2.5D CAM. *Exercises:* #1153 / PR
#4158 2.5D milling, tool library, workholding on a cast surface, cutting physics through the skin,
machine library. *Pass:* the toolpath runs in simulation against the worst-case casting position
without gouging or air-cutting more than a set amount; the hold check passes at the worst case;
the setup sheet lists the probe step.

**Stage 3: bore and drill the housing; turn the shaft.** Second milling setup located on the
op 10 datums: bearing bores, bolt holes, tapped holes. Turning of the shaft leaving grind stock.
*Exercises:* stock integrity (thin walls around the bores), turning tools and chuck or centre
workholding, machine library for a lathe. *Pass:* the op 20 stock is the op 10 result; the turned
shaft matches the Stage 1 turned state.

**Stage 4: the process route with heat treat and paint.** Add the non-machining steps: heat treat
the shaft (hardness zones, masked threads, growth allowance), grind journals, wash, paint the
housing with masks. *Exercises:* non-machining process steps, face-level attributes, the paint
and hardness checks. *Pass:* the route shows every step in order; the check flags a deliberately
unmasked bearing bore and a deliberately soft-steel finish pass on a hardened journal.

**Stage 5: the assembly, hardware, stack-up and shims.** Assemble housing, shaft, bearings, seals,
spacer, shim pack, cap and fasteners from catalogue parts. Run the endplay stack-up. *Exercises:*
assemblies and joints, fastener stacks (#2072), standard parts (#787), tolerances on the model,
stack-up, shim selection. *Pass:* worst-case and RSS results match a hand calculation; given a
measured gap, the shim chooser returns the fewest stocked shims that land in the endplay band, and
says so when no combination fits.

**Stage 6: close the loop with inspection.** Feed made-up measured values from each step into the
per-part record: casting probe, op 20 bore depths, post-heat-treat runout, ground shoulder
lengths, bench gap. *Exercises:* in-process inspection and the traveller. *Pass:* the record
predicts the shim pack from part measurements before assembly, and flags a deliberate mismatch
between the predicted and bench-measured gap; a deliberate post-heat-treat runout larger than the
grind stock is flagged before the grinding step.

**Stage 7: end to end.** Change one design dimension (for example the bearing spacing) and let it
flow through every state, setup, route step, stack-up and record. *Exercises:* everything.
*Pass:* every downstream item either updates or is marked out of date with the reason; nothing
silently keeps the old value.

**Optional Stage 8: real metal.** If anyone makes the stand-in housing and shaft, compare predicted
and measured forces, positions and shim answers. This is where the machine-health and self-tuning
ideas would get real data.

> **[TODO Jason]** Which stage would be most useful to your shop first, and whether anyone at your
> company might review the stand-in for realism.

## Glossary

- **Allowance (machining):** extra material on a casting or turned part, removed by a later step.
- **As-cast, as-machined, heat-treated, painted:** names for the states of a part as it moves
  through its route.
- **Datum target:** a point, line or small area on a rough surface used to locate a part for its
  first operation (ASME Y14.5, ISO 5459).
- **DCTG / RMAG:** casting dimensional tolerance grade and required machining allowance grade in
  ISO 8062-3.
- **Endplay / preload:** axial play (positive) or axial interference (negative) of a bearing
  pair after assembly.
- **Grind stock:** allowance left on a turned surface to be ground after heat treatment.
- **RSS:** root sum of squares, a statistical way to add tolerances.
- **Shim pack:** a set of thin, precisely sized plates chosen at assembly to set a gap.
- **Stack-up:** adding the dimensions and tolerances along a chain of parts to find the variation
  of an assembly gap.
- **Traveller:** the record that goes with a part through its route, with steps, sign-offs and
  measurements.

## Open questions

1. Are manufacturing states best modelled as **configurations of one part**, as **derived bodies
   in the timeline**, or as **separate documents** linked to the design? (Maintainers' call.)
2. Should non-machining steps live in the **CAM plugin**, in a separate **process planning**
   area, or in the **inspection** records?
3. Where do tolerances live before the drawings milestone lands: on the model now, or only on
   drawings at first?
4. Does stack-up belong to assemblies (geometry) or to a tolerance plugin (statistics), or both?
5. Is a stand-in fan drive welcome as test data in the repository, and where should it go?
6. **[TODO Jason]** What can be shared publicly about the real fan drive (see the thread).

## Sources

Standards, cited by name and scope only:

- ISO 8062-3, *Geometrical product specifications (GPS) — Dimensional and geometrical tolerances
  for moulded parts — Part 3: General dimensional and geometrical tolerances and machining
  allowances for castings.*
- ASME Y14.5, *Dimensioning and Tolerancing* (datum targets, geometric tolerances).
- ISO 5459, *GPS — Geometrical tolerancing — Datums and datum systems.*
- ISO 286-1 and ISO 286-2, *GPS — ISO code system for tolerances on linear sizes* (fits for
  bearing seats).
- ISO 6508-1, *Metallic materials — Rockwell hardness test.*
- ISO 12944 (series), *Paints and varnishes — Corrosion protection of steel structures by
  protective paint systems.*
- SAE AS9102, *Aerospace First Article Inspection Requirement.*

General references (public): bearing makers' published application and setting guides for
tapered roller bearings (endplay, preload and shim setting methods); standard machine design and
manufacturing process textbooks for tolerance stack-up (worst case, RSS) and heat-treat
distortion.

SolidCraft:

- ROADMAP.md (assemblies and joints, drawings milestones), read October 2026.
- [#1621](https://github.com/tangency-labs/solidcraft/issues/1621) CAM research request;
  [#1153](https://github.com/tangency-labs/solidcraft/issues/1153) 2.5D CAM RFC;
  [#2072](https://github.com/tangency-labs/solidcraft/issues/2072) fastener stacks;
  [#787](https://github.com/tangency-labs/solidcraft/issues/787) standard components.
