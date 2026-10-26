# CAM research: a marketplace for stock material, sized from the model and ordered with its certificates

Research for [#1621](https://github.com/tangency-labs/solidcraft/issues/1621) (Research Request
for Plugin: CAM). This file covers the step **before the first cut: getting the raw material**.
The idea (Jason's): SolidCraft already knows the finished part, the material and, once CAM
exists, how much stock each setup needs. It could **work out the blank, ask several metal
suppliers for a price and ship date, let the user order the best offer, and carry the mill
certificate that arrives with the metal into the part's traveller**, so the part's record starts
at the heat of metal it was cut from.

It is one of a set of files from the same author that describe CAM as one physical system:

| Part of the system | File | What it gives this file |
|---|---|---|
| Overview | [CAM physics system](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-vgqo3g/research/cam-research-nsnlookupjn-physics-system.md) | the shared Setup record that names the stock |
| Physical properties | [Physical properties of the design and the stock](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-uxzal2/research/design-research-nsnlookupjn-physical-properties.md) | the stock body, its surface condition and the "material certificate data on the stock instance" this file fills in |
| Stock integrity | [Stock structural integrity](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-stock-integrity/research/cam-research-nsnlookupjn-stock-integrity.md) | residual stress, which depends on the temper ordered (for example T651, stress relieved) |
| Workholding | [Workholding physics](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-workholding/research/cam-research-nsnlookupjn-workholding.md) | how much extra stock the vise or fixture has to grip |
| Cutting physics | [Cutting tool physics](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-cutting-physics/research/cam-research-nsnlookupjn-cutting-physics.md) | the machinability data keyed by alloy and temper |
| Machine library | [Machine library](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-machine-library/research/cam-research-nsnlookupjn-machine-library.md) | travel, table load and chuck capacity, which cap the blank size |
| Inspection | [Part certification by in-process inspection](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-y2w06n/research/cam-research-nsnlookupjn-in-process-inspection.md) | the per-part record and traveller the certificate joins |
| **Stock marketplace (this file)** | | the blank from the model, supplier quotes and orders, the certificate on the stock instance |

Licence: MIT OR Apache-2.0, the same as SolidCraft (see the Licence section of CONTRIBUTING.md).

## About this research

- **Author:** Jason ([@nsnlookupjn](https://github.com/nsnlookupjn)). Idea and direction by Jason
  (the five suppliers to start with are his choice); source research and writing done with an AI
  assistant (Claude) in October 2026.
- **How it was produced:** reading the suppliers' public web pages, published API overview pages
  and standards catalogue pages, accessed 2026-10-26. No supplier account was opened, no price
  was requested, no order was placed and no site was scraped. One file (the Nox Metals OpenAPI
  document) was **not** read because the site's robots rules disallow automated download; a
  person can download it from the API page by hand. No commercial CAD/CAM program was driven, no
  GPL/LGPL/AGPL source was read (one digital-certificate schema project is AGPL and is named but
  not opened; see "Digital certificates"), and no text is copied beyond short quotes.
- **Units:** millimetres and kilograms. US suppliers sell in inches, so inch sizes are given
  next to the millimetre values where they matter (1 in = 25.4 mm exactly).
- **Prices:** only figures shown publicly on a product page are quoted, with the date. Metal
  prices change daily; treat every price here as an example of the pricing **shape**, not a
  current price.
- **Marking:** "(unverified)" marks a claim not confirmed against a source. **Fact**,
  **opinion** and **marketing** are labelled where they mix.
- **No affiliation** with any supplier named here. They are named because Jason asked to start
  with them; the design below is meant to work with any supplier.

> **[TODO Jason: personal experience]** Where you buy stock today, how you size blanks (rules of
> thumb per face, per vise, per lathe chuck), how long quotes take, what goes wrong with
> certificates (missing, wrong heat, wrong spec), and whether your customers ask for DFARS or
> domestic-melt material. Label this as personal experience.

## Summary

1. **Only one of the five suppliers publishes an ordering API.** Nox Metals (Ferndale, Michigan)
   has a public developer page with an API key from its customer portal, JSON over HTTPS, a test
   (UAT) server, endpoints to submit an RFQ, read back price, availability and lead time, place
   an order "on terms or by card", list alloys and tempers, and receive webhooks. Its range is
   narrow (aluminium plate and bar, four stainless bar grades). **Alro** and **OnlineMetals** offer
   electronic business integration to account customers (Alro lists mill certifications among its
   electronic documents; OnlineMetals says it integrates with customers' "EDI and API systems"),
   but neither publishes an API to the public. **Speedy Metals** and **Metal Supermarkets** sell
   through web stores and quote forms only.
2. **So a SolidCraft "marketplace" is mostly a common stock request, not a set of live APIs.**
   The useful core is a supplier-neutral **stock request** (shape, alloy, temper, specification,
   size, cut tolerance, quantity, certificate type, origin requirement) that SolidCraft computes
   from the model and CAM. Each supplier is reached through the best route it offers: a real API
   where one exists, the shop's own EDI or punchout account where the shop has one, a prefilled
   web cart or quote form otherwise, and a drafted RFQ e-mail as the fallback that always works.
3. **The blank comes from the model in four additions.** Finished size, plus a clean-up
   allowance per face, plus what the workholding grips, plus the supplier's cut tolerance (which
   is one-sided: saw cuts are typically **-0 / +1/8 in (+3.2 mm)**), then rounded **up** to the
   next size the supplier sells. A worked example below turns a 100 × 60 × 22 mm 6061 part into
   an order for **1 × 2-1/2 × 4-1/16 in** plate, about 0.45 kg.
4. **The certificate is the start of the traveller.** A mill test report (MTR) or EN 10204
   inspection certificate names the heat, the specification, the chemistry and the measured
   properties. SolidCraft should attach it to the **stock instance**, read the heat number,
   alloy, temper and specification from it, check them against what was ordered, and let every
   part cut from that stock inherit the link. EN 10168 already defines the fields of such a
   document; a structured (machine-readable) certificate format would let this happen without
   retyping.
   For defence work the request also carries the **melt-origin requirement** (DFARS specialty
   metals; the often-quoted 252.225-7014 was replaced by 252.225-7008/-7009 in 2009), and the
   receipt check decides from the heat's own chemistry whether the metal is a specialty metal and
   holds it if the certificate's melt country is not allowed.
5. **The hard rules shape the design.** SolidCraft must work offline and in the browser, never
   crash on hostile input and never open dialogs from programmatic calls. So supplier access is
   an **optional, sandboxed plugin** (the planned Wasm plugin system, #1221, with a network
   permission), quotes are **snapshots** that expire, credentials never enter the design file,
   and **placing an order is always a person's explicit action**, never something an agent or a
   script does on its own.
6. **Neutral by design (opinion).** An open project should not rank suppliers by who pays it.
   Offers are sorted by the user's own rule (delivered price, ship date, certificate type,
   origin), every connector is open source, and any commercial arrangement would be disclosed in
   the connector's manifest. Maintainers decide whether such connectors belong in the main
   repository at all.

## Fundamentals

### What a machine shop buys

- **Shapes:** plate and sheet (thickness × width × length), round bar (diameter × length),
  square, flat and hex bar, tube and pipe (outside diameter × wall or schedule), angle, channel
  and beam. CNC milling mostly uses **plate and flat bar**; turning uses **round and hex bar**.
- **Alloy and temper (condition):** for example 6061-T651 aluminium (T651 = solution treated,
  stress relieved by stretching, artificially aged), 7075-T7351, 4140 annealed or pre-hardened
  ("heat treated" on Speedy Metals' pages), 17-4 PH Condition A, 304/304L annealed. Temper
  changes both machinability (cutting physics file) and how much the part moves after machining
  (stock integrity file): a stretched "T651" plate holds less residual stress than plain "T6"
  plate of the same alloy (fact, general metallurgy).
- **Specification:** the standard the material was made to, for example ASTM B209 (aluminium
  plate), ASTM B211 / B221 (aluminium bar), AMS specifications in aerospace (unverified which
  apply to which product here). A drawing may call a specification, not just an alloy, and then
  the certificate must show it.
- **Form and finish:** cold-finished, hot-rolled, turned-ground-polished bar, rolled plate, cast
  tooling plate (flat, low stress, tighter thickness), pre-machined "squared" blocks. Alro sells
  milled "SuperSquare" blocks held to 0.002 in per foot squareness and parallelism, which saves the
  shop its first squaring operations (marketing claim on benefits; tolerance is the vendor's spec).
- **Cut tolerance:** how close the delivered length and width are to what was ordered. Nox
  Metals' standard is **-0.00 / +0.125 in**; Speedy Metals' by-the-inch cuts are **+1/8 in, -0**,
  while its fixed lengths are looser (±1/4 in at 12 in up to ±1 in at 72 in). Alro lists a
  precision saw-cut option of **±0.005 in** for aluminium plate under 2 in thick and under 30 in
  long (the rest of that table was hard to read; unverified). MISUMI (outside the five, included
  as a model) sells precision-cut aluminium plate configured to 0.1 mm with ±0.05 to ±0.25 mm on
  the cut sides depending on grade and size.

### What makes a stock "buy" decision

| Input | Where SolidCraft would get it |
|---|---|
| Finished envelope of the part per setup | the model and CAM setups (bounding box in the setup frame, not the world frame) |
| Clean-up per face | CAM: the first facing or turning pass depth; default from material form (hot-rolled needs more than cold-finished; scale and decarburised skin, physical properties file) |
| Workholding grip | workholding file: vise jaw height, dovetail, soft-jaw depth, chuck grip length, parting width |
| Machine limits | machine library: travel, table load, chuck through-hole and maximum diameter |
| Allowed alloys, tempers, specs | the design's material card plus the drawing's notes |
| Quantity and nesting | the job: parts per blank (several parts from one plate or one bar length) |
| Certificate requirement | the customer's order: none, certificate of conformance, EN 10204 3.1 / MTR, domestic or DFARS-compliant melt |
| Supplier's sizes, steps, cut tolerance, minimums | the supplier connector |

### Certificates: what they are

- **EN 10204** defines types of inspection document: **2.1** declaration of compliance (no test
  results), **2.2** test report on non-specific inspection, **3.1** certificate with results of
  specific inspection on the delivered product, validated by the manufacturer's independent
  inspection department, and **3.2**, which is also validated by the purchaser's or a third party's
  inspector (general knowledge, consistent with the sources; the standard itself is paywalled).
- **EN 10168** lists **what information** goes in those documents, in groups: **A** commercial
  transaction and parties (manufacturer, document type and number, purchaser, order), **B**
  description of the product (designation, delivery condition, marking, identification, quantity,
  dimensions, mass), **C** inspection (tensile, hardness, impact, other mechanical tests,
  chemical composition and steelmaking process), **D** other tests, and **Z** validation. Each
  item has a code number (A01, B02 and so on), with free numbers for the issuer's own use. Groups
  C and D are absent on a 2.1 document. This coded structure is exactly what a parser or a
  digital certificate needs.
- **US practice:** a **mill test report (MTR)** or "mill cert" from the producing mill, passed
  through the distributor, often with the distributor's own **certificate of conformance** and a
  lot or bundle tag that ties the cut piece to the heat (unverified for each supplier below).
  Nox Metals says "every order ships with mill certs and full traceability documentation";
  OnlineMetals advertises "free material certifications"; Alro lists "Mill Certifications" among
  the documents it exchanges electronically.
- **Origin rules:** DFARS 252.225-7009 requires **specialty metals** in US defence deliverables to
  be melted or produced in the US or a qualifying country. The clause defines specialty metals as
  certain alloy steels, nickel and cobalt alloys, titanium and zirconium; **aluminium is not in
  that list**. Some suppliers still sell "DFARS compliant" or domestic-melt aluminium (Nox Metals
  does), presumably because customers flow down domestic-source requirements beyond the clause
  (opinion; ask the supplier what the claim covers). The stock request should carry the
  requirement in words the supplier uses ("domestic melt", "DFARS 252.225-7009"), and the check
  on receipt should read the melt country from the certificate. The full treatment is in the
  next section.

### Melt origin and DFARS specialty metals

Jason's addition: raw CNC stock for defence work falls under the DFARS **specialty metals**
rules, so the stock request and the certificate check must capture **where the metal was
melted**, not only what it is.

**Which clause applies (fact, from the Federal Register and the current DFARS text):**

| Clause | Status | What it requires |
|---|---|---|
| 252.225-7014, Preference for Domestic Specialty Metals | **Removed and reserved** by the final rule effective **29 July 2009** | (the older rule; still quoted on old purchase orders and supplier quality clauses, unverified how often) |
| 252.225-7008, Restriction on Acquisition of Specialty Metals | current | used when the contract delivers **specialty metals themselves as end items** (for example mill products): "Any specialty metal delivered under this contract shall be melted or produced in the United States or its outlying areas" |
| 252.225-7009, Restriction on Acquisition of Certain Articles Containing Specialty Metals | current | used for aircraft, missile and space systems, ships, tank and automotive items, weapon systems and ammunition, **and their components**: specialty metal melted or produced in the US, its outlying areas **or a qualifying country**, with listed exceptions (commercial items, a 2 % minimal-content exception, and others) |
| 252.225-7010, Commercial Derivative Military Article | current | the contractor's certification for that exception |

A machine shop making a component under a 7009 flowdown is the common case. The practical
consequence is the same for the stock: the shop needs evidence of the melt country for every
heat of specialty metal it machines. Because purchase orders often quote clause numbers as
written years ago, the stock request should **keep the clause exactly as the customer cited it**
and show, next to it, which current clause SolidCraft reads it as ("252.225-7014 was replaced by
252.225-7008/-7009 in 2009; treated as -7009"), for the person to confirm. Legal responsibility
for compliance stays with the contractor; SolidCraft records and checks, it does not certify
(opinion, and important to state in the interface).

**Does the material count as a specialty metal?** The clause decides by chemistry, so the check
can be automatic from the certificate's chemistry (EN 10168 group C):

- Steel with **manganese over 1.65 %**, **silicon or copper over 0.60 %**, or **more than 0.25 %**
  of any of aluminium, chromium, cobalt, molybdenum, nickel, niobium, titanium, tungsten or
  vanadium. So 4140 (about 0.8 to 1.1 % Cr) and every stainless steel count; plain 1018 or 12L14
  normally does not (typical chemistry; the actual heat decides).
- Nickel and iron-nickel alloys, and cobalt alloys, with more than 10 % of other metals.
- Titanium and its alloys; zirconium and its alloys.
- **Not aluminium alloys.** A "DFARS" request on 6061 plate is a customer flowdown or a
  domestic-melt preference, not this clause; the request should say which (see above).

From the material card SolidCraft can predict the answer when the request is made (4140: yes),
and from the certificate it can confirm it for the actual heat.

**What the stock request carries:**

- `origin.clause_cited`: the text from the customer's order, unchanged (for example "DFARS
  252.225-7014").
- `origin.clause_read_as`: the current clause SolidCraft maps it to, confirmed by the person.
- `origin.allowed`: "US and outlying areas" (7008), or "US, outlying areas and qualifying
  countries" (7009). The qualifying-country list is in DFARS 225.003 and changes; it is kept as
  dated data that can be updated, never written into code (opinion).
- `origin.specialty_metal`: predicted from the material card (yes / no / unknown).
- Whether the shop accepts a supplier's **certificate of conformance** for origin, or requires
  the **mill's** test report showing the melt location (customer quality clauses differ).

**What a supplier offer must state:** the melt-origin claim for that product ("domestic melt",
"DFARS compliant", "melted in country X"). An offer for a specialty metal with no claim is
shown, but flagged, and sorted below compliant ones when the request has an origin requirement.

**The check on receipt:**

1. Read the **melt country** and the **melting mill** from the certificate (US mill reports
   commonly say "melted and manufactured in USA" or name the melt country; the field and its
   wording vary by mill, so extraction must be reviewed by a person).
2. Recompute **specialty metal yes/no** from the heat's own chemistry.
3. If it is a specialty metal and the melt country is not in `origin.allowed`, or is missing,
   the stock is **held**: it cannot be issued to a job with that requirement. A person can
   release it only with a recorded reason (for example "covered by the 2 % minimal-content
   exception", which is a contract-level judgement, not a per-piece one).
4. If the supplier's certificate is a distributor's certificate of conformance and the request
   requires the mill's report, the check says so.

**Traceability back from a heat.** Because every part records the stock instance and its heat
(see "The certificate on the stock instance and in the traveller"), if a mill's origin claim is
later found false, the shop can list every part, serial and shipment cut from that heat. This
is the main value of putting origin in the record rather than in a filing cabinet (opinion).

**Related standards Jason listed (for the defence workflow, not all about stock):**

| Discipline | Military / federal | Commercial equivalent | How it touches this file |
|---|---|---|---|
| Drawings and CAD data | MIL-STD-31000 (technical data packages) | ASME Y14.100, Y14.5 | the drawing or model calls the material specification and any origin note the request reads |
| Castings | MIL-STD-2175 | AMS-STD-2175 | casting grade and radiographic inspection; belongs to the raw-part model in the physical properties file |
| Forgings | MIL-F-7190 | SAE AMS series | forging stock integrity; same |
| Raw CNC stock | DFARS specialty metals (now 252.225-7008/-7009) | | this section |
| First article inspection | DI-NDTI-80809B | AS9102 | AS9102's raw material and specification accountability (Form 2) can be filled from the stock instance and its certificate (unverified field-by-field; inspection file) |

### Digital certificates

Today most certificates arrive as PDFs, often scans. Two directions exist:

- **Structured certificates built on EN 10168 codes.** The coded groups map naturally to JSON or
  XML. A "Material Identity" project publishes JSON schemas for digital certificates following
  EN 10204, but its (archived) schema repository is licensed **AGPL-3.0**, so under SolidCraft's
  rules nobody working on SolidCraft should read that code, and the licence of its successor
  repository should be checked before anyone opens it; an independent schema written from EN 10168's
  field list (which the standard defines, not that project) avoids the problem.
- **Reading the PDF.** A parser (or a person with a form) extracts heat number, specification,
  alloy, temper, size, chemistry and mechanical results, and keeps the original PDF unchanged as
  the evidence. The extraction is a convenience, never the record of truth (opinion).

## The five suppliers

All facts below are from the suppliers' public pages, accessed 2026-10-26. "Not stated" means the
page reviewed did not say; it does not mean the supplier does not offer it.

| | Nox Metals | Alro | OnlineMetals | Speedy Metals | Metal Supermarkets |
|---|---|---|---|---|---|
| Who | Ferndale (Detroit), MI; ISO 9001:2015; raised an $11.5 M seed round in June 2026 | Large North American distributor (metals, industrial supplies, plastics); "over 25,000 customers" | Web-first distributor; owned by thyssenkrupp Materials NA since 2007 (from the press-release listing; page body not retrieved); "eight locations" | Two Wisconsin stores (New Berlin, Appleton) with a web store | Franchise chain, about 130 to 140 stores in the US, Canada and the UK, HQ Toronto; majority owned by Riverarch Equity Partners since Dec 2024 |
| Range | Aluminium plate (6061, 7050, 7075, 5000-series cast, cast tooling plate), aluminium bar, 304/316/17-4/416 stainless round bar | Very wide: steel, stainless, aluminium, tool steel, plastics, industrial supplies | Wide: plate, sheet, bar, tube, many alloys | Wide bar and plate range, many steel grades | "Over 8,000 metal types, grades and shapes" |
| Cut to size | Plate saw-cut to length and width; optional facing | Saw, waterjet, laser; precision plate saw; milled SuperSquare blocks | Free cut to size; shears and plate saws; same-day shipping | By the inch (1/16 in steps) or fixed lengths | Cut while you wait; "no minimum order" |
| Price on line | Yes, instant price and ship date per ZIP code | Yes, for logged-in MyAlro users | Yes, in the web store | Yes, per length, on each product page | US: quote request and stores; UK site sells on line |
| Cut tolerance stated | -0 / +0.125 in | ±0.005 in precision option (aluminium plate, size limits) | Not stated | +1/8 / -0 by the inch | Not stated |
| Certificates | Mill certs with every order; DFARS / domestic melt available | Mill certifications sent electronically | Free material certifications | Not stated on the pages read | Not stated on the pages read |
| Minimum | $180 per order | Not stated | Not stated | Not stated | No minimum |
| **Machine interface** | **Public REST API** (below) | e-business: electronic documents, MyAlro web store, e-Kanban and vending; no public API | "EDI and API" integration for pro accounts, by arrangement; no public docs | None found | None found |

### Nox Metals API (the only public one)

From the [API overview page](https://noxmetals.co/api-docs):

- Base URLs `https://developer-api.noxmetals.co` (production) and
  `https://developer-api-uat.noxmetals.co` (test). One API key, "JSON over HTTPS", created in the
  customer portal. `GET /api/health` needs no key.
- **RFQs:** `POST /api/v1/rfqs` submits a request (priced asynchronously), `POST /api/v1/rfqs/item`
  submits one line, `GET /api/v1/rfqs/{id}` returns it "with pricing, availability and lead time",
  `GET /api/v1/rfqs` lists them.
- **Orders:** `POST /api/v1/orders` orders "one fulfillment option of a priced quote, on terms or
  by card"; list and fetch endpoints. An older card-purchase endpoint is marked legacy.
- **Materials:** `GET /api/v1/materials` lists alloy codes and tempers by product type.
- **Webhooks:** register, list and revoke endpoints; the page has a "Webhook signatures"
  section, so deliveries are signed (details not on the overview).
- **Not on the overview page:** field names, units, rate limits, error format and terms of use.
  These are in the downloadable OpenAPI file, which was not read here (see "About this
  research"). A connector author would read it with their own account.

What this shape means for SolidCraft: the RFQ, priced options, order flow is the right general
model (quote first, choose one fulfilment option, order). "Priced asynchronously" plus webhooks
means a desktop app needs either polling with a backoff or a relay, because a desktop client
cannot receive webhooks directly (opinion).

### Terms of use

None of the five publishes terms that grant third-party software the right to read their
catalogue or prices automatically. Integration therefore means: use the published API under the
account holder's own key (Nox), use the integration the supplier sets up for the shop's account
(Alro, OnlineMetals), or open the supplier's own page for the person to finish (Speedy Metals,
Metal Supermarkets). **Scraping prices is out** (opinion, and against SolidCraft's ground rule of
not taking data from products without permission).

## Worked examples

### 1. A milled block from plate

Part: 100 × 60 × 22 mm in 6061-T651, two setups in a vise (Op 10 top, flip, Op 20 bottom).

| Step | Thickness (Z) | Width | Length |
|---|---|---|---|
| Finished size | 22.0 | 60.0 | 100.0 |
| Clean-up, 1.0 mm top and bottom; 1.5 mm each side in X and Y (the saw-cut edges) | 24.0 | 63.0 | 103.0 |
| Workholding: Op 20 grips the finished Op 10 surfaces, so no extra grip stock here (a dovetail or a "grip tab" would add it) | 24.0 | 63.0 | 103.0 |
| In inches | 0.945 | 2.480 | 4.055 |
| Next size the supplier sells: plate thickness from the list, width and length rounded **up** to the cut step (1/16 in) | **1 in (25.4)** | **2-1/2 in (63.5)** | **4-1/16 in (103.2)** |
| Supplier cut tolerance -0 / +1/8 in: delivered size can be up to 3.2 mm larger, never smaller | | 63.5 to 66.7 | 103.2 to 106.4 |

Blank volume 25.4 × 63.5 × 103.2 mm ≈ 166 cm³, about **0.45 kg** at 2.70 g/cm³. The finished
part's envelope is 132 cm³, so the buy-to-envelope ratio is 1.26 (the true buy-to-fly ratio is
higher, because the part has pockets). The +1/8 in tolerance matters to CAM: the first facing
and contour passes must assume the **largest** delivered size, or the first cut enters the
material at full depth. So the stock body in CAM should be the **maximum material** blank, and
the ordering size is the **minimum**.

### 2. A turned part from bar

Part: Ø40 × 50 mm in 4140 pre-hardened, turned complete and parted off.

- Diameter: 40 + 2 × 0.75 mm clean-up = 41.5 mm. Standard sizes either side are 1-5/8 in
  (41.28 mm, too small) and **1-3/4 in (44.45 mm)**, so the order is 1-3/4 in bar. The 3 mm of
  extra diameter costs material and a roughing pass; a "cold-finished" or "turned, ground and
  polished" bar might allow less clean-up (physical properties file: surface condition).
- Length per part: 50 + 1 mm face + 3 mm parting tool width = 54 mm. A single part also needs
  about 15 mm of chuck grip, so 69 mm per single blank, about 0.84 kg at 7.85 g/cm³. A bar
  feeder run of ten parts needs 10 × 54 + one grip + a remnant, so ordering **one length** is
  cheaper than ten blanks: this is the "nesting" decision for bar.
- Speedy Metals' public page for 3-1/4 in round 4140 cold-finished, heat treated showed a
  price that falls per inch with length (about $14.70 per inch at 12 in down to about $10.04 per
  inch at 72 in, computed from the listed prices on 2026-10-26). That curve is why the request
  should let the supplier offer a longer piece when it is cheaper.

### 3. Many parts from one plate

Ten of the block in example 1 could come as ten saw-cut pieces (ten cuts, ten tolerances, ten
pieces to square) or as one plate machined into ten parts on a fixture plate. The second uses
less material per part but needs tabs and a bigger machine. Nox Metals describes nesting
customers' plate orders onto shared plates on its side (its "Factos" engine, guillotine
bin-packing over about 100 candidate layouts per plate, marketing description). SolidCraft
would not nest across customers, but it can offer both requests and show the delivered price
and the extra machining time side by side (opinion).

## Accuracy

- **Sizes are nominal.** Plate thickness has its own tolerance by specification and thickness
  band (for example MISUMI lists ±0.04 to ±0.15 mm for 4 to 50 mm precision plate; rolled
  plate to ASTM B209 is looser; unverified figures for each alloy). The blank calculation should
  use the supplier's or specification's **minimum** thickness when deciding whether a size
  cleans up, and the **maximum** when planning the first cut.
- **Prices and ship dates are quotes.** They change with the metal market and stock; a
  snapshot needs an expiry time and the date it was taken.
- **Certificates can be wrong** (wrong heat, wrong document, a typo in the specification). The
  receipt check compares, it does not trust. Where the part is critical, a hardness check on
  receipt (inspection file) confirms the condition.
- **Unit conversion** is a real source of error: a model in millimetres ordered in inches must
  round **up** after converting, never to nearest.

## Programs

What exists today (public descriptions only; details unverified):

- **Supplier web stores and configurators** (all five, MISUMI): price and order one line at a
  time; the person types the size.
- **Nox Metals API**: machine ordering for its range (above).
- **Shop ERP/MRP systems** commonly hold a material requirement per job and send purchase orders
  to suppliers, sometimes by EDI; some job-shop ERPs list integrations with metal-buying services
  (unverified which). They start from a typed material line, not from the model.
- **Quoting tools for machined parts** estimate stock from the CAD bounding box to price a part;
  they do not order the stock (unverified).
- **CAM programs** let the user define stock as a box or cylinder with an offset around the part,
  or pick from a stock library; none of the public descriptions reviewed links that stock to a
  supplier order and a certificate (unverified; a gap worth checking).

In SolidCraft today: physical material cards (`material.create/edit` with a validated
`physical_card`, `docs/physical-material-cards.md`), configurations, MCP and the control
protocol, and a **design** for sandboxed WebAssembly plugins with manifest permissions
(`docs/plugins.md`, #1221). CAM is not merged; related work is #1153 (2.5D RFC) and PR #4158
(2.5D CAM core).

## Complaints and requests

Common shop complaints about buying stock (opinion, from general shop practice; to be replaced
or confirmed by Jason's experience):

- Phoning or e-mailing several suppliers for one quote; answers arrive in hours or days.
- Typing the same size into several web stores, each with its own units and steps.
- Blanks sized by hand, too small (the part does not clean up) or too large (wasted metal and
  roughing time).
- Certificates missing, arriving separately from the metal, or not matching the tag on the bar.
- Certificates filed in a drawer, not linked to the parts, so an audit means a search.
- Drops and remnants in the shop not known to the person ordering new stock.

## What users need

### 1. A stock request computed from the model (no network needed)

- **One stock request per setup chain**, derived from: the part envelope in the first setup's
  frame, clean-up per face (from CAM, with defaults per material form), workholding grip, the
  machine's limits, the quantity, and the allowed materials from the material card.
- **Shown and editable**: every addition is listed with where it came from ("+1.0 mm top: Op 10
  face pass"), in the style of the physics-system file's "explain every number".
- **Two bodies in CAM**: the **minimum** blank (what is ordered) and the **maximum material**
  blank (ordered size plus the supplier's plus tolerance), and the first operations are planned
  against the maximum.
- **Remnants first**: before any order, match the request against the shop's own drops and
  inventory (a local list the user keeps), so the cheapest supplier is often the rack.

### 2. Supplier connectors, one per supplier, all optional

- A connector turns the stock request into that supplier's terms: nearest stocked size, cut
  steps, cut tolerance, certificate options, minimums, units.
- **Four routes, in order of preference**: (a) the supplier's published API with the user's own
  key; (b) the shop's EDI or punchout account, through the shop's ERP, where SolidCraft only
  exports the request; (c) open the supplier's web page with the size filled in, for the person
  to finish; (d) a drafted RFQ e-mail with the request attached, which works with any supplier,
  including a local one.
- **Offers** come back as a list of snapshots: supplier, product, size delivered, cut tolerance,
  certificate type, origin, price, delivered price, ship date, expiry. Sorted by the user's rule.
- **Ordering is a person's action.** The order command returns a confirmation step; an agent or
  script can prepare the order but cannot complete it without the person. Credentials live in
  the operating system's credential store (desktop) or the browser's storage (web), never in the
  design or in a shared file.

### 3. The certificate on the stock instance and in the traveller

- On receipt, the person (or a scanner) attaches the certificate PDF to the **stock instance**:
  supplier, order, heat or lot number, specification, alloy, temper, size, melt country, document
  type (EN 10204 2.1/2.2/3.1/3.2 or "MTR"), and the measured chemistry and mechanical values
  where present. Field names follow EN 10168 codes so a structured certificate imports directly.
- **Receipt check**: compares the certificate with the order (alloy, temper, specification,
  size, origin requirement) and shows mismatches. A mismatch blocks the stock from being issued
  to a job until a person accepts it with a reason (recorded).
- **Traveller link**: the inspection file's per-part record starts at "material issue". That
  entry points to the stock instance and its certificate, and each serial cut from that stock
  inherits the heat number. A finished-part report can then list the heat and the certificate.
- **The original document is kept unchanged** (with a hash), and any extracted values are marked
  as extracted.

### 4. Where the data lives

- **In the design:** only the stock **request** and the stock body (geometry, material, form),
  because these are part of how the part is made. Saving them is a new design-format entry
  (one file in `crates/doc/src/format/pending/`).
- **Outside the design, in a shop or job record:** quotes, orders, prices, certificates and
  heat numbers, because they belong to a particular batch, not to the design, and prices must
  not travel with a design shared in git.

## Data model sketch (opinion)

```text
StockRequest {
  setup_ref,                         // CAM setup chain this blank feeds
  shape: Plate { t, w, l } | RoundBar { d, l } | HexBar { af, l } | Tube { od, wall, l } | ...
  min_size_mm,                       // finished envelope + clean-up + grip, before rounding
  material: { alloy, temper, specs: [..], form: "cold-finished" | "rolled" | "cast-tooling" | .. },
  quantity: { parts, blanks, allow_longer_piece: bool },
  certificate: { required: "none" | "CoC" | "EN10204-3.1" | "MTR" | .., mill_report_required: bool },
  origin: {                          // see "Melt origin and DFARS specialty metals"
    clause_cited: "DFARS 252.225-7014", // as written on the customer's order
    clause_read_as: "DFARS 252.225-7009", // confirmed by a person
    allowed: "any" | "domestic-melt" | "us-outlying" | "us-outlying-qualifying",
    qualifying_list_as_of: date,
    specialty_metal: "yes" | "no" | "unknown", // predicted from the material card
  },
  allowances: [ { face, mm, source: "Op10 face pass" } ],
}

Offer {                               // snapshot, never edited
  supplier, connector_version, product_ref, delivered_size, cut_tol: { minus, plus },
  certificate, origin, price, delivered_price, currency, ship_date, taken_at, expires_at,
}

StockInstance {                       // the physical piece on the rack
  id, request_ref?, offer_ref?, order_ref?, received_at, measured_size?,
  certificate: { document_hash, type, heat, lot, spec, alloy, temper, melt_country,
                 en10168: { "A01": .., "B02": .., "C71": .. }, extracted: bool },
  receipt_check: { result, mismatches, accepted_by?, reason? },
  remnant_of?                         // a drop from another instance
}
```

## Phased plan (opinion)

1. **Stock request from the model**, no network: compute, explain, edit, save; min and max
   material blanks; remnant matching against a local list. Useful even with no supplier.
2. **RFQ export**: a drafted e-mail and a neutral file (JSON plus a readable PDF) any supplier
   can answer. Works offline until sent, works in the browser.
3. **Certificate attachment and receipt check** on the stock instance; the traveller's material
   issue entry (with the inspection work).
4. **First API connector** as a sandboxed plugin, against a supplier with a published API (Nox
   Metals is the only one found), using its test server, with the order step gated on the person.
5. **More connectors** as suppliers publish APIs or agree to integrations; prefilled web pages
   for the rest. Maintainers decide whether connectors live in the main repository or a separate
   one.

## Glossary

- **Blank:** the piece of stock a part is machined from.
- **Clean-up allowance:** material left on a face so that the first pass removes the raw surface
  completely.
- **Cut tolerance:** how far a delivered cut size may differ from the ordered size; often
  one-sided (-0 / +x).
- **Drop / remnant:** a leftover piece kept for later jobs.
- **EDI:** electronic data interchange, structured business documents (purchase order, order
  acknowledgement, ship notice, invoice) exchanged between company systems.
- **DFARS specialty metals:** US defence rules (252.225-7008/-7009) requiring certain steels, nickel,
  cobalt, titanium and zirconium alloys to be melted or produced in the US or allowed countries.
- **Heat number:** the identifier of one melt of metal at the mill; traceability links a part to it.
- **MTR / mill cert:** mill test report, the certificate of chemistry and properties for a heat.
- **Punchout:** a link from a buyer's purchasing system into a supplier's web store that returns
  the cart to the buyer's system for approval (cXML and OCI are common formats).
- **RFQ:** request for quotation.
- **Temper:** the heat-treatment and work condition of an alloy (T6, T651, annealed, Condition A).

## Open questions

1. Should the stock request be part of CAM (a setup property) or of the design (a "raw part"
   configuration, as the physical properties file suggests for castings)?
2. Is a supplier connector a plugin (#1221) or a separate companion service? A plugin keeps
   everything in one sandbox; a service could receive webhooks.
3. Who maintains a connector when the supplier changes its API? The supplier itself, the
   community, or nobody (then it is disabled)?
4. Should SolidCraft define an open, EN 10168-based JSON certificate schema, written
   independently, and ask suppliers to emit it?
5. How should a shop's own inventory be shared between machines and people: a file in git, the
   Raspberry Pi edge device from the inspection file, or the shop's ERP?
6. Would suppliers offer a referral fee, and if so, is that acceptable for an open project?
   (Opinion: only if disclosed and never used to sort offers.)

## Sources

All accessed 2026-10-26.

### Suppliers

- Nox Metals home page: https://noxmetals.co/
- Nox Metals API overview: https://noxmetals.co/api-docs (OpenAPI download not read; robots disallowed)
- Nox Metals instant quote page: https://noxmetals.co/quote
- Nox Metals standard sizes: https://noxmetals.co/standard-sizes
- Nox Metals seed round announcement (June 14, 2026): https://noxmetals.co/blog/nox-metals-seed-round
- Nox Metals, "How AI Is Improving Metal Procurement for Aerospace Manufacturers" (Jan 20, 2026): https://noxmetals.co/blog/ai-metal-procurement
- Alro e-Business: https://www.alro.com/dataPDF/e-Business.pdf
- Alro MyAlro page: https://www.alro.com/divgeneral/ebiz_myalro.aspx and https://myalro.com/About.aspx
- Alro aluminium plate saw cutting: https://www.alro.com/dataPDF/Metals/AluminumPlateSawCutting.pdf
- Alro SuperSquare: https://www.alro.com/Resources/Webresources/Alrocom/Literature/PDFs/SuperSquare20.pdf
- OnlineMetals pro buyers guide: https://www.onlinemetals.com/en/pro-buyers-guide
- OnlineMetals cut to size: https://www.onlinemetals.com/en/cut-to-size
- OnlineMetals quote page: https://www.onlinemetals.com/en/buy/quote
- thyssenkrupp Materials NA acquires OnlineMetals.com (press-release listing, Aug 17, 2007): https://www.thyssenkrupp.com/en/newsroom/press-releases/thyssenkrupp-materials-na-acquires-onlinemetals-com-1875.html
- Speedy Metals product page, 3-1/4 in round 4140 cold finished, heat treated: https://www.speedymetals.com/p-1528-3-14-rd-4140-cold-finished-heat-treated.aspx
- Speedy Metals pickup page: https://speedymetals.com/t-pickup.aspx
- Metal Supermarkets about page: https://www.metalsupermarkets.com/about/
- Metal Supermarkets UK: https://metalsupermarkets.co.uk/
- Metal Supermarkets (Wikipedia): https://en.wikipedia.org/wiki/Metal_Supermarkets
- MISUMI configurable precision aluminium plate (model for configured blanks): https://api.uk.misumi-ec.com/vona2/detail/110300539030
- McMaster-Carr Product Information API (for comparison: approved customers, client certificate, product data and price, no ordering described): https://www.mcmaster.com/help/api

### Standards and rules

- EN 10168:2004, steel products, inspection documents, list of information (catalogue preview): https://iteh.es/catalog/standards/sist/9dcd36dc-b1c1-402d-b4b7-3bbb643f3b62/sist-en-10168-2004
- DFARS 252.225-7009, restriction on acquisition of certain articles containing specialty metals: https://www.law.cornell.edu/cfr/text/48/252.225-7009
- DFARS 252.225-7008, restriction on acquisition of specialty metals: https://qafcs.acquisition.gov/dfars/252.225-7008-restriction-acquisition-specialty-metals.
- Final rule adding 252.225-7008/-7009/-7010 and removing 252.225-7014, Federal Register 74 FR (29 July 2009): https://www.govinfo.gov/content/pkg/FR-2009-07-29/html/E9-17967.htm
- Proposed rule (21 July 2008) to remove 252.225-7014: https://www.acq.osd.mil/DPAP/dars/dfars/changenotice/2008/20080721/E8-16675.htm
- MIL-STD-31000, ASME Y14.100, MIL-STD-2175 / AMS-STD-2175, MIL-F-7190, DI-NDTI-80809B, AS9102: listed by Jason; not reviewed in detail here.
- Material Identity CoA schemas (AGPL-3.0; README only, code not read): https://github.com/material-identity/CoA-schemas

### SolidCraft

- `docs/physical-material-cards.md`, `docs/plugins.md` (#1221), ROADMAP.md; CAM: #1153, PR #4158.
