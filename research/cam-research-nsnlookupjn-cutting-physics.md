# CAM research: cutting tool physics and the parameters a tool library must hold

Research for [#1621](https://github.com/tangency-labs/solidcraft/issues/1621) (Research Request
for Plugin: CAM). This file covers one part of that request: **the physics of metal cutting with
milling tools, the standards and references behind it, the parameters a tool library must store
so CAM can calculate feeds, speeds and toolpaths, and how CAM uses those calculations.**

It is a companion to
[cam-research-nsnlookupjn-tool-library.md](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-tool-library/research/cam-research-nsnlookupjn-tool-library.md),
which covers sharing a tool catalogue, holders and assemblies, and tool tags. That file
introduces the Kienzle and edge-force models and gives a table of `kc1.1`/`mc` values; this file
does not repeat the table and goes deeper into the calculations and the data they need.

A second companion,
[cam-research-nsnlookupjn-workholding.md](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-workholding/research/cam-research-nsnlookupjn-workholding.md)
("Workholding physics: have CAM tell us how to hold the work"), takes the cutting forces computed
here as its input and returns workholding limits to the same feed calculation; the shared
interface is in "Outputs for workholding" below. A third,
[cam-research-nsnlookupjn-machine-library.md](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-machine-library/research/cam-research-nsnlookupjn-machine-library.md),
holds the machine data, including the spindle power and torque curves the checks here use.

Licence: MIT OR Apache-2.0, the same as SolidCraft (see the Licence section of CONTRIBUTING.md).

## About this research

- **Author:** Jason ([@nsnlookupjn](https://github.com/nsnlookupjn)). Idea and direction by Jason,
  including the conservative-to-aggressive slider in "What users need"; source research and
  writing done with an AI assistant (Claude) in October 2026.
- **How it was produced:** web searches of public sources (tool makers' public data sheets and
  formula pages, standards bodies' catalogue entries, CAM vendors' public pages, published papers
  and trade press). No commercial CAD/CAM program was driven or scripted, no GPL/LGPL/AGPL source
  was read, and no text is copied beyond single-sentence quotes. Standards are described from
  their public catalogue entries, not their paid text.
- **Units:** millimetres, newtons, watts or kilowatts, radians, unless a source is quoted in other
  units. Vendor formulas often use m/min, degrees and kW; these are kept where quoted.
- **Marking:** "(unverified)" marks a claim not confirmed against a source. "(derived)" marks a
  formula worked out here from geometry, not quoted. **Opinion** is labelled.

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
- **How the author's shop sets up cutting data today:** CAM tool libraries are set up both by
  machine and by material, with each tool tracked by a barcoded tag ID (see the companion tool
  library file).

**This research is not complete.** It is a starting point: the author will keep researching
and adding to it, and others are encouraged to do the same, correct it and contribute their own
findings and shop experience.

## Summary

1. **Vendors publish cutting data as ranges, not physics.** A typical tool maker's data sheet,
   for example Haas Tooling's 5-flute end mill sheet, gives per ISO material group a cutting speed
   range (SFM), a chip load per diameter (IPT), and axial/radial depth limits as multiples of
   diameter, with a separate slotting row ("slotting feed is reduced by 20 %")
   ([Haas Tooling, 5-flute HSPM cutting data](https://www.haascnc.com/content/dam/haascnc/ecommerce-assets/speeds-and-feeds/5-Flute_HSPM_Cutting%20Data_Style%202.pdf),
   accessed 2026-10-26). These are correct only for the engagement they were written for.
2. **Five calculations turn those ranges into safe, fast feeds for any engagement:** chip
   thinning (keep the real chip thickness on target when engagement is light), specific cutting
   force (Kienzle), force/power/torque, tool deflection, and stability (chatter). A sixth, tool
   life (Taylor), sets the trade-off between speed and tool cost. All of them have closed-form
   textbook versions that SolidCraft can implement and test against analytic values.
3. **The parameters are known and mostly standardised.** Tool geometry has ISO 13399 symbols
   (`DC`, `NOF`, `FHA`, `GAMF`, `KAPR`, `RE`, `APMX`, `LU`, `RPMX`, `TQ`…)
   ([Mitsubishi Materials, ISO 13399 property list](https://www.mmc-carbide.com/us/technical_information/iso/iso13399_property),
   accessed 2026-10-26). Work materials are grouped by ISO 513 (P, M, K, N, S, H). What has **no
   common standard** is the cutting data itself (force coefficients, tool life constants,
   recommended ranges), which every vendor publishes in its own PDF, app or format.
4. **CAM uses the physics in two places:** when shaping the path (adaptive or constant-engagement
   clearing keeps radial engagement steady so force does not spike
   ([Autodesk Fusion blog, 2020-09-03](https://www.autodesk.com/products/fusion-360/blog/speed-up-your-milling-with-constant-cutter-forces/)))
   and after the path exists (a feed optimiser recomputes chip thickness and force along the path
   and sets the feed block by block, as VERICUT Force does
   ([VERICUT Force](https://vericut.com/products/force-optimization), accessed 2026-10-26)).
5. **A conservative-to-aggressive slider** (Jason's idea) maps cleanly onto the limits these
   calculations check: one slider moves the safety factors on force, power, deflection, chip
   thickness, stability margin and target tool life together, while hard limits (maximum RPM,
   balance rating, finishing tolerance) never move. A proposed mapping is in "What users need".

## Fundamentals

### 1. Kinematics: speed and feed

Standard milling formulas
([Sandvik Coromant, Formulas and definitions for milling, metric](https://cdn.sandvik.coromant.com/files/sitecollectiondocuments/services/metal-cutting-e-learning/formulas-and-definitions/formulas-and-deinitions-for-milling-metric-enu.pdf),
accessed 2026-10-26):

| Quantity | Formula | Units |
|---|---|---|
| Cutting speed | `vc = π · Dcap · n / 1000` | m/min |
| Spindle speed | `n = vc · 1000 / (π · Dcap)` | rev/min |
| Table feed | `vf = fz · n · zc` | mm/min |
| Metal removal rate | `Q = ap · ae · vf / 1000` | cm³/min |
| Net power | `Pc = ae · ap · vf · kc / (60 · 10⁶)` | kW |
| Torque | `Mc = Pc · 30 · 10³ / (π · n)` | N·m |

`Dcap` is the cutting diameter at the depth of cut `ap`, `zc` the number of effective teeth,
`ae` the radial engagement, `kc` the specific cutting force (section 3).

**Ball and bull-nose tools cut at a smaller diameter than their nominal one** at shallow depth.
For a ball of diameter `D` at axial depth `ap < D/2`: `Dcap = 2 · √(ap · (D − ap))` (derived,
circle geometry). Example: a 10 mm ball at 0.5 mm depth has `Dcap ≈ 4.36 mm`, so for the same
`vc` the spindle must turn about 2.3 times faster than a calculation with `D` would give. A
library must store the corner/ball radius `RE` for this.

### 2. Chip thickness and chip thinning

The vendor's chip load is a **target maximum chip thickness** `hex`, not a feed per tooth. When
radial engagement is less than half the diameter, each tooth cuts a thinner chip than `fz`, so
the feed must rise to keep `hex` on target. Formulas
([Machining Doctor, chip thinning calculator](https://www.machiningdoctor.com/calculators/chip-thinning-calculator/),
accessed 2026-10-26):

- **Radial chip thinning:** `RCTF = 1 / √(1 − (1 − 2·ae/D)²)`, applied when `ae < D/2`;
  `fz = RCTF · hex`.
- **Lead-angle (axial) chip thinning** for chamfer mills and high-feed cutters:
  `ACTF = 1 / sin(κr)`, with `κr` the entering angle (ISO 13399 `KAPR`). No thinning at 90°.
- **Ball nose at shallow depth:** `ACTF = D / √(D² − (D − 2·ap)²)`.
- **Round inserts:** `ACTF = 1 / √(4·ap/IC − (2·ap/IC)²)`, `IC` the inscribed circle.
- Combined: `fz = hex · RCTF · ACTF`.

**Average chip thickness** `hm` is what the force model uses. For a straight-flute cutter
entering at the start of a cut and leaving at engagement angle `φe`, the instantaneous chip is
`h(φ) = fz · sin φ · sin κr`, so (derived):

```text
φe = arccos(1 − 2·ae/D)            (radians)
hm = fz · sin κr · (1 − cos φe) / φe = fz · sin κr · (2·ae/D) / φe
```

Seco frames average chip thickness as the key quantity for milling force and tool life
([Seco Tools, "The role of average chip thickness in milling operations", Patrick de Vos](https://www.secotools.com/article/the_role_of_average_chip_thickness_in_milling_oper_22d4386),
undated; article body not loaded, so only its premise is cited).

**Why it matters for CAM:** adaptive clearing at 10 % stepover with an un-thinned chip load
runs the tool at 60 % of its intended chip thickness, which rubs instead of cutting and shortens
tool life (direction from the sources above; the 60 % follows from the RCTF formula at
`ae/D = 0.1`).

### 3. Specific cutting force (Kienzle with corrections)

The force needed per square millimetre of chip falls as the chip gets thicker. Common working
form ([Machining Doctor, specific cutting force](https://www.machiningdoctor.com/glossary/specific-cutting-force-kc-kc1),
accessed 2026-10-26):

```text
kc = kc1.1 · hm^(−mc) · (1 − 0.01 · γ0)
```

`kc1.1` is `kc` at 1 mm chip thickness and 0° rake (N/mm²), `mc` the slope, `γ0` the effective
rake angle in degrees (ISO 13399 `GAMF`/`GAMP` for end mills); the same page suggests +7° when
the rake is unknown and gives `kc1.1` from about 700 to 4000 N/mm² across material groups.
Example values per group are in the companion file's table. Vendors also publish wear
corrections (a worn edge raises force) (unverified for a specific factor).

**Tangential force and the rest.** `Fc = kc · b · h`. The radial force (which bends the tool)
and the axial force (which pulls it from the holder) are not given by Kienzle alone; the
mechanistic model below gives them.

### 4. Mechanistic (edge-force) model

The model used in research and in force-simulation software (Altintas, *Manufacturing
Automation*, 2nd ed., Cambridge University Press, 2012; chapter 2, unverified chapter number):
for each tooth in cut, at each small axial slice `dz`,

```text
dFt = (Ktc · h(φ) + Kte) · dz      tangential
dFr = (Krc · h(φ) + Kre) · dz      radial
dFa = (Kac · h(φ) + Kae) · dz      axial
h(φ) = fz · sin φ                   (plus runout terms if known)
```

summed over teeth and slices, with each slice's angle lagged by the helix:
`ψ(z) = 2 · z · tan(β) / D` (β = helix angle, ISO 13399 `FHA`). The six coefficients are fitted
from slotting tests at several feeds per tooth (linear regression of average force against
`fz`), and vary with feed, immersion and speed, so one set is an approximation
([Rubeo & Schmitz, Precision Engineering, 2016](https://mtrc.utk.edu/wp-content/uploads/sites/45/2019/09/mechanistic_force_model_comparison.pdf)).

**What it buys over Kienzle:** the direction and the time history of the force, not just its
average. That gives the radial force for deflection, the peak (not average) load on a tooth, and
the input to the stability calculation. **What it costs:** six coefficients per material *and*
tool family, which almost no vendor publishes. Opinion: SolidCraft should start with Kienzle
(widely tabulated) and accept edge-force coefficients when a user or shared dataset has them.

### 5. Force, power and torque limits: worked example

12 mm, 4-flute carbide end mill, low-alloy steel (`kc1.1` 1700 N/mm², `mc` 0.25, rake 7°),
`ap` 18 mm, `ae` 1.2 mm (10 %), `vc` 150 m/min, target `hex` 0.05 mm (all derived with the
formulas above; inputs are illustrative, not a recommendation):

| Step | Result |
|---|---|
| RCTF at `ae/D = 0.1` | 1.67, so `fz` = 0.083 mm |
| `n`, `vf` | 3980 rev/min, 1326 mm/min |
| `φe`, `hm` | 36.9°, 0.026 mm |
| `kc` | ≈ 3940 N/mm² |
| `Pc`, `Mc` | ≈ 1.9 kW, ≈ 4.5 N·m |
| `Q` | ≈ 28.6 cm³/min |
| Average tangential force at the edge (`Mc / (D/2)`) | ≈ 750 N |

Each result is checked against a limit: `Pc` and `Mc` against the spindle's **power and torque
curves at that speed**, read from the machine library
([cam-research-nsnlookupjn-machine-library.md](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-machine-library/research/cam-research-nsnlookupjn-machine-library.md)),
never against a single rated value, and in the active gear range where the spindle has one; any speed change (for chatter or the slider) is re-checked against the curve. Below the spindle's base speed the available power falls
with RPM, so a cut that passes on rated power can stall at low speed (opinion, general spindle
behaviour; curve shapes are in the machine library). `Mc` is also checked against the tool's
and the holder's rated torque (`TQ` in ISO 13399), and force against the tool's breakage
strength and the holder's pull-out resistance. None of these limits is in a typical CAM tool
entry today (opinion, from the vendor and CAM pages cited).

### 6. Tool deflection

The tool is a cantilever: tip deflection `δ = F · L³ / (3 · E · I)` with `I = π · d⁴ / 64`.
Deflection grows with the cube of stick-out and falls with the fourth power of diameter; in the
fluted part the load-bearing diameter is the **core diameter**, not the cutting diameter, so a
12 mm tool with a 10 mm core deflects about 85 % more than the cutting diameter predicts.
Carbide `E` is about 600 to 650 GPa, HSS about 200 to 220 GPa, so HSS deflects about three
times more ([Machining Doctor, end mill deflection](https://www.machiningdoctor.com/classroom/endmiils-deflection/),
accessed 2026-10-26).

Worked (derived): the example above with 750 N radial-ish load at 36 mm stick-out deflects about
0.019 mm with a solid 12 mm section and about 0.047 mm with a 9.6 mm effective core (assuming
the whole stick-out is fluted). In practice the radial force is a fraction of the tangential one
and the shank is solid, so a real tool lies between those figures; the point is that the flute
geometry (`LU`, `LF`, core or neck diameter `DN`) and the stick-out must be in the library for
any deflection check. Holder and spindle compliance add to this and are not captured by the
beam model (see Stability).

### 7. Runout

Runout makes one tooth cut a thicker chip than the others. When total indicated runout is a
large fraction of `fz` (small tools, finishing passes), one tooth can carry most of the load and
wear or break first. Trade press reports that "a seemingly small amount of runout can still be
too large for the tool" and that shops commonly accept about 0.0005 in (0.013 mm)
([Modern Machine Shop, "The real cost of runout", Peter Zelinski, 2006-06-14](https://www.mmsonline.com/articles/the-real-cost-of-runout)).
For the library: store measured runout per assembly; the force model can add it to `h(φ)` per
tooth (Altintas, as above, unverified section).

### 8. Stability (chatter)

Covered in the companion file (Altintas & Budak lobes, receptance coupling). The data the
library must hold for it: per assembly, the tool-tip **frequency response** (or its modal
parameters: natural frequency, stiffness, damping ratio per direction), plus the radial and
tangential force coefficients from section 4. Without measured dynamics, CAM can only apply rules
of thumb (stick-out ratio, depth limits from the vendor sheet).

### 9. Tool life

Taylor's equation `vc · T^n = C` and its extended form with feed and depth terms
(`vc · T^n · fz^a · ap^b = C`, unverified exponent naming varies by author) relate cutting speed
to minutes of cutting before a wear limit. Tool life tests are standardised: ISO 8688-2:1989
covers tool life testing in end milling, with flank wear `VB` as the usual end-of-life criterion
([SCC catalogue entry, ISO 8688-2:1989](https://ccn-scc.ca/standardsdb/standards/8107176);
wear criterion details unverified). With `n` and `C` per tool family and material, CAM can turn
"I want this tool to last one shift" into a cutting speed, and predict tool changes per job.

### 10. Balance and maximum speed

Holder balance grades come from ISO 1940-1 (now ISO 21940-11) and ISO 16084 for rotating tools;
makers typically balance holders "to G2.5 at 25,000 rpm" (Schunk, as quoted by
[MSC, "Toolholder balancing: G2.5 vs. G6.3"](https://www.mscdirect.com/knowledge-center/articles/toolholder-balancing-manufacturing-g25-vs-g63)).
The speed at which an assembly is rated, the tool's own `RPMX`, and the machine's maximum
spindle speed are **hard limits**: no optimiser or slider should exceed the lowest of them.

## Standards and references

| Standard / reference | What it gives a tool library | Access |
|---|---|---|
| **ISO 513** (2012) | Work material groups P (steel), M (stainless), K (cast iron), N (non-ferrous), S (heat-resistant and titanium), H (hardened); colour codes blue, yellow, red, green, orange, grey (colours unverified against the standard text); application ranges for cutting materials. Six main groups since about 2003 ([Mitsubishi Materials, classification](https://www.mmc-carbide.com/permanent/courses/78/classification.html)) | Paid ([SCC entry](https://scc-ccn.ca/standardsdb/standards/8152214)) |
| **VDI 3323** | Finer material subgroups used by many vendors (P1…P6, M1…M3, etc.) (unverified attribution of the exact subgroups to VDI 3323; Haas sheets use P0…P6, M1…M3, K1…K3, S1…S4, H1…H4 with tensile strength and hardness per row) | Paid |
| **ISO 13399** (2006 onward) | Vendor-neutral tool and holder data: property symbols, classes, 3D model rules; developed under ISO TC 29 with Sandvik Coromant, KTH and Kennametal among contributors ([Wikipedia: ISO 13399](https://en.wikipedia.org/wiki/ISO_13399)) | Paid; property lists public via vendors |
| **GTC** (Generic Tool Catalog) | Packaging of ISO 13399 data with a common class hierarchy, pictures and 3D models; Sandvik's CoroPlus Tool Library exports GTC packages ([Sandvik support: export GTC package](https://www.sandvik.coromant.com/en-us/support/coroplus-tool-library-support/export-gtc-package)) | Free packages from vendors |
| **DIN 4000** | German property lists (Sachmerkmal-Leisten) for tools and other parts; used for tool data alongside ISO 13399 ([DIN 4000-170 entry](https://din.de/en/wdc-beuth:din21:209462628)); exact parts for cutting tools unverified | Paid |
| **ISO 3002-1** | Basic quantities in cutting: definitions of rake, clearance, entering angles (unverified edition) | Paid |
| **ISO 1832** | Designation code for indexable inserts (shape, clearance, tolerance, size) (unverified edition) | Paid |
| **ISO 8688-2, ISO 3685** | Tool life testing for end milling and single-point turning | Paid |
| **ISO 1940-1 / ISO 21940-11, ISO 16084** | Balance quality; balancing of rotating tools | Paid |
| **Holder interfaces** | ISO 7388 (steep taper), ISO 12164 (HSK), ISO 26623 (polygon), ASME B5.50 (CAT); DIN 69873 data-carrier bore (unverified editions) | Paid |
| **ISO 14649-111 (STEP-NC)** | Tool description for milling in STEP-NC (unverified scope) | Paid |
| **MTConnect** | Cutting tool asset model; MachiningCloud delivers vendor speed and feed recommendations in MTConnect format ([MachiningCloud GTC white paper, 2016](https://files.solidworks.com/partners/pdfs/13987-5267-GTC_White_Paper_MachiningCloud.pdf)) | Free |
| **Altintas, *Manufacturing Automation*** | Textbook derivation of milling forces, stability, deflection | Book |
| **Kienzle (1952)** | Origin of `kc1.1`/`mc` | Paper |

**ISO 13399 symbols a CAM library needs** (from the Mitsubishi list cited above): `DC` cutting
diameter, `DCX`/`DCN` max/min cutting diameter, `DMM` shank diameter, `DCON` connection
diameter, `LU` usable length, `LF` functional length, `OAL` overall length, `LH` head length,
`APMX` maximum depth of cut, `NOF` flute count, `ZEFP`/`ZEFF` effective peripheral/face edges,
`RE` corner radius, `CHW` chamfer width, `KAPR` cutting edge angle, `PSIR` lead angle, `FHA` helix
angle, `GAMF`/`GAMP` radial/axial rake, `RPMX` maximum speed, `TQ` torque, `WT` weight; for
holders `BD`, `DN` neck diameter, `LPR` protruding length, `LSC` clamping length, `CCMS` machine
side connection code, coolant fields `CNSC`, `CXD`, `CP`.

## Accuracy

- **Kienzle coefficients are averages.** Tabulated `kc1.1`/`mc` are per material group, not per
  heat or hardness, and are normalised to a reference rake. Expect force predictions within tens
  of percent, not a few percent (unverified magnitude; Rubeo & Schmitz show coefficients shift
  with cutting conditions).
- **Thin chips are the weak spot.** Power-law `kc` rises without bound as `hm → 0`, while real
  edges have a minimum chip thickness below which they rub (edge-radius size effect)
  (unverified threshold; depends on edge preparation). The edge terms in the mechanistic model
  capture part of this.
- **Deflection from the beam formula is a lower bound**: holder and spindle compliance add to it.
  Receptance coupling (companion file) is how research handles the whole stack.
- **Vendor ranges are starting points**: Haas notes that its parameters assume ideal conditions,
  that lower speeds suit high stock removal or harder material, and that tools over 1/2 in on
  smaller taper machines need adjustment (5-flute sheet cited above).
- **Testing in SolidCraft:** every formula here has an analytic answer (chip thinning, `hm`,
  `Dcap`, Kienzle, power, torque, beam deflection), so `tests/oracle` cases can check them
  without any commercial program (opinion).

## Programs

How CAM and related tools use the physics, from public pages:

- **Path shaping, adaptive / constant-engagement clearing.** Fusion's adaptive clearing keeps
  "a constant radial tool engagement throughout the entire cut", which "eliminates spikes in the
  cutting forces", and uses chip thinning to "increase the cutter feed rate to maintain the chip
  thickness" ([Autodesk Fusion blog, 2020-09-03](https://www.autodesk.com/products/fusion-360/blog/speed-up-your-milling-with-constant-cutter-forces/)).
  Here physics sets the stepover and feed for a whole operation; the path generator's job is to
  hold engagement at the chosen value.
- **Feed optimisation after the path, VERICUT Force.** Reads the NC program, computes tool and
  stock contact block by block, "performs block break-up to remove material using a constant
  ideal chip thickness", limits "excessive cutting forces and spindle power requirements", and
  reports chip thickness, torque, power, feed per tooth and deflection per block; it writes a
  separate optimised file ([VERICUT Force](https://vericut.com/products/force-optimization),
  accessed 2026-10-26). Its material data comes from dynamometer tests at Okuma's technical
  centre, and Sandvik end mill data can be downloaded from CoroPlus into it
  ([VERICUT news, CGTech, Okuma and Sandvik Coromant](https://archive.vericut.com/358-okuma-sandvik/),
  undated).
- **Vendor data apps.** Kennametal NOVO (2013) covers tool assemblies, presetting data and speeds
  and feeds, exports ISO 13399, and feeds ESPRIT CAM; NOVO Optimize recommends speed and feed
  changes from shop-floor performance via MTConnect
  ([Cutting Tool Engineering, 2014-09-01](https://www.ctemag.com/products/novosphere-cloud-enabled-digital-tooling-data)).
  Sandvik CoroPlus Tool Library exports GTC (cited above).
- **Vendor PDFs.** Haas Tooling publishes cutting data per tool series as PDFs: material group
  rows, SFM ranges, chip load per diameter, `ap`/`ae` as multiples of D, separate side milling and
  slotting (5-flute sheet cited above); drill heads get SFM min/start/max and feed per revolution
  by diameter band ([Haas modular drill heads data](https://www.haascnc.com/content/dam/haascnc/ecommerce-assets/speeds-and-feeds/TDM1.pdf)).
  Product pages list `DC`, shank, LOC, OAL, flutes, corner radius, helix, coating, variable pitch,
  centre cutting and primary/secondary ISO groups
  ([Haas Tooling 03-1059](https://www.haastooling.com/p/03-1059)).
- **Calculators.** Machining Doctor publishes chip thinning, `kc` and deflection calculators
  (cited above). Other commercial calculators exist (unverified feature lists; not reviewed).

## Complaints and requests

From the sources above (facts) and their implications (opinion):

- **Cutting data is not machine readable.** Haas's cutting data is PDF; its 16 mm HSPM2 end mill
  page links a data sheet whose filename names a different series ("htpm (03-1040 to 03-1069)")
  (observed on the product page above). A CAM program cannot import this without a human.
- **Presets ignore engagement.** A preset written for slotting is wrong for 10 % stepover and the
  reverse; the Haas sheet itself needs two rows for side milling and slotting.
- **Units and conventions mix:** SFM and IPT vs m/min and mm; chip load meaning `hex` in one source
  and `fz` in another; rake angle conventions.
- **Force coefficients are scarce.** Kienzle values are tabulated per material group by some
  vendors; edge-force coefficients mostly come from the user's own tests or from commercial
  optimisers' own material libraries (VERICUT's dynamometer work above).
- **Limits are missing:** tool torque, holder torque, machine power curve and balance speed are
  rarely in the CAM tool entry, so the program cannot warn that a cut stalls the spindle or pulls
  the tool.

## What users need

### Calculation pipeline in SolidCraft (opinion)

```text
tool + holder + assembly (ISO 13399 fields, stick-out, runout)   ─┐
work material (ISO 513 group, hardness, kc1.1/mc or edge coeffs) ─┼─> per operation:
machine (power/torque curve, RPM, feed limits)                   ─┤    1. vc, hex from vendor range at slider position
workholding limits (companion file)                              ─┤    2. n from Dcap; fz from chip thinning
risk slider (below)                                              ─┘    3. hm, kc, Fc, Pc, Mc
                                                                       4. deflection, stability margin, tool life
                                                                       5. lower fz / ae / ap until every limit holds:
                                                                          min(tool, holder, spindle, machine, workholding)
                                                                     per path segment (later):
                                                                       6. re-run 2–5 with actual engagement, set feed
```

Step 6 is the VERICUT-style optimiser and needs the stock model; steps 1 to 5 work per
operation with only the library data and are a good first milestone.

### The conservative-to-aggressive slider (Jason's idea)

**The idea:** one slider per job (or per operation) from "Conservative: I don't trust this" to
"Aggressive: I don't care if I break a tool or two". The slider does not change the physics; it
changes how close to each limit the calculation is allowed to go.

Proposed mapping (opinion; every number is a starting point for discussion):

| Limit | Conservative (0) | Balanced (0.5) | Aggressive (1) |
|---|---|---|---|
| Position in vendor `vc` range | low end | middle | high end |
| Position in vendor chip load range (`hex`) | low end | middle | high end |
| Force vs. tool / holder rating | ≤ 50 % | ≤ 70 % | ≤ 90 % |
| Spindle power and torque, from the curves at the operating RPM | ≤ 60 % | ≤ 80 % | ≤ 95 % |
| Roughing deflection budget | ≤ 0.01 mm or 20 % of stock to leave | ≤ 40 % of stock | ≤ 70 % of stock |
| Depth vs. predicted stability limit (only with measured dynamics) | ≤ 50 % | ≤ 70 % | ≤ 90 % |
| Target tool life (Taylor) | e.g. 60 min | e.g. 30 min | e.g. 15 min |
| Workholding safety factor, friction-held (from the workholding file) | ≈ 3.0 | ≈ 2.5 | ≈ 2.0, never below 1.5 |

**Why this matters most for novices (Jason's observation):** a novice CAM programmer has little
idea of the limits of a tool or of the workholding. An experienced programmer carries those
limits in their head and edits presets to suit; a novice copies the preset and finds the limit by
breaking a tool or moving the part. The physics checks put that experience into the program.
Opinion, following from this: new users should start at the conservative end (or Balanced at
most), every result should name the limit that set the feed in plain words ("the vise would let
the part slip above this feed"), and moving toward Aggressive should be a deliberate choice, not
the default.

**Never moved by the slider (hard limits):** the lowest of tool `RPMX`, holder balance speed and
machine maximum speed; machine feed and power maxima; the finishing tolerance (finishing
deflection must stay inside it); minimum chip thickness (going thinner rubs and wears tools,
which is not "aggressive", only wasteful); collision and holder clearance.

**Unknown data pulls toward conservative.** "I don't trust this" has two meanings: distrust of
the data, and unwillingness to risk the tool. Opinion: keep one slider for the user, but when an
input is missing or estimated (no `kc1.1` for this material, no measured runout, no dynamics) the
program should show that, and cap that limit at the conservative setting until real data
exists. Each result should say which limit set the feed ("limited by spindle power at 3980 rpm"),
so the user learns why a cut is slow.

**Feedback (later):** spindle load from the machine (MTConnect or the controller's load meter)
compared with predicted power would show whether the shop can move the slider, and could refine
`kc1.1` for that material over time.

### Manual feeds and speeds: sanity check before posting (Jason's idea)

*"If manual feeds and speeds are entered by the operator, do a sanity check through physics so
that you don't run a tool at too high an RPM or feed too fast into the work. There is a sane
limit to everything and it should be caught before posting code."*

Opinion: the same checks that pick feeds automatically also **check** feeds a person typed. They
run on every operation when it is edited and again, block by block, on the final program just
before the post processor writes it, so no number reaches the machine unchecked. Results come in
two levels:

| Level | Meaning | Examples |
|---|---|---|
| **Error: blocks posting** | Physically unsafe or impossible; fix the value (or the tool data, if that is what is wrong) | RPM above the lowest of tool `RPMX`, holder balance speed and machine maximum; feed above the machine's maximum; plunging straight down with a tool that is not centre-cutting (Haas product pages list "center cutting" per tool); torque or power above the spindle curve at that RPM; force above the tool's or the workholding's limit |
| **Warning: post allowed after acknowledgement** | Outside the sane range, but not certain to fail | `vc` outside the vendor or generic range; chip load per tooth (after chip thinning) above the vendor maximum, or below the minimum chip thickness (rubbing); plunge or ramp feed above the vendor's plunge rate; deflection larger than the finishing tolerance; recut risk (coolant section) |

Each finding names the operation, the program block, the limit and the value that would pass
("Block N1240: 9000 rpm exceeds the holder's G2.5 rating of 8000 rpm; 8000 rpm or less passes").

Warnings are where the slider applies: at Conservative a warning needs an explicit override, at
Aggressive it is shown but does not stop the post. Errors are hard limits and ignore the slider.
An override is logged with the operator's name and reason, so a later broken tool can be traced
back (opinion).

Two common mistakes the check should catch specifically (opinion): **units**, a feed entered in
in/min where mm/min was meant (or the reverse) is off by a factor of 25.4, which the chip load
check exposes at once; and **feed per tooth vs feed per revolution**, off by the flute count.

### Tools with unknown physics (Jason's question)

*"What if a tool is unknown for its physics? Can we have standard physics for like a 1/2" HSS end
mill, or can the system handle a non-physics defined tool?"*

**Most of the physics belongs to the work material, not the tool.** Cutting force comes from
`kc1.1`/`mc` of the material (section 3); the tool only contributes geometry (diameter, flutes,
rake, helix, lengths), its substrate's stiffness and its limits. So a tool with no vendor data
still gets usable physics once its basic geometry is known (opinion, following from the formulas
above).

Proposed fallback tiers (opinion):

| Tier | What is known | Where the rest comes from |
|---|---|---|
| 1. Vendor | Catalogue entry (GTC/ISO 13399) and vendor cutting data | Vendor |
| 2. Generic class | Tool type (square/ball/bull end mill, drill…), substrate (HSS, cobalt HSS, carbide), coating, flute count, diameter | A built-in or shared **generic class** with typical rake, helix, core-diameter ratio, `E` (HSS ≈ 200–220 GPa, carbide ≈ 600–650 GPa per section 6) and a typical `vc`/chip load range per ISO material group from public handbook tables (to be sourced; unverified values not given here) |
| 3. Measured geometry only | The user's own diameter, flutes, stick-out, maybe a photo | Tier 2 defaults for everything else |
| 4. Nothing | A tool number and a diameter | Physics checks run with tier 2 defaults for "end mill, HSS, 2 flutes" and say so, or the user types feeds by hand and the program only checks hard limits |

So a "1/2 in HSS 4-flute end mill" is a valid tool on its own: tier 2 fills in everything else.

**Every value carries a confidence label:** vendor, measured, generic or assumed. Results show
which labels they depend on ("feed limited by deflection; core diameter assumed").

**Unknown tools are derated automatically.** Any limit that depends on a generic or assumed value
is capped at the conservative end of the slider, whatever the user chose, until better data
exists. This fits the novice point above: a tool nobody has described is exactly where a novice
gets hurt.

**Learning the tool over time:** spindle load during the cut (from the controller or a monitor)
compared with predicted power refines the force estimate; measured deflection, surface finish and
part inspection results refine stiffness and chip load; tool life records refine Taylor
constants. Each refinement moves the field from "generic" to "measured" and lifts the derating
for that tool (opinion; the data path is open question 3 and the machine library file).

**Where generic classes live:** they are natural shared entries in the cloud catalogue proposed in
[the tool library file](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-tool-library/research/cam-research-nsnlookupjn-tool-library.md):
a "generic HSS square end mill" class next to vendor part numbers, which a shop's own tool can
reference and then override field by field.

### Coolant, air blast and chip clearing fans (Jason's questions)

*"Does coolant type, refraction or temp affect tool physics?"* and *"Use of chip clearing fans."*

**Short answer:** coolant changes tool **life**, edge damage and part **size** much more than it
changes cutting **force**. So the force, power and deflection checks above barely move, but the
tool life constants, the choice of cutting data and the risk of chip recutting all depend on how
the cut is cooled and cleared (opinion, from the sources below).

| Effect | What happens | Source / status |
|---|---|---|
| Cutting force | Lubrication lowers friction between chip and rake face, so `kc` drops somewhat; tabulated `kc1.1` values rarely state the coolant condition | Direction general; magnitude unverified |
| Tool life | Taylor `n`, `C` differ for dry, air, MQL, flood and high-pressure through-tool coolant; vendor ranges often assume one mode | Store life constants and cutting data **per coolant mode** (opinion) |
| Thermal cracking (carbide) | ISCAR lists the causes as "excessive variations in surface temperature, intermittent machining, or variations in coolant supply", and the remedy as "significantly increase coolant supply or turn it off" | [ISCAR, milling inserts wear problems and solutions](https://www.iscar.com/products.aspx/countryid/1/productid/12356) |
| Built-up edge | Also from ISCAR: a cutting zone that is too cool is a cause of built-up edge | Same source |
| Coolant concentration (refractometer) | Too lean shortens tool life and brings corrosion; too rich brings foaming and residue; measured with a refractometer times the fluid's factor | Covered in the [machine health research](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-9w3gq9/research/cam-research-nsnlookupjn-machine-health.md), "Coolant condition" |
| Coolant temperature | Mainly moves part and fixture size (thermal growth), not tool force | Machine health research (same section) |
| Chip recutting | Chips left in the cut are recut, "a primary contributor to early onset tool wear"; through-tool air blast clears them, "particularly useful for deep cavity machining" | [ETMM / Haas Automation Europe, 2015-10-20](https://www.etmm-online.com/optimum-cooling-for-a-better-finish-longer-tool-life-a-508424/?p=2) |

**Chip clearing fans and air blast (opinion):** a fan or air nozzle clears chips from open
surfaces and keeps the cut dry and visible; it cools little and does not reach the bottom of deep
pockets or slots. Through-tool air or coolant does. So the program should treat "deep pocket or
slot with only a fan" as a recutting risk: a warning, a lower tool life estimate, or a preference
for toolpaths that leave an open side for chips. This is a separate flag, not a slider factor,
because no safety margin makes recut chips harmless.

**What the library needs for this (opinion):**

- **Tool:** through-coolant capable (ISO 13399 holder coolant fields `CNSC`, `CXD`, `CP`; Haas
  product pages list "through-spindle coolant"), and the coolant modes the vendor allows (some
  carbide grades are meant to run dry in steel).
- **Machine** (machine library): coolant modes fitted (flood, through-spindle and its pressure,
  air blast, MQL, fan), coolant type in the tank.
- **Operation:** the coolant mode chosen, which selects the cutting data row and the tool life
  constants, and warns about thermal cracking when a carbide tool in an interrupted cut gets
  partial coolant.
- **Shop state** (machine health): concentration and temperature in range or not; out of range
  shortens the expected tool life.

### Outputs for workholding (shared with the workholding file)

The workholding research uses these as inputs, per toolpath move or segment (opinion; agreed
names between the two files):

| Output | Meaning | Units |
|---|---|---|
| `Ft`, `Fr`, `Fa` | tangential, radial, axial force in the tool frame, peak and mean over a revolution (section 4; Kienzle gives `Ft` only) | N |
| `Fx`, `Fy`, `Fz` | resultant force vector in part/machine coordinates | N |
| `Mc`, `Pc` | spindle torque and power | N·m, W |
| `f_tp = n · z / 60` | tooth-passing frequency, for vibration of part and fixture | Hz |
| contact point | where the tool meets the stock, so moments about the fixture can be computed | mm |

It returns per setup: the maximum allowed force in each direction and moment before slip, lift
or tip (after its safety factor), the lowest natural frequency of part plus fixture, and part
wall deflection limits. The feed calculation treats these as one more limit.

The two sides negotiate per force vector (Jason's idea, detailed in the workholding file): CAM
recommends a vise torque window whose low end is the least clamp force that passes the slip
checks for the cutting forces above, and whose high end is set by part deformation, jaw marks or
the vise maker's limit. If the window is empty, the feed calculation lowers the load, or a
different hold is suggested. Assumed values (estimated `Fr`/`Fa`, default friction, unknown
dynamics) are labelled as assumed wherever a result depends on them.

### Tool library parameter schema (opinion; ISO 13399 symbols where one exists)

Which calculation needs each field: **S** speed/feed, **T** chip thinning, **F** force/power/
torque, **D** deflection, **C** chatter, **L** tool life, **H** hard limit.

| Group | Field | Used by |
|---|---|---|
| Cutter geometry | `DC`, `DCX`, `NOF`, `ZEFP`/`ZEFF` | S T F D |
| | `RE` (corner/ball radius), `CHW`, `KAPR`, `PSIR` | S T F |
| | `FHA` helix, variable pitch/helix flag | F C |
| | `GAMF`, `GAMP` rake angles | F |
| | `APMX`, `LU`, `LF`, `OAL`, `DMM`, `DN` neck, core diameter (no symbol found) | D H |
| Cutter material | substrate (carbide/HSS/ceramic…), `E` modulus, coating | D L |
| Cutter limits | `RPMX`, `TQ`, max `ap`/`ae` per strategy | H |
| Holder | `CCMS`, `BD`, `DN`, `LPR`, `LSC`, clamping type, `TQ`, balance grade + rated RPM, coolant (`CNSC`, `CP`) | D C H |
| Assembly | stick-out, measured runout, measured length/diameter, tool-tip FRF or modal parameters | D C T |
| Work material | ISO 513 group + subgroup, hardness, tensile strength, `kc1.1`, `mc`, reference rake, edge-force coefficients (optional), source | F |
| Cutting data (per tool × material × strategy) | `vc` min/start/max, `hex` per diameter, `ap`/`ae` limits, slotting factor, coolant, source, date | S T |
| Tool life (per tool family × material) | Taylor `n`, `C` (and feed/depth exponents), wear criterion | L |
| Machine (detailed in the machine library file) | spindle power and torque curves vs RPM (checked at the operating RPM, not a single rated value), max RPM, max feed, taper/interface | F H |

SolidCraft already has **physical material cards** with SI properties, sources and conditions
(`docs/physical-material-cards.md`: density, elasticity, strengths, thermal). Opinion: a
"machinability" block on the same card (ISO 513 group, hardness, `kc1.1`, `mc`, reference rake,
source) would let the CAM work reuse the material a design already assigns, instead of a separate
material list in the tool library.

### Where time is lost

Converting vendor PDFs into CAM presets by hand; re-doing presets for each engagement; trial cuts
to find a feed that does not chatter; broken tools when a preset meets a corner it was not written
for.

**From the author's shop (personal experience, not a cited source):**

- **Broken tools and guesswork cost the most:** the biggest losses are broken tools and the time
  spent working out the best speed and feed for a given tool in a given material.
- **Short overload spikes are common and avoidable:** tight corners and lead-ins overload the
  tool for an instant, caused by incomplete simulation and poor toolpaths. The computer has the
  geometry and should know the physics, so these spikes should not happen.
- **Under-loaded tools waste time:** low tool load when the cut could be deeper and faster, and
  time spent cutting air. Every second counts; the goal is the most out of both the machine and
  the tool.
- **Tools are cheap compared with machine time,** so load them to their full ability. In the
  author's experience machines do not wear faster under high steady load; what they hate is
  sudden, crash-type load.
- **The target:** every cut smooth, steady and loaded to the maximum.

What this means for SolidCraft (opinion): the default goal is **steady maximum load, not just a
safe average**. That needs three things from this file working together: constant-engagement
path shaping so corners and lead-ins do not spike (Programs); a per-segment check of peak force,
not only average force (sections 4 and 5, step 6 of the pipeline), so a momentary overload in a
corner is found before posting; and feed raised wherever the load is low, including rapid or
fast feed through air where the stock model shows no material. The slider then sets how close
to the maximum that steady load sits, and the hard limits stop the crash-type loads.

## Export control, sanctions and government use

*Added 2026-10-29 from the [government use research](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-qk5riz/research/gov-research-nsnlookupjn-government-use.md). Opinion and policy proposals, not legal advice.*

**What in this file crosses a border:** a cloud tool library parameter schema and shared vendor cutting data.

- **Hosted parts block sanctioned regions and screen people.** Any service this file proposes that the project would run blocks comprehensively sanctioned regions, plus Russia and Belarus (US, EU and UK rules restrict CAD/CAM software and related services there), and screens accounts, payers, contributors and marketplace parties against the US Consolidated Screening List and the UK, EU and UN lists.
- **No support there.** No support, consulting or custom work for users in sanctioned regions or for listed parties.
- **Government mode.** At the Controlled level (CUI and allied equivalents) online parts are off or point to the organisation's own mirror; at the Isolated level they arrive only as signed packs on media; a design flagged Export controlled (ITAR/EAR or national equivalent) never leaves through them.
- **Only generic data is shared.** Nothing derived from a controlled design (geometry, toolpaths, part or drawing numbers, test or inspection results) goes into a shared library, report or issue. Tool data stays shareable through the field allow-list in the high-security research.
- **Code stays public.** Every build the project ships is public, which keeps it "published" software outside the EAR. The rules are tracked by the standing export and sanctions watch.
- **Checked for controlled data (2026-10-29):** this file contains no controlled technical data. Its examples are public sources, generic values or made-up stand-ins, and the author's experience is stated in general terms.

## Glossary

- **`ae` / `ap`:** radial / axial depth of cut (mm).
- **`Dcap`:** effective cutting diameter at depth `ap`.
- **Chip load / `hex`:** maximum chip thickness per tooth that the vendor recommends.
- **Chip thinning:** the real chip is thinner than `fz` at light radial engagement or with a lead
  angle; the feed is raised to compensate.
- **`hm`:** average chip thickness over the engagement arc.
- **`kc`, `kc1.1`, `mc`:** specific cutting force, its value at 1 mm chip, and its slope (Kienzle).
- **Edge-force (mechanistic) coefficients:** `Ktc, Krc, Kac` (cutting) and `Kte, Kre, Kae` (edge).
- **Engagement angle `φe`:** the arc over which a tooth is in the material.
- **FRF:** frequency response function at the tool tip; input to chatter prediction.
- **Runout (TIR):** total indicated runout of the cutting edges as the tool turns.
- **Taylor tool life:** `vc · T^n = C`.
- **ISO 513 groups:** P steel, M stainless, K cast iron, N non-ferrous, S superalloys and
  titanium, H hardened materials.

## Open questions

1. Is there any open, redistributable set of `kc1.1`/`mc` values per material subgroup that
   SolidCraft may ship, or must users supply their own? Vendor tables are copyrighted documents
   even when the numbers are facts (unverified legal position; maintainers' call).
2. Should the first CAM milestone stop at per-operation limits (steps 1 to 5), leaving per-segment
   feed optimisation for later?
3. Which machine data can SolidCraft read for power and torque curves? Most controllers do not
   expose them; a user-entered curve may be the only source.
4. How should the slider interact with vendor ranges that are already conservative or already
   aggressive? Normalise per vendor, or trust the range as given?
5. Do the GTC packages from Sandvik, Kennametal or others include cutting data or only geometry?
   (The MachiningCloud paper suggests speed and feed data travels separately, in MTConnect.)
6. Is there a public specification for the "minimum chip thickness" per edge preparation, or must
   it be a per-tool user field?

## Sources

### Standards (catalogue entries only)

- ISO 513:2012 — [SCC catalogue](https://scc-ccn.ca/standardsdb/standards/8152214)
- ISO 13399 — [Wikipedia overview](https://en.wikipedia.org/wiki/ISO_13399); property symbols via [Mitsubishi Materials](https://www.mmc-carbide.com/us/technical_information/iso/iso13399_property)
- ISO 8688-2:1989 — [SCC catalogue](https://ccn-scc.ca/standardsdb/standards/8107176)
- DIN 4000-170 — [DIN catalogue](https://din.de/en/wdc-beuth:din21:209462628)
- ISO 513 groups — [Mitsubishi Materials, classification](https://www.mmc-carbide.com/permanent/courses/78/classification.html)

### Books and papers

- Altintas, Y., *Manufacturing Automation*, 2nd ed., Cambridge University Press, 2012.
- Kienzle, O., VDI-Z 94, 1952 (page numbers unverified).
- Rubeo, M. & Schmitz, T., mechanistic force model comparison, Precision Engineering, 2016 — [PDF](https://mtrc.utk.edu/wp-content/uploads/sites/45/2019/09/mechanistic_force_model_comparison.pdf)
- Taylor, F. W., "On the art of cutting metals", Trans. ASME 28, 1907.

### Vendor and program pages (accessed 2026-10-26)

- Sandvik Coromant, [Formulas and definitions for milling (metric)](https://cdn.sandvik.coromant.com/files/sitecollectiondocuments/services/metal-cutting-e-learning/formulas-and-definitions/formulas-and-deinitions-for-milling-metric-enu.pdf)
- Sandvik Coromant, [CoroPlus Tool Library: export GTC package](https://www.sandvik.coromant.com/en-us/support/coroplus-tool-library-support/export-gtc-package)
- Seco Tools, [The role of average chip thickness in milling operations](https://www.secotools.com/article/the_role_of_average_chip_thickness_in_milling_oper_22d4386)
- Haas Tooling, [5-flute HSPM cutting data](https://www.haascnc.com/content/dam/haascnc/ecommerce-assets/speeds-and-feeds/5-Flute_HSPM_Cutting%20Data_Style%202.pdf), [modular drill head data](https://www.haascnc.com/content/dam/haascnc/ecommerce-assets/speeds-and-feeds/TDM1.pdf), [product 03-1059](https://www.haastooling.com/p/03-1059)
- Machining Doctor, [chip thinning](https://www.machiningdoctor.com/calculators/chip-thinning-calculator/), [specific cutting force](https://www.machiningdoctor.com/glossary/specific-cutting-force-kc-kc1), [end mill deflection](https://www.machiningdoctor.com/classroom/endmiils-deflection/)
- VERICUT, [Force optimization](https://vericut.com/products/force-optimization); [CGTech, Okuma and Sandvik Coromant collaboration](https://archive.vericut.com/358-okuma-sandvik/)
- Autodesk, [Speed up your milling with constant cutter forces](https://www.autodesk.com/products/fusion-360/blog/speed-up-your-milling-with-constant-cutter-forces/) (2020-09-03)
- MachiningCloud, [GTC white paper](https://files.solidworks.com/partners/pdfs/13987-5267-GTC_White_Paper_MachiningCloud.pdf) (2016)

### Trade press

- Cutting Tool Engineering, [NOVOsphere cloud-enabled digital tooling data](https://www.ctemag.com/products/novosphere-cloud-enabled-digital-tooling-data) (2014-09-01)
- Modern Machine Shop, [The real cost of runout](https://www.mmsonline.com/articles/the-real-cost-of-runout) (2006-06-14)
- MSC Direct, [Toolholder balancing: G2.5 vs. G6.3](https://www.mscdirect.com/knowledge-center/articles/toolholder-balancing-manufacturing-g25-vs-g63)
