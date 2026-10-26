# Compliance research: ITAR and quality-system records from model to finished part

Can the compliance framework of an **ITAR-registered, AS9100-certified** shop be built into the
**modelling** stage of a design, and carried on through **simulation** and **manufacturing**
(CAM, inspection, the part traveller)? And can a plain **ISO 9001** shop use SolidCraft to
control **revisions, releases and corrective actions**? This file answers both, and treats the
quality standards as **profiles on one shared framework**: ISO 9001:2015 underneath, with
AS9100D (aerospace and defense), ISO 13485:2016 (medical devices) and IATF 16949:2016
(automotive) adding their own rules on top. ITAR is handled separately at each stage, because it
is law rather than a certification.

Short answer: **yes, mostly, and the design file is the right place to start**, because every
later record (analysis, program, inspection result, traveller entry, corrective action) needs
to point at *one exact, released revision* of the design. SolidCraft cannot be "certified" or
"ITAR compliant" by itself, and it should not try to become a quality-management system (QMS),
product-lifecycle-management system (PLM) or ERP. What it can do is make the records those
systems need **correct at the source, tamper-evident, and easy to hand over**.

This is a cross-cutting topic and leans on other research files from the same author rather than
repeating them:

- [High-security data model](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-2fa4g1/research/security-research-nsnlookupjn-high-security.md):
  **owns** ITAR/CUI markings as design data, the high-water mark, flow control to agents and
  vendors, the hash-chained audit log, FIPS crypto, air-gap and GovCloud deployment. This file
  uses those pieces and does not redefine them.
- [In-process inspection](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-y2w06n/research/cam-research-nsnlookupjn-in-process-inspection.md):
  **owns** tolerances as data, probing, the per-part record, AS9102 form contents, QIF, the part
  traveller and serialisation. This file adds what compliance needs on top (who signed, which
  revision, delta-FAI scope, record retention).
- [CAM Physics System](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-vgqo3g/research/cam-research-nsnlookupjn-physics-system.md):
  **owns** the simulation gate before posting and overrides with a recorded approval. This file
  makes that approval one kind of the general signature record below.
- [NSN part library](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-nc3c94/research/parts-research-nsnlookupjn-nsn-library.md)
  (#787): approved sources for standard parts, which matters for counterfeit-parts prevention.

Licence: MIT OR Apache-2.0, the same as SolidCraft (see the Licence section of CONTRIBUTING.md).

## About this research

- **Author:** Jason ([@nsnlookupjn](https://github.com/nsnlookupjn)). Idea and direction by
  Jason; source research and writing done with an AI assistant (Claude) in October 2026.
- **How it was produced:** web searches of public sources (eCFR for ITAR, FDA pages, certification
  bodies' and consultants' public summaries of the standards), each claim checked against the
  cited page where possible. The quality standards themselves (AS9100D, AS9102C, ISO 9001,
  ISO 13485, IATF 16949) are paid documents; clause numbers and wording here come from public
  summaries and general knowledge, and are **paraphrased, never quoted**. SolidCraft's own docs
  were read to fit the proposal to what exists: `docs/book/src/guide/files.md` (Design
  Properties, read-only open), `docs/design-container.md`, `docs/oracle.md` (reference tests),
  `skills/solidcraft-checking/SKILL.md` (designs kept in git), `ROADMAP.md` (Drawings milestone).
  No GPL code was read and no commercial CAD, CAM, PLM or QMS program was driven.
- **Not legal advice, and not a compliance claim.** An organisation is registered with DDTC and
  certified to AS9100; software is not. A program can (1) not undermine the organisation's
  system and (2) produce its evidence well. That is the whole scope here.
- **Marking:** "(unverified)" marks a claim not confirmed against a source. **Fact**,
  **opinion** and **proposal** are labelled where they mix. Dates are as of 2026-10-26.

> **[TODO Jason: personal experience]** How your shop (or shops you know) handle revisions,
> releases, FAI, travellers, NCRs and ITAR today; which tools; where records get lost or copied
> by hand; what an AS9100 auditor or a prime's source inspector asked to see. Label this as
> personal experience.

## Summary

1. **One record spine, many standards.** Every quality standard asks for the same few things in
   different words: controlled documents with revisions; a design process with reviews,
   verification and approved changes; identification and traceability; validated processes and
   software; nonconformity handling and corrective action; retained records. Build those once as
   **design data** (the "compliance thread"), then let a **profile** (ISO 9001, AS9100,
   ISO 13485, IATF 16949, or a customer's own) decide which records are required, which forms
   are produced and how long records are kept.
2. **Released revisions are immutable and content-addressed.** Releasing revision C freezes the
   design's content hash. Every downstream artefact (drawing, STEP, analysis, CAM program, probe
   program, inspection plan, traveller) records the hash it was made from. If the design changes,
   anything built from the old hash is shown **stale** before it reaches a machine. This is the
   single most valuable piece for every standard, and the cheapest to build first.
3. **Signatures are records, not checkboxes.** Each approval (design review, release, analysis
   sign-off, CAM override, FAI acceptance, MRB disposition) is a signature record: who, role,
   meaning, time, and the content hash signed. Chained into the high-security audit log. Optional
   cryptographic signing with the organisation's own keys (including smart cards).
4. **Change control from the model.** A change request names what is affected; SolidCraft can
   compute **which characteristics changed** between two revisions, so it can propose the scope
   of a **delta (partial) first article inspection**, which programs must be re-proved, and which
   analyses must be re-run.
5. **Corrective action links back.** An NCR or CAPA links to the design revision, characteristic,
   program revision, machine, serial numbers and inspection results. Because everything carries
   the same identities, "which other parts were made with this program on this machine?" is a
   query, not an archaeology project.
6. **ITAR at every stage, not only on export.** Marking (high-security file), who may *see* the
   data (US persons only unless licensed: a release in the US to a foreign person is an export),
   what may *leave* (vendors, cloud, agents, a foreign subcontractor) with a recorded licence or
   exemption reference, and **five-year** record keeping. G-code, probe programs, inspection
   data and setup sheets for an ITAR part are treated as technical data by default (opinion).
7. **Integrate, don't replace.** Small ISO 9001 shops get a built-in, git-friendly workflow;
   larger shops connect SolidCraft to their existing PLM/QMS/ERP through the command interface,
   open formats (STEP AP242, QIF, JSON) and events. SolidCraft never has to be the system of
   record for CAPA or supplier management.
8. **Validate SolidCraft itself.** AS9100D 8.5.1.1, ISO 13485 4.1.6/7.5.6 and FDA rules want
   production and QMS software validated. SolidCraft already has a strong start (83 reference
   tests against analytic and independent-kernel results, `docs/oracle.md`). A `validation
   report` command and version pinning per job would let a shop reuse that work as evidence.
9. **Answer to Jason's ISO 9001 question:** today an ISO 9001 shop can record part number and
   revision and keep designs in git, but release states, sign-offs, change orders and CAPA links
   do not exist yet. The proposal below adds them without changing any default.

## Fundamentals

### The regimes, in one table

| Regime | What it is | Who it binds | What it asks of design/CAM software (summary) |
|---|---|---|---|
| **ISO 9001:2015** | Generic QMS standard, certified by accredited bodies | Any organisation that chooses it | Controlled documented information (versions, approval, retention), design and development control, traceability, control of changes, nonconformity and corrective action. A 2026 revision is expected late 2026 with moderate changes (DNV). |
| **AS9100D** (2016) | ISO 9001 plus aviation, space and defense additions (IAQG; SAE AS9100 in the Americas, EN 9100, JIS Q 9100) | Suppliers to aerospace/defense primes, usually by contract | Adds configuration management, operational risk, product safety, counterfeit prevention, key characteristics, special processes, validation of production software, FAI per AS9102. Being re-issued as **IA9100**, targeted for late 2026 with a 2–3 year transition (Smithers, unverified final date). |
| **AS9102C** (2023) | First Article Inspection requirement | AS9100 suppliers, by flowdown | Form 1 (part number accountability), Form 2 (product accountability: materials, special processes, functional tests), Form 3 (characteristic accountability with actual values). Full or partial ("delta") FAI on defined triggers. |
| **ISO 13485:2016** | QMS for medical devices | Device makers and suppliers; incorporated into US law by FDA's **QMSR** (21 CFR 820), effective **2 Feb 2026** | Design history/medical device file, validation of software used in the QMS and in production, traceability (stronger for implants), records kept for the device's lifetime and at least 2 years. 21 CFR Part 11 still governs electronic records and signatures in the US. |
| **IATF 16949:2016** | Automotive QMS, used with ISO 9001 and the AIAG/VDA "core tools" (APQP, PPAP, FMEA, MSA, SPC) | Automotive suppliers | Special characteristics carried from design to control plan, product safety, PPAP submission, customer engineering specs reviewed promptly, record retention for the life of the part plus one year (unverified wording). |
| **ITAR** (22 CFR 120–130) | US export-control law for defense articles, services and technical data, run by DDTC (State Dept.) | US manufacturers and exporters of USML items; manufacturers must register even if they never export (§122.1) | Control *release* of technical data to foreign persons anywhere (including in the US), licences/exemptions for exports, records kept **5 years** (§122.5), end-to-end encryption carve-out for storage and transit (§120.54(a)(5)). |
| **EAR** (15 CFR 730–774) | Commerce Dept. export controls for dual-use and "600-series" military items moved off the USML | Same shops, often for the same part family | Similar data-handling logic, different classification (ECCN). Handled the same way as ITAR in the data model (export regime field). |
| **NADCAP** | Industry accreditation (PRI) of special processes: heat treat, chemical processing, NDT, welding, coatings, etc. | Suppliers doing special processes for primes that require it | Process records, pyrometry, certified operators. SolidCraft's role: carry the *special-process call-outs* and their certificate references from design to traveller. |
| **DFARS** counterfeit rules (252.246-7007/7008), **AS5553** (electronics), **AS6174** (materiel) | Counterfeit-prevention requirements | Defense contractors, by contract | Buy from approved/authorised sources, keep traceability, report suspects (GIDEP). SolidCraft's role: approved-source data on standard parts and materials, and traceability fields on the traveller. |

### What "compliance in the model" actually means

Every standard above is a **records** standard as much as a process standard. An auditor asks
three kinds of question, and each maps to something software can hold:

1. **"Show me the requirement."** Customer drawing or model revision, contract flowdowns,
   specifications (materials, special processes, finishes), key/special characteristics, export
   classification.
2. **"Show me you met it."** Analysis reports, design reviews, the CAM program that was proved,
   inspection results with actual values, material certs, special-process certs, the signed
   traveller.
3. **"Show me it's the same thing."** That the part number, revision, serial, program, setup and
   inspection result all refer to the same configuration. This is **configuration management**
   and it is where most audit findings come from (opinion, common auditor experience; unverified
   as a statistic).

Question 3 is the one software is uniquely good at, and the one paper and spreadsheets are
uniquely bad at.

### AS9100D clauses that touch design, simulation and manufacturing data

Paraphrased from public summaries (clause numbers from AS9100D; wording is ours):

| Clause | Topic | What it means for SolidCraft data (opinion) |
|---|---|---|
| 7.1.5 | Monitoring and measuring resources (calibrated, traceable; includes measurement software) | Probe and inspection results record the instrument/probe ID and its calibration due date; the Pi box and CMM import do the same (inspection file). |
| 7.5 | Documented information, including 7.5.3.2 records protection, retention and disposal | Released revisions locked; records retained per profile; nothing deleted silently. |
| 8.1.1 | Operational risk management | Risks linked to features, operations and machines (PFMEA rows); the CAM physics risk slider is one input. |
| **8.1.2** | **Configuration management** (identify, control changes, status accounting, audit) | Part number + revision + content hash for every design and artefact; baselines; effectivity; stale-artefact detection. |
| **8.1.3** | **Product safety** (critical items, safety-critical characteristics, awareness) | Flag safety-critical features and characteristics in the model; flag propagates to drawing, program, inspection plan, traveller. |
| **8.1.4** | **Prevention of counterfeit parts** | Standard parts and materials carry spec, approved sources and lot traceability fields. |
| 8.3.2–8.3.4 | Design planning, inputs, controls (reviews, verification, validation; 8.3.4.1 test results) | Requirements list in the design, design-review records, analysis results attached as verification evidence. |
| **8.3.5** | Design outputs, including **key characteristics** and critical items | KCs are model data (from PMI), not drawing text. |
| 8.3.6 | Design and development changes (reviewed, verified, approved; effect on product delivered) | ECR/ECO records with change impact computed from the model. |
| 8.4.3 | Information for external providers (flowdowns: specs, special processes, FAI, records retention, right of access, counterfeit, export) | A vendor package export carries the flowdown list and the export-control record. |
| **8.5.1.1** | **Control of equipment, tools and software programs** used to automate, control, monitor or measure production: validated before release for production and maintained | CAM programs, post-processors, probe programs **and SolidCraft itself** need validation evidence. |
| **8.5.1.2** | **Validation and control of special processes** | Special-process call-outs in the model; NADCAP/process cert references on the traveller. |
| **8.5.1.3** | **Production process verification** (FAI, per AS9102) | FAI forms generated from model characteristics; delta-FAI scope from change impact. |
| 8.5.2 | Identification and traceability (configuration, serials/lots, records of manufacture) | Part traveller keyed by serial, pointing at design hash and program hash. |
| 8.5.6 | Control of production process changes | Program, tooling, machine and setup changes are versioned and approved. |
| 8.6 | Release of products and services | Final acceptance signature on the traveller; certificate of conformance generated from it. |
| 8.7 | Control of nonconforming outputs (MRB; use-as-is or repair needs the design authority's approval) | NCR record linked to characteristic, serials and program; dispositions signed. |
| 10.2 | Nonconformity and corrective action (including human factors) | CAPA links to NCRs and to the design/program changes that fixed them. |

### The four standards side by side

| Need | ISO 9001:2015 | AS9100D adds | ISO 13485:2016 adds | IATF 16949:2016 adds |
|---|---|---|---|---|
| Document/revision control | 7.5.3 | 7.5.3.2 records; 8.1.2 configuration management | 4.2.4 documents; 4.2.3 medical device file | 7.5.3.2.2 customer engineering specs reviewed within a set time (10 working days, unverified) |
| Design control | 8.3 | 8.3.4.1 V&V test records; 8.3.5 key characteristics | 7.3 incl. 7.3.9 changes, 7.3.10 design and development file (the US "design history file") | 8.3.3.3 special characteristics; APQP |
| Traceability | 8.5.2 | serials/lots, configuration | 7.5.8–7.5.9, stronger for implantables | 8.5.2.1 |
| Software validation | (implicit in 7.1.5) | 8.5.1.1 production software | 4.1.6 QMS software; 7.5.6 production software; US: Part 11 and FDA's Computer Software Assurance guidance (unverified status) | via control plan and MSA |
| First production approval | none | AS9102 FAI | process validation (IQ/OQ/PQ) | PPAP |
| Risk | 6.1 | 8.1.1 operational risk | ISO 14971 (risk management for devices) | FMEA (AIAG-VDA 2019), control plan |
| Product safety | none | 8.1.3 | central throughout | 4.4.1.2 |
| Nonconformity, CAPA | 8.7, 10.2 | MRB rules; human factors | 8.3, 8.5.2 CAPA, complaint handling | 10.2.3 problem solving; 10.2.4 error-proofing |
| Record retention | organisation decides | customer/regulatory; often decades in aerospace (unverified) | device lifetime, at least 2 years from release | part active life plus one year (unverified wording) |

The table is the argument for **profiles**: the data is the same, the required set and the
output forms differ.

### ITAR in plain words, stage by stage

- **Technical data** (§120.33) includes the information needed to design, develop, produce,
  manufacture, assemble, operate, repair, test or modify a defense article: blueprints,
  drawings, models, instructions. A CAD model of a USML part is technical data. **Opinion,
  widely held in shops (unverified as a ruling):** the CNC program, setup sheet, probe program
  and inspection data for that part are also technical data, because they describe how to make
  and check it.
- **Export** (§120.50) includes a **release** of technical data to a foreign person *in the
  US* (sometimes called a deemed export). **Release** (§120.56) includes letting a foreign person
  see it. So "who can open this design, see this screen, read this traveller" is an export
  question, not only an IT one.
- **Registration** (§122.1): manufacturers of defense articles register with DDTC even if they
  never export. Registration is an organisation fact, not a design fact, but the registration
  code is useful on vendor packages.
- **Licences and exemptions:** sending technical data to a foreign subcontractor needs a
  licence (typically a DSP-5 for unclassified technical data) or an exemption (for example
  §125.4), and defense services need an agreement (TAA). The software cannot judge which applies;
  it can **require that a reference be recorded** before an export command runs on an ITAR
  design.
- **Encryption carve-out** (§120.54(a)(5)): sending, taking or storing unclassified technical
  data is not an export if it is end-to-end encrypted with FIPS 140-2 (or successor) modules or
  at least AES-128-comparable strength, and not sent to, stored in, or sent from a §126.1
  proscribed country. This is what makes encrypted cloud storage and encrypted sync workable;
  the details belong to the high-security file.
- **Records** (§122.5): records of manufacture, acquisition and disposition of defense
  articles, and of exports, kept for **5 years** from the transaction or licence expiry.
- **Destination control statement** (§123.9(b)): exports of defense articles, including
  technical data, carry a statement on the shipping/transfer documents (unverified for
  electronic transfers' exact form). A vendor-package export can add it.

### Digital thread and model-based definition (MBD)

- **Digital thread:** one connected chain of data from requirement to design to analysis to
  manufacturing to inspection to service, so that each record points at the exact configuration
  it belongs to. The US DoD made this policy (Digital Engineering Strategy 2018; DoDI 5000.97
  *Digital Engineering*, 2023).
- **MBD:** the 3D model carries the product definition (dimensions, GD&T, notes, materials,
  special processes) as **semantic PMI**, so no separate drawing is the authority. Standards:
  ASME Y14.41-2019 (digital product definition data practices), ASME Y14.47 (model organisation
  schema), MIL-STD-31000B (technical data packages), STEP **AP242** (ISO 10303-242) for exchange,
  **QIF** (ISO 23952) for measurement plans and results.
- **Long-term archiving:** **LOTAR** (EN/NAS 9300 series) defines how to archive 3D CAD with
  PMI for decades and retrieve it with proven fidelity, based on STEP AP242 and validation
  properties (volume, area, centroid). SolidCraft's reference tests already compare exactly
  those properties (`docs/oracle.md`), which is a natural bridge (opinion).

SolidCraft today: Drawings are in progress (ROADMAP, milestone 8); tolerances-as-data are
proposed in the inspection file. Compliance does not need to wait for full MBD: the record spine
works on part number, revision, hash and a characteristic list, and gains precision when PMI
lands.

## Accuracy

Accuracy here means: are the records **true, complete and the same configuration**?

- **Content hash as identity.** A SHA-256 over the canonical design (the same JSON SolidCraft
  already writes) identifies the configuration exactly. Two files with the same part number and
  revision but different hashes are a configuration-management failure, and SolidCraft can
  detect it (proposal). The JSON format is already stable enough to diff in git
  (`skills/solidcraft-checking/SKILL.md`); canonicalisation rules (key order, number formatting)
  need to be defined for the hash (open question).
- **Regeneration fidelity.** A released design is a feature history; a later SolidCraft version
  could rebuild it slightly differently. For compliance the **released geometry** must be frozen
  too: store the evaluated B-rep (the design container's neutral B-rep schema,
  `docs/design-container.md`) and/or a STEP AP242 copy with validation properties at release,
  and compare on reopen. A difference becomes a warning, never a silent change.
- **Analysis reproducibility.** An analysis result is evidence only with its inputs: solver
  name and version, mesh settings, material data source, load cases, boundary conditions,
  design hash. Store them with the result; re-running with the same inputs should reproduce it
  within a stated tolerance.
- **Measured values, not "OK".** AS9102 Form 3 and PPAP dimensional results want actual values.
  Probe and CMM imports (inspection file) supply them with instrument IDs.
- **Clock and identity.** Signature times come from the local clock; in a controlled shop that
  clock should be domain/NTP-synced (organisation's job). Identity comes from the OS login or
  the organisation's directory/smart card, not from a name typed into a field.

## Programs

What exists, so SolidCraft can fit in rather than compete (facts from public product pages;
**no product was driven** and nothing is copied):

- **PLM/PDM** (revision control, release workflows, ECO): enterprise PLM suites from the large
  CAD vendors; Aras Innovator (open-core); vault-style PDM bundled with commercial CAD; git or
  self-hosted git servers used as lightweight PDM by small teams.
- **QMS software** (documents, NCR/CAPA, audits, supplier management, training): many vendors
  serve AS9100 and ISO 13485 shops; FAI tools generate AS9102 forms from ballooned drawings.
- **Shop ERP/MES** (jobs, routers/travellers, material certs, time): products aimed at
  AS9100 job shops commonly hold the traveller and NCRs (unverified which ones hold FAI).
- **Measurement**: CMM software exports QIF or proprietary reports; ballooning tools number
  characteristics on PDFs.

The gap (opinion): the **CAD/CAM program is where the configuration is created**, but most
compliance tools receive it later as a PDF or STEP with the configuration identity typed in by
hand. That hand-off is where wrong-revision errors start.

## Complaints and requests

From public forums, consultants' articles and general shop knowledge (opinion unless sourced;
unverified as statistics):

- "We machined to the old revision." The program was made from rev B, rev C was released, no one
  re-posted. The most common and most expensive configuration failure in job shops.
- "Partial FAI scope is a guess." After a change, deciding which characteristics need re-FAI is
  done by eye comparing two drawings.
- "Balloon numbers drift between revisions," so Form 3 rows don't line up with the new drawing.
- "The traveller says operator signed, but who approved the deviation?" Approvals on paper or
  in email, separate from the part record.
- "We can't prove the CAM software or post-processor is validated" (8.5.1.1), or the evidence is
  a one-time prove-out never repeated after a software update.
- "ITAR files end up in the wrong place": emailed to a vendor, uploaded to a quoting site,
  synced to a personal cloud folder, opened on a laptop by a foreign-national visitor or
  employee without a licence.
- "Audit prep takes weeks" because records live in five systems and the links between them are
  in people's heads.
- ISO 9001 small shops: "we don't need a PLM, we need released vs. work-in-progress to be obvious
  and the shop floor to never get WIP."

## What users need

### 1. A compliance profile per design (and per organisation)

**Proposal.** An optional `compliance` block in the design, plus an organisation policy file
(the same machine-wide policy mechanism the high-security file proposes). No block = today's
behaviour exactly; nothing changes by default.

- **Organisation policy:** which standards apply (ISO 9001, AS9100D/IA9100, ISO 13485,
  IATF 16949), DDTC registration code if any, signature roles and who holds them, retention
  periods per record class, whether cryptographic signatures are required, which export paths
  are allowed (high-security flow control).
- **Design profile:** customer, contract/PO, applicable flowdowns (a list of clause references
  the customer imposed), export regime and classification (from the high-security marking
  block; this file only *reads* it), key/special/safety-critical characteristics, special
  processes, material specifications, applicable FAI/PPAP/process-validation requirement.
- **Profiles are data.** Each standard is a small, openly licensed definition file: which
  records are required before release, which signatures, which output forms. A customer-specific
  profile (a prime's supplier quality manual) is just another file, written by the shop.

### 2. Revision and release workflow (the ISO 9001 answer)

This is what an ISO 9001 shop needs first, and every other profile builds on it.

**States** (proposal):

```text
Work in progress ──submit──▶ In review ──approve + release──▶ Released ──supersede──▶ Superseded
       ▲                        │                               │
       └────── reject ──────────┘                               └──obsolete──▶ Obsolete
```

- **Work in progress (WIP):** editable. Banner in the viewport and on every export: "WIP, not
  for manufacture". CAM can program it, but **posting** to a machine from WIP needs the
  organisation's explicit allowance (prototype work), recorded.
- **In review:** frozen for editing; reviewers comment and sign. Reject returns to WIP with the
  comments kept.
- **Released:** content hash frozen; opens read-only (SolidCraft already has a read-only open
  path for files open elsewhere, `docs/book/src/guide/files.md`); any edit creates the next
  revision as WIP. Exports carry "Released, rev C, hash …".
- **Superseded / Obsolete:** still openable for records, banner says so, cannot be the source of
  a new program or vendor package without an override record.
- **Revision scheme** from policy: letters (A, B, … skipping I, O, Q, S, X, Z as many drawing
  standards advise; exact list unverified), numbers, or `A.1` for pre-release iterations. The
  existing free-text `revision` in Design Properties becomes the displayed label, filled by the
  workflow when compliance is on.
- **Baselines for assemblies:** releasing an assembly records the exact revision and hash of
  every component (a "baseline"). An assembly cannot be released over a WIP component.
- **Effectivity:** a revision can carry "effective from serial 0042" or "from date", so the
  traveller knows which revision applies to which part.

**Change control:**

- **ECR (engineering change request):** reason, requester, affected items. Created from an NCR,
  a CAPA, a customer change notice or an internal idea.
- **ECO (engineering change order):** the approved change; links the old and new revision.
- **Change impact, computed (proposal; the key differentiator):** SolidCraft compares the two
  revisions feature by feature and characteristic by characteristic and lists:
  - characteristics added, removed or changed (nominal, tolerance, datum reference);
  - faces whose geometry changed (persistent face names from the kernel, already a design
    goal in `docs/design-container.md`);
  - downstream artefacts made from the old hash: drawings, analyses, CAM programs, probe
    programs, inspection plans, vendor packages, all marked stale;
  - the proposed **delta-FAI scope** (AS9102), PPAP re-submission need (IATF), or design
    verification/validation re-run (ISO 13485 7.3.9).
  People review and sign the impact; the software proposes, never decides.
- **Customer approval:** for design-authority-owned parts (the shop is building to a customer's
  design) the shop cannot change the design itself; the ECR becomes a *request* to the customer
  and the workflow records the customer's change notice number.

**Git option for small shops (proposal):** for an ISO 9001 shop without a PLM, the workflow can
live in git: WIP on a branch, review as a pull/merge request, release as a **signed tag**
`P1042-rev-C`, hash recorded in the tag message. SolidCraft's commands do the git steps so no
one has to learn git. For ITAR work the git server must be inside the controlled boundary
(self-hosted, US-person administered); a public code-hosting service is not appropriate
(opinion; check the host's own export-control terms).

### 3. Signatures and the audit trail

- **Signature record:** signer identity (from OS login, directory or smart card), role
  (designer, checker, approver, quality, customer representative), **meaning** (authored,
  reviewed, approved, released, accepted, dispositioned), time, the **content hash** signed, and
  an optional comment. This matches the elements US rules list for electronic signatures:
  21 CFR Part 11 asks for printed name, date/time and meaning to be shown with the signed record
  and for the signature to be linked to the record so it cannot be copied to another
  (§11.50, §11.70); FAA AC 120-78A gives similar guidance for aviation (uniqueness, significance,
  scope, security, non-repudiation, traceability; summary, unverified wording).
- **Cryptographic option:** sign the hash with the organisation's keys (a key file, or a PIV/CAC
  smart card through the OS). Verification needs only the public key, so an auditor or customer
  can verify offline. Not required for ISO 9001; often expected for Part 11 (opinion).
- **Separation of duties** from policy: the author cannot be the sole approver; two-person rule
  for releasing safety-critical items or lowering an export marking.
- **Audit trail:** every signature and every state change goes into the **hash-chained audit
  log from the high-security file** (command dispatcher level, so UI, scripts and MCP agents are
  covered alike). An AI agent can draft, check and *propose*; it cannot sign. Signatures need a
  person's interactive confirmation (opinion; also the safest reading of Part 11 and of
  "programmatic calls never open dialogs": the sign command fails over MCP with a clear error
  instead of opening a dialog).
- **CAM overrides** from the CAM Physics System file are signature records with meaning
  "override", carrying the limit, margin and reason that file defines.

### 4. Modelling stage

- **Requirements in the design:** a list of design inputs (customer spec references, loads,
  interfaces, regulatory requirements), each linkable to the features or analyses that meet it.
  This is the ISO 13485 design-input trace and AS9100 8.3.3.
- **Characteristics as data:** from PMI when it exists; until then a characteristic list
  attached to faces/edges (inspection file). Each carries a stable **characteristic ID** that
  survives revisions, so balloon numbers do not drift. Flags: **key** (AS9100/AS9103),
  **special** (IATF: customer symbols such as safety or fit/function), **safety-critical**
  (AS9100 8.1.3, ISO 13485).
- **Materials and specifications:** material by specification (for example an AMS or ASTM
  number with condition), not only a render appearance; the physical-properties research file
  covers the property side. Approved-source list per material where the customer restricts it.
- **Special processes:** heat treat, plating, anodise, passivation, shot peen, NDT, welding as
  structured call-outs (process spec, class/type, areas affected as face sets, masking). They
  flow to the traveller as operations needing a certificate reference, and to AS9102 Form 2.
- **Standard parts:** inserted from the NSN/standard-parts library with part number,
  specification and approved sources (counterfeit prevention, AS9100 8.1.4, AS6174).
- **ITAR in modelling:** the marking and high-water mark (high-security file). Added here:
  **who may open**. In an ITAR profile the organisation policy lists who is a US person (or
  licensed foreign person, with the licence reference) and SolidCraft refuses to open a marked
  design for anyone else, logging the attempt. This is a convenience over the OS's own access
  control, not a replacement for it (opinion; the OS and file server remain the real boundary).
- **Design reviews:** a review is a record with participants, the revision/hash reviewed,
  findings and actions; findings can be pinned to faces in the model.

### 5. Simulation stage

- **Analysis as verification evidence:** each analysis (FEA, tolerance stack, thermal,
  mass properties, kinematics) produces a result record: inputs, solver and version, design hash,
  pass/fail against the linked requirement, and a reviewer signature. Stale when the design hash
  changes.
- **Solver verification:** like the kernel's reference tests, each solver ships with benchmark
  cases (analytic solutions; public benchmark suites such as those published by NAFEMS) and
  their results, so the shop can show the solver was verified for the class of problem
  (AS9100 8.3.4, ISO 13485 4.1.6, unverified as to what each auditor accepts).
- **Manufacturing simulation** (CAM Physics System): the simulation gate before posting is
  itself a verification record; its *Stop* overrides are signed (section 3).
- **ITAR in simulation:** analysis models and results of an ITAR design inherit its marking.
  Cloud solvers or GPU farms (GPU compute research file) are export paths and follow the
  high-security flow control; the §120.54(a)(5) encryption carve-out covers storage, but
  *processing* in a cloud in clear text needs the provider to be inside the boundary (opinion;
  GovCloud-style hosting).

### 6. Manufacturing stage (CAM, programs, setups)

- **Program bound to a released hash.** A CAM operation set records the design hash it was
  programmed from. Posting checks: design released (or prototype allowance recorded), hash
  matches the current released revision, simulation gate passed. Otherwise the post is blocked
  with the reason.
- **Program revision control:** programs get their own revision and release, approved by the
  programmer and (per policy) a second person after prove-out. The posted G-code file carries a
  header comment with part number, revision, program revision, design hash and the G-code's own
  hash.
- **Delivery integrity:** the G-code hash is recorded at release; the Pi box or DNC link can
  re-hash what the control actually loaded and log a mismatch (inspection and machine-health
  files). This closes the "someone edited the program at the control" gap; edits at the control
  become a recorded deviation, not invisible.
- **Post-processor and machine validation (AS9100 8.5.1.1):** each post-processor has a version
  and a validation record (test parts or simulated reference programs and their expected
  output). A changed post invalidates "proved" status on the programs it posted, prompting
  re-proof per policy.
- **Setups, tools and workholding:** setup sheets list tool assemblies, holders, workholding,
  work offsets and the datums probed, all versioned (tool-holding, workholding and machine
  library files). A changed tool assembly is a production-process change (8.5.6) the shop can
  see.
- **ITAR in manufacturing:** G-code, setup sheets and probe programs inherit the design's
  marking (opinion, as above). Sending a program to an outside shop is an export path:
  domestic US-person vendor = recorded transfer; foreign vendor = licence/exemption reference
  required before the vendor-package command runs. Machine-monitoring data sent to a machine
  builder's cloud (machine-health file) must not carry geometry or program text from an ITAR job
  unless that path is approved.

### 7. Inspection, FAI and the traveller

The inspection file owns the mechanics; compliance adds:

- **FAI from the model:** AS9102 Form 1 from the design profile (part number, revision, FAI
  reason, full or partial), Form 2 from materials, special processes and their certificate
  references on the traveller, Form 3 from the characteristic list with actual values. The forms
  are *renderings*; QIF results are the master record (inspection file's opinion, agreed here).
- **Delta-FAI scope** from the change impact (section 2), plus the other AS9102 triggers the
  shop records: change of manufacturing source, process, inspection method, location, tooling
  or material; a lapse in production of two years; a natural or man-made event affecting the
  process. SolidCraft can detect some of these itself (program, post, tool assembly or machine
  changed since the last FAI; last FAI more than two years ago).
- **PPAP (IATF profile):** the same data produce the dimensional results and the design record
  part of a PPAP package; the control plan rows come from special characteristics plus the
  probe/inspection plan (proposal; PPAP element list is AIAG's, not reproduced here).
- **Process validation (ISO 13485 profile):** IQ/OQ/PQ records for a validated machining
  process reference the program revision, machine and fixture; a change to any of them flags
  re-validation.
- **The traveller carries the compliance header:** part number, revision, hash, serial/lot,
  export marking banner, contract/PO, special processes with certificate fields, KCs and
  safety-critical items highlighted, and the signatures collected at each operation. Final
  acceptance (AS9100 8.6) is a signature; a certificate of conformance is rendered from it.
- **ITAR on the floor:** a printed traveller of an ITAR part carries the marking; the DataMatrix
  encodes a record ID, never technical data itself (opinion).

### 8. Nonconformance and corrective action (NCR / CAPA)

Answering Jason's question directly: corrective action is one of the places SolidCraft adds the
most value, because a CAPA is only as good as its links.

- **NCR record:** what is wrong (characteristic ID, measured vs. required), which serials/lots,
  where found, containment, disposition (rework, repair, use-as-is, scrap, return to supplier),
  who signed. For design-authority parts, use-as-is and repair need the customer's approval
  (AS9100 8.7), recorded with their reference.
- **Automatic context:** because the traveller points at design hash, program hash, machine,
  tool assemblies, operator and inspection results, the NCR is created with all of that filled
  in. The **suspect population** query is immediate: "every serial made with program rev 3 on
  machine VF2-02 since tool T12 was changed."
- **CAPA record:** root cause (with the method the shop uses: 5 whys, fishbone, 8D), actions,
  effectiveness check. Actions link to the **ECR/ECO** that changed the design or program,
  so the auditor sees NCR → CAPA → change → re-FAI → effectiveness in one chain.
- **Learning loops into the libraries:** a CAPA that found a wrong feed or an unsafe clamp
  pressure can propose an update to the shared tool/workholding/machine libraries (those
  files), with the CAPA ID as provenance.
- **SolidCraft holds the links, not necessarily the CAPA system.** A shop with a QMS keeps the
  CAPA there; SolidCraft stores the QMS's record ID and shows its status if the QMS exposes it
  (section 10).

### 9. Records retention and archiving

- **Retention classes** from policy: design records, analysis records, programs, inspection
  results, travellers, NCR/CAPA, export records. Each gets a period: ITAR 5 years minimum
  (§122.5), ISO 13485 at least device lifetime and 2 years, IATF part life plus one year,
  aerospace customers often much longer (life of the aircraft type is common, unverified).
  The **longest applicable** period wins.
- **No silent deletion:** deleting a released revision or a record under retention is refused;
  disposal after the period is a logged command, and **legal hold** blocks it.
- **Archive bundle (proposal):** `records.archive` writes a self-contained, signed folder:
  design JSON, evaluated B-rep, STEP AP242 with validation properties (LOTAR-style), QIF plan and
  results, traveller and signatures as JSON plus PDF/A renderings, audit-log segment and its
  hash chain, and a manifest with hashes. Readable without SolidCraft in 30 years; checkable
  with any SHA-256 tool.

### 10. Integration with existing QMS, PLM and ERP

Most AS9100 and ISO 13485 shops already own a QMS and an ERP; some own a PLM. SolidCraft should
**plug in**, not compete (opinion):

- **Identities everywhere:** part number, revision, design hash, program hash, characteristic
  ID, serial. If every system stores SolidCraft's IDs, links work in both directions.
- **Commands already are the API.** Every SolidCraft action is a command with schemas, scriptable
  from Python and MCP (`docs/mcp.md`). Compliance commands (below) give a PLM or QMS everything
  it needs to drive release, read state and fetch records.
- **Events out:** "revision released", "artefact stale", "post blocked", "NCR created" as JSON
  lines a connector can forward (file drop, local socket, or HTTP to an *allowed* endpoint
  under high-security egress policy).
- **Open formats out:** STEP AP242 (design), QIF (plan/results), JSON (records), PDF/A (forms).
- **External IDs in:** a design or NCR can store "this is QMS record NCR-2026-0113" or "PLM item
  10042 rev C"; the source system stays authoritative.
- **Who is the system of record?** Per record class, in policy: e.g. design and program =
  SolidCraft (or PLM), NCR/CAPA = QMS, job/traveller = ERP or SolidCraft. SolidCraft never
  overwrites a record owned elsewhere.

**Fit with SolidCraft's own PDM work:** open issues #2869 (versioned document server RFC),
#2995 (local file workspace with version browsing and pinned retrieval) and #3002 (version panel
with safe working copies) are the natural home for revisions and pinned retrieval. The release
states, signatures and stale-artefact checks here would sit on top of that versioning rather
than beside it (opinion; based on the issue titles).

For a **small ISO 9001 shop with nothing else**, the built-in workflow (sections 2, 3, 8) plus
the git option is enough to show an auditor controlled revisions, approved releases, change
records and corrective-action links.

### 11. Validating SolidCraft itself

AS9100 8.5.1.1, ISO 13485 4.1.6/7.5.6 and FDA expect software used in production or in the QMS
to be validated for its intended use, with effort scaled to risk. SolidCraft can make that
cheap:

- **Validation report command (proposal):** `validation.report` runs the reference tests
  (`docs/oracle.md`: 83 cases against analytic and independent-kernel results, each also
  exported to STEP and read back) plus CAM/post and solver benchmark cases, on the shop's own
  computer, and writes a signed report: SolidCraft version and build hash, OS, results, and
  date. Re-run after every update. This is "operational qualification" evidence a shop can hand
  to an auditor.
- **Shop-specific cases:** the existing "designs kept in git with a test file and frozen
  values" practice (`skills/solidcraft-checking/SKILL.md`) is exactly what a shop needs to
  validate its own typical parts. Add them to the report.
- **Version pinning:** a released design and a released program record the SolidCraft version
  and post-processor version used. An update does not change released artefacts; reopening in
  a new version compares and reports.
- **Release integrity** (high-security file): signed releases, SBOM, reproducible builds make
  "which software produced this" provable.
- **Long-term support:** regulated shops cannot update monthly. A slower, supported release line
  for production use (opinion) would help; a maintainer decision, not a requirement.

### 12. What must not change

- **No default changes.** No policy file and no `compliance` block = SolidCraft behaves exactly
  as today. Per the repository's rule, the new design data is **one** new file in
  `crates/doc/src/format/pending/`.
- **Programmatic calls never open dialogs.** Sign, release and override commands return a clear
  error when interactive confirmation is required and the caller is a script or agent.
- **Never crash on hostile records:** compliance blocks from other shops or customers are
  untrusted input, bounded and validated like everything else.

### Data model sketch

```text
CompliancePolicy (organisation, machine-wide file; absent = off)
  standards:         ["ISO9001:2015", "AS9100D", "ISO13485:2016", "IATF16949:2016", <custom>]
  ddtc_registration?: string
  roles:             { approver: [identity...], quality: [...], ... }
  separation:        { author_cannot_sole_approve: true, two_person: ["release.safety", "marking.lower"] }
  signatures:        { crypto: "none" | "keyfile" | "smartcard" }
  retention:         { <record class>: { years | "device_life+2y" | "part_life+1y" } }
  systems_of_record: { ncr: "qms", capa: "qms", traveller: "solidcraft", design: "solidcraft" }

ComplianceProfile (in the design; one pending format file)
  customer, contract, po
  flowdowns:         [ { source, clause, text_ref } ]
  requirements:      [ { id, text, source, satisfied_by: [feature | analysis | characteristic] } ]
  characteristics:   [ { id, faces, nominal, tolerance | gdt, flags: [key, special, safety] } ]
  materials:         [ { spec, condition, approved_sources } ]
  special_processes: [ { spec, class, faces, cert_required } ]
  fai:               { standard: "AS9102C" | "PPAP" | "process-validation" | none }
  export:            -> reads the high-security Marking block (not duplicated)

Revision
  label, state (wip | review | released | superseded | obsolete)
  content_hash, brep_hash, step_ap242_hash
  baseline:          [ { component, revision, hash } ]    # assemblies
  effectivity?:      { from_serial | from_date }
  eco?:              id
  signatures:        [ Signature ]

Signature
  identity, role, meaning, time, signed_hash, comment?, crypto_sig?

Artefact (drawing | analysis | program | probe_program | inspection_plan | vendor_package)
  id, revision, source_design_hash, own_hash, tool_versions, state, signatures
  stale: computed (source_design_hash != current released hash)

ChangeRecord (ECR/ECO)
  id, reason, origin (ncr | capa | customer | internal), from_rev, to_rev
  impact:            { characteristics_changed, faces_changed, stale_artefacts, delta_fai_scope }
  customer_ref?, signatures

Nonconformance / CorrectiveAction
  id, external_id?, characteristic_ids, serials, program_hash, machine, disposition,
  root_cause?, actions -> ChangeRecord ids, effectiveness, signatures

ExportRecord (per transfer of an ITAR/EAR design or artefact)
  what (hashes), to (person/org, country), path (vendor, cloud, media), authority
  (licence no. | exemption cite | domestic US person), dcs_added, time, signer
```

### Proposed commands (sketch)

| Command | What it does |
|---|---|
| `compliance.profile` / `compliance.profile.set` | Read/write the design's profile |
| `revision.submit`, `revision.approve`, `revision.reject`, `revision.release`, `revision.supersede`, `revision.obsolete` | Workflow states; release freezes hashes |
| `revision.compare` | Change impact between two revisions (characteristics, faces, stale artefacts, delta-FAI scope) |
| `change.request`, `change.order` | ECR/ECO records |
| `sign` | Add a signature (interactive only; refuses over MCP) |
| `compliance.check` | List what a profile still requires before release or posting (like the simulation gate) |
| `artefacts.stale` | Everything built from a non-current hash |
| `fai.forms`, `ppap.package` | Render AS9102 forms / PPAP dimensional results from QIF and the traveller |
| `ncr.create`, `capa.create`, `ncr.suspects` | Records and the suspect-population query |
| `export.authorize` | Record licence/exemption before a vendor package or transfer of a marked design |
| `records.archive` | Signed long-term archive bundle |
| `validation.report` | Run reference and benchmark tests and write a signed report |

All have input and output schemas, in line with the repository's command rules.

### What contributors could build first (opinion, smallest useful steps)

1. **Design content hash** in `document.inspect` and on exports (no workflow yet). Useful on its
   own: anyone can check two files are the same configuration.
2. **Source hash on derived artefacts** (STEP/3MF metadata, future CAM programs) and an
   `artefacts.stale` check.
3. **Release state** with read-only released revisions and WIP banners, opt-in.
4. **`validation.report`** wrapping the existing reference tests.
5. Then signatures, ECO/impact, and the profile files.

## Glossary

- **APQP**: Advanced Product Quality Planning (automotive).
- **AS9102**: aerospace First Article Inspection requirement; Forms 1–3.
- **Baseline**: frozen set of exact revisions of all items in a configuration.
- **CAPA**: corrective and preventive action.
- **Configuration management**: identifying, controlling changes to, recording the status of,
  and auditing a product's configuration.
- **Content hash**: a SHA-256 over a file's canonical content; same hash = same content.
- **DDTC**: Directorate of Defense Trade Controls (US State Department), administers ITAR.
- **Deemed export**: release of controlled technology/technical data to a foreign person inside
  the US (EAR term; ITAR treats it as an export by release).
- **Delta FAI**: partial FAI covering only characteristics affected by a change.
- **ECR / ECO**: engineering change request / order.
- **Effectivity**: the serials or dates from which a revision applies.
- **FAI**: First Article Inspection.
- **Flowdown**: a customer requirement passed down to the supplier and its sub-tier suppliers.
- **IQ/OQ/PQ**: installation, operational and performance qualification (process validation).
- **Key characteristic (KC)**: a feature whose variation significantly affects fit,
  performance, life or manufacturability (AS9100/AS9103).
- **LOTAR**: Long Term Archiving and Retrieval (EN/NAS 9300).
- **MBD / PMI**: model-based definition / product and manufacturing information in the 3D model.
- **MRB**: Material Review Board, decides dispositions of nonconforming product.
- **NADCAP**: accreditation programme for special processes, run by PRI.
- **NCR**: nonconformance report.
- **PPAP**: Production Part Approval Process (automotive).
- **QIF**: Quality Information Framework (ISO 23952), XML for measurement plans and results.
- **QMSR**: FDA Quality Management System Regulation, 21 CFR 820 incorporating ISO 13485.
- **Special characteristic**: IATF term for product/process characteristics affecting safety,
  regulatory compliance, fit, function or further processing.
- **Technical data**: ITAR §120.33 information for designing, making, operating or repairing a
  defense article.
- **USML**: United States Munitions List (ITAR Part 121).

## Open questions

1. **Canonical hash rules.** What exactly is hashed (feature history only, or also evaluated
   B-rep and metadata)? Which metadata changes (description typo) should *not* change the
   configuration hash? Probably two hashes: configuration and file.
2. **Where do signatures live?** In the design file (travels with it, but bloats and mixes
   concerns) or in a side ledger keyed by hash (cleaner, but can be separated)? Proposal: side
   ledger, with a summary of release signatures embedded at release.
3. **Identity source** for the desktop app, web app and air-gapped installs: OS account,
   directory (LDAP/AD), smart card, or an organisation key file. Probably pluggable.
4. **Is G-code technical data?** This file treats it as such by default for ITAR designs; a
   shop's export counsel decides. Should the default be configurable per policy?
5. **How strict is "open" access control** when the OS already controls the file? Refusal
   with logging, or warning only?
6. **IA9100 changes.** IA9100 is expected to add or tighten product safety, counterfeit
   prevention and possibly an information-security clause. Re-check against the published text
   when it is out.
7. **Customer profiles.** Can primes' supplier quality requirements be expressed as profile
   files without copying their (often proprietary) manuals? Probably as references and
   checklists the shop writes itself.
8. **Persistent face and characteristic identity across revisions** depends on kernel
   persistent naming; how robust is it today for change-impact purposes?

## Sources

- eCFR, 22 CFR §120.54 Activities that are not exports, reexports, retransfers, or temporary
  imports (end-to-end encryption carve-out at (a)(5)):
  https://www.ecfr.gov/current/title-22/chapter-I/subchapter-M/part-120/subpart-C/section-120.54
- Cornell LII copy of §120.54: https://www.law.cornell.edu/cfr/text/22/120.54
- Steptoe, "State Department publishes long-awaited ITAR rule on encryption and other excluded
  activities":
  https://www.steptoe.com/en/news-publications/international-compliance-blog/state-department-publishes-long-awaited-itar-rule-on-encryption-and-other-excluded-activities.html
- NDIA, Annotated ITAR: https://ndia.org/divisions/international/export-import-compliance-resources/annotated-itar
- Williams Mullen, "Is my company required to register under ITAR?":
  https://www.williamsmullen.com/blog/my-company-required-register-under-itar
- Smithers, "AS9100 D explained: what the 'D' means and what comes next" (March 2026; IA9100
  timeline): https://www.smithers.com/resources/2026/march/as9100-d-explained-and-what-comes-next
- NASA SMA, status of the development of IA9100:
  https://sma.nasa.gov/news/articles/newsitem/2024/04/17/status-of-the-development-of-ia9100-quality-management-systems-requirements-for-aviation-space-and-defense-organizations
- CVG Strategy, "IA9100 series standards due for release in 2026": https://cvgstrategy.com/ia9100-series-standards/
- DNV, "The draft ISO 9001:2026 has been released: what to expect":
  https://www.dnv.com/news/2025/iso-9001-2026-new-standard-update-released/
- FDA, Quality Management System Regulation (QMSR): https://www.fda.gov/quality-systems-regulation
- Covington, "FDA finalizes rule incorporating ISO 13485 into new QMSR":
  https://cov.com/en/news-and-insights/insights/2024/02/fda-finalizes-rule-incorporating-iso-13485-into-new-quality-management-system-regulation-qmsr
- Advisera community, AS9100 Rev D auditing of monitoring software (8.5.1.1 discussion):
  https://community.advisera.com/topic/as9100-rev-d-auditing-of-monitoring-software
- Elsmar Cove, software verification and validation AS9100:
  https://elsmar.com/elsmarqualityforum/threads/software-verification-and-validation-as9100.82203/latest
- 21 CFR Part 11 (electronic records; electronic signatures), §11.10, §11.50, §11.70:
  https://www.ecfr.gov/current/title-21/chapter-I/subchapter-A/part-11 (general knowledge;
  sections summarised, unverified wording)
- FAA Advisory Circular 120-78A, Electronic Signatures, Electronic Recordkeeping, and Electronic
  Manuals (general knowledge; unverified wording)
- SolidCraft docs: `docs/oracle.md`, `docs/design-container.md`, `docs/book/src/guide/files.md`,
  `skills/solidcraft-checking/SKILL.md`, `ROADMAP.md`.
- Related research by the same author (linked at the top): high-security, in-process
  inspection, CAM Physics System, NSN part library, physical properties, tool-holding,
  workholding, machine library, machine health, GPU compute.
