# CAM research: optimising toolpaths by the structural integrity of the stock during a cycle

Research for [#1621](https://github.com/tangency-labs/solidcraft/issues/1621) (Research Request
for Plugin: CAM), with a link to [#1620](https://github.com/tangency-labs/solidcraft/issues/1620)
(FEA), whose solver this topic would eventually use. This file covers one part of the CAM request:
**the stock gets weaker as it is machined, so CAM should track how stiff the remaining material
is, predict how it deflects, vibrates and distorts, and plan the order of cuts so that the stock,
the tool, the workholding and the machine stay inside their limits at every moment of the cycle.**

It is one of a set of files from the same author that describe CAM as **one physical system**:

| Part of the system | File | What it owns |
|---|---|---|
| Overview | [CAM physics system](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-vgqo3g/research/cam-research-nsnlookupjn-physics-system.md) | the shared setup, the solver loop over all limits, the one risk slider |
| Tool library | [Shared cloud tool library](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-tool-library/research/cam-research-nsnlookupjn-tool-library.md) | tool, holder and cutting-data records |
| Cutting physics | [Cutting tool physics](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-cutting-physics/research/cam-research-nsnlookupjn-cutting-physics.md) | force vector, torque, power, tooth-passing frequency per move |
| Workholding | [Workholding physics](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-workholding/research/cam-research-nsnlookupjn-workholding.md) (section "The complete physical system") | clamp force, contacts, capacity envelope per direction |
| Machine library | [Machine library](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-machine-library/research/cam-research-nsnlookupjn-machine-library.md) | per-axis stiffness and thrust, spindle curve, dynamics, machine 3D model |
| **Stock integrity (this file)** | | in-process stock, its stiffness, deflection, vibration and release distortion, and the cut-sequence planner |
| Inspection | [Part certification by in-process inspection](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-y2w06n/research/cam-research-nsnlookupjn-in-process-inspection.md) | probing that checks what this file predicts |

Licence: MIT OR Apache-2.0, the same as SolidCraft (see the Licence section of CONTRIBUTING.md).

## About this research

- **Author:** Jason ([@nsnlookupjn](https://github.com/nsnlookupjn)). Idea and direction by Jason;
  source research and writing done with an AI assistant (Claude) in October 2026.
- **How it was produced:** web searches of public sources (published papers' abstracts and open
  full texts, national-laboratory pages, trade press, machinists' public forums), each claim
  checked against the cited page where possible. No commercial CAD/CAM program was driven or
  scripted, no GPL/LGPL/AGPL source was read, and no text is copied beyond short quotes. The
  worked examples are textbook beam mechanics computed for this file.
- **Units:** millimetres, newtons, newtons per micrometre (N/µm) for stiffness, megapascals,
  hertz, radians, unless a source is quoted in other units (then converted).
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

**This research is not complete.** It is a starting point: the author will keep researching
and adding to it, and others are encouraged to do the same, correct it and contribute their own
findings and shop experience.

**The author's shop experience with stock that moves (personal experience, not a cited source):**

- Subtractive machining always disturbs the internal stresses ("grain forces") in the stock:
  cutting either sets new stress up or releases what is already there.
- **How the stock was made decides how it behaves:**
  - saw-cut, cold-formed stock can behave quite well;
  - forged blanks are very good when the forging is designed for the machining that follows;
  - castings tend to be highly stressed;
  - hot-rolled stock curls like a spring as it is cut.
- **What the process does on top of that:** the order of cuts, how aggressive they are, the
  local heating from cutting, and the clamping all change how the part moves as those stresses
  are released.
- **Heat-treated stock** adds a whole further dimension: cutting it is cutting a moving target.
- **Then add the stock flexing under the cutting force**, and the problem becomes very hard to
  manage by experience alone. That combination is what this file asks CAM to model.

## Summary

1. **The stock is part of the machine structure, and it changes.** During a cycle the force from
   the tool passes through a chain: tool, holder, spindle, machine axes, fixture, and the stock
   itself. Every other link in that chain is fixed for the cycle. The stock is the only one that
   **gets weaker as the program runs**, because each cut removes material that was stiffening the
   rest. A wall that was solid during roughing can be a thin, springy plate during finishing.
2. **Three things go wrong as the stock weakens** (Jason's framing). It **deflects** under the
   cutting force (the tool cuts less than programmed and the wall comes out thick or tapered); it
   **vibrates** (its natural frequency falls into the range of the tooth-passing frequency and it
   chatters); and it **distorts** when the clamps are released, because residual stress in the
   raw material and clamp force are no longer balanced by the material that was cut away.
3. **The order of cuts decides most of this.** The same final shape can be reached by many
   sequences. Research and shop practice agree on the principle: **remove material so that what
   remains stays stiff for as long as possible, and machine the flexible features last, while
   their supporting material is still there.** In the worked example below, finishing a 2 mm
   wall in alternating top-down bands instead of side by side lowers the predicted deflection by
   about eight times for the same force.
4. **Real-time negotiation across the whole system.** Jason's idea: as the cutter moves in three
   dimensions it pushes and pulls the part in different directions. In every direction the
   workholding, the machine, the tool and the **current stock** each have a limit. The toolpath
   adjusts the load (feed, width or depth of cut) so the force along its instantaneous direction
   stays below the smallest of those limits. This file supplies the stock's limit, and it changes
   along the toolpath as material disappears.
5. **Plan the sequence, not only the feed.** Lowering feed is the last resort. A planner can
   first choose a better order (alternate sides, rough everything before finishing anything,
   finish thin walls in bands from the top down, leave a non-uniform allowance), choose where to
   leave temporary support (tabs, ribs, a skin over a floor), and choose when to unclamp and
   re-clamp at lower force so residual stress is released before the finishing passes.
6. **Start simple, grow into FEA.** A useful first version needs only beam and plate formulas on
   the features recognised in the in-process stock (walls, floors, ribs, bosses). Research shows
   how to replace them with fast, updated finite-element or reduced models later, which is where
   this topic meets the FEA request #1620.
7. **Nothing checked publicly does this end to end.** CAM programs keep an in-process stock
   model for collision checking and rest machining, and some optimise feed for tool load and tool
   deflection. In the public pages reviewed, none plans the cut order from the changing stiffness
   of the stock together with workholding and machine limits. Research prototypes do parts of
   it (see Programs).

## Fundamentals

### The compliance chain

When the tool pushes on the stock with force `F`, the tool and the stock move apart by
`δ = F / k_total`. The links act like springs in series:

`1 / k_total = 1/k_tool + 1/k_holder + 1/k_spindle + 1/k_machine + 1/k_fixture + 1/k_stock`

The **weakest link dominates**. That link moves the surface away from where it was programmed
(static error) and is usually the one that chatters (dynamic error).

Illustrative numbers (computed for this file; machine and fixture values are placeholders until
the machine library has real entries):

| Link | Example | Stiffness |
|---|---|---|
| Tool | 10 mm carbide end mill, 30 mm stick-out (`k = 3EI/L³`, `E ≈ 600 GPa`, effective core diameter 8 mm) | ≈ 13.4 N/µm |
| Machine at tool point | placeholder, from the machine library | 50 N/µm (unverified) |
| Fixture | placeholder, from the workholding library | 20 N/µm (unverified) |
| Stock: wall 30 mm high, 4 mm thick, 100 mm long, 6061 | cantilever `k = 3EI/L³` | ≈ 4.1 N/µm |
| Stock: same wall at 2 mm thick | | ≈ 0.51 N/µm |

With the 2 mm wall, `k_total ≈ 0.48 N/µm`, and the stock contributes about 93 % of the total
compliance. The machine, which a machinist might expect to be the weak point, hardly matters
here. **This is why the stock has to be in the model**: on thin features it is the limit, and it
only becomes the limit late in the program.

### Deflection of the remaining stock

First estimates (the same formulas as the workholding file, so the two agree):

- **Wall as a cantilever** of height `L`, thickness `t`, loaded length `b`, force `F` at height
  `c` above its root: `δ = F c³ / (3 E I)` with `I = b t³ / 12`. Thickness enters **cubed** and
  height enters **cubed**. Halving the thickness makes the wall eight times softer.
- **Stepped wall** (thin finished band of height `h` on top of thicker unfinished stock, force at
  the free edge): `δ = F/(3E) · [ h³/I_thin + (L³ − h³)/I_thick ]`. This is the formula that
  makes band-by-band finishing work: while most of the wall's height is still thick, the
  deflection stays small.
- **Wall held at both ends** (between two corners or ribs) behaves as a plate and is stiffer than
  the cantilever. The cantilever is a safe upper bound (opinion: start with the bound, refine it
  with a plate formula or FEA later).
- **Floor (web) as a plate**: a thin floor loaded by axial force deflects like a plate on its
  edges. Thin floors are why axial force matters as much as radial force.

Research models go further: the cutting force itself depends on the deflection, because a wall
that bends away takes a thinner chip. Huang, Ratchev and colleagues coupled a force model with a
finite-element model of the part and updated the force at every step for exactly this reason
([Huang et al. 2004](https://researchportal.beds.ac.uk/en/publications/a-flexible-force-model-for-end-milling-of-low-rigidity-parts-4/)).
Polzer et al. predicted the deflection of a thin wall by FEM with progressive material removal
and wrote it back into the NC program as a curved compensating path; the flatness error fell from
0.034 mm to 0.019 mm ([Polzer, Dufkova & Pokorny 2015](https://yadda.icm.edu.pl/baztech/element/bwmeta1.element.baztech-50e1d792-c2d2-4cbc-bc95-096426153d2a/c/4_POLZER.pdf)).

### Vibration of the remaining stock

The lowest natural frequency of a cantilever wall is
`f₁ = (1.875² / 2π) · √(E I / (ρ A L⁴))`, and for a uniform wall it scales **linearly with
thickness**. The 30 mm aluminium wall is about 3.6 kHz at 4 mm and 1.8 kHz at 2 mm.

The tooth-passing frequency from the cutting physics file is `f_tp = n · z / 60`. A risk exists
when `f_tp` or a low harmonic of it lands near the stock's current `f₁`. Because `f₁` falls as
the wall is cut, a spindle speed that is safe at the start of a finishing pass can become unsafe
by its end. Research calls this **time-varying** or **position-dependent** workpiece dynamics. A
2023 review of thin-wall chatter organises the field around it: identifying the part's modes
(experimentally, analytically or by finite elements), updating them efficiently as material is
removed (substructuring, mode and freedom reduction), and suppressing chatter by supports,
dampers, stock shape and process sequence
([Sun et al. 2023, Machines 11(3) 359](https://www2.mdpi.com/2075-1702/11/3/359)).
Kiran and Bediz (2025) predict the evolving frequency response at the tool contact with a
spectral element method, which they report is much cheaper than re-meshing FEA each time the
part changes ([Thin-Walled Structures 217, 113851](https://research.sabanciuniv.edu/id/eprint/52232/)).

### Residual stress and release distortion

Raw stock is not stress free. Rolled and quenched plate carries stress that is in balance only
while all of it is there.

- **When material is cut away, the remaining stress rebalances and the part moves.** Quench
  stress is the main cause, because it extends through a large volume of material
  ([Lambda Research, Diffraction Notes 24, 1999](https://www.lambdatechs.com/wp-content/uploads/dn-24.pdf)).
- **Stress-relieved tempers help a lot.** Prime and Hill measured 75 mm thick 7050 aluminium in
  the T74 temper and in T7451 (stress relieved by stretching 1.5 % to 3 %): the stretched plate's
  residual stress was about ten times lower
  ([LANL slitting method, Aluminum plate](https://www2.lanl.gov/residual/alum.shtml); Scripta
  Materialia 46, 2002). So the **material record** should say whether stock is stress relieved,
  and CAM should treat non-relieved stock as higher risk.
- **Where the part sits in the plate matters.** Seger et al. measured 860 points of residual
  stress in a 90.5 mm thick 7050-T7451 plate with neutron diffraction and found that residual
  stress alone could distort a slender part beyond aerospace limits, and that parts taken from
  different depths in the plate could distort in **opposite** directions
  ([ORNL page; J. Manuf. Sci. Eng. 146(10), 2024](https://www.ornl.gov/publication/effects-aluminum-plate-initial-residual-stress-machined-part-distortion)).
  Opinion: the stock record should carry where the part sits in the plate (offset from the
  plate's mid-plane), and CAM should offer to centre the part in the thickness.
- **Sequence matters when the stress is symmetric.** In Lambda Research's finite-element example
  of a forged disk with a symmetric stress field, removing all of one side first was the worst
  case; alternating 1 mm passes between top and bottom cut the displacement by roughly an order
  of magnitude, and smaller depths reduced it further. With an asymmetric stress field no
  sequence reliably minimised distortion
  ([Diffraction Notes 24](https://www.lambdatechs.com/wp-content/uploads/dn-24.pdf)).
- **Machining also adds stress**, near the surface, and thin features feel it more. That is a
  smaller, separate effect (opinion: out of scope for a first version, noted in Open questions).

Shop practice follows the same physics: rough, release and re-clamp, then finish; or rough one
side, flip, rough the other, flip again, and finish; stress relieve the raw stock if it moves too
much (paraphrased from a public machinists' forum thread,
[eMastercam 2018](https://www.emastercam.com/forums/topic/97551-machining-stress/)).

### Clamp force on a weakening part

The clamp is also a load. A clamp force that a solid block takes without noticing can bend a
part that has been hollowed out, and the part springs back when released. So the workholding
file's torque window (`T_min` to hold, `T_max` before distortion) is **not constant over the
cycle**: `T_max` falls as the stock thins. This is the reason for a common shop habit that CAM
can recommend explicitly: **after roughing, release and re-clamp at a lower torque before
finishing** (agreed with the workholding file). The workholding file supplies clamp positions and
force; this file treats them as loads and boundary conditions on the current stock.

### Worked example: finishing a thin wall in a better order

A 6061 aluminium wall, finished size 30 mm high, 2 mm thick, 100 mm long, between two pockets.
Roughing leaves 1 mm per side, so the wall is 4 mm thick before finishing. The finishing radial
force is taken as 30 N (an assumed value; in SolidCraft it would come from the cutting physics
file). Tolerance on thickness ±0.05 mm. Cantilever estimate, `E ≈ 68.9 GPa`, force at the top
edge, which is the worst case.

**Order A: finish one side at full depth, then the other.**

- Side 1, wall still 3 mm (1 mm left on the other side): `δ ≈ 0.017 mm`.
- Side 2, wall now 2 mm everywhere: `δ ≈ 0.059 mm`. **Out of tolerance** at the top edge, and
  the wall comes out thicker at the top than at the bottom (taper).

**Order B: finish both sides in 5 mm bands from the top down, alternating sides in each band.**

- While the top band is cut, the 25 mm below it is still 4 mm thick. Treating the whole wall as
  4 mm thick under the force at the top gives `δ ≈ 0.007 mm`, and the thin finished band adds
  very little because it is short (stepped-beam formula above).
- By the time the last band is reached, the force acts only 5 mm above the root, and the
  deflection there is a fraction of a micrometre.
- **Worst case about 0.007 mm instead of 0.059 mm**, roughly eight times less, with the same tool,
  force and feed.

The same calculation also tells the planner what it may trade: with order B the stock is no
longer the limiting link, so the feed can rise until the tool, machine or workholding limit takes
over. The cost is more passes and more tool changes of direction (cycle time), which is what the
optimiser weighs. Grossi et al. reported the same trade on a thin aluminium blade: their
stiffness-aware toolpath brought the error from up to 0.17 mm to within ±0.02 mm, at 52 s
cutting time instead of 46 s
([Grossi et al. 2019, MM Science Journal](https://flore.unifi.it/retrieve/e398c37f-0070-179a-e053-3705fe0a4cff/mmscience_2019-11_adaptive-toolpath-for-3-axis-milling-of-thin-walled-parts.pdf)).

## How a sequence planner can work

### Known strategies (fact, from research and shop practice)

- **Search the order backwards.** Wang, Ibaraki and Matsubara split the volume to be removed into
  blocks, start from the finished part, and add blocks back in an order that keeps deformation
  under a threshold at every step (checked by FEM); reversing that order gives the cutting
  sequence. Block size trades deflection (smaller axial depth) against efficiency (larger radial
  depth) ([abstract, Hiroshima University](https://mecdes.hiroshima-u.ac.jp/~ibaraki.h0523/pubs/abstract/pe17_wang.html)).
  Grossi et al. also build their toolpath in reverse, start at the stiffest point, and choose the
  side so the most flexible points are cut while the most material remains
  ([Grossi et al. 2019](https://flore.unifi.it/retrieve/e398c37f-0070-179a-e053-3705fe0a4cff/mmscience_2019-11_adaptive-toolpath-for-3-axis-milling-of-thin-walled-parts.pdf)).
- **Design the stock left for finishing.** Kienast et al. combine material removal simulation,
  FEA and stability simulation to design the shape of the stock left for semi-finishing and
  finishing of thin-walled parts, then optimise spindle speed along the toolpath
  ([Procedia CIRP 126, 2024](https://research.sabanciuniv.edu/id/eprint/47611/)). Ren et al.
  combine rigidity optimisation, a non-uniform allowance and an optimised removal sequence
  ([Advanced Materials Research 97–101, 2010](https://pure.nwpu.edu.cn/en/publications/research-on-chatter-suppression-in-machining-thin-walled-componen)).
- **Cut both sides together** where a machine allows it: Guo et al. balance forces from two
  tools on opposite sides of a thin blade and report less deformation and about twice the
  efficiency ([J. Intelligent Manufacturing 36, 2025](https://hit.globalimpact.cn/en/publications/dual-sided-co-machining-scheme-optimization-for-thin-wall-compone/)).
- **Alternate faces of a thin wall** with fast, shallow cuts
  ([Modern Machine Shop video summary](https://www.mmsonline.com/videos/video-basics-of-thin-wall-machining)).
  Some machinists prefer full-depth radial finishing passes on thin walls instead, because step
  downs leave visible steps, and correct taper with a spring pass
  ([eMastercam forum](https://www.emastercam.com/forums/topic/32575-thin-wall-ti-finishing-strategy/)).
  Opinion: both are right for different walls; the stock model is what tells them apart (a
  stiff-enough wall can take full depth, a flexible one needs bands).
- **Design rules of thumb**: keep wall height-to-thickness at or below about 10:1; walls supported
  at both ends can be thinner, up to about 25:1 for short supported ribs; about 0.8 mm is a
  general minimum wall ([Xometry](https://www.xometry.com/blog/cnc-machining-thin-walls)).
  These are useful as a zero-cost first warning before any calculation.
- **Balance distortion by alternating sides, or condition the stock first.** An EU case study on
  ribbed aluminium aircraft parts describes alternate-side machining as the traditional fix
  (long machine and setup times, hard to clamp a part with fins), and replaced it with an
  analytical residual-stress model, a stock-conditioning fixture and an even hydraulic clamp,
  reporting distortion down from 2 mm to 0.3 mm and machine time from four days to two
  ([I4MS case study 2.3](https://i4ms.eu/?p=3948)).

### A planner for SolidCraft (opinion)

The planner works on the **operation list** that CAM already has, plus the in-process stock after
each step.

1. **Recognise features in the in-process stock** after each operation: walls (height,
   thickness, length, supported edges), floors (span, thickness), ribs, bosses, and the material
   that connects them to the clamped region. A simple version can use the part's own features
   plus the planned allowance.
2. **Estimate each feature's stiffness, mode and deflection** with the tier the user can afford
   (see "Levels of model" below).
3. **Check every move against the stock's limit** in the move's force direction (next section),
   and against clamp loads, and estimate release distortion at each unclamp.
4. **Repair the plan in this order** (cheapest first, Jason's "plan the correct sequence of
   cuts"):
   1. reorder operations: rough everything before finishing anything; machine the most flexible
      features last but before their support is removed; balance material removal on opposite
      sides;
   2. split operations: finish thin walls in bands from the top down, alternating sides; leave a
      larger allowance where the stock is flexible (non-uniform allowance);
   3. add support: leave tabs, ribs or a floor skin to remove at the end; suggest a support or
      filler from the workholding library;
   4. add a release step: unclamp and re-clamp at the lower torque the workholding file
      recommends, after roughing and before finishing; probe after release if the inspection
      plan allows;
   5. only then lower the load (feed, width or depth of cut) on the moves that are still over the
      limit.
5. **Explain the result**: "Wall W3 finishes at 2.0 mm; finishing it in one pass deflects 0.06 mm
   (limit 0.025 mm). Planned in 6 alternating bands: 0.007 mm, cycle +40 s."

The planner can search with simple rules first and a search method later (the papers above use
FEM-in-the-loop searches, a neural-network surrogate and multi-objective optimisation). The
objective is the same as in the workholding file: **pass every check, then shortest cycle time,
then least predicted distortion, then least setup effort**.

### Real-time negotiation of force direction across the whole system (Jason's idea)

As the cutter moves, the direction of the force on the stock turns with the tool's engagement,
the feed direction and, on 5-axis machines, the tool axis. In every direction each part of the
system has its own limit:

| Link | Limit in direction `u` | From |
|---|---|---|
| Tool | breakage, deflection, chip load | cutting physics, tool library |
| Machine | axis thrust `F_axis_max` per axis, stiffness `k_X, k_Y, k_Z`, spindle power and torque `P(n), T(n)` | machine library |
| Workholding | capacity `F_allow(u)` and moment limits after safety factor | workholding |
| **Stock** | `F_allow_stock(u) = k_stock(u) · δ_allow(u)` and the chatter check | this file |

`δ_allow(u)` is the part of the tolerance budget given to stock deflection (opinion: start with
half of the tolerance band for finishing, shared with tool deflection; much larger for roughing,
where only gouging and chatter matter). For each move the feed (or width or depth of cut) is
scaled so that

`|F(u)| ≤ min( F_tool(u), F_machine(u), F_workholding(u), F_stock(u) )`

and the move's tooth-passing frequency stays away from the stock's current `f₁`. Because
`k_stock` and `f₁` fall along the toolpath as material leaves, the stock's limit is a **map along
the toolpath**, computed in advance from the planned stock states, not a single number. During
cutting, a control with load-based adaptive feed could follow the same limit live (opinion: the
plan should be correct without it; live adaptation is a later step tied to the machine library).

The same chain gives the expected **surface error** of a finishing move:
`δ = |F| · (1/k_tool + 1/k_machine + 1/k_fixture + 1/k_stock)` along the surface normal. The
inspection file can then compare the probed result with this prediction.

### Levels of model (opinion)

| Level | Model | Cost | Use |
|---|---|---|---|
| 0 | Rules of thumb (height-to-thickness, minimum wall, stress-relieved stock or not) | none | warnings while programming |
| 1 | Beam and plate formulas on recognised features of the in-process stock | milliseconds | first planner and feed limit; upper bounds |
| 2 | Reduced finite-element model (shell or spectral elements) updated per operation | seconds to minutes | thin-walled and aerospace parts; needs #1620 |
| 3 | Full FEA with residual stress field and release | minutes to hours | distortion studies, high-value parts |

Grossi et al. used a shell model updated at each step and needed about 17 minutes on a laptop for
a full blade optimisation; Kiran and Bediz show a cheaper spectral method. Level 1 is enough to
catch the common cases (thin walls finished in the wrong order, thin floors cut with large axial
force, non-relieved plate) and costs almost nothing.

### The in-process stock model

Every CAM program keeps an **in-process stock** (also called IPW, in-process workpiece) to avoid
air cutting, plan rest machining and check collisions. Common representations are voxels, dexels
(depth columns) or meshes. Siemens NX can turn its in-process stock into a solid (Parasolid) model
through an add-on, with analytic faces where they can be recognised and mesh elsewhere, so it can
be used for fixtures and inspection
([Machinery, Solid IPW](https://www.machinery.co.uk/content/news/solid-ipw-for-siemens-nx-based-on-machineworks-new-parasolid-bridge)).

For this topic the stock model must also give **thicknesses and connectivity**: which material
holds a wall to the clamped region, and how thick it is. Opinion: a dexel or voxel stock is enough
for Level 0–1 (thickness by ray casting), and Level 2 needs a mesh for the FE solver. SolidCraft's
kernel work is paused (CONTRIBUTING.md), so this should not depend on new kernel features.

## Accuracy

- **Beam estimates are upper bounds.** Real walls are supported at their ends and by fillets and
  floors, so the cantilever over-predicts deflection. That is safe for warnings, but it can make
  a plan more conservative than needed (opinion: show "estimate (beam)" next to the number).
- **Force inputs are uncertain.** The deflection is only as good as the cutting force; the
  cutting physics file labels assumed forces (estimated radial and axial forces), and that label
  should pass through to every stock result.
- **Residual stress prediction is hard.** In a careful 2019 study on quenched 7050 coupons, FE
  predictions of distortion were off by 18 % to 42 % on machined faces and by about 100 % at some
  points after release ([Bilkhu et al., Procedia CIRP 82](https://eprints.whiterose.ac.uk/151965/1/17th-CIRP-Submission-02.pdf)).
  Seger et al. found that a common measurement method (slitting) may not capture the stress well
  enough to predict distortion. So release distortion should be presented as **risk and
  direction**, not a precise number, unless the user supplies measured stress data.
- **Dynamics change with the mode, not only the frequency.** A beam formula gives `f₁` but not
  damping or mode shape; chatter prediction needs both. Opinion: Level 1 only warns ("tooth
  harmonic near the wall's estimated mode"); stability lobes for the part belong to Level 2.
- **Compensation can make things worse** if the model is wrong. Polzer et al.'s compensated path
  improved flatness by about 45 %, not 100 %, and Grossi et al. had a 20 mm over-cut on an
  unmodelled entry ramp. Opinion: offer compensation only at Level 2 and after probing confirms
  the prediction (inspection file).

## Programs

What public pages say about existing tools (checked 2026-10-26):

- **Vericut Force** (CGTech) optimises feed from tool load and, since version 9.3, computes
  deflection for the whole rotating tool assembly; the public article mentions stock material
  data but not stock deflection or thin walls
  ([MoldMaking Technology](https://www.moldmakingtechnology.com/products/simulation-software-updates-for-smarter-machining-mold-manufacturing)).
  So: the tool side of the chain, not the stock side (from the public page).
- **Siemens NX CAM**: multi-stripe blade finishing machines blades in sections to reduce the
  forces applied to them; a minimum cut depth setting controls flute engagement
  ([Siemens blog, January 2019](https://blogs.sw.siemens.com/nx-manufacturing/whats-new-in-nx-for-manufacturing-january-2019/)).
  This is a geometric strategy, not a stiffness calculation.
- **In-process stock as a solid**: Solid IPW for NX (Janus Engineering with MachineWorks) exports
  the stock between operations for fixture design and inspection (link above).
- **Research prototypes** (Grossi et al., Wang et al., Kienast et al., Guo et al., Polzer et al.)
  do the stiffness-aware sequencing and compensation this file describes, each for one part type,
  usually with commercial FEA in the loop.
- **Shop-floor practice** fills the gap by hand: rough, release, re-clamp, finish; flip and
  alternate; spring passes; stress-relieved stock.

In the pages reviewed, no program combined in-process stock stiffness, workholding capacity and
machine stiffness to choose the order of cuts. (Absence from public pages is not proof of absence.)

## Complaints and requests

Paraphrased from public forum threads and case studies cited above:

- Parts that measure good on the machine and move after unclamping.
- Thin walls that come out tapered or chatter near the top.
- Alternate-side machining works but costs machine and setup time, and clamping a half-machined
  part with thin fins is hard (I4MS case study).
- Stainless and titanium parts that keep moving even with careful sequences.
- No way to see, before running the program, which operation will make the part weak.

## What users need

1. **A warning in the setup sheet** when a feature becomes thin before it is finished, or when a
   finishing pass on a thin feature exceeds the deflection budget, with the operation named.
2. **A suggested better order**, shown as a change to the operation list the user can accept or
   reject, with the cycle-time cost.
3. **Release steps** in the operation list (unclamp, re-clamp at a given torque), produced
   together with the workholding recommendation.
4. **Stock records that carry stress state**: alloy and temper, how the stock was made
   (saw-cut cold-formed bar, forging, casting, hot-rolled, plate), heat treatment, stress
   relieved or not, plate thickness, and where the part sits in it, so the planner knows the
   risk. A first ranking from the author's experience (above): cold-formed and purpose-designed
   forgings low, castings and heat-treated stock high, hot-rolled stock high (it curls).
5. **One risk slider** for the whole system (overview and cutting physics files). For the stock it
   moves the share of tolerance given to deflection and the margin around the stock's natural
   frequency; it never allows the predicted deflection to exceed the tolerance band.
6. **Feedback from the machine**: probe results after release and after finishing go back into
   the stock model to correct it (inspection file).

### Interface with the other files

Agreed between the files (names shared with the workholding and machine library files):

| Direction | Quantity | Unit | Notes |
|---|---|---|---|
| Cutting → stock | `Fx`, `Fy`, `Fz` per move, peak and mean | N | from the cutting physics file |
| Cutting → stock | contact point and engaged surface normal | mm | where the force acts on the stock |
| Cutting → stock | `f_tp = n · z / 60` | Hz | for the stock mode check |
| Workholding → stock | clamp force `F_N`, contact and support locations and areas | N, mm, mm² | boundary conditions and clamp loads |
| Workholding → stock | capacity envelope `F_allow(u)` and moment limits | N, N·m | for the combined limit |
| Workholding → stock | clamp-induced distortion for a given clamp force | mm | to trade clamp force against deformation |
| Machine → stock | `k_X`, `k_Y`, `k_Z` at the tool point (optionally a map over the work envelope) | N/µm | compliance chain |
| Machine → stock | dominant tool-tip modes `f_n`, `ζ`, `k` per direction | Hz, –, N/µm | for combined dynamics |
| Machine → stock | `F_axis_max` per axis, `P(n)`, `T(n)`, max feed and acceleration | N, W, N·m, mm/min | for the combined limit |
| Stock → workholding | in-process stock per step (for vacuum sealed area, grip region, contact patches) | mesh / dexel | |
| Stock → workholding | stiffness at the tool contact and at the clamp contacts | N/µm | |
| Stock → feed optimiser | `k_stock(u)` at the contact point per direction, per move | N/µm | a map along the toolpath |
| Stock → feed optimiser | `F_allow_stock(u)` from the deflection budget | N | the stock limit in the min |
| Stock → feed optimiser | stock deflection under the predicted force | mm | |
| Stock → feed optimiser | lowest stock mode `f₁` per move | Hz | |
| Stock → planner and setup sheet | predicted release distortion after each unclamp (risk and direction) | mm | |
| Stock → inspection | predicted deflection and release distortion at probe points | mm | for comparison with probed values |

## Glossary

- **In-process stock (IPW, in-process workpiece):** the shape of the material at a given point in
  the program, after the cuts so far.
- **Stiffness (`k`):** force needed per unit deflection, here in N/µm. **Compliance** is `1/k`.
- **Compliance chain:** tool, holder, spindle, machine, fixture and stock acting as springs in
  series between the cutting edge and the machine table.
- **Deflection budget (`δ_allow`):** the part of the tolerance band allowed for elastic
  deflection during a cut.
- **Natural frequency (`f₁`):** the lowest frequency at which the remaining stock rings.
- **Tooth-passing frequency (`f_tp`):** how often a cutting edge enters the stock, `n · z / 60`.
- **Residual stress:** stress locked into the material by rolling, forging or quenching, in
  balance only while all the material is there.
- **Release distortion (spring-back):** how the part moves when the clamps are released.
- **Stress relieved (for example aluminium T651, T7451):** plate stretched after quenching to
  lower residual stress.
- **Non-uniform allowance:** leaving more stock for finishing where the part is flexible.
- **Waterline bands:** finishing a wall in horizontal steps from the top down.
- **Tab / sacrificial rib / skin:** material left on purpose to support the part, removed last.

## Open questions

1. Should the stock model live in the CAM plugin or in a shared engine service that the FEA work
   (#1620) also uses?
2. Which level should the first version ship: Level 0 warnings only, or Level 1 beam and plate
   estimates on recognised features?
3. Where do residual stress profiles come from? Generic shapes per alloy and temper in the shared
   material library, user-supplied measurements, or nothing until Level 3?
4. How should the planner present a re-ordered operation list so the programmer stays in
   control (accept per change, or a "suggested plan" beside theirs)?
5. Should near-surface machining-induced stress (from the cutting edge) be modelled for thin
   features, or left out?
6. Is a dexel stock enough for thickness and connectivity queries, or does it need a mesh from
   the start?
7. Lathe parts (thin-walled tubes in chucks) have the same physics with radial clamping; include
   them in the first version or later?

## Sources

All accessed 2026-10-26.

### Papers and laboratory pages

- Huang W., Ratchev S., Liu S., Becker A. A., "A flexible force model for end milling of low
  rigidity parts", J. Materials Processing Technology 153–154 (2004) 134–138.
  [Abstract](https://researchportal.beds.ac.uk/en/publications/a-flexible-force-model-for-end-milling-of-low-rigidity-parts-4/)
- Polzer A., Dufkova K., Pokorny P., "On the modern CNC milling with a compensation of cutting
  tools and thin-walled workpiece deflections", J. Machine Engineering 15(3) (2015).
  [PDF](https://yadda.icm.edu.pl/baztech/element/bwmeta1.element.baztech-50e1d792-c2d2-4cbc-bc95-096426153d2a/c/4_POLZER.pdf)
- Grossi N., Scippa A., Croppi L., Morelli L., Campatelli G., "Adaptive toolpath for 3-axis
  milling of thin walled parts", MM Science Journal, HSM2019 special issue (2019).
  [PDF](https://flore.unifi.it/retrieve/e398c37f-0070-179a-e053-3705fe0a4cff/mmscience_2019-11_adaptive-toolpath-for-3-axis-milling-of-thin-walled-parts.pdf)
- Wang J., Ibaraki S., Matsubara A., "A cutting sequence optimization algorithm to reduce the
  workpiece deformation in thin-wall machining" (year and venue not on the abstract page).
  [Abstract](https://mecdes.hiroshima-u.ac.jp/~ibaraki.h0523/pubs/abstract/pe17_wang.html)
- Kienast P., Tunç L. T., Koca R., Olgu O., Ganser P., Bergs T., "Improving dynamic process
  stability in finishing of thin-walled workpieces by optimal selection of stock shape",
  Procedia CIRP 126 (2024) 372–377. [Record](https://research.sabanciuniv.edu/id/eprint/47611/)
- Kiran K., Bediz B., "Predicting workpiece dynamics in milling of thin-walled structures using
  three-dimensional spectral element method", Thin-Walled Structures 217 (2025) 113851.
  [Record](https://research.sabanciuniv.edu/id/eprint/52232/)
- Sun Y., Zheng M., Jiang S., Zhan D., Wang R., "A state-of-the-art review on chatter stability
  in machining thin-walled parts", Machines 11(3) (2023) 359.
  [Open access](https://www2.mdpi.com/2075-1702/11/3/359)
- Ren J., Yang B., Liang Y., Tian W., Yao C., "Research on chatter suppression in machining
  thin-walled components based on rigidity optimization", Advanced Materials Research 97–101
  (2010) 1947–1951. [Abstract](https://pure.nwpu.edu.cn/en/publications/research-on-chatter-suppression-in-machining-thin-walled-componen)
- Guo X., Wang R., Zhong S., Xu X., "Dual-sided co-machining scheme optimization for thin-wall
  components", J. Intelligent Manufacturing 36(8) (2025) 5737–5756.
  [Abstract](https://hit.globalimpact.cn/en/publications/dual-sided-co-machining-scheme-optimization-for-thin-wall-compone/)
- Seger M., Mathews R., Marais D., Venter A., Halley J., Wang J., Malik A., "Effects of aluminum
  plate initial residual stress on machined-part distortion", J. Manufacturing Science and
  Engineering 146(10) (2024) 101006. [ORNL record](https://www.ornl.gov/publication/effects-aluminum-plate-initial-residual-stress-machined-part-distortion)
- Bilkhu R., Ayvar-Soberanis S., Pinna C., McLeay T., "Machining distortion in asymmetrical
  residual stress profiles", Procedia CIRP 82 (2019) 395–399.
  [PDF](https://eprints.whiterose.ac.uk/151965/1/17th-CIRP-Submission-02.pdf)
- Prime M. B., Hill M. R., slitting measurements of 7050-T74 and T7451 plate, Scripta Materialia
  46 (2002) 77–82, summarised at [LANL](https://www2.lanl.gov/residual/alum.shtml).
- Lambda Research, "Minimizing distortion in machining", Diffraction Notes 24 (1999).
  [PDF](https://www.lambdatechs.com/wp-content/uploads/dn-24.pdf)

### Case studies, vendor and trade pages

- I4MS, "Case Study 2.3: Distortions in aeronautical structural parts". [Page](https://i4ms.eu/?p=3948)
- MoldMaking Technology, Vericut 9.3 update.
  [Page](https://www.moldmakingtechnology.com/products/simulation-software-updates-for-smarter-machining-mold-manufacturing)
- Siemens, "What's new in NX for Manufacturing, January 2019".
  [Blog](https://blogs.sw.siemens.com/nx-manufacturing/whats-new-in-nx-for-manufacturing-january-2019/)
- Machinery, "Solid IPW for Siemens NX based on MachineWorks' new Parasolid Bridge".
  [Article](https://www.machinery.co.uk/content/news/solid-ipw-for-siemens-nx-based-on-machineworks-new-parasolid-bridge)
- Xometry, "CNC machining thin walls". [Blog](https://www.xometry.com/blog/cnc-machining-thin-walls)
- Modern Machine Shop, "Basics of thin wall machining" (video page).
  [Page](https://www.mmsonline.com/videos/video-basics-of-thin-wall-machining)

### Forums (paraphrased)

- eMastercam, "Machining stress" (2018). [Thread](https://www.emastercam.com/forums/topic/97551-machining-stress/)
- eMastercam, "Thin wall Ti finishing strategy". [Thread](https://www.emastercam.com/forums/topic/32575-thin-wall-ti-finishing-strategy/)
