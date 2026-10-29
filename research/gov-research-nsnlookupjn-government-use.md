# Government research: SolidCraft inside government (DoD, NASA, civil agencies, allies, and the rest)

What it means for a **government employee** to use SolidCraft at work, whether the US
Department of Defense (DoD), NASA and other agencies would **adopt it in house**, what the same
question looks like for **allied and other foreign governments**, what an open-source CAD/CAM
program means when **unfriendly governments** can download it too, and a proposal for one
**government mode** with per-country profiles that covers every kind of government use from
public work to an isolated classified enclave.

This file builds on two earlier files by the same author and does not repeat them:

- [High-security research](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-2fa4g1/research/security-research-nsnlookupjn-high-security.md):
  the contractor side (CMMC, NIST SP 800-171, DFARS 7012, FedRAMP, ITAR carve-out, CUI
  markings, FIPS 140-3, SBOM), the admin policy file, egress inventory, markings, audit log,
  air-gap packs and GovCloud hosting. **Section numbers in brackets like [HS 4] point there.**
- [Compliance research](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-ymp2ma/research/compliance-research-nsnlookupjn-itar-quality.md):
  ITAR and quality records, release states, signatures, the military standards profile, and
  section 14 on bug and crash reports never carrying a controlled design. **[CR 14]** points there.

Licence: MIT OR Apache-2.0, the same as SolidCraft (see the Licence section of CONTRIBUTING.md).

## About this research

- **Author:** Jason ([@nsnlookupjn](https://github.com/nsnlookupjn)). Questions and direction by
  Jason; source research and writing done with an AI assistant (Claude) in October 2026.
- **How it was produced:** web searches of public sources (DoD CIO and Code.mil pages, NASA
  software engineering handbook, Army news, GSA rule summaries from law firms, BIS export pages,
  UK, Canadian, Australian and EU summaries), each claim checked against the cited page where
  possible. SolidCraft's own `SECURITY.md`, `docs/agent-chat.md`, `docs/web-runtime.md`,
  `docs/book/src/guide/updates.md` and the app `Cargo.toml` files were read to fit the proposal to
  what exists (main at commit `2016ba7`, 26 Oct 2026). No GPL code was read (BRL-CAD is cited
  from Army news only) and no commercial CAD program was driven.
- **Not legal advice.** Export control, sanctions and government ethics are legal questions with
  heavy penalties. This file says where to look and what the software can do; an export
  compliance officer (for an ITAR registrant, the Empowered Official), agency counsel or an
  ethics official decides.
- **Software is never "approved" or "certified" by itself.** An agency authorizes a *system* that
  includes the software, for a stated use. What a program can do is make that authorization
  cheap and quick, and never undermine it.
- **Marking:** "(unverified)" marks a claim not confirmed against a source during this research
  (often from the author's or the assistant's background knowledge). **Fact**, **opinion** and
  **marketing** are labelled where they mix. Dates are as of 2026-10-29.

**Author's experience (personal experience, not a cited source):**

- **Shop floor since 1987:** machine shop apprenticeship 1987 to 1991 from age 16, manual
  machines, then G-code at the control on Milacron-era CNC with no way to save a program.
- **CAD/CAM over about 35 years:** at Ford, Unisurf, Siemens, Sketchpad and NX (punch cards,
  then tape, then floppy disks); later CATIA, AutoCAD and Pro/Engineer; fluent now in Fusion,
  Mastercam, SOLIDWORKS and NX.
- **Engineering and software:** BTEC HND Electrical and Mechanical Engineering, certified welder,
  MEng, PhD in Materials Science. Programming since age 8 (machine code, BASIC, C++, C#,
  assembler, web, databases).
- **Government-facing work:** owns several engineering and software companies focused on
  engineering military systems and manufacturing and supplying military spare parts; has produced
  flight-safety components, nuclear submarine parts, satellite parts and weapons systems. Owner
  and operator of NSN Lookup Inc. That is the contractor side of the government relationship;
  this file looks from the other side, the government's own people.

**This research is not complete.** It is a starting point: the author will keep researching and
adding to it, and others (especially people who have pushed software through an agency approval)
are encouraged to correct it and contribute.

## Summary

1. **Can a government employee use SolidCraft? Yes, but not just by downloading it.** US policy
   is friendly to open source: the DoD CIO's 24 Jan 2022 memo says open-source software (OSS) is
   *commercial computer software*, must get *"equal consideration"* with proprietary products and
   sits at the front of an *"Adopt, Buy, Create"* order. What stops an employee is the ordinary
   rule for every program on a government computer: it must be on the organisation's approved
   software list and inside a system's authorization (Authority to Operate, ATO). Free does not
   skip that, and free does not skip accessibility law (Section 508) either.
2. **What the employee must watch (fact and opinion).** The design they make is government
   data, often CUI or export controlled, so the browser app hosted on the public internet, agent
   chat to a commercial AI endpoint, and bug reports with the design attached are all off limits
   for it unless the agency approves them. Personal devices are out for government data.
   Contributing code back is allowed as official duty under the 2022 memo, but ethics rules
   (no endorsement, no use of government time for private interests, financial conflicts) and the
   fact that federal employees' work has no US copyright (17 U.S.C. 105) shape how they do it.
3. **Would DoD and NASA adopt it in house? Not as their production CAD today; plausibly in
   niches soon, and more widely if the project does a short list of things.** There is real
   precedent: the Army's **BRL-CAD** has been open source since 2004 and the Army calls it the
   *"primary tri-service cross-platform solid modeling"* system for military analysis; NASA
   publishes **OpenVSP**, its own open parametric aircraft geometry tool; the German armed forces'
   IT provider chose the open **openDesk** suite in 2025 for digital sovereignty. What blocks
   SolidCraft today: it is young and supports only the latest release; one person signs the
   security policy; there is no STIG, no FIPS build, no SBOM or build provenance on releases, no
   accessibility conformance report, no company that can sign a support contract, and programs of
   record live in NX, Creo and CATIA with model-based technical data packages to match. The DoD's
   2025 **Software Fast Track (SWFT)** push says plainly that open source *"presents a significant
   and ongoing challenge"* because of unclear provenance, so supply-chain evidence is the price
   of entry.
4. **Where it fits first (opinion):** research labs, test ranges and depots that need a modeller
   on an isolated network with no licence server; expeditionary and shipboard additive
   manufacturing; schools and service academies; scripting and agent automation (the MCP server)
   inside a boundary; small agencies; and allied governments that want **sovereign** software they
   can inspect and build themselves.
5. **One government mode, many profiles (proposal, the centre of this file).** A single opt-in
   mode, set by an administrator's policy file and not by the user, with **four levels**
   (Public, Controlled, Export controlled, Isolated) and **country profiles** (US, UK, Canada,
   Australia, New Zealand, NATO, EU and member states) that supply the marking vocabulary,
   approved cryptography, export regime names and evidence report format. An **offline build
   flavour** with networking compiled out covers classified enclaves, where the strongest evidence
   is that the code is not there. The mode switches things off (egress, hosted services,
   commercial AI, crash attachments), switches things on (markings, audit, signed packs, FIPS
   provider, security report) and produces an evidence pack for the agency's authorization.
   It cannot authorize itself, clear people, decide classification or make an export ruling.
6. **Foreign governments.** Every allied defence ministry has the same shape of rules under
   different names: classification labels (UK OFFICIAL / SECRET / TOP SECRET; Australia adds
   PROTECTED; Canada PROTECTED A/B/C; NATO and EU their own), a supplier cyber standard (UK
   Def Stan 05-138 Issue 4 with Cyber Essentials; Australia's DISP and ISM; US CMMC), an export
   regime (UK Export Control Order, EU Regulation 2021/821, Canada's Controlled Goods Program,
   Australia's DSGL) and a national crypto approval. Canada has had an open-source-first IT
   directive since 2018. A profile per country is mostly *data*, not code.
7. **Unfriendly governments (defensive analysis).** MIT/Apache gives everyone the same rights,
   and no open-source licence may discriminate by user, so adversary governments can use it and
   nothing in the licence can stop them; a "no military use" clause would make it non-open-source
   and break the very DoD policy that lets the US use it. Published source code is generally not
   subject to US export rules; the real risks are **elsewhere**: (a) controlled design data
   leaking *into* public libraries, issues and crash reports; (b) US persons (contributors
   included) giving *defence services* to foreign military users even with public software;
   (c) hosted services, support and payments reaching sanctioned parties; and (d) the reverse
   threat, hostile contributions and dependencies, which is what the DoD worries about most.
   The mitigations are the same supply-chain controls friendly governments ask for anyway.
8. **Russia and drones (the hard case).** Russia can download public SolidCraft, as it can any
   open CAD program. US rules since 2024 restrict CAD/CAM *software* exports to Russia and bar US
   persons from giving Russia *IT support and cloud services* for design and manufacturing
   software; the EU and UK ban supplying CAD/CAM to Russia, including by download, and say nothing
   specific about free open source. So the project should geo-block its own hosted services, give
   no support to users in Russia or Belarus, keep every build public, and run a **standing
   export and sanctions watch** (weekly automated scan, monthly triage, quarterly written review,
   yearly adviser check). Party lists matter too: sponsors, accounts and marketplace parties are
   screened against the US Consolidated Screening List and the UK, EU and UN lists. GitHub
   enforces US sanctions itself but cannot geo-block a public repository, so the project keeps
   code public there and puts country rules on the downloads and hosted app it serves itself.

## Fundamentals

### How software gets onto a US government computer

| Step | What it is | Who owns it | What it means for SolidCraft |
|---|---|---|---|
| **Need and market research** | The 2022 DoD CIO memo puts OSS first: *Adopt* existing government or open-source solutions, then *Buy*, then *Create*. Refusing OSS only because it is OSS may be *"contrary to statutory and regulatory preferences"*. | Program office, engineering lead | Policy is on SolidCraft's side. Someone inside still has to want it. |
| **Licence review** | Use *"must conform to the terms of the OSS license"*; the memo names Apache-2.0, BSD, GPL, LGPL and MIT as acceptable licences for DoD releases. | Agency counsel | MIT OR Apache-2.0 is the easiest case. No indemnity, no auto-renewal, no venue clause (the clauses GSAR 552.212-4 strikes from commercial licences because of the Anti-Deficiency Act). |
| **Supply-chain and security review** | OSS *"must meet the same rigorous standards for SCRM and cyber threat testing as any other product"*; *"careful consideration must be made regarding the source"*; be cognizant of *"influence on OSS projects by foreign governments"* (2022 memo). | Cybersecurity office, SCRM office | Needs SBOM, provenance, signed releases, a vulnerability history and a clear maintainer list. |
| **Approved software list** | Each component keeps a list of software allowed on its networks; there is *"no official approved or recommended OSS product list"* across DoD (Code.mil OSS FAQ, July 2026). Names differ: the Navy's DADMS is one known example; Army AR 25-2 covers OSS sustainment and approval. | Component CIO | A local sponsor files the request, with the evidence pack. |
| **System authorization (ATO)** | The Risk Management Framework (DoDI 8510.01, NIST SP 800-37) authorizes a system, usually for one to three years; reviews often take **12 to 18 months** and do not carry over between services (Atlantic Council, June 2025). SWFT (memo of 24 Apr 2025) aims to speed this with supply-chain requirements and *"Federal Government-led risk determinations"*. | Authorizing Official | A desktop CAD tool normally rides inside an existing workstation ATO as an added application, not its own ATO (opinion). A hosted SolidCraft would need its own. |
| **Hardening** | DISA STIGs: the Application Security and Development STIG (version of 12 Feb 2025 on public STIG viewers) is the generic checklist for applications; vendors can write a product STIG through DISA's vendor process. | Program, vendor | No STIG exists for SolidCraft. A short hardening guide mapped to the ASD STIG items is a cheap first step. |
| **Accessibility** | Section 508 applies to ICT an agency *"buys, builds, maintains, or uses"*, including free software (Access*Ability, opinion piece citing the Revised 508 Standards). Agencies ask for an Accessibility Conformance Report (a filled VPAT). | Agency 508 office | SolidCraft builds egui with `accesskit` on desktop and web (app `Cargo.toml` files); how far screen readers reach the viewport, panels and dialogs is untested. Related: the UI for aging eyes request (issue #7426). |

**NASA (fact, from the Software Engineering Handbook):** NPR 7150.2 requirement SWE-041 makes
projects require suppliers to *"notify the project"* whether open source will be in delivered
code, for Classes A to C and safety-critical D and E software, because licensing needs legal
analysis of *"ownership, registration and licensing issues"*. NASA's own releases go through NPR
2210.1 (software release, including open source). For a NASA engineer using SolidCraft as a
tool, the centre's IT approval is the gate (the exact NASA process name was not confirmed;
unverified). If SolidCraft output becomes part of flight software or a safety-critical analysis
chain, NPR 7150.2's tool qualification and classification rules apply to that use (opinion).

**Civil agencies and law (fact):** the SHARE IT Act (signed December 2024; summaries by
Morrison Foerster and JD Supra) makes agencies share custom-developed code across government.
OMB's Federal Source Code Policy M-16-21 (2016) set open-by-default for a share of new custom
code. Both are about code agencies *write*; they make it normal for agency engineers to work in
public repositories, which helps SolidCraft get contributions from government.

### What changes for the employee personally

| Topic | Rule (fact, sources in the table's last column where checked) | Practical meaning (opinion) |
|---|---|---|
| **Install** | Only approved software on government equipment (agency configuration management, NIST SP 800-171 3.4.9 equivalent for agencies is NIST SP 800-53 CM-11) (control numbers unverified) | Ask the IT or cyber office first; bring the evidence pack (section "What users need"). |
| **Licence acceptance** | MIT/Apache impose no payment, indemnity or venue terms. Clicking "I agree" never binds the government to open-ended indemnity anyway (GSAR 552.212-4, effective 22 Feb 2018, Morrison Foerster summary) | No contracting officer is needed for the licence itself. |
| **The design is government data** | CUI rules (32 CFR 2002; DoDI 5200.48), export rules, classification | Use only the desktop app on approved storage. Not the public browser app, not agent chat to a commercial endpoint, not a GitHub issue with the design attached [CR 14]. |
| **Personal devices** | Government data stays on government systems (agency policy) | Do not "try it at home" on real work. |
| **Contributing code** | The 2022 memo *"authorizes government employees to contribute to OSS projects as part of their official duties"* (Code.mil OSS FAQ) and says improvements *"should be contributed back"* and DoD-specific forks *"should be avoided"* | Contribute through the agency's process (component CIO reporting for a first release). |
| **Copyright** | A work of the US Government (made by an employee as part of official duties) has no US copyright (17 U.S.C. 105) (Code.mil's own FAQ did not cover this; unverified detail) | The contribution is public domain in the US; it can still be merged into an MIT/Apache project. Whether SolidCraft's contributor terms (sign-off) fit public-domain contributions is an open question below. |
| **Ethics** | No endorsement of a product in an official capacity (5 CFR 2635.702); no use of government time or property for private purposes (2635.704–705); no official action on matters where the employee has a financial interest (18 U.S.C. 208) (citations from background knowledge; unverified against text) | Say "we use", not "we recommend". An employee who also owns a company that sells SolidCraft services, or a contractor that bids on work using it, must clear that with an ethics official. |
| **Export** | ITAR and EAR bind government employees too; releasing controlled technical data to a foreign person, even a colleague, needs authorization (unverified for each agency's internal procedure) | The ITAR switch [HS 13] matters as much inside an agency as at a contractor. |

### Precedents: open-source design tools inside government

| Tool | Owner | Status | What it shows |
|---|---|---|---|
| **BRL-CAD** | US Army Ballistic Research Laboratory, now DEVCOM Data & Analysis Center | Open source since 2004 (*"the first U.S. Army code still in active development"* to do so); *"primary tri-service cross-platform solid modeling"* CAD for military analysis; geometry engine of the AJEM vulnerability model; moved to GitHub around 2021 (Army news, 21 Sep 2021) | DoD will run open-source solid modelling on its most sensitive analysis work when it controls the code. It is a CSG/analysis modeller, not a parametric design and CAM tool, so it does not fill SolidCraft's role. (Licence noted as LGPL/BSD in background knowledge; its code was not read, per SolidCraft's rule.) |
| **OpenVSP** | NASA | NASA's parametric aircraft geometry tool, released as open source under the NASA Open Source Agreement (openvsp.org; release year 2012 unverified) | NASA engineers use and maintain open parametric geometry tools in their daily work. |
| **openDesk** | ZenDiS for German federal administration; BWI framework contract for the Bundeswehr | Framework contract reported 24 Apr 2025 (Interoperable Europe) to secure *"control and operational independence in the cyber and information realms"* | Allied defence ministries now buy open source *because* it is open, for sovereignty. |
| **Code.mil, code.nasa.gov, code.gov** | DoD, NASA, GSA | Catalogues of government open source | Government employees routinely work in public repositories. |

## Accuracy

What is solid, what is soft:

- **Solid (checked against primary pages):** the 2022 DoD CIO memo text and its "equal
  consideration", "Adopt, Buy, Create", SCRM and foreign-influence lines; Code.mil's OSS FAQ
  (updated 28 Jul 2026) on official-duty contributions and no DoD-wide approved list; SWFT dates
  and its RFIs; BRL-CAD history from Army news; NASA SWE-041; GSAR 552.212-4 dates; BIS's page on
  publicly available encryption source code; GitHub's 2019 sanctions restrictions and 2021 Iran
  licence (news); BWI–openDesk; Canada's 2018 open-source-first directive; Def Stan 05-138
  Issue 4 date and ISN 2025/04 (law firm and consultancy summaries).
- **Soft (summaries, not primary text):** the DoD ATO timelines (one expert's article); the FY2026
  NDAA section numbers (law firm summary); EU Cyber Resilience Act dates (consultancy summary);
  the DFARS proposed rule on foreign source-code reviews (law firm summary, year not on the page).
- **Background knowledge, marked (unverified) in place:** NIST SP 800-53 control numbers,
  government ethics citations, classification label sets, national crypto approval names,
  Canada's CPCSC, and any agency-internal procedure names other than those cited.
- **Moving targets:** SWFT is still being implemented; CMMC phase 2 starts 10 Nov 2026; the EU
  CRA's reporting duties began 11 Sep 2026 and full application is 11 Dec 2027. Re-check before
  relying on any of them.

## Programs

How other CAD/CAM programs get into government (public pages and general knowledge; no
commercial program was run):

- **Commercial CAD/CAM** (NX, Creo, CATIA, SOLIDWORKS, Mastercam, Fusion) reaches agencies through
  enterprise agreements, GSA schedules and resellers. The vendors supply what reviewers ask for:
  support contracts, accessibility reports, security documents, and in some cases government
  cloud regions for their cloud products (Fusion's and Onshape's government offerings exist;
  their current authorization levels were not checked; unverified). Licence servers are the
  recurring pain in isolated networks (author's experience).
- **BRL-CAD** reached the services by being *built* by the Army. **OpenVSP** by being built by
  NASA. Neither had to pass a vendor procurement.
- **Open-source infrastructure** (Linux, Kubernetes, PostgreSQL) reached DoD through companies
  that package, support and harden it, and through hardened container registries such as the
  Air Force's Platform One Iron Bank (background knowledge). That "supported distribution" route
  is the likely path for SolidCraft too: a company or foundation that ships a government LTS
  build with evidence, while the code stays open (opinion).

## Complaints and requests

What agency engineers and reviewers say about getting software in (public articles, plus author
experience where labelled):

- **"The ATO takes longer than the project."** 12 to 18 months, and no reciprocity between
  services (Atlantic Council). A desktop tool that needs nothing new from the network gets
  through faster than anything with a server (opinion).
- **"Where did this code come from?"** SWFT's opening complaint about open source. Reviewers want
  SBOMs, signed builds and a named, accountable maintainer.
- **"Who do I call?"** No support contract means no one to hold responsible; this is the most
  common objection to community open source in government (opinion, widely reported).
- **"It works only on the latest version."** SolidCraft's `SECURITY.md` supports only the latest
  release; government sites patch on a schedule and need a supported long-term release.
- **"Licence servers do not work on our isolated network."** (Author's experience with commercial
  CAD/CAM in controlled environments.) SolidCraft needs none: a real advantage.
- **"Our program is in NX/Creo/CATIA and the TDP must be native."** MIL-STD-31000B treats native
  CAD as the authoritative master [CR 13]. SolidCraft can be a secondary tool long before it is a
  program-of-record tool.
- **"Accessibility report, please."** Agencies routinely ask for it; free software is not exempt.

## What users need

### 1. Government mode (the central proposal)

One mode, one switch, many profiles. **Opinion throughout**, built on the existing proposals in
[HS 1–13]. Default off; no file means today's behaviour, so no default changes.

#### Who sets it

A machine-wide **policy file** (the [HS 1] policy file) placed by the agency's administrator,
read-only to users, signed by the organisation's key when the profile says so. It names the
**level**, the **country profile**, the allowed endpoints, the data root and the trust keys. The
user can see the mode (a banner and Help ▸ About ▸ Security) but cannot weaken it.

#### Four levels

| Level | For | Examples |
|---|---|---|
| **G0 Public** | Unclassified, releasable government work; public outreach; education | NASA education, a public-release drawing (Distribution A), a city's public works |
| **G1 Controlled** | Sensitive unclassified: US CUI and FCI, UK OFFICIAL-SENSITIVE, Canada PROTECTED A/B, Australia OFFICIAL: Sensitive and PROTECTED, NATO RESTRICTED, EU RESTREINT UE | Depot repair drawings, lab test fixtures, contractor-delivered models |
| **G2 Export controlled** | G1 plus ITAR/EAR or the national equivalent; a per-design flag, not a whole-machine level | Defence article parts, satellite hardware |
| **G3 Isolated** | Systems accredited for classified information (US Secret and above, UK SECRET, NATO SECRET...) or any enclave with no external network | Classified design networks, test ranges, ship and submarine engineering spaces |

G2 is a **flag on designs** inside a G1 or G3 machine [HS 13], because one engineer may handle
both controlled and export-controlled designs on the same day.

#### What each level switches

| Control | G0 | G1 | G2 (per design) | G3 | Requirement it serves (US / allied) |
|---|---|---|---|---|---|
| Update check | on, may be off | off (`SOLIDCRAFT_UPDATE_CHECK=0` today) | as G1 | **not compiled in** | least functionality (800-171 3.4.6; ISM application control) |
| Egress | anything | deny by default, allow-list [HS 2] | design may go only to endpoints flagged for export-controlled data | **no network code** | boundary protection (3.13.1, 3.13.6) |
| Public browser app and hosted cloud | allowed | blocked; self-hosted in GovCloud / national cloud only [HS 9] | US-person / national-only storage | none | FedRAMP / DoD IL; UK and Australian cloud assurance |
| Agent chat and MCP | any endpoint | local model or an endpoint inside the boundary (Bedrock in GovCloud, Azure Government) [HS 4] | endpoints approved for export-controlled data only | local model only | CUI flow control (3.1.3) |
| Markings | optional | required; country vocabulary; high-water mark onto exports [HS 3] | adds export banner, USML/ECCN or national entry | required, classification banner | CUI marking (32 CFR 2002); national marking policy |
| Crash and bug reports | as today | no design or parameters in any report, mock-up instructions only [CR 14] | as G1 | reports stay local; export by media | data minimisation; spillage prevention |
| Audit log | off | hash-chained, append-only [HS 5] | adds every export, share and marking change | as G1, plus media transfer events | audit (3.3); ISM event logging |
| Cryptography | default | **FIPS 140-3 provider** for US/Canada; national approved provider where a profile names one [HS 6] | as G1 | as G1 (local file encryption only) | 3.13.11; national crypto approval |
| Libraries | online | signed packs from the organisation's mirror [HS 8] | tool data shareable through the field allow-list [HS 12] | packs on scanned media only | supply chain; media protection (3.8) |
| Plugins | any | only plugins signed by keys in the policy | as G1 | as G1, no network capability granted | 3.4.9 user-installed software |
| Release channel | any | **LTS** builds only (see section 3) | as G1 | LTS offline build | flaw remediation (3.14.1) on a schedule |
| Evidence | none | `security.report` [HS 7] in the country's format | adds the list of export-controlled designs and their flags | adds the offline-build attestation | the agency's System Security Plan / ATO package |

#### The offline build flavour for G3

For classified and other isolated systems the strongest evidence is absence. A Cargo feature
set, `offline`, that compiles out every crate that can open a socket (HTTP client, TLS, the update
checker, agent chat transports, cloud IO), leaving the localhost control channel behind a policy
switch or removed too. CI builds it, `cargo xtask egress` proves the dependency graph has no
networking crates, and the SBOM shows it. A reviewer can then check one property with tools they
already trust, instead of reading a configuration. Same source, same tests, reproducible [HS 7].
Classified accreditation is still done by the system's own authority (for US contractors, DCSA
under the NISPOM rule, 32 CFR 117; for agencies, their authorizing officials) (unverified detail).

#### Country profiles

A profile is a **data file** in the repository (the same pattern as SolidCraft's locale and
library catalogues), reviewed by people from that country, never code. It holds:

| Field | US | UK | Canada | Australia | NATO / EU |
|---|---|---|---|---|---|
| Marking vocabulary | CUI, CUI categories (CTI, export controlled), distribution statements A–F (DoDI 5230.24); classified: CONFIDENTIAL, SECRET, TOP SECRET | OFFICIAL, OFFICIAL-SENSITIVE, SECRET, TOP SECRET (Government Security Classifications, 2014) (unverified detail) | PROTECTED A/B/C; CONFIDENTIAL, SECRET, TOP SECRET (unverified detail) | OFFICIAL, OFFICIAL: Sensitive, PROTECTED, SECRET, TOP SECRET (PSPF) (unverified detail) | NATO UNCLASSIFIED to COSMIC TOP SECRET; EU RESTREINT UE to TRÈS SECRET UE (unverified detail) |
| Supplier cyber standard (for evidence wording) | CMMC 2.0 / NIST SP 800-171 Rev 2 | Def Stan 05-138 Issue 4 (14 May 2024; mandatory for contracts with DEFCON 658 from 3 Nov 2025 under ISN 2025/04), Cyber Essentials / Plus, cyber risk profiles CRP0–CRP3 | ITSG-33 for government systems; CPCSC for defence suppliers (unverified) | ISM (ASD) and Essential Eight; DISP for defence industry (unverified detail) | national rules plus NATO security policy (unverified) |
| Crypto approval | FIPS 140-3 (CMVP) | NCSC guidance (no single equivalent of FIPS for most OFFICIAL work; unverified) | FIPS 140-3 (CMVP is run jointly with the Canadian Centre for Cyber Security) | ASD-approved algorithms and evaluated products (ISM) (unverified detail) | national (e.g. Germany BSI TR-02102, France ANSSI) (unverified) |
| Export regime names | ITAR (USML), EAR (CCL, ECCN) | Export Control Order 2008, UK Strategic Export Control Lists (unverified detail) | Controlled Goods Program, Export Control List (unverified detail) | Defence and Strategic Goods List, Defence Trade Controls Act (unverified detail) | EU Dual-Use Regulation 2021/821 plus national military lists (unverified detail) |
| Hosted cloud reference | FedRAMP; DoD IL4/IL5 | national cloud guidance (unverified) | Government of Canada cloud guardrails (unverified) | IRAP-assessed cloud (unverified) | national (Germany's sovereign cloud initiatives) (unverified) |
| Evidence report format | SSP-ready JSON/PDF mapped to 800-171 and 800-53 | mapped to Def Stan 05-138 controls | mapped to ITSG-33 | mapped to ISM controls | per nation |

Starting set (opinion): US first (the author's experience and the biggest user base), then the
Five Eyes (UK, Canada, Australia, New Zealand) because their structures are close, then NATO and
EU templates that a member state fills in. A profile can be **combined**: a UK contractor on a US
programme needs UK markings plus ITAR flags on the US-origin designs.

#### What government mode cannot do

Said plainly so no one oversells it:

- **It does not authorize anything.** The agency's authorizing official accepts the risk for a
  system. The mode makes that faster with evidence.
- **It does not decide classification or export status.** Original classification authorities
  and export officers do. The mode records their decision and enforces its consequences.
- **It does not protect the computer.** OS hardening, disk encryption, physical security,
  emanations security and cross-domain transfer are the system's job.
- **It does not clear people.** Need-to-know and US-person or national status come from the
  identity system [HS 9]; the app enforces what it is told.
- **It cannot stop a determined insider** from photographing a screen. It makes accidental
  release hard and deliberate release visible in the log.

### 2. An evidence pack an agency can file

What a sponsor inside an agency needs to submit with an approval request (opinion; each item
exists or is proposed in [HS 7]):

1. **Security report** from `security.report`: version, commit, provenance attestation, SBOM
   hash, crypto provider and certificate, policy in force, egress list.
2. **SBOM** (CycloneDX or SPDX) with licences and hashes, and `cargo auditable` data in the binary.
3. **Licence statement:** MIT OR Apache-2.0; `cargo xtask licences --check` output showing no
   GPL/LGPL/AGPL code (already a SolidCraft rule).
4. **Hardening guide** mapped to the DISA Application Security and Development STIG items, with
   the ones that do not apply to a local desktop tool marked "not applicable" and why.
5. **Accessibility Conformance Report** (VPAT format) for the desktop app, honest about gaps.
6. **Vulnerability history:** published advisories with affected and fixed versions.
7. **Maintainer and governance statement:** who can merge, who signs releases, how the signing
   key is protected, how a maintainer is added or removed.
8. **Network behaviour statement:** every network path, with the switch that disables it.

All eight are documents and CI jobs, not product changes, which is why they are the cheapest way
to move SolidCraft toward government use (opinion).

### 3. A government long-term-support release

`SECURITY.md` today supports only the latest release. Government (and regulated industry) needs
a release line that gets **security fixes only** for a stated period, so a site can patch on a
schedule without taking new features. Proposal: one LTS line per year, security fixes for 18
months, published advisories naming affected and fixed versions, and the offline build flavour
built from each LTS tag. Who pays for the backporting is the real question; a supporting company
or foundation is the usual answer (opinion).

### 4. Someone who can sign

Agencies buy accountability. Options, cheapest first (opinion):

- **A named security contact and a published key-protection policy** in the repository.
- **A fiscal host or foundation** (the patterns used by many open-source projects) that can sign
  a no-cost agreement, receive vulnerability reports formally and hold the signing keys.
- **Commercial support** by one or more companies (any company, including small US businesses),
  sold through normal contract vehicles. That makes SolidCraft a *commercial item* with a support
  line, which the FY2026 NDAA's commercial-first acquisition sections (1821–1828, King & Spalding
  summary) favour.

### 5. Making it easy for government engineers to contribute

- Accept contributions marked as US Government works (public domain in the US) and document how,
  next to the existing contributor sign-off rules.
- Keep the **one upstream, no forks** line the 2022 memo asks for: a government feature should
  land upstream behind the policy switch, not in a DoD fork.
- A `government` label and a short guide: what may be posted (nothing controlled, ever), how to
  write a reproduction without the real design [CR 14].

## Foreign governments

### Allies

The same pattern repeats with different names (sources in the table above and in Sources):

- **United Kingdom.** Defence suppliers work to Def Stan 05-138 Issue 4 (mandatory on DEFCON 658
  contracts from 3 Nov 2025), with Cyber Essentials expected wherever MOD-identifiable information
  is handled and Cyber Essentials Plus often required at higher cyber risk profiles; a third-party
  Defence Cyber Certification launched in mid-2025 (Logiq summary). Internally the MOD's security
  policy is JSP 440 (background knowledge; its current software-approval content was not checked).
  UK projects buying ITAR-controlled items carry US flow-down rules on top: a UK engineer working
  with a US-origin design needs both profiles.
- **Canada.** A 2018 Government of Canada directive tells departments to use open standards and
  **open source first** and to release their own code openly (GovFresh). Defence goods are
  regulated by the Controlled Goods Program, which works alongside ITAR (unverified detail). The
  joint US–Canada cryptographic module programme means a FIPS 140-3 build serves both.
- **Australia.** The Australian Signals Directorate's Information Security Manual (updated
  December 2025, adding AI guidance; Graylog summary) and the Essential Eight (which put
  **application control** at the centre) govern government systems; the Defence Industry Security
  Program covers suppliers (unverified detail). Application control means an allow-listed,
  signed binary: signing and hashes matter more there than features.
- **NATO.** NATO classified information and NATO-funded projects follow NATO security policy on
  top of national rules (unverified detail). A NATO profile is mostly the marking vocabulary and
  the rule that the strictest national profile in a project wins.
- **European Union and member states.** The EU Cyber Resilience Act now reaches software sold in
  the EU: reporting duties began 11 Sep 2026 and full application is 11 Dec 2027; a legal person
  that gives sustained support to open source used commercially is an *open-source steward* with
  light duties (a security policy, vulnerability handling, cooperation with authorities), not
  fines, while individual volunteers stay out of scope (noze summary of the Commission guidance).
  If a company starts selling SolidCraft support in Europe, it becomes a manufacturer for that
  product under the CRA. Germany's ZenDiS and the Bundeswehr's openDesk contract show the
  sovereignty demand; a government CAD/CAM tool that a ministry can build from source at home fits
  that demand exactly (opinion).

### Why allies may move first (opinion)

For an allied ministry, US commercial CAD/CAM is a foreign product with foreign cloud ties. An
open program they can inspect, build, host in-country and patch themselves answers the
sovereignty question that a US vendor cannot. That is a stronger reason than price, and it is the
reason openDesk exists.

## Unfriendly governments

Defensive policy analysis only: what open-source status means for the project and its
contributors, and how friendly governments can trust it.

### 1. They can use it, and the licence cannot stop them

- **Fact:** MIT and Apache-2.0 grant rights to anyone. The Open Source Definition forbids
  discrimination against persons, groups or fields of endeavour (background knowledge; OSI's
  definition, criteria 5 and 6), so no open-source licence can exclude a government.
- **Opinion:** adding a "no military" or "no adversary" clause would make SolidCraft
  non-open-source, would not be honoured by those it targets, and would remove it from the DoD
  memo's list of acceptable licences. The answer is not a licence clause; it is controlling
  **data** and **services**, which law already does.
- **Marginal benefit to an adversary is small** (opinion): capable states already have commercial
  and domestic CAD/CAM. Government mode's features (markings, audit, flow control) are defensive
  and generic; they help a defender more than an attacker.

### 2. Export status of SolidCraft itself

- **Published software is generally not subject to the EAR** (15 CFR 734.3(b)(3) and 734.7):
  software made available to the public without restriction, as SolidCraft's source is.
- **Encryption:** BIS states that publicly available encryption source code is *"not subject to
  the EAR once the email notification per section 742.15(b) is sent"*, and that object code is
  excluded when the corresponding source is also publicly available. SolidCraft uses standard TLS
  through rustls. BIS also warns that an item is *"not considered publicly available merely
  because it incorporates"* open source; it is judged as a whole. A 2021 BIS rule narrowed the
  notification duty to non-standard cryptography (background knowledge; unverified against the
  rule text). **Action for the project (opinion): ask an export adviser once whether a 742.15(b)
  notification is needed for SolidCraft and record the answer in the repository.**
- **ITAR:** information that is in the *public domain* is not technical data (22 CFR 120.34),
  but only if it was put there lawfully. SolidCraft's general-purpose code is not ITAR technical
  data (opinion). A feature written under a defence contract, specially for a defence article,
  could be, until it is approved for public release (DoD public release review; distribution
  statement A) (background knowledge; unverified).
- **CAM and machine-tool controls:** dual-use lists control some software for numerically
  controlled machine tools and their development (EAR Category 2; Wassenaar), but published,
  publicly available software is outside the EAR either way (BIS; New America and Privacy
  International commentary on export controls and open source). Hosted CAM services, support and
  any unpublished build are a different matter (opinion).
- **UK and EU:** both exempt information "in the public domain" from technology controls under
  the general technology and software notes (HEECA and university export-control guidance); the
  EU Dual-Use Regulation 2021/821 has the same structure (unverified detail).

### 3. Where the real risk is

| Risk | Example | Mitigation (opinion) |
|---|---|---|
| **Controlled data leaks into public places** | A defence engineer attaches a real design to a bug report; a tool record carries a part number; an NSN library contribution includes restricted data; a crash folder ships with a design [CR 14] | G1–G3 strip designs from all reports; ITAR flag blocks public shares [HS 13]; the tool-data field allow-list [HS 12]; library contribution review with a "controlled?" question and automated scans for markings |
| **Defence services by US persons** | A US contributor (or support company) helps a foreign military user design or modify a defence article, using public SolidCraft | Not something software can prevent. A plain project policy: general help on public features is fine; help designing a specific weapon part for a foreign military is not, and the project does not do it (ITAR defence services, 22 CFR 120.32 in the reorganized ITAR; background knowledge, unverified numbering) |
| **Services, support and payments to sanctioned parties** | A hosted SolidCraft cloud, paid support, a shared library server, or sponsorship from a person on a sanctions list | Hosted services screen users against OFAC and EU/UK lists and block comprehensively sanctioned regions (GitHub itself blocked private repos and paid services in Iran, Syria, Crimea, Cuba and North Korea in 2019 while keeping public repositories open everywhere, and later obtained an OFAC licence to restore service in Iran in January 2021) |
| **Aggregation** | A shared library slowly assembles performance data for controlled processes, contribution by contribution | Field allow-lists and the rule that only generic tool data is shared [HS 12]; the library operator reviews categories, not just entries |
| **Hostile contributions** | A long-trusted contributor slips in a backdoor (the 2024 xz-utils case is the well-known example; background knowledge); a dependency is taken over | see next section |

### 4. The reverse: adversary-origin code as a supply-chain risk

This is the part friendly governments care about most, and it is what SWFT and the 2022 memo
point at.

- **Policy signals (fact):** the 2022 memo tells program managers to watch *"influence on OSS
  projects by foreign governments"* and to treat OSS under the same SCRM and testing rules as any
  product; SWFT's first public complaint is lack of visibility into where code comes from; a DoD
  proposed rule implementing the FY2019 NDAA would make contractors disclose whether foreign
  governments or persons were allowed to *review* source code of products DoD uses, with products
  containing only open-source software able to be acquired without those disclosures (Steptoe
  summary; status: proposed); the FY2026 NDAA bars DoD contractors from using DeepSeek AI
  (section 1532, King & Spalding summary), a sign that origin of AI tools in the development chain
  is now a contract question; Section 889 (FY2019 NDAA) already bars certain Chinese telecom and
  video equipment, which matters for the Edge box hardware rather than the software (background
  knowledge).
- **What SolidCraft can show (opinion):**
  - **Signed commits and signed releases**, with the release key protected and documented.
  - **Two-person review** on every merge, and CODEOWNERS for security-sensitive paths (file IO,
    importers, the control channel, MCP, update, crypto).
  - **Reproducible builds** so anyone, including an agency, can rebuild and compare [HS 7].
  - **Dependency provenance**: `cargo deny` already checks sources and advisories; add a
    per-release list of crate owners for new dependencies and a rule that a new dependency or a
    change of owner needs a maintainer's explicit review.
  - **Public scorecard** (OpenSSF Scorecard) and the SBOM, so reviewers see the same facts.
  - **AI-written code:** much of SolidCraft is written by AI agents under maintainer review
  (AGENTS.md). Reviewers will ask which models and services were used and where; a short,
  honest statement in the governance document is better than silence.
  - **A government LTS branch maintained by vetted people** (for example US or allied nationals
  working for the support company) that only takes reviewed security backports. This answers the
  "who touched this build" question without closing the open project (opinion).

### 5. Risks for contributors like the author

Opinion, not legal advice:

- A contributor who is also an ITAR registrant or defence manufacturer must keep **everything
  posted public-source and generic**: no design, drawing, process sheet, test or inspection data
  from controlled work, even as a "small example". The author's earlier rule already says only
  tool data is shareable [HS 12].
- Research files built from public sources (like this one) are fine to publish; experience
  stated in general terms is fine.
- Answering a foreign user's general question about a SolidCraft feature is ordinary support;
  helping them design a defence article is a defence service. When in doubt, ask the company's
  Empowered Official before replying.
- A contributor who becomes a government employee takes on the ethics rules above; a contributor
  who owns a company selling SolidCraft services must keep the two roles apart in any government
  decision.

### 6. Worked example: can Russia download SolidCraft and use it to design drones that attack Ukraine?

**Short answer (opinion, not legal advice): technically yes, as it can download any public
open-source CAD program today; legally, the public download is probably not where the project's
exposure lies, but hosted services, support and anything done knowingly for a Russian military
user are, and the EU and UK rules are less clear than the US ones for free downloads.**

What the rules say (fact, from the sources named):

| Rule | What it covers | Does it reach SolidCraft? |
|---|---|---|
| **US EAR, 15 CFR 746.8** (BIS rule of 12 Jun 2024, software part effective 16 Sep 2024) | Licence requirement for exports to Russia and Belarus of listed **EAR99 enterprise software, explicitly including CAD and CAM** (also ERP, CRM, PLM, BIM, CMMS, ETO) and **updates to previously deployed software** (Gibson Dunn summary) | The rule controls items *subject to the EAR*. Published, publicly available software is outside the EAR (734.3(b)(3), 734.7), so the public source and free downloads are most likely not covered (opinion; the summaries read do not address published software, so an adviser should confirm). A **non-public** build (a paid or private government LTS, a customised binary) would be subject to it. |
| **US OFAC determination** under EO 14071 (12 Jun 2024, effective 12 Sep 2024) | Bars US persons from providing to anyone in Russia **IT consultancy and design services**, and **IT support and cloud-based services for enterprise management software and design and manufacturing software** (Cleary Gottlieb summary) | A **hosted** SolidCraft (the browser app, a collaboration cloud, a library server) used from Russia, and **support** given by US persons to users in Russia, are squarely the kind of service this targets (opinion). Whether answering a public GitHub issue is "IT support" is an open legal question. |
| **EU Regulation 833/2014, Article 5n(2b)** | Bans selling, supplying, transferring, exporting or providing software for industrial design and manufacture, named as **BIM, CAD, CAM, ETO** and their components, to the Russian government and entities in Russia, including **downloads** ("in intangible form (e.g. download from a cloud…)") (Commission FAQ, 6 Feb 2024) | The FAQ is **silent on free and open-source software**. EU-based maintainers, mirrors and hosts should get advice; geo-blocking the project's own download site is the conservative reading (opinion). |
| **UK Russia (Sanctions) Regulations, amended by SI 2025/504** (in force 24 Apr 2025) | Restricts transfers of "sectoral software", including **industrial design applications**, for use in Russia or by Russia-connected persons, including downloads and cloud, with an exemption for **non-commercial and personal use** (Squire Patton Boggs summary) | Same uncertainty for UK-based people; no open-source exemption is described. |
| **Military end-user and Entity List rules** (EAR 744.21, Entity List) | Extra licence duties and a "knowledge" standard for military end uses in Russia and Belarus (background knowledge) | If anyone in the project **knows** a user is a Russian military or drone-programme entity, any help beyond what the public already has becomes a serious risk (opinion). |
| **GitHub** | Public repositories stay available everywhere; private repositories and paid services are restricted in sanctioned regions (2019 restrictions; GitHub trade-controls page) | GitHub handles its own compliance; it does not decide the project's own duties for its own website, cloud and support. |

What maintainers **can** do (opinion):

1. **Geo-block the project's own hosted services** (browser app, any cloud, library servers,
   update and download servers the project runs) for Russia, Belarus, Iran, North Korea, Cuba,
   Syria where still sanctioned, and occupied regions of Ukraine; screen accounts for any paid or
   signed-in service against OFAC, EU and UK lists.
2. **No support, consulting or custom work** for users in Russia or Belarus, or for anyone on a
   sanctions or Entity List, written into the project's support policy; walk away from any issue
   or chat that shows a military end use.
3. **Keep everything public.** The protection for open source rests on the code being published
   without restriction; a private "special build" loses it.
4. **Write it down.** A short `docs/export-and-sanctions.md` that states the project's position,
   the blocks in force, the date of the last review and who reviewed it.

What maintainers **cannot** do: stop someone in Russia from obtaining public source code from a
mirror or a copy. The licence cannot forbid it and law does not require the impossible; it
requires that people under its jurisdiction do not *supply* restricted items and services, and do
not knowingly help prohibited end users (opinion).

What it means for **contributors** (opinion): writing public, general-purpose CAD/CAM code is not
an export to Russia. Giving a Russian user help, testing their designs, or building a feature on
request for them could be. A contributor who is a US person, or in the EU or UK, should not
provide support to users in Russia or Belarus at all, and a contributor in the defence industry
should treat any such request as a matter for their export compliance officer.

### 7. A standing export and sanctions watch for maintainers

The rules change often (the EU adopts a new Russia package every few months; BIS and OFAC add
names monthly), so a one-time review is not enough. Proposal (opinion):

| Part | Proposal |
|---|---|
| **Sources** | US: Federal Register (BIS rules, DDTC ITAR amendments), BIS Entity List and Russia/Belarus pages, OFAC recent actions and SDN list, DDTC notices. EU: Official Journal (Regulation 833/2014 and 269/2014 amendments), Commission sanctions FAQs. UK: OFSI and Export Control Joint Unit notices to exporters, Russia regulations amendments. Also Canada (SEMA regulations), Australia (DFAT autonomous sanctions), Japan (METI), Switzerland (SECO), for maintainers or hosts located there. |
| **Automation** | A scheduled CI job (weekly) that queries the Federal Register API and EUR-Lex for keywords (`computer-aided design`, `CAD`, `CAM`, `design and manufacturing software`, `open source`, `publicly available`, `Russia`, `Belarus`, `software services`) and opens a `compliance-watch` issue listing new hits with links. No secrets, public APIs only. |
| **Cadence** | Weekly automated scan; **monthly** human triage of the issue by a named maintainer; **quarterly** written review entry in `docs/export-and-sanctions.md`; **yearly** review by a qualified export or sanctions adviser (pro bono help from a law clinic or a supporting company is common for open-source projects). |
| **Who** | One named compliance maintainer plus one backup; neither needs to be a lawyer, both need to know when to call one. |
| **Triggers for immediate action** | Any new rule naming CAD, CAM, design or manufacturing software, or software services; any guidance on open-source or publicly available software; a new comprehensively sanctioned country or region; the project starting a hosted service, paid support, a private build or a new cryptographic feature; a report that a sanctioned or military end user is using the project's services. |
| **Actions** | Update geo-blocks and screening; pause support to the affected place; update the policy document; announce changes in release notes. |
| **Records** | Every review dated, with the sources checked and the decision, so the project can show diligence if asked. |

The same watch serves government mode: the country profiles (section "Government mode") carry
the export regime names and are reviewed in the same quarterly entry.

**Party lists in the watch:** the weekly job also downloads the US Consolidated Screening List
(section 9) and the UK, EU and UN consolidated lists, and reports changes that match any name in
the project's own records (sponsors, paying customers of any project service, marketplace
participants). New Russia or Belarus designations of software, CAD or engineering companies
are flagged for the monthly triage.

### 8. GitHub and country filtering

GitHub is where SolidCraft's source, issues and release downloads live, so its rules are part of
the picture. What GitHub's trade-controls page says (fact, page read October 2026):

- **GitHub enforces US sanctions itself.** Services are not available to developers in **North
  Korea**. In **Crimea, Donetsk and Luhansk**, free public repositories stay available for
  personal communications; **private repositories, paid services, Sponsors and Marketplace are
  suspended** for restricted organisations, and paid Actions minutes stop. **Cuba** has general
  access. **Iran** is covered by an OFAC licence GitHub obtained (January 2021) for public and
  private, free and paid services.
- **Russia and Belarus** appear in GitHub's own export restrictions for GitHub Enterprise Server
  and Copilot (no sale or export without authorisation; Country Group E:1); public GitHub.com use
  is not blocked by region.
- **How GitHub decides:** IP address and payment history, not nationality; travel can affect an
  account; appeals go to GitHub Support.
- **What a repository owner cannot do:** GitHub *"does not currently offer the ability to
  restrict repository access by country."* A public repository is public everywhere GitHub
  serves.

What the maintainers **can** do (opinion):

| Option | Effect | Cost |
|---|---|---|
| **Keep the source public on GitHub** | Preserves the "published" status that keeps the code outside the EAR; a public repository cannot be geo-blocked anyway | none |
| **Serve release binaries and the browser app from the project's own site** (solidcraft.io) behind a CDN with country rules, and point GitHub release notes to it | Lets the project block comprehensively sanctioned regions (and Russia and Belarus, given the EU and UK CAD/CAM rules) for the things the project itself *supplies*: installers, updates, the hosted app | small; most CDNs offer country rules on free or low tiers (unverified per provider) |
| **Turn on GitHub Sponsors only through GitHub** (which already screens) and refuse direct payments without screening | Keeps money flows screened | none |
| **Self-hosted GitHub Enterprise Server or a self-hosted forge for a government LTS mirror** | Lets a supporting company host an access-controlled mirror for government customers (GitHub says Enterprise Server can hold export-controlled data, with the operator responsible) | licence and hosting |
| **Ask GitHub** | GitHub may say whether release-asset downloads can be region-restricted, and confirms the project's approach is consistent with GitHub's | one email |

Recommended action (opinion): keep the code public on GitHub; move the *download and update*
endpoints the project controls behind country rules; and send GitHub Support a short request so
the project's position is on record. Draft for the maintainers (not sent):

```text
Subject: Trade controls and country restrictions for release downloads (tangency-labs/solidcraft)

Hello GitHub Support,

We maintain SolidCraft (github.com/tangency-labs/solidcraft), an open-source (MIT OR Apache-2.0)
CAD/CAM application. Because the EU (Regulation 833/2014, Article 5n(2b)) and the UK
(SI 2025/504) restrict supplying CAD/CAM software to Russia and Belarus, and US rules restrict
related services, we want the downloads we supply to respect those rules while keeping our
source code public.

1. Can a repository owner restrict release-asset downloads, GitHub Pages or Packages by country
   or region, now or on a roadmap?
2. If not, is it consistent with GitHub's terms for us to keep source and releases public on
   GitHub while hosting installers and updates on our own site with country restrictions?
3. Does GitHub apply any restriction for Russia or Belarus to public repositories or release
   assets beyond what the trade-controls page describes?

Thank you,
<maintainer name>, on behalf of the SolidCraft maintainers
```

### 9. Party-based sanctions lists (SDN, Entity List and others)

Separate from country rules, governments list **people and companies** (fact, trade.gov):

- The US **Consolidated Screening List (CSL)** combines Commerce's Denied Persons, Unverified,
  Entity and Military End User lists; State's Nonproliferation Sanctions and AECA (ITAR)
  Debarred lists; and Treasury OFAC's SDN, Foreign Sanctions Evaders, Sectoral Sanctions,
  Non-SDN Chinese Military-Industrial Complex and other lists. It has a free search, downloadable
  files and an API, and trade.gov says it *"may be used as an aid"*; a match needs *"additional
  due diligence"* against the official lists.
- Other regimes keep their own lists: UK (OFSI consolidated list), EU (consolidated financial
  sanctions list), UN Security Council, Canada, Australia, Japan, Switzerland (background
  knowledge).

**Does this affect the project? Yes, wherever it deals with a person** (opinion, guided by the
Linux Foundation's OFAC guidance for open source):

| Activity | Risk | What to do |
|---|---|---|
| Someone listed downloads or uses public code | Low: the project is not transacting with them | Nothing beyond keeping it public |
| **Sponsorships, donations, paid support, grants** | High: US persons may not deal with SDNs or companies 50% or more owned by them | Take money only through screened channels; screen any direct payer against the CSL and UK/EU/UN lists |
| **Hosted services and accounts** (browser app with sign-in, cloud libraries, the collaboration cloud) | High | Screen at sign-up and on list updates; block listed parties and sanctioned regions |
| **Marketplace and supplier features** ([stock marketplace](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-um9nlo/research/cam-research-nsnlookupjn-stock-marketplace.md), supplier links, NSN supplier lookups, shared tool and machine libraries with contributor accounts) | High: the service brokers deals between parties | Screen every buyer, seller and contributor account; for US government supply also check SAM.gov exclusions (background knowledge) |
| **Support and two-way collaboration** with a developer employed by a listed company | Medium to high: the Linux Foundation notes two-way collaboration *"could be interpreted as a prohibited provision of services"*, and that OFAC has no blanket "publicly available" exemption like the EAR's | Maintainers do not mentor, review for or give support to people known to work for listed companies; ask an adviser before removing anyone (the Linux kernel removed several maintainers tied to sanctioned Russian companies in October 2024; background knowledge from news) |
| Accepting a one-way public contribution | Unsettled | Ask an adviser; record the project's policy |

**Should the project screen against the CSL?** Yes (opinion): at every point where money,
accounts or brokered deals exist, using the CSL API plus the UK, EU and UN lists, with matches
reviewed by a person and the result logged. For a desktop-only, donation-free project the
screening load is small; it grows with every hosted feature, which is why each research file
that proposes one now carries an export and sanctions section.

## Would DoD and NASA use it in house? A direct answer

Opinion, from everything above:

- **Today:** a curious engineer could get it approved on a lab or test network as a modelling
  aid, with a sponsor willing to write the request. It would not be chosen as a program's
  production CAD/CAM, and it would not be allowed near CUI on a standard network without the
  evidence pack.
- **With the evidence pack, an LTS line and a supporting organisation (perhaps 6–12 months of
  project work):** realistic for labs, depots, test ranges, schools, expeditionary additive
  manufacturing, isolated networks and agent-driven automation. NASA is the more natural first
  home because of its open-source culture and research centres; DoD the larger prize once SWFT's
  supply-chain requirements are published and met.
- **Program-of-record CAD:** only after mature STEP AP242 with PMI, model-based TDP delivery that
  satisfies MIL-STD-31000B [CR 13], and years of track record. That is a long road, and BRL-CAD
  shows the services will walk it with open source when the value is there.
- **Allies:** possibly sooner, for sovereignty reasons, especially in Europe and Canada.

## Proposed commands (sketch)

| Command | What it does |
|---|---|
| `government.status` | Level, country profile(s), policy file path and hash, what is blocked and why. Read-only. |
| `security.report` | [HS 7], with a `format` field per country profile (`us-800-171`, `uk-05-138`, `ca-itsg-33`, `au-ism`). |
| `government.evidence_pack` | Writes the eight-item evidence pack (section 2) to a folder for the sponsor. |
| `government.check_design` | Before a share or report: lists the markings and flags on a design and every destination the policy allows for it. |

All read-only or file-writing; none opens a dialog when called programmatically; schemas follow
the commands skill.

## What contributors could build first (opinion, smallest useful steps)

1. **Network behaviour statement and the egress check** [HS 2]: a document plus one xtask; no UI.
2. **SBOM and provenance on releases** [HS 7]: CI only.
3. **A hardening guide** mapped to the Application Security and Development STIG: docs only.
4. **An accessibility self-assessment** in VPAT format, listing gaps as issues.
5. **The `offline` build flavour** behind a Cargo feature, built in CI.
6. **The policy file and `government.status`**, then country profile data files, US first.

## Glossary

- **ACR / VPAT:** Accessibility Conformance Report, usually written on the Voluntary Product
  Accessibility Template, to show Section 508 conformance.
- **Adopt, Buy, Create:** the order of preference in the 2022 DoD CIO memo.
- **ATO:** Authority to Operate, the decision by an authorizing official that a system's risk is
  acceptable.
- **CRA:** EU Cyber Resilience Act.
- **CUI:** Controlled Unclassified Information (US).
- **DADMS:** the Navy's application and database management system used for approved software
  (one example of a component approved list).
- **Defence services:** under ITAR, help given to foreign persons in designing, making or using
  defence articles.
- **Def Stan 05-138:** UK MOD standard for cyber security of defence suppliers.
- **DISP:** Australia's Defence Industry Security Program.
- **Essential Eight:** Australia's baseline mitigation strategies, including application control.
- **FIPS 140-3:** US/Canada standard for validated cryptographic modules.
- **ISM:** Australia's Information Security Manual.
- **LTS:** long-term-support release line.
- **OFAC:** US Treasury's Office of Foreign Assets Control (sanctions).
- **RMF:** Risk Management Framework (NIST SP 800-37; DoDI 8510.01).
- **SBOM:** software bill of materials.
- **SCRM:** supply chain risk management.
- **STIG:** DISA Security Technical Implementation Guide.
- **SWFT:** DoD Software Fast Track initiative (2025).

## Open questions

1. **Contributor terms for US Government works.** How should SolidCraft accept a public-domain
   contribution from a federal employee alongside its existing sign-off rule? Other projects have
   handled this; their wording should be checked, not copied.
2. **742.15(b).** Does SolidCraft need a one-time encryption notification to BIS, given standard
   TLS through rustls? An export adviser should answer and the answer should live in the repo.
3. **Who backs an LTS line and signs for it?** A foundation, a fiscal host, a company, or several?
4. **How far does `accesskit` reach today?** A screen-reader test of the desktop app would turn
   the accessibility row from unknown into a list of issues.
5. **Which country goes second?** UK (Def Stan 05-138, many US programmes) or Canada
   (open-source-first policy, shared crypto programme)?
6. **The localhost control channel in G3.** Keep it (useful for agents on the same machine) or
   compile it out with everything else? Probably a policy switch, off by default in G3.
7. **What SWFT finally requires.** Watch the published framework and fit the evidence pack to it.
8. **Free downloads under EU Article 5n(2b) and UK SI 2025/504.** Does making free, public
   CAD/CAM software downloadable count as "supplying" it to Russia? Neither the Commission FAQ nor
   the UK summary answers this; it is the question to put to an adviser first.
9. **Is answering a public issue "IT support" under the OFAC determination?** If yes, the support
   policy must say so and maintainers must be able to tell where a reporter is.
10. **Who hosts solidcraft.io and the browser app, and under which country's law?** That decides
   which sanctions regimes the project's own services follow.

## Sources

Accessed October 2026. Quotations are short; everything else is paraphrased.

- DoD CIO, *Software Development and Open Source Software*, memo of 24 Jan 2022:
  https://dodcio.defense.gov/portals/0/documents/library/softwaredev-opensource.pdf
- Code.mil, Department of War Open Source Software FAQ (updated 28 Jul 2026):
  https://code.mil/oss-faq.html
- DoD CIO, Software Fast Track (SWFT) initiative page:
  https://dowcio.war.gov/In-the-News/Article/4367436/software-fast-track-initiative
- TechRadar, *Pentagon looks to shake up "outdated" software procurement* (7 May 2025):
  https://www.techradar.com/pro/pentagon-looks-to-shake-up-outdated-software-procurement-declares-war-on-open-source
- Atlantic Council, *The Pentagon's software approval process is broken* (4 Jun 2025):
  https://www.atlanticcouncil.org/blogs/new-atlanticist/the-pentagons-software-approval-process-is-broken-heres-how-to-fix-it/
- DoDI 8510.01, Risk Management Framework for DoD Systems (2022):
  https://dml.armywarcollege.edu/wp-content/uploads/2024/07/DoDI-8510.01-Risk-Management-Framework-for-DoD-Sys-2022.pdf
- DISA Application Security and Development STIG (STIG Viewer):
  https://stigviewer.com/stigs/application_security_and_development
- NASA Software Engineering Handbook, SWE-041 Open Source Software Notification:
  https://swehb.nasa.gov/spaces/7150/pages/16450601/SWE-041+-+Open+Source+Software+Notification
- NASA open source development: https://www.nasa.gov/nasa-open-source-development/
- About OpenVSP: https://openvsp.org/learn.shtml
- U.S. Army, *Army's powerful open source solid modeling system undergoes modernization*
  (21 Sep 2021): https://www.army.mil/article/249241/armys_powerful_open_source_solid_modeling_system_undergoes_modernization
- Morrison Foerster, *SHARE IT Act requires agencies to share custom-developed source code*:
  https://www.mofo.com/resources/insights/250113-share-it-act-requires-agencies
- Morrison Foerster, *Final GSA rule on commercial license terms that conflict with federal law*
  (Mar 2018): https://govcon.mofo.com/topics/final-gsa-rule-on-commercial-license-terms-that-conflict-with-federal-law
- Access*Ability, *Even free software needs VPATs*:
  https://accessability.substack.com/p/even-free-software-needs-vpats
- BIS, *Encryption items not subject to the EAR*:
  https://www.bis.gov/learn-support/encryption-controls/encryption-items-not-subject-to-ear
- eCFR, 15 CFR 734.7 (published):
  https://www.ecfr.gov/current/title-15/subtitle-B/chapter-VII/subchapter-C/part-734/section-734.7
- New America, *Export controls and open source software*:
  https://www.newamerica.org/insights/export-controls-and-open-source-software/
- HEECA, *'Public domain' exemption guidance* (Cranfield):
  https://exportcontrols.cranfield.ac.uk/videos/docs/HEECA%20'Public%20Domain'%20Exemption%20Guidance.pdf
- GeekWire, *Microsoft-owned GitHub restricts accounts in countries facing U.S. sanctions* (2019):
  https://www.geekwire.com/2019/microsoft-owned-github-restricts-accounts-countries-facing-u-s-sanctions-including-iran-syria/
- GitHub blog, *Advancing developer freedom: GitHub is fully available in Iran* (5 Jan 2021):
  https://github.blog/2021-01-05-advancing-developer-freedom-github-is-fully-available-in-iran
- GitHub, *GitHub and trade controls*:
  https://docs.github.com/en/site-policy/other-site-policies/github-and-trade-controls
- Steptoe, *DoD proposes new rules requiring disclosure of foreign access to software code*:
  https://www.steptoe.com/en/news-publications/international-compliance-blog/dod-proposes-new-rules-requiring-disclosure-of-foreign-access-to-software-code.html
- King & Spalding, *FY 2026 NDAA: domestic sourcing, AI, cybersecurity and acquisition reforms*
  (9 Jan 2026): https://www.kslaw.com/insights/articles/fy-2026-ndaa-domestic-sourcing-artificial-intelligence-cybersecurity-and-acquisition-reforms
- Logiq, *Navigating MOD cyber compliance* (Def Stan 05-138 Issue 4, ISN 2025/04):
  https://www.logiq.co.uk/insights/navigating-mod-cyber-compliance/
- NDISAC, Defence Standard 05-138 link page:
  https://ndisac.org/dibscc/links/ukregs/defence-standard-05-138-issue-3-cyber-security-for-defence-suppliers/
- Graylog, *Understanding the Australian Information Security Manual (ISM)*:
  https://graylog.org/post/understanding-the-australian-information-security-manual-ism
- GovFresh, *Canada adopts open source mandate for government software* (Dec 2018):
  https://govfresh.com/signal/canada-adopts-open-source-mandate-for-government-software
- noze, *Cyber Resilience Act: a practical guide after the Commission's clarifications*:
  https://www.noze.it/en/insights/cra-commission-guidance/
- Interoperable Europe (OSOR), *BWI/Bundeswehr chooses open source, adopting openDesk* (24 Apr 2025):
  https://interoperable-europe.ec.europa.eu/collection/open-source-observatory-osor/news/bwibundeswehr-chooses-open-source-adopting-opendesk
- Gibson Dunn, *New BIS export controls target Russian and Belarusian industrial sectors*
  (June 2024): https://gibsondunn.com/wp-content/uploads/2024/06/new-bis-export-controls-target-russian-and-belarusian-industrial-sectors-and-technology-infrastructure-and-crack-down-on-diversion.pdf
- Cleary Gottlieb, *US, UK and EU sanctions alignment: US IT and software sector service bans and
  export controls take effect*: https://clearygottlieb.com/news-and-insights/publication-listing/us-uk-and-eu-sanctions-alignment-us-it-and-software-sector-service-bans-and-export-controls-take-effect-as-russia-sanctions-continue-to-expand
- European Commission, FAQ on software-related sanctions against Russia (6 Feb 2024):
  https://finance.ec.europa.eu/system/files/2024-02/faqs-sanctions-russia-software_en%2C.pdf
- Squire Patton Boggs, *UK expands its Russia sanctions through new controls on technology,
  software and strategic goods* (Apr 2025): https://www.squirepattonboggs.com/insights/publications/uk-expands-its-russia-sanctions-through-new-controls-on-technology-software-and-strategic-goods/
- International Trade Administration, Consolidated Screening List:
  https://www.trade.gov/consolidated-screening-list
- Linux Foundation, *OFAC sanctions and open source collaboration*:
  https://bestpractices.linuxfoundation.org/ofac/ofac-open-source.html
- SolidCraft repository: `SECURITY.md`, `docs/agent-chat.md`, `docs/web-runtime.md`,
  `docs/book/src/guide/updates.md`, `apps/solidcraft/Cargo.toml`, `apps/solidcraft-web/Cargo.toml`,
  `AGENTS.md` (main at `2016ba7`).
