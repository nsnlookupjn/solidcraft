# Parts research: a military/aerospace part library, or "insert a part by its NATO Stock Number"

Research for [#787](https://github.com/tangency-labs/solidcraft/issues/787) (Add: Standard
Components Library), with links to the fastener-family work in
[#2071](https://github.com/tangency-labs/solidcraft/issues/2071) / draft PR
[#2074](https://github.com/tangency-labs/solidcraft/pull/2074) and the library contract in
[`docs/book/src/dev/libraries.md`](../docs/book/src/dev/libraries.md). This file covers one idea:
**a library of military and aerospace parts that a user adds to an assembly by typing a NATO
Stock Number (NSN)**, and a way for users to **contribute models, dimensions and properties for
NSNs that do not have them yet**. Other component-library topics (structural sections, wood,
appearances) are out of scope.

It shares a pattern with two other files from the same author: the
[shared cloud tool library](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-tool-library/research/cam-research-nsnlookupjn-tool-library.md)
and the [workholding library](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-workholding/research/cam-research-nsnlookupjn-workholding.md)
(#1621). All three are "a shared, reviewed catalogue of real-world items that SolidCraft copies
into a design"; see "Shared-library pattern" below.

Licence: MIT OR Apache-2.0, the same as SolidCraft (see the Licence section of CONTRIBUTING.md).

## About this research

- **Author:** Jason ([@nsnlookupjn](https://github.com/nsnlookupjn)). Idea and direction by Jason;
  source research and writing done with an AI assistant (Claude) in October 2026.
- **How it was produced:** web searches of public sources (Defense Logistics Agency (DLA) pages and
  brochures, NATO's codification pages, public specification indexes, commercial NSN sites'
  public pages, university export-control guides), each claim checked against the cited page
  where possible. No commercial CAD program was driven or scripted, no GPL/LGPL/AGPL source was
  read, and no text is copied beyond short quotes. Standards are described from their public
  index entries, not their paid text. SolidCraft's own code (`crates/engine/src/cmd/parts.rs`,
  `component_properties.rs`, `bom.rs`) and docs were read to fit the proposal to what exists.
- **Not legal advice.** The export-control and copyright sections summarise public guidance so
  maintainers know which questions to ask. They are not a ruling on any item.
- **Units:** millimetres and radians, as in SolidCraft. FLIS characteristic values are mostly
  in inches; they are converted at 1 in = 25.4 mm.
- **Marking:** "(unverified)" marks a claim not confirmed against a source. **Fact**,
  **opinion** and **marketing** are labelled where they mix.

> **Disclosure of interest.** The author, Jason, is the **owner and operator of NSN Lookup Inc**
> (nsnlookup.com), one of the NSN data providers compared in this file. The offer that "NSN
> Lookup could make this free for SolidCraft users" comes from him in that role. For that reason
> the provider comparison below is written to stand on public facts, keeps an offline,
> government-sourced baseline that needs no vendor, and lists the questions maintainers should
> put to any provider, NSN Lookup included.

> **Personal experience (Jason).** Running NSN Lookup has given me a deep understanding of how
> the militaries in NATO specify, manufacture, test, inspect (QC) and use their parts and
> equipment.
>
> **[TODO Jason: optional detail]** Concrete examples would help maintainers: where NSNs show up
> in that work, how users of NSN Lookup find a part's dimensions today, and which parts people
> most often wish they could drop into a model.

## Summary

1. **The idea.** In military and aerospace work, parts lists, technical manuals and supply
   requisitions identify parts by **NSN** (13 digits, e.g. `5306-01-509-0781`). Today a designer
   who has an NSN has to look it up on a website, find the spec or part number behind it, read
   the dimensions, then model the part or hunt for a CAD file. The proposal: type the NSN in
   SolidCraft, see what the item is, and insert a correctly sized model into the assembly, with
   the NSN, part number, manufacturer code (CAGE) and description recorded on the component so
   the bill of materials (BOM) carries them.
2. **The data exists and much of it is free.** The US catalogue behind NSNs is the **Federal
   Logistics Information System (FLIS)**, run by DLA. Its publicly releasable part is published
   monthly as **PUB LOG**, free to download with no account, in CSV files: item names, supply
   classes, part numbers with CAGE codes, and **characteristics** (coded dimension and
   material fields) ([DLA PUB LOG](https://www.dla.mil/Customer-Support/Brochures/Details/Article/2893128/public-logistics-pub-log-data/),
   [FLIS e-Reading Room](https://www.dla.mil/Information-Operations/FLIS-Data-Electronic-Reading-Room/)).
   Commercial sites, including nsnlookup.com, re-present this data with search and extras.
3. **Three kinds of NSN, three kinds of model.**
   - *Spec-defined standard hardware* (AN, MS, NAS, NASM bolts, screws, nuts, washers, rivets,
     pins): the part number encodes family, size and length, so a **parametric generator** can
     build it, as SolidCraft's `parts.insert` already does for ISO metric fasteners.
   - *Fully described items* whose FLIS record lists dimensions (thread diameter, threads per
     inch, length, head height, width across flats): the **characteristics** themselves can
     drive the same generator, even with no spec in hand.
   - *Reference-only or proprietary items* (an OEM part number and CAGE, no public dimensions):
     no model can be generated. SolidCraft can insert a labelled **placeholder** and let a user
     **contribute** a model.
4. **Community contributions fill the gaps (Jason's addition).** Most NSNs will have no model.
   Users should be able to contribute a model, dimensions and properties for an NSN, through a
   reviewed process with provenance, licence and confidence levels, checked automatically
   against the FLIS characteristics where those exist. This mirrors the shared tool library's
   contribution model.
5. **The hard constraints are legal, not geometric.** Copyright on standards (NAS/NASM are sold
   by the Aerospace Industries Association), supplier CAD terms, and **US export controls (ITAR,
   EAR)** on technical data decide what may be shared. Public NSN data and published standard
   hardware are generally fine; OEM drawings and models of defense articles are not. FLIS's own
   **DEMIL code** gives a usable first screen.
6. **Data provider: keep it neutral (opinion).** SolidCraft should define one provider interface
   with an **offline PUB LOG extract as the baseline** and online providers (nsnlookup.com,
   OpenFLIS and others) as optional plug-ins the user turns on. NSN Lookup's offer (made by the
   author as its owner; see the disclosure above) to make its
   data free for SolidCraft users is a real option; its value, terms and risks are laid out in
   "Data providers" below.

## Fundamentals

### What an NSN is

**Fact** ([Wikipedia, NATO Stock Number](https://en.wikipedia.org/wiki/NATO_Stock_Number);
[NATO NCS fact sheet](https://www.nato.int/structur/ac/135/data/pdf/NCS-Fact-Sheet.pdf);
[DLA NSN booklet](https://www.dla.mil/Portals/104/Documents/J3LogisticOperations/Brochures/J3_nsnbooklet_200323.pdf)):

```
5306 - 01 - 509 - 0781
└┬─┘   └┬┘   └────┬───┘
 FSC    NCB   item serial
        └──────┬───────┘
              NIIN
```

| Part | Digits | Meaning |
|---|---|---|
| FSC (Federal / NATO Supply Class) | 4 | What kind of item. The first two digits are the **group** (FSG), e.g. 53 = hardware and abrasives; the class narrows it, e.g. 5306 = bolts. |
| NCB code | 2 | The **National Codification Bureau** that codified the item: 00 and 01 = USA, 12 = Germany, 14 = France, 21 = Canada, 66 = Australia, 99 = UK, and so on (about 60 countries). |
| Item serial | 7 | Non-significant serial number. |
| NIIN | 9 (NCB + serial) | **National Item Identification Number**: unique on its own. The FSC can change when an item is reclassified; the NIIN does not. |

Design consequences:

- **Key the library on the NIIN**, not the full NSN, and store the FSC alongside it (the FSC is a
  classification that can change).
- Accept input with or without dashes, and a bare NIIN.
- The **NATO Codification System (NCS)** covers more than 60 nations; the
  [NATO Master Catalogue of References for Logistics (NMCRL)](https://www.nato.int/structur/AC/135/nmcrl/data/nsn)
  is the official NATO-wide catalogue. PUB LOG **excludes NATO data**, so a free, offline
  library starts as a library of **US-codified** NSNs (NCB 00/01). Non-US NSNs need NMCRL or a
  national bureau (access terms vary by country; unverified for each).

FSCs most relevant to a first library (hardware an assembly needs most often):

| FSC | Name (H2 handbook) |
|---|---|
| 5305 | Screws |
| 5306 | Bolts |
| 5307 | Studs |
| 5310 | Nuts and washers |
| 5315 | Nails, machine keys and pins |
| 5320 | Rivets |
| 5325 | Fastening devices |
| 5330 | Packing and gasket materials (O-rings) |
| 3110 | Bearings, antifriction, unmounted |
| 4730 | Hose, pipe, tube, lubrication and railing fittings |
| 5935 | Connectors, electrical |

### What else FLIS knows about an item

**Fact** ([DLA self-help tools comparison](https://www.dla.mil/Customer-Support/Brochures/Details/Article/2893363/self-help-tools-side-by-side-comparison/)):
PUB LOG, FED LOG and WebFLIS all carry characteristics, CAGE codes, reference (part) numbers,
DEMIL codes and unit price. PUB LOG also carries the H-series handbooks: H2 (supply classes),
H5 (company data) and H6 (approved item names). It **excludes** restricted data: service-unique
data, weapon-system data, proprietary data and NATO data.

| Record | What it gives a CAD library |
|---|---|
| **Item name** (H6, e.g. "BOLT, MACHINE") | The family to generate, or the placeholder's label. |
| **Reference numbers** (part number + CAGE) | The spec part number (e.g. `NASM3-5A`, `MS24693C4`) or the OEM's number. A spec part number is the key to a parametric model. |
| **CAGE** (Commercial and Government Entity code, 5 characters; NCAGE outside the US) | Who designs or makes it. A government CAGE on a spec number means "this is a standard part". |
| **Characteristics** (Master Requirement Codes, MRCs) | Coded fields such as nominal thread diameter, threads per inch, length, head height, width across flats, material. Enough to build many fasteners directly. |
| **DEMIL code** | Whether the item is on the US Munitions List or the Commerce Control List (see "Export control"). |
| **Unit price, unit of issue** | BOM cost roll-up (optional). |

Worked example: what FLIS characteristics give a bolt generator. The values below are shown
for NSN 5306-01-509-0781 on a public FLIS viewer
([HigherGov](https://www.highergov.com/nsn/5306015090781/)); the MRC code names are FLIS's.

| MRC | Requirement | Value (as published) | In mm | Generator parameter |
|---|---|---|---|---|
| AHYM | Nominal thread diameter | 1.000 in | 25.40 | `d` |
| CMLP | Threads per inch | 8 | pitch 3.175 | `pitch` |
| THSD | Thread series | UNC | | thread series (inch) |
| AAJD | Thread class | 2A | | tolerance class (metadata) |
| AASB | Fastener length | 2.469–2.531 in | 62.71–64.29 | `length` (use nominal 63.5) |
| AASA | Thread length | 2.500 in nominal | 63.50 | `thread_length` (full thread) |
| AATB | Width across flats | 0.993–1.004 in | 25.22–25.50 | head `af` |
| AASU | Head height | 0.998–1.000 in | 25.35–25.40 | head `k` |
| AATC / AATD | Washer face diameter / thickness | 1.419–1.500 / 0.396–0.416 in | 36.04–38.10 / 10.06–10.57 | head flange |
| MATT / MDCL | Material / material spec | Nickel alloy 718, AMS 5662 | | material assignment (density) |
| DMIL | DEMIL code | A | | export screen: no DEMIL required |

So a fully described record is close to a complete parameter set. Where FLIS gives a range,
the generator should use the midpoint (or the nominal, where one is given) and keep the range
as a tolerance on the component.

### Standard hardware specifications

**Fact** unless marked:

| Prefix | Owner and status | Access |
|---|---|---|
| **AN** (Air Force–Navy) | Old joint standards. Many cancelled and superseded by NASM with the **same part numbers**, e.g. AN3–AN20 aircraft bolts cancelled 1999, superseded by NASM3–NASM20 ([EverySpec, AN3 thru AN20 notice](https://everyspec.com/AN-SPECS/AN100-AN999/AN3_THRU_AN20_REV-12_NOTICE-1_27719/)). | Cancelled US government documents; copies on public spec archives. |
| **MS** (Military Standard sheets, e.g. MS24693, MS21042, MS27039) | DoD; many still active, many transferred to NASM. | DoD's [ASSIST Quick Search](https://quicksearch.dla.mil/qsOverview.aspx) is public; documents cleared for public release download free. |
| **NAS** (National Aerospace Standard, e.g. NAS1352, NAS6203, NAS1149) | Aerospace Industries Association (AIA). | **Sold**, copyrighted ([example listing](https://shop.standards.ie/en-ie/standards/nasm21266-2016-92052_saig_aia_aia_192905)). |
| **NASM** (MS standards converted to NAS) | AIA. Part numbers unchanged from MS/AN. | **Sold**, copyrighted. |
| **SAE AS / AMS** | SAE International. AMS are material specs (e.g. AMS 5662 above). | Sold. |
| **FED-STD, A-A (Commercial Item Descriptions)** | US government. | ASSIST, public. |

Part numbers encode the geometry. For example, in the AN bolt scheme the number after "AN"
is the diameter in sixteenths of an inch, the dash number is the grip length (first digit
inches, last digit eighths) and a suffix letter marks a drilled or undrilled shank, so
`AN4-12A` would be a 1/4-28 bolt with a 1 1/4 in grip and undrilled shank (unverified against
the AN3–AN20 sheet; the generator must be built from the sheet's own tables).

**Copyright and the library (opinion, following SolidCraft's own rule).**
[`libraries.md`](../docs/book/src/dev/libraries.md) already says: "identifying a standard does
not grant redistribution rights to its published size tables." US government works (AN, MS,
FED-STD, FLIS data) are generally not under copyright in the US (17 U.S.C. § 105), so their
tables can feed generators after review. NAS/NASM/SAE tables cannot be copied into the
repository. Two clean routes for NASM parts: (a) because part numbers did not change, use the
**last public-domain MS/AN revision** of the same sheet where the dimensions have not changed
(needs a reviewer to confirm per sheet), or (b) use the **FLIS characteristics** of the NSN,
which are government data.

### Shapes SolidCraft can already make

`parts.library` / `parts.insert` / `parts.resize` (`crates/engine/src/cmd/parts.rs`) generate
seven built-in families: socket head cap screw, hex bolt, hex nut, washer, set screw, dowel pin
and bearing envelope, in ISO metric sizes. Each insert records `family`, `size`, `length` on
the component and sets its part number and description. Draft PR #2074 adds JSON fastener
families (`solidcraft-parts/1`) with heads, lengths, grip ranges and a per-length **article
number**. An NSN library needs, on top of that:

- **Inch threads** (UNC, UNF, UNJF) and aerospace heads (12-point, washer face, 100° flush, pan).
- **Families for AN/MS/NAS hardware**: bolts, screws, nuts (incl. self-locking), washers,
  rivets (solid and blind), pins, O-rings.
- **A link from NSN to family parameters** (the "NSN record" below).
- **A placeholder** family for items with no geometry: a labelled box or cylinder from known
  envelope dimensions, clearly shown as a placeholder (libraries.md: envelope models must stay
  distinguishable from detailed ones).

Open-source fastener generators exist in other CAD ecosystems, but several are GPL/LGPL or
built on CadQuery/OCP, which SolidCraft's rules forbid reading. **None were read for this
file**; SolidCraft's own generator is the base.

### How data maps to a model: three tiers

| Tier | When | Model | Confidence label |
|---|---|---|---|
| **1. Spec-generated** | Reference number is a recognised standard part number (AN/MS/NAS/NASM) with a reviewed family recipe | Parametric, exact to the spec's nominal dimensions | "Standard: <spec>" |
| **2. Characteristic-generated** | FLIS characteristics cover the family's required parameters | Parametric from MRC values | "From FLIS characteristics" |
| **3. Contributed or placeholder** | Only OEM part number + CAGE | Community model if one exists and passed review; else envelope placeholder | "Contributed (level n)" or "Placeholder" |

Tiers 1 and 2 should cross-check each other: when both exist and disagree beyond tolerance,
show the discrepancy instead of picking one silently.

## Accuracy

- **FLIS data quality varies.** Characteristics are entered by cataloguers to Federal Item
  Identification Guides; older or reference-type items may have few or no characteristics, or
  values that conflict with the current spec revision (unverified rate; worth measuring on a
  sample of FSC 5306).
- **Ranges, not nominals.** Many characteristics are min/max. Use the midpoint for geometry;
  keep the range for fit checks.
- **Threads are cosmetic.** SolidCraft's current fasteners model threads as cosmetic; that is
  right for assemblies and BOMs. Interference checks should use the major diameter.
- **Spec revisions.** A part number can survive several spec revisions. Record the spec and
  revision used to build the model.
- **Contributed models** are only as good as their source; see the confidence levels below.

## Programs

How NSN data is used today, from public pages (no programs were run):

| Program / source | What it does | Access |
|---|---|---|
| [PUB LOG](https://www.dla.mil/Customer-Support/Brochures/Details/Article/2893128/public-logistics-pub-log-data/) (DLA) | Monthly public FLIS extract; desktop search tool plus CSV files with layout files. Replaced the FLIS legacy files on 1 April 2023. | Free, no account, no CAC. |
| FED LOG (DLA) | Fuller offline FLIS for government. | .mil/.gov and CAC holders. |
| [WebFLIS](https://www.dla.mil/Customer-Support/Training/Details/Article/2925963/webflis-federal-logistics-information-system/) (DLA) | Online FLIS, updated several times a day. | Mainly government and sponsored contractors. |
| [NMCRL](https://www.nato.int/structur/AC/135/nmcrl/data/nsn) (NATO) | NATO-wide NSN and NCAGE catalogue. | Account; terms per country (unverified). |
| [nsnlookup.com](https://www.nsnlookup.com/) | Search by NSN/NIIN; says it covers 37+ million NSNs with characteristics, technical manuals and supply data; earns affiliate commissions on merchant links. No public API page found. Owned by this file's author (see disclosure). | Free web search. |
| [OpenFLIS](https://openflis.com/) (Synergetics Inc.) | JSON API over 50+ FLIS tables by NIIN, CAGE, FSC; public, releasable FLIS data only; free plan with monthly API credits. | Free tier + enterprise. |
| [Niinja](https://niinja.co/Welcome/Developers) | NSN/CAGE API, pay as you go. **Going offline December 2026**, moving into OpenFLIS. | Paid. |
| [ISO Group](https://iso-dlms.com/Help) | API with demand, availability and risk indexes per NSN. | Commercial. |
| [ASSIST Quick Search](https://quicksearch.dla.mil/qsOverview.aspx) | DoD specs, standards, MS drawings; public-release documents downloadable. | Public. |
| DLA cFolders / JCP | Export-controlled drawings and technical data packages for bidders. | Joint Certification Program (DD Form 2345) plus DLA Enhanced Validation; US/Canada companies only ([NFFS](https://www.nffs.org/news/jcpejcp-fundamentals-what-every-foundry-should-know)). |

**Observation:** in the sources reviewed, no CAD program inserts a part by NSN. NSN sites stop
at data; CAD part libraries stop at supplier part numbers. That gap is the opportunity.

## Data providers: the NSN Lookup offer and the alternatives

Jason's proposal, as owner of NSN Lookup Inc: NSN Lookup could make its data or API free for
SolidCraft users. Because the proposer has a commercial interest, maintainers should weigh it
on its written terms like any other provider. Presented
neutrally, alongside the alternatives (opinion throughout this section):

| Option | For | Against / questions |
|---|---|---|
| **A. Offline PUB LOG extract** built by a SolidCraft tool into a library pack (e.g. FSC 53xx only) | Free; government source; no account; works offline and in the browser; no lookups leave the user's machine; monthly refresh is easy to automate. | Size (millions of rows; a hardware-only pack must be measured); US-only; no images or manuals; someone must maintain the build script. |
| **B. NSN Lookup API, free for SolidCraft users** | Curated search; may add images, manuals, cross-references beyond PUB LOG; offered by its owner, who can answer the questions in the next column directly. | Needs a published API and written terms: rate limits, whether results may be **cached in packs and saved in designs** (libraries.md requires designs to open offline), attribution, how long "free" lasts, and what happens if the service changes (Niinja shutting down in December 2026 is a live example). Privacy: which NSNs a user looks up can reveal what program they work on. An open-source project should not appear to endorse a vendor; affiliate links must not appear inside SolidCraft. |
| **C. OpenFLIS API** | Documented JSON API; free tier; states it serves only releasable FLIS data. | Same terms, privacy and longevity questions as B. |
| **D. User-supplied provider** | Organisations with FED LOG or their own catalogue can plug it in. | Needs a stable, documented provider interface. |

Recommendation (opinion): one **provider interface** in the engine, **A as the default
baseline**, and B/C/D as optional providers the user enables explicitly in settings, with a
one-line notice that lookups are sent to that provider. Any provider's answer is normalised into
the same **NSN record** (below) and copied into the design, so designs never depend on a
provider being online. If NSN Lookup's offer goes ahead, a short written agreement covering the
questions in row B would let maintainers decide on facts.

## Export control and distribution

**Not legal advice.** Summary of public guidance:

- **ITAR** (22 CFR 120–130) controls defense articles and their **technical data**; **EAR**
  (15 CFR 730–774) controls dual-use items and technology. Information in the **public
  domain** (ITAR § 120.34) or **published** (EAR § 734.7, which includes posting on the
  internet without restrictions) is not controlled, though sending anything to sanctioned
  destinations can still need a licence
  ([Stony Brook export-control guide](https://stonybrook.edu/research-security/export-controls/_export-control-regulations/_federal-export-control-regulations/publicly-available.html)).
- **NSN data itself** in PUB LOG is published by DLA as publicly releasable, so a pack built
  from it is published information.
- **Standard hardware geometry** from public specs (AN/MS) is published information.
- **OEM drawings and models of defense articles** are typically controlled technical data.
  DLA gates them behind JCP certification. They must **never** enter a public library, even if
  a user has access to them.
- **DEMIL codes** in FLIS give an automatic first screen
  ([DLA DEMIL codes](https://www.dla.mil/portals/104/Documents/DispositionServices/ddsr/userguide/demilcodes.html)):
  **A** = no DEMIL or trade security controls required; **B, C, G, P** = US Munitions List items;
  **Q** = Commerce Control List items; **D, E, F** = DEMIL required or special handling.

Proposed rules (opinion):

1. The public library accepts **contributed geometry only for DEMIL code A** items. Other items
   may show their public FLIS data but get a placeholder, with a note why.
2. Contributors attest that a model is their own work from **public information or their own
   measurement of a part they lawfully hold**, and that it is not from an OEM drawing, a TDP,
   cFolders, or any document marked with a distribution limitation or export-control notice.
3. Takedown: any contribution can be pulled immediately on a credible report, pending review.
4. The SolidCraft maintainers should get their own legal view before hosting contributed
   geometry; this file only lists the questions.

## Community contributions: models, dimensions and properties (Jason's addition)

Most NSNs will reach tier 3. Jason's idea: let users **contribute** what is missing, so the
library grows the way the tool library does.

### What can be contributed

| Contribution | Example | Checked against |
|---|---|---|
| **Parameters for an existing family** | "NIIN 01-509-0781 = hex bolt, d 25.4, pitch 3.175, L 63.5, washer face 37.1" | FLIS characteristics if present; family bounds |
| **Spec mapping** | "Part number NAS1352-3-8 is spec NAS1352, size 3, length code 8" | Reference numbers in FLIS |
| **A model** (STEP AP242 or a SolidCraft design) | An O-ring gland, a connector body | Bounding box, mass vs material density, FLIS dimensions |
| **Properties** | Mass, material, finish, torque value, envelope | FLIS where present; units and finite checks |
| **Connection frames** | Where a bolt seats and its axis | Family convention |

### Confidence levels (opinion)

| Level | Meaning |
|---|---|
| 0 Placeholder | Envelope only, from FLIS dimensions or user estimate. |
| 1 Approximate | Modelled from photos or general knowledge; good for layout, not fit. |
| 2 From public data | Built from public spec tables or FLIS characteristics; reviewer checked the source. |
| 3 Measured | Measured from a physical part with stated instruments; ties in with the part certification and inspection research (in progress). |
| 4 Reviewed by two | Level 2 or 3, independently confirmed by a second contributor. |

### Review and validation

- **Automatic checks** on every submission: schema, units, finite values, size caps (the
  limits style in libraries.md and PR #2074); bounding box and key dimensions within the FLIS
  ranges; mass within tolerance of volume × material density; DEMIL code A; licence present.
- **Human review** like a pull request: a reviewer confirms the source and the attestation.
- **Provenance** on every entry: contributor, date, source description, licence, tools used,
  confidence level, and the FLIS snapshot date it was checked against.
- **Licence**: contributions under **CC0 or CC-BY-4.0** so designs that embed them stay
  shareable (open question for maintainers). Supplier CAD downloads (catalogue sites) usually
  forbid redistribution and are refused.
- **Versioning**: entries are immutable revisions; a correction is a new revision. Designs keep
  the revision they inserted, with an explicit "check for update" action (libraries.md).
- **Disputes**: two conflicting contributions both stay visible with their levels until a
  reviewer resolves them.

### Where contributions live (opinion)

A public GitHub repository of JSON (and STEP) files, reviewed by pull request with CI running
the automatic checks, as proposed for the tool library. SolidCraft builds packs from it. An
in-app "Contribute this part" action could export the user's model and properties into the
right file layout and open the contribution page, never uploading on its own.

## What users need

### Workflow today (typical)

1. Find the NSN in a parts list, manual or requisition.
2. Look it up on a website; read the item name, part number, CAGE and characteristics.
3. Find the spec (ASSIST, purchased NAS) or a supplier page.
4. Model the part by hand, or download a supplier CAD file if the terms allow.
5. Type NSN, part number and description into the CAD properties so the BOM shows them.

### Proposed workflow in SolidCraft (opinion)

1. **Insert → Part by NSN** (or the command palette): type `5306-01-509-0781`, a NIIN, or paste
   a list of NSNs.
2. SolidCraft shows the **NSN record**: item name, FSC, part numbers and CAGEs, key
   characteristics, DEMIL code, the model tier it can offer, and the data source and date.
3. Click **Insert**. The part is placed like any `parts.insert` part (at a point or into a hole,
   as #2072 proposes for hole stacks), one undo step.
4. The component gets properties: `part_number`, `description`, plus `nsn`, `niin`, `fsc`,
   `cage`, `spec`, `model_tier`, `source`. The BOM (`bom.export`) gets an NSN column; group
   by NSN.
5. For a placeholder, the panel offers **Contribute a model** (see above).
6. **Batch**: paste an NSN parts list (CSV) to insert every resolvable part and report the rest.

### Commands (opinion; ids follow SolidCraft's `<area>.<thing>` rule)

| Command | Params → result |
|---|---|
| `parts.nsn.lookup` | `nsn` (13 digits, dashes optional) or `niin` → NSN record; never inserts; errors name the provider. |
| `parts.insert` with `nsn` | Same placement params as today; resolves the record, picks the tier, inserts, records properties. |
| `parts.nsn.providers` | Lists providers, which are enabled, data date. |
| `parts.nsn.check` | `nsn` → which tier would be used and why (missing characteristics, DEMIL, no family). |

All take JSON and never open dialogs (repo rule). Input is hostile: validate digits, cap batch
sizes, finite-check every FLIS value before it becomes geometry.

### Data model sketch: the NSN record (opinion)

```json
{
  "niin": "015090781",
  "fsc": "5306",
  "item_name": "BOLT, MACHINE",
  "references": [{"part_number": "…", "cage": "…", "spec": null}],
  "characteristics": [{"mrc": "AHYM", "value_mm": 25.4},
                      {"mrc": "AASB", "min_mm": 62.71, "max_mm": 64.29}],
  "demil": "A",
  "model": {"tier": "characteristics", "family": "hex_bolt_inch", "params": {"d": 25.4, "pitch": 3.175, "length": 63.5}},
  "provenance": {"source": "PUB LOG", "snapshot": "2026-10-01", "provider": "pub_log_offline"}
}
```

It fits the library pack envelope in libraries.md (pack, entry, immutable revision,
per-entry provenance). On insert the record is copied into the design, so the design opens
offline. It also fits PR #2074's per-length article number, which could carry the NSN.

### Fit with SolidCraft's current work

- **Builds on** `parts.library` / `parts.insert` / `parts.resize`, component properties and BOM
  (all on `main`), the library contract (#787, `libraries.md`), and fastener families from a
  file (#2071, draft PR #2074, held while kernel work is paused).
- **Layering:** the PUB LOG reader and pack builder belong in `io`; the NSN record and design
  copy in `doc`; lookup/insert commands in `engine`; online providers must not pull network
  code into layers that build for wasm without a fallback.
- **A design-format change** (new component properties) is one file in
  `crates/doc/src/format/pending/`.
- **Order (opinion):** (1) NSN properties on components and an NSN BOM column; (2) offline PUB LOG
  hardware pack and `parts.nsn.lookup`; (3) inch families for AN/MS bolts, screws, nuts,
  washers and tier 1–2 insert; (4) placeholders; (5) contribution repository; (6) optional
  online providers.

### Shared-library pattern

The tool library, the workholding library and this part library all need the same things:
stable identities, immutable revisions, provenance, licence, confidence levels, community
review, offline copies inside designs, and optional online providers. One shared pack and
contribution process (libraries.md phase 1) should serve all three rather than three systems.

## Complaints and requests

From public sources and general practice (opinion where noted):

- NSN data and CAD data live in different worlds; the designer is the bridge (opinion).
- Supplier CAD downloads for MS/NAS parts usually come from catalogue sites whose terms forbid
  redistribution, so models cannot be shared with a design (opinion; check per site).
- NAS/NASM specs cost money per sheet, which hurts small shops and hobbyists (fact: they are sold).
- BOMs for government work often must carry NSNs; CAD rarely has a field for them (opinion).

## Glossary

- **AN / MS / NAS / NASM**: standard hardware specification series (see table).
- **CAGE / NCAGE**: Commercial and Government Entity code, the 5-character supplier code.
- **DEMIL code**: FLIS code saying whether and how an item must be demilitarised; also flags
  USML/CCL items.
- **DLA**: Defense Logistics Agency, which runs FLIS.
- **EAR / ITAR**: US export regulations for dual-use and defense items.
- **FIIG**: Federal Item Identification Guide, which defines the characteristics for an item name.
- **FLIS**: Federal Logistics Information System, the US NSN catalogue.
- **FSC / FSG**: Federal Supply Class (4 digits) / Group (first 2 digits).
- **MRC**: Master Requirement Code, a 4-letter code for one characteristic (e.g. AHYM).
- **NCB**: National Codification Bureau; also the 2-digit country code in the NIIN.
- **NCS**: NATO Codification System.
- **NIIN**: National Item Identification Number, the 9-digit unique part of the NSN.
- **NMCRL**: NATO Master Catalogue of References for Logistics.
- **NSN**: NATO Stock Number (National Stock Number in the US), 13 digits.
- **PUB LOG / FED LOG / WebFLIS**: DLA's public, government and online FLIS tools.
- **TDP**: Technical Data Package (MIL-STD-31000), the drawings/models behind a part.

## Open questions

1. Will maintainers host contributed geometry at all, or only parameters and spec mappings?
2. Which licence for contributions: CC0, CC-BY-4.0, or MIT OR Apache-2.0 like the code?
3. How large is a PUB LOG pack for FSC 53xx, and should it ship with SolidCraft or download on
   request?
4. Does NSN Lookup publish an API, and on what written terms (caching, redistribution into
   designs, attribution, rate limits, duration, privacy)?
5. Should DEMIL code A be the only gate, or also the Controlled Inventory Item Code (CIIC)?
6. Which tier wins when a spec model and FLIS characteristics disagree?
7. For NASM parts, which MS/AN sheets are dimensionally identical and can be used as public
   sources? Needs a per-sheet reviewer.
8. Non-US NSNs: is NMCRL or any national bureau's data usable on open terms?

## Sources

### Government and NATO

- DLA, [Public Logistics (PUB LOG) data brochure](https://www.dla.mil/Customer-Support/Brochures/Details/Article/2893128/public-logistics-pub-log-data/)
- DLA, [FLIS Data Electronic Reading Room](https://www.dla.mil/Information-Operations/FLIS-Data-Electronic-Reading-Room/)
- DLA, [Self-help tools side-by-side comparison](https://www.dla.mil/Customer-Support/Brochures/Details/Article/2893363/self-help-tools-side-by-side-comparison/)
- DLA, [The NSN booklet](https://www.dla.mil/Portals/104/Documents/J3LogisticOperations/Brochures/J3_nsnbooklet_200323.pdf)
- DLA, [WebFLIS training page](https://www.dla.mil/Customer-Support/Training/Details/Article/2925963/webflis-federal-logistics-information-system/)
- DLA, [DEMIL codes](https://www.dla.mil/portals/104/Documents/DispositionServices/ddsr/userguide/demilcodes.html)
- DLA, [ASSIST Quick Search overview](https://quicksearch.dla.mil/qsOverview.aspx)
- NATO, [NCS fact sheet](https://www.nato.int/structur/ac/135/data/pdf/NCS-Fact-Sheet.pdf) and
  [NMCRL NSN page](https://www.nato.int/structur/AC/135/nmcrl/data/nsn)
- NIST, [MIL-STD-31000 technical data packages (MBE Summit 2024)](https://www.nist.gov/document/2024-mbe-summit-windham-mil-std-31000-technical-data-packages-acquisition)

### Reference and guidance

- Wikipedia, [NATO Stock Number](https://en.wikipedia.org/wiki/NATO_Stock_Number)
- Stony Brook University, [Publicly available, public domain and published](https://stonybrook.edu/research-security/export-controls/_export-control-regulations/_federal-export-control-regulations/publicly-available.html)
- NFFS, [JCP/eJCP fundamentals](https://www.nffs.org/news/jcpejcp-fundamentals-what-every-foundry-should-know)
- EverySpec, [AN3 thru AN20 cancellation notice](https://everyspec.com/AN-SPECS/AN100-AN999/AN3_THRU_AN20_REV-12_NOTICE-1_27719/)
- NSAI shop, [example NASM listing (sold standard)](https://shop.standards.ie/en-ie/standards/nasm21266-2016-92052_saig_aia_aia_192905)

### NSN data services (public pages only)

- [nsnlookup.com](https://www.nsnlookup.com/)
- [OpenFLIS](https://openflis.com/)
- [Niinja developers](https://niinja.co/Welcome/Developers)
- [ISO Group API reference](https://iso-dlms.com/Help)
- [HigherGov NSN 5306-01-509-0781](https://www.highergov.com/nsn/5306015090781/) (FLIS characteristics example)

### SolidCraft

- `crates/engine/src/cmd/parts.rs`, `component_properties.rs`, `bom.rs`
- [`docs/book/src/dev/libraries.md`](../docs/book/src/dev/libraries.md),
  [`local-libraries.md`](../docs/book/src/dev/local-libraries.md)
- Issues [#787](https://github.com/tangency-labs/solidcraft/issues/787),
  [#2071](https://github.com/tangency-labs/solidcraft/issues/2071),
  [#2072](https://github.com/tangency-labs/solidcraft/issues/2072); draft PR
  [#2074](https://github.com/tangency-labs/solidcraft/pull/2074)
