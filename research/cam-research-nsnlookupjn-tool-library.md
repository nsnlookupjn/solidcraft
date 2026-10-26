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

> **[TODO Jason: personal experience]** Which CAM programs you have used (for example Fusion,
> Mastercam, others), on which machines (for example Haas mills), and for how long. How your
> shop manages tools today: where the tool list lives, who enters tool data, how tools are
> labelled, and what goes wrong. Label this as personal experience.

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

> **[TODO Jason: personal experience]** How you label tools today, whether your holders have the
> DIN 69873 bore, whether you have used the MakerWorld label holder, label printer used and label
> size, and what you would want to see when you scan a tool.

### Where tool data meets the machine

- Machine controls keep their own tool table. LinuxCNC's `tool.tbl` has tool number `T`, pocket
  `P`, per-axis offsets, diameter `D` and a comment, in machine units
  ([LinuxCNC 2.5 docs](https://linuxcnc.org/docs/2.5/html/gcode/tool_compensation.html)).
- Users keep the CAM library and machine table in step by hand, e.g. one master library whose
  numbers match the controller, copying other tools into it with new numbers
  ([Autodesk forum, 2023-02-08, revived 2026-06-07](https://forums.autodesk.com/t5/fusion-manufacture-forum/fusion-tool-libraries-vs-machine-tool-library/td-p/11741597)).
- The shop-data layer above (tool number, pocket, measured offsets) is what a post processor or
  a future "send tool table" export would write.

> **[TODO Jason: personal experience]** On your Haas machines: how offsets get into the control
> (typed, presetter, probe), and whether you would want SolidCraft to export a Haas offset table.

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
CatalogueItem   id, kind (cutter|holder|extension), vendor, part_number, gtc_class,
                properties { DC, DCON, APMX, LU, LF, OAL, NOF, RE, ... }  (mm, rad),
                model_ref (STEP), source, version, licence
CuttingDataSet  id, material_group, tool_family, model (kienzle|edge_force),
                coefficients { kc11, mc } | { Ktc, Krc, Kac, Kte, Kre, Kae },
                valid_range { fz, ae, vc }, source, date
Assembly        id, cutter -> CatalogueItem, holder -> CatalogueItem, stickout_mm,
                measured { DC, length }, notes
MachineTool     machine_id, T, pocket, assembly -> Assembly, offsets, life_used
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
