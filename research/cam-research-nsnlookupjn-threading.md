# CAM research: threading as a critical operation, from hole to gauge

Research for [#1621](https://github.com/tangency-labs/solidcraft/issues/1621) (Research Request
for Plugin: CAM). This is the first file in a topic Jason calls **General CAM feature
requests**: operations that every CAM program offers but treats as "point and splat". Threading
is the first, because it is where tools break most often. This file covers **a threading module
that plans the whole process, from the hole to the gauge**: the drill, the chamfer, the choice
between taps (straight flute, spiral point, spiral flute, forming) and thread mills (single-flute,
multi-flute, single-profile, multi-form), lead-ins, partial threads, external threads, tapered
pipe threads (NPT, NPTF, BSPT), thread classes, the physics of each process, how threads fail
and how to prevent it, in-machine tricks (a fixed brush that cleans the tap, a cup of tapping
compound the tap dips into), wear compensation that sets the thread fit, and a feedback loop with
**SolidCraft Edge** (Jason's name for the Raspberry Pi computer at the machine,
which sits on the edge of CAM and brings machine and automation data back into it) that dials in a thread cutter and keeps that compensation
between jobs and machines.

Companion files (same author, same fork):

- [Cutting tool physics](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-cutting-physics/research/cam-research-nsnlookupjn-cutting-physics.md):
  Kienzle `kc1.1`/`mc`, deflection, tool life and the conservative-to-aggressive slider. Thread
  milling below reuses that model; tapping adds its own torque model.
- [CAM Physics System](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-vgqo3g/research/cam-research-nsnlookupjn-physics-system.md):
  the umbrella (one Setup, one limits engine, one slider, "explain every number", the simulation
  gate). The threading planner is another client of that engine.
- [In-process inspection](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-y2w06n/research/cam-research-nsnlookupjn-in-process-inspection.md):
  thread classes on the model, go/no-go gauging, the Xometry rule that threads follow a named
  standard, the compensation-in-cycle pattern (target, dead band, feedback fraction, limit), the
  Raspberry Pi box (now named SolidCraft Edge) and "Self-tuning CAM operations". This file is the thread-specific case of that loop.
- [Cloud tool library](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-tool-library/research/cam-research-nsnlookupjn-tool-library.md):
  tool tags and the shop instance of each tool, where a thread cutter's saved compensation lives.
- [Tool holding](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-4aoliy/research/cam-research-nsnlookupjn-tool-holding.md):
  collet slip torque (section 4), stick-out (section 7) and tapping holders (rigid, floating,
  tension/compression).
- [Machine health](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-9w3gq9/research/cam-research-nsnlookupjn-machine-health.md):
  spindle/feed synchronisation drift and circular-interpolation error, which both change thread
  size as a machine ages.
- [Machine library](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-machine-library/research/cam-research-nsnlookupjn-machine-library.md)
  and [workholding](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-workholding/research/cam-research-nsnlookupjn-workholding.md):
  rigid tapping limits, spindle torque at tapping speed, and the reaction torque the part must
  hold.
- [DFM check](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-uhz8i7/research/cam-research-nsnlookupjn-dfm-check.md)
  and [physical properties](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-uxzal2/research/design-research-nsnlookupjn-physical-properties.md):
  thread depth rules on the design side, hardened regions and coatings that change how a thread
  is cut and what size it must be cut to.

Licence: MIT OR Apache-2.0, the same as SolidCraft (see the Licence section of CONTRIBUTING.md).

## About this research

- **Author:** Jason ([@nsnlookupjn](https://github.com/nsnlookupjn)). Topic and direction by
  Jason: "Add Threading as a critical operation. Have a module which designs threading process
  from hole-making through threading [...] This is where I break the most tools because no other
  software does the physics, it is kind of just point and splat." Source research and writing
  done with an AI assistant (Claude) in October 2026.
- **Author's experience (personal experience, not a cited source):**
  - **Shop floor since 1987:** a machine shop apprenticeship from 1987 to 1991, starting on
    manual machines, then writing G-code at the control on early CNC machines (Milacron and
    similar, with no way to save a program).
  - **CAD/CAM over about 35 years:** most major packages. In automotive work: Unisurf, Siemens,
    Sketchpad and NX, with programs stored on punch cards, then tape, then floppy disks; later
    CATIA, AutoCAD and Pro/Engineer. Currently fluent in Fusion, Mastercam, SOLIDWORKS and NX.
  - **Engineering and software:** BTEC HND in Electrical and Mechanical Engineering, MEng, PhD
    in Materials Science, certified welder. Programming since childhood (machine code, BASIC,
    C++, C#, assembler, web and database technologies). Owns engineering and software companies.
  - **Current work:** engineering and manufacturing spare parts for defence and aerospace,
    including flight-safety components.
- **How it was produced:** web searches of public sources (tool makers' technical pages and
  catalogues, gauge makers' technical notes, trade press such as Modern Machine Shop and
  Production Machining, peer-reviewed papers, CAM programs' public help pages). No commercial
  CAD/CAM program was driven or scripted, no GPL/LGPL/AGPL source was read, and no text is copied
  beyond short quotes. Standards (ISO 68-1, ISO 261, ISO 965, ASME B1.1, ASME B1.20.1, ASME
  B1.20.3, ISO 7-1, ISO 1502, DIN 2197/ISO 8830, DIN 76) are described from catalogue entries and
  secondary summaries, not their paid text, so every table value must be checked against the
  purchased standard before it ships.
- **Units:** millimetres, newtons, newton-metres, seconds. Inch threads are described in their
  own units where the standard uses them.
- **Marking:** "(unverified)" marks a claim not confirmed against a source. "(derived)" marks a
  formula or number worked out here from geometry or from the companion files' models.
  "(textbook)" marks classical thread geometry or control theory that appears in every reference.
  **Opinion** is labelled.

> **[TODO Jason: personal experience]** Which threads break your tools (size, material, blind or
> through, depth in diameters), with which tool (cut tap, form tap, thread mill), on which
> machine and holder. What you do today to prevent it (peck tapping, brushing, compound, air
> blast, oversize drill, thread milling instead). How you set thread fit today (comp by feel,
> swap taps, gauge every N parts). Label this as personal experience.

## Summary

1. **A thread is a chain of operations, not one operation.** Spot, drill, chamfer, (ream or
   bore), clean the hole, thread, clean the thread, gauge. Most tap breaks start upstream: a
   drill that is too small, too shallow or wandered, or chips left in a blind hole. The module
   plans the whole chain from the hole feature (which already knows its thread, class and depth)
   and checks each link against the next one.
2. **Tapping has simple, usable physics.** Cutting torque ≈ `kc · P² · D / 8000` N·m (tool
   maker's formula, Lubero); the cut area falls quickly as the percentage of thread is reduced
   (75 % → 65 % thread removes about 21 % less material, derived); chip volume in a blind hole can
   be compared with flute volume (derived: an M8 straight-flute tap in a 16 mm blind hole makes
   about 1.5 to 3 times more loose chip than its flutes can hold); synchronisation error in rigid
   tapping makes thrust (one test: 66 lb in a collet chuck, 4.4 lb in a compensating holder). Each
   becomes a check the planner runs and explains.
3. **Thread milling has its own physics**: feed at the tool centre is a fraction of the edge feed
   for internal threads (`(D − d)/D`, a quarter for M8 with a 6 mm cutter); tool deflection makes
   the thread tapered along its length (derived: about 27 µm on pitch diameter at the tool tip and
   5 µm near the shank for an assumed M8 case, a sixth of the 6H tolerance); and servo lag on a
   1 mm path radius can shrink an internal thread by tens of micrometres at thread milling feeds
   (textbook servo model). These three are why thread mill compensation "doesn't carry over"
   between jobs and machines; the module models them so it can.
4. **Process selection is a table the module can fill** from the feature (size, depth, blind or
   through, class, taper), the material (ductile or brittle, hardness, chip form), the machine
   (rigid tapping, synchronisation, max tapping speed, through-coolant) and the tools in the
   library. It proposes a process, shows why, and lets the user pick another.
5. **Tapered threads are different in kind.** For NPT/NPTF/BSPT the size is the axial position
   of the gauge: one turn of gauge standoff equals `P/16` on diameter (derived from the 1:16
   taper). A taper tap's size is therefore set by its depth (Z), and a taper thread mill's by its
   radius: `Δr = k · P/32` for `k` turns. For 1/2-14 NPT that is 0.057 mm of radius per turn.
6. **Failure modes map to signals and to rules.** Undersize or shallow hole, chip packing, tap
   bottoming, misalignment, synchronisation thrust, reversal shear, worn tap, holder slip, part
   rotation, form tap in brittle material, thread mill deflection, chatter. Each has a
   prevention rule in the planner and a signal SolidCraft Edge can watch (spindle load envelope per
   hole, missing drill load, torque trend).
7. **In-machine tricks become stations in the machine library** (Jason's ideas): a fixed tap
   cleaning brush, a cup of tapping compound, an air blast to empty blind holes before tapping, a
   probe or load check that the hole was drilled. CAM inserts the visits automatically at an
   interval set by the physics (chip volume, compound film life), and the simulation checks them
   like any other move.
8. **Wear compensation sets the fit, and must aim at the right end of the tolerance.** A
   thread mill wears smaller, so an internal thread drifts small and an external thread drifts
   large. Start new tools on the safe side (internal small, external large: you can always cut
   more, never less), dial in by gauge, then aim at the end of the tolerance that leaves the
   most room for wear (opinion, derived from the drift direction).
9. **Jason's feedback loop** splits the saved compensation into parts that travel with the tool
   (its ground size and wear), with the tool and holder in a material (deflection), with the
   machine (servo lag, synchronisation), and with the material lot (spring-back). Moving a tool
   to another machine keeps the first three and swaps only the machine term, which is what makes
   "permanent compensation between jobs and machines" possible. Indicating thread gauges with
   digital output (or operator entry) give SolidCraft Edge a number instead of pass/fail.

## Fundamentals

### 1. Words used in this file

- **Major diameter** `D` (internal) / `d` (external): the largest diameter of the thread.
  **Minor diameter** `D1` / `d1`: the smallest. **Pitch diameter** `D2` / `d2`: where the groove
  and the ridge are equal width; the diameter that gauges check and that sets the fit.
- **Pitch** `P`: distance between threads. **Lead**: advance per turn (`= P` for single-start).
  **TPI**: threads per inch, `P = 25.4 / TPI` mm.
- **Percentage of thread**: how much of the full thread depth the pre-drilled hole leaves for
  the tap to make. Below 100 % the internal thread's crests are truncated by the drilled hole.
- **Thread class / tolerance class**: the allowed band for pitch diameter (and minor/major).
  ISO 965: tolerance grade + position, e.g. `6H` internal, `6g` external, `4H5H`. ASME B1.1
  (unified): `1B/2B/3B` internal, `1A/2A/3A` external.
- **Chamfer (tap)**: the tapered start of a tap, where the first few teeth do the cutting.
  **Chamfer (hole)**: the countersink at the hole entry.
- **Rigid (synchronous) tapping**: the control locks feed to spindle rotation. **Floating /
  tension-compression holder**: lets the tap float axially to absorb mismatch.
- **Thread mill**: a rotating cutter that makes the thread by helical interpolation. **Single
  profile** (one tooth row, many helical passes along Z) or **multi-form** (many rows, one turn
  makes the whole length). **Single-flute / multi-flute**: number of cutting edges around the
  tool (Jason's terms; both exist for both profile types).

### 2. Thread systems and classes the module must know

| System | Standard (catalogue names) | Angle | Taper | Typical classes | Notes |
|---|---|---|---|---|---|
| ISO metric coarse/fine | ISO 68-1, ISO 261, ISO 965 | 60° | none | 6H/6g; 4H5H/4h, 6G/6e for coatings | SolidCraft's catalogue has these sizes but not the class (docs/thread-catalogue.md) |
| Unified UNC/UNF/UNEF | ASME B1.1 | 60° | none | 2B/2A, 3B/3A | catalogue has 14 UNC + 14 UNF sizes |
| UNJ | ASME B1.15 (catalogue name, unverified scope) | 60°, rounded root | none | 3B/3A | aerospace; controlled root radius on the external thread |
| NPT | ASME B1.20.1 | 60° | 1:16 on diameter | ±1 turn at L1 | sealed with tape or compound |
| NPTF (Dryseal) | ASME B1.20.3 | 60° | 1:16 | Class 1, Class 2 | seals by crest/root interference; class 2 needs crest checks |
| BSPT | ISO 7-1 (Rc internal taper, Rp internal parallel, R external taper) | 55° | 1:16 | ±1 turn (unverified wording) | |
| BSPP | ISO 228-1 | 55° | none | A/B external | catalogue has G and Rp |

**Fact** for the system names and angles; class lists are the common ones, not complete.

Basic metric geometry (textbook, from ISO 68-1): fundamental triangle height `H = 0.866 P`;
`D2 = D − 0.6495 P`; `D1 = D − 1.0825 P`. For M8×1.25: `D2 = 7.188`, `D1 = 6.647`. In class 6H
the pitch diameter may be up to `TD2 = 0.160` mm larger and the minor diameter up to
`TD1 = 0.265` mm larger (values as commonly tabulated for this size; unverified against
ISO 965-1).

### 3. The process family

| Process | Holes | Materials | How chips leave | Sets size by | Main risks |
|---|---|---|---|---|---|
| Straight-flute cut tap | through, shallow blind | short-chipping (cast iron, brass, some cast Al) | stored in flutes | the tap | chip packing in ductile material |
| Spiral-point (gun) tap | through | ductile | pushed ahead, out of the bottom | the tap | packs chips if used blind |
| Spiral-flute tap | blind | ductile | pulled up the flutes | the tap | weaker core; chips wind and break on reversal |
| Forming (roll) tap | blind or through | ductile only (steel to about 40 HRC, unverified; Al, stainless, copper) | none: material is displaced | the tap **and** the drilled size | high torque, needs exact drill size and lubrication; never in grey cast iron |
| Taper pipe tap | through or blind | most | flutes | **depth** (Z) | highest torque: the whole length cuts |
| Thread mill, multi-form | blind or through, D ≥ about 3 mm (unverified) | almost any, incl. hardened | normal milling chips | **radius** (cutter comp) | deflection taper; profile distortion if tool is large for the hole |
| Thread mill, single-profile | same, also deep or large threads | almost any | milling chips | radius, along Z if programmed | long cycle; good for deflection control |
| Single-point turning | external and internal on lathes | any | turning chips | X position | chip control, infeed method |
| Die head / thread rolling | external | rolling: ductile | none (rolling) | the dies | outside the mill module; listed for completeness |

**Fact** for the general family (tool makers' guides; Protool's tap-breakage guide for the
straight/spiral-point/spiral-flute/form split); the size and hardness limits are unverified rules
of thumb.

### 4. Hole-making: the first link

**Drill size.** Two published formulas give the drill for a chosen percentage of thread `%`
(inch forms are `0.01299·%/TPI` and `0.0068·%/TPI`; metric forms from an eMastercam forum post
and the same arithmetic):

- Cut tap: `d_drill = D − % · P / 76.98`
- Form tap: `d_drill = D − % · P / 147.06`
- Shortcut for cut taps: `d_drill = D − P` (about 77 %; "works for all 60-degree threads",
  American Machinist; also Lubero's core-hole formula `d4 = D − P`, "not valid for TR, RD or
  conical threads").

Worked example, M8×1.25 (derived):

- Cut tap, 75 %: `8 − 75 · 1.25 / 76.98 = 6.782` → 6.8 mm drill. Minor diameter limits for 6H
  are 6.647 to 6.912, so 6.8 is inside.
- Form tap, 65 %: `8 − 65 · 1.25 / 147.06 = 7.447` → 7.4 or 7.45 mm drill. The tap pushes
  material inwards to make the minor diameter, so the drilled size sets the crest fill: a
  0.05 mm smaller drill raises forming torque sharply and can break the tap ("An undersize hole
  can fail very quickly", Protool). The form tap hole therefore deserves a tolerance of its own,
  and in many shops a reamed or bored hole for larger sizes (opinion).

SolidCraft today: UNC/UNF use Guhring's 75 % cut-tap drills; metric keeps `D − P`
(docs/thread-catalogue.md). The module should store **both** a cut-tap and a form-tap drill per
size and pick by process, and let the material change the percentage (opinion).

**Percentage of thread versus torque (derived).** The material a cut tap removes from an
internal 60° thread is a groove whose width grows linearly from `P/8` at the major diameter to
`3P/4` at the basic minor. Cut from the major diameter down to a radial depth `t`, the groove
area per pitch is `A(t) = 0.125·P·t + 0.577·t²`. With unified percentage defined on `0.6495·P`:

| % thread | depth t | A(t) / P² | relative cut area (≈ torque) |
|---|---|---|---|
| 75 % | 0.487 P | 0.198 | 1.00 |
| 65 % | 0.422 P | 0.156 | 0.79 |
| 60 % | 0.390 P | 0.137 | 0.69 |
| 50 % | 0.325 P | 0.102 | 0.51 |

Because the widest part of the groove is near the minor diameter, a slightly larger drill cuts
torque more than linearly. Thread stripping strength falls much more slowly; a widely repeated
claim is that 100 % thread is only a few percent stronger than 75 % (unverified; stripping
strength depends on engagement length and the weaker of the two materials). **Opinion:** in
tough materials (stainless, titanium, Inconel) the module should default to 60 to 65 % when the
class allows (the minor diameter must stay inside `D1 max`) and say so.

**Drill depth for a blind hole (derived).**

```
full-diameter depth  ≥ thread depth (full threads)
                       + tap chamfer length (chamfer threads × P)
                       + clearance (≥ 1 P, plus chip room and reversal over-travel)
drill depth to tip   = full-diameter depth + drill point length (0.30 · d for 118°, 0.21 · d for 140°)
```

M8×1.25, 16 mm full threads, spiral-flute tap with a 2 to 3 thread chamfer (form C): chamfer
`2.5 × 1.25 = 3.1`, clearance 1.5: full diameter to 20.6 mm, drill tip at `20.6 + 0.30 × 6.8 =
22.6` mm. If the hole feature or the part cannot take that depth, the planner says so on the
design (DFM) and offers a bottoming-chamfer tap (form E, 1.5 to 2 threads) or a thread mill,
which can thread to about 1 to 1.5 P from the bottom (unverified rule of thumb).

Tap chamfer forms (DIN 2197 / ISO 8830, from common summaries; unverified against the standard):
A 6 to 8 threads, B 3.5 to 5.5 with spiral point, C 2 to 3, D 3.5 to 5, E 1.5 to 2.
The number of chamfer threads is also the number of **incomplete threads** the tap leaves at
the bottom, so the module can draw them on the model and the drawing.

**Hole quality.** Drill wander, bell-mouth, entry burrs and an off-square hole bend the tap
(Protool). The planner adds a spot drill and an entry chamfer by default and checks drill length
(L/D) against the drill's straightness class (opinion).

### 5. Physics of tapping

**Torque.** Lubero (a tap maker) publishes:

```
M_D = kc · P² · D / 8000      [N·m]    kc in N/mm², P and D in mm
P_c = M_D · n / 9550          [kW]     (the page prints 9500)
```

with a kc table: unalloyed structural steel 2300, quenched and tempered medium strength 2600,
austenitic stainless 3200, tool steel 3600, long-chipping aluminium 800, grey cast iron 1250
N/mm². The formula is the cut area per pitch (∝ `P²`) times the radius (∝ `D`), with the
constant absorbing the thread geometry and the units (derived interpretation).

Worked example, M8×1.25 in 42CrMo4 (kc 2600), at 10 m/min (n ≈ 400 rpm) (derived):
`M_D = 2600 · 1.5625 · 8 / 8000 = 4.1 N·m`; `P_c = 4.1 · 400 / 9550 = 0.17 kW`.

**What that torque must fit under** (each from a companion file):

- **The tap itself.** A rough torsional limit is `T_break ≈ τ · π · d_core³ / 16`. With an
  assumed core of 4.0 mm and an assumed allowable shear of 1000 MPa for HSS-E with flutes:
  `T ≈ 12.6 N·m`, about 3 times the cutting torque (derived; both inputs are assumptions, and the
  library should hold the maker's break torque when published). That margin is thin: a worn tap
  (torque rising, below), packed chips or a slightly small drill can use all of it. Bottoming has
  no limit at all.
- **The holder.** Collet slip torque (tool holding, section 4). Tapping collets with a square
  drive do not rely on friction (fact); plain ER collets do.
- **The spindle** at tapping speed, which is low, so the torque curve matters more than power
  (machine library). Reversal at the bottom needs the spindle to stop and reverse the inertia
  in a fraction of a turn.
- **The workholding**, which must resist the reaction torque about the hole axis, the largest
  rotating moment in many small parts (workholding).

**Torque along the hole.** A measured study of an M8 tap in 42CrMo4 (Springer, 2020) found the
undeformed chip thickness about 0.12 mm per tooth, the largest chip on the first tooth that cuts
to full depth, radial forces about half the tangential, and **torque rising with depth in blind
holes because chips are removed against the feed direction**, which their force model did not
capture. This is the signature SolidCraft Edge should watch: steady torque after the chamfer is
engaged, plus a depth-dependent rise from chip friction. A steeper rise than the hole's own
history means packing.

**Chip volume versus flute volume (derived).** For M8×1.25 drilled 6.8 (`t = 0.6`), the groove
area per pitch is `0.125·1.25·0.6 + 0.577·0.36 = 0.30 mm²`; at the mean diameter of about 7.4 mm
that is about 7 mm³ per pitch, or 5.6 mm³ of solid metal per mm of depth. A 16 mm blind thread
makes about 90 mm³ of solid chip. Loose chips take several times the solid volume (assumed bulk
factor 4 to 8, unverified), so 360 to 720 mm³. If a straight-flute tap's flutes are about 30 % of
its cross-section (assumed, about 15 mm²), the flutes inside a 16 mm hole hold about 240 mm³.
The chips need 1.5 to 3 times the room the flutes have: a straight-flute tap must pack in this
hole, and only a spiral flute (which moves chips out) or peck tapping (which empties the flutes)
or a thread mill avoids it. The planner can run this check for every blind tapped hole and pick
the tap type or the peck count from it.

**Synchronisation and thrust.** In rigid tapping the feed per revolution never matches the tap's
lead exactly; "some amount of synchronization error is inevitable with rigid tapping", and it
grows as a machine ages (Production Machining). The tap's flanks then push against the thread
it just cut, making axial force and an oversize or thin-flanked thread. Measured peak thrust in
one Modern Machine Shop item: 66 lb in a typical collet chuck, 4.4 lb in a holder with a small
flexible axial float (Tapmatic/Big Kaiser-type "synchro" holders; vendor data). Derived scale: a
1 % mismatch at P = 1.25 is 12.5 µm per revolution; over the 13 revolutions of a 16 mm thread
that is 0.16 mm that the flanks or the holder must absorb.

**Reversal.** At the bottom of a blind hole the tap stops and reverses. Chips in the flutes are
sheared at their roots and can jam behind the cutting edges; "breakage can also occur on
reversal from chip drag or abrupt reversing" (Protool). Commercial tap monitoring learns the
forward power profile and goes into a "hold mode" during reversal to ignore its spikes (Caron
TMAC description), which also means reversal is a blind spot; SolidCraft Edge should record reversal
torque separately and trend it (opinion).

**Wear.** As a tap wears, torque rises and the thread it cuts gets smaller (fact: dull taps
still "look serviceable", Protool, "replace taps before torque becomes unstable"). Tap life is
better counted in threaded length (or in holes of the same size in the same material) than in
time (opinion).

### 6. Physics of thread milling

**Feed at the tool centre** (textbook; in every tool maker's thread milling data). Tool makers
give feed per tooth at the cutting edge; the control moves the centre.

```
internal:  F_centre = F_edge · (D − d_tool) / D
external:  F_centre = F_edge · (D + d_tool) / D
path radius (internal) = (D − d_tool) / 2     (ISCAR: A = (Do − D)/2, tool path radius A/2 for its example)
```

M8 internal with a 6 mm cutter: `F_centre = 0.25 · F_edge`. Forgetting this quadruples the chip
load on the edge, which is a classic way to break small thread mills (opinion; the module must
not let a programmer enter centre feed by mistake).

**Engagement.** Internal thread milling with a tool close to the hole size has a large arc of
engagement, so the chip is thicker than `fz` suggests for a given radial depth; the
cutting-physics file's chip-thinning maths applies in reverse (larger effective engagement). The
module computes engagement from the tool, path radius and radial depth for each pass (derived).

**Deflection makes the thread tapered.** The cutter is a cantilever; the tip deflects most. Using
the tip-load cantilever (textbook), the deflection at a distance `x` from the holder of a tool
with free length `L` is `δ(x) = F · x² · (3L − x) / (6 · E · I)`, `I = π · d_eff⁴ / 64`.

Worked example (derived; inputs assumed): M8×1.25 internal, 16 mm deep, solid carbide
multi-form, effective neck diameter 4.5 mm (`I = 20.1 mm⁴`), `E = 600 GPa`, free length 20 mm,
radial force 60 N on the finishing turn.

- At the tip (`x = 20`): `δ = 60 · 8000 / (3 · 600000 · 20.1) = 0.013 mm`; pitch diameter
  `2δ = 27 µm` small.
- At the top of a 12.5 mm cutting length (`x = 7.5`): `δ = 0.0025 mm`; pitch diameter 5 µm
  small.

So the thread is about 22 µm smaller on pitch diameter at the bottom than at the top: a sixth of
the 6H tolerance (160 µm), and the go gauge stops before full depth even when the top is right.
ISCAR's troubleshooting table lists the same symptom ("tapered thread") with tool overhang as the
cause. Consequences for the module:

- A **multi-form** cutter cuts the whole length at once, so its taper cannot be programmed
  away; the remedies are lower radial force (more radial passes, a final spring pass at nearly
  zero depth, a shorter or stiffer tool).
- A **single-profile** cutter cuts one pitch at a time while moving along Z, so the path radius
  can follow the predicted deflection: `r(z) = r_0 + δ(z)` (a "deflection-compensated helix").
  We found no CAM help page that describes this (opinion; Fusion's thread operation offers
  passes, spring passes, compensation and lead-in settings, but no Z-dependent correction).
- Climb milling, bottom to top, is the usual direction for right-hand internal threads (ISCAR
  example program), which also keeps the first (most deflected) cut at the bottom where the tool
  is fresh to the hole.

**Profile distortion.** A multi-form cutter's teeth are rings, not a helix. When the cutter is
large compared with the hole, or the lead angle is steep, the rings cut the flanks away from the
true thread form (fact as a phenomenon; the size of the error and the usual "cutter at most about
70 % of the hole" rule are unverified). The module should compute the flank error for the chosen
cutter and hole, the same way the gear file generates tooth flanks as the envelope of the tool
(the
[gear, spline and broach file](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-gyhruq/research/cam-research-nsnlookupjn-gear-spline-broach.md)
proposes that algorithm).

**Servo lag shrinks small circles** (textbook, first-order servo without feed-forward): the
actual radius of a circle of radius `R` run at contour speed `v` with position-loop gain `Kv` is
smaller by about

```
Δr ≈ v² / (2 · R · Kv²)
```

M8 with a 6 mm cutter has `R = 1` mm. At a centre feed of 320 mm/min (5.3 mm/s, e.g. a 4-flute
cutter at 8000 rpm and 0.04 mm/tooth at the edge) and `Kv = 30 s⁻¹`: `Δr = 28.4 / (2 · 1 · 900)
= 0.016 mm`, so the thread is 32 µm small on pitch diameter, a fifth of the tolerance (derived).
Feed-forward and high-precision modes reduce this greatly; the real figure is machine-specific and
comes from a ballbar test or the machine library. The important point for compensation: **this
term depends on the feed and the path radius and the machine**, so a wear offset found at one
feed on one machine is wrong at another feed or on another machine. The module computes it so
the saved offset can be split (see "Saving compensation between jobs and machines").

**Single-flute vs multi-flute** (Jason's terms). A single-flute thread mill has the lowest cutting
force and the most chip room: right for small, deep or tough threads, and for keeping deflection
low; multi-flute is faster. Both exist as single-profile and multi-form. The planner picks by the
deflection budget: the largest flute count whose predicted pitch-diameter error (deflection +
servo) stays under a set fraction of the tolerance (for example a quarter; opinion).

### 7. Single-point threading on lathes

Kennametal's threading data (via MSC) gives the infeed methods and the constant-chip-area rule:

- Radial (0°), flank (30°), **modified flank** (thread angle minus 3 to 5°, e.g. 29.5° for 60°
  threads) and alternating flank; chip-control inserts need 20 to 30°, recommended 28 to 29.5°.
- Accumulated depth after `n` passes `D_n = d_1 · √n`; pass `n` infeed
  `d_n = d_1 · (√n − √(n−1))`; a first pass of about 25 % of thread depth.
- Minimum pass 0.13 mm for chip control, at least 0.05 mm on last passes to avoid rubbing and
  work hardening; spring pass 0.02 to 0.08 mm.
- A pass-count table by pitch (4 to 5 passes at 0.5 to 0.75 mm pitch up to 15 to 25 at 8 mm for
  V-threads; row assignment to thread types to be checked against the source).

The module reuses the same physics engine: each pass is an edge-force cut with a known chip area,
so force and deflection on the last pass give the X compensation needed for size (derived).

### 8. External threads

- **Processes**: single-point turning (above), thread milling (`F_centre = F_edge·(D+d)/D`),
  dies/die heads and rolling (outside the mill module, but the planner should know the blank
  diameter they need, opinion).
- **Blank diameter**: an external thread is cut on a blank slightly under the major diameter;
  for class 2A/6g the allowance already makes the major diameter smaller than basic (fact for
  the existence of the allowance; exact figures from the standard).
- **Start chamfer**: chamfer the end to just below the minor diameter so the first thread is not
  a sharp half-turn (common practice; exact angle and depth vary by drawing standard, unverified).
- **Runout to a shoulder**: a thread undercut (DIN 76 forms; unverified detail) lets a full thread
  reach the shoulder; without one, the drawing must accept incomplete threads over a length the
  module can compute from the tool nose and lead.
- **Coatings and plating**: a plated external thread must be cut smaller before plating
  (allowance positions such as 6g/6e exist partly for this; fact for the principle). The target
  size comes from the
  [physical properties](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-uxzal2/research/design-research-nsnlookupjn-physical-properties.md)
  file's coating thickness: for a 60° thread a coating of thickness `a` on both flanks changes
  pitch diameter by `4a` (Lubero's coating formula gives the tap oversize `h = 2·a/sin α` with
  α the half flank angle, i.e. `4a` for 60°).

### 9. Tapered threads: NPT, NPTF, BSPT

- **Taper 1:16 on diameter** (0.0625 per unit length; half angle 1.79°). One turn of axial
  engagement (one pitch) changes diameter by `P/16` (derived).
- **Gauging** (Thread Check technical page): L1 plug and ring gauges check pitch diameter at
  hand-tight engagement, **±1 turn** from the notch (plug) or the small end face (ring) for NPT
  and NPTF; NPTF adds L2 (ring) / L3 (plug) at ±½ turn from the L1 position for taper and
  wrench-tight engagement, and 6-step gauges for crest and root truncation (Class 2). NPT is
  sealed with tape or compound and needs no crest check.
- **So size is a position, not a diameter.** A taper thread is "big" when the plug goes in too
  far. One turn of standoff equals `P/16` on diameter, `P/32` on radius:

| Size | TPI | P (mm) | Δ diameter per turn | Δ radius per turn |
|---|---|---|---|---|
| 1/8-27 NPT | 27 | 0.941 | 0.059 | 0.029 |
| 1/4-18 NPT | 18 | 1.411 | 0.088 | 0.044 |
| 1/2-14 NPT | 14 | 1.814 | 0.113 | 0.057 |
| 1-11.5 NPT | 11.5 | 2.209 | 0.138 | 0.069 |

(derived from the 1:16 taper; values in mm)

- **Taper taps** cut over their whole engaged length, so they have the highest torque of any
  tap; a taper pipe reamer before the tap lowers it (common practice). **The size of a
  taper-tapped hole is set by how deep the tap goes**: one turn of gauge error equals one pitch
  of Z. Tap wear compensation for NPT is therefore a **Z offset** (go deeper as the tap wears),
  not a tap change (derived; opinion that it should be automatic).
- **Taper thread milling** uses a tapered cutter (1:16 tools exist; CTE item) or a single-profile
  cutter on a tapered helix. Either way the size correction is a radius correction `Δr = k·P/32`
  for `k` turns of gauge error. Several CAM forum threads show users struggling to set this up
  (Mastercam and BobCAD forum titles; pages not studied); a module that takes "gauge reads +¾
  turn" and applies the radius change is a clear win (opinion).
- **BSPT** (ISO 7-1, 55°, Rc/Rp/R) follows the same 1:16 arithmetic; Rp (parallel internal) mates
  with R (taper external).

### 10. Lead-ins, chamfers, partial threads and thread start

- **Hole entry chamfer** (countersink) to slightly more than the major diameter, so the first
  thread is not a feather edge and a form tap's raised crest does not stand proud (common
  practice; exact size unverified). Form taps push material up at the entry; the module should
  put the countersink **before** a form tap, and a light deburr pass after it when the drawing
  asks (opinion).
- **Thread mill lead-in**: tangential arc entry with Z moving on the helix, so the tool never
  plunges radially into full depth (fact for the practice; CAM help pages list ramp, arc angle
  and "wind fan" settings). The arc radius and angle set the entry chip; the planner checks it
  against the same chip-load limit as the cut.
- **Partial threads**: a thread that must stop short of a face, a cross-hole or the bottom.
  Taps leave chamfer threads incomplete at the bottom; thread mills can stop at a programmed
  Z within about one pitch; single-point turning leaves a lead-out the length of the pull-out
  move. The CAD feature should carry "full thread length" and "allowed incomplete length"
  separately, and the module reports which process meets them (opinion).
- **Thread start angle (phase)**: thread mills and lathes can start a thread at a chosen angle
  (lathes use the spindle index pulse; multi-start threads shift it); rigid taps start wherever
  the spindle happens to be unless the control orients it first (unverified per control). It
  matters for re-chasing a thread, for paired parts and for threads that must stop at a defined
  clock position. A "start angle" field on the thread feature would let CAM honour it (opinion).

## Accuracy

What each process can hold, and what moves it (fact for directions; magnitudes as cited or
derived above):

| Process | Size set by | What drifts it | Can CAM correct it? |
|---|---|---|---|
| Cut tap | tap's H-limit/class marking | wear (smaller), built-up edge, sync thrust (oversize, thin flanks), runout (oversize) | only by choosing another tap or a holder with float |
| Form tap | tap + drilled diameter | drill size (minor diameter/crest fill), wear, lubrication | yes for the drill (bore or ream to a target); no for pitch diameter |
| Taper tap | depth | wear (smaller at the same depth) | yes: Z offset per gauge turn |
| Thread mill | radius | wear (internal small, external large), deflection (taper along length), servo lag (internal small), sync of Z to angle (lead error, small) | yes: radius wear offset; Z-dependent radius with single-profile |
| Single-point turning | X | insert wear, deflection, thermal growth | yes: X wear |

**Gauges.** Go/no-go plug and ring gauges (ISO 1502; ASME B1.2 for unified, catalogue name)
answer pass or fail. **Indicating** gauges give a number: tri-roll comparators for external
threads and segment-type indicating plugs for internal threads exist (unverified names and
availability per size); with a digital indicator and a data port they can feed SolidCraft Edge
(see in-process inspection, "Manual gauging"). For taper threads, the standoff in turns read off
the gauge is already a number.

## Programs

From public help pages and descriptions only (no program was driven):

- **Autodesk Fusion** thread milling: rough and finish passes, stepover, finishing allowance,
  spring passes, cutter compensation (including outputting the part line for in-control comp),
  and lead-in/out ramps and arc angles. No tap torque, chip volume, deflection or wear model is
  described on that page.
- **Hurco conversational** "Mill Thread" and similar conversational controls: a thread form with
  passes and direction (help page title only studied).
- **Tap monitoring** (Caron TMAC and similar): learns the forward tapping power profile, alarms
  on linear or area-under-curve limits, expires worn tools, checks that the drill reached its
  expected power, ignores reversal spikes. This is a machine add-on, not CAM.
- **Tapping holders** with axial micro-float (Tapmatic SynchroFlex, Big Kaiser Mega Synchro,
  BASS HST Synchro) are a hardware answer to sync thrust; CAM does not know which holder is
  used unless the holder library says so (tool holding).

**What we did not find** in any public description (opinion; not a survey of every program): a
CAM program that (a) checks drill depth and size against the chosen tap's chamfer and the
thread's class, (b) checks chip volume against flute room, (c) predicts tap torque against tap,
holder, spindle and fixture limits, (d) predicts thread-mill taper from deflection and corrects
it, or (e) keeps a thread cutter's offset as data that follows the tool between jobs and
machines. That is the gap behind Jason's "point and splat".

## Complaints and requests

- **Jason (shop owner):** threading is where he breaks the most tools, because no CAM does the
  physics.
- Taps break from undersize or misplaced holes, wrong geometry for the hole, chips that do not
  clear, sync and reversal errors, misalignment and runout, wrong data and worn tools, and form
  taps in the wrong hole or material (Protool's guide, summarised in Fundamentals).
- Rigid tapping wears holders and taps as machines age (Production Machining, MMS).
- Users ask on forums how to set the hole diameter and comp for NPT thread milling in mainstream
  CAM (thread titles; not studied).
- Thread mill tapered threads from overhang (ISCAR troubleshooting).

## What users need

### Jason's request

"Have a module which designs threading process from hole-making through threading by
single-flute mill, multi-flute mill, straight tap, forming tap, spiral tap, etc. Think of ways to
prevent thread cutting failure. Think of tricks like an operation to clean a tap with a fixed
brush in machine and dipping tap into threading compound in machine. Think about lead-ins,
partial threads, chamfers, external threading, tapered threading like NPT [...] Think hard about
wear compensation in machine to adjust thread fits. Can there be a feedback loop with [SolidCraft Edge]
to dial in a cutter for a thread and save compensations permanently between jobs and even
machines."

### The module: from feature to plan (opinion)

Input: a SolidCraft hole or thread feature (it already knows designation, pitch, depth and
whether it is blind; it should also carry **system, class, handedness, full and incomplete
length, start angle, entry chamfer**, as the inspection file proposes for tolerances), the setup
(material and hardness region from the physical properties file, workholding, machine) and the
shop's tool library.

1. **Enumerate candidate chains.** For each process the library can do on this machine:
   spot → drill (cut or form size, possibly bore/ream) → chamfer → [clean hole] → thread →
   [clean tool / dip] → [gauge]. External threads: turn/mill blank → chamfer → thread → gauge.
2. **Check every link** with the physics above and the companion files' limits:
   drill depth vs tap chamfer + clearance; drilled diameter vs class minor limits; chip volume vs
   flute room (blind holes); tap torque vs tap, holder, spindle at tapping speed, fixture
   reaction; sync: does the machine rigid tap, and is the holder rigid or floating; reversal
   room; thread mill: centre feed, engagement, deflection and servo-lag error vs tolerance,
   profile distortion; taper threads: depth or radius per turn; form tap: material ductility.
3. **Score** survivors by risk (margin to each limit, scaled by the slider), cycle time and tool
   cost per thread, and **propose one**, with the reason in words: "Thread mill, single flute:
   the M6 spiral-flute tap would need 2.6 N·m against an estimated 4.0 N·m tap limit in this
   17-4 PH; your slider is at Conservative (margin needed 2.0)." (illustrative numbers.)
4. **Generate** the operations, with stations visits (below), peck counts, spring passes and the
   compensation plan, and send them to the simulation gate.
5. **Explain every number** (the umbrella file's rule): the user can open any check and see the
   formula, the inputs and their source.

**Process selection table** the planner starts from (opinion; the physics overrides it):

| Situation | First choice | Why |
|---|---|---|
| Through hole, ductile, rigid tapping | spiral-point tap | chips go ahead and out |
| Blind, ductile, depth ≤ 2 D, chip room OK | spiral-flute tap | chips come up |
| Blind, chip volume > flute room | thread mill, or spiral-flute with pecks | packing |
| Ductile, high volume, no chip allowed (e.g. hydraulic, clean parts) | form tap with exact hole | no chips, strong thread |
| Grey cast iron, brass | straight-flute cut tap | short chips; form tap forbidden in grey iron |
| Hard (> about 45 HRC, unverified), titanium, superalloys, large or expensive part | thread mill | small cutting force, no stuck tap in a finished part |
| Tight class (4H5H, 3B) or a fit to dial in | thread mill | size by radius |
| Large threads (beyond the machine's tapping torque) | thread mill | torque |
| NPT/NPTF on mill | taper thread mill, or taper tap with Z control | size by position |

### Hard limits and the slider

The cutting physics slider moves safety factors; some limits never move (opinion, following the
umbrella file's rule):

- **Never move:** no form tap in a brittle material; drilled depth ≥ tap chamfer + clearance for
  blind holes (a tap that bottoms breaks); drilled diameter inside the class minor limits; no
  tapping on a machine without rigid tapping unless the holder floats; reversal over-travel
  inside the drilled depth; thread start not inside a cross hole.
- **Slider:** tap torque margin, chip-volume margin (how many pecks), thread mill deflection
  budget (fraction of tolerance), spring passes, tool life before replacement.

### Preventing failure: modes, signals, rules

| Failure | Physics / cause | Rule in the planner | Signal at the machine (SolidCraft Edge) |
|---|---|---|---|
| Hole not drilled / broken drill | upstream tool lost | drill load must reach expected power before tapping (Caron) | drill power below the learned envelope → stop before the tap |
| Undersize hole | wrong drill, worn drill | drill diameter in class limits, form-tap tolerance | tap torque from the first full tooth above model |
| Shallow hole / chips at the bottom | drill depth, drilling chips left in a blind hole | depth = thread + chamfer + clearance + point; air blast or flush the hole before tapping | torque rising sharply near the bottom |
| Chip packing | chip volume > flute room | choose spiral-flute or thread mill, or pecks | torque rise with depth steeper than this hole's history |
| Bottoming | depth error, wrong tool length | simulation checks the tap tip against the drilled point | torque spike at bottom |
| Sync thrust | machine sync error, rigid holder | prefer micro-float holders; trend thrust per machine (machine health) | axial servo load during tapping (if the control reports it) |
| Reversal break | chips sheared on reversal, abrupt stop | peck before full depth in deep blind holes; reversal speed limit | reversal torque trend (recorded separately) |
| Misalignment / runout | holder, collet, datum | holder runout from library; spot drill; tap axis normal to the face | radial vibration (accelerometer) |
| Worn tap | wear | life in threaded length; retire before torque becomes unstable | mean torque trend per hole |
| Holder slip | friction collet over its slip torque | torque < slip torque / SF, or square-drive collet | sudden torque drop with spindle still turning |
| Part rotation | fixture reaction torque | fixture check with the tapping moment | part movement probe after |
| Thread mill break | wrong centre feed, entry plunge | centre-feed conversion; arc entry; chip load per pass | spindle load spike at entry |
| Thread mill taper | deflection | spring passes, single-profile with `r(z)` | go gauge stops before full depth |
| Galling / built-up edge (stainless, titanium, aluminium) | lubrication | compound dip or through-coolant; coatings | torque noise and rise |

**Fact** for the causes (Protool; Springer; Caron; MMS); the rules and signals are opinion.

### In-machine tricks as stations (Jason's ideas)

Jason's examples, a fixed brush that cleans the tap and a cup of tapping compound the tap dips
into, are both "go to a fixed place in the machine and do something with the tool". The module
should treat them as **stations** in the machine library, the same way a tool setter or a probe
calibration sphere is a fixed object (opinion):

- **Station record**: type (brush, compound cup, air nozzle, chip vacuum, tool setter), position
  in machine coordinates (or on a fixture plate), envelope for collision checking, approach
  direction, consumable level (compound), what tools it suits (diameter range).
- **Tap brush**: the tap enters the brush **turning in reverse at low speed** while moving
  axially, so the bristles sweep chips out of the flutes without loading the cutting edges
  (opinion on direction; unverified as standard practice). Interval: every N holes, where N
  comes from the chip-volume check (a straight-flute tap in a blind hole: after every hole;
  spiral-flute in a through hole: rarely).
- **Compound dip**: the tap dips to the length of its chamfer + engaged threads, turning slowly
  so it coats all flutes, then waits a moment for excess to drip (opinion). Interval from film
  life (holes per dip; learned). Cautions: tapping compound must suit the material (chlorinated
  compounds are commonly avoided on titanium; unverified as a universal rule) and must not ruin
  the machine's coolant; the cup's level needs a check (a level sensor on SolidCraft Edge, or a
  count of dips since refill).
- **Air blast before tapping**: blow drilling chips out of blind holes before the tap goes in
  (common practice). A through-spindle air or coolant flush works too.
- **Hole check before tapping**: a drill power check (above), or a quick probe or tool-setter
  check that the drill is whole after drilling (common in unattended cells; see cell
  automation).
- **Chasing pass**: run the tap (or thread mill) a second time through a finished thread to clean
  it (opinion: only rigid taps that can re-sync phase, or thread mills).

CAM inserts the visits automatically, shows them in the operation list as children of the
thread operation, and the simulation checks the moves like any other. Cycle-time cost is shown,
so the user can see what safety costs.

### Wear compensation and thread fit

**Which way each process drifts (derived from the accuracy table):**

- Internal thread, thread mill: cutter wears smaller → pitch diameter gets **smaller**.
- External thread, thread mill or turning: → pitch diameter gets **larger**.
- Cut tap: wear → smaller (and torque higher).
- Taper tap: wear → smaller at the same depth → go deeper.

**Where to aim.** Two rules (opinion):

1. **Start on the safe side.** A thread that is too small (internal) or too large (external) can
   be re-cut; a thread that is too big (internal) is scrap. A new thread mill's first part is cut
   with an offset that puts the pitch diameter in the lower part of the tolerance (internal), then
   gauged, then opened up in steps.
2. **Run near the end of the tolerance the wear moves away from.** An internal thread drifts
   small, so after dialling in, aim in the upper half of the band; external, the lower half. This
   gives the longest run before the next correction. The inspection file's "target the middle"
   default is right for features that drift both ways; threads drift one way.

**Dialling in from go/no-go (opinion; worked example derived).** M8×1.25-6H, thread mill,
`TD2 = 160 µm` (unverified table value).

- Pass/fail gauges tell you only which third you are in. Use a step of about a sixth of the band
  on diameter (27 µm, i.e. 13 µm on radius) and a cap on steps.
- Part 1: offset from the model (wear 0, deflection and servo terms predicted: +29 µm radius
  from the examples above, split as about 13 µm deflection at the tip + 16 µm servo lag). Go
  gauge enters 10 mm of 16, stops: the bottom is small (taper from deflection). Planner response:
  add a spring pass, or with a single-profile tool raise `r(z)` at the bottom; re-cut.
- Part 1, re-cut: go enters fully, no-go does not start: pass. With pass/fail only, the module
  knows the size is somewhere in the band; it opens up one more step (+13 µm radius) only if the
  history says it is in the lower third (go was tight; operator can answer "tight / normal /
  loose" at the prompt, opinion).
- With an **indicating** gauge, the reading goes straight into the compensation-in-cycle
  pattern from the inspection file: target, dead band, feedback fraction, correction limit, and
  re-cut.

**Wear model.** SolidCraft Edge records, per thread cutter, every thread cut (helix length in that
material) and every gauge result. The slope of size against helix length is the wear rate (µm
per metre); the module applies predicted wear between gauged parts and gauges every N parts,
where N comes from how fast the predicted drift reaches the dead band (opinion). This is the
inspection file's "Self-tuning CAM operations" applied to threads.

### Saving compensation between jobs and machines (Jason's loop)

The obstacle to "save the compensation permanently" is that a single wear-offset number mixes
several causes, and only some of them follow the tool (derived from the physics above). The
module saves the parts separately:

| Part | Depends on | Follows | How it is known |
|---|---|---|---|
| Ground size error | the individual tool | the tool (tool tag / serial) | measured on a presetter, or learned from the first gauged thread |
| Wear | the tool's usage | the tool | wear rate × helix length since new or regrind |
| Deflection | tool + holder stick-out + material + radial depth | the tool assembly, scaled by material and pass plan | physics, calibrated by gauge results |
| Servo / interpolation | machine, feed, path radius, feed-forward setting | the machine | ballbar or learned (machine library, machine health) |
| Sync / lead (taps, rigid) | machine | the machine | thrust trend; holder choice |
| Spring-back / closing | material lot (stainless, plastics close up after cutting) | the material | learned per material grade; scaled by thread size |
| Experience value | anything left | the combination | the residual, kept small and visible |

Then:

- **Same tool, new job, same machine:** reuse ground size + wear; recompute deflection for the
  new material and stick-out; servo term for the new feed and path radius. First part still
  starts on the safe side, but by one step instead of the whole band.
- **Same tool, another machine:** keep the tool terms; swap the servo and sync terms for the new
  machine's values. If the new machine has no ballbar data, start on the safe side and learn.
- **New tool of the same type:** reuse deflection and the machine terms; ground size from the
  presetter or from the safe-side first part; wear from the type's learned rate.
- **Regrind or replace:** the tool tag tells the module; wear resets, ground size re-measured.

Storage: in the **shop instance** of the tool in the tool library (keyed by the tool tag), with
history, who approved each change and the evidence (gauge results, part IDs), never in the
shared catalogue (the same rule as the physics system and inspection files). The control's
own wear offsets are written by the post-processor or SolidCraft Edge at the start of the job from
this record, so the control and SolidCraft never disagree (opinion; needs the control
integration the inspection file describes).

**Taps.** A cut tap has no size offset, but the loop still helps: the tap's torque history
predicts when it will cut small or break, and the gauge history confirms it. Retire on the
earlier of the two (opinion). Taper taps get the Z offset, which follows the tap the same way.

### Data model sketch (opinion)

```
ThreadSpec            system, designation, pitch/TPI, class, hand, starts,
                      full_length, incomplete_allowed, start_angle, entry_chamfer, taper (none | 1:16)
ThreadProcessPlan     chain: [Spot, Drill(size, depth), Chamfer, Clean(station), Thread(op), Clean/Dip(station), Gauge]
                      checks: [{name, value, limit, margin, source}], binding_limit, slider_position
ThreadOp              Tap(type, chamfer_form, peck_plan, holder, reversal_speed)
                      | ThreadMill(type: single-profile|multi-form, flutes, passes, spring_passes, r(z), centre_feed)
                      | SinglePoint(infeed_method, passes)
Station               kind, pose, envelope, consumable_level, tool_range
ThreadCompRecord      tool_tag, ground_size, wear_rate, wear_since_new, deflection_cal,
                      per_machine: {servo_term, sync_term}, per_material: {springback},
                      history: [{part_id, gauge_result, applied_offset, approved_by, time}]
```

### Phased plan (opinion)

1. **Design side**: thread system, class, hand, full/incomplete length and start angle on the
   hole/thread feature; cut-tap and form-tap drill per size in the catalogue.
2. **Planner, checks only**: given a chosen tool, run the drill-depth, drill-size, chip-volume,
   torque and centre-feed checks, with explanations.
3. **Planner, proposals**: candidate chains and the selection table, slider-aware.
4. **Thread milling physics**: deflection and servo terms, spring passes, `r(z)` for
   single-profile tools; taper threads by turns.
5. **Stations**: brush, compound cup, air blast, hole check in the machine library; automatic
   visits.
6. **Loop**: SolidCraft Edge torque envelopes per hole, gauge entry (manual and indicating gauges), the
   compensation record and its transfer between jobs and machines.

### Fit with SolidCraft

- The tapped-hole catalogue and the Thread feature already exist (docs/thread-catalogue.md,
  docs/cosmetic-threads.md); classes, handedness, NPT and sealing are out of scope today
  ("Rc, external R, NPT, thread classes, handedness and sealing behavior are outside this
  tranche").
- No CAM is merged yet; related open work in the upstream repo includes the 2.5D CAM RFC (#1153)
  and PR #4158 (2.5D CAM core with a basic tool library). The threading planner belongs on top
  of that core as a hole-feature operation (opinion).
- Everything is a command and runs headless (AGENTS.md), so the planner's checks are also
  available to agents over MCP: "plan threads for this part" returns the chains and the reasons.

## Export control, sanctions and government use

*Added 2026-10-29 from the [government use research](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-qk5riz/research/gov-research-nsnlookupjn-government-use.md). Opinion and policy proposals, not legal advice.*

**Our three rules:**

- **No one goes to jail.** Our team has no in-house counsel. So wherever a rule is unclear we take the most conservative reading and refuse rather than warn. Questions that need an outside export-control lawyer are marked as such, and free official help (BIS, DDTC, OFAC) is the first call.
- **Help users comply.** SolidCraft stops and explains in plain language, using rule packs for each jurisdiction that the administrator picks. It flags conflicts between countries' laws, never resolves them, and never gives legal advice.
- **Prevention first.** Nothing in this file that is hosted, shared, paid or networked ships until its compliance gates are in place and checked: region blocks, screening, ITAR flag handling, government-mode behaviour, and an outside lawyer's sign-off for any hosted or paid service.

**What in this file crosses a border:** local threading features and CAM; standards references only.

- **No hosted service is proposed here.** If one is added later, it blocks comprehensively sanctioned regions, plus Russia and Belarus, and screens accounts and payers against the US Consolidated Screening List and the UK, EU and UN lists.
- **No support there.** No support, consulting or custom work for users in sanctioned regions or for listed parties.
- **Government mode.** At the Controlled level (CUI and allied equivalents) online parts are off or point to the organisation's own mirror; at the Isolated level they arrive only as signed packs on media; a design flagged Export controlled (ITAR/EAR or national equivalent) never leaves through them.
- **Only generic data is shared.** Nothing derived from a controlled design (geometry, toolpaths, part or drawing numbers, test or inspection results) goes into a shared library, report or issue. Tool data stays shareable through the field allow-list in the high-security research.
- **Code stays public.** Every build the project ships is public, which keeps it "published" software outside the EAR. The rules are tracked by the standing export and sanctions watch.
- **Checked for controlled data (2026-10-29):** this file contains no controlled technical data. Its examples are public sources, generic values or made-up stand-ins, and the author's experience is stated in general terms.

## Glossary

- **Chamfer form**: how many tapered threads start a tap (A to E/F).
- **Chasing**: re-running a threading tool through a finished thread.
- **Dryseal (NPTF)**: taper thread that seals by metal contact at crests and roots.
- **Form (roll) tap**: tap that displaces material instead of cutting it.
- **Indicating thread gauge**: gauge that reads pitch-diameter deviation as a number.
- **L1 / L2 / L3 gauges**: taper thread gauges for hand-tight and wrench-tight engagement.
- **Multi-form thread mill**: thread mill with many tooth rows; one helical turn makes the
  full length.
- **Percentage of thread**: radial thread depth left by the drill, as a percentage of full.
- **Rigid tapping**: spindle and feed locked together by the control.
- **Single-profile thread mill**: one tooth row; cuts the length in many helical turns.
- **Spring pass**: a repeat pass at the same size to remove deflection.
- **Station**: a fixed object in the machine a tool visits (brush, compound cup, air nozzle).
- **Sync error**: mismatch between programmed feed per revolution and the tap's lead.

## Open questions

1. Which tap makers publish break torque per tap, and in what form, so the library can hold it
   instead of the estimate?
2. Is the Lubero torque formula's constant consistent with other makers' published torque
   tables (for example by checking a few sizes against a second maker)?
3. Can a touch probe measure an internal thread's pitch diameter with a ball sized like a
   thread wire, accurately enough to close the loop without a gauge? (Some probe software may
   support this; unverified.)
4. What do controls report during rigid tapping (spindle load, Z servo load, sync error) and at
   what rate? (Machine library, per control.)
5. Forming tap torque relative to cutting tap torque: tool makers' published ratios
   (commonly said to be higher; no figure verified here).
6. Brush and compound station designs that already exist commercially, and whether any control
   maker supports them as a cycle.
7. Should start angle, class and incomplete length go on the Hole feature or only on the Thread
   feature (maintainers' call)?

> **[TODO Jason]** Your thread-mill and tap choices by material, your holders, and any numbers
> from your own broken taps (size, material, depth, what you were doing).

## Sources

### Standards (catalogue entries only)

- ISO 68-1, ISO 261, ISO 965-1 (metric basic profile, sizes, tolerances); ASME B1.1 (unified);
  ASME B1.15 (UNJ); ASME B1.20.1 (NPT); ASME B1.20.3 (NPTF); ISO 7-1 (BSPT); ISO 228-1 (BSPP);
  ISO 1502 and ASME B1.2 (gauges); DIN 2197 / ISO 8830 (tap chamfer forms); DIN 76 (thread
  runouts and undercuts).

### Tool makers, gauge makers and vendors (accessed 2026-10-26)

- Lubero, Calculation formulas – tapping and thread forming: https://www.lubero.de/en/technology/calculations
- Protool, Tap breakage causes and how to prevent them: https://www.protool-ltd.co.uk/blogs/blog/tap-breakage-causes
- Kennametal threading technical data (via MSC): https://www1.mscdirect.com/images/solutions/kennametal/tnThreadingTechData.pdf
- ISCAR, Thread milling information: https://www.iscar.com/eCatalog/Ecat/datafileENI/INFO/THREAD%20MILLING.pdf
- Thread Check, NPT and NPTF gages technical information: https://www.threadcheck.com/npt-nptf-gages/technicalinfo/
- Caron Engineering, Tool monitoring for drilling and tapping: https://www.caroneng.com/news/tool-monitoring-drilling-and-tapping-operations/
- CTE, 1:16 tapered solid carbide end mills: https://ctemag.com/node/41635
- BASS HST Synchro (brochure): https://www.targikielce.pl/storage/file/core_files/2024/3/8/29108833b44e3ede3a655a2bbe835aae/bass-en-hst-synchro.pdf
- Guhring, Tap Drill Sizes; Yamawa European catalogue (as cited in docs/thread-catalogue.md)

### Papers

- Measuring and modelling of process forces during tapping using single tooth analogy process,
  Production Engineering (Springer), 2020: https://link.springer.com/article/10.1007/s11740-020-01004-4

### Trade press

- Modern Machine Shop, Reducing thrust loads during rigid tapping: https://www.mmsonline.com/articles/reducing-thrust-loads-during-rigid-tapping
- Production Machining, A fix for rigid tapping errors: https://productionmachining.com/articles/a-fix-for-rigid-tapping-errors
- American Machinist, Formula: taps' drill sizes: https://www.americanmachinist.com/practical-ideas/formula-taps-drill-sizes
- Modern Machine Shop, A taper thread milling custom macro (introduction only read): https://mmsonline.com/columns/a-taper-thread-milling-custom-macro

### Programs' public help and community

- Autodesk Fusion, Thread Mill page (help): https://help.autodesk.com/cloudhelp/2025/ENU/FCAM/files/GUID-3BAFCC88-0773-4D06-A35D-38B8843A8D31.htm
- Hurco, Mill Thread (help page title): https://support.hurco.com/wp-content/uploads/hurco_mill/Conversational_Programming/Mill_Thread.htm
- eMastercam forum, Form tapping drill size formula: https://www.emastercam.com/forums/topic/104971-form-tapping-drill-size-formula/
- Forum titles on NPT thread milling setup (not studied): https://forum.bobcad.com/t/npt-thread-mill/1347 ; https://industrialmonitordirect.com/blogs/knowledgebase/mastercam-npt-thread-mill-hole-diameter-configuration

### SolidCraft files read

docs/thread-catalogue.md, docs/cosmetic-threads.md, docs/roadmap-detail.md, AGENTS.md.
