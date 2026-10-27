# CAM research: tool makers, their physics data, and choosing tools by physics

Research for [#1621](https://github.com/tangency-labs/solidcraft/issues/1621) (Research Request
for Plugin: CAM). A companion to
[cam-research-nsnlookupjn-tool-library.md](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-tool-library/research/cam-research-nsnlookupjn-tool-library.md).
This file covers **which tool makers publish data that physics-based CAM can use, where to find
it, in what format, where the tools are bought, and how SolidCraft could guide a user to choose
tools by physics**.

Licence: MIT OR Apache-2.0, the same as SolidCraft (see the Licence section of CONTRIBUTING.md).

## About this research

- **Author:** Jason ([@nsnlookupjn](https://github.com/nsnlookupjn)). Idea and direction by Jason;
  source research and writing done with an AI assistant (Claude) in October 2026.
- **Author's experience (personal experience):** machine shop apprenticeship from 1987, CNC
  programming since early controls, fluent in Fusion, Mastercam, SOLIDWORKS and NX; engineering
  and manufacturing defence and aerospace spare parts (see the tool library file for the full
  note).
- **Method:** web searches of tool makers' public pages, trade press, CAM vendor help and
  published manuals. Many tool makers' pages render only in a browser and returned no text to
  the research tools; where a page could not be read, the claim is marked (unverified) or taken
  from trade press. No commercial CAD/CAM program was driven, and no maker's software was run.
- **Units:** millimetres, newtons, watts, radians unless quoted otherwise.
- **Marking:** "(unverified)" marks a claim not confirmed against a source. Facts, opinion and
  marketing are labelled where they mix.

**This research is not complete.** It is a starting point: the author will keep researching
and adding to it, and others are encouraged to do the same, correct it and contribute their own
findings and shop experience. A table like the one below goes out of date quickly; corrections
from tool makers are especially welcome.

## Summary

1. **"Physics data" means more than feeds and speeds.** To choose a tool by physics, CAM needs
   five kinds of data (see Fundamentals): exact geometry, the cutting material and coating,
   force coefficients for the work material, stiffness and dynamics of the assembly, and limits
   and life. Most makers publish only the first two well, plus **start values** for speed and
   feed.
2. **Geometry is solved: ISO 13399 / GTC.** Large makers publish tool geometry and 3D models in
   ISO 13399 form, often as GTC packages: Kennametal offered ISO 13399 data for about 50,000
   products through NOVO
   ([Machinery, c. 2014](https://www.machinery.co.uk/content/features/online-data-and-more)),
   Sandvik Coromant exports GTC packages from CoroPlus Tool Library
   ([Sandvik support](https://www.sandvik.coromant.com/en-us/support/coroplus-tool-library-support/export-gtc-package)),
   and aggregators carry many brands (MachiningCloud, ToolsUnited).
3. **Force data is published per work material, not per tool.** Tool makers' technical guides
   publish `kc1`/`mc` tables per material group (the values on
   [Wikipedia](https://en.wikipedia.org/wiki/Cutting_force_(machining)) are Sandvik's), and
   reference sites publish tables of about 40 material groups with `kc1` from about 700 to
   4000 N/mm² and `mc` about 0.2 to 0.3
   ([Machining Doctor](https://www.machiningdoctor.com/glossary/specific-cutting-force-kc-kc1)).
   That is enough for power and torque. Per-tool force coefficients (the six edge-force
   coefficients) are measured by research labs and simulation vendors, not published by tool
   makers (no public per-tool set was found; (unverified) that none exists).
4. **Dynamics and stiffness are almost never published.** No tool maker was found publishing
   tool-tip FRFs, stiffness or runout per tool. Chatter software (e.g. CutPro) gets them from tap
   tests and calibration cuts
   ([CutPro guide, 2013](https://www.malinc.com/wp-content/uploads/2014/05/CutPro_Guide.pdf)).
5. **Makers' own advisers do some physics, inside their own walls.** Iscar's ITA takes available
   machine power into account and is free
   ([Iscar](https://www.iscar.com/newarticles.aspx/lang/en/newarticleid/2565));
   MachiningCloud validates feeds and speeds "against machine torque, power, and maximum spindle
   speed" and gives recommended projection lengths
   ([MachiningCloud–Mastercam white paper, 2017](https://files.solidworks.com/partners/pdfs/13987-5267-MachiningCloud_Mastercam_Connection_White_Paper.pdf));
   Helical's Machining Advisor Pro computes running parameters for its own end mills
   ([Harvey Performance, 2020](https://www.harveyperformance.com/in-the-loupe/machining-advisor-pro-1-5-improvements/)).
   None of these exports its underlying physics model to other CAM.
6. **So SolidCraft has to combine sources:** geometry from ISO 13399/GTC, force from published
   material tables, dynamics from the shop's own tap tests or RCSA, life and limits from vendor
   start values corrected by shop results (SolidCraft Edge). Every value carries its source and
   confidence, as set out in the tool library and cutting physics files.
7. **Choosing tools by physics is new to most programmers and operators** (the author's view).
   SolidCraft should explain choices in plain shop terms ("this 12 mm 3-flute takes the pocket
   in 4 min at 62 % spindle power; the 10 mm is limited by deflection at this reach") and give
   novices a short ranked list while pros see every tool and every limit.

## Fundamentals

### The five kinds of tool data physics needs

| Kind | What it is | Used for | Who publishes it today |
|---|---|---|---|
| 1. Geometry | ISO 13399 properties (`DC`, `APMX`, `LU`, `OAL`, `NOF`, `RE`, helix, rake), 3D model | Toolpaths, collision, stiffness | Most large makers, in ISO 13399/GTC; small makers often only drawings |
| 2. Cutting material | Substrate (HSS, carbide grade), coating, ISO 513 group | Speed limits, life, which materials | Makers' catalogues; grade names are maker-specific |
| 3. Force coefficients | `kc1`/`mc` per work material, or six edge-force coefficients per tool + material | Force, power, torque, deflection | Per material in technical guides; per tool only by labs/simulation vendors |
| 4. Dynamics | Stiffness, tool-tip FRF, runout | Chatter, deflection, finish | Practically nobody; measured in the shop |
| 5. Limits and life | Max `vc`, `fz` range, max `ap`/`ae`, ramp angle, Taylor constants, max RPM of holder | Safe envelope, tool life | Start values per material on product pages and charts; life rarely |

The details of each kind and how the physics uses it are in the
[cutting physics file](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-cutting-physics/research/cam-research-nsnlookupjn-cutting-physics.md);
holder limits (RPM, balance, clamping torque, runout) are in the
[tool holding file](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-4aoliy/research/cam-research-nsnlookupjn-tool-holding.md).

### Formats the data comes in

| Format | Contents | Notes |
|---|---|---|
| ISO 13399 (`.p21` STEP Part 21) | Geometry properties and classification | Machine-readable; best source |
| GTC package (`.zip`) | ISO 13399 data + class hierarchy + 3D models + pictures | Fusion imports it but ignores the STEP model ([Autodesk help](https://help.autodesk.com/cloudhelp/ENU/Fusion-CAM/files/MFG-TOOL-LIBRARY-IMPORT.htm)) |
| DIN 4000 | German tool classification and properties | Carried by ToolsUnited (unverified detail of coverage) |
| STEP / DXF models | 3D model and 2D drawing | Geometry only |
| CAM-vendor libraries (e.g. Fusion `.json`/`.tools`) | Geometry + presets for one CAM | Locked to that CAM; quality varies (complaint 10 in the tool library file) |
| PDF/web speeds-and-feeds charts | Start values per material | Human-readable only; needs extraction (AI datasheet reading in the tool library file) |
| Maker adviser apps (ITA, NOVO, MAP, Seco Assistant, CoroPlus ToolGuide) | Recommended conditions, sometimes power | Results only; the model stays inside the app |
| Material `kc1`/`mc` tables | Force data per work material group | In technical guides and handbooks |

## Accuracy

- **Start values are not limits.** Harvey Tool says its posted parameters are "suggested starting
  values that may be increased given optimal setup conditions"
  ([Harvey Tool](https://www.harveytool.com/resources/speeds-feeds)). They assume a rigid setup;
  physics must correct them for the user's machine, holder and stick-out.
- **Material tables are averages.** One `kc1`/`mc` pair per material group; real values vary with
  condition, rake, feed and wear (see Rubeo & Schmitz 2016 in the tool library file). Grouping
  schemes differ by maker (Sandvik's MC codes vs ISO 513 letters vs about 40 numbered groups);
  the numbered 41-group scheme is (unverified) as VDI 3323.
- **Same nominal tool, different performance.** The author's jobber drill anecdote (tool library
  file): a Haas Tooling drill outlived a generic drill by hundreds of cycles at faster data.
  Geometry alone can't predict that; maker identity and shop results can.
- **Maker advisers are marketing as well as engineering.** They recommend the maker's own tools
  (opinion). SolidCraft should rank tools from all makers with the same physics.

## Programs and platforms (tool makers, aggregators, advisers)

Status as found in October 2026. "Physics" = what the maker's own software computes, as far as
public pages say.

| Maker / platform | Where the data is | Formats | Cutting data | Physics in their software | Where to buy |
|---|---|---|---|---|---|
| **Sandvik Coromant** (also Seco, Walter in the Sandvik group (unverified ownership detail)) | CoroPlus Tool Library; CoroPlus ToolGuide | GTC export ([Sandvik](https://www.sandvik.coromant.com/en-us/support/coroplus-tool-library-support/export-gtc-package)); Mastercam add-in ([Sandvik](https://www.sandvik.coromant.com/en-us/support/coroplus-tool-library-support/mastercam-add-in)) | ToolGuide recommendations (unverified detail; page not readable) | `kc1`/`mc` per material published ([Wikipedia table, Sandvik data](https://en.wikipedia.org/wiki/Cutting_force_(machining))) | Distributors; maker's web shop (unverified per country) |
| **Kennametal** | NOVO / NOVOsphere | ISO 13399, ~50,000 products ([Machinery](https://www.machinery.co.uk/content/features/online-data-and-more)) | Tool Advisor | NOVO Optimize collected machine data via MTConnect (subscription) (same source) | Distributors, e.g. MSC added 15,000+ Kennametal tools ([MMS](https://mmsonline.com/news/msc-adds-kennametal-cutting-tools)) |
| **Iscar** | E-catalogue, ITA, IQ Cloud | 2D/3D models, assembly models ([Iscar](https://www.iscar.com/newarticles.aspx/lang/en/newarticleid/2565)) | ITA, free | Takes available machine power into account | Distributors |
| **Seco Tools** | Seco Assistant app; DataMatrix on packs | (unverified) | App gives product info and calculators ([Seco, 2020-09](https://www.secotools.com/article/save_time_and_boost_productivity_with_the_new_seco)) | (unverified) | Distributors |
| **Walter** | GPS tool selection | (unverified; source pages not readable) | GPS recommendations | (unverified) | Distributors |
| **Mitsubishi Materials** | Web catalogue; ISO 13399 property guide | ISO 13399 ([MMC](https://www.mmc-carbide.com/us/technical_information/iso/iso13399_property)) | Catalogue | (unverified) | Distributors |
| **Harvey Tool / Helical** | Per-product speeds & feeds; Machining Advisor Pro | Charts (format unverified) | Per material, per tool | MAP: speed, feed, depths, MRR for its own tools | Distributors and direct (unverified) |
| **Haas Tooling** | Product pages: speeds and feeds chart, ISO material chart, drawings, cutting videos; data in Haas control VPS templates ([Haas](https://www.haascnc.com/Community/Events/Benefits_of_Haas_Tooling.html)); Fusion library ([Autodesk blog](https://www.autodesk.com/products/fusion-360/blog/haas-tooling-fusion-360-tool-library/)) | Fusion library; drawings | Per tool, from Haas testing | Built into Haas control templates | Direct online ([haastooling.com](https://www.haastooling.com/c/milling)) or through a Haas Factory Outlet |
| **MachiningCloud** (aggregator) | Many makers | Job file export to Mastercam and others; supports ISO 13399/GTC ([MMS](https://mmsonline.com/news/machiningcloud-provides-iso-13399-gtc-support)) | Feeds and speeds | Validated against machine torque, power, max RPM; projection lengths; inventory ([white paper](https://files.solidworks.com/partners/pdfs/13987-5267-MachiningCloud_Mastercam_Connection_White_Paper.pdf)) | Links to stock and distributors |
| **ToolsUnited / CIMSOURCE** (aggregator) | ~550,000 tool records in 2014 ([MMS](https://mmsonline.com/products/software-accesses-550000-tool-data-records)) | DIN 4000 / ISO 13399 (unverified) | (unverified) | (unverified) | — |
| **CutPro (MAL Inc.)** (simulation, not a tool maker) | Material database built from calibration cuts | Own files | — | Force coefficients, FRF from tap tests, stability lobes ([CutPro guide](https://www.malinc.com/wp-content/uploads/2014/05/CutPro_Guide.pdf)) | Commercial software |
| **Generic / marketplace tools** | Seller listing | Often none | Rarely | None | Online marketplaces |

**Gaps to fill next:** OSG, Guhring, YG-1, Emuge-Franken, Kyocera, Tungaloy, Sumitomo, Garr,
SGS/Kyocera SGS, Ingersoll, Dormer Pramet, Nachi, regional makers. Pull requests to this table
are the point of the "not complete" note above.

## Complaints and requests

- From the tool library file: vendor libraries load badly into CAM; vendor cutting data is too
  aggressive or, sometimes, really is that good; vendors should provide real-world physics data
  (author's experience).
- **New request (the author):** give the user every piece of information needed to choose tools
  by physics, both for the tools they already own and when buying new ones.

## What users need

### Two situations

**A. Tools the user already owns.** The question is "which of my tools does this job best on
this machine?"

1. SolidCraft reads the shop library (by machine and material scope, tool library file).
2. For each operation, it runs every candidate tool through the physics: power and torque at
   the spindle speed, deflection at the needed reach, chatter risk (if the spindle FRF is known),
   chip thinning and chip evacuation, holder RPM and clamping limits, life.
3. It ranks tools by predicted time (or cost) within limits, and shows the **binding limit** for
   each ("limited by deflection", "limited by spindle power at 3,000 rpm", "limited by holder
   max RPM").
4. If a tool lacks data, it is still ranked, but derated and labelled ("generic HSS class").

**B. Buying new tools.** The question is "what should I buy for this job, and where?"

1. Same physics, run over the **shared catalogue** instead of the shop library, filtered to tools
   that fit the machine (taper, max length, weight), the material, and the holders the shop owns
   (or a holder to buy too).
2. Shows the predicted gain against the best tool already owned ("saves 6 min per part; at 400
   parts, 40 h") and the tool's price where a seller lists it (pricing feeds (unverified)
   availability).
3. **Where to buy** for each candidate: the maker's own shop if it has one (e.g. Haas Tooling
   direct), authorised distributors (e.g. MSC for Kennametal), and aggregator stock links
   (MachiningCloud). SolidCraft lists sources; it doesn't sell (opinion; commercial partnerships
   are for maintainers to decide).
4. **Data quality is part of the ranking:** a tool with complete, vendor-measured data and shop
   results ranks above an otherwise equal tool with only a PDF chart (the data completeness
   score in the tool library file).

### Guiding programmers and operators who have never chosen tools by physics

The author expects choosing tools by physics to be new to almost every programmer and operator.
Proposed (opinion):

- **Novice view:** for each operation, three suggestions ("Best", "Good", "Safe"), one sentence
  each in shop language, with the risk slider (cutting physics file) set conservative by default.
  No coefficients shown unless asked.
- **Pro view:** every candidate tool, every limit as a number and as a bar against its maximum,
  the source and confidence of each input, and the option to lock any value.
- **"Why not this tool?"** for any tool the user expected to see: "your 10 mm end mill deflects
  0.06 mm at 45 mm reach; finish tolerance is 0.02 mm".
- **Learn by comparison:** show the user's current choice beside the physics choice with
  predicted time, load and life, and after the job show the measured result from
  SolidCraft Edge, so trust grows from evidence.
- **Same engine for both views:** novices and pros get the same physics and the same hard
  limits; only the amount shown differs.
- **Plain glossary links** on every term (chip thinning, deflection, stability lobe).

### Data each catalogue entry should carry for this to work

Geometry (ISO 13399, 3D model), substrate and coating with ISO 513 groups, force data
(maker-measured coefficients or a reference to a material table), dynamics where measured,
start values with their test conditions, limits, life data, price and sellers, and for every
field its source, date and confidence (the vendor data profile in the tool library file).

## Glossary

| Term | Other names | Definition | Note |
|---|---|---|---|
| Physics data | technical tool data | Measured or modelled values that let software predict cutting forces, power, deflection, stability and life. | Five kinds in Fundamentals. |
| Start values | recommended cutting data, speeds and feeds | A maker's suggested speed, feed and engagement for a tool in a material. | Not limits; assume a rigid setup. |
| Tool adviser | tool selector, cutting data app | A program from a maker that recommends tools and conditions for a job. | E.g. ITA, NOVO, MAP. |
| Aggregator | tool data platform | A service that collects tool data from many makers in one format. | E.g. MachiningCloud, ToolsUnited. |
| Binding limit | limiting factor | The physical limit that stops a tool from cutting faster. | Shown for each tool in the ranking. |
| Material group | material class, MC code | A set of work materials treated as one for cutting data. | Schemes differ by maker. |
| Data completeness score | — | A measure of how many of a tool's physics fields are filled from trusted sources. | Proposed in the tool library file. |

## Open questions

1. Which makers publish per-tool force coefficients, stiffness or FRF data anywhere, even on
   request? Ask makers directly; check maker papers at CIRP and NAMRC.
2. Licence of maker data for a shared open catalogue (same question as the tool library file).
3. Do any makers offer an API for their catalogue or adviser (e.g. Sandvik CoroPlus, Kennametal
   NOVO)? Check developer pages.
4. Which aggregators allow open-source CAM to integrate, and on what terms?
5. Price and stock feeds: which distributors offer them publicly?
6. Fill the "gaps to fill next" makers in the table.
7. Which material grouping SolidCraft should use internally, with mappings to makers' schemes
   (shared with the material scope question in the tool library file).

## Sources

### Books, papers, manuals
- MAL Inc., CutPro guide (V9, 2013). [PDF](https://www.malinc.com/wp-content/uploads/2014/05/CutPro_Guide.pdf)
- See also the Sources of the tool library and cutting physics files.

### Tool makers and platforms
- [Sandvik Coromant: export GTC package](https://www.sandvik.coromant.com/en-us/support/coroplus-tool-library-support/export-gtc-package)
- [Sandvik Coromant: CoroPlus Tool Library Mastercam add-in](https://www.sandvik.coromant.com/en-us/support/coroplus-tool-library-support/mastercam-add-in)
- [Iscar: ITA and e-catalogue](https://www.iscar.com/newarticles.aspx/lang/en/newarticleid/2565)
- [Seco Tools: Seco Assistant app (2020-09)](https://www.secotools.com/article/save_time_and_boost_productivity_with_the_new_seco)
- [Mitsubishi Materials: ISO 13399 properties](https://www.mmc-carbide.com/us/technical_information/iso/iso13399_property)
- [Harvey Tool: speeds and feeds](https://www.harveytool.com/resources/speeds-feeds)
- [Harvey Performance: Machining Advisor Pro 1.5 (2020-01-30)](https://www.harveyperformance.com/in-the-loupe/machining-advisor-pro-1-5-improvements/)
- [Haas: benefits of Haas Tooling](https://www.haascnc.com/Community/Events/Benefits_of_Haas_Tooling.html)
- [Haas Tooling: milling](https://www.haastooling.com/c/milling)
- [MachiningCloud–Mastercam connection white paper (2017)](https://files.solidworks.com/partners/pdfs/13987-5267-MachiningCloud_Mastercam_Connection_White_Paper.pdf)
- [Autodesk: Haas Tooling in the Fusion tool library (2022-02-02)](https://www.autodesk.com/products/fusion-360/blog/haas-tooling-fusion-360-tool-library/)
- [Autodesk: tool library import](https://help.autodesk.com/cloudhelp/ENU/Fusion-CAM/files/MFG-TOOL-LIBRARY-IMPORT.htm)

### Trade press and reference sites
- [Machinery: Online data and more (Kennametal NOVO, c. 2014)](https://www.machinery.co.uk/content/features/online-data-and-more)
- [MMS: MachiningCloud provides ISO 13399/GTC support](https://mmsonline.com/news/machiningcloud-provides-iso-13399-gtc-support)
- [MMS: ToolsUnited, 550,000 tool data records (2014-07-30)](https://mmsonline.com/products/software-accesses-550000-tool-data-records)
- [MMS: MSC adds Kennametal cutting tools](https://mmsonline.com/news/msc-adds-kennametal-cutting-tools)
- [MMS: Machining Advisor Pro (2018-03-27)](https://mmsonline.com/products/software-provides-customized-running-parameters-for-end-mills)
- [Machining Doctor: specific cutting force kc1](https://www.machiningdoctor.com/glossary/specific-cutting-force-kc-kc1)
- [Machining Doctor: kc1 and mc chart](https://www.machiningdoctor.com/?p=7620)
- [Wikipedia: Cutting force (machining)](https://en.wikipedia.org/wiki/Cutting_force_(machining))

### Companion research in this fork
- [Tool library](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-tool-library/research/cam-research-nsnlookupjn-tool-library.md)
- [Cutting physics](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-cutting-physics/research/cam-research-nsnlookupjn-cutting-physics.md)
- [Tool holding](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-4aoliy/research/cam-research-nsnlookupjn-tool-holding.md)
