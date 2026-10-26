# CAM research: workholding physics, or "have CAM tell us how to hold the work"

Research for [#1621](https://github.com/tangency-labs/solidcraft/issues/1621) (Research Request
for Plugin: CAM). This file covers one part of that request: **how a part is held on the machine,
the physics that decides whether that hold is good enough, and how CAM could recommend or check
the workholding** instead of leaving it entirely to the machinist. Other CAM topics are out of
scope here.

It is one part of a set that together describes machining as a complete physical system (see
"The complete physical system" below): tool library, cutting physics, workholding, stock
structural integrity and the machine library. It pairs most closely with:

- [Cutting tool physics](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-cutting-physics/research/cam-research-nsnlookupjn-cutting-physics.md)
  (section "Outputs for workholding"): where the cutting forces, torque, power and tooth-passing
  frequency used below come from, and the conservative-to-aggressive risk slider.
- [Shared cloud tool library](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-tool-library/research/cam-research-nsnlookupjn-tool-library.md):
  where tool, holder and cutting-data records live.

Licence: MIT OR Apache-2.0, the same as SolidCraft (see the Licence section of CONTRIBUTING.md).

## About this research

- **Author:** Jason ([@nsnlookupjn](https://github.com/nsnlookupjn)). Idea and direction by Jason;
  source research and writing done with an AI assistant (Claude) in October 2026.
- **How it was produced:** web searches of public sources (fixture makers' public engineering
  pages, vise manuals, CAM vendors' public pages and forums, published papers' abstracts),
  each claim checked against the cited page where possible. No commercial CAD/CAM program was
  driven or scripted, no GPL/LGPL/AGPL source was read, and no text is copied beyond short
  quotes. Standards are described from their public catalogue entries, not their paid text.
  The worked examples are textbook mechanics computed for this file.
- **Units:** millimetres, newtons, newton-metres, watts, hertz, radians unless a source is quoted
  in other units (then converted: 1 lbf = 4.448 N).
- **Marking:** "(unverified)" marks a claim not confirmed against a source. **Fact**,
  **opinion** and **marketing** are labelled where they mix.

> **Personal experience (Jason):**
>
> - **Turning:** on a CNC lathe the hold is almost always a 3-jaw chuck with straight jaws, a
>   3-jaw chuck with custom soft jaws, or a collet; rarely anything else.
> - **Milling:** the default is a vise, with straight jaws for the first setup and custom soft
>   jaws for later setups. For production I use rail systems for single and multi-part setups
>   where I can. I often set up extra vises on the table at angles so the spindle can reach
>   several faces without the cost of a 4th or 5th axis.
> - **Rotating the vise for heavy roughing:** I often turn a vise to an angle where the fixed jaw
>   takes most of the force vector. This cuts workpiece vibration and chatter and stops the work
>   slipping.
> - **Dovetails** for smaller parts. I try to **avoid fixture plates** because of accuracy and
>   wear. **Zero-point systems** are excellent but generally expensive and mostly for large
>   production runs.
> - **The real problems:** pull-out under heavy roughing, and deformation of the stock in later
>   setups when the clamping force exceeds the structural integrity of the already-machined stock.

## Summary

0. **Why this matters: novices cannot see the limits.** A novice CAM programmer has little idea
   of the limits of a tool or of the workholding (Jason's observation). Experienced machinists
   carry those limits in their heads: how hard a 6" vise can grip, how shallow a grip is too
   shallow, which direction to push toward the fixed jaw. A novice sees a toolpath that looks
   fine on screen and learns the limits by scrapping parts or breaking tools. If CAM computes the
   limits and explains them, that knowledge becomes visible to everyone.
1. **The gap.** CAM programs compute toolpaths from the cutting tool, the part and the stock.
   Workholding enters them mostly as **geometry**: a fixture body to avoid and to show in
   simulation. In the public sources reviewed, none checks whether the hold is **strong
   enough** for the forces the toolpath produces. That judgement is left to the machinist.
2. **The physics is simple enough to automate.** Most holds are either **friction holds** (vise
   jaws, vacuum, magnets, strap clamps, lathe chucks), where the part stays put only while the
   cutting force is below *friction coefficient × clamping force*, or **form holds** (locators,
   positive stops, dovetails, pins), where geometry carries the load. Checking a hold means
   comparing each cutting load with what the hold can resist in that direction, with a safety
   factor. The textbook figure for that factor is **2** (Carr Lane, Schmalz).
3. **The cutting forces already exist.** A physics-based CAM computes, per move, the cutting
   force vector, the spindle torque and the tooth-passing frequency (see the cutting physics
   file). Moving that force to the part's reference point gives a force and moment ("wrench")
   that the hold must resist. The same numbers then answer three questions: *does it slip, lift
   or turn?*, *does the part or its wall deflect too much?*, and *does it ring (chatter)?*
4. **The hold becomes a feed limit.** The workholding capacity becomes one more limit in the feed
   optimiser: allowed load = min(tool, holder, spindle, machine, **workholding**). The
   conservative-to-aggressive slider moves its safety factor, but never below a floor.
5. **CAM can recommend, not only check.** From the same model CAM can suggest a vise torque,
   a minimum grip depth, a vacuum level, where a locator or positive stop should sit (cut *into*
   the fixed jaw), when to switch to a dovetail or soft jaws, and which operations to run before
   a wall becomes thin. It can also warn of the opposite failure: **clamping harder than needed
   distorts thin parts**, so the best clamp force is the smallest that passes the check.
6. **A shared workholding library makes it practical.** Like the shared tool library, products
   such as a Kurt DX6 vise are described once, collaboratively, as physics-based entries
   (clamp force curve, friction per jaw type, positive stops, stiffness, geometry). CAM then
   **negotiates** between the toolpath's loads and each candidate hold's capacity in every
   direction, and suggests the best hold and an **optimum torque window** that holds safely
   with the least deformation. A **confidence slider** tunes the safety factors.

## Fundamentals

### Locating versus clamping (3-2-1)

A rigid part has six ways to move: three translations and three rotations. Fixture texts count
twelve, one per direction of each ("A workpiece has twelve degrees of freedom that must be
restricted", [Carr Lane](https://www.carrlane.com/engineering-resources/fixture-design-principles/locating-clamping-principles)).

- **3-2-1 locating:** three locators on the largest (primary) face, two on the secondary face and
  one on the third face. Together they restrict nine of the twelve directions; clamps take the
  remaining three ([Carr Lane](https://www.carrlane.com/engineering-resources/fixture-design-principles/locating-clamping-principles)).
- **Division of roles (fact, Carr Lane):** locators "are designed to handle the main cutting
  forces"; clamps hold the part against the locators and resist forces that pull it away, such
  as a drill lifting the part as it breaks through. Clamp force should be directed "into the
  supports or locators", over solid support, to avoid distortion.
- **Consequence for CAM:** a cutting force pointing **into** a locator (a positive stop) is
  carried by geometry and is limited only by the strength of the fixture. A force pointing
  **away** from locators, or along a surface, is carried only by **friction** or by the clamp
  itself, and that is where parts move. The classic rule for a vise follows: cut toward the
  fixed jaw, never push the part toward the moving jaw.

A CAM check can use this directly. For each load, split the force into components along the
locators' inward normals (only non-negative amounts are allowed, since a locator can push but
not pull). Whatever is left over must be carried by friction or clamps. (Opinion: this is the
cheapest useful check and covers most vise and fixture-plate setups.)

### Friction holds

For a part squeezed with normal force `F_N` across `k` contact faces with static friction
coefficient `μ`, the largest tangential load before slip is roughly

```
F_slip ≈ k · μ · F_N
```

A vise has two jaw faces (`k = 2`), so the part resists `2 μ F_clamp` along the jaws and
vertically (plus whatever the parallels carry downward). The safety check is

```
S · |F_tangential| ≤ k · μ · F_N        →        F_N,required = S · |F_tangential| / (k · μ)
```

**Friction is the weakest number in the chain.** Measured static friction at fixture joints "is
not a simple function of the workpiece materials"; it also depends on the shape of the fixture
element, surface texture, clamp force, coolant and joint stiffness
([Xie, De Meter & Trethewey 2000](https://pure.psu.edu/en/publications/experimental-evaluation-of-coefficients-of-static-friction-of-com/)).
Published examples span a wide range: Carr Lane's fixture example uses **μ = 0.19**; a CTE
article uses **μ = 0.6** for clean 1018 steel on 6061 aluminium
([Taylor, CTE, 2022-07-14](https://ctemag.com/node/41595)); Schmalz says no generally valid value
exists for vacuum cups and uses a measured 0.5 in its example
([Schmalz](https://www.schmalz.com/en/support/know-how/vacuum-knowledge/the-vacuum-system-and-its-components/vacuum-suction-cups/design-of-the-suction-cup)).
Coolant and oil lower it (unverified how much). Serrated or gripper jaws raise it by biting
into the part (unverified how much). (Opinion: a CAM default should be the low end, around
0.15–0.2 for smooth hardened jaws with coolant, with the shop able to override it per device and
material pair; the value used should always be shown next to the result.)

### Device by device

| Hold | Normal force comes from | What resists sliding | What resists lifting | Public figures found |
|---|---|---|---|---|
| **Machine vise** (hard or soft jaws) | Screw torque → clamp force (vendor table) | Friction at 2 jaws; positive stop toward fixed jaw | Friction at 2 jaws; jaw lift if gripped high | Kurt HDL6: 10 ft·lb → 1 540 lbf (6.85 kN), 40 ft·lb → 4 300 lbf (19.1 kN), 70 ft·lb → 7 450 lbf (33.1 kN) ([Kurt HDL6 manual via Haas, 2024-05-10](https://www.haascnc.com/content/dam/haascnc/ecommerce-assets/setup-manuals/Kurt-HDL6_Manual_2024-05-10.pdf)) |
| **Soft jaws** (machined to the part) | As vise | Friction plus form lock where the pocket wraps the part | As vise, plus pocket shoulders | Contact stress must stay below yield of jaw and part ([Taylor, CTE](https://ctemag.com/node/41595)) |
| **Dovetail** | Clamp pulls the dovetail into the fixture | Form lock + friction; locating pin | Form lock (angled faces) | 60° dovetail, "less than 1/8" [3mm]" of stock to hold ([Haas Tooling](https://www.haastooling.com/c/dovetail-fixtures), marketing claims "eliminates jaw lift or deflection") |
| **Strap clamps / fixture plate** | Bolt torque through strap lever | Friction under clamps + locators | Clamp force | Carr Lane worked example, 2:1 safety factor |
| **Vacuum** | `Δp · A_sealed` | Friction at gasket or spoilboard | `Δp · A_sealed` | `F = Δp · A` ([Schmalz](https://www.schmalz.com/en/support/know-how/vacuum-knowledge/the-vacuum-system-and-its-components/vacuum-suction-cups/design-of-the-suction-cup)); leaks cut 285 mbar to 150 mbar in one build ([Mekanika, 2023-12-05](https://www.mekanika.io/en/blog/cnc-milling/how-to-design-a-vacuum-table-for-your-cnc-machine)) |
| **Magnetic chuck** | Magnetic pressure × pole contact area; drops with thin or rough parts | Friction (low) + stops | Magnetic pull | Vendors quote "high lateral holding forces" without numbers on public pages ([SCHUNK](https://schunk.com/ca/en/workpiece-clamping-technology/magnetic-clamping-technology/milling-applications/mfps/c/PGR_4391)) |
| **Lathe chuck** | Jaw force, reduced by centrifugal force at speed | Friction torque `μ F_c D/2` | Friction + jaw step | Permissible speed: VDI 3106 ([VDI](https://vdi.de/en/home/vdi-standards/details/vdi-3106-determination-of-permissible-speed-rpm-of-lathe-chucks-jaw-chucks)) |

Notes per device:

- **Vise.** The vendor's torque table is the input (a curve of clamp force against handle torque,
  per vise model). The Kurt manual warns that "clamping only on one side or above the movable
  and stationary jaws can result in jaw lift or loss of accuracy", and that breaker bars and
  impact wrenches damage the screw; "if you need more clamping force you may require a larger
  vise". So CAM should cap the recommended torque at the table's last row.
- **Vacuum.** Only the sealed area counts, not the whole part. Atmospheric pressure is about
  101 kPa at sea level and lower at altitude, so `Δp` can never exceed local ambient pressure.
  Small parts get little force; leaks reduce the vacuum level of the whole table
  ([Mekanika](https://www.mekanika.io/en/blog/cnc-milling/how-to-design-a-vacuum-table-for-your-cnc-machine)).
  As parts are cut free from a sheet, their sealed area falls, which is why routers use onion
  skins and tabs (unverified as a CAM rule, common practice).
- **Lathe chuck.** At speed `n` the jaws' own mass pulls outward, so the effective grip falls:
  `F_c(n) = F_c0 − Σ m_j r_j ω²`, with `ω = 2πn/60`. The check is `S · M_c ≤ μ · F_c(n) · D/2`
  for torque and `S · F_axial ≤ μ · F_c(n)` along the axis. VDI 3106 covers how permissible
  chuck speed is determined; its content is not public, so values must come from the chuck maker
  (opinion).

### Moments: lift, tip and turn

A force acting far from the hold also produces a **moment** that tries to tip or turn the part.
The cutting force `F` at contact point `p`, moved to a reference point `o` on the hold, becomes
the wrench `(F, M = (p − o) × F)`. Three checks follow (simplified, opinion on which matter most):

- **Lift:** upward axial force from a helical end mill or a drill breaking through
  (Carr Lane's example of a drill lifting a part). Resisted by clamp friction (vise) or `Δp·A`
  (vacuum).
- **Tip:** a side load high above a short grip. The moment `F · h` (height above the jaw top) is
  resisted by the jaw contact pressure over the grip depth `d`. Roughly, the part rocks when
  `F · h` exceeds `μ F_clamp · d` (unverified simplification; real jaws also bend). This is why
  shallow grips and tall parts need lighter cuts, or a dovetail or soft jaw.
- **Turn:** torque about an axis through the grip, from a drill, a tap, a face mill or a side
  load acting off-centre. Friction resists it over the contact patch:
  `M_resist ≈ μ F_N · r_eff` per face, with `r_eff` the average distance of the patch from the
  turning axis (for a small patch, about a third of its size, unverified).

### Part deflection and distortion

Even if the part does not move, it can **bend** under the cutting force or under the clamp.

- **Cutting deflection of a thin wall.** As a first estimate, a wall of height `L`, thickness `t`
  and loaded width `b` is a cantilever: `δ = F L³ / (3 E I)` with `I = b t³ / 12`, so
  `δ = 4 F L³ / (E b t³)`. Worked example: 6061 aluminium (`E ≈ 68.9 GPa`), wall 30 mm high,
  2 mm thick, 100 mm long, radial force 100 N at the top: `δ ≈ 0.20 mm`, far beyond a typical
  ±0.05 mm tolerance. Thickness enters cubed: at 4 mm the same load gives 0.025 mm. Walls held at
  their ends behave as plates and deflect less (opinion: a CAM tool should start with the beam
  formula as an upper bound and offer a finite-element check later).
- **Remaining stock matters.** The wall is thinnest at the end of the operation. Research on
  thin-wall milling models the tool partly separating from the deflected wall and the changing
  workpiece dynamics as material is removed ([Wan, Budak & Zhang 2024](https://research.sabanciuniv.edu/id/eprint/49444/)).
  The practical rule CAM can apply: finish a wall while the material behind or beside it still
  stiffens it, and order operations so walls become thin last.
- **Clamping distortion.** Clamp force bends a part that is not flat, or a thin plate, and it
  springs back when released; flatness is then lost. Carr Lane's guidance to clamp over supports
  is the geometric side of this. Optimisation research picks clamp forces and positions to
  minimise deformation at key points under clamp and cutting loads, using finite elements
  ([Liao, SAE 1999-01-0414](https://saemobilus.sae.org/content/1999-01-0414)). The useful
  conclusion for CAM: **more clamp force is not always safer**. The recommendation should be the
  smallest force that passes the slip checks with the chosen safety factor.

### Vibration (chatter) of the part and fixture

Chatter normally lives in the tool and spindle (see the cutting physics file), but a thin wall
or a part on a tall fixture can be the weakest mode instead
([Wan, Budak & Zhang 2024](https://research.sabanciuniv.edu/id/eprint/49444/), which also shows
absorbers and a moving support fixture to suppress it).

- First natural frequency of a cantilever wall:
  `f₁ = (1.875² / 2π) · √(E I / (ρ A L⁴))`. For the 30 mm × 2 mm aluminium wall above
  (`ρ ≈ 2700 kg/m³`): `f₁ ≈ 1.8 kHz`.
- Tooth-passing frequency from the cutting physics file: `f_tp = n · z / 60`. A 3-flute tool at
  10 000 rpm gives 500 Hz; its fourth harmonic (2 kHz) is close to the wall's mode.
- CAM can flag when `f_tp` or a low harmonic lands near the part's estimated mode, and suggest a
  different speed (re-checked against the spindle torque and power curves at the new speed), leaving stock, or a support. (Opinion: a warning is enough at first; full
  part-dynamics stability lobes are research-grade.)

### Worked example: is the vise tight enough?

Inputs (illustrative):

- Cutting physics gives a peak tangential force `Ft = 2 000 N` (for instance 10 kW at
  `v_c = 300 m/min` = 5 m/s, since `Ft ≈ P_c / v_c`) acting along the jaws.
- Smooth steel jaws on aluminium with coolant, `μ = 0.2`; two jaws, `k = 2`; safety factor `S = 2`.

Required clamp force: `F_N = 2 × 2 000 / (2 × 0.2) = 10 000 N` (10 kN, about 2 250 lbf).
From the Kurt HDL6 table, 20 ft·lb gives 2 500 lbf (11.1 kN), so the recommendation is
**about 20 ft·lb (27 N·m)**. On a Kurt DX6 (table in "A shared (cloud) workholding
library" below) the same 10 kN needs **about 30 ft·lb (41 N·m)**, which gives 10.9 kN: the
same job needs different torque on different vises, which is why the curve belongs in the
library entry. If the same force pointed into the fixed jaw, friction would not
be needed at all; if it pointed toward the moving jaw, CAM should flag the move or suggest
reversing the cut direction.

Same load on a vacuum fixture: a 100 × 100 mm gasketed area at 80 kPa gives `F_N = 800 N`, and
with `μ = 0.5` and `S = 2` it can take 200 N sideways, a tenth of the load. CAM should cap the
feed (or the width of cut) for that setup, which is exactly what the shared workholding limit
does.

## Accuracy

What the model can and cannot promise (opinion, based on the sources above):

- **Friction is uncertain by a factor of two or more** (0.19 vs 0.6 in two published examples),
  so the result must carry its `μ` and safety factor visibly. A default of `S = 2` matches Carr
  Lane and Schmalz; Schmalz gives 1.5 as a legal minimum for vacuum handling in Germany and 2.5
  or more when the load swings.
- **Clamp force from torque** depends on screw friction and vise condition; vendor tables are
  for a clean, lubricated vise (unverified, typical for torque-tension data).
- **Cutting forces** carry their own error (see the cutting physics file). With only Kienzle
  data, only the tangential force is known; the radial and axial forces then have to be
  estimated (for example as a fraction of `Ft`, unverified) until edge-force coefficients exist.
- **Deflection and frequency** from beam formulas are upper bounds for simple walls; real parts
  need FEA. SolidCraft already plans FEA research (#1620), which could later supply this.
- The check should be validated against **measured** slips and lifts. A test rig with a force
  sensor, or a simple pull test with a spring scale on a vised part, is cheap (Mekanika pulled
  a vacuum-held board with a 50 kg dynamometer).

## Programs

What the public sources show about CAM programs and workholding. Only public help pages and
forums were read; no program was run.

- **Fusion (Autodesk):** a fixture body chosen in the setup is checked for collisions in
  simulation. For years fixtures were ignored when computing toolpaths; since around 2025, 3D
  toolpaths can avoid fixtures, while 2D toolpaths are not model-aware
  ([Autodesk forum thread, 2017–2025](https://forums.autodesk.com/t5/fusion-manufacture-forum/why-is-it-ignoring-my-fixture/m-p/10168818)).
  No force check was found.
- **Siemens NX CAM 2512:** work-holder libraries, dragging a part into a holder, automatic
  fixture placement and alignment, synchronised simulation
  ([Siemens blog, 2026-02-19](https://blogs.sw.siemens.com/nx-manufacturing/whats-new-in-cam-for-nx-2512-integrated-work-holding-device-management/)).
  The post does not mention clamping force.
- **Research:** computer-aided fixture design (CAFD) has decades of papers on automatic locator
  and clamp layout and clamp force optimisation
  ([review, IJE](https://www.ije.ir/article_72421.html); [Liao 1999](https://saemobilus.sae.org/content/1999-01-0414)).
  Little of it has reached mainstream CAM (opinion).
- **Calculators:** standalone web calculators for vise clamp force exist but are not tied to
  toolpaths (search result only, not reviewed).

**Finding:** geometry (collision, placement, libraries) is common; **force-aware workholding
checks inside CAM were not found** in the sources reviewed. This is the opening for SolidCraft.

## Complaints and requests

From the sources above (more from Jason's experience to come):

- Fixtures ignored by toolpath calculation for years; the workaround was to model fixtures
  oversized and add them as part geometry (Autodesk forum thread above).
- Jaw lift and loss of accuracy when gripping high or on one side (Kurt manual).
- Users adding handle extensions or impact wrenches to get more force, which damages vises
  (Kurt manual): a sign that people guess at torque.
- Vacuum tables losing hold through leaks and small parts (Mekanika).

- **Pull-out under heavy roughing** is the most common real failure, and **crushing or
  distorting already-machined stock** in a later setup when the clamp force is higher than the
  thinned part can take (Jason, personal experience). Both are exactly what the slip check and
  the torque window's upper limit are meant to catch.

## What users need

A proposal for SolidCraft (opinion; command ids follow the repository's `<area>.<thing>` style
and are placeholders, not a design):

1. **A workholding device record** in the setup, stored like a tool-library entry and shareable
   the same way:
   - kind (vise, soft jaw, dovetail, strap clamps, vacuum, magnetic, chuck, collet, custom);
   - geometry (for collision) and **contact patches** with their normals;
   - **locators / positive stops** (which faces can only push);
   - normal force source: a torque → force table (vise), a vacuum level and sealed area,
     a magnetic pressure, a chuck force and jaw mass/radius for the speed loss;
   - friction coefficient `μ` per contact (default low, user-overridable, shown in results);
   - grip depth, jaw height, maximum allowed force (vendor limit).
2. **A per-move check** (`cam.workholding.check`): take the cutting physics outputs per move
   (`Ft/Fr/Fa`, `Fx/Fy/Fz`, `Mc`, `Pc`, `f_tp`, contact point), form the wrench at the hold, and
   compute utilisation for slip, lift, tip, turn, wall deflection and part frequency. Show it as
   a timeline over the program with the worst moves highlighted in the viewport.
3. **A feed limit:** report the largest allowed load per setup and direction, so the feed
   optimiser uses min(tool, holder, spindle, machine, **workholding**).
4. **Recommendations** (`cam.workholding.recommend`): the vise torque from the vendor table, the
   minimum grip depth, the cut direction toward the fixed jaw, the vacuum level or a smaller cut
   on small vacuum parts, "consider soft jaws / dovetail" when grip is too shallow, operation
   order so thin walls are finished last, and tabs or onion skin when a part loses sealed area.
5. **The risk slider** (shared with the cutting physics file): the workholding safety factor
   moves with the slider but has a floor, because a slipping part fails suddenly (scrap, broken
   tool, possible injury) while tool wear fails gradually.

   | Slider | Friction holds (vise, vacuum, magnet, chuck) | Form holds (positive stop, dovetail, pinned) |
   |---|---|---|
   | Conservative | S ≈ 3.0 | S ≈ 2.0 |
   | Default | S = 2.0 | S ≈ 1.5 |
   | Aggressive | S ≈ 2.0 | S ≈ 1.3 |
   | Floor (never relaxed) | S = 1.5 | vendor limit of the device |

   Numbers are a starting proposal; the floors follow Schmalz's 1.5 legal minimum for vacuum
   and Carr Lane's 2:1 recommendation.
6. **Defaults built for novices** (because a novice has little idea of tool or workholding
   limits; opinion on the details):
   - the slider starts at **conservative**, and moving it toward aggressive says what changes;
   - the check runs **automatically** whenever a toolpath is computed, not only on request;
   - an unknown value (friction, torque curve) falls back to the cautious end of its range and
     is labelled "assumed", so the user is told what to measure or look up;
   - every warning is in plain words with the fix, for example "this cut pushes the part toward
     the moving jaw: reverse the direction or add a stop", and points at the move in the viewport;
   - the setup sheet prints the recommended torque, grip depth and stop position, so the person
     at the machine gets the same guidance.
7. **Built into the simulation, so the rules are enforced (Jason's point).** The physics checks
   are not a separate report a novice can skip. The toolpath simulation runs them on the
   in-process stock as it plays, and they decide what happens next:
   - the simulation shows the hold's utilisation per step (colour on the part and a timeline),
     and stops at the first step where the part would slip, lift, tip, turn or distort past
     tolerance, with the reason in plain words and the fix;
   - a program that fails a **hard** workholding limit (below the slider floor, past the vise's
     maximum force, moment over the rotary clamp torque) cannot be posted to G-code until it is
     fixed, unless the user **overrides** it (Jason's decision): the override shows a disclaimer
     naming the limit, the step and the margin, and records the approval (who, when, which
     limit, by how much) in the design and on the setup sheet, so the decision can be traced
     later;
   - **soft** findings (margin below the chosen slider setting) are fixed automatically by the
     feed optimiser where possible and listed where not;
   - the setup sheet carries the conditions the simulation assumed (hold, torque, grip depth,
     stop position, warm-up done), so the machine is set up the way it was simulated.
   This is what protects the work, the tools and the machine from novice errors. The system-wide
   version of this rule, covering tool, spindle, machine and stock limits, is in the
   [CAM Physics System](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-vgqo3g/research/cam-research-nsnlookupjn-physics-system.md)
   file.
8. **Suggesting the workholding itself, not only checking it (Jason's ideas).**
   - **Orientation.** A hold can be rotated on the table, for example a vise at 45°, so the
     fixed jaw takes most of the roughing force (Jason does this for heavy roughing; see
     personal experience). CAM can search orientations: for each candidate angle, rotate the
     predicted force vectors into the hold's frame and pick the angle where the largest share
     of the load goes into the positive stop and the friction utilisation is lowest, then check
     the angle fits the table and the travels (machine library).
   - **Choice by volume.** The best hold depends on how many parts are made: a one-off uses what
     the shop has (vise, straight jaws); low volume justifies soft jaws or a dovetail; high volume
     justifies rails, multi-part fixtures, tombstones or a zero-point system. CAM asks for the
     quantity and ranks holds by **cost per part**: setup time and cycle time (from the physics
     check, since a stronger hold allows faster cuts) plus the hold's purchase or machining cost
     spread over the run.
   - **Investment case.** When a hold the shop does not own would win, CAM shows the payback:
     "a second vise and rail would save N minutes per part; at this quantity it pays for itself
     after M parts". Better workholding is often one of the largest productivity and
     profitability gains available (Jason's point), and the library can carry list prices and
     links so the case is concrete.
   - **A catalogue of options** for the suggestion engine: angled vise jaws, angled parallels,
     soft jaws (with an automatic designer, below), soft jaws or fixture plates with extra
     hardware (pins, stops, toe clamps, supports under thin walls), dovetails, rails, vacuum,
     zero-point. Each is a workholding library entry with its physics.
   - **Automatic soft jaw designer.** From the part's shape at that setup (the in-process stock),
     generate the jaw pocket: contact faces chosen so the main force direction goes into a
     shoulder, pocket depth from the tip and grip checks, relief for burrs and corners, and a
     clamp force that passes the slip check without crushing the part (torque window). The jaw
     is then a CAM job of its own on the same machine.
   - **Multi-vise, multi-angle setups.** Several vises on one table, one in the normal XY
     orientation, one rotated 90°, another rotated 90° the other way and so on (or on angle
     plates), so a part moves from vise to vise and is machined on all sides in one machine
     without a 4th or 5th axis (Jason does this; see personal experience). The soft jaw designer
     should design the set together, each jaw holding the part as it is after the previous
     station, and the checks run per station with that station's force directions.
9. **No dialogs from scripts:** like every SolidCraft command, the check and the recommendation
   run from the JSON form, so agents and scripts can use them.

### A shared (cloud) workholding library

Jason's idea: workholding products get the same treatment as cutting tools in the
[shared tool library](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-tool-library/research/cam-research-nsnlookupjn-tool-library.md).
A product such as a **Kurt DX6 vise** is described **once**, collaboratively, as a
**physics-based entry**, and every SolidCraft user can pull it into a setup. The entry carries
geometry for collision *and* the numbers the physics check needs, so CAM can weigh what the hold
can take against what the toolpath will apply.

Same three layers as the tool library (opinion: reuse the same storage, review and tagging):

1. **Catalogue entry (shared, reviewed):** one per product and size, with cited sources for every
   number.
2. **Shop instance (private):** the user's own vise: serial or asset tag, condition, measured
   jaw parallelism, which jaws are fitted (hard, serrated, soft jaw set N), the torque wrench used.
3. **Setup use (per design):** where it sits on the table, which jaws, grip depth, part position,
   torque applied.

Sketch of a catalogue entry, with the public numbers found for the DX6 (fields without a public
source are left empty on purpose):

```json
{
  "kind": "vise",
  "maker": "Kurt",
  "model": "DX6",
  "sources": ["https://www.kbctools.ca/itemdetail/1-5721-DX6"],
  "geometry": { "step": "kurt-dx6.step", "jaw_width_mm": 152.4, "jaw_depth_mm": 44.07,
                "max_opening_mm": 230.6, "bed_height_mm": 73.0 },
  "clamp_force": {
    "max_N": 32694,
    "torque_curve_Nm_to_N": [[13.6, 2224], [27.1, 6450], [40.7, 10898], [54.2, 15346],
                             [67.8, 19572], [81.3, 24020], [94.9, 28246], [108.5, 32694]],
    "curve_source": "Kurt DX6 clamping force table, supplied by Jason (10-80 ft·lb)"
  },
  "screw": { "max_handle_torque_Nm": null, "no_extensions": true },
  "jaws": [
    { "id": "hard-smooth", "mu_static": { "low": 0.15, "high": 0.25, "source": null } },
    { "id": "grooved",     "mu_static": { "low": null, "high": null, "source": null } }
  ],
  "positive_stop": "fixed_jaw",
  "accuracy": { "repeatability_mm": 0.025, "bed_height_tol_mm": 0.013 },
  "stiffness": { "jaw_lift_N_per_mm": null, "first_mode_Hz": null }
}
```

Public figures for the DX6 from a distributor listing: maximum clamping force 7 342 lbf
(32.7 kN), jaw width 6" (152.4 mm), jaw depth 1.735" (44.1 mm), maximum opening 9.08" (230.6 mm),
repeatability "within .001"" (0.025 mm) ([KBC Tools](https://www.kbctools.ca/itemdetail/1-5721-DX6),
not checked against a Kurt datasheet). The DX6 torque-to-force curve, from Kurt's DX6 data as supplied by Jason, is:

| Handle torque (ft·lb) | 10 | 20 | 30 | 40 | 50 | 60 | 70 | 80 |
|---|---|---|---|---|---|---|---|---|
| Handle torque (N·m) | 13.6 | 27.1 | 40.7 | 54.2 | 67.8 | 81.3 | 94.9 | 108.5 |
| Clamp force (lbf) | 500 | 1 450 | 2 450 | 3 450 | 4 400 | 5 400 | 6 350 | 7 350 |
| Clamp force (kN) | 2.2 | 6.5 | 10.9 | 15.3 | 19.6 | 24.0 | 28.2 | 32.7 |

Its top row (7 350 lbf at 80 ft·lb) agrees with the distributor's 7 342 lbf maximum. Above
10 ft·lb the curve is close to linear, about 95–100 lbf per ft·lb (roughly 0.44 kN per N·m), so
CAM can interpolate between rows; it should not extrapolate past 80 ft·lb. For comparison
the Kurt DX4 manual gives 20 ft·lb → 1 900 lbf (8.5 kN) up to
70 ft·lb → 7 400 lbf (32.9 kN), and says extensions and impact wrenches void the warranty
([DX4/DX4R manual via MSC](https://cdn.mscdirect.com/global/images/ProductDataSheet/pds_sku_365071_installationguide_install.pdf)).
**Filling the remaining fields (friction for each jaw type, maximum handle torque, jaw-lift stiffness) is
exactly what a collaborative library is for**: makers, shops and test rigs contribute measured,
cited values, and reviewers check them, as for cutting data.

### Negotiating between tool physics and workholding physics

Jason's goal: CAM should not only check a hold, it should **negotiate** between what the
toolpath wants (from tool physics) and what the hold can take (from workholding physics), and
end with a suggestion: **the best workholding, and for example the optimum vise torque that
balances holding strength against part deformation.**

**1. Capacity in every direction.** For a given hold and clamp force, the workholding model
returns how much load it can take *in each direction* with the chosen safety factor: an
allowed force for ±X, ±Y, ±Z and an allowed moment about each axis (a "capacity envelope").
Directions into a positive stop get the device's structural limit; friction directions get
`k μ F_N / S`; lift and tip get their own limits. The toolpath calculator receives this envelope
and keeps every move's force and moment inside it, by lowering feed or width of cut where needed
(the workholding term in min(tool, holder, spindle, machine, workholding)).

**1a. The negotiation is continuous along the toolpath (Jason's point).** It is not one check
per operation. As the cutter moves through three dimensions it pushes and pulls the part in
directions that keep changing: into the fixed jaw on one pass, along the jaws on the next, up
out of the vise as a helical tool ramps, around the grip as it turns a corner. The hold is strong
in some directions (into a positive stop) and weak in others (along the jaws, toward the moving
jaw, upward), so **the right tool load depends on the direction of the force at that instant**,
and the workholding physics decides which directions are weak.

How CAM can do this (opinion on the method, mechanics as above):

1. Walk the toolpath in small steps over the **in-process stock** (the stock as it is at that
   point in the cycle, see the stock integrity topic below).
2. At each step take the cutting force vector `F` and moment `M` from the cutting physics, and
   its direction `u = F / |F|`.
3. Ask the workholding model for the **allowed load in that direction**, `F_allow(u)`, after the
   safety factor: very large into a positive stop, `k μ F_N / S` along friction directions, the
   lift limit upward, and so on, together with the moment limits.
4. Scale the feed so the load meets the limit and no more. Cutting force grows with chip thickness
   `h` roughly as `h^(1 − m_c)` (Kienzle exponent from the cutting physics file), and chip
   thickness grows with feed per tooth, so a feed scale of about
   `s = (F_allow(u) / |F|)^(1 / (1 − m_c))` brings the force to the limit (approximate; capped by
   the tool, spindle and machine limits and by the slider).
   **The spindle's torque curve is always respected (Jason's point):** at the step's spindle speed
   `n`, the cutting torque must stay under the curve from the machine library,
   `M_c ≤ M_spindle(n)` and `P_c ≤ P_spindle(n)` (with the slider's margin), read from the
   torque and power curves of that machine's spindle, including its gear or winding range.
   A spindle often has less torque at low speed (unverified for any given machine; it depends
   on the curve), so the workholding check never "frees up" load that the spindle cannot deliver
   at that speed.
   **Warm up before maximum torque (Jason's point):** a cold spindle has unevenly spread
   lubricant and has not yet grown to its working size, so early heavy cuts wear the bearings
   and parts drift in size; "just a few minutes of warm-up" is the maker's advice
   ([Okuma, 2020-12-14](https://www.okuma.com/blog/the-benefits-of-a-proper-spindle-warm-up-routine)).
   The per-step limit should therefore carry a **machine-state gate**: until the machine
   library's warm-up routine has run (or the spindle has been running recently), the spindle
   torque and power limits are derated and the slider cannot move them up; the program can
   start with the warm-up routine or with light operations. The warm-up routine itself, and how
   long the machine may stand before it is needed again, belong to that machine's library entry.
5. Where a different choice would point the force into a strong direction (cut toward the fixed
   jaw, climb instead of conventional, ramp instead of plunge, a different entry side), suggest
   it, or let the path planner prefer it, instead of only slowing down.

The result is a feed that changes **along the path** (written into the program as varying
feed per segment, the way feed optimisers already work, see the cutting physics file), so the
tool works as hard as the weakest of tool, machine, stock and hold allows in **each** direction,
not as hard as the worst direction of the whole operation allows. A further step is closing the loop on the machine with measured data (adaptive control); the
planned model would then be the starting point and the safety net.

**Live feedback from the machine: "self-tuning CAM operations" (Jason's name and idea).** Some controls publish spindle load, axis
loads or torques and vibration while cutting (for example over an open machine-data protocol
such as MTConnect; which machines publish which signals is unverified and belongs in the
machine library). The Raspberry Pi edge box from the in-process inspection topic could collect
these in real time and feed them back. For workholding this gives three things:
- **validation:** compare measured loads with the predicted force per step, so the friction
  values and the torque curves in the library get corrected from real cuts, shop by shop;
- **slip and lift detection:** a part that moves shows as a sudden change in load or a
  vibration signature that the model did not predict; the box can raise an alarm or, where
  the control allows it, ask for a feed hold;
- **part and fixture modes:** vibration spectra show where the hold or a thin wall rings,
  which checks the frequency estimates above.
**Tied to in-process inspection, so the process can go harder without losing accuracy
(Jason's point).** The [in-process inspection](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-y2w06n/research/cam-research-nsnlookupjn-in-process-inspection.md)
topic measures the part on the machine (probing, gauging). Linking those measurements to the
recorded loads shows, cut by cut, what a given load did to size, flatness and surface finish:
for example, that a wall finished at 1.2× the predicted force still measured in tolerance, or
that a higher clamp torque cost 0.01 mm of flatness. Over time that evidence lets the
confidence slider, and the clamp torque window, move toward aggressive **for that shop, machine,
hold and material** while the measured accuracy and finish stay in tolerance, and pulls them
back automatically when a measurement drifts. Roughing can go harder where inspection shows the
finishing passes still recover the size; finishing stays within the deflection and finish
budget the measurements confirm. (Opinion: tuning should only ever use measured evidence from
the same setup type, and the hard floors still apply.)

**Predicting failure and bad work (Jason's point).** The same recorded history can show trends
before they become failures: a hold that needs more torque each week to pass (worn screw or
jaws), rising vibration at the same cut on the same machine (spindle bearings, loose gibs), or
loads drifting from prediction on a known job (tool wear, wrong material). The monitor can flag
the likely cause and order a maintenance check (for example "inspect vise screw and jaw
plates" or "spindle vibration trend") before parts are scrapped. The machine-side detail
belongs to the separate "Predict machine failure and order maintenance" topic (in progress), with the machine library and the CAM Physics System topics.

**Add-on sensors.** Jason's idea of a sensor package for the edge box (vibration, temperature,
coolant concentration by refractometer and coolant temperature) matters for workholding too:
coolant changes the friction at the jaws (see "Friction holds"), so a measured concentration
lets the check pick the right friction value instead of assuming one. The package itself is
covered in the machine monitoring topics.

**Cameras and machine connections (Jason's ideas).** Cameras on the edge box (covered in the
inspection and cell automation topics) could one day watch the part and the hold and detect
slip or lift directly, as a change in the part's position against the jaws (aspirational:
chips, coolant and lighting make this hard). The connections themselves, such as a CAM program
talking to a Haas mill over Ethernet for in-process control and probing, what interfaces
machine makers offer today, and the suggestion that makers add an MCP interface so an edge
device can monitor and control the machine, are researched in the separate
[machine interfaces](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-fz0g0g/research/cam-research-nsnlookupjn-machine-interfaces.md#what-each-control-maker-offers)
topic. Slip and lift detection from those signals stays in this file.

**Aligned with the G-code and the timeline (Jason's point).** Every recorded sample is stamped
so it can be placed on the program: the time, the G-code line or block number being executed,
and the machine position. The simulation's per-step predictions are indexed the same way, so a
measured load jump lands on the exact move (line, toolpath step, position on the part) that
caused it, next to the predicted force and the hold's limit at that step. Clock drift between
the box and the control is corrected by matching known events (program start, tool changes,
the block number the control reports). This is what turns "it chattered somewhere in op 3" into
"line N1234, corner at X…, Y…: load 40 % over prediction, hold margin fell to 1.2".
(Opinion: start with recording and comparison only; letting the box change feeds on the machine
is a later step that needs the control maker's interface and its own safety review.)

**2. A torque window, not a single number.** Torque `T` sets clamp force `F_N(T)` through the
vise curve. Two constraints bound it:

- **Too loose:** below `T_min` the hold fails the slip, lift or turn check for the worst move:
  `F_N(T_min) = S · F_worst / (k μ)`.
- **Too tight:** above `T_max` the part deforms too much or gets marked: the clamp-induced
  deflection (bending of thin or hollow parts, from a beam estimate or later FEA) uses up more
  than its share of the tolerance, or jaw contact stress `F_N / A_contact` approaches the part's
  yield strength (the check Taylor makes in CTE), or the vise's own limit is reached.

If `T_min ≤ T_max`, the **recommended torque is the low end of the window plus the slider's
margin**: the least clamp force that holds safely, which also gives the least distortion and the
best accuracy. If the window is empty, CAM negotiates:

1. lower the load (feed, width or depth of cut, tool choice, climb/conventional, cut toward the
   fixed jaw) until `T_min` falls inside the window, at the cost of cycle time; or
2. change the hold (deeper grip, soft jaws, dovetail, extra support under a thin wall, another
   device from the library), which raises capacity without more squeeze; or
3. report that no safe combination was found, with the limiting move and direction.

**3. Picking the best workholding.** For a part, CAM tries the candidate devices the shop owns
(shop instances from the library), runs the window and negotiation for each, and ranks the
feasible ones. A sensible order (opinion): first must pass all checks; then the shortest cycle
time; then the smallest predicted distortion; then the least setup effort (no soft jaws to
machine). Each suggestion shows why: for example "DX6, hard jaws, grip 8 mm, 30 N·m: holds the
roughing at 2.2× margin, clamp distortion 0.004 mm; soft jaws would allow 25 % higher feed".

**4. The confidence slider.** The slider expresses the user's confidence in the process (how
well they trust the friction values, the torque wrench, the cutting force data and the machine).
It moves the safety factors on the workholding side (table above) together with the tool side
(cutting physics file):

- high confidence → smaller `S` → lower `T_min` → a lower recommended torque (less distortion)
  *and/or* higher allowed feeds;
- low confidence → larger `S` → more torque and gentler cuts;
- the floors never move, and the vendor's maximum clamp force caps the torque at every setting.

Opinion: the slider should show its effect in the setup, for instance "conservative: 40 N·m,
cycle 12 min; aggressive: 25 N·m, cycle 9 min", so the trade-off is visible rather than hidden.

### The complete physical system

System-level design (shared setup, per-step solver, one slider across the chain, roadmap) lives
in the umbrella [CAM Physics System](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-vgqo3g/research/cam-research-nsnlookupjn-physics-system.md)
file; this section keeps the load path and the workholding interfaces.

Jason's view: these topics are one physical system, not separate features. The load path in a
cut runs **spindle → holder → tool → chip → stock → workholding → table → machine axes**, and
the weakest link at any instant sets how hard the tool may work. Each link gets a library
entry or a model, and the toolpath calculator reads all of them:

| Link | What it supplies | Where it is researched |
|---|---|---|
| **Tool library** (tools, holders, cutting data) | geometry, stick-out, coefficients, tool limits | [shared tool library](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-tool-library/research/cam-research-nsnlookupjn-tool-library.md) |
| **Cutting physics** | force vector, torque, power, tooth-passing frequency per step; feed optimiser; risk/confidence slider | [cutting tool physics](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-cutting-physics/research/cam-research-nsnlookupjn-cutting-physics.md) |
| **Workholding library and physics** | allowed load per direction, torque window, best hold | this file |
| **Stock structural integrity** | how stiff and strong the in-process stock is at each point of the cycle; cut sequence that keeps it stiff; deformation under tool and clamp | [stock integrity](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-stock-integrity/research/cam-research-nsnlookupjn-stock-integrity.md) (new topic, in progress) |
| **Machine library** | the machine model: stiffness and available force in each axis, spindle power and torque curves, table load, rotary-axis clamp torque, work envelope | [machine library](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-machine-library/research/cam-research-nsnlookupjn-machine-library.md) (new topic, in progress) |

How workholding ties into the two new topics:

- **Stock integrity.** The hold's limits change during the cycle because the stock changes. A
  part on vacuum loses sealed area as it is cut free; a thin part in a vise distorts more under
  the same clamp force once its centre is pocketed; a wall far from the jaws loses stiffness as
  material behind it is removed. So the workholding check runs on the in-process stock, and
  the cut **sequence** is part of the negotiation: the stock integrity topic plans the order of
  cuts so the stock stays stiff while forces are high, and this file supplies the hold's limits
  for each candidate order.
- **Machine library.** The machine model closes the load path. Its axis stiffness adds to the
  deflection budget, its axis thrust and spindle torque cap the load just like the hold does, and
  on a 4th or 5th axis the **rotary axis brake or clamp torque is itself part of the
  workholding**: a heavy part on a trunnion is held by the vise *and* by the rotary clamp, so the
  moment about the rotary axis must stay under both. The workholding entry's position on the
  table (or on the trunnion) comes from the machine model.

Agreed with those two topics: stiffness combines as a **series compliance chain**,
`1/k_total = 1/k_tool+holder+spindle + 1/k_machine + 1/k_fixture + 1/k_stock`, per direction.
The machine library returns per-axis allowed thrust (N), static stiffness at the table (N/µm)
and dominant modes; the stock integrity model returns local stock stiffness per direction,
stock deflection, its lowest mode, release distortion and an allowed-force envelope from the
tolerance budget, and takes from this file the clamp force, contact and support locations and
the capacity envelope (clamp locations are its boundary conditions). One consequence for the
torque window: `T_max` must be re-checked as the stock thins, and CAM may recommend
**re-clamping at a lower torque before finishing**, which roughing forces would not allow but
light finishing passes do.

**Physical properties of the design and the stock.** The separate "Define physical properties
in design and stock" topic (in progress) covers hardness and other properties per body, surface
and feature, local hardness of the stock and in-process stock, and unmachined (for example cast)
stock surfaces. Workholding reads two things from it: local hardness, which changes the cutting
force the hold must resist, and the surface condition at each contact (cast, sawn, machined),
which changes grip and the friction coefficient used in the checks.

All five read and write through the same setup: one machine (from the machine library), one or
more holds (from the workholding library), the stock, and tools (from the tool library). The
confidence slider applies to the whole chain at once.

### Interface with the cutting physics file

Agreed between the two research files:

| Direction | Quantity | Unit | Notes |
|---|---|---|---|
| Cutting → workholding | `Ft`, `Fr`, `Fa` (peak and mean per revolution, tool frame) | N | `Fr`, `Fa` optional or estimated when only Kienzle data exist |
| Cutting → workholding | `Fx`, `Fy`, `Fz` resultant in part/machine frame | N | |
| Cutting → workholding | `Mc` spindle torque, `Pc` power | N·m, W | |
| Cutting → workholding | `f_tp = n · z / 60` | Hz | for part-mode warnings |
| Cutting → workholding | tool contact point | mm | to form moments about the hold |
| Workholding → feed optimiser | allowed force per direction and allowed moment, after safety factor | N, N·m | the workholding limit |
| Workholding → feed optimiser | lowest part/fixture mode estimate | Hz | |
| Workholding → feed optimiser | wall deflection limit | mm | from tolerance |

## Glossary

- **Workholding / fixture:** whatever holds the part on the machine (vise, plate, chuck, vacuum).
- **Locator / positive stop:** a fixed contact that sets the part's position and can carry load
  pushed into it.
- **3-2-1:** locating with three, two and one points on three faces.
- **Clamp force (`F_N`):** the normal force pressing the part against its contacts.
- **Static friction coefficient (`μ`):** ratio of the sideways force that starts slip to the
  normal force.
- **Safety factor (`S`):** how many times the expected load the hold must resist.
- **Wrench:** a force and a moment together, acting at a reference point.
- **Grip depth:** how much of the part is inside the jaws.
- **Soft jaws:** jaws machined to the part's shape.
- **Dovetail:** a tapered tenon machined on the stock so the fixture grips it by form.
- **Tooth-passing frequency (`f_tp`):** how often a cutting edge enters the part, `n · z / 60`.
- **Natural frequency (`f₁`):** the lowest frequency at which the part or wall rings.

## Open questions

1. Where should default friction values come from? A shared, cited table per device and
   material pair (like the cutting-data catalogue), or measured per shop?
2. Should vendors' vise torque tables live in the shared tool library, next to holders?
3. Is a beam/plate estimate for wall deflection good enough to start, or should this wait for
   the FEA work in #1620?
4. How should the check treat interrupted holds, such as a part that loses sealed area or
   stiffness as it is cut? Re-evaluate on the in-process stock after each operation?
5. How much of the 3-2-1 layout should CAM suggest automatically, and how much should the user
   place?

## Sources

All accessed 2026-10-26.

- Carr Lane Manufacturing, "Guide To Locating & Clamping Principles":
  https://www.carrlane.com/engineering-resources/fixture-design-principles/locating-clamping-principles
- B. Taylor, "Fixture considerations for vise jaws", Cutting Tool Engineering, 2022-07-14:
  https://ctemag.com/node/41595
- Kurt HDL6 vise manual (hosted by Haas), 2024-05-10:
  https://www.haascnc.com/content/dam/haascnc/ecommerce-assets/setup-manuals/Kurt-HDL6_Manual_2024-05-10.pdf
- Schmalz, "Design of the suction cup":
  https://www.schmalz.com/en/support/know-how/vacuum-knowledge/the-vacuum-system-and-its-components/vacuum-suction-cups/design-of-the-suction-cup
- M. Gravet, "How to design a vacuum table for your CNC machine", Mekanika, 2023-12-05:
  https://www.mekanika.io/en/blog/cnc-milling/how-to-design-a-vacuum-table-for-your-cnc-machine
- Haas Tooling, Dovetail fixtures: https://www.haastooling.com/c/dovetail-fixtures
- SCHUNK, magnetic milling chucks (MFPS):
  https://schunk.com/ca/en/workpiece-clamping-technology/magnetic-clamping-technology/milling-applications/mfps/c/PGR_4391
- VDI 3106:2004-04, Determination of permissible speed of lathe chucks (catalogue entry):
  https://vdi.de/en/home/vdi-standards/details/vdi-3106-determination-of-permissible-speed-rpm-of-lathe-chucks-jaw-chucks
- W. Xie, E. C. De Meter, M. W. Trethewey, "Experimental evaluation of coefficients of static
  friction of common workpiece-fixture element pairs", Int. J. Machine Tools and Manufacture
  40(4), 2000, doi:10.1016/S0890-6955(99)00082-6:
  https://pure.psu.edu/en/publications/experimental-evaluation-of-coefficients-of-static-friction-of-com/
- M. Wan, E. Budak, W.-H. Zhang, "Mechanics and dynamics of thin-wall milling process", in
  *A Closer Look at Milling Machines and Processes*, Nova, 2024:
  https://research.sabanciuniv.edu/id/eprint/49444/
- Y. J. G. Liao, "Optimization Methods Applied to Determine Clamping Forces in Fixture Design",
  SAE 1999-01-0414: https://saemobilus.sae.org/content/1999-01-0414
- "Fixture Design Automation and Optimization Techniques: Review and Future Trends",
  International Journal of Engineering: https://www.ije.ir/article_72421.html
- Autodesk Fusion forum, "Why is it ignoring my fixture?" (2017–2025):
  https://forums.autodesk.com/t5/fusion-manufacture-forum/why-is-it-ignoring-my-fixture/m-p/10168818
- Siemens, "What's new in CAM for NX 2512: integrated work holding device management",
  2026-02-19:
  https://blogs.sw.siemens.com/nx-manufacturing/whats-new-in-cam-for-nx-2512-integrated-work-holding-device-management/
