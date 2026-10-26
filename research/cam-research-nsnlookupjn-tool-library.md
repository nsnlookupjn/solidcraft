# CAM research: shared cloud tool library, cutting data and tool tags

Research for [#1621](https://github.com/tangency-labs/solidcraft/issues/1621) (Research Request
for Plugin: CAM). This file covers one part of that request: **tools and holders, cutting data,
and how a tool library is stored, shared and tied to physical tools.** Other CAM topics are out of
scope here.

Licence: MIT OR Apache-2.0, the same as SolidCraft (see the Licence section of CONTRIBUTING.md).

## About this research

- **Author:** Jason ([@nsnlookupjn](https://github.com/nsnlookupjn)). Idea and direction by Jason;
  source research and writing done with an AI assistant (Claude) in October 2026.
- **How it was produced:** web searches of public sources (standards bodies' catalogue pages,
  tool makers' public pages, CAM vendors' public help pages, published papers, user forums),
  each claim cross-checked against the cited page where possible. No commercial CAD/CAM program
  was driven or scripted, no GPL/LGPL/AGPL source was read, and no text is copied beyond
  single-sentence quotes. Standards are described from their public catalogue entries, not
  their paid text.
- **Units:** millimetres, newtons, watts, radians unless a source is quoted in other units.
- **Marking:** "(unverified)" marks a claim not confirmed against a source. **Facts**, **opinion**
  and **marketing** are labelled where they mix.

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

- **How the author's shop manages tools today:** every tool has a barcoded 3D-printed tag, and
  the CAM tool libraries are set up both by machine and by material, with each tool tracked by
  its tag ID (details under "Tool identification" below).

## Summary

1. **The idea.** A tool (for example a vendor end mill by part number) and a holder are each
   defined **once**, in a shared catalogue, and every SolidCraft user can pull them in instead of
   re-typing geometry. A user's library then holds *assemblies* (cutter + holder + stick-out) and
   *shop data* (tool number, pocket, measured length, wear, cutting presets) that reference the
   shared catalogue entries.
2. **A standard already exists for the shared part.** ISO 13399 defines a vendor-neutral way to
   describe cutting tools and holders, with short property symbols such as `DC` (cutting
   diameter), `APMX` (maximum depth of cut), `OAL` (overall length) and `LU` (usable length)
   ([Mitsubishi Materials, ISO 13399 property list](https://www.mmc-carbide.com/us/technical_information/iso/iso13399_property),
   accessed 2026-10-26). Tool makers ship it as **GTC (Generic Tool Catalog)** packages, built
   by Siemens PLM, Sandvik Coromant, Iscar and Kennametal on top of ISO 13399
   ([Machinery](https://www.machinery.co.uk/content/news/vendor-neutral-cutting-tools-catalogue-structure-supports-standardised-data-exchange-with-plm-and-cadcam-systems),
   undated). Fusion already imports GTC `.zip` packages and `.p21` files
   ([Autodesk help: Import and export tools](https://help.autodesk.com/cloudhelp/ENU/Fusion-CAM/files/MFG-REF-LIBRARY-EXP-IMP.htm),
   accessed 2026-10-26). SolidCraft should read GTC rather than invent a catalogue format.
3. **Cutting data is the part nobody shares well.** Vendor libraries ship *presets* (spindle
   speed, feed, step-down, step-over per material); for example, Haas's library in Fusion includes
   "feeds, speeds, step-downs, and stepovers for a range of materials"
   ([Autodesk Fusion blog, 2022-02-02](https://www.autodesk.com/products/fusion-360/blog/haas-tooling-fusion-360-tool-library/)).
   Presets are fixed numbers. **Physics-based** toolpaths instead need **material coefficients**
   (Kienzle `kc1.1`/`mc`, or the six-coefficient linear edge-force model) and **tool–holder–spindle
   dynamics**, from which force, power, deflection and chatter-free depths can be computed for any
   engagement. These coefficients are a natural second shared catalogue: per material, per tool
   family.
4. **Tool tags tie the data to the real tool.** Industry does this with RFID chips in a
   standardised bore in the holder (DIN 69873) or with laser-marked/printed DataMatrix codes.
   Zoller already lets data "traditionally stored on RFID chips" be read from a DataMatrix
   instead ([Control Design, 2024-05-13](https://controldesign.com/industry-news/news/55038158/collaboration-transfers-tool-information-to-mms-software)).
   A **printed label in a 3D-printed holder that clips into that bore**
   ([MakerWorld model 1291677, HSK63/SK40 data-holder label holder](https://makerworld.com/en/models/1291677-hsk63_sk_40_dataholder-label-holder))
   is a near-zero-cost way to get the same benefit in small shops.
5. **The complaints are consistent:** tools re-entered in every program, duplicates, CAM library
   and machine tool table drifting apart, holders missing geometry. A shared catalogue plus
   per-shop assemblies plus scannable tags addresses all four.
6. **Four requirements from Jason (2026-10-26), covered under Fundamentals below:** the library
   and the tags record **which insert and grade** is in an indexable tool right now; a tool can
   be added by **AI reading its datasheet**, with a person approving every value; a **custom tool
   with its own 3D model** is a first-class entry; and a library can be **locked to a machine or
   machine group**, including a fixed, preloaded tool set that doesn't change.

## Fundamentals

### Tools, holders and assemblies

- **Cutting item / tool item / adaptive item / assembly.** ISO 13399 splits a tool into
  components with separate reference dictionaries; the published parts include ISO 13399-1
  (overview and fundamental principles, 2006), ISO/TS 13399-2 (cutting items), -3 (tool items),
  -50 (reference systems and common concepts) and -80 (3D model creation)
  ([BSI Knowledge, cutting-items dictionary](https://knowledge.bsigroup.com/products/cutting-tool-data-representation-and-exchange-reference-dictionary-for-the-cutting-items);
  [ISO/TS 13399-50:2013 listing](https://infostore.saiglobal.com/en-au/standards/iso-ts-13399-50-2013-595585_saig_iso_iso_1364355/);
  [ISO/TS 13399-80:2014 listing](https://ecommerce.inn.cl/isots-13399-80201449355)). Exact part
  scopes and current editions are (unverified) beyond the catalogue titles.
- **Property symbols.** Each property has a short symbol (e.g. `DC`, `DCON` connection diameter,
  `LF` functional length, `NOF` flute count, `RE` corner radius, `KAPR` cutting-edge angle)
  ([Mitsubishi Materials](https://www.mmc-carbide.com/us/technical_information/iso/iso13399_property)).
  Using these symbols as SolidCraft's field names makes import and export lossless and avoids
  inventing a vocabulary.
- **Holders** are their own catalogue items. Onshape CAM Studio, for example, keeps arbors and
  holders in separate libraries and defines each by a profile of diameter/length points; a tool
  is "a cutter with an optional arbor and holder"
  ([Onshape tech tip](https://www.onshape.com/en/resource-center/tech-tips/machine-tools-cnc-programming-cam-studio),
  accessed 2026-10-26).
- **Assembly** = cutter + holder (+ extensions) + **stick-out** (gauge length). The stick-out is
  set per assembly in the shop, not per catalogue item, and it drives both collision checking and
  dynamics (see below).

### What the shared catalogue holds vs. what the shop holds

| Layer | Example | Owner | Shared? |
|---|---|---|---|
| Catalogue item (cutter) | Vendor end mill, part number, `DC` 12 mm, `NOF` 3, `APMX` 26 mm, 3D model | Tool maker or community | Yes, read-only, versioned |
| Catalogue item (holder) | CAT40 / BT40 / HSK-A63 shrink-fit, profile, max RPM, data-carrier bore | Holder maker or community | Yes |
| Cutting-data set | Material group × tool family: `kc1.1`, `mc` or edge-force coefficients; vendor start values | Tool maker, community, or shop | Yes, with provenance |
| Assembly | Cutter + holder + stick-out 38 mm | Shop | Within a shop/team |
| Machine tool | Tool number T12, pocket, measured length/diameter offsets, life used | Shop / machine | No (per machine) |
| Tag | ID printed on label / in RFID chip → points at the assembly or machine tool | Shop | No |

This split is the design proposal of this file (**opinion**): the first three layers are what
"create once, share to all users" means; the last three are private. It is also why
de-duplication works: two shops using the same vendor part number refer to the same catalogue
entry instead of each typing their own copy.

### Cutting data and physics-based toolpaths

- **Presets (what most CAM ships today).** Fixed spindle speed, feed per tooth, step-down and
  step-over per material per tool. Example: Haas Tooling in Fusion, with "dedicated side milling
  and slot milling presets"
  ([Autodesk Fusion blog](https://www.autodesk.com/products/fusion-360/blog/haas-tooling-fusion-360-tool-library/)).
  A preset is only valid for the engagement it was written for.
- **Specific cutting force (Kienzle).** Cutting force is proportional to the uncut chip
  cross-section through a specific cutting force `kc`, which depends on chip thickness through a
  reference value `kc1.1` (at 1 mm chip thickness) and exponent `mc`. Commonly written
  `Fc = b · h · kc1.1 · h^(−mc)` with `b` chip width (mm) and `h` chip thickness (mm) (Kienzle,
  VDI-Z 94, 1952, pp. 299–305, (unverified) page numbers). Tabulated example values
  (Sandvik data, reproduced on
  [Wikipedia: Cutting force (machining)](https://en.wikipedia.org/wiki/Cutting_force_(machining)),
  accessed 2026-10-26):

  | Material group | `kc1.1` (N/mm²) | `mc` |
  |---|---|---|
  | Unalloyed steel, C < 0.55 % | 1500 | 0.25 |
  | Low-alloy steel, not hardened | 1700 | 0.25 |
  | Stainless steel | 2800 | 0.21 |
  | Grey cast iron | 1100 | 0.28 |
  | Aluminium alloys | 350 | 0.25 |
  | Titanium alloys | 1300 | 0.23 |

  Cutting power is then `Pc = Fc · vc` (W, with `vc` in m/s); feed and normal components add
  about 1 % (same source). That is enough to check a toolpath against **spindle power** and
  **holder/tool torque** for any engagement, which presets cannot do.
- **Linear edge-force (mechanistic) model.** Six coefficients per material/tool pair: three
  cutting coefficients (tangential, radial/normal, axial) times chip area, plus three edge
  coefficients times chip width. Calibrated from average forces over a sweep of feed per tooth
  (linear regression), or from one cut by nonlinear fitting; coefficients vary with feed, radial
  immersion and spindle speed, so a single set is an approximation
  ([Rubeo & Schmitz, Precision Engineering, 2016](https://mtrc.utk.edu/wp-content/uploads/sites/45/2019/09/mechanistic_force_model_comparison.pdf)).
  Standard textbook treatment: Altintas, *Manufacturing Automation*, 2nd ed., Cambridge
  University Press, 2012, ch. 2 (unverified chapter number).
- **Deflection.** With force known, tool deflection follows from cantilever stiffness of the
  stick-out; this is why the assembly's stick-out belongs in the library and not just the cutter.
- **Chatter / stability.** The chatter-free axial depth depends on spindle speed and on the
  **tool-tip frequency response function (FRF)**, which changes with every tool, holder and
  stick-out (Altintas & Budak, "Analytical prediction of stability lobes in milling",
  *CIRP Annals* 44(1), 1995, pp. 357–362, (unverified) page numbers). **Receptance coupling
  (RCSA)** predicts the tool-tip FRF by measuring the spindle once and modelling tool and holder
  as beams, so "many tool and holder combinations can be evaluated quickly"
  ([Schmitz et al., "Review and status of tool tip frequency response function prediction using
  receptance coupling", Precision Engineering 79, 2023, pp. 60–77](https://mtrc.utk.edu/wp-content/uploads/sites/45/2022/10/RCSA-review.pdf)).
  **This is the strongest argument for a shared holder/tool geometry catalogue:** if holder and
  cutter geometry (and material) are exact and shared, each shop only measures its spindle once,
  and stability for any assembly can be predicted.
- **Tool life.** Taylor's tool-life equation `vc · T^n = C` (Taylor, "On the art of cutting
  metals", Trans. ASME 28, 1907) is the usual base for life tracking; per-material `n`, `C`
  belong in the cutting-data set (unverified that any GTC package carries them).

### Tool identification (tool tags)

- **RFID in the holder.** DIN 69873 ("Data media for tools and clamping devices; dimensions for
  data media and their mounting space", 1993-07) standardises the chip and its mounting bore
  ([DIN Media](https://dinmedia.de/en/standard/din-69873/2113394)). Chips store tool data read
  without contact, or only an ID that points to an external database
  ([US 10,896,363, Franz Haimer, 2021](https://patents.justia.com/patent/10896363)). Balluff
  sells low-frequency and high-frequency RFID systems that pass tool data to the CNC
  ([Balluff](https://www.balluff.com/en-us/application-examples-and-solutions/automated-tool-management-with-rfid)).
  Steep-taper (SK/CAT/BT 40) and HSK holders commonly have this bore on the flange
  (unverified per holder type).
- **DataMatrix / barcode.** Zoller reads data "traditionally stored on RFID chips" from a
  DataMatrix code ([Control Design, 2024](https://controldesign.com/industry-news/news/55038158/collaboration-transfers-tool-information-to-mms-software));
  Seco prints DataMatrix codes that open tool data in its app
  ([Seco Tools](https://www.secotools.com/article/seco_data_matrix)).
- **Printed label holder.** Jason's suggested example is a 3D-printed label holder for HSK63 and
  SK40 holders that sits in the data-carrier position
  ([MakerWorld 1291677](https://makerworld.com/en/models/1291677-hsk63_sk_40_dataholder-label-holder);
  page could not be fetched for this research, so fit, label size and licence are (unverified)).
  A small shop can print a QR/DataMatrix label per assembly with no RFID reader.
- **What the tag should encode (opinion).** Only a stable ID (e.g. a short UUID or a
  `shop/assembly/serial` string), never the data itself, so the label never goes stale and the
  library stays the single source of truth. Scanning it opens the assembly: catalogue items,
  stick-out, measured offsets, life used.

**Author's practice today (personal experience):**

- Every tool has a **3D-printed tool tag** with a **barcode carrying an ID**.
- The tag also shows, for people: the **vendor**, **part number**, **applicable materials as
  colour-coded dots**, the **tool-setter length**, and optionally the **insert family and grade**.
- **The tag stays with the tool.** When the tool is loaded into a machine, its tag goes into a
  **tag rack on that machine, in the slot for its pocket number**, so the rack is a physical
  copy of the machine's tool table.
- In CAM, the **ID tracks the tool** through libraries set up **by machine and by material**,
  which is the same structure as the machine-scoped and material-scoped libraries below.

**What this suggests for SolidCraft (opinion):**

- **The barcode carries only the ID; the printed text is a convenience.** This refines the point
  above: human-readable fields on the tag are useful at the machine, but the library is the
  record. SolidCraft should print the tag from the library (barcode + vendor + part number +
  material dots + length + insert), and flag a tag for reprinting when a printed field changes
  (new insert grade, new measured length).
- **Material dots come from the `material_scope`** (see "Material-scoped libraries"): one colour
  per ISO 513 group (the usual P blue, M yellow, K red, N green, S orange, H grey convention
  is (unverified) here), so the tag, the library and the job's material check agree.
- **The tag rack becomes the drift check.** Scanning the rack left to right (or each tag as it
  goes in) gives the machine's actual tool load, which SolidCraft compares with the locked
  machine library and the program's tool list before the job runs.
- **Load and unload are scan events:** scanning a tag into a rack slot assigns the tool to that
  machine and pocket in the library; scanning it out releases it.

> **[TODO Jason: personal experience]** Optional: whether your holders have the DIN 69873 bore,
> the tag's size and how it attaches to the tool, and the barcode type (1D, QR, DataMatrix).

### Indexable tools: inserts and grades

*Jason's requirement: "Tool library and tool tags system should handle indexable insert grades so
we always know what is in the tool."*

- **The insert's shape and size are standardised, but its grade isn't.** The ISO 1832 code
  (for example `CNMG 120408`) describes only the physical insert: shape `C` (80° diamond),
  clearance `N` (0°), tolerance `M`, type `G`, size `12` (12.7 mm inscribed circle), thickness
  `04` (4.76 mm), corner radius `08` (0.8 mm). The grade, coating and chipbreaker are not
  standardised; the source says "Manufacturer chipbreaker names are not universal ISO codes"
  ([Protool, 2026-08-23](https://www.protool-ltd.co.uk/blogs/blog/iso-turning-insert-nomenclature)).
  So two `CNMG 120408` inserts from different makers, or in different grades from the same
  maker, cut differently.
- **Grades map to ISO 513 application groups.** Grades are classed by work material: P (steel),
  M (stainless), K (cast iron), N (non-ferrous), S (heat-resistant alloys and titanium), H
  (hardened). The six-group system has been in force since 2003
  ([Mitsubishi Materials, carbide classification](https://www.mmc-carbide.com/permanent/courses/78/classification.html));
  ISO 513:2004 is the current edition listed by
  [CCN-SCC](https://ccn-scc.ca/standardsdb/standards/8117151) (unverified that no later edition
  exists). The two-digit sub-scale (e.g. P10 to P40, from wear-resistant to tough) is
  (unverified) from these sources.
- **What changes when an insert changes (opinion):** cutting data (the grade sets the safe
  `vc` range and the material coefficients used), tool-life tracking (life belongs to the
  insert edge, not the body) and sometimes geometry (corner radius `RE`, which changes the tool
  offset and the finish). A holder or cutter body with the wrong insert grade is a silent error
  today, because the CAM program only knows the body.

**Proposal (opinion):**

| Layer | What's stored |
|---|---|
| Catalogue | Indexable body (e.g. face mill, turning holder) with its **insert seat** (ISO 1832 shape/size it accepts, number of seats). Inserts as separate catalogue items: ISO 1832 code + maker + chipbreaker + grade + ISO 513 group + coating. |
| Assembly | The body plus the **insert currently fitted**: catalogue insert ID, quantity, date fitted, which edge is in use (an 80° diamond has 2 usable corners per side, double-sided negative inserts 4) (unverified edge counts per shape). |
| Machine tool | Edge life used, inserts changed history. |
| Tag | Points at the assembly, so scanning shows "CNMG 120408-xx, grade yyyy, ISO P, fitted 2026-10-26, edge 2 of 4". |

- **Changing an insert is a scan event:** scan the tool tag, scan or pick the insert box
  (makers print barcodes/DataMatrix on boxes, e.g.
  [Seco](https://www.secotools.com/article/seco_data_matrix); box code contents (unverified)),
  confirm. The assembly updates, life resets, and if the grade or `RE` changed, programs using
  that tool are flagged for re-checking.
- **Mixed or unknown grades:** if the fitted grade isn't recorded, cutting data falls back to the
  conservative end of the generic class (see "Tools with unknown physics" in the
  [cutting physics file](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-cutting-physics/research/cam-research-nsnlookupjn-cutting-physics.md)).

### Adding tools: AI datasheet reading, custom tools and 3D models

*Jason's requirements: "Can we use ai to read a tool datasheet and add to library" and "Need to
be able to add a custom tool including 3d model".*

**Order of preference for getting a tool in (opinion):**

1. **Shared catalogue hit** by part number: nothing to enter.
2. **Vendor GTC / ISO 13399 package:** exact data, imported without interpretation.
3. **AI reading a datasheet or catalogue page** (PDF, photo of a catalogue page, web page).
4. **Manual entry or a custom tool** with the user's own 3D model.

**AI datasheet reading (opinion on the design; sources on the risks):**

- **The risk is misplaced numbers, not missing ones.** Text extracted from PDFs often loses
  table structure, so a model has to guess which value belongs to which column; one benchmark
  scored a common PDF text extractor 0.000 on table-structure recovery
  ([Nutrient blog, updated 2026-10-26](https://www.nutrient.io/blog/why-your-ai-agent-hallucinates-pdf-table-data/);
  vendor source, marketing for its own extractor). A study extracting tables from government
  PDFs with an LLM, using page images, passed 84 % of its internal sum checks overall and
  recommends algorithmic validation, pointing reviewers at the exact failed row, and human
  review ([Aggarwal et al., arXiv 2511.10659v2, 2025](https://arxiv.org/html/2511.10659v2)).
  Tool catalogues are dense multi-column tables with inch/mm variants, so the same failure modes
  apply (opinion).
- **Safeguards proposed:**
  - Read the page as an image as well as text, and keep a link from each extracted value to
    its location on the page so the reviewer sees the source next to the value.
  - Map every field to an ISO 13399 symbol (`DC`, `APMX`, `LU`, `OAL`, `NOF`, `RE`…) with its
    unit, and convert to mm on entry.
  - **Consistency checks before review:** `OAL ≥ LU ≥ APMX`; `DC` within the series' range;
    shank `DCON` a standard size; flute count an integer; inch values that convert to the
    catalogue's own mm column. Failed checks are highlighted.
  - **Every value carries a confidence label** ("AI-extracted, unreviewed" → "reviewed"), using
    the same labels as the [cutting physics file](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-cutting-physics/research/cam-research-nsnlookupjn-cutting-physics.md) (vendor, measured, generic,
    assumed).
  - **Nothing AI-extracted reaches the shared catalogue without a person approving it**, and the
    entry records the source document and who approved it. Until approved, the tool is usable
    only in the user's own library, and its cutting data is derated like a generic tool.
  - The datasheet's own cutting data (recommended `vc`, `fz`) goes in as a vendor preset with
    its source, not as material coefficients.
- **Licence:** catalogue data is the maker's; whether extracted values can be shared in an open
  catalogue is the same question as GTC redistribution (Open questions).

**Custom tools with a 3D model:**

- **Why the model matters:** collision checking and simulation need the real shape, especially
  for form tools, special step drills, long-neck tools and odd holders. Fusion, for example,
  ignores the STEP model inside GTC packages and uses only the parametric P21 data
  ([Autodesk help: tool library import](https://help.autodesk.com/cloudhelp/ENU/Fusion-CAM/files/MFG-TOOL-LIBRARY-IMPORT.htm)),
  and importing a maker's holder STEP needs a third-party script that converts it to the
  library's JSON
  ([DSI knowledge base, screenshots 2023-07](https://know.dsi-mfg.com/kb/f360-import-holder)).
  Users drawing their own holders is complaint 5 below.
- **Proposal (opinion), using SolidCraft's strength as a CAD program:**
  - A custom tool is a **SolidCraft design**: sketch a half-profile and revolve it, or import a
    STEP from the maker. SolidCraft already models revolved solids, so no separate tool editor
    is needed.
  - On save, the user marks the **cutting part** (faces or the profile segment that cuts) and the
    **non-cutting part** (shank, neck, holder), and the **gauge point** (where the tool meets
    the holder face, on the Z axis). These map to ISO 13399 reference points such as the
    coordinate system at the connection (ISO/TS 13399-50, (unverified) point names).
  - SolidCraft derives the ISO 13399 properties it can (`DC`, `LU`, `OAL`, `DCON`) from the
    model and asks for the rest (flutes, helix, substrate, coating), so physics can fall back
    to a generic class.
  - The profile used for toolpaths and the full 3D model for collision checking are both kept;
    non-revolved tools (e.g. some turning tools) keep the 3D model only.
  - A custom tool can stay private or be **submitted to the shared catalogue** with its model,
    under a licence the submitter chooses.

### Machine-scoped libraries and fixed tool sets

*Jason's requirement: "Tool library should be able to be locked to a particular machine as we
often a certain set of tools only fits one machine or group of machines or a machine is
preloaded with a set load of tools which do not change."*

- **Why:** a tool only fits machines with the same spindle taper (CAT40 vs BT40 vs HSK-A63),
  within their max tool length, diameter and weight, and within the spindle's speed. Many shops
  also keep a **standard tool load** on each machine that stays in the magazine permanently, so
  jobs are programmed against what is already there (opinion; supporting source: the master
  library matched to the controller in the
  [Autodesk forum thread](https://forums.autodesk.com/t5/fusion-manufacture-forum/fusion-tool-libraries-vs-machine-tool-library/td-p/11741597)).
- **Proposal (opinion):**
  - **Machine groups.** A library can be scoped to one machine or a named group of machines
    (e.g. "Haas VF-2 cell, CAT40"). Machine definitions come from the
    [machine library file](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-machine-library/research/cam-research-nsnlookupjn-machine-library.md).
  - **Compatibility is checked, not just labelled:** taper/interface, max tool length, diameter
    and weight, max RPM vs the holder's balance rating. A tool that doesn't fit can't be added
    to that machine's library, with the reason shown.
  - **Fixed load ("locked") library:** a machine's standard tool set with fixed tool numbers and
    pockets. When a job is programmed for that machine, tools are picked from the fixed load
    first; using anything else is flagged as an extra tool to load (and, if pockets are full,
    which tool must come out). The post processor emits the fixed numbers.
  - **Locking is a permission:** only a named role can change a locked library, every change is
    logged, and the change produces a new tool table to send to the machine.
  - **Drift check:** compare the locked library with the machine's actual tool table (exported
    file, or live via the controller) and list differences: wrong tool in a pocket, offset
    changed, insert changed (see above).

### Material-scoped libraries

*Jason's requirement: "In the machine library and tool library be able to lock to one or several
materials. For example many people have a machine that only cuts aluminum 6061 and many people
have a library of tools for only aluminum. It would be possible for a machine or tool library to
be good for a range of materials, but rarely all materials."*

- **Why:** tools bought for one material (for example 2- or 3-flute polished, uncoated or
  DLC-coated carbide for aluminium) are often wrong for others, and some shops keep a machine
  for one alloy so chips and coolant don't mix (opinion; reasons vary by shop and are
  (unverified) as general practice).
- **Proposal (opinion):**
  - A library (tool or machine) carries a **`material_scope`**: a list of entries, each either an
    ISO 513 group or subgroup (P, M, K, N, S, H; e.g. N1 wrought aluminium, (unverified)
    subgroup numbering; see "Indexable tools" above) or a specific grade and temper
    (e.g. 6061-T6, linked to SolidCraft's physical material card). A scope can mix levels:
    "6061-T6, 7075-T6" or "N only". An empty scope means all materials.
  - The scope has a **mode**: `locked` (only these; anything else is refused with the reason) or
    `preferred` (others allowed with a warning).
  - Each tool can also carry its **own** suitable materials (from the vendor's ISO 513 icons, the
    insert grade's group, or the shop's choice); the library's scope is a filter on top.
  - **When a job's stock material is set**, the effective scope is
    machine scope ∩ tool library scope ∩ workholding scope. SolidCraft only offers tools and
    machines inside it; an empty intersection is reported as "no machine/tool set approved for
    this material". Overriding a `preferred` scope is flagged on the setup sheet.
  - **Scope also picks the physics:** the cutting-data sets used for a library are only those
    for its materials, so an aluminium-only library never falls back to steel coefficients.
  - **Locking works like the machine lock:** only named editors can widen a locked scope, and
    changes are logged.
  - **Machine and tool scopes combine:** a job on an aluminium-only machine with a mixed tool
    library sees only the tools whose scope includes the job material. The machine side is in
    the [machine library file](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-machine-library/research/cam-research-nsnlookupjn-machine-library.md).

### Where tool data meets the machine

- Machine controls keep their own tool table. LinuxCNC's `tool.tbl` has tool number `T`, pocket
  `P`, per-axis offsets, diameter `D` and a comment, in machine units
  ([LinuxCNC 2.5 docs](https://linuxcnc.org/docs/2.5/html/gcode/tool_compensation.html)).
- Users keep the CAM library and machine table in step by hand, e.g. one master library whose
  numbers match the controller, copying other tools into it with new numbers
  ([Autodesk forum, 2023-02-08, revived 2026-06-07](https://forums.autodesk.com/t5/fusion-manufacture-forum/fusion-tool-libraries-vs-machine-tool-library/td-p/11741597)).
- The shop-data layer above (tool number, pocket, measured offsets) is what a post processor or
  a future "send tool table" export would write.

**Author's experience (personal experience):** offsets get into the control in one of two ways
today. Either the tool is set to a known length on an offline tool setter and the values are typed
into the machine by hand, or the machine's own tool setter measures length (and optionally
diameter) with its built-in macros. Both need a rough tool length first, taken with a tape
measure, to save setting time.

**What this suggests (opinion):** the library already knows the assembly's nominal length
(catalogue `OAL`/`LF` plus holder gauge length plus stick-out), so it can replace the tape
measure: the tool tag or tool table gives the machine macro a rough length to start probing
from, and the measured result is written back to the assembly. Typing presetter values is the
step a scanned tag plus a tool-table export removes.

## Accuracy

- **Catalogue geometry.** Vendor GTC data is the maker's nominal geometry; real tools differ
  (reground tools, measured diameter). The shop layer must carry *measured* `DC` and length,
  separate from nominal, and collision checks should use the larger envelope (opinion).
- **Cutting coefficients.** Tabulated `kc1.1` values are per broad material group; Rubeo &
  Schmitz show coefficients vary nonlinearly with feed per tooth and with radial immersion
  ([2016](https://mtrc.utk.edu/wp-content/uploads/sites/45/2019/09/mechanistic_force_model_comparison.pdf)).
  Any computed force or power should be shown as an estimate with its source and the range it
  was calibrated over.
- **Stability.** Stability lobes need a measured spindle FRF; without it, SolidCraft can only
  give a conservative depth. RCSA reduces measurements but still needs one per spindle
  ([Schmitz et al., 2023](https://mtrc.utk.edu/wp-content/uploads/sites/45/2022/10/RCSA-review.pdf)).
- **Provenance in a shared catalogue.** Every shared value should say who supplied it (vendor,
  community, shop measurement), when, and for what conditions, so users can judge trust
  (opinion).
- **Verification ideas:** round-trip a GTC package (import → export → diff); compare computed
  spindle power against the vendor's own calculator for a published example (unverified that a
  vendor publishes worked examples with all inputs); compare measured offsets against nominal.

> **[TODO Jason: personal experience]** Cases where vendor feeds/speeds or presets were wrong
> for your machine, and by how much.

## Programs

| Program | Library storage | Sharing | Vendor catalogues | Cutting data | Tool ID |
|---|---|---|---|---|---|
| Fusion | Local and Cloud libraries; `.json`, `.tools` (zipped JSON), `.hsmlib`, `.csv` | Cloud libraries per account/team (behaviour (unverified)) | Imports GTC `.zip` and `.p21` ([Autodesk help](https://help.autodesk.com/cloudhelp/ENU/Fusion-CAM/files/MFG-REF-LIBRARY-EXP-IMP.htm)); downloadable vendor libraries incl. Haas | Presets per material | None found |
| Onshape CAM Studio | Personal libraries; holders and arbors in separate libraries | Import libraries "that other users have shared" ([Onshape](https://www.onshape.com/en/resource-center/tech-tips/machine-tools-cnc-programming-cam-studio)) | (unverified) | (unverified) | None found |
| Mastercam | Tool libraries and assemblies | (unverified) | (unverified) | (unverified) | None found |
| MachiningCloud | Cloud, vendor-neutral catalogue ([MMS, 2017](https://mmsonline.com/blog/post/managing-cutting-tool-data-in-the-cloud)) | Public catalogue | Many makers publish to it; supports ISO 13399/GTC ([MMS news](https://mmsonline.com/news/machiningcloud-provides-iso-13399-gtc-support)) | (unverified) | — |
| Sandvik CoroPlus Tool Library | Cloud | Exports GTC packages ([Sandvik support](https://www.sandvik.coromant.com/en-us/support/coroplus-tool-library-support/export-gtc-package)) | Sandvik group | (unverified) | — |
| Zoller (presetters) + Fastems MMS | Tool management software | Shop | — | Tool life limits | RFID and DataMatrix ([Control Design](https://controldesign.com/industry-news/news/55038158/collaboration-transfers-tool-information-to-mms-software)) |
| LinuxCNC | `tool.tbl` text file | — | — | — | — |
| SolidCraft (open PR #4158) | Library in the data folder, 8 tool kinds, `manufacture.tool.*` and `manufacture.library.*` commands ([PR #4158](https://github.com/tangency-labs/solidcraft/pull/4158), open) | Not described | No | Not described | No |
| SolidCraft (RFC #1153) | Machine-wide `tools.json` beside the recovery folder, overridable by `SOLIDCRAFT_TOOLS` ([#1153](https://github.com/tangency-labs/solidcraft/issues/1153)) | Not described | No | Not described | No |

> **[TODO Jason: personal experience]** What you like and dislike in the tool libraries you've used.

## Complaints and requests

Grouped by theme, most common first. Vote counts are given where the page showed them.

1. **Library and machine drift apart / numbering clashes.** Keeping a CAM library matched to the
   controller's tool table by hand; imported libraries clash on tool numbers
   ([Autodesk forum, 2023, 6 replies](https://forums.autodesk.com/t5/fusion-manufacture-forum/fusion-tool-libraries-vs-machine-tool-library/td-p/11741597)).
2. **Duplicate tools.** Tools saved to a library reappear as separate entries without their
   holder; "about 20 different tool #1s"
   ([eMastercam forum, 2024-11-06, 4 replies](https://www.emastercam.com/forums/topic/110355-duplicate-tools-how-do-you-stop-them-from-occuring/)).
3. **Library edits don't propagate.** Thread titled "Having to update tool library changes in
   multiple places is not…"
   ([Autodesk forum](https://forums.autodesk.com/t5/fusion-manufacture-forum/having-to-update-tool-library-changes-in-multiple-places-is-not/m-p/12250843/highlight/true);
   page could not be fetched, content (unverified)).
4. **Cloud library reliability.** Threads "I give up, is there a way to make cloud tool libraries
   work?" and "Cloud CAMTool library lost"
   ([Autodesk forum](https://forums.autodesk.com/t5/fusion-support-forum/i-give-up-is-there-a-way-to-make-cloud-tool-libraries-work/td-p/7785830);
   [Autodesk forum](https://forums.autodesk.com/t5/fusion-manufacture-forum/cloud-camtool-library-lost/m-p/8476477);
   content (unverified), titles only).
5. **Holders without geometry.** A catalogue holder found by number shows no geometry; the user
   measured and drew their own
   ([Tormach forum, 2023-07-27](https://forums.tormach.com/t/toolholder-models-in-fusion360/1042)).

**Requests implied (opinion):** define once and reference everywhere; offline-first with sync,
not cloud-only; holders with real geometry; one source of truth for tool numbers per machine.

> **[TODO Jason: personal experience]** Your own top complaints, in your words.

## What users need

### Workflow today (typical, from the sources above)

1. Find the tool in a vendor catalogue or website.
2. Re-enter it, or import a vendor library, into the CAM program.
3. Build or pick a holder; often draw it by hand.
4. Set stick-out and tool number to match the machine.
5. Pick or type feeds and speeds from a preset or vendor calculator.
6. Post the program; set the tool up on the machine, measure length, type offsets.
7. Keep the CAM library and the machine table in step by hand.

### Proposed workflow in SolidCraft (opinion)

1. **Search the shared catalogue** by vendor part number (e.g. a Haas Tooling end mill) or by
   ISO 13399 properties (`DC` 12 mm, `NOF` 3, aluminium). If missing, create it once from the
   vendor's GTC package or by hand; it is then available to everyone.
2. **Pick a holder** from the shared holder catalogue (e.g. CAT40 shrink fit by part number).
3. **Make an assembly**: set stick-out; SolidCraft computes reach, collision envelope and, if a
   spindle FRF is on file, a stability map.
4. **Choose material**; SolidCraft looks up the cutting-data set (`kc1.1`, `mc` or edge-force
   coefficients) and computes speed, feed, depth and width from the machine's spindle power and
   torque curve and the tool's limits, instead of copying a preset.
5. **Assign to a machine**: tool number and pocket; export the machine's tool table.
6. **Print a tag** (QR or DataMatrix label) for the assembly; clip it into the holder.
7. **On the shop floor**, scan the tag to open the assembly, record measured length, diameter and
   wear; the library updates.

### Data model sketch (opinion; field names follow ISO 13399 symbols where one exists)

```text
CatalogueItem   id, kind (cutter|indexable_body|insert|holder|extension|generic_class),
                vendor, part_number, gtc_class,
                properties { DC, DCON, APMX, LU, LF, OAL, NOF, RE, ... }  (mm, rad),
                insert { iso1832_code, grade, iso513_group, chipbreaker, coating },
                seats { iso1832_shape_size, count },
                model_ref (STEP or SolidCraft design), profile_ref, gauge_point,
                source (gtc|vendor|ai_extracted|user), confidence, approved_by,
                version, licence
CuttingDataSet  id, material_group, tool_family, model (kienzle|edge_force),
                coefficients { kc11, mc } | { Ktc, Krc, Kac, Kte, Kre, Kae },
                valid_range { fz, ae, vc }, source, date
Assembly        id, cutter -> CatalogueItem, holder -> CatalogueItem, stickout_mm,
                fitted_inserts [ { insert -> CatalogueItem, qty, edge, fitted_at } ],
                measured { DC, length }, notes
MachineGroup    id, machines [machine_id], interface (e.g. CAT40), limits
                { max_length, max_diameter, max_mass, max_rpm }
Library         id, scope (user|shop|machine|machine_group),
                material_scope { mode (locked|preferred),
                  entries [ iso513_group | material_card_ref ] } (empty = all),
                locked, editors, change_log
MachineTool     machine_id, T, pocket, fixed (bool), assembly -> Assembly, offsets,
                life_used (per insert edge for indexable tools)
Tag             tag_id -> Assembly | MachineTool
```

### Formats involved

GTC packages (`.zip` with ISO 13399 data, `.p21`, 3D models); vendor CAM libraries (e.g.
Fusion `.json`/`.tools`); machine tool tables (e.g. LinuxCNC `tool.tbl`); label printing
(QR/DataMatrix).

### Where time is lost

Re-entering tools; drawing holders; reconciling tool numbers with the machine; trusting presets
that don't match the engagement; typing offsets.

### Fit with SolidCraft's current CAM work

The tool library in PR #4158 and the `tools.json` in #1153 are local, per-machine files. The
shared catalogue proposed here would sit *above* them: local libraries keep assemblies and
machine tools, and reference shared catalogue IDs. It should work offline (cached catalogue) and
not require an account for local use (opinion).

The same catalogue / shop-instance / tag pattern is proposed for workholding (vises, chucks,
fixture plates) in the companion file
[cam-research-nsnlookupjn-workholding.md](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-workholding/research/cam-research-nsnlookupjn-workholding.md);
vise clamping-torque curves could sit in the shared catalogue next to holders (opinion).

> **[TODO Jason: personal experience]** Your own step-by-step workflow from new job to first cut,
> and which step costs the most time.

## Glossary

| Term | Other names | Definition | Note |
|---|---|---|---|
| Tool library | tool database, tool crib list | A stored set of tool definitions available for programming. | Here split into shared catalogue and shop library. |
| Catalogue item | component, tool item | A single tool or holder component described by its maker's properties. | Example: one end mill part number. |
| Holder | tool holder, adaptive item, arbor | A component that connects a cutter to the machine spindle. | Examples: CAT40, BT40, HSK-A63. |
| Assembly | tool assembly, tool | A cutter mounted in a holder at a set stick-out. | The unit that is measured and loaded into a machine. |
| Stick-out | gauge length, projection | The length a cutter extends from the holder face. | Drives reach, collision and stiffness. |
| ISO 13399 | — | An international standard for representing and exchanging cutting tool data. | See Sources. |
| GTC | Generic Tool Catalog | A vendor-neutral classification and package format for tool data built on ISO 13399. | Tool makers publish GTC packages. |
| Preset | cutting preset, feeds and speeds preset | A fixed set of speed, feed and engagement values for one tool and material. | Valid only near the conditions it was written for. |
| Specific cutting force | kc, specific cutting pressure | The cutting force per unit uncut chip area. | Depends on chip thickness. |
| kc1.1 | ks11 | The specific cutting force at 1 mm chip thickness and 1 mm chip width. | Unit N/mm². |
| mc | z | The exponent describing how specific cutting force changes with chip thickness. | Dimensionless. |
| Edge-force coefficients | mechanistic coefficients | The six coefficients relating milling forces to chip area and chip width. | Calibrated by test cuts. |
| Tool-tip FRF | tool point frequency response | The dynamic response of the tool tip to a force over frequency. | Input to stability lobes. |
| Stability lobe diagram | SLD, chatter map | A chart of the largest chatter-free depth of cut against spindle speed. | Needs the tool-tip FRF. |
| RCSA | receptance coupling | A method that predicts the tool-tip FRF by joining modelled and measured component responses. | Spindle measured once. |
| Tool tag | tool label, data carrier | A physical marker on a holder that identifies an assembly. | RFID chip or printed code. |
| Data carrier bore | chip bore | A standardised hole in a holder for mounting an identification chip. | DIN 69873. |
| Tool table | offset table, tool.tbl | The controller's list of tool numbers, pockets and offsets. | Per machine. |
| Indexable tool | insert tool | A tool body that carries replaceable cutting inserts. | Face mills, turning holders. |
| Insert | indexable insert, tip | A replaceable cutting piece clamped in an indexable tool body. | Described by ISO 1832 code plus grade. |
| Grade | carbide grade, insert grade | A maker's name for an insert's substrate and coating combination. | Not standardised; classed by ISO 513 group. |
| ISO 513 group | application group, P/M/K/N/S/H | A letter class of work materials a cutting material is meant for. | P steel, M stainless, K cast iron, N non-ferrous, S superalloys, H hardened. |
| Machine group | machine cell | A named set of machines that share an interface and limits. | A library can be scoped to it. |
| Fixed tool load | standard tool load, permanent tools | A set of tools kept in a machine's magazine across jobs. | Programmed against first. |
| Material scope | material lock | The set of work materials a library or machine is allowed to be used for. | Empty means any material. |
| Gauge point | gauge line, tool reference point | The point on the tool axis where tool length is measured from. | Usually the holder face. |

## Open questions

1. GTC package internals: exact directory layout and which ISO 13399 parts are mandatory. Look
   next at gtc-tools.com and a free vendor GTC download.
2. Licence of vendor GTC data: can it be redistributed in a shared open catalogue, or only
   linked? Fusion shows a legal disclaimer on some GTC packages
   ([Autodesk help](https://help.autodesk.com/cloudhelp/ENU/Fusion-CAM/files/MFG-REF-LIBRARY-EXP-IMP.htm)).
   This decides whether SolidCraft hosts copies or only references.
3. Whether any public, openly licensed dataset of `kc1.1`/`mc` or edge-force coefficients
   exists beyond textbook tables. Candidate: dry cutting dataset for Ti6Al4V and Ck45
   ([arXiv 2209.04197](https://arxiv.org/pdf/2209.04197)), licence (unverified).
4. Hosting and governance of a shared catalogue: GitHub repository of JSON files with review,
   versus a hosted service. Who approves edits?
5. Which Haas Tooling part numbers and holders are in the public Haas/Fusion library, and can
   their geometry be used. Look at the Haas Tooling web store and the Fusion download page.
6. Fit, label size and licence of the MakerWorld label holder (page could not be fetched).
7. Which holder types carry the DIN 69873 bore by default (CAT40, BT40, HSK-A63).
8. What insert makers encode in the barcodes or DataMatrix codes on insert boxes (part number
   only, or grade and batch too), and whether it is documented publicly.
9. Whether vendor datasheet values extracted by AI can be shared in an open catalogue
   (same licence question as 2).
10. How controllers expose the current tool table for a drift check (file export, MTConnect,
    maker API); see the machine library file.
11. Which material naming to use for scopes (ISO 513 groups, UNS/EN alloy numbers, or a shared
    material list), so tool, machine and workholding libraries agree.

> **[TODO Jason: personal experience]** Anything you know that answers the above.

## Sources

### Standards
- ISO 13399 series, Cutting tool data representation and exchange: parts 1 (2006), 2, 3, 50
  (2013), 80 (2014); catalogue listings at
  [BSI Knowledge](https://knowledge.bsigroup.com/products/cutting-tool-data-representation-and-exchange-reference-dictionary-for-the-cutting-items),
  [SAI Global](https://infostore.saiglobal.com/en-au/standards/iso-ts-13399-50-2013-595585_saig_iso_iso_1364355/),
  [INN](https://ecommerce.inn.cl/isots-13399-80201449355).
- DIN 69873:1993-07, Data media for tools and clamping devices
  ([DIN Media](https://dinmedia.de/en/standard/din-69873/2113394)).
- ISO 1832, Indexable inserts for cutting tools: designation (described from
  [Protool, 2026-08-23](https://www.protool-ltd.co.uk/blogs/blog/iso-turning-insert-nomenclature)).
- ISO 513:2004, Classification and application of hard cutting materials
  ([CCN-SCC listing](https://ccn-scc.ca/standardsdb/standards/8117151)).

### Books and papers
- Kienzle, O., "Die Bestimmung von Kräften und Leistungen an spanenden Werkzeugen und
  Werkzeugmaschinen", VDI-Z 94, 1952 (unverified pages).
- Taylor, F. W., "On the art of cutting metals", Trans. ASME 28, 1907.
- Altintas, Y., Budak, E., "Analytical prediction of stability lobes in milling", CIRP Annals
  44(1), 1995.
- Altintas, Y., *Manufacturing Automation*, 2nd ed., Cambridge University Press, 2012.
- Rubeo, M. A., Schmitz, T. L., mechanistic force model coefficient comparison, Precision
  Engineering, 2016 ([PDF](https://mtrc.utk.edu/wp-content/uploads/sites/45/2019/09/mechanistic_force_model_comparison.pdf)).
- Schmitz, T. et al., "Review and status of tool tip frequency response function prediction
  using receptance coupling", Precision Engineering 79, 2023
  ([PDF](https://mtrc.utk.edu/wp-content/uploads/sites/45/2022/10/RCSA-review.pdf)).
- US Patent 10,896,363, Franz Haimer Maschinenbau KG, 2021
  ([Justia](https://patents.justia.com/patent/10896363)).

### Programs' documentation and vendor pages
- [Autodesk: Import and export tools and tool libraries](https://help.autodesk.com/cloudhelp/ENU/Fusion-CAM/files/MFG-REF-LIBRARY-EXP-IMP.htm)
- [Autodesk Fusion blog: Haas Tooling in the Fusion 360 Tool Library (2022-02-02)](https://www.autodesk.com/products/fusion-360/blog/haas-tooling-fusion-360-tool-library/)
- [Onshape: CAM Studio machine tools tech tip](https://www.onshape.com/en/resource-center/tech-tips/machine-tools-cnc-programming-cam-studio)
- [Sandvik Coromant: Export GTC package](https://www.sandvik.coromant.com/en-us/support/coroplus-tool-library-support/export-gtc-package)
- [Mitsubishi Materials: ISO 13399 properties](https://www.mmc-carbide.com/us/technical_information/iso/iso13399_property)
- [Balluff: Automated tool management with RFID](https://www.balluff.com/en-us/application-examples-and-solutions/automated-tool-management-with-rfid)
- [Seco Tools: Data Matrix codes](https://www.secotools.com/article/seco_data_matrix)
- [LinuxCNC 2.5: Tool compensation and tool table](https://linuxcnc.org/docs/2.5/html/gcode/tool_compensation.html)
- [Autodesk: Tool library import (GTC STEP models not used)](https://help.autodesk.com/cloudhelp/ENU/Fusion-CAM/files/MFG-TOOL-LIBRARY-IMPORT.htm)
- [DSI: Importing a holder into Fusion](https://know.dsi-mfg.com/kb/f360-import-holder)
- [Mitsubishi Materials: carbide classification](https://www.mmc-carbide.com/permanent/courses/78/classification.html)
- [Protool: ISO turning insert nomenclature (2026-08-23)](https://www.protool-ltd.co.uk/blogs/blog/iso-turning-insert-nomenclature)
- [Nutrient: Why your AI agent hallucinates PDF table data](https://www.nutrient.io/blog/why-your-ai-agent-hallucinates-pdf-table-data/)
- Aggarwal, V. et al., "Information Extraction From Fiscal Documents Using LLMs", arXiv
  2511.10659v2, 2025 ([HTML](https://arxiv.org/html/2511.10659v2))
- Companion research in this fork:
  [cutting physics](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-cutting-physics/research/cam-research-nsnlookupjn-cutting-physics.md),
  [machine library](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-machine-library/research/cam-research-nsnlookupjn-machine-library.md),
  [workholding](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-workholding/research/cam-research-nsnlookupjn-workholding.md)

### Trade press
- [Machinery: vendor-neutral cutting tools catalogue structure (GTC)](https://www.machinery.co.uk/content/news/vendor-neutral-cutting-tools-catalogue-structure-supports-standardised-data-exchange-with-plm-and-cadcam-systems)
- [Modern Machine Shop: Standard tool classification (2016-01-01)](https://mmsonline.com/articles/standard-tool-classification-for-better-data-communication)
- [Modern Machine Shop: Managing cutting tool data in the cloud (2017-07-28)](https://mmsonline.com/blog/post/managing-cutting-tool-data-in-the-cloud)
- [Modern Machine Shop: MachiningCloud provides ISO 13399/GTC support](https://mmsonline.com/news/machiningcloud-provides-iso-13399-gtc-support)
- [Control Design: Zoller and Fastems (2024-05-13)](https://controldesign.com/industry-news/news/55038158/collaboration-transfers-tool-information-to-mms-software)

### Community sources
- [Autodesk forum: Fusion tool libraries vs machine tool library](https://forums.autodesk.com/t5/fusion-manufacture-forum/fusion-tool-libraries-vs-machine-tool-library/td-p/11741597)
- [Autodesk forum: tool library changes in multiple places](https://forums.autodesk.com/t5/fusion-manufacture-forum/having-to-update-tool-library-changes-in-multiple-places-is-not/m-p/12250843/highlight/true)
- [Autodesk forum: cloud tool libraries](https://forums.autodesk.com/t5/fusion-support-forum/i-give-up-is-there-a-way-to-make-cloud-tool-libraries-work/td-p/7785830)
- [Autodesk forum: Cloud CAMTool library lost](https://forums.autodesk.com/t5/fusion-manufacture-forum/cloud-camtool-library-lost/m-p/8476477)
- [eMastercam forum: duplicate tools](https://www.emastercam.com/forums/topic/110355-duplicate-tools-how-do-you-stop-them-from-occuring/)
- [Tormach forum: toolholder models in Fusion 360](https://forums.tormach.com/t/toolholder-models-in-fusion360/1042)
- [MakerWorld: HSK63/SK40 data-holder label holder](https://makerworld.com/en/models/1291677-hsk63_sk_40_dataholder-label-holder)
- SolidCraft: [#1153](https://github.com/tangency-labs/solidcraft/issues/1153),
  [#1621](https://github.com/tangency-labs/solidcraft/issues/1621),
  [PR #4158](https://github.com/tangency-labs/solidcraft/pull/4158)
