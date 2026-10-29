# CAM research: a design-for-manufacturing (DFM) check and a design-to-CAM cost optimiser

## About this research

- **Author:** Jason ([@nsnlookupjn](https://github.com/nsnlookupjn)). The idea of building a
  comprehensive DFM check into SolidCraft's CAM, like the checks that instant-quote services
  (Xometry, RapidDirect, Protolabs and others) run on uploaded parts, and the idea of an
  optimiser that runs from design through CAM to cut stock cost and machining time, are Jason's.
  Source research and writing done with an AI assistant (Claude) in October 2026.
- **How it was produced:** from public pages only: service help articles and design guides,
  vendor product pages, papers and Wikipedia. No quoting service, CAD or CAM program was driven
  or scripted, no part was uploaded anywhere, no GPL/LGPL/AGPL source was read, and no text is
  copied beyond short quotes. Every service is described from its own public pages; what they
  run behind the page is not known and is not guessed at.
- **Units:** millimetres, newtons, N/µm for stiffness, minutes. Inch values from US pages are
  converted and marked.
- **Marking:** "(unverified)" marks a claim not confirmed against a source. **Fact**,
  **opinion** and **Jason's idea** are labelled where they mix. Everything in "What users need"
  is opinion: a proposal for discussion, not a decision. The design rules quoted below are
  public rules of thumb from service providers, not standards; SolidCraft would ship them as
  editable starting values, with each number's source.
- **Licence:** MIT OR Apache-2.0, as the project asks.
- **Sibling files** (same fork, same author):
  [CAM Physics System](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-vgqo3g/research/cam-research-nsnlookupjn-physics-system.md),
  [tool holding](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-4aoliy/research/cam-research-nsnlookupjn-tool-holding.md),
  [in-process inspection](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-y2w06n/research/cam-research-nsnlookupjn-in-process-inspection.md)
  (per-process tolerance profiles),
  [physical properties](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-uxzal2/research/design-research-nsnlookupjn-physical-properties.md),
  [stock marketplace](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-um9nlo/research/cam-research-nsnlookupjn-stock-marketplace.md),
  [additive process physics](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-j72y6h/research/3d-printing-research-nsnlookupjn-process-physics.md)
  (DfAM, #1625).

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

> **[TODO Jason: personal experience]** One or two parts where an instant quote flagged
> something that was not a real problem in your shop, or missed something that was (for example
> a deep corner that needed a long-reach tool, or a thread nobody could tap to depth). Also, if
> you can, the name of the website you remember that suggested design changes to cut stock and
> machining cost (see "Which website was it?"). Label this as personal experience.

## Summary

**The idea (Jason's).** Online manufacturers check every uploaded part for manufacturability
before they quote it: thin walls, sharp inside corners, deep pockets, holes and threads that are
too deep, features no tool can reach, tolerances tighter than the process holds. SolidCraft
should run the same kind of check inside the design and the CAM, and go further: an optimiser
that proposes design changes which make the stock cheaper and the machining faster, with the
cost and time saved shown for each one.

**What the services flag (fact, from their public pages).** The lists are short and similar
across Protolabs, Xometry, Hubs (Protolabs Network) and RapidDirect: thin walls, sharp internal
corners, material a 3-axis tool cannot reach (undercuts, "material left behind"), small or deep
holes, narrow areas, voids and floating bodies, oversize parts. Moulding adds wall thickness
variation, zero draft, texture draft and fill. Protolabs separates **required** changes (you
cannot order) from **recommended** ones (accept the limitation), and says false positives exist;
Xometry says the same and offers a manual review. The numbers behind the flags are public rules
of thumb (for example: pocket depth up to 4× its width, inside corner radius at least a third of
the depth, metal walls at least 0.8 mm, holes up to 4× diameter as standard and 10× typical,
threads 1.5× to 3× diameter).

**Where SolidCraft can do better (opinion).** A quote service checks against a generic machine
with a generic tool set, after the design is finished. SolidCraft can check, while the part is
being designed:

1. **against the real resources:** the tool, holder, machine and workholding libraries already
   proposed in the sibling files, so "can a tool reach this corner?" is answered with the shop's
   actual end mill in its actual holder at the stick-out it needs, and that stick-out's
   deflection under the cutting force from the physics system;
2. **with the design intent:** a SolidCraft hole already knows it is an M6 tapped hole with a
   thread depth, so no feature recognition guesses; recognition is only needed for imported or
   direct-edited geometry;
3. **with a reason and a fix for every flag:** which rule, which number, which resource bound
   it, what change clears it, and what that change saves;
4. **with tolerances:** each toleranced feature against the process tolerance profile of the
   operation that will make it (from the inspection file), so a tight bore is flagged as "adds a
   ream or bore operation" rather than ignored;
5. **as a command** (`dfm.check`) like everything else in SolidCraft, so agents and scripts get
   the same result as the panel.

**The optimiser (Jason's idea; design is opinion).** A cost model (stock + machine time + setups
+ tooling + inspection) evaluated by the same CAM and physics pipeline, and a set of small,
named design "moves" (raise a corner radius, snap a thickness to a stock size, change a hole to a
standard drill, shorten a thread, turn a feature to remove a setup) tried in a copy of the design
with SolidCraft's existing `inspect.parameter_sweep` machinery. Faces the designer marks as
functional, and every toleranced feature, are locked. Each proposal shows its cost and time
delta and goes through the revision workflow; accepted changes are measured by the inspection
loop, and real cycle times calibrate the estimates.

**What SolidCraft already has (fact, from the code).** `inspect.accessibility` (faces visible
from a direction), `inspect.draft`, `inspect.minimum_radius`, `inspect.interference`, plastic
rules (minimum and maximum thickness, minimum draft), sheet metal rules (bend radius, relief
width and depth, corner relief), a hole feature that carries thread, thread depth, counterbore
and countersink, `inspect.parameter_sweep` (rebuild a design copy over a range of values) and
`document.revision`. There is no CAM yet (ROADMAP: Manufacturing / CAM milestone; #1153, #1621).
A first DFM check can be built from these before toolpaths exist.

## Fundamentals

### What "DFM" means here

- **Design for manufacturing (DFM):** shaping a part so it can be made, reliably and cheaply,
  by a chosen process. **DFMA** adds design for assembly. **DfAM** is the additive version
  (#1625; additive file).
- **DFM check (or DFM analysis):** an automatic review of a model against process rules, with
  results shown on the faces. Quote services call the result "manufacturability feedback",
  "manufacturing analysis" or "design analysis".
- **Cost drivers:** the features that dominate cost: stock volume, number of setups, small or
  long tools (slow), extra operations (ream, tap, EDM, grind), tight tolerances (slow passes and
  inspection), special tools.
- **Should-cost:** an estimate of what a part ought to cost from its geometry and a process
  model, as opposed to a supplier's quote (aPriori's term, from its public pages).

### The checks, by process

The numbers below are public rules of thumb, cited per row. They are **starting values** that a
shop overrides. "Recommended" and "feasible" follow Hubs' wording.

**CNC milling**

| Check | Rule of thumb | Source |
|---|---|---|
| Pocket depth | ≤ 4× pocket width recommended; past 6× tool diameter needs special tooling | Hubs design guide |
| Inside vertical corner radius | ≥ ⅓ of the pocket depth; a little more than the tool radius (about +1 mm) for finish | Hubs |
| Sharp inside corner | cannot be milled; machined at 2 mm radius if the drawing does not say otherwise; EDM if truly sharp | RapidDirect, Hubs blog |
| Floor radius | 0.5 mm, 1 mm or none recommended; any radius with a ball end tool | Hubs |
| Wall thickness, metal | 0.8 mm recommended, 0.5 mm feasible | Hubs, RapidDirect |
| Wall thickness, plastic | 1.5 mm recommended, 1.0 mm feasible | Hubs |
| Feature height to width | below 4:1 | Hubs blog |
| Hole diameter | standard drill sizes; anything over 1 mm feasible | Hubs |
| Hole depth | 4× diameter recommended, 10× typical, 40× feasible | Hubs |
| Thread size | M6 or larger recommended; taps down to M2 | Hubs |
| Thread length | at least 1.5× diameter, 3× recommended | Hubs |
| Blind tapped hole below M6 | 1.5× diameter unthreaded at the bottom | Hubs |
| Small features | below 2.5 mm counts as micro-machining | Hubs |
| Undercuts | standard T-slot and dovetail sizes; clearance ≥ 4× undercut depth to other walls | Hubs |
| End mill cutting length | typically 3–4× diameter | Hubs |
| Tolerance | ±0.1 mm typical, ±0.02 mm feasible; ISO 2768 m or f by default | Hubs |
| Setups | three to four orientations generally acceptable | Hubs |
| Lathe candidate | cylindrical parts flagged "OK to machine on lathe" | Protolabs |
| Part size | outside the process envelope is flagged "Part exceeds limitations" | Protolabs |

**Sheet metal** (T = thickness, R = inside bend radius; Fabcon guide, general starting values)

| Check | Rule of thumb |
|---|---|
| Minimum flange length | 4T |
| Inside bend radius | ≥ 1T for mild and stainless steel; ≥ 0.5T for 3003/5052 aluminium; 1.5–3T for 6061-T6 |
| Hole edge to bend line | ≥ 2.5T + R |
| Hole edge to part edge | ≥ 1T, 1.5–2T preferred |
| Hole to hole | ≥ 2T centre to centre |
| Bend relief | width ≥ 1T, depth ≥ R + T |
| Minimum hole diameter | ≥ 1T punched (laser can go smaller on thin sheet) |
| Slot | width ≥ T; length to width < 5:1 laser, < 3:1 punch |
| Notch corner radius | ≥ 0.5T |

**Injection moulding** (Protolabs public pages)

| Check | Rule of thumb |
|---|---|
| Wall thickness by resin | for example ABS 1.14–3.56 mm, nylon 0.76–2.92 mm, PC 1.02–3.81 mm (converted from inch) |
| Thickness variation | thinner walls no less than 40–60 % of the adjacent wall |
| Draft | 0.5° on vertical faces minimum, 2° in most cases, 3° at shut-offs and light texture, 5°+ for heavier texture; about 1° per inch (25.4 mm) of depth |
| Zero-draft faces | highlighted |
| Undercuts | side actions within a size limit |
| Fill | flow analysis maps fill pressure |

**Additive.** Covered by the additive file and #1625 (overhangs, minimum walls and holes,
trapped powder or resin, support access). Protolabs notes that thick walls and radii matter
less in printing than in moulding.

### How the services show results (fact)

- **Protolabs:** colour-coded status per part (white: needs configuration, grey: analysis
  running, yellow: review advisories, red: revisions required or cannot manufacture, green:
  ready). Required changes block ordering; recommended changes can be accepted as limitations.
  "Material left behind" is shown in blue with the percentage milled vs left. Threads are
  assigned by clicking holes, with the cost impact shown immediately. For moulding, gate and
  ejector layouts and "Protolabs Proposed Revisions", a revised model offered free for review.
- **RapidDirect:** a "Review manufacturability feedback" button beside the 3D viewer; issues
  highlighted on the part and listed under "Failed checks".
- **Xometry:** automated checks for voids, oversize dimensions, thin areas, sharp internal
  corners, floating parts and tool access, also inside its CAD add-ins (Onshape, Fusion). The
  help article says the feedback "can include false positives" and offers manual review.
- **Hubs:** three checks in the instant tool (hard-to-remove material, sharp internal corners,
  thin walls), next to the 3D viewer.

### How a check finds the features: feature recognition

A rule like "hole depth ≤ 10× diameter" needs to know what a hole is. There are two routes.

**1. Design intent (no recognition).** When the part was modelled in SolidCraft, the feature
tree says what each feature is: `solid.hole` carries diameter, depth, thread, `thread_depth`,
counterbore and countersink; plastic and sheet metal features carry their rules. This is exact.
It breaks only when later features cut through the hole, or the geometry is imported (STEP) or
edited directly.

**2. Recognition from the B-rep (for imported or edited geometry).** The academic field is old
and well documented (Wikipedia "Feature recognition" summarises it):

- **Graph-based:** build an attributed adjacency graph of faces (edges marked concave or convex)
  and match subgraph templates for slots, pockets, steps (Joshi and Chang, 1988). Fails when
  features intersect.
- **Hint-based:** look for the minimum evidence of a feature (two opposing planar faces hint at
  a slot) and grow it (Vandenbrande and Requicha, 1993). Handles intersections; produces false
  hints.
- **Hybrid:** graph reasoning to find safe regions, hints for the rest (Rahmani and Arezoo,
  2006–07).
- **Volume decomposition:** split the removed volume (stock minus part) into machinable volumes
  (Kim 1990; Sakurai and Chin 1993). Natural for CAM, because the removed volume is what the
  toolpaths cut.
- **Learning on B-rep graphs:** face classification networks trained on synthetic datasets.
  MFCAD (15,488 models, 16 feature classes) and MFCAD++ (59,655 models, 3–10 features each)
  are the usual training sets; methods include Hierarchical CADNet, UV-Net (Autodesk, MIT
  licence per a third-party acknowledgement page), BRepNet (with the Fusion 360 Gallery
  segmentation set of 35,680 real designs labelled by modelling operation), AAGNet and
  BrepMFR (MIT licence per the same page). Dataset licences were not checked (unverified).

The Wikipedia summary notes the classic problem: real industrial parts with hundreds of faces
make template matching explode, and most research targets 2.5D features.

**A third route that matters most for CAM (opinion):** many DFM questions do not need to name
the feature at all. "Can a tool reach this face?", "how thin is the material here?" and "what
is left after the available tools cut?" are geometric questions:

- **Thickness** by casting rays inward along the normal, or inscribing spheres (the
  sphere method catches thin regions between non-parallel faces).
- **Accessibility** per approach direction (SolidCraft's `inspect.accessibility` already shades
  faces visible from a direction).
- **Concave radius** on inside edges and faces (SolidCraft's `inspect.minimum_radius` measures
  curvature in sketches; the same idea on solid edges gives the smallest tool that fits).
- **Rest material:** subtract what each library tool can sweep from each setup direction; what
  remains is Protolabs' "material left behind", and it is exact for the shop's real tools.

### Tool reach is a physics question, not a lookup

"Pocket depth ≤ 4× width" stands in for a chain the CAM Physics System and tool holding files
describe in full:

1. The inside corner radius R sets the largest tool that finishes the corner: D ≤ 2R, and in
   practice a tool radius a little smaller than R so the tool does not wrap the full corner.
2. The depth h sets the shortest **stick-out** L that reaches the floor while the holder nose
   clears the walls (the holder profile is in the tool library).
3. The tool is a cantilever: tip stiffness `k = 3·E·I / L³`, with `I = π·d_eff⁴ / 64` and
   `d_eff ≈ 0.8·D` for a fluted tool (tool holding file, unverified first estimate). The holder,
   taper and spindle add compliance in series (machine library file).
4. The cutting force at the chosen parameters (cutting physics file) times the compliance gives
   tip deflection, which must stay inside the tolerance and finish of that face; the tool's
   bending frequency falls with `1/L²` and moves stable speeds.

So a deep, tight corner is not "too deep" in itself: it is too deep **for the tools, holders
and machine you have, at the cost you accept**. That is the check SolidCraft can make and a
quote service cannot.

### Tolerance feasibility

A quote service applies a default general tolerance (Hubs: ISO 2768 m or f; Xometry: ISO 2768,
or ±0.005 in (0.13 mm) for features of size on metals, from its US standards page as summarised
in the inspection file). Tighter tolerances are accepted with a drawing and priced up.
SolidCraft's inspection file proposes **general-tolerance profiles per process** and tolerances
as first-class data in the model. The DFM check then compares every toleranced feature with the
capability of the operation planned for it (milled ±0.1 mm typical; reamed or bored for H7;
ground for tighter) and reports the consequence: "adds a finishing operation", "needs a
temperature-controlled measurement", or "beyond this machine".

## Worked example (computed; process values illustrative)

A 6061 aluminium block, 100 × 60 × 40 mm, with a closed pocket 80 × 40 mm and 30 mm deep,
modelled with 3 mm inside corner radii and a sharp floor, plus four M5 tapped blind holes 15 mm
deep and one 10 H7 bore.

**1. Corner radius vs depth.** R = 3 mm limits the finishing tool to about D = 6 mm (in practice
5 mm for a non-wrapping cut). The rule of thumb wants R ≥ 30/3 = 10 mm. With R = 10 mm a 16 mm
end mill finishes the corners.

**2. What that does to the tool (from the tool holding formula, carbide E ≈ 600 GPa,
stick-out 33 mm to reach a 30 mm floor):**

| Tool | d_eff | I (mm⁴) | k_tool (N/mm) | Deflection at 100 N |
|---|---|---|---|---|
| 6 mm end mill | 4.8 mm | 26.1 | ≈ 1,300 | ≈ 77 µm |
| 16 mm end mill | 12.8 mm | 1,318 | ≈ 66,000 | ≈ 1.5 µm |

Same stick-out, about fifty times the stiffness (stiffness scales with D⁴). With the 6 mm tool
the wall finish and the ±0.1 mm general tolerance are at risk unless the cut is made very
lightly, which is what makes small deep corners slow. The 6 mm tool also needs a 5× D reach,
beyond the 3–4× D cutting length of a standard end mill (Hubs), so it is a long-reach or necked
tool.

**3. What that does to time (illustrative removal rates, unverified, for the shape of the
argument only).** The pocket removes 96,000 mm³ (96 cm³). At an assumed 15 cm³/min with the
6 mm tool that is about 6.4 min of roughing; at an assumed 100 cm³/min with the 16 mm tool about
1 min. The real numbers come from the physics system's limits for the actual tool, holder and
machine.

**4. Threads.** M5 is below the recommended M6, and a blind M5 wants 1.5 × 5 = 7.5 mm unthreaded
below the thread: with a 15 mm hole the thread can be about 7.5 mm long, which is exactly
1.5 × D, the minimum. Proposal: thread depth 10 mm (2× D) and drill depth 17.5 mm if the wall
below allows, or M6.

**5. Bore.** 10 H7 is 0 / +0.015 mm, tighter than a milled ±0.1 mm: flagged "adds a ream (or
bore) operation and a bore gauge check", which the inspection file turns into a probe or gauge
step.

**6. Stock.** With an assumed 3 mm per side allowance the blank is 106 × 66 × 46 mm
(321,816 mm³, about 0.87 kg at 2.70 g/cm³) for a part of 144,000 mm³ (0.39 kg): 45 % of the
bought metal ends up in the part. If the design can be 35 mm tall instead of 40, the blank is
106 × 66 × 41 mm (286,836 mm³, 0.77 kg), 11 % less metal, and it may move to a thinner standard
plate or bar (the stock marketplace file sizes the request; prices come from suppliers).

The optimiser's proposal list for this part would read: raise pocket corner radius 3 → 10 mm
(roughing and finishing tool 6 → 16 mm, time saved, deflection risk removed); add a 1 mm floor
radius or keep sharp (cost of a second tool); thread depth 7.5 → 10 mm; part height 40 → 35 mm
if not functional (stock saving). Each line names the faces it touches; locked faces are never
touched.

## Accuracy

- **Rules of thumb are not limits.** Hubs' "feasible" column (0.5 mm metal walls, 40× D holes,
  ±0.02 mm) shows how far the recommended values are from what a good shop can do. A check that
  reports rules as errors produces exactly the false positives Xometry and Protolabs warn about.
  Opinion: rules are **advice**; only physics and resource limits are **blockers**.
- **Recognition is the main error source** for imported geometry: intersecting features,
  fillets that hide pockets, and freeform faces confuse template methods; learned methods are
  trained on synthetic parts and their accuracy on real industrial parts is not established in
  the sources read here (unverified).
- **Thickness by rays** misses thin regions between non-parallel faces; sphere or medial-axis
  methods catch them but cost more. Both depend on tessellation density for curved faces.
- **Accessibility** from a fixed set of directions misses what a 5-axis machine or a lollipop
  tool can reach; the honest result is per machine and per tool library.
- **Cost estimates** are only as good as rates and removal rates. Toolpath (the company) and
  CloudNC compute cycle times from real machining strategies (their public pages); quote
  services price from their own models and markets. A design-time estimate should show its
  inputs and be calibrated by measured cycle times (inspection and machine health files).

## Programs

All from public pages; behaviour behind the pages is not known.

- **Protolabs** (ProtoQuote / manufacturing analysis): per-process advisories, required vs
  recommended changes, colour status, material left behind, lathe suitability, thread
  assignment with immediate cost, moulding flow, gates and ejectors, proposed revised models.
- **Xometry** (instant quoting engine; Onshape and Fusion add-ins): automated DFM checks (voids,
  oversize, thin areas, sharp inside corners, floating parts, tool access), price inside the CAD
  tool, manual review for false positives.
- **Hubs / Protolabs Network:** three instant checks; the most detailed public CNC design guide
  of the group (the numbers above).
- **RapidDirect:** DFM report with failed checks (small and deep holes, sharp corners, thin
  walls, narrow areas, hard-to-remove volume).
- **aPriori (aP Design, aP Pro):** should-cost for designers, many processes (machining, sheet
  metal, moulding, casting, forging, additive, assembly), setups, tolerances, machine
  requirements, "see how a design change will impact tooling, cycle time, or fixture costs",
  CO2e, CAD plug-ins.
- **Toolpath:** machinability analysis by automatic feature recognition, tooling recommendations
  for hard features, cycle-time estimating from real machining strategies, toolpaths in Fusion.
- **CloudNC CAM Assist cycle time estimator** (announced September 2024, Mastercam and Fusion):
  cycle times, strategies, and per-part part volume, stock volume, machining efficiency and
  operation count.
- **Paperless Parts:** shop-side quoting; geometric interrogation finds dimensions, features and
  DFM warnings "based on the shop's capabilities"; sheet metal unfolding and nesting.
- **CAD-embedded DFM** (SOLIDWORKS DFMXpress, others): rule checks for drilling, milling,
  turning, sheet metal and moulding inside the CAD tool (rule details not confirmed; the help
  page did not load, unverified).

### Which website was it? (Jason's question)

Jason remembers a site that took a solid model, used AI to analyse CNC manufacture, and offered
design changes that made the stock cheaper and the machining shorter. From public pages, the
closest matches are:

| Candidate | What its public pages say | Fits Jason's description? |
|---|---|---|
| **aPriori aP Design** | for design engineers; should-cost; impact of design changes on tooling, cycle time, fixtures; material use | closest on "suggests design changes to cut cost" |
| **Toolpath** | AI feature recognition, machinability, cycle time, estimates | close on "AI analyses CNC"; design changes not stated |
| **CloudNC CAM Assist estimator** | AI cycle time, stock volume, machining efficiency | close on stock and time; inside Mastercam/Fusion, not a website |
| **Protolabs** | proposed revised models (moulding), material left behind, lathe advice | design revisions, but not AI-branded and not cost-ranked |
| **Paperless Parts** | shop quoting with DFM warnings | for shops, not designers |

None of the pages read says, in those words, "change this to make the stock cheaper". **Open
for Jason:** if you recall the name or the URL, it goes here.

## Complaints and requests

From the services' own pages (fact) and opinion where marked:

- **False positives.** Xometry's help article says automated feedback "can include false
  positives" and offers manual review by e-mail; Protolabs lets users accept recommended
  changes as limitations. Opinion: the checks are generic, so they flag what a given shop
  could make.
- **Silent defaults.** Sharp inside corners are machined at 2 mm radius unless the drawing says
  otherwise (RapidDirect, Hubs). The designer may not notice the fit has changed.
- **Too late.** The check runs at quote time, after the design is done; Protolabs' full
  analysis for moulding comes back by e-mail, typically within hours.
- **No reason, no fix, no price per change** (opinion). Most results are a highlighted face and
  a label. Protolabs' thread assignment is the rare case where the cost of a choice is shown
  immediately.
- **Not your shop** (opinion). A shop with a 5-axis machine, long-reach tools or EDM gets the
  same flags as one with a 3-axis mill.
- **Tolerances barely checked** (opinion, from the lists above): the instant checks are about
  geometry; tolerances are handled by default classes and drawings.

## What users need

Everything below is opinion: a proposal for discussion.

### 1. One check, three resource profiles

The same `dfm.check` runs against one of three profiles:

- **Generic** (ships with SolidCraft): the public rules of thumb above, each with its source, as
  editable values. Works with no CAM, no libraries, no network. For designers sending parts out.
- **Marketplace** (optional plugin): a service's published rules as a profile, for a designer
  who will order from that service. Rules only from public pages, never scraped.
- **My shop:** the tool, holder, machine and workholding libraries from the sibling files. Here
  reach, deflection, envelope, setups and tolerances are computed, not assumed.

### 2. What each finding carries

```text
Finding {
  rule:        "milling.inside_corner_radius"   // stable id, documented
  severity:    blocker | cost | advice          // blocker = no resource can make it
  faces:       [persistent face names]          // highlighted in the viewport
  measured:    { radius: 3.0, depth: 30.0 }     // mm
  limit:       { radius_min: 10.0 }             // and where it came from:
  bound_by:    "rule (Hubs, 1/3 depth)" | "tool T12 6 mm, stick-out 33 mm, deflection 77 µm > 25 µm"
  consequence: "long-reach 6 mm tool; ~5 min extra roughing (estimate)"
  fixes:       [ { move: "corner_radius", to: 10.0, saves: { minutes: 5.4, cost: … } } ]
  accepted:    null | { by, when, note }        // "accept as limitation", recorded
}
```

Severity follows Protolabs' useful split, made stricter: **blocker** only when no available
resource can make it (so rule-of-thumb violations are never blockers); **cost** when it can be
made but needs a special tool, operation or setup; **advice** otherwise. Accepting a finding is
recorded with who and when (the compliance file's record pattern), and the finding stays
visible but quiet.

### 3. The checks, in the order they become possible

**Phase 1, before any CAM (design side, generic profile).** Built on what exists:

- wall and rib thickness (new ray or sphere thickness analysis; plastic rules already hold
  minimum and maximum thickness);
- inside corner radius vs depth (concave edges and cylindrical concave faces, depth from the
  accessibility direction);
- holes from the hole feature: depth / diameter, standard drill size, thread size and length,
  unthreaded length below blind threads, hole near an edge or a bend;
- draft and zero-draft faces (existing `inspect.draft` with the plastic rule's `min_draft`);
- sheet metal: flange length, bend radius vs material, hole to bend, relief size (the sheet
  rules already hold relief width and depth);
- accessibility per setup direction (existing `inspect.accessibility`), counting how many
  directions are needed: the setup count;
- envelope against a chosen process size;
- lathe candidate: the body is (nearly) a solid of revolution;
- tolerances against the process tolerance profile (inspection file), once tolerances are data;
- imported geometry: the same checks on recognised holes and pockets (hybrid graph and hint
  recognition first, learning later and optional).

**Phase 2, with the libraries (my-shop profile).** Reach from the real tool and holder
profiles; smallest finishing tool per corner; stick-out, deflection and stable speed from the
physics system; machine envelope, axes and spindle limits from the machine library; workholding
clamp faces and access (workholding file); hardness regions from the physical properties file
(a hardened region turns a cheap cut into a hard-milling or grinding operation).

**Phase 3, with CAM.** Rest material after the planned operations (exact "material left
behind"), real cycle time per operation, the simulation gate from the CAM Physics System.

### 4. At design time, not quote time

- The check runs in the background after each rebuild, like a linter, and shows a count in the
  status bar; the panel lists findings by severity; clicking one selects and highlights its
  faces (the viewport already draws analysis overlays).
- Long checks follow the performance skill: off the window thread, cancellable, and in wasm.
- `dfm.check` is a command with input and output schemas (the commands skill), so MCP agents and
  scripts can run it and read findings as JSON. It never opens a dialog when called
  programmatically.
- Hostile geometry (open shells, zero-thickness faces, huge parts) returns findings or errors,
  never a panic (crash-safety skill).

### 5. The design-to-CAM cost optimiser (Jason's idea)

**Goal:** suggest design changes that reduce stock cost and machining time, with numbers, and
keep the function of the part.

**The cost model** (shown with all inputs, editable rates):

```text
part cost = stock        (blank volume × price, from the stock request / supplier quote)
          + setups       (setup count × setup time × rate)
          + machining    (Σ operation time × machine rate; times from the physics limits)
          + tooling      (special tools, wear share)
          + extra ops    (ream, tap, EDM, grind, heat treat, finish)
          + inspection   (checks implied by tolerances, from the inspection file)
```

**Moves.** A small library of named, explainable changes, each a parameter or feature edit:

| Move | Changes | Usually saves |
|---|---|---|
| Raise inside corner radius | fillet value | smaller/longer tool, finishing time |
| Floor radius to standard tool | fillet value | a tool change |
| Snap a size to stock | an overall dimension | blank size, a facing pass |
| Shorten a pocket or thread | depth parameter | reach, tap risk |
| Hole to standard drill / thread to standard | hole feature | special tool |
| Open a pocket to a side | sketch/extent | reach, chip evacuation |
| Turn a feature to a setup direction already used | feature direction | a setup |
| Relax a tolerance to the profile default | PMI value | a finishing op and an inspection |
| Split or merge a body | structure | stock form (bar vs plate) |

**Search.** For each applicable move, rebuild a copy of the design with the new value
(SolidCraft's `inspect.parameter_sweep` already rebuilds copies over a range of values and
reports failures, without touching the design), re-run the DFM check and the cost model, and
keep moves that pass and save. Combine the best non-conflicting moves greedily. No black box:
every proposal is a list of moves with their deltas. An AI model can help rank moves or
recognise features on imported geometry, but every number comes from the physics and cost
models, and the reason is always shown.

**What is never changed.** Faces the designer marks as functional ("keep": mating faces, seal
faces, datums), every toleranced or GD&T feature unless the move is explicitly "relax
tolerance", anything referenced by an assembly joint, and anything driven by a configuration
the designer locks. The optimiser asks before touching a parameter used elsewhere.

**How changes are applied.** Proposals are shown as a side-by-side preview (existing
`inspect.compare` for geometry). Accepting creates a new design revision (`document.revision`;
the compliance file's release and change-impact records), so an ECO-style shop sees which
features changed and which first-article checks must be repeated (delta FAI, compliance file).

**Closing the loop.** After the part is made, the inspection file's per-part record and the
machine data (machine health file) give measured cycle times and measured feature results.
They calibrate the time model per machine and tool, and show whether an accepted change (for
example a relaxed tolerance or a thinner wall) actually held, which is the "Self-tuning CAM
Operations" loop applied to design.

### 6. Other processes

The same finding model covers sheet metal and moulding (rules already in SolidCraft), turning,
and additive (with the additive file and #1625). For moulding a later phase can add simple
thickness-based fill hints; a real flow simulation is out of scope here.

### 7. Phased plan (opinion)

1. **Generic DFM check** (no CAM): thickness analysis, corner radius vs depth, hole and thread
   rules from the hole feature, draft and sheet metal rules, setup count from accessibility,
   findings panel and `dfm.check` command, rule values with sources in a data file.
2. **Tolerances**: check toleranced features against process profiles (needs the inspection
   file's tolerance data).
3. **My-shop profile**: tool reach and deflection with the libraries; machine envelope.
4. **Cost model** and **optimiser** on top of phase 1–3 estimates, proposals as revisions.
5. **With CAM**: rest material, real cycle times, calibration from measured times.
6. **Imported geometry**: hybrid recognition; optional learned recogniser if a permissively
   licensed model and dataset can be used and run in pure Rust and wasm.

## Export control, sanctions and government use

*Added 2026-10-29 from the [government use research](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-qk5riz/research/gov-research-nsnlookupjn-government-use.md). Opinion and policy proposals, not legal advice.*

**Our three rules:**

- **No one goes to jail.** Our team has no in-house counsel. So wherever a rule is unclear we take the most conservative reading and refuse rather than warn. Questions that need an outside export-control lawyer are marked as such, and free official help (BIS, DDTC, OFAC) is the first call.
- **Help users comply.** SolidCraft stops and explains in plain language, using rule packs for each jurisdiction that the administrator picks. It flags conflicts between countries' laws, never resolves them, and never gives legal advice.
- **Prevention first.** Nothing in this file that is hosted, shared, paid or networked ships until its compliance gates are in place and checked: region blocks, screening, ITAR flag handling, government-mode behaviour, and an outside lawyer's sign-off for any hosted or paid service.

**What in this file crosses a border:** the cost optimiser's supplier quotes, uploads to online manufacturers and marketplace links.

- **Hosted parts block sanctioned regions and screen people.** Any service this file proposes that the project would run blocks comprehensively sanctioned regions, plus Russia and Belarus (US, EU and UK rules restrict CAD/CAM software and related services there), and screens accounts, payers, contributors and marketplace parties against the US Consolidated Screening List and the UK, EU and UN lists.
- **No support there.** No support, consulting or custom work for users in sanctioned regions or for listed parties.
- **Government mode.** At the Controlled level (CUI and allied equivalents) online parts are off or point to the organisation's own mirror; at the Isolated level they arrive only as signed packs on media; a design flagged Export controlled (ITAR/EAR or national equivalent) never leaves through them.
- **Only generic data is shared.** Nothing derived from a controlled design (geometry, toolpaths, part or drawing numbers, test or inspection results) goes into a shared library, report or issue. Tool data stays shareable through the field allow-list in the high-security research.
- **Code stays public.** Every build the project ships is public, which keeps it "published" software outside the EAR. The rules are tracked by the standing export and sanctions watch.
- **Checked for controlled data (2026-10-29):** this file contains no controlled technical data. Its examples are public sources, generic values or made-up stand-ins, and the author's experience is stated in general terms.

## Glossary

- **Advisory:** a manufacturability note that does not block ordering (Protolabs' term).
- **Attributed adjacency graph (AAG):** faces as nodes, shared edges as arcs marked concave or
  convex; the classic input to graph-based recognition.
- **Blocker / cost / advice:** the three severities proposed here.
- **Cost driver:** a feature or choice that dominates the part's cost.
- **DFM / DFMA / DfAM:** design for manufacturing / and assembly / for additive manufacturing.
- **Feature recognition:** finding machining features (holes, pockets, slots) in a B-rep.
- **Material left behind / rest material:** stock the available tools cannot remove.
- **Move:** one named, explainable design change the optimiser can try.
- **Should-cost:** an estimate of what a part ought to cost from its geometry and a process
  model.
- **Stick-out:** tool length from the holder nose to the tip.
- **Process tolerance profile:** the tolerance a process holds by default and at best
  (inspection file).

## Open questions

1. Which upstream issue should own this: #1621 (CAM research) as proposed here, or a separate
   design-side issue, since phase 1 needs no CAM?
2. Should rule values ship as a data file per process (like the plastic and sheet metal rules),
   and who maintains their sources?
3. Is a thickness analysis (rays or spheres) wanted as a general inspect command in its own
   right? It also serves plastic parts and castings.
4. Would the maintainers accept an optional learned feature recogniser, given the pure-Rust and
   wasm rule? Which runtime, and which model and dataset licences are acceptable?
5. Where do shop rates (machine rate, setup time) live: the machine library, the shop record, or
   a cost profile?
6. Should "functional face" locks be a design-side attribute (also useful for drawings and
   GD&T), or local to the optimiser?
7. Jason: which website did you remember (see "Which website was it?")?

## Sources

Accessed 2026-10-26.

**Services and vendors**
- Protolabs, Manufacturing Analysis (help centre): https://protolabs.com/help-center/manufacturing-analysis
- Protolabs, A deep dive into design analysis: https://www.protolabs.com/en-gb/resources/blog/a-deep-dive-into-design-analysis-accessing-analysis-interpreting-feedback-optimising-your-cad-model/
- Protolabs, Plastic injection moulding design guidelines: https://www.protolabs.com/services/injection-molding/plastic-injection-molding/design-guidelines/
- Xometry, Instant Pricing App for Onshape FAQ: https://community.xometry.com/kb/articles/721-xometry-app-for-onshape-faqs
- Xometry, manufacturing standards (via the inspection file): https://www.xometry.com/manufacturing-standards/
- Hubs, How to design parts for CNC machining: https://hubs.com/knowledge-base/how-design-parts-cnc-machining
- Hubs, Reduce the costs for CNC machining with instant DfM feedback: https://www.hubs.com/blog/reduce-the-costs-for-cnc-machining-with-instant-dfm-feedback/
- RapidDirect, DFM analysis feature update: https://www.rapiddirect.com/blog/rapiddirect-new-dfm-analysis-feature/
- Fabcon, Sheet metal DFM guide: https://fabcon.com/articles/?p=127
- aPriori, DFM: https://www.apriori.com/dfm-plus
- aPriori, aP Design flyer (PDF): https://www.apriori.com/wp-content/uploads/2025/11/aP-Design-Flyer.pdf
- Toolpath, machinability analysis: https://toolpath.com/platform/design-advisor
- Toolpath, estimating: https://toolpath.com/platform/estimating
- Digital Engineering 24/7, CloudNC AI estimating (11 Sep 2024): https://www.digitalengineering247.com/article/ai-solution-launches-to-boost-cnc-machining-estimating-process/cloudnc
- Paperless Parts, pricing and costing automation: https://www.paperlessparts.com/pricing-costing-automation/
- SOLIDWORKS help, DFMXpress (page did not load): https://help.solidworks.com/2022/english/SolidWorks/dfmxpress/r_settings.htm

**Feature recognition**
- Wikipedia, Feature recognition: https://en.wikipedia.org/wiki/Feature_recognition
- Geometric Deep Learning for Computer-Aided Design: A Survey (arXiv 2402.17695): https://arxiv.org/pdf/2402.17695
- Jayaraman et al., UV-Net: Learning from Boundary Representations (arXiv 2006.10211): https://arxiv.org/pdf/2006.10211
- MFCAD++ dataset (Queen's University Belfast): https://pure.qub.ac.uk/en/datasets/mfcad-dataset-dataset-for-paper-hierarchical-cadnet-learning-from/
- Tech Soft 3D, HOOPS AI acknowledgements (licences of UV-Net and BrepMFR): https://docs.techsoft3d.com/hoops/ai/resources/Acknowledgments.html

**SolidCraft (this repository)**
- `crates/engine/src/cmd/surface_analysis.rs` (`inspect.accessibility`, `inspect.draft`),
  `crates/engine/src/cmd/sketch_inspect.rs` (`inspect.minimum_radius`),
  `crates/engine/src/cmd/parameter_sweep.rs`, `crates/engine/src/cmd/inspect.rs`
  (`document.revision`), `crates/engine/src/cmd/features.rs` (`solid.hole`),
  `crates/doc/src/plastic.rs`, `crates/doc/src/sheet.rs`, `docs/features.md`, `ROADMAP.md`.
