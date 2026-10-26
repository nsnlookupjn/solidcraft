# CAD/CAM research: physical properties of the design and of the CAM workpiece

Research for [#1621](https://github.com/tangency-labs/solidcraft/issues/1621) (Research Request
for Plugin: CAM), with a design-side half that also belongs next to SolidCraft's physical
material cards ([docs/physical-material-cards.md](../docs/physical-material-cards.md)). This
file covers Jason's request:

> **A model should define not only dimensions and roughness but also hardness and other physical
> properties, for the whole part and per surface or feature. CAM should know the hardness, and
> the local hardness, of the stock and of the in-process stock. And a design should be able to
> describe stock surfaces that are not machined, such as the skin of a casting.**
>
> Added later: **design to final size with tolerances once, derive from it the casting, forging
> or 3D-printed stock design, load that into CAM and machine it to final size; and plan heat
> treatment, which changes size, followed by re-machining to the final tolerance.**

It is one of a set of files from the same author that describe CAM as **one physical system**.
This file adds the **material state** (what the part and the stock are made of, how hard they are
and where) that the others read:

| Part of the system | File | What it owns |
|---|---|---|
| Overview | [CAM physics system](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-vgqo3g/research/cam-research-nsnlookupjn-physics-system.md) | the shared setup, the solver loop over all limits, the one risk slider |
| Tool library | [Shared cloud tool library](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-tool-library/research/cam-research-nsnlookupjn-tool-library.md) | tool, holder and cutting-data records, `kc1.1` per material group |
| Cutting physics | [Cutting tool physics](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-cutting-physics/research/cam-research-nsnlookupjn-cutting-physics.md) | force vector, torque, power per move; the "machinability" block on a material card |
| Workholding | [Workholding physics](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-workholding/research/cam-research-nsnlookupjn-workholding.md) | clamp force, contacts, friction per surface condition |
| Machine library | [Machine library](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-machine-library/research/cam-research-nsnlookupjn-machine-library.md) | per-axis stiffness and thrust, spindle curve |
| Stock integrity | [Stock structural integrity](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-stock-integrity/research/cam-research-nsnlookupjn-stock-integrity.md) | in-process stock geometry, stiffness, residual stress, cut sequence |
| Inspection | [Part certification by in-process inspection](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-y2w06n/research/cam-research-nsnlookupjn-in-process-inspection.md) | tolerances, surface texture, PMI, probing, the per-part record |
| Additive | [Additive process physics](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-j72y6h/research/3d-printing-research-nsnlookupjn-process-physics.md) (#1626) | printed and DED preforms, whose properties vary by layer |
| **Physical properties (this file)** | | hardness, case depth, heat treatment, coatings and surface condition, per body, region and depth, in the design, the stock and the in-process stock |

**Not repeated here:** size and geometric tolerances, ISO 21920 surface texture, roughness
prediction from feed and step-over, and STEP AP242 / QIF PMI basics are in the inspection file
("Tolerances in the model"). Kienzle `kc1.1`/`mc`, chip thinning and the tool-library schema are
in the cutting physics file. Residual stress and release distortion are in the stock integrity
file. This file links to them and adds only what is new.

Licence: MIT OR Apache-2.0, the same as SolidCraft (see the Licence section of CONTRIBUTING.md).

## About this research

- **Author:** Jason ([@nsnlookupjn](https://github.com/nsnlookupjn)). Idea and direction by Jason;
  source research and writing done with an AI assistant (Claude) in October 2026.
- **How it was produced:** web searches of public sources (standards catalogue pages, an
  industry association's notes on ISO 18203, published papers' abstracts, finishers' and
  foundries' technical pages, machinists' public forums), each claim checked against the cited
  page where possible. Standards are paid documents; only their public scope pages and previews
  were read. No commercial CAD/CAM program was driven or scripted, no GPL/LGPL/AGPL source was
  read, and no text is copied beyond short quotes. The worked examples are computed for this
  file from the formulas shown.
- **Units:** millimetres, micrometres for coatings and case depths below 1 mm, newtons,
  megapascals, Vickers (HV), Rockwell C (HRC) and Brinell (HB) hardness as the source gives them.
  SolidCraft's material cards store SI values; hardness has no SI unit (see "Hardness is not one
  number").
- **Marking:** "(unverified)" marks a claim not confirmed against a source. **Fact**,
  **opinion** and **marketing** are labelled where they mix.

> **[TODO Jason: personal experience]** Parts you have made that were heat treated, case
> hardened, plated or anodised between operations; castings, forgings or flame-cut blanks whose
> skin or edges were hard; how the drawing told you (notes, chain lines, a spec number), and what
> went wrong when CAM did not know (broken tools on a hard edge, a bore that closed up after
> coating, a case ground through). Label this as personal experience.

## Summary

1. **A part is not one material.** Most CAD stores one material per body, for mass and
   appearance. Real parts have **regions**: a carburised journal and a soft threaded end; an
   induction-hardened track; a hard-anodised bore and a masked one; a cast surface left as cast
   next to a machined face. Each region has properties **through its depth**: a case, a
   transition, a core. Drawings already say this, in notes and with chain lines (ISO 15787 for
   heat-treated steel parts). A model-based definition should say it as data, attached to faces
   the way tolerances are.
2. **Three levels, one pattern.** (a) The **material** (the alloy, its card of properties;
   SolidCraft already has this). (b) The **body condition**: the state the whole part is
   delivered or finished in (annealed, quenched and tempered to 28–32 HRC, solution treated and
   aged). (c) **Regions** of faces or areas on faces, each with a treatment, a property range,
   a depth profile and a note on when it applies (before or after coating, after grinding). Each
   level overrides the one above it.
3. **The same record describes the stock.** A casting, a forging, a hot-rolled bar or a
   flame-cut plate is a body with regions too: cast skin, forging scale and decarburised layer,
   mill scale, the hardened heat-affected zone of a thermal-cut edge. CAM should **read the
   stock's properties from the same kind of record the design uses**, not from a separate list.
4. **The in-process stock carries a property field.** As material is cut away the skin goes,
   the case is reached or removed, and a heat treatment between setups changes everything that
   is left. The in-process stock (the stock integrity file's model) should carry, for each piece
   of remaining material, which property region it belongs to and its depth below the original
   surface, so that each move can look up the **hardness in the cut**.
5. **Hardness changes the physics of every move.** At the same chip, a harder workpiece needs
   more force and power, wears the tool faster and can move the material into another ISO 513
   group (H, hardened) with other tools. In the worked example below, the same cut in 4140 at
   30 HRC instead of annealed raises the predicted force by about 45 %, and holding the old
   force needs about 60 % of the old feed. The limits engine in the overview file then does the
   rest.
6. **Properties drive the process plan, not just the cutting data.** A case depth specified
   on the finished part, a grinding allowance after hardening and a coating's growth together
   set **intermediate targets**: how deep the heat treater must case, how far oversize to machine
   before coating, where to leave material so a case can be cut off a zone that must stay soft.
   Software can compute these and catch impossible combinations (an H7 bore under a 50 µm hard
   anodise that varies by ±20 % cannot be pre-compensated; it must be masked).
7. **Unmachined surfaces are part of the design.** A cast or forged part needs a model of the
   **raw part** (with draft, fillets, parting line, machining allowance per face and the casting
   tolerance grade) and a model of the **finished part**, linked so that each finished face knows
   whether it is machined or left as cast, and each machined face knows how much material is on
   it. ISO 8062-3 gives the tolerance and allowance grades; datum targets (ISO 5459) locate the
   raw part for the first setup. SolidCraft's configurations are a natural home for "as cast"
   and "as machined" states of one design.
8. **Design once at final size; derive the stock** (Jason's idea). From the final part with its
   tolerances, derive the raw blank for the process (casting, forging, 3D-printed preform, bar)
   and any intermediate states (before and after heat treatment). CAM loads the blank, ideally
   as measured, and machines it to the final part. Heat treatment changes size (D2 grew 0.08 %
   in one direction in a published example, with a recommended 0.15 % per side allowance), so a
   "before heat treatment" state carries extra material that is re-machined to final tolerance
   afterwards, checked against the case depth that the finishing must not remove.

## Fundamentals

### Hardness is not one number

Hardness is a **test result**, not a material constant: the resistance to an indenter under a
given load, read on a given scale. The common scales use different indenters and loads, so they
measure different things and only convert approximately. **Fact.**

| Scale | Standard | Indenter and reading | Typical use |
|---|---|---|---|
| Brinell HBW | ISO 6506-1, ASTM E10 | hard-metal ball, diameter of the impression | castings, forgings, soft and medium steels, large grains |
| Vickers HV | ISO 6507-1, ASTM E92 | diamond pyramid, diagonals of the impression; micro-loads for layers | case-depth traverses, thin layers, all hardnesses |
| Rockwell HRC, HRB… | ISO 6508-1, ASTM E18 | depth under a major load after a minor load | hardened steels (HRC), soft metals (HRB); quick shop checks |
| Leeb HL | ISO 16859-1, ASTM A956 | rebound speed of an impact body | portable checks on large parts (unverified that both standard numbers are current) |
| Knoop HK | ISO 4545-1 | elongated diamond, micro-loads | very thin layers, coatings |

**Conversion.** ISO 18265 gives conversion tables between scales and to an estimated tensile
strength, separately for groups of materials (unalloyed and low-alloy steel, cast steel,
quenched-and-tempered steel, cold-work steel, high-speed steel, tool steel, hard metals,
non-ferrous metals). It warns that a converted value is only directly applicable to the exact
material tested, and that tensile strength estimates are the least reliable converted values
([ISO 18265:2013 page](https://www.iso.org/standard/53810.html)). ASTM E140 plays the same role
in the US. **Fact.** The shop rule of thumb for steel, tensile strength in MPa ≈ 3.45 × HB, is
an approximation of this kind and only for steels (unverified as to which table it comes from).

**What the model must store, therefore (opinion):** a hardness value is a **range** (min, max or
nominal ± band), a **scale** with its load where it matters (HV 1, HV 10, HRC, HBW 10/3000), the
**place** it applies (surface, a depth, the core), and the **source** (spec, measured, converted
from another scale, and by which table). Converted values must stay marked as converted. This
matches the existing material card design, where each property already carries a source and
conditions ([physical-material-cards.md](../docs/physical-material-cards.md): "Conditions belong
to the property").

### Hardened layers and how their depth is defined

Surface treatments make a **hard case over a softer core**. The depth of the case is defined by
where a hardness traverse (a series of Vickers indents from the surface inwards on a cut
section) falls below a **limit hardness**. ISO 18203 (2016; a 2026 edition is listed) defines
these depths ([AWT notes on DIN EN ISO 18203, 2023](https://www.awt-online.org/awt-wAssets/docs/publikationen/2023-09-22-AWT-Hinweise_Anwendung_DIN_EN_ISO_18203_final.pdf)):

| Depth | After | Limit hardness |
|---|---|---|
| **CHD**, case-hardening hardness depth | carburising (or carbonitriding) and hardening | **550 HV** (the maximum hardness of martensite at 0.35 % carbon, per the AWT notes) |
| **SHD**, surface-hardening hardness depth | induction, flame, laser or electron-beam hardening | usually **80 % of the minimum specified surface hardness** in HV; other values by agreement |
| **NHD**, nitriding hardness depth | nitriding, nitrocarburising | **core hardness + 50 HV** ([ZwickRoell, NHD](https://www.zwickroell.com/industries/metals/metals-standards/nitriding-hardness-depth-nhd-din-50190-3/)) |
| **CLT**, compound layer thickness | nitriding | not by hardness; measured on a micrograph (DIN 30902 per the AWT notes) |

ISO 18203:2016 combined and replaced ISO 2639 (carburised case depth), EN 10328 (surface
hardening depth) and DIN 50190-3 (nitriding hardness depth), per the AWT notes. SAE J423 defines
the US methods, including a **total case depth** where the hardness curve meets the core.
**Fact.**

So a case specification is not "1 mm case" but, for example, **"carburised and hardened,
surface 58–62 HRC, CHD 0.8–1.2 mm, core 30–40 HRC"**: a surface range, a depth range at a
defined limit, and a core range. A model can store this as a **depth profile**: surface value,
depth to the limit, core value, and the shape between them (a straight or smooth fall; the real
curve is measured, not specified). **Opinion** on the representation; the fields are standard.

### How drawings show heat treatment today

**ISO 15787** (2016; earlier edition 2001) is "Technical product documentation, heat-treated
ferrous parts, presentation and indications": how a drawing states the heat-treated condition of
a steel or iron part ([BSI catalogue entry](https://knowledge.bsigroup.com/products/technical-product-documentation-heat-treated-ferrous-parts-presentation-and-indications)).
From public descriptions (unverified against the standard text): the drawing gives the final
condition in words (for example "quench hardened and tempered", "case hardened and tempered",
"nitrided"), the hardness and its scale with tolerance, the hardening depth with its definition,
and, where only part of the surface is treated, marks the treated zone with a **thick
dash-dot (chain) line** next to the surface, dimensioned like a feature; zones that must not be
hardened, and hardness test points, can be marked too. ASME Y14.5 likewise uses a chain line to
mark a **limited area** to which a requirement applies (fact; the chain line convention is
widely documented).

Coatings and plating are usually a **note with a specification**: for example a hard anodise to
a military or ISO anodising specification with a class and thickness, zinc plating to ISO 2081
with a passivation, electroless nickel to a thickness, plus a list of masked features. The
critical sentence on many drawings is whether **dimensions apply before or after coating**
([Richconn, hard anodising dimensional control](https://richconn.com/hard-anodizing-dimensional-control/)
suggests a note such as "DIMENSIONS APPLY BEFORE COATING"). **Fact** (common practice; the
anodising specification numbers and their current editions are not checked here).

### Semantic data for these properties

The inspection file explains presentation versus semantic PMI and the STEP AP242 and QIF
formats. For properties specifically:

- **STEP AP242** carries a material designation per part and generic property and attribute
  mechanisms; the MBx-IF (formerly CAx-IF) recommended practices for PMI
  ([v4.0.6 PDF](https://mbx-if.org/home/wp-content/uploads/2024/06/rec_pracs_pmi_v406.pdf)) cover
  dimensions, geometric tolerances, datums, surface texture and notes. Whether there is an agreed
  practice for **semantic** heat treatment, case depth or coating on a face set, rather than a
  note, was not found (unverified; to be checked against the practices before an exporter is
  designed). A note attached to faces is the safe fallback: readable by people, linked to the
  right geometry.
- **QIF** (ISO 23952) characteristics can carry measured results per part; whether hardness is a
  standard characteristic type or needs a user-defined one is unverified.
- **Material certificates** (EN 10204 type 3.1 inspection certificates in Europe, mill test
  reports in the US) carry the heat number, chemistry and delivered hardness of a lot. These are
  the stock-side source for "what did we actually receive". **Fact.**

**Opinion:** SolidCraft should store properties semantically in its own design file first and
export them as AP242 notes on the right faces until a semantic practice exists. The internal
data is what CAM and inspection need; the export can improve later without changing the model.

### Unmachined stock surfaces

**Castings.** ISO 8062-3 (2023 edition) gives general dimensional and geometrical tolerances and
**machining allowance grades** for castings: dimensional casting tolerance grades (**DCTG**),
geometrical casting tolerance grades, and required machining allowances (**RMA**) with grades
(RMAG), how to indicate them on drawings, and definitions of draft angles; it advises agreeing
the parting surfaces and draft angles with the foundry before the design is finished
([ISO 8062-3:2023 preview](https://webstore.ansi.org/preview-pages/ISO/preview_ISO+8062-3-2023.pdf)).
**Fact.** The grade tables were not readable in the preview; the letters A to K for RMAG and
DCTG 1 to 16 are commonly quoted (unverified).

What a cast surface means for machining (fact, from machinists' accounts; details unverified):
- The **skin** of a sand casting is harder and more abrasive than the metal under it and can hold
  sand. Machinists advise taking the first cut **under the skin**, about 1.5 mm (1/16 in) on cast
  iron, and grinding the skin off by hand when there is not enough allowance
  ([Model Engineer forum, machining castings](https://model-engineer.co.uk/?p=4714)).
- **Chilled spots** (white iron) form in thin sections and edges that cool fast; they can be hard
  enough that carbide struggles, vary between batches, and annealing does not always cure them
  (same thread).
- Raw castings are located by **datum targets** (points, lines or small areas on the raw
  surface, ISO 5459 / ASME Y14.5) because a cast face is not flat enough to be a datum plane.
  **Fact.**

**Forgings and hot-rolled stock.** Forgings carry flash lines, draft and **scale** (iron oxide,
abrasive), and steel heated in air can lose carbon near the surface, leaving a **decarburised
layer** that is **softer** than the core (fact, metallurgy texts; depth is process dependent).
Hot-rolled bar and plate carry **mill scale**. Closed-die steel forgings have their own tolerance
standard in Europe (EN 10243-1, unverified as current). Cold-drawn bar can be work hardened at the
surface (unverified magnitude).

**Thermally cut plate.** Plasma, oxy-fuel and laser cutting leave a **heat-affected zone** at the
cut edge. In steels with enough carbon the edge can **quench harden**; the hardness depends on
the heat input ([VŠB thesis abstract, effect of heat input on the hardness of plasma cut
edges](https://dspace.vsb.cz/items/1fc82278-38c4-41a9-a904-fcd6bc51e6fb)). Fabricators report
plasma-cut edges hard enough to ruin bandsaw blades and drills; air plasma is worse than laser
with nitrogen assist; the advice is to grind through the edge at the entry point, take cuts deep
enough to stay in soft material and mill conventionally so the cutter does not dive into the
hard surface ([iForgeIron forum, hardened edges](https://www.iforgeiron.com/topic/64933-hardened-edges-on-plasma-and-lased-cut-steel-plate/)).
ISO 9013 classifies thermal-cut quality (perpendicularity, roughness) but not edge hardness
(unverified). **Fact** for the effect; magnitudes vary.

**Additive preforms.** Wire-arc and powder DED preforms, and printed metal parts, vary in
hardness by layer and by distance from the substrate; see the additive file (#1626). The record
in this file can describe them as regions or layered depth profiles.

### Hardness made during machining: work hardening

Austenitic stainless steels and nickel alloys **work harden**: the cut leaves a thin layer harder
than the bulk under the new surface, and a later pass that takes a chip thinner than that layer
cuts in hardened material and makes it worse. In Inconel 718, wider and steeper edge chamfers
and larger uncut chip thickness both deepened the hardened layer, while the maximum surface
hardness changed little; the measured range in that study was 378 to 560 HV
([Materials 2022, work hardening of Inconel 718 by edge geometry](https://pmc.ncbi.nlm.nih.gov/articles/PMC8780571)).
**Fact.** The practical rule, "do not rub; keep the chip thicker than the hardened layer", is
standard advice (unverified as a single source). In the model this is a property **made by the
toolpath**, so it belongs to the in-process stock, not the design.

### What hardness does to cutting

From the cutting physics file: tangential force `Fc = kc · b · h` with `kc = kc1.1 · h^(−mc)`.
Harder material of the same alloy has a higher `kc1.1`. Material groups in ISO 513 split steels
(P) from hardened materials (H), with different tool grades. Hardness also shortens tool life
(the Taylor constant falls), and hard, abrasive skins (scale, cast skin, thermal-cut edges) wear
the edge at the depth-of-cut line. **Fact** (textbook machining; the cutting physics file has the
sources).

Published `kc1.1` tables usually give a value per group at a typical hardness (the tool library
file quotes Sandvik's values, for example 1700 N/mm² for low-alloy steel, not hardened). A simple
correction **within one group** (opinion; a common approximation, not a standard) scales
`kc1.1` with tensile strength, and so roughly with Brinell hardness for steels:
`kc1.1(HB) ≈ kc1.1(HB_ref) · HB / HB_ref`. Better is a vendor or measured value at the actual
hardness, which the tool library's cutting-data records can hold per hardness band.

## Worked examples

### 1. The same cut in soft and hardened 4140

Side milling with one tooth engaged, axial depth `b = 6 mm`, mean chip thickness `h = 0.03 mm`,
`mc = 0.25`, rake correction ignored. 4140 annealed about 197 HB; quenched and tempered to
30 HRC, about 286 HB (approximate conversion; typical table value, unverified against
ISO 18265). Reference `kc1.1 = 1700 N/mm²` at the annealed state (tool library file).

| | Annealed (197 HB) | Q&T 30 HRC (≈ 286 HB) |
|---|---|---|
| `kc1.1` (scaled by HB) | 1700 N/mm² | 1700 · 286/197 ≈ 2470 N/mm² |
| `h^(−mc)` = 0.03^(−0.25) | 2.40 | 2.40 |
| `kc` | ≈ 4090 N/mm² | ≈ 5930 N/mm² |
| `Fc` per engaged tooth = `kc · b · h` | ≈ 740 N | ≈ 1070 N |

The force rises about 45 %. To hold the annealed force, the chip must shrink: since
`Fc ∝ h^(1−mc) = h^0.75`, `h_new = h · (1/1.45)^(1/0.75) ≈ 0.61 · h`, so about 60 % of the feed
per tooth. Cutting speed would also come down for tool life. **This is what the limits engine in
the overview file does automatically once it knows the hardness in the cut.** If the part is
hardened **between** operations, the roughing and the finishing see different material, and a
plan that used one material for both is wrong for one of them.

### 2. A case depth through the process chain

Design: a shaft journal, "carburised and hardened, 58–62 HRC, **CHD 0.8–1.2 mm after
grinding**". Process: turn, carburise and harden, grind. The grinder must remove enough to clean
up the heat-treatment distortion; suppose it removes **0.10 to 0.25 mm per side** depending on
how much the shaft moved (illustrative numbers).

- Least case allowed before grinding: worst-case removal plus the minimum finished case,
  `0.8 + 0.25 = 1.05 mm`.
- Most case allowed before grinding: least removal plus the maximum finished case,
  `1.2 + 0.10 = 1.30 mm`.

So the heat treater gets **CHD 1.05–1.30 mm**, a band of 0.25 mm instead of the design's 0.40 mm.
If distortion needed up to 0.45 mm, the band would vanish (`1.25` to `1.30`), and the software
should say so before anyone cuts metal: reduce distortion, widen the case tolerance or grind
less. **Opinion:** this "derived intermediate specification" is one of the most useful things a
CAD/CAM program that knows properties can do, because today it is done by hand on the traveller.

**Selective hardening** fits the same model. If the threaded end must stay soft, the common
options are a stop-off (masking paste or copper plating) during carburising, or **carburising
the whole part and machining the case off the soft zone before quenching** (common heat-treat
practice; unverified source). The second option is a CAM operation that exists only because of
a property region: the turned diameter on the thread zone must be left oversize by more than the
case depth before carburising, then cut down before hardening. A region-aware planner can create
that step.

### 3. A bore under a hard anodise

Ø20 H7 bore, 20.000–20.021 mm, in a part with a Type III hard anodise. Per the finisher's notes
cited above, Type III defaults to about 50 µm when the drawing does not say, finishers hold
about ±20 % of nominal, and the coating grows about 50 % outward and 50 % into the metal, so a
bore's diameter shrinks by about the coating thickness: here **40–60 µm**.

To land in H7 after coating, the bore would have to be machined to `20.000 + 0.060` at least and
`20.021 + 0.040` at most: **20.060–20.061 mm**, a 1 µm band. That cannot be machined. The same
source says pre-compensation is not suitable for H7 or tighter fits and threads, and recommends
**masking** them. **A program that knows the coating region and the tolerance can flag this at
design time** and propose: mask the bore (a property region with "no coating"), or specify the
dimension before coating with a looser fit.

### 4. The first cut on a casting

A cast face with a required machining allowance of 2.5 mm, a casting that varies ±1.0 mm on that
face (illustrative), and a skin about 1.5 mm deep (the forum figure above). The first pass removes
between 1.5 and 3.5 mm depending on where the real surface is. At the low end the tool runs
**exactly at the bottom of the skin**, the worst place: the depth-of-cut line of the edge rubs
scale and sand. The planner should (opinion):
1. **probe the cast surface first** (the inspection file's cycles) and set the pass from the real
   surface, or
2. plan the first pass at least `skin + casting variation` deep where force and power allow, and
3. use a tool and grade chosen for interrupted, abrasive cuts on that pass only.

## From the final part to the stock and back (Jason's idea)

Jason's request: **design the part once, at final size with its tolerances; from it, produce the
stock design for the process that makes the blank (a casting, a forging, a 3D-printed preform, or
bar and plate); load that stock into CAM and machine it to final size; and where the part is heat
treated, which changes its size, plan the extra material and re-machine it to the final tolerance
after heat treatment.**

### The chain of states

One part passes through several **states**, each a body with its own geometry, tolerances and
properties:

| State | Geometry | Who uses it |
|---|---|---|
| **Final** (the design) | nominal size, tolerances, property regions | the designer, inspection |
| **Before finishing** (one per finishing step) | final + finishing allowance on each finished face | CAM for the step before; the heat treater |
| **Raw blank** | final + all allowances, plus what the blank process needs (draft, fillets, filled small holes, supports) | CAM's stock for the first setup; the foundry, forge or printer |
| **Pattern / die / build file** | raw blank scaled or compensated for the process (casting shrink, print warping) | the pattern maker, die maker or build preparation |

**The final part is the master; every other state is derived from it**, so a change to the design
flows down to the blank, the pattern and the CAM stock. Opinion: SolidCraft's configurations
already switch materials and appearances per state ([docs/configurations.md](../docs/configurations.md));
a "stock state" could be a configuration whose extra features are the derivation steps below,
and CAM picks the configuration it needs as its stock and the next one down as its target.

### Deriving the raw blank, per process

Every face of the final part is tagged, using the property regions of this file, as **machined**
(it gets an allowance) or **left raw** (as cast, as forged, as printed; no allowance, and its
tolerance must be achievable by the blank process). Then, per process (fact for the allowance
types; values are illustrative or as cited):

- **Casting.** Classic pattern allowances are shrinkage, draft, machining (finish), distortion and
  rapping ([NPTEL, metal casting lecture 3](https://archive.nptel.ac.in/content/storage2/courses/112107144/metalcasting/lecture3.htm)).
  In SolidCraft terms: offset each machined face by its required machining allowance (ISO 8062-3
  RMA grade, or per face); add draft to faces parallel to the pull direction away from the parting
  line; raise inside corners to a minimum fillet radius; fill holes below the smallest diameter
  the foundry can core; then, for the **pattern only**, scale by the contraction allowance. The
  NPTEL table gives, for patterns up to 2 ft, 0.125 in/ft for grey cast iron (about 1.0 %),
  0.251 in/ft for cast steel (about 2.1 %) and 0.155 in/ft for aluminium (about 1.3 %), falling for
  larger patterns. The cast part is the raw blank; the pattern is a separate, scaled state.
- **Forging.** The same steps with forging rules: draft on die walls, a parting (flash) line,
  larger corner and fillet radii, filled holes and pockets the die cannot form, machining allowance
  over scale and the decarburised layer (see "Unmachined stock surfaces"), and the forging
  tolerance standard for the raw faces (EN 10243-1 in Europe; unverified as current).
- **3D-printed preform** (metal powder bed or DED). Allowance only on machined faces, usually
  small for powder bed and larger for DED; extra material on support-side faces that will be
  cleaned up; **sacrificial features** added for the machining: probe and datum pads, fixturing
  tabs or a clamping boss, removed in the last operation. Shrink and warp compensation belongs to
  build preparation (the additive file, #1626), so the build file is its own state.
- **Bar, plate or sawn blank.** The smallest standard size that contains the final part plus
  allowances and the clamping material the workholding plan needs (the workholding file); this is
  the simplest derivation and the one CAM programs usually do today.

Opinion: these are mostly existing modelling operations. SolidCraft already has
`solid.offset_face`, `solid.draft`, `solid.scale` and `solid.shell` commands; a derivation is a
recorded chain of them driven by the face tags, so it updates when the design changes (the faces
are found by their persistent names, [docs/face-attachments.md](../docs/face-attachments.md)).
Nothing here needs the paused kernel work beyond what those commands already do; where they fail
(for example draft on complex faces), the derivation reports which face and leaves the rest.

**Checks only this link makes possible** (opinion):
- a face left raw whose tolerance is tighter than the casting or forging grade must be machined;
- a machined face with less allowance than the blank's tolerance plus the skin depth will not
  clean up (worked example 4 above);
- a sacrificial feature that collides with a finished face or with the workholding;
- the raw blank's weight and volume against the foundry's or printer's limits, using the material
  density already on the material card.

### Heat treatment changes size: the re-machining step

Hardening moves a part in three ways: a **size change** (growth or shrink, from the change of
structure; it can differ by direction relative to the steel's rolling direction), **distortion**
(bending, bowing, out-of-round from residual stress and uneven cooling; see the stock integrity
file), and, for case hardening, a hard case that the finishing step must not remove. **Fact.**

Published example: a 3.15 in (80 mm) cube of D2, vacuum hardened, **grew 0.08 % in one direction
and shrank in the other two**; size changes from hardening and from tempering add together, and
tempering temperature (so the hardness asked for) changes the result. The recommended **minimum
machining allowance was 0.15 % per side**, assuming a stress relief between roughing and
semi-finishing, and more without it. The article's main advice is to make the size change
**repeatable** by controlling the heat treatment, and to know the part's orientation to the
mill's grain direction ([Production Machining, predicting size change from heat treatment](https://www.productionmachining.com/articles/predicting-size-change-from-heat-treatment)).
**Fact** for these numbers; they are one steel and one process.

**Worked example (D2 die block, 80 mm, illustrative).** Size change 0.08 % of 80 mm is 0.064 mm.
Allowance 0.15 % per side is 0.12 mm per side, 0.24 mm on the size. So the semi-finished block
before hardening is 80.24 mm (shifted by the predicted size change in the direction that grows or
shrinks, if it is known), and after hardening the hard-milling or grinding step removes about
0.12 mm per side to reach 80 mm within tolerance. That last step cuts material at maybe 58–62 HRC,
so its cutting data come from the hardened state through the property field above, not from the
annealed material the roughing used.

**For a case-hardened part the allowance has a ceiling as well as a floor.** The finishing
allowance must exceed size change plus distortion plus a clean-up minimum, but must stay below
what the case can lose and still meet its finished depth (worked example 2). Software that holds
both the size-change model and the case specification can show the window, or show that there is
none.

**What the model needs** (opinion):
1. A **heat-treatment step** in the job (it is not a machine operation): the process, its
   specification, and a **size-change model** for the material and process: expected change per
   material axis (the material card's `material123` axes can carry the rolling direction), a
   distortion band, and its source (steel maker's data, the heat treater's records, the shop's own
   measurements).
2. A **"before heat treatment" stock state** derived from the final part: each finished face
   offset by the planned finishing allowance and shifted by the predicted size change. CAM
   machines the raw blank to this state before heat treatment.
3. An **"after heat treatment" stock state** = the before state with the size change applied and
   the distortion band attached as uncertainty. CAM's finishing operations start from it.
4. A **clean-up check** on the real part after heat treatment: probe or scan it (the inspection
   file), place the final part inside the measured part by best fit, and confirm there is enough
   material on every finished face, before any finishing cut. If not, stop and report; it is
   cheaper to find a scrap part with a probe than with the last cut.
5. **Learning:** measured size change per heat-treat batch goes back into the size-change model,
   the same closed loop the overview file calls Self-tuning CAM Operations.

### Loading the real stock into CAM

The derived blank is **nominal**. A real casting, forging or printed preform differs from it, so
CAM should locate the final part inside the **measured** blank, not the nominal one:

- **Probe or scan, then best-fit.** For AM parts, a trade article describes touch probing of datum
  features for location, multipoint best-fit when there are no clean datum planes, a half-depth
  "gauge cut" to measure deflection and apply an offset, and on-machine scanning to build a model of
  the as-built part from which an adaptive toolpath is generated, with re-probing to catch drift
  ([Manufacturing Engineering, April 2026, adaptive measurement-driven machining of AM parts](https://digitaleditions.walsworth.com/article/Adaptive%2C+Measurement%E2%80%91Driven+Machining+for+Additively+Manufactured+Parts/5123110/861841/article.html)).
- **Per-operation placement.** For cast near-net blanks, Cserteg, Kovács and Váncza (CIRP Annals,
  2023) place each operation's group of features with its own part zero instead of one rigid
  placement for the whole part, under a minimum allowance and the tolerances between operations.
  In their case study (an automotive casting, four operations) the largest achievable allowance
  rose from about 0.15 mm to about 0.33 mm, about 2.2 times, and the average tolerance error at a
  0.14 mm allowance fell from 14.7–19.8 % to 3.4 %
  ([CIRP 2023 paper](https://akovacs.pages.sztaki.hu/publications/CIRP2023.pdf)). **Fact** (their
  results; physical tests were still in progress when published).
- **The measured blank becomes the stock.** Whatever is measured replaces the nominal raw state as
  CAM's starting stock, with the property regions (skin, case) carried over by face, so the
  property field and the limits engine work on the real part.

### Commands this suggests (opinion)

In SolidCraft's own command style, each with a contract and output schema:

| Command | Does |
|---|---|
| `stock.derive.casting` | raw blank from tagged faces: allowance per face or RMA grade, pull direction and parting line, draft, minimum fillet and cored-hole size; optional pattern state with shrink |
| `stock.derive.forging` | the same with forging rules |
| `stock.derive.additive` | allowance on machined faces, support-side allowance, sacrificial probe pads and tabs |
| `stock.derive.bar` | smallest standard bar or plate plus allowance and clamping material |
| `stock.stage.add` | a stock state between operations (finishing allowance, heat treatment with size-change model, coating) |
| `stock.check` | the checks above: raw-face tolerances, clean-up, allowance window against case depth, collisions of sacrificial features |
| `stock.measured.set` | replaces a nominal state with a probed or scanned one and records the fit |

## What exists today

### In SolidCraft

- **Physical material cards** ([docs/physical-material-cards.md](../docs/physical-material-cards.md)):
  per-material SI properties with sources and conditions, including a free-text `state` for
  "grade, temper, processing or specimen condition". Property kinds are density, elasticity,
  tensile, compressive, yield and shear strength, thermal conductivity and expansion, and specific
  heat. **There is no hardness kind**, and a card belongs to a material, not to a region of a
  body. (Checked in the docs and the source in October 2026.)
- **Semantic face appearances and configurations** ([docs/configurations.md](../docs/configurations.md)):
  a configuration can switch a body's physical material and per-face appearances. Faces are
  addressed by body and a point on the face.
- **Persistent face names** ([docs/face-attachments.md](../docs/face-attachments.md)): faces carry
  history names (`F5:top`, `F7:blend:…`, `F4:top@F6.2`) that follow them through later features.
  These are what a property region should store, so a region follows its faces when the design
  changes.
- **Cosmetic threads** ([docs/cosmetic-threads.md](../docs/cosmetic-threads.md)): an example of
  information shown on faces without changing geometry, which property regions can copy for
  display (hatching or a tint on treated faces, a chain line on drawings).
- The cutting physics file already proposes a **machinability block** (ISO 513 group, hardness,
  `kc1.1`, `mc`) on the material card. This file extends that to hardness **per region and depth**.

### In other programs (from public documentation; details unverified)

- Mainstream CAD programs let a user assign a material per body or part and, in some, per face for
  appearance. Heat treatment and coatings are almost always **text notes** on drawings or in
  the title block, not data that CAM reads.
- CAM programs ask for a stock material (often from a list of material groups with a hardness
  shown) per setup. The material and hardness apply to the **whole stock**, for the whole program.
  No program found tracks hardness per region of the in-process stock. Users enter a different
  "material" per operation by hand when a part is hardened between operations (unverified as to
  how common this is).
- Casting design tools and some CAD programs model the **raw casting** and the **machined part**
  as two linked models or two configurations; the machining allowance is added as offset faces
  (fact for the practice; product details not checked here).

**Opinion:** the gap is not the property list (standards define it well) but the **link**: from
property to faces, from faces to the stock, from the stock to each move.

## Complaints and requests

Paraphrased from public forums and the sources above (unverified as to frequency):

- "The edge was hard and broke the drill / ruined the blade" (thermal-cut plate).
- "Chilled spots in the casting; carbide just skids" (thin cast sections).
- "The bore closed up after anodise / plating; nobody said dimensions were before coating."
- "The case was ground through on one side" (distortion larger than the grinding allowance
  allowed for).
- "CAM only lets me pick one material for the job, but half the operations are after hardening."
- "The note says 'harden as marked' and the marking is a chain line on one view only."

## What users need

### Design side: properties as first-class data

1. **Hardness on the material card**, as a property kind with scale, range and source, so a
   body's default hardness is known. Converted values marked as converted.
2. **A body condition**: the delivered or final heat-treatment state (name, spec, hardness range
   and scale, depth profile if any), separate from the material so one material can be used in
   several conditions. Configurations can switch it (for example "as machined" and "hardened").
3. **Property regions**: a set of faces (by persistent name) or an area on faces bounded by a
   sketch (the drawing chain line), with one or more of:
   - **heat treatment**: process (carburise, nitride, induction harden…), surface hardness range,
     depth type and range (CHD/SHD/NHD and its limit), core range;
   - **coating or plating**: process and spec, thickness range, growth split (outward/inward),
     whether dimensions apply before or after coating;
   - **masking / exclusion**: "no coating", "no hardening", "soft zone";
   - **surface process**: shot peening intensity and coverage, burnishing, grinding;
   - **as-cast / as-forged / not machined**: the face is left in its raw state;
   - **test points**: where hardness is to be checked, with scale and load.
4. **Display**: tint or hatch treated regions in the viewport (off by default, like cosmetic
   threads); on drawings, the chain line and note per ISO 15787; in exports, AP242 notes on the
   faces.
5. **Checks** that only properties make possible: tolerance versus coating variation (example 3),
   case depth versus grinding allowance (example 2), a soft zone that a hardened region
   overlaps, a test point on a masked face.

### Stock side: the raw part

1. A **stock body with regions**, using the same record: surface condition per face (cast skin,
   forging scale, decarburised layer, mill scale, thermal-cut HAZ, sawn, pre-machined), each with
   a depth profile (hard skin over normal core, soft decarburised layer over normal core).
2. For castings and forgings, a **raw-part model** linked to the finished part: draft, parting
   line, machining allowance per face (from an RMA grade or per face), casting tolerance grade,
   datum targets. Opinion: an "as cast" configuration of the same design is the simplest link,
   since configurations already switch materials and appearances.
3. **Material certificate data** on the stock instance: heat number, delivered hardness measured,
   source document. This joins the per-part record in the inspection file.

### CAM side: the in-process stock carries properties

1. **The property field.** Every piece of remaining material knows its region (from the stock
   or from a treatment applied during the job) and its depth below the region's original surface.
   With a dexel stock (stock integrity file), each dexel segment can carry a region id and the
   depth of its ends; with voxels, each cell can. Hardness at a point = the region's depth profile
   at that depth. Opinion: this is a few bytes per segment and needs no kernel work.
2. **Per move, the hardness in the cut.** The cutting physics engine already computes the engaged
   volume per move; it should look up the hardness (or the material state) of that volume and use
   the matching `kc1.1`/`mc` and tool-life data. When a move crosses a skin or a case, it takes
   the worst along the move or splits the move.
3. **Group switching.** When the hardness in the cut crosses into ISO 513 group H, the planner
   should warn that the tool and its cutting data are for another group (the threshold is commonly
   quoted around 45 HRC; unverified as a standard boundary).
4. **Process steps between setups.** Heat treatment, plating and anodising are **operations in
   the job** (outside the machine), which change the property field of the in-process stock and
   its geometry (coating growth). The traveller in the inspection file is the natural place to
   list them, and the limits engine must use the post-treatment field for every later operation.
5. **Skin and edge rules** (opinion, from the shop practice above): first pass below the skin
   where possible; entry away from or through hardened edges; conventional rather than climb
   milling through scale (machinists' advice, unverified as a rule); a different tool for the
   skin pass.
6. **Work hardening.** For materials flagged as work hardening, a minimum chip thickness per pass
   (no rubbing), and an optional thin hardened layer added to the in-process stock after each
   pass, so a light spring pass is predicted to cut harder material (Level 2; opinion).
7. **Intermediate targets.** Oversize before coating, grinding allowance after hardening, extra
   allowance where a case must be cut off a soft zone: computed from the regions and shown on the
   operation (examples 2 and 3).
8. **Verification.** Hardness test points become inspection steps in the per-part record; a
   portable tester (Leeb, UCI) result is a measured value against the region's range. Probing the
   raw surface before the first cut (example 4) uses the inspection file's cycles.

### Interfaces with the other files

| From | To | Quantity |
|---|---|---|
| This file | Cutting physics | hardness (scale, value) and material group of the engaged volume per move; skin/edge flag |
| This file | Workholding | surface condition at each contact (cast, scale, machined) for the friction coefficient; local hardness for indentation of serrated jaws (opinion) |
| This file | Stock integrity | region-dependent stiffness only if a treatment changes the modulus (rarely; hardness barely changes steel's modulus, fact); residual stress from case hardening and peening as an input to release distortion |
| This file | Inspection | property regions as characteristics: hardness test points, coating thickness, case depth; dimensions before or after coating |
| This file | Additive | layered depth profiles for printed preforms |
| Stock integrity | This file | the in-process stock structure (dexels or voxels) that carries the property field |
| Inspection | This file | measured hardness and coating thickness per part, to update the field and close the loop |

## Data model sketch (opinion)

Field names are illustrative; the point is the shape. Values are SI where an SI unit exists;
hardness keeps its scale.

```json
{
  "hardness": {
    "scale": "HRC",
    "load": null,
    "min": 58, "max": 62,
    "source": { "kind": "specified", "ref": "drawing note 4" }
  },

  "depth_profile": {
    "surface": { "scale": "HV", "min": 650, "max": 750 },
    "depth": { "kind": "CHD", "limit_hv": 550, "min_m": 0.0008, "max_m": 0.0012 },
    "core": { "scale": "HRC", "min": 30, "max": 40 },
    "applies": "after_grinding"
  },

  "property_region": {
    "id": "R1",
    "faces": ["F5:side:l1", "F9:blend:…"],
    "bounded_by": { "sketch": "S7", "side": "inside" },
    "treatments": [
      { "kind": "heat_treatment", "process": "carburise_harden_temper",
        "spec": "customer spec text", "profile": "<depth_profile>" },
      { "kind": "coating", "process": "anodise_hard", "spec": "…",
        "thickness_m": { "min": 4.0e-5, "max": 6.0e-5 }, "outward_fraction": 0.5,
        "dimensions_apply": "after_coating" },
      { "kind": "exclude", "what": "coating" },
      { "kind": "raw_surface", "state": "as_cast", "skin_depth_m": 0.0015,
        "skin_hardness": { "scale": "HBW", "min": 220, "max": 300 } }
    ],
    "test_points": [ { "point": [12.0, 0.0, 30.0], "scale": "HRC" } ]
  },

  "body_condition": {
    "name": "Q&T", "hardness": { "scale": "HRC", "min": 28, "max": 32 },
    "spec": "…", "source": { "kind": "certificate", "ref": "heat 12345" }
  }
}
```

**How it fits SolidCraft's rules** (opinion; checked against AGENTS.md and CONTRIBUTING.md):
- **Everything is a command.** Ids in SolidCraft's own style, for example
  `material.properties` gaining a `hardness` kind; `body.condition.set`;
  `property.region.add / edit / remove`; `property.region.inspect`; `property.check` (the
  feasibility checks above). Each with a contract and output schema.
- **Design file format.** One new `crates/doc/src/format/pending/<name>.rs` file for regions and
  body conditions; older designs need no migration (no regions = the body's material and
  condition everywhere, as today).
- **Layering.** Regions and conditions live in `doc`; the checks in `engine`; display in
  `ui-egui`; drawing chain lines in the drawing work; AP242 notes in `io`. The CAM property field
  belongs to the future CAM plugin, outside the core layers.
- **Kernel paused.** Nothing here needs new kernel features: regions reference faces by the
  existing persistent names; "bounded by a sketch" can be stored now and evaluated on a mesh.
- **Independent.** Fields come from public standards' terms (ISO 18203, ISO 15787, ISO 8062-3)
  and general practice; no other product's property lists or dialogs.

## Phased plan (opinion)

1. **Hardness on the material card** (scale, range, source, conversion flag), and the body
   condition. Small, useful at once: CAM and the cutting physics file can read one hardness per
   body.
2. **Property regions** on faces with heat treatment, coating, exclusion and raw-surface
   treatments; viewport display off by default; notes on export.
3. **Feasibility checks**: coating versus tolerance, case depth versus grinding allowance,
   overlapping soft and hard regions.
4. **Raw-part link**: "as cast" / "as machined" configurations with allowance per face and datum
   targets; stock regions for skin, scale and thermal-cut edges.
5. **CAM property field** on the in-process stock; hardness in the cut per move; treatment steps
   between setups on the traveller.
6. **Derived stock states**: bar and casting derivations from tagged faces with the existing
   offset, draft and scale commands; a heat-treatment stage with a size-change model; the
   clean-up check after heat treatment.
7. **Closed loop**: measured hardness, coating thickness and heat-treat size change from
   inspection update the field and the size-change model; measured blanks replace nominal ones;
   work-hardening layers (Level 2).

## Glossary

- **Body condition:** the heat-treatment or delivery state of a whole part (annealed, normalised,
  quenched and tempered, solution treated and aged).
- **Case:** a hardened surface layer over a softer core.
- **CHD / SHD / NHD:** case-hardening, surface-hardening and nitriding hardness depths, each
  measured to a defined limit hardness (ISO 18203).
- **CLT:** compound (white) layer thickness after nitriding; measured on a micrograph.
- **Chain line:** a thick dash-dot line beside a surface marking a limited area to which a
  requirement applies.
- **Chill:** hard white iron formed where a cast iron section cooled fast.
- **Decarburised layer:** a surface layer of steel that lost carbon during heating in air and is
  softer than the core.
- **DCTG / RMA / RMAG:** dimensional casting tolerance grade; required machining allowance and
  its grade (ISO 8062-3).
- **Datum target:** a point, line or small area on a raw surface used to locate the part.
- **HAZ:** heat-affected zone, here at a thermally cut edge.
- **Mill scale / forging scale:** iron oxide on hot-worked steel; hard and abrasive.
- **Property region:** (this file) a set of faces, or an area on faces, carrying treatments and
  property ranges.
- **Property field:** (this file) the region and depth of each piece of in-process stock, used to
  look up hardness in the cut.
- **Work hardening:** hardening of a material by plastic deformation, here by the cutting edge.

## Open questions

1. Is there an MBx-IF recommended practice, or an AP242 entity, for **semantic** heat treatment,
   case depth and coating on faces? If not, should SolidCraft propose one, or keep notes?
2. Should hardness be a property **kind** on the existing card (one value per card) or a
   **depth profile** that the card's simple value is a special case of?
3. Is "bounded by a sketch" on faces enough for partial treatments, or do users need imprinted
   split lines (which need the kernel)?
4. What threshold, and from which source, should switch a cut to ISO 513 group H?
5. How should a converted hardness (HRC from HB) be shown so nobody mistakes it for a measured
   one?
6. Should the "as cast" model be a configuration of the machined design, or a separate design
   that the machined one is derived from (and which owns the foundry's revisions)?
7. Can hardness per hardness band be added to the shared tool library's cutting data without
   multiplying records unmanageably?
8. Where do size-change models come from in an open project: steel makers' data sheets, heat
   treaters, or only the shop's own measurements?

## Sources

### Standards (catalogue and preview pages only)

- ISO 15787:2016, Technical product documentation, heat-treated ferrous parts, presentation and
  indications: [BSI](https://knowledge.bsigroup.com/products/technical-product-documentation-heat-treated-ferrous-parts-presentation-and-indications),
  [SCC](https://scc-ccn.ca/standardsdb/standards/8155001).
- ISO 18203, Steel, determination of the thickness of surface-hardened layers:
  [2016 at AFNOR](https://www.boutique.afnor.org/en-gb/standard/iso-182032016/steel-determination-of-the-thickness-of-surfacehardened-layers/xs023476/123505),
  [2026 at SCC](https://scc-ccn.ca/standardsdb/standards/8190912);
  [AWT notes on applying DIN EN ISO 18203 (2023, German)](https://www.awt-online.org/awt-wAssets/docs/publikationen/2023-09-22-AWT-Hinweise_Anwendung_DIN_EN_ISO_18203_final.pdf).
- [ZwickRoell, nitriding hardness depth](https://www.zwickroell.com/industries/metals/metals-standards/nitriding-hardness-depth-nhd-din-50190-3/).
- ISO 8062-3:2023, dimensional and geometrical tolerances for moulded parts, castings:
  [preview](https://webstore.ansi.org/preview-pages/ISO/preview_ISO+8062-3-2023.pdf).
- ISO 18265:2013, conversion of hardness values: [ISO](https://www.iso.org/standard/53810.html).
- MBx-IF, Recommended Practices for the Representation and Presentation of PMI:
  [v4.0.6](https://mbx-if.org/home/wp-content/uploads/2024/06/rec_pracs_pmi_v406.pdf).
- Hardness test standards named in the table (ISO 6506, 6507, 6508, 16859, 4545; ASTM E10, E18,
  E92, A956, E140), EN 10204, ISO 5459, ISO 9013, ISO 2081, EN 10243-1: names only, not read.

### Papers

- [Machining-induced work hardening behaviour of Inconel 718 considering edge geometries,
  Materials 2022](https://pmc.ncbi.nlm.nih.gov/articles/PMC8780571).
- [Effect of heat input on the hardness of plasma cut edges, VŠB thesis (abstract)](https://dspace.vsb.cz/items/1fc82278-38c4-41a9-a904-fcd6bc51e6fb).

### Derived stock and heat-treat size change (accessed 2026-10-26)

- [Production Machining, predicting size change from heat treatment](https://www.productionmachining.com/articles/predicting-size-change-from-heat-treatment).
- [NPTEL, metal casting, lecture 3: pattern allowances](https://archive.nptel.ac.in/content/storage2/courses/112107144/metalcasting/lecture3.htm).
- [Cserteg, Kovács, Váncza, multi-operation optimal blank localization for near net shape
  machining, CIRP Annals 2023](https://akovacs.pages.sztaki.hu/publications/CIRP2023.pdf).
- [Manufacturing Engineering, April 2026, adaptive, measurement-driven machining for additively
  manufactured parts](https://digitaleditions.walsworth.com/article/Adaptive%2C+Measurement%E2%80%91Driven+Machining+for+Additively+Manufactured+Parts/5123110/861841/article.html).

### Vendor, finisher and forum pages (accessed 2026-10-26)

- [Richconn, hard anodising dimensional control](https://richconn.com/hard-anodizing-dimensional-control/)
  (thickness default, ±20 %, 50/50 growth, masking for H7 and threads).
- [Model Engineer forum, machining castings](https://model-engineer.co.uk/?p=4714) (skin, chill),
  paraphrased.
- [iForgeIron forum, hardened edges on plasma and laser cut plate](https://www.iforgeiron.com/topic/64933-hardened-edges-on-plasma-and-lased-cut-steel-plate/),
  paraphrased.

### SolidCraft

- [docs/physical-material-cards.md](../docs/physical-material-cards.md),
  [docs/configurations.md](../docs/configurations.md),
  [docs/face-attachments.md](../docs/face-attachments.md),
  [docs/cosmetic-threads.md](../docs/cosmetic-threads.md).
