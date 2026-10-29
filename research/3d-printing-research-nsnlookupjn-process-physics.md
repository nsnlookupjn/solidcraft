# 3D printing research: process physics and a shared process-data library

Research for [#1626](https://github.com/tangency-labs/solidcraft/issues/1626) (Research Request
for Plugin: 3D Printing). This file covers one part of that request: **the physics that limits
how fast and how well an additive process can build, and the material, machine and process data
a program needs to reason about it.** It touches #1626's topics 8 (machine limits), 9 (FDM
calibration), 10 (resin accuracy), 15 (where compensation lives) and 16 (time estimates). Metal
powder bed fusion and directed energy deposition are included because the same data questions
arise there; their design rules belong to [#1625](https://github.com/tangency-labs/solidcraft/issues/1625)
(DfAM).

It is the additive companion to the CAM tool-library research for #1621
([cam-research-nsnlookupjn-tool-library.md](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-tool-library/research/cam-research-nsnlookupjn-tool-library.md)),
and asks the same question: *what is the additive equivalent of a physics-based tool library?*

Licence: MIT OR Apache-2.0, the same as SolidCraft (see the Licence section of CONTRIBUTING.md).

## About this research

- **Author:** Jason ([@nsnlookupjn](https://github.com/nsnlookupjn)). Idea and direction by Jason;
  source research and writing done with an AI assistant (Claude) in October 2026.
- **How it was produced:** web searches of public sources (standards bodies' catalogue pages,
  papers and their public abstracts, firmware and slicer *documentation*, vendor help pages, user
  forums). No commercial CAD/CAM or slicer program was driven or scripted. No GPL/LGPL/AGPL source
  code was read (Klipper and PrusaSlicer were consulted through their published documentation and
  help pages only, which #1626 allows). Standards are described from their public catalogue
  entries, not their paid text. No text is copied beyond single-sentence quotes.
- **Units:** millimetres, seconds, watts, joules, kelvin or degrees Celsius. Volumetric flow is
  given in mm³/s; energy density in J/mm³ (volumetric) or J/mm² (areal).
- **Marking:** "(unverified)" marks a claim not confirmed against a source. "(estimate)" marks an
  order-of-magnitude calculation made for this document from general physics, not taken from a
  source. **Opinion** is labelled where it mixes with facts.
- **Scope:** process physics and data. Slicing algorithms, G-code dialects and printer
  connections are other parts of #1626 and are only mentioned where they set a limit.

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

> **[TODO Jason: 3D printing experience]** Which printers and processes you have used (FDM,
> resin, metal), which slicers, and for how long. How you choose speeds and temperatures for a new
> filament today, what you calibrate, and what goes wrong.

**This research is not complete.** It is a starting point: the author will keep researching
and adding to it, and others are encouraged to do the same, correct it and contribute their own
findings and experience.

## Summary

1. **Yes, additive processes have the same kind of physics as cutting.** In milling, chip load,
   cutting coefficients and spindle power decide what a tool can remove per second. In additive,
   the corresponding quantity is the **rate at which material can be melted or cured and joined**,
   and each process has its own limiting physics:
   - **FDM / material extrusion:** the hot end can only melt so much plastic per second
     (*maximum volumetric flow*, mm³/s). Above it the core of the filament is not molten, pressure
     rises, and the extruder skips or under-extrudes. Melt viscosity, pressure lag in the nozzle
     and cooling (time above the glass transition for layer bonding) are the other limits.
   - **Resin / vat photopolymerisation:** cure depth follows the resin's *working curve*, set by
     two material constants (penetration depth `Dp` and critical exposure `Ec`).
   - **Metal powder bed fusion (LPBF) and DED:** the melt pool must be deep enough to fuse to
     the layer below and overlap its neighbour, but not so energetic that it turns into a
     keyhole. The common shorthand is *volumetric energy density* `E = P / (v · h · t)`, though
     papers show it is an unreliable single number.
2. **The numbers are large and well documented for FDM.** Prusa's help centre lists typical
   maximum volumetric speeds of 8–12 mm³/s for a standard all-metal hot end and 25–40 mm³/s for
   high-flow hot ends, and per material from 2–5 mm³/s (TPU) to 12–20 mm³/s (PLA)
   ([Prusa: Max volumetric speed](https://help.prusa3d.com/en/article/max-volumetric-speed_127176),
   accessed 2026-10-26). A research hot end reached a peak of 282 cm³/h, about 78 mm³/s
   ([Go & Hart, *Fast Desktop-Scale Extrusion Additive Manufacturing*, arXiv:1709.05918](https://arxiv.org/pdf/1709.05918)).
3. **A shared library maps cleanly onto the tool-library idea.** Split it the same way:
   - *Shared material catalogue* (per filament, resin or powder, from the maker): density,
     diameter, temperature windows, melt flow rate, glass transition, drying; `Dp`/`Ec` for resin;
     thermophysical properties and absorptivity for metal powder.
   - *Shared machine catalogue* (per hot end and nozzle, laser system or DED head): melt capacity
     curves, heater power, build volume, kinematic limits.
   - *Shop layer* (per user, per printer, per spool): measured flow limit, flow ratio, pressure
     advance, the parameter set that passed a density test, remaining spool weight.
4. **The physical-tag idea already exists for spools.** Prusa's **OpenPrintTag** is an MIT-licensed
   open standard for NFC tags on filament spools (ISO 15693 / NFC-V tags) that carries material
   type, brand, colour, temperature ranges, density, diameter and remaining weight
   ([SimplyPrint: OpenPrintTag](https://help.simplyprint.io/en/article/openprinttag-nfc-standard-in-simplyprint-1hjcz8/);
   [github.com/prusa3d/OpenPrintTag](https://github.com/prusa3d/OpenPrintTag)). It is the additive
   twin of the barcoded tool tag in the CAM research. Creality's CFS format, by contrast, is
   proprietary and AES-encrypted
   ([SimplyPrint: Creality material standard](https://help.simplyprint.io/en/article/the-creality-material-standard-nfcrfid-for-the-creality-cfs-1crrofa/)).
5. **Standards exist for the data, not yet for the physics constants.** ISO/ASTM 52950 covers AM
   data exchange in general, ASTM F3490 defines a common data dictionary for AM data pedigree, and
   the 3MF Consortium publishes a Materials and Properties extension and, since 2025, a Toolpath
   extension that started with LPBF. None of them standardises a "max volumetric flow" or a
   melt-pool criterion as a field (as far as their public scopes show).
6. **What it means for SolidCraft (opinion).** #2595 keeps the slicer hand-off to a plain 3MF and
   #6871 (an in-app slicer) was closed as not planned, so SolidCraft would not *use* this physics
   to slice. It would use it to **estimate** (time and material, #1626 topic 16) and to **check a
   design against the process** (minimum wall vs nozzle width, layer-time warnings for small
   features, feature size vs melt-pool width for LPBF). A small, cited, shared material and
   machine catalogue, readable from OpenPrintTag and Cura's CC0 material files, is enough for that.

## Fundamentals

### The analogy, term by term

| Subtractive (from the #1621 research) | Material extrusion (FDM) | Vat photopolymerisation | Laser powder bed fusion / DED |
|---|---|---|---|
| Material removal rate (mm³/s) | Volumetric flow `Q` (mm³/s) | Cured volume per exposure | Build rate (mm³/s) |
| Chip load per tooth (mm) | Extrusion cross-section `w · h` (mm²) | Layer thickness vs cure depth | Layer thickness, hatch spacing |
| Spindle power and torque limit | Hot-end melt capacity, extruder force | Light-engine irradiance (mW/cm²) | Laser power (W) |
| Cutting coefficients (Kienzle `kc1.1`, `mc`) | Melt flow rate, viscosity, glass transition, heat capacity | `Dp`, `Ec` (working curve) | Absorptivity, conductivity, melting and boiling points |
| Cutter (diameter, flutes, length) | Nozzle (diameter, melt-zone length, material) | Pixel size, light wavelength | Spot size, wavelength |
| Holder and spindle dynamics, chatter | Gantry dynamics, ringing, pressure lag | Peel forces | Recoater, spatter, residual stress |
| Tool tag in the holder bore | Spool tag (OpenPrintTag) | Resin bottle label | Powder lot certificate |

### Material extrusion (FDM / FFF)

**Volumetric flow.** The plastic leaving the nozzle in one second is the extrusion's
cross-section times its speed: `Q = w · h · v`, with line width `w`, layer height `h` and speed
`v`. For example, a 0.45 mm line at 0.2 mm layer height and 150 mm/s needs
`0.45 × 0.2 × 150 = 13.5 mm³/s`. PrusaSlicer's AutoSpeed uses exactly this relation in reverse: it
caps a target volumetric speed at the *max volumetric speed* (MVS) set in the print and filament
profiles (the lower of the two wins) and divides by each extrusion's cross-section to get its
speed ([Prusa: Max volumetric speed](https://help.prusa3d.com/en/article/max-volumetric-speed_127176)).

**Why there is a limit.** Go, Schiffres, Stevens and Hart identified three rate limits of FFF:
the force the feed mechanism can push with, the rate of heat transfer from the liquefier wall
into the core of the filament, and the speed of the stepper-driven gantry
([Go et al., *Additive Manufacturing*, 2017](https://mechanosynthesis.mit.edu/rate-limits-of-additive-manufacturing-by-fused-filament-fabrication-and-guidelines-for-high-throughput-system-design);
summarised in [arXiv:1709.05918](https://arxiv.org/pdf/1709.05918)). The heat-transfer limit is
the one users meet. A rough scaling shows why (estimate):

- *Heater power is not the problem.* Heating PLA (density about 1.24 g/cm³, specific heat about
  1.8 J/(g·K), unverified) by 190 K at 20 mm³/s takes about
  `0.0248 g/s × 1.8 J/(g·K) × 190 K ≈ 8.5 W`, well inside a 40 W heater.
- *Conduction time is the problem.* Heat must diffuse from the wall to the centre of the
  filament while it passes through the melt zone. The time scale is `r² / α`; with a polymer's
  thermal diffusivity around 0.1 mm²/s (unverified) and a 1.75 mm filament (`r ≈ 0.875 mm`),
  that is several seconds. Faster flow means less time in the melt zone, so a cold core.
  High-flow hot ends attack exactly this: longer melt zones, split or flattened filament paths,
  or, in Go and Hart's research system, laser preheating of the filament before the liquefier
  ([arXiv:1709.05918](https://arxiv.org/pdf/1709.05918)).

**Typical limits** (all from [Prusa: Max volumetric speed](https://help.prusa3d.com/en/article/max-volumetric-speed_127176)):

| Hot end class | MVS (mm³/s) | Material | MVS (mm³/s) |
|---|---|---|---|
| Standard all-metal (E3D V6 class) | 8–12 | PLA | 12–20 |
| Prusa Nextruder | 15–20 | PETG | 8–15 |
| High-flow nozzles | 20–35 | ABS / ASA | 10–16 |
| High-flow hot ends | 25–40 | PC | 6–10 |
| | | PA (nylon) | 8–12 |
| | | TPU | 2–5 |

The usable limit is the **lower** of the hot end's and the material's, and both shift with
nozzle diameter and temperature. E3D publishes its Revo nozzle limits from a
"semi-standardised testing procedure" and notes that high-flow-designed filaments reach much
higher rates ([E3D: Revo High Flow nozzles](https://e3d-online.com/products/revo-high-flow-nozzles)).
So the honest data shape is a **curve or table per (hot end, nozzle, material, temperature)**,
not one number per filament.

**Melt rheology.** ISO 1133 measures a thermoplastic's melt flow as mass per time (MFR,
g/10 min) or volume per time (MVR, cm³/10 min) through a standard die at a set temperature and
load ([ZwickRoell: ISO 1133](https://www.zwickroell.com/industries/plastics/thermoplastics-and-thermosetting-molding-materials/melt-flow-index-test-iso-1133/)).
A higher melt flow generally means lower viscosity. It is a single-point test, so it ranks
materials but does not give the shear-thinning viscosity curve that nozzle pressure depends on
(opinion; a full model would need capillary rheometry data, rarely published for filaments).

**Pressure lag (pressure or linear advance).** Plastic only flows once pressure has built up in
the nozzle, so during acceleration the extruder under-extrudes and during deceleration the stored
pressure blobs material out
([Ellis' Print Tuning Guide: Pressure/linear advance](https://ellis3dp.com/Print-Tuning-Guide/articles/pressure_linear_advance/introduction.html)).
Firmware compensates by adding extruder motion proportional to acceleration. Klipper documents
typical `pressure_advance` values between 0.050 and 1.000, the high end usually only with bowden
extruders ([Klipper: Pressure advance](https://www.klipper3d.org/Pressure_Advance.html)); Marlin
calls the same idea Linear Advance. Klipper recommends keeping slicer-side pressure features
disabled when the firmware does it
([Klipper: Slicers](https://www.klipper3d.org/Slicers.html)). The value changes with filament,
nozzle size, large temperature changes and bowden length (Ellis), so it belongs in the **shop
layer**, per printer and per material.

**Cooling and layer bonding.** Layers bond by neck growth and polymer diffusion while the
interface is hot. Sun, Rizvi, Bellehumeur and Gu measured ABS P400 parts and found bonding needs
the interface to stay above the glass transition long enough. Bottom layers, which stayed hot
longer, showed dimensionless neck growth of 0.310 against 0.187 in the top layers, and parts with
shorter paths per layer bonded better
([Sun et al., 2003](https://utw10945.utweb.utexas.edu/Manuscripts/2003/2003-29-Bellehumeur.pdf)).
This is the physics behind *minimum layer time* and fan settings. Too little cooling and small
features sag; too much and layers bond weakly. It is also why throughput and strength trade off.

### Vat photopolymerisation (resin)

Cure depth grows with the logarithm of exposure: `Cd = Dp · ln(E / Ec)`, where `E` is radiant
exposure (mJ/cm²), `Dp` is the depth at which light falls to 1/e in the resin and `Ec` is the
critical exposure that starts gelation (the Jacobs working curve; equation from general
literature, the meaning of `Dp` and `Ec` from
[Kolibaba et al., NIST, *Additive Manufacturing* 2024](https://www.nist.gov/node/1842876)). The two
constants are the resin's "cutting coefficients": with them and the light engine's irradiance, a
program can compute exposure time per layer instead of copying a preset.

### Metal powder bed fusion (LPBF)

**Volumetric energy density.** The usual shorthand is `E = P / (v · h · t)` (J/mm³), with laser
power `P`, scan speed `v`, hatch spacing `h` and layer thickness `t`. Example from a 2024 study on
316L: 285 W, 1250 mm/s, 90 µm hatch and 50 µm layers give
`285 / (1250 × 0.09 × 0.05) ≈ 50.7 J/mm³` and 99.2 % relative density; the authors cite
50–80 J/mm³ from the literature for dense 316L
([Berghaus et al., *Progress in Additive Manufacturing* 2024](https://link.springer.com/article/10.1007/s40964-024-00693-y)).

**Why one number is not enough.** Scipioni Bertoli et al. showed that `E` folds several
parameters together but "does not capture any of the complex physical phenomena" of the melt
pool, so equal `E` can give different results
([UCI Engineering on Scipioni Bertoli et al., *Materials & Design* 2017](https://engineering.uci.edu/news/2018/9/grad-student-earns-paper-award)).
The better-founded approach predicts the **melt pool** (length `L`, width `W`, depth `D`) from a
moving-heat-source model such as Eagar–Tsai (1983) and checks it against defect criteria. A 2023
printability-map framework collects them
([arXiv:2304.04113](https://arxiv.org/pdf/2304.04113)):

| Defect | Criterion (fails when) | Source cited there |
|---|---|---|
| Lack of fusion | `D ≤ t` (pool shallower than the layer) | Zhang et al. 2021 |
| Lack of fusion | `(h/W)² + t/(t + D) ≥ 1` (tracks do not overlap enough) | Zhu et al. 2021 |
| Keyhole | `W/D ≤ 2.5`–`2.75` (deep, narrow pool; threshold varies) | Johnson et al. 2019 |
| Keyhole | normalised enthalpy `ΔH/h_s > π · T_boil / T_liquidus` | King et al. 2014 |
| Balling | `L/W ≥ 2.3` (literature range 2 to π) | Zhang 2021; Johnson 2019 |

The material inputs to such a model are thermal conductivity, density, specific heat, liquidus
and boiling temperatures and laser absorptivity. That is the metal equivalent of a cutting-data
set, and it is per alloy, not per machine.

### Directed energy deposition (DED)

DED adds powder or wire into a melt pool moved by a robot or gantry. Its main parameters are
laser power, travel speed, spot size and powder (or wire) feed rate. ORNL researchers kept the
**areal energy density** (power over speed times spot size, J/mm²) constant while changing spot
size, adjusting power, feed rate and powder flow together, to keep hardness and microstructure
consistent ([Fillingim et al., *Heliyon* 2024](https://www.ornl.gov/publication/process-parameter-translation-strategies-variable-directed-energy-deposition-spot-size)).
That is a physics-based *translation rule*, the kind of thing a parameter library can store
alongside raw parameter sets.

### Process categories

ISO/ASTM 52900 groups additive processes into seven categories: vat photopolymerisation, powder
bed fusion, binder jetting, material jetting, sheet lamination, material extrusion and directed
energy deposition ([Additive Manufacturing Media](https://www.additivemanufacturing.media/kc/what-is-additive-manufacturing/3d-printing-processes)).
A shared library should key every record by one of these categories, so that "speed" or
"energy" fields are never mixed across processes.

### What a shared library would hold (proposal)

Opinion, built on the facts above and the structure of the CAM tool-library research:

1. **Material records (shared, from the maker).** Identity (maker, product, GTIN, process
   category); physical (density, diameter, colour); thermal windows (nozzle, bed, chamber,
   drying); rheology (MFR/MVR with its ISO 1133 test condition, glass transition); resin `Dp`/`Ec`
   with the wavelength they were measured at; metal powder thermophysical properties and
   absorptivity at a stated wavelength. Each value carries its source and test method.
2. **Machine records (shared, from the maker or measured by many users).** Hot end and nozzle
   melt capacity as a table over temperature and material class; heater power; laser power range
   and spot size; build volume; kinematic limits.
3. **Process parameter sets (shared or shop).** For LPBF and DED: power, speed, hatch, layer and
   the density result that validated them, on which machine. For FDM: a validated MVS and the
   test that measured it.
4. **Shop layer (local).** Per printer and spool: flow ratio, pressure advance, measured MVS,
   remaining weight, what failed. This mirrors the CAM research, where the shared catalogue
   defines the tool once and the shop keeps its own offsets.

A spool tag (OpenPrintTag) would carry the shared material identity and per-spool data, and point
at the full material record, the same way the tool tag points at a tool assembly.

## Accuracy

- **One-number descriptors mislead.** `E = P/(v·h·t)` for LPBF is the clearest case (Scipioni
  Bertoli et al. 2017, above), and the paper on 316L notes lack of fusion, keyholes and balling can
  appear together for a single parameter set
  ([Berghaus et al. 2024](https://link.springer.com/article/10.1007/s40964-024-00693-y)). A
  library should store the raw parameters, not just the derived density.
- **Material constants scatter between laboratories.** In NIST's interlaboratory study, 24
  laboratories measured the working curve of the same batch of resin; `Dp` varied by up to 7× and
  `Ec` by up to 70×, which the authors attribute to the lack of a common procedure, differing light
  engines, limits of the Jacobs equation and imprecise tools
  ([Kolibaba et al. 2024](https://www.nist.gov/node/1842876)). Every shared value needs its test
  method and equipment recorded, or it is not comparable.
- **Default flow limits are disputed.** Users have argued that slicer defaults are too high, for
  example a Prusa forum thread titled "Please reduce default values for 'max volumetric speed' from
  15 to 11.5 mm³/s" and another asking "PETG max volumetric speed only 8mm3/s?"
  ([Prusa forum](https://forum.prusa3d.com/forum/original-prusa-i3-mk3s-mk3-general-discussion-announcements-and-releases/slic3r-pe-please-reduce-default-values-for-max-volumetric-speed-from-15-to-11-5-mm%C2%B3-s/paged/2/);
  [Prusa forum](https://forum.prusa3d.com/forum/prusaslicer/petg-max-volumetric-speed-only-8mm3-s)).
  Measured, per-printer values beat catalogue values; the catalogue should be a starting point.
- **Vendor flow numbers depend on test method.** E3D's figures come from its own
  "semi-standardised" procedure ([E3D](https://e3d-online.com/products/revo-high-flow-nozzles)); no
  public standard for hot-end flow testing was found (unverified that none exists).
- **Where compensation lives (topic 15).** Flow ratio and pressure advance can be set in the
  slicer's filament profile, the printer profile *and* the firmware. Applying them in two places
  doubles the correction (Klipper's advice to disable slicer pressure features is one instance,
  [Klipper: Slicers](https://www.klipper3d.org/Slicers.html)). A library should mark each value
  with the layer it belongs to (material, machine, shop) and which program applies it.
- **Closing the loop.** The FDM checks are a flow test (raise speed until extrusion fails), a
  pressure-advance pattern (Ellis, above) and dimensional test prints. For LPBF they are single
  tracks and density cubes (Berghaus et al. used cubes and relative density). For resin, a working
  curve measurement. A shared library becomes trustworthy when each value links to such a test.

## Programs

Only what bears on process data; #1626's full program survey is a separate part.

- **PrusaSlicer** (AGPL; help pages only consulted). Max volumetric speed in both print and
  filament profiles, lower wins, and AutoSpeed derives speeds from it
  ([Prusa help](https://help.prusa3d.com/en/article/max-volumetric-speed_127176)). Shows the
  physics-based limit already works in practice for FDM.
- **Klipper** (GPL firmware; documentation only consulted). Pressure advance in firmware, a
  maximum extrusion cross-section check, extrude-only velocity limits
  ([Klipper: Slicers](https://www.klipper3d.org/Slicers.html);
  [Pressure advance](https://www.klipper3d.org/Pressure_Advance.html)).
- **Cura material profiles (`fdm_materials`).** A database of FDM material profiles used by Cura,
  packaged under CC0-1.0 ([FreshPorts: cad/fdm_materials](https://freshports.org/cad/fdm_materials)).
  If the licence is confirmed upstream, it is a possible seed for a shared material catalogue
  (unverified that every file is CC0).
- **OpenPrintTag** (MIT). Open NFC spool standard with a machine-readable field and enum
  definition and a reference implementation in Python
  ([github.com/prusa3d/OpenPrintTag](https://github.com/prusa3d/OpenPrintTag)); supported by
  SimplyPrint ([SimplyPrint](https://help.simplyprint.io/en/article/openprinttag-nfc-standard-in-simplyprint-1hjcz8/)).
  Whether it carries process limits such as MVS was not confirmed (unverified).
- **Creality CFS RFID.** Proprietary, AES-encrypted MIFARE Classic tags with material, colour,
  weight bucket, serial and batch; community reverse-engineered
  ([SimplyPrint](https://help.simplyprint.io/en/article/the-creality-material-standard-nfcrfid-for-the-creality-cfs-1crrofa/)).
  A counter-example: closed tags lock material data to one ecosystem.
- **3MF Consortium.** Royalty-free specifications including Materials and Properties (colour and
  multi-material), Production, Beam Lattice, Volumetric and a Toolpath extension for additive
  processes ([3mf.io/spec](https://3mf.io/spec/)). The Toolpath work began with LPBF and is being
  extended to DED, pellet extrusion, cold spray and friction stir, with "advanced metadata
  capabilities for machine-specific parameters"
  ([3MF Consortium, 2025-03-14](https://3mf.io/news/2025/03/ai-build-join-the-3mf-consortium-to-help-develop-the-toolpath-extension-for-multi-axis-3d-printing/)).
  SolidCraft already writes 3MF (including its own face-tag namespace,
  `docs/formats/3mf-face-tags.md`).
- **NIST AM data work.** A Common Data Model for AM, built on a community data dictionary later
  standardised by ASTM, with use cases from NIST's AM Material Database and AM Bench 2022 data
  ([Kuan et al., *IMMI* 2024](https://www.nist.gov/publications/common-data-dictionary-and-common-data-model-additive-manufacturing)).

## Complaints and requests

- **Defaults that are too optimistic** (flow limits above what a stock hot end melts), see the
  Prusa forum threads under Accuracy.
- **Locked spool data.** Proprietary, encrypted tags (Creality CFS) need reverse engineering to
  use third-party filament; OpenPrintTag exists partly as the open answer.
- **In-app slicing is unwanted by some contributors.** On #6871 a contributor argued that G-code
  is not standardised and vendor slicers are locked, so slicing should stay external; the issue
  was closed as not planned on 2026-10-26 (from the public issue page).
- **Parameters that do not transfer.** The DED spot-size study exists because parameters tuned
  at one spot size do not carry to another (Fillingim et al. 2024).

## What users need

Opinion, derived from the sections above:

1. **Honest estimates.** Print time from path length and the *volumetric* limit, not from the
   requested speed; material use from volume and density.
2. **Design checks tied to the process.** Wall thickness against nozzle and line width; small
   islands that will violate minimum layer time; for LPBF, features near the melt-pool width.
   These belong with DfAM (#1625) and need the same material and machine records.
3. **One place for each value**, labelled material, machine or shop, so compensation is not
   applied twice.
4. **Data with provenance.** Each shared value carries its source, test method and units, in the
   spirit of ASTM F3490's data pedigree.
5. **Open, readable tags.** Read OpenPrintTag spools; do not depend on encrypted formats.

## Export control, sanctions and government use

*Added 2026-10-29 from the [government use research](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-qk5riz/research/gov-research-nsnlookupjn-government-use.md). Opinion and policy proposals, not legal advice.*

**Our three rules:**

- **No one goes to jail.** Our team has no in-house counsel. So wherever a rule is unclear we take the most conservative reading and refuse rather than warn. Questions that need an outside export-control lawyer are marked as such, and free official help (BIS, DDTC, OFAC) is the first call.
- **Help users comply.** SolidCraft stops and explains in plain language, using rule packs for each jurisdiction that the administrator picks. It flags conflicts between countries' laws, never resolves them, and never gives legal advice.
- **Prevention first.** Nothing in this file that is hosted, shared, paid or networked ships until its compliance gates are in place and checked: region blocks, screening, ITAR flag handling, government-mode behaviour, and an outside lawyer's sign-off for any hosted or paid service.

**What in this file crosses a border:** a shared material and machine library with per-shop measured values and spool-tag data.

- **Hosted parts block sanctioned regions and screen people.** Any service this file proposes that the project would run blocks comprehensively sanctioned regions, plus Russia and Belarus (US, EU and UK rules restrict CAD/CAM software and related services there), and screens accounts, payers, contributors and marketplace parties against the US Consolidated Screening List and the UK, EU and UN lists.
- **No support there.** No support, consulting or custom work for users in sanctioned regions or for listed parties.
- **Government mode.** At the Controlled level (CUI and allied equivalents) online parts are off or point to the organisation's own mirror; at the Isolated level they arrive only as signed packs on media; a design flagged Export controlled (ITAR/EAR or national equivalent) never leaves through them.
- **Only generic data is shared.** Nothing derived from a controlled design (geometry, toolpaths, part or drawing numbers, test or inspection results) goes into a shared library, report or issue. Tool data stays shareable through the field allow-list in the high-security research.
- **Code stays public.** Every build the project ships is public, which keeps it "published" software outside the EAR. The rules are tracked by the standing export and sanctions watch.
- **Checked for controlled data (2026-10-29):** this file contains no controlled technical data. Its examples are public sources, generic values or made-up stand-ins, and the author's experience is stated in general terms.

## Glossary

Definitions in ISO 704 style (genus, then distinguishing features). Terms follow ISO/ASTM 52900
where it defines them (unverified for wording; the standard's text was not read).

- **volumetric flow (rate)**: rate of material deposition expressed as volume per unit time
  through a nozzle, equal to extrusion cross-section times speed. Unit mm³/s.
- **maximum volumetric speed (MVS)**: *volumetric flow* above which a given hot end, nozzle and
  material no longer extrude reliably at a given temperature.
- **melt zone**: part of a liquefier in which the feedstock is heated from solid to melt.
- **melt mass-flow rate (MFR)**: mass of thermoplastic melt extruded through a standard die per
  unit time under a set temperature and load (ISO 1133). Unit g/10 min.
- **melt volume-flow rate (MVR)**: as MFR, expressed as volume. Unit cm³/10 min.
- **pressure advance**: firmware compensation that adds extruder motion in proportion to
  acceleration to offset pressure lag in the nozzle. Called *linear advance* in Marlin.
- **minimum layer time**: lower limit on the time spent printing one layer, set so that the
  layer can cool enough before the next is laid.
- **working curve**: relation between cure depth and radiant exposure of a photopolymer resin.
- **penetration depth (`Dp`)**: depth in a resin at which irradiance has fallen to 1/e of its
  surface value.
- **critical exposure (`Ec`)**: radiant exposure at which a photopolymer resin starts to gel.
- **volumetric energy density (`E`)**: laser power divided by the product of scan speed, hatch
  spacing and layer thickness. Unit J/mm³.
- **areal energy density**: laser power divided by the product of travel speed and spot
  diameter. Unit J/mm².
- **hatch spacing (`h`)**: distance between adjacent parallel scan tracks in one layer.
- **melt pool**: region of molten metal under a moving heat source.
- **lack-of-fusion porosity**: porosity caused by melt pools that do not reach the previous
  layer or do not overlap their neighbours.
- **keyhole porosity**: porosity caused by vapour-depressed, deep and narrow melt pools that
  collapse and trap gas.
- **balling**: break-up of a melt track into droplets when the melt pool is long relative to
  its width.
- **spool tag**: data carrier attached to a feedstock spool that identifies the material and
  carries spool-specific data. Example: OpenPrintTag.

## Open questions

1. Does OpenPrintTag carry, or plan to carry, process limits (MVS, pressure advance hints,
   drying time), or only identity and temperature windows?
2. Is every file in Cura's `fdm_materials` CC0, and does its schema hold anything beyond
   temperatures and identity?
3. Is there any public test standard for hot-end volumetric flow, so maker numbers are
   comparable?
4. Will the 3MF Toolpath extension's machine-parameter metadata carry per-material process
   parameters (LPBF power/speed/hatch), and is it usable for a library, not just a build file?
5. Which ASTM standard holds the AM common data dictionary that NIST's data model builds on, and
   which of its fifteen modules cover process parameters? (ASTM F3490 is the likely one.)
6. Should SolidCraft keep this data at all, or only read it from a slicer's profile or a tag at
   estimate time? (This is a scope question for maintainers.)
7. **[TODO Jason]** Which of these would you use first: estimates, design checks, or reading
   spool tags?

## Sources

All accessed 2026-10-26 unless a date is given.

- Prusa Research, *Max volumetric speed* (help article 127176). https://help.prusa3d.com/en/article/max-volumetric-speed_127176
- Go, J. and Hart, A. J., *Fast Desktop-Scale Extrusion Additive Manufacturing*, arXiv:1709.05918 (submitted to *Additive Manufacturing*). https://arxiv.org/pdf/1709.05918
- Go, J., Schiffres, S., Stevens, A. G. and Hart, A. J., *Rate limits of additive manufacturing by fused filament fabrication and guidelines for high-throughput system design*, *Additive Manufacturing* (2017). https://mechanosynthesis.mit.edu/rate-limits-of-additive-manufacturing-by-fused-filament-fabrication-and-guidelines-for-high-throughput-system-design
- E3D, *Revo High Flow nozzles* (product page). https://e3d-online.com/products/revo-high-flow-nozzles
- ZwickRoell, *Melt flow index test ISO 1133*. https://www.zwickroell.com/industries/plastics/thermoplastics-and-thermosetting-molding-materials/melt-flow-index-test-iso-1133/
- Ellis' Print Tuning Guide, *Pressure / linear advance: introduction*. https://ellis3dp.com/Print-Tuning-Guide/articles/pressure_linear_advance/introduction.html
- Klipper documentation, *Pressure advance*. https://www.klipper3d.org/Pressure_Advance.html
- Klipper documentation, *Slicers*. https://www.klipper3d.org/Slicers.html
- Sun, Q., Rizvi, G. M., Bellehumeur, C. T. and Gu, P., *Experimental Study of the Cooling Characteristics of Polymer Filaments in FDM and Impact on the Mesostructures and Properties of Prototypes*, Solid Freeform Fabrication Symposium (2003), pp. 313–323. https://utw10945.utweb.utexas.edu/Manuscripts/2003/2003-29-Bellehumeur.pdf
- Kolibaba, T. et al., *Results of an Interlaboratory Study on the Working Curve in Vat Photopolymerization*, *Additive Manufacturing* (2024), doi:10.1016/j.addma.2024.104082. https://www.nist.gov/node/1842876
- Berghaus, M. et al., *Effect of high laser scanning speed on microstructure and mechanical properties of additively manufactured 316L*, *Progress in Additive Manufacturing* (2024), doi:10.1007/s40964-024-00693-y. https://link.springer.com/article/10.1007/s40964-024-00693-y
- UCI Samueli School of Engineering, *Grad Student Earns Paper Award* (2018), on Scipioni Bertoli, U. et al., *On the limitations of volumetric energy density as a design parameter for selective laser melting*, *Materials & Design* (2017). https://engineering.uci.edu/news/2018/9/grad-student-earns-paper-award
- *An Automated Fully-Computational Framework to Construct Printability Maps for Additively Manufactured Metal Alloys*, arXiv:2304.04113 (2023); cites Eagar & Tsai (1983), King et al. (2014), Johnson et al. (2019), Zhang et al. (2021), Zhu et al. (2021), Yadroitsev et al. (2010). https://arxiv.org/pdf/2304.04113
- Fillingim, K. B. et al., *Process parameter translation strategies for variable directed energy deposition spot size*, *Heliyon* 10(18) e37658 (2024). https://www.ornl.gov/publication/process-parameter-translation-strategies-variable-directed-energy-deposition-spot-size
- Additive Manufacturing Media, *3D printing processes*. https://www.additivemanufacturing.media/kc/what-is-additive-manufacturing/3d-printing-processes
- SIS, *SS-EN ISO/ASTM 52950:2021, Additive manufacturing — General principles — Overview of data processing* (catalogue entry). https://www.sis.se/en/produkter/manufacturing-engineering/additive-manufacturing/ss-en-isoastm-529502021/
- ASTM, *F3490-21 Standard Practice for Additive Manufacturing — General Principles — Overview of Data Pedigree* (catalogue entry). https://store.astm.org/f3490-21.html
- EVS, *ISO/ASTM 52904:2024* (catalogue entry; metal PBF process characteristics and performance for critical applications). https://www.evs.ee/en/iso-astm-52904-2024
- Kuan, A. et al., *Common Data Dictionary and Common Data Model for Additive Manufacturing*, *Integrating Materials and Manufacturing Innovation* (2024). https://www.nist.gov/publications/common-data-dictionary-and-common-data-model-additive-manufacturing
- 3MF Consortium, *Specifications*. https://3mf.io/spec/
- 3MF Consortium, *AI Build join the 3MF Consortium to help develop the Toolpath Extension* (2025-03-14). https://3mf.io/news/2025/03/ai-build-join-the-3mf-consortium-to-help-develop-the-toolpath-extension-for-multi-axis-3d-printing/
- Prusa Research, *OpenPrintTag* repository. https://github.com/prusa3d/OpenPrintTag
- SimplyPrint, *OpenPrintTag NFC standard in SimplyPrint*. https://help.simplyprint.io/en/article/openprinttag-nfc-standard-in-simplyprint-1hjcz8/
- SimplyPrint, *The Creality material standard: NFC/RFID for the Creality CFS*. https://help.simplyprint.io/en/article/the-creality-material-standard-nfcrfid-for-the-creality-cfs-1crrofa/
- FreshPorts, *cad/fdm_materials*. https://freshports.org/cad/fdm_materials
- Prusa forum, *Slic3r PE: Please reduce default values for "max volumetric speed" from 15 to 11.5 mm³/s*. https://forum.prusa3d.com/forum/original-prusa-i3-mk3s-mk3-general-discussion-announcements-and-releases/slic3r-pe-please-reduce-default-values-for-max-volumetric-speed-from-15-to-11-5-mm%C2%B3-s/paged/2/
- Prusa forum, *PETG max volumetric speed only 8mm3/s?* https://forum.prusa3d.com/forum/prusaslicer/petg-max-volumetric-speed-only-8mm3-s
- SolidCraft issues [#2595](https://github.com/tangency-labs/solidcraft/issues/2595) (Send to Slicer), [#6871](https://github.com/tangency-labs/solidcraft/issues/6871) (built-in slicer, closed), [#1625](https://github.com/tangency-labs/solidcraft/issues/1625) (DfAM research), [#1626](https://github.com/tangency-labs/solidcraft/issues/1626) (3D printing research).
