# IP research: protecting designs from theft, from NDA to finished part

How SolidCraft could become the best CAD/CAM program in the world at **tracking and helping to
prevent intellectual property (IP) theft**: digital fingerprints in every file that leaves the
program, signed provenance that proves who made what and when, NDA-aware sharing with suppliers
and job shops, evidence-grade records a court can use, and help for honest users to avoid
infringing other people's rights. It covers the five kinds of IP that touch a CAD model (trade
secrets and NDAs, copyright, design rights, patents and contract terms) in the main countries.

The guiding principle for all of it: **novices get safe protection by default with one switch,
and professionals get every control they have ever asked for.**

This file builds on three earlier files by the same author and does not repeat them:

- [High-security research](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-2fa4g1/research/security-research-nsnlookupjn-high-security.md):
  markings as design data, the high-water mark, flow control, announcing the marking to every
  connected system, the hash-chained audit log and the per-design ITAR switch.
  **[HS n]** points to section n there.
- [Compliance research](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-ymp2ma/research/compliance-research-nsnlookupjn-itar-quality.md):
  release states, signatures, retention, and section 14 on bug and crash reports never carrying a
  controlled design. **[CR n]** points there.
- [Government use research](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-qk5riz/research/gov-research-nsnlookupjn-government-use.md):
  the team policy ("no one goes to jail"), the golden rule (help users comply), rule packs per
  jurisdiction, and prevention-first compliance gates. **[GOV]** points there.

This file adds the **commercial** side: a customer's confidential design, a company's own trade
secrets, and the rights of third parties, none of which need a government marking.

Licence: MIT OR Apache-2.0, the same as SolidCraft (see the Licence section of CONTRIBUTING.md).

## About this research

- **Author:** Jason ([@nsnlookupjn](https://github.com/nsnlookupjn)). Questions and direction by
  Jason; source research and writing done with an AI assistant (Claude) in October 2026.
- **How it was produced:** web searches of public sources (statute and regulation texts on
  legislation.gov.uk, law-firm and university summaries, vendor help pages and published terms,
  a peer-reviewed survey of 3D mesh watermarking, the C2PA Rust crate documentation), each claim
  checked against the cited page where possible. The three earlier research files above, and their
  audit of SolidCraft's crash and bug-report paths (main at commit `2016ba7`), were reused. No
  GPL code was read and no commercial CAD program was driven. Competitor behaviour comes only
  from their public help pages and terms.
- **Not legal advice.** IP law differs by country, changes often and turns on facts. This file
  says where to look and what software can do. A lawyer qualified in the relevant country decides
  what a user must do, and items marked **[Lawyer]** need one before anything ships.
- **Marking:** "(unverified)" marks a claim not confirmed against a source during this research
  (often from the author's or the assistant's background knowledge). **Fact**, **opinion** and
  **marketing** are labelled where they mix. Dates are as of 2026-10-29.

**Author's experience (personal experience, not a cited source):**

- **Shop floor since 1987:** machine shop apprenticeship 1987 to 1991 from age 16, manual
  machines, then G-code at the control on Milacron-era CNC.
- **CAD/CAM over about 35 years:** at Ford, Unisurf, Siemens, Sketchpad and NX (punch cards,
  then tape, then floppy disks); later CATIA, AutoCAD and Pro/Engineer; fluent now in Fusion,
  Mastercam, SOLIDWORKS and NX.
- **Engineering and software:** BTEC HND Electrical and Mechanical Engineering, certified welder,
  MEng, PhD in Materials Science. Programming since age 8 (machine code, BASIC, C++, C#,
  assembler, web, databases).
- **Why this topic:** owns several engineering and software companies focused on engineering
  military systems and manufacturing and supplying military spare parts, and has produced
  flight-safety components, nuclear submarine parts, satellite parts and weapons systems. Owner
  and operator of NSN Lookup Inc. That work runs on customer drawings shared under NDA and
  flow-down terms, and on supplier chains where a drawing can pass through several shops. The
  author has seen both sides: holding other people's confidential designs, and having his own
  work copied.

**This research is not complete.** It is a starting point that the author will keep adding to.
Corrections are welcome, especially from IP lawyers and from people who run supplier portals.

## Team rules that apply here

The three rules from [GOV] apply to every proposal in this file:

- **No one goes to jail.** The project has no in-house counsel, so wherever a rule is unclear we
  take the most conservative reading. For IP that means: never ship a feature that helps someone
  break a licence, an NDA or a patent; never ship a watermarking or similarity method until its own
  patent position is checked [Lawyer]; never make a claim about a user's legal position.
- **Help users comply.** SolidCraft makes the rules visible, makes the safe choice the easy one,
  and stops and explains when an action would break a recorded NDA or licence. It never says "you
  are protected" or "you are not infringing". It says "this NDA or licence term applies to this
  action".
- **Prevention first.** Stopping a leak beats proving one afterwards. Every sharing feature is off
  by default, deny by default when on, and fails closed.

One more rule that is specific to this topic:

- **Never turn SolidCraft into spyware.** Every fingerprint, mark and record is made by the
  **design owner**, on the owner's **own** outgoing files, is **visible** to the owner, and is
  **listed** to anyone who opens a marked file in SolidCraft. SolidCraft never hides an identifier
  of its own user in their files, never phones home, and never reports anyone to anyone. (Colour
  laser printers that hide tracking dots in every page are the cautionary example; background
  knowledge, unverified here.)

## Summary

1. **No program can stop a determined thief; SolidCraft is open source, so pretending otherwise
   would be dishonest.** Anyone can build a copy with protections removed. Digital rights
   management (DRM) that tries to stop an authorised recipient from copying a file fails in
   closed programs too. So the goal is not DRM. It is four things that **do** work: **make the
   owner's protection legally strong, make leaks traceable, make authorship provable, and make it
   easy for honest people to stay honest.**
2. **Trade-secret law rewards the owner who took "reasonable steps".** The US Defend Trade
   Secrets Act (2016), the EU Trade Secrets Directive (2016/943), the UK regulations (2018),
   Germany's GeschGehG, Japan's Unfair Competition Prevention Act and China's Anti-Unfair
   Competition Law (amended, in force 15 October 2025) all protect only information its owner
   actively kept secret. A German labour court (BAG, 17 October 2024, 8 AZR 172/23) refused
   protection because the employer could not show what secrecy measures it had taken. **The
   biggest legal gift SolidCraft can give users is a record that proves their reasonable steps:
   markings, need-to-know access, NDAs linked to designs, and a log of every share.**
3. **Digital fingerprints, in layers.** Each file that leaves for a named recipient carries:
   (a) a visible notice ("Confidential, prepared for Acme Ltd under NDA 2026-014"); (b) signed
   metadata naming the owner, NDA and a pseudonymous recipient code; (c) an invisible
   **per-recipient fingerprint** hidden in choices that do not change the part (mesh vertex
   order, tessellation, G-code segment splitting, and optional sub-tolerance shifts only on faces
   marked non-critical); and (d) a **content hash** in a signed, timestamped provenance record.
   Strip one layer and the others usually remain. Removing copyright notices and rights
   information knowingly is itself unlawful in the US (17 U.S.C. 1202) and the EU (InfoSoc
   Directive Article 7) (unverified for exact scope).
4. **"Who leaked this?" in one command.** Drop a suspicious file (STL, STEP, 3MF, G-code, PDF)
   onto SolidCraft. It reads every surviving fingerprint layer, matches it to the owner's local
   share ledger, and if nothing survives, runs a **geometric similarity search** against the
   owner's designs. The answer is a ranked list with confidence, never an accusation.
5. **Provenance that a court can use.** Hash-chained audit log [HS 5], signed design revisions,
   and optional trusted timestamps (RFC 3161) prove what existed when. In the US, hash-verified
   copies of electronic data can be self-authenticating with a qualified person's certificate
   (Federal Rule of Evidence 902(14), in force 1 December 2017). An **evidence bundle** command
   packs the design, its history, hashes and log in one sealed archive for the owner's lawyer.
6. **NDAs as data.** An NDA register (parties, purpose, term, permitted recipients, return or
   destroy duty, governing law) is linked to designs. Sharing outside the NDA's recipients is
   refused; expiry raises return-or-destroy reminders; flow-down to sub-tier suppliers is
   recorded. Supplier platforms already work this way in their terms: Xometry's partner terms
   require written consent before subcontracting and return or destruction of customer files
   within 18 months of finishing a job.
7. **Share the minimum.** A share wizard asks the purpose (quote, make, inspect, assemble) and
   sends only what that purpose needs: a drawing and a dumb solid for a quote, never the feature
   history; G-code only for a machine; a coarse mesh for a fit check.
8. **Design law now reaches CAD files.** The EU design reform (Regulation 2024/2822, applying
   mostly from 1 May 2025) makes creating, downloading, copying and sharing a file that records a
   protected design, in order to make the product, an infringing act (Article 19(2)(d)). Patent
   law in the US treats the person who prints a patented object as the direct infringer, and the
   file host as a possible inducer. So SolidCraft's **public libraries** must carry licences and
   provenance, and a **licence check** warns before a library part is used against its terms.
9. **Patent and design awareness, optional and offline-safe.** An opt-in patent and design
   watch searches public databases for the user's keywords and classes, and a "design-around
   notes" field lets a team record what they checked. SolidCraft never says "free to operate".
10. **All of it is off by default.** Opening and saving ordinary designs works exactly as today.
    Markings and fingerprints are added in ONE new design-format file, and older designs open as
    unmarked.

## Fundamentals

### The five kinds of IP in a CAD model

| Right | What it protects in a CAD/CAM context | How it arises | Typical term | Key weakness |
|---|---|---|---|---|
| **Trade secret (and NDA)** | Unreleased geometry, tolerances, process know-how, CAM strategies, feeds and speeds, fixture designs, supplier lists, costings | Automatically, **only while kept secret by reasonable steps** | Unlimited while secret | Lost once public; reverse engineering of a sold part is usually lawful (unverified per country) |
| **Copyright** | Drawings, renders, manuals, software, sometimes the creative expression in a model | Automatically on creation (Berne Convention) | Life plus 50 to 70 years (unverified per country) | Does not protect function. A faithful digital copy of an existing object can lack originality (Meshwerks v. Toyota, 10th Cir. 2008). "Useful articles" are protected only for separable artistic features in the US (unverified) |
| **Design right / design patent** | The appearance of a product: shape, contours, ornament | Registration (EU RCD, UK registered design, US design patent); some unregistered rights | EU registered up to 25 years; US design patent 15 years; EU unregistered 3 years (unverified) | Appearance only; functional shapes excluded; must be new |
| **Patent** | A new technical function: a mechanism, a process, a material, a toolpath method | Application and examination | 20 years from filing (unverified) | Public by design; expensive; enforcement against file sharing is indirect (see below) |
| **Contract** | Whatever the parties agree: NDAs, supply terms, licences on library parts, EULAs | Signature or acceptance | As agreed | Binds only the parties; a third party who receives a leak may be outside it |

Trademarks matter too (a counterfeit part bearing a brand's logo), but they protect the mark, not
the design, so this file treats them only in the counterfeit section.

### What "reasonable steps" means in practice

Every major trade-secret law asks the same question: **did the owner take steps to keep it
secret?** Fact, from the cited texts:

- **EU (Directive 2016/943, Article 2(1)):** a trade secret is information that is secret, has
  commercial value because it is secret, and "has been subject to reasonable steps under the
  circumstances, by the person lawfully in control of the information, to keep it secret."
- **US (DTSA, 18 U.S.C. 1839(3)):** the owner "has taken reasonable measures to keep such
  information secret". The DTSA was signed on 11 May 2016. It also requires employers to include
  a **whistleblower immunity notice** (18 U.S.C. 1833(b)) in any agreement with an employee or
  contractor that governs trade secrets or confidential information, entered into or updated
  after that date; without the notice the employer cannot recover double damages or attorney fees
  against that person.
- **Japan (UCPA):** three requirements: secrecy management, usefulness and not publicly known.
  Secrecy management is the one companies most often fail. Examples: a "Confidential" marking,
  access rights and access logs for electronic data, and rules in work rules or a confidentiality
  agreement. A shared folder every employee can open is unlikely to qualify. METI publishes Trade
  Secret Management Guidelines (revised March 2025).
- **Germany (GeschGehG, 2019; BAG 8 AZR 172/23, 17 October 2024):** a standard-form clause
  binding an ex-employee to keep "all internal matters" secret forever was void, and the employer
  also lost its statutory claim because it could not show what secrecy measures it took, with
  "no confidentiality clauses and adequate technical and IT safeguards". Lesson: **specific
  markings on specific information, plus records of the measures.**

**What this means for SolidCraft (opinion):** the program can make "reasonable steps" visible and
provable at almost no cost to the user: a marking on the design, a need-to-know list, an NDA
linked to every outside share, and a log. It cannot decide whether the steps are "reasonable",
which is for a court.

### How IP reaches a CAD or G-code file

| Law | Does the file itself infringe? | Source |
|---|---|---|
| **EU design reform** | Yes: Article 19(2)(d) of Regulation 2024/2822 gives the design holder the right to stop "creating, downloading, copying and sharing or distributing to others any medium or software recording the design for the purpose of enabling a product" to be made. Member states must match it (Directive 2024/2823, Article 16(2)(d)). Commentators debate whether a CAD file "records the design"; the recital on 3D printing suggests yes. | Hogan Lovells summary |
| **US patents** | The person who prints or machines the patented object directly infringes. A file host is more likely liable for **inducing** infringement; direct liability for files needs "computer program product" claims. | BSKB summary |
| **Copyright** | A file can copy a drawing, a render or a creative sculpted form. It does not copy a function. | General principle; Meshwerks |
| **Trade secret** | Taking, using or disclosing the file without consent is misappropriation, wherever the file goes. | DTSA, EU Directive |
| **Contract** | Whatever the NDA or licence says about copies, sub-tier suppliers, retention and return. | Xometry partner terms as an example |

## Countries at a glance

Opinion on emphasis; facts as cited, others marked. A user's administrator would pick the
relevant **rule packs** [GOV], one per country, exactly as for export control.

| Country / bloc | Trade secrets | Design | Copyright and rights information | Notes for SolidCraft |
|---|---|---|---|---|
| **United States** | DTSA 2016 (federal civil action, ex parte seizure in extreme cases) plus state laws based on the Uniform Trade Secrets Act; Economic Espionage Act crimes (unverified detail) | Design patents (15 years from grant, unverified) | Copyright Act; 17 U.S.C. 1202 bars knowingly removing copyright management information (unverified scope) | Whistleblower notice in NDAs with individuals; FRE 902(14) hash authentication |
| **European Union** | Directive 2016/943, transposed nationally | Regulation 2024/2822 (from 1 May 2025, rest from 1 July 2026) and Directive 2024/2823: CAD files that record a design are covered | InfoSoc Directive 2001/29 Article 7 protects rights-management information (unverified) | Strictest file-level design right; GDPR limits recipient tracking |
| **Germany** | GeschGehG 2019 (implements the Directive); BAG 2024 demands proof of measures | EU rules | EU rules | Works-council consent likely needed for employee monitoring (BetrVG §87(1) no. 6, unverified) |
| **United Kingdom** | Trade Secrets (Enforcement, etc.) Regulations 2018 (SI 2018/597) plus the older law of breach of confidence | Registered designs, UK unregistered design right, supplementary unregistered design (unverified detail) | CDPA 1988; s51 limits copyright in design documents for functional articles (unverified) | Breach of confidence is broad and old, so markings matter |
| **China** | Anti-Unfair Competition Law, amended, in force 15 October 2025: higher fines; punitive damages kept for trade-secret cases | Design patents (15 years since 2021, unverified) | Copyright Law | Strong reason for markings and logs on shared files; the government-use doc covers export and sanctions limits |
| **Japan** | Unfair Competition Prevention Act; three requirements; METI guidelines revised March 2025 | Design Act (unverified detail) | Copyright Act | Explicit link between access logs and protection |
| **South Korea** | Unfair Competition Prevention and Trade Secret Protection Act (unverified) | Design Protection Act (unverified) | Copyright Act | Unverified; needs a pack reviewer |
| **India** | No trade-secret statute; protection through contract and equity (unverified) | Designs Act 2000 (unverified) | Copyright Act 1957 | NDA terms carry more weight than elsewhere |
| **Canada, Australia** | Mostly breach of confidence and contract; Canada added a Criminal Code trade-secret offence (unverified) | Industrial designs / designs acts | Copyright acts | Unverified; needs a pack reviewer |
| **International** | TRIPS Article 39 requires WTO members to protect undisclosed information (unverified) | Hague system for designs | Berne Convention | PCT for patents |

**Every row needs a lawyer in that country before a rule pack ships [Lawyer].**

## Accuracy

What the program can and cannot honestly promise (opinion, grounded in the cited sources):

- **Visible notices and metadata:** reliable as long as the file isn't edited; trivially stripped
  by anyone who wants to. Their value is legal (notice, reasonable steps, rights information), not
  technical.
- **Invisible fingerprints in meshes:** the standard survey (Wang, Lavoué, Denis and Baskurt,
  2008) says robust blind 3D mesh watermarking is hard. Rotation, scaling and noise can be
  survived; **remeshing, simplification and subdivision are the most damaging attacks**, and
  conversion to another representation (for example mesh to NURBS) was not addressed by any work
  it surveyed. So SolidCraft should promise "often survives casual copying", never "survives".
- **Fingerprints in vertex order, tessellation choices and G-code segmentation:** cheap,
  invisible, and harmless to the part, but fragile: a re-save in another program usually
  destroys them. Good for catching a straight copy of a file, which is the common case (opinion).
- **Sub-tolerance geometric shifts:** more robust, but they change the geometry. They must stay
  far inside the tolerance band, never touch critical or key characteristics (see the in-process
  inspection research for importance levels 0 to 5), and never be used on flight-safety,
  nuclear, medical or other safety-critical parts. Default: **off**.
- **Content hashes and signatures:** exact and strong, but they only identify an unchanged file.
- **Geometric similarity search:** finds copies after the file has been rebuilt, but also finds
  innocent look-alikes (every M8 bolt resembles every other). Results are leads, not proof.
- **Trusted timestamps:** prove that a hash existed at a time, if the timestamp authority is
  trusted. They prove nothing about who first had the idea.

## Programs

How other programs and services handle confidential designs, from public pages (October 2026).

| Product | What it does for confidentiality | Source and status |
|---|---|---|
| **Onshape** | Enterprise documents "cannot be public"; all documents created in an Enterprise are owned by it; sharing to another company only through admin-approved **Connections**; per-share permissions for copy, link, export, share, comment, delete; link sharing (view-only, optional export) can be revoked at once. The help page does not describe expiring access. | Onshape help, "Share Documents" |
| **Siemens Teamcenter** | **IP classification** on objects, ordered levels compared with a user's **IP clearance**; **IP licences** grant named users access to licensed objects; ACL rules and propagation to related objects. Digital rights management via the NextLabs add-on (commercial). | PLM Handbook blog; NextLabs pages |
| **SOLIDWORKS PDM** | Folder and workflow-state permissions; suppliers given read access to selected files is a common forum question with no single built-in answer (unverified details) | 3DSwym forum question; SOLIDWORKS PDM help (vault access) |
| **Autodesk Fusion** | Share links from the Fusion data panel; password and download options (unverified; forum threads only) | Autodesk forums |
| **Xometry (marketplace)** | Customer models and drawings are Confidential Information owned by Xometry or the customer (§5.3); disclosure only to personnel bound in writing (§5.1(b)); no reverse engineering or derivative works (§5.1(c)); **return or destroy Work IP within 18 months** of finishing (§5.3); no subcontracting without written consent, and flow-down to approved subcontractors (§2.1); $25,000 fee per prohibited contact with a customer (§2.5); trade secrets protected as long as law allows, other information five years after termination (§9.4). Customers can sign Xometry's mutual NDA or send their own. | Xometry Partner Terms; Xometry community article |
| **Protolabs Network (ex Hubs)** | Confidentiality across the whole order under the sales terms; separate NDA on request, signed digitally; manufacturing partners pass onboarding with contractual data-protection duties; encrypted transfers; files on AWS servers, production servers in Europe. No public detail on watermarking, deletion or ITAR. | Hubs IP protection page |

**What nobody offers (opinion, from the pages above):** an NDA register linked to designs; refusal
to share outside the NDA's named recipients; per-recipient fingerprints in exported files; a
"who leaked this?" detector; and evidence bundles. Marketplaces enforce confidentiality in
contracts, PLM systems enforce it in access control, and nothing traces a leak once a file has
left. That gap is where SolidCraft can lead.

## Complaints and requests

From the author's experience and common shop-floor practice (opinion, not a survey):

- **"We send a STEP for a quote and it comes back as someone else's product."** Quote requests
  go to many shops; most never win the job but keep the files.
- **"The supplier subcontracted our part without asking."** Sub-tier flow-down is in the
  contract but invisible in the file.
- **"Nobody remembers which NDA covers which drawing."** NDAs live in a filing cabinet or an
  email, designs in a CAD vault, and the link is in someone's head.
- **"When the NDA ended, we had no list of who still had our files."**
- **"We found a copy of our part for sale and couldn't prove it was ours."** No dated, hashed
  record of the original; the native file had been edited since.
- **"An engineer left and took the CAM templates."** Process know-how (strategies, feeds,
  fixtures) is often more valuable than the geometry, and is rarely marked.
- **"I didn't know that library part had a non-commercial licence."**
- **"We're a small shop and we can't afford a PLM system to protect customer data."** This is the
  novice case SolidCraft should solve with one switch.

## What users need

### 1. One switch for novices: "This design is confidential"

A single toggle on the design (and on a project folder) that applies safe defaults:

- marks the design **Confidential** (a commercial marking that sits beside the government
  markings of [HS 3], using the same data model and the same high-water mark);
- puts a "Confidential" banner in the viewport and on drawings, PDFs and renders;
- adds owner and notice metadata to every export;
- turns on the local audit log for that design;
- blocks bug reports, crash bundles and agent prompts from carrying the design, exactly as for a
  controlled design [CR 14] [HS 4];
- asks "Who is this for?" the first time the design is shared, and records the answer.

That alone gives a small shop most of the "reasonable steps" record a court looks for (opinion).

### 2. Confidentiality levels and need-to-know for professionals

```text
Confidentiality (one per design; optional per component, body, drawing sheet)
  level:        "public" | "internal" | "confidential" | "restricted" | "trade-secret"
  owner:        organisation that owns the information (may be a customer)
  ndas:         [nda id, ...]                 # NDAs this design is shared under
  need_to_know: [person or group, ...]        # optional; empty = whole organisation
  notice:       text shown on banners and in file metadata
  know_how:     ["cam-strategy", "fixture", "process", ...]   # marks non-geometric secrets
  set_by, set_at, reason
```

Behaviour (opinion):

- **Customer-owned designs** are marked with the customer as owner, so the shop's own staff see
  whose information it is.
- **Know-how gets a marking too.** CAM templates, tool libraries with proven feeds and speeds, and
  fixture designs can be marked "trade-secret" separately from part geometry. This also fits the
  tool-data sharing rule from [HS 12]: generic tool data can be shared; data derived from a
  marked customer job cannot.
- **Need-to-know** uses the organisation's identity system where present (SSO groups), otherwise
  OS user names. On the desktop it is a soft control plus a log; in a future collaboration cloud
  it becomes enforced access, like Teamcenter's IP clearance.
- **Downgrade is deliberate,** a command with a reason, logged, never a side effect [HS 3].
- **Format:** one new file in `crates/doc/src/format/pending/` (for example `confidentiality.rs`),
  per the repository rule. Older designs open as unmarked.

### 3. NDA register linked to designs

```text
Nda
  id, title
  parties:          [{ organisation, role: "discloser" | "recipient" | "both" }]
  type:             "mutual" | "one-way"
  purpose:          text, e.g. "quote and manufacture of bracket family 4xx"
  effective, term_end, confidentiality_end     # dates; confidentiality often outlives the term
  permitted_recipients: [organisation or person]
  sub_tier:         "forbidden" | "with-written-consent" | "named-only" | "allowed"
  return_or_destroy: { trigger: "on-request" | "at-end" | "days-after-job", days? }
  residuals_clause: bool                       # memory-based use allowed
  governing_law:    country/state
  whistleblower_notice: bool                   # US agreements with individuals (DTSA 1833(b))
  document:         path or hash of the signed PDF (never uploaded)
```

What SolidCraft does with it (opinion):

- **Refuse shares outside the NDA.** Exporting a "for supplier" package to an organisation not in
  `permitted_recipients` of any linked NDA stops with a plain message: "No NDA on record covers
  Bolt & Nut Ltd for this design." The owner can add an NDA or record an override with a reason.
- **Flow-down.** When a supplier asks to pass the design to a sub-tier shop, the owner records
  the consent; the sub-tier share gets its own fingerprint and appears in the ledger as a child of
  the supplier's.
- **Expiry and return-or-destroy.** Reminders before `term_end`; at the trigger, a list of every
  share under that NDA with "confirm returned or destroyed" and a place to attach the supplier's
  certificate of destruction. This matches the duty Xometry imposes on its partners (18 months),
  for any shop's own suppliers.
- **The DTSA notice flag** is a reminder only: if an NDA with a US individual lacks the notice,
  SolidCraft shows the consequence in one line and links the statute. It does not draft the
  clause [Lawyer].
- **Templates are not legal advice.** SolidCraft ships no NDA text of its own (opinion: a sample
  from a reputable public source could be linked later, after review [Lawyer]).

### 4. Share the minimum: purpose-bound packages

A share wizard (`share.prepare`) asks the purpose and recipient, then builds only what that
purpose needs:

| Purpose | Default contents | Never included |
|---|---|---|
| Quote | 2D drawing PDF with banners; a dumb STEP solid (no feature history) or a coarse mesh | Native design, sketches, CAM, tool library, costings |
| Manufacture | Released drawing; STEP; optional CAM setup sheet | Native design history, other parts of the assembly |
| Machine only | G-code with notice comments and fingerprint | Model geometry |
| Inspect | Drawing with characteristics, QIF plan | CAM, process know-how |
| Fit check | Simplified envelope mesh | Internal features |

Every package carries the layered fingerprint (section 5), is recorded in the share ledger
(recipient, purpose, NDA, contents, hashes), and the recipient's copy of the notice says what
they may do with it.

### 5. Digital fingerprints, in layers

The headline proposal. Each outgoing file for a named recipient carries several independent
layers, so that stripping one rarely removes all (opinion on design; the robustness limits are
from the cited survey):

| Layer | What it is | Survives | Lost by |
|---|---|---|---|
| **L1 Visible notice** | Banner on drawings, PDFs and renders: "Confidential. Owner: X. Prepared for Acme Ltd under NDA 2026-014 on 2026-10-29." G-code: a header comment block | Printing, screenshots, PDF re-save | Deliberate cropping or editing |
| **L2 Signed metadata** | STEP header and product property, 3MF metadata, PDF XMP, a side-car for STL/OBJ: owner, marking, NDA id, **pseudonymous recipient code**, file hash, signature (Ed25519) | Copying the file unchanged | Re-export from another program |
| **L3 Fragile structural fingerprint** | Recipient code hidden in choices that do not change the part: triangle and vertex order, tessellation seeds, the order of equivalent STEP entities, G-code arc/segment splitting and harmless last-digit feed variations within the controller's resolution | Copying, renaming, light editing in a text editor | Re-meshing, re-export, CAM re-post |
| **L4 Robust geometric fingerprint (opt-in)** | A keyed, low-frequency, sub-tolerance shift of mesh vertices or surface control points, only on faces marked non-critical, far inside their tolerance band | Rotation, scaling, noise, some simplification | Heavy remeshing, subdivision, re-modelling from scratch |
| **L5 Provenance record** | The owner's local, signed ledger entry with the hashes of every file sent, linked to the audit log, optionally timestamped by an RFC 3161 authority | Always (it stays with the owner) | Nothing, but it only matches unchanged files |
| **L6 Shape signature** | The owner keeps compact shape descriptors of each design (volume, area, moments, distribution of distances, feature graph) | Re-modelling, reformatting | Deliberate redesign |

Rules (opinion):

- **Keyed.** L3 and L4 are derived from a secret key held by the owner, so knowing the published
  algorithm does not let anyone forge or read a fingerprint (Kerckhoffs's principle: the method is
  open source; the key is not).
- **Never on safety-critical parts without a decision.** L4 is off by default; turning it on
  needs per-face tolerance data and is refused on characteristics marked critical or key, and on
  any design tagged flight-safety, nuclear, medical or pressure (opinion; ties to the in-process
  inspection research's importance levels). The change is shown as a deviation map before export.
- **Pseudonymous.** The recipient code is meaningless without the owner's ledger, so a leaked
  file does not expose who the owner's suppliers are, and GDPR data minimisation is respected.
- **Shown, not hidden.** When SolidCraft opens a file with any fingerprint layer, it says so in
  the file properties ("This file carries a recipient mark from Owner X"). It does not decode the
  recipient for anyone but the key holder.
- **One fingerprint engine.** The high-security research's section 14 (marketing an ITAR product)
  uses per-recipient fingerprints on scrubbed marketing derivatives; it should call this same
  engine rather than a second one [HS 14].
- **Patent check first.** Mesh watermarking has been an active research and patent field since
  the late 1990s. Before L4 (and the similarity search) is built, its methods need a patent check,
  preferring methods from expired patents or well-known academic work [Lawyer].

### 6. "Who leaked this?": the detector

`ip.trace { file }` reads any file and reports, in order of strength:

1. matching signatures and hashes from the owner's ledger (exact copy of a sent file);
2. decoded L3 or L4 recipient codes, with a confidence figure;
3. visible notices and metadata found;
4. if nothing survives, the closest designs by shape signature (L6), with a similarity score and
   a side-by-side overlay.

The report always says: "These are leads, not proof. Ask your lawyer before acting." It can be
added to an evidence bundle (section 8). It never contacts anyone.

**Canary variants (opinion, pro feature):** for quotes to many shops, SolidCraft can make each
shop's drawing trivially different in a way that doesn't matter (a note's wording, a non-critical
chamfer callout's position). A copy then shows which shop it came from even after re-drawing.
This "canary trap" is an old counter-intelligence technique (background knowledge, unverified).

**Marketing material** (renders, web 3D viewers, brochures) needs the same idea in reverse:
deliberate canary details and optionally a falsified scale in the marketing geometry, so a part
copied from a brochure or viewer can be recognised. That case is covered by the author's separate
design-to-brochure marketing pipeline research, which uses this file's fingerprint layers and
`ip.trace` rather than its own.

### 7. Provenance: proving it's yours, and when

- **Signed revisions.** Each saved revision gets a content hash; released revisions get a
  signature record [CR 3] in the hash-chained audit log [HS 5].
- **Trusted timestamps (opt-in, online).** The hash (never the design) is sent to an RFC 3161
  timestamp authority chosen by the user, or recorded offline for later stamping. Air-gapped and
  government-mode installs [GOV] skip this unless the policy allows a named authority.
- **Authorship record.** Who created and changed each feature (from the command log, since every
  change is a command), useful to show independent creation in a copyright or design dispute.
- **Not a C2PA replacement, but compatible.** The C2PA "Content Credentials" standard signs
  provenance for media files; the C2PA Rust crate lists images, video, audio and PDF handlers and
  no 3D formats (fact, crate docs). SolidCraft could sign its rendered images and PDFs with C2PA
  and use its own manifest for CAD files, aligned with C2PA's structure so a 3D extension could
  adopt it later (opinion). The crate's licence needs checking against `cargo xtask licences`
  before use.

### 8. Evidence bundles

`ip.evidence_bundle { design, since?, include_trace? }` builds one sealed archive for the
owner's lawyer:

- the design at each relevant revision, with hashes;
- the audit log extract and its chain verification [HS 5];
- the share ledger entries and NDA records involved;
- any `ip.trace` report;
- a manifest of SHA-256 hashes and a signature, so a qualified person can certify that copies
  match (in the US, FRE 902(14); other countries have their own rules, unverified).

SolidCraft does not certify the bundle itself; a qualified person does [Lawyer].

### 9. Not infringing others: licence and patent awareness

The other half of IP protection is helping honest users stay clear of other people's rights:

- **Licence tags on every library part.** Standard components, NSN items, tool and holder models,
  and community parts carry a licence, a source and a provenance hash (the project's
  ATTRIBUTION.md discipline, applied to user libraries). Inserting a non-commercial or
  share-alike part into a design marked commercial shows the conflict before it happens.
- **Imported-file origin.** An imported STEP or STL keeps a record of where it came from (file
  name, hash, any metadata found), so a later dispute can show what was copied and what was made.
- **Patent and design watch (opt-in, online).** Searches public patent and design databases
  (for example the USPTO, EPO and EUIPO public APIs, unverified for current terms) for keywords
  and classes the user picks, and lists new results. It sends **keywords, never geometry**.
  Off in government mode levels above Public [GOV]. It never says "free to operate".
- **Design-around notes.** A field on a design where the team records which patents and designs
  they looked at and how they differ, dated and logged. Useful evidence of good faith (opinion,
  [Lawyer] on whether such notes help or hurt in a given country; in the US, wilful-infringement
  rules make this a lawyer's call).
- **Counterfeits.** For spare-parts makers (the author's field), the same shape signatures help
  spot a counterfeit or unauthorised copy of a qualified part in a supplier's submission (opinion).

### 10. AI, MCP, plugins and bug reports

The same rules as for government markings, with the commercial level added:

- A confidential design is **announced** to every connected agent, MCP client and plugin at
  connection time, with handling instructions such as `no-training`, `no-retention`,
  `owner-only` [HS 4]. Silence from the peer means no design data is sent.
- Agent endpoints carry a trust level that includes `confidential`; a user's personal cloud AI
  account can be refused for customer-owned designs while their company's contracted endpoint is
  allowed.
- Bug and crash reports never carry a confidential design; the developer gets mock-up
  instructions written inside the boundary [CR 14].
- Screenshots and renders through MCP carry the L1 notice.

### 11. Employees and leavers

- **Joiner:** the organisation's policy can show new users which markings exist and what they
  mean (Japan's guidelines and the BAG decision both value staff being able to recognise what is
  secret).
- **Leaver:** an administrator's report lists the confidential designs and know-how a user
  opened, exported or shared in a chosen period (from the audit log), to support an exit
  interview. It is a report, not surveillance: the audit log is off by default outside a policy,
  and its use for staff monitoring may need works-council or employee consent in some countries
  (Germany, unverified) [Lawyer].

### 12. Collaboration cloud and supplier portal (future)

If SolidCraft ever hosts collaboration, the same model becomes enforceable: per-share
permissions like Onshape's, organisation ownership, admin-approved connections, expiring links,
download-off by default for "restricted", and the NDA register enforced on the server. It falls
under the prevention-first compliance gates of [GOV] (screening, region blocks, outside-lawyer
sign-off) before launch.

## What SolidCraft must never do

- Claim a design is "protected", "safe to share", "not infringing" or "free to operate".
- Hide an identifier of its own user in files, or report on its users to anyone.
- Phone home, or upload a design, hash or fingerprint without the owner's explicit, logged choice.
- Change geometry for a fingerprint on a safety-critical characteristic, or without showing the
  deviation.
- Ship NDA or licence text of its own as if it were legal advice.
- Ship a fingerprint or similarity method before its patent position is checked [Lawyer].

## Proposed commands (sketch)

Everything is a command, with input and output schemas (opinion; names follow the existing style):

| Command | Does |
|---|---|
| `confidentiality.set {level, owner?, ndas?, need_to_know?, notice?, reason}` | Set or change the marking (downgrade needs `reason`) |
| `nda.add` / `nda.update` / `nda.list` / `nda.link {design, nda}` | Manage the NDA register |
| `nda.due {within_days}` | NDAs ending, and shares awaiting return or destruction |
| `share.prepare {design, recipient, purpose, nda}` | Build the purpose-bound package with fingerprints; refuse if no NDA covers it |
| `share.ledger {design?, recipient?}` | List what was sent to whom, with hashes |
| `share.confirm_destroyed {share, certificate?}` | Record return or destruction |
| `fingerprint.inspect {file}` | Show which layers a file carries (no decoding without the key) |
| `ip.trace {file}` | "Who leaked this?" report |
| `ip.similar {file or design, scope}` | Shape-signature search in the owner's designs |
| `provenance.sign {design}` / `provenance.verify {file}` / `provenance.timestamp {design}` | Sign, verify and timestamp hashes |
| `ip.evidence_bundle {design, since?}` | Sealed archive for a lawyer |
| `library.licence_check {design}` | Licence conflicts among inserted parts |
| `ip.watch.set {keywords, classes}` / `ip.watch.results` | Opt-in patent and design watch |

None of them opens a dialog when called programmatically.

## What contributors could build first (opinion, smallest useful steps)

1. **The Confidential switch and banner** (section 1), reusing the marking model of [HS 3]. Small,
   visible, and gives novices a real "reasonable steps" record.
2. **Notice metadata on exports** (L1 and L2) for STEP, 3MF, PDF and G-code comments.
3. **The share ledger** with hashes, even before NDAs exist: "what did we send to whom?"
4. **`fingerprint.inspect`**, so SolidCraft shows any marks in files it opens. Transparency first.
5. **The NDA register** and refusal of shares outside it.
6. **L3 structural fingerprints** for STL/3MF and G-code (no geometry change, so no safety risk).
7. Later, after a patent check: L4, shape signatures and `ip.trace`.

## Export control, sanctions and government use

*Opinion and policy proposals, not legal advice. Follows the
[government use research](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-qk5riz/research/gov-research-nsnlookupjn-government-use.md).*

**Our three rules:**

- **No one goes to jail.** Our team has no in-house counsel. So wherever a rule is unclear we take
  the most conservative reading and refuse rather than warn. Questions that need an outside lawyer
  are marked [Lawyer], and free official help (BIS, DDTC, OFAC) is the first call for export
  questions.
- **Help users comply.** SolidCraft stops and explains in plain language, using rule packs for
  each jurisdiction that the administrator picks. It flags conflicts between countries' laws,
  never resolves them, and never gives legal advice.
- **Prevention first.** Nothing in this file that is hosted, shared, paid or networked ships until
  its compliance gates are in place and checked.

**What in this file crosses a border:** supplier share packages, trusted timestamps, the patent
and design watch, and a future collaboration cloud or supplier portal.

- **A confidentiality marking is not an export classification.** A design can be commercially
  confidential and also ITAR or EAR controlled. The export flag [HS 13] is checked first and
  always wins: a share package to a foreign supplier is refused for an export-controlled design
  unless the policy records an authorisation, whatever NDA exists.
- **NDAs don't make an export lawful.** An NDA with a supplier in another country does not
  replace an export licence, and the share wizard says so when recipient and owner are in
  different countries.
- **Screen recipients.** In government mode, and for any hosted sharing, recipients are screened
  against the US Consolidated Screening List and the UK, EU and UN lists before a package is
  built. A match stops the share; a person decides on false positives.
- **Sanctioned regions.** Hosted parts block comprehensively sanctioned regions, plus Russia and
  Belarus (US, EU and UK rules restrict CAD/CAM software and related services there). No
  support, consulting or custom work for users in sanctioned regions or for listed parties.
- **Only hashes leave for timestamps; only keywords leave for the patent watch.** Neither is
  allowed above the Public level of government mode without a policy entry.
- **Fingerprints don't leak markings.** Recipient codes are pseudonymous and carry no export
  classification, so a fingerprint can't disclose controlled information.
- **Code stays public.** Every build the project ships is public, which keeps it "published"
  software outside the EAR. The rules are tracked by the standing export and sanctions watch.
- **Checked for controlled data (2026-10-29):** this file contains no controlled technical data.
  Its examples are public sources, generic values or made-up stand-ins (for example "Acme Ltd",
  "NDA 2026-014"), and the author's experience is stated in general terms.

## Glossary

- **Canary trap:** giving each recipient a slightly different copy so a leak shows its source.
- **C2PA / Content Credentials:** an industry standard for signed provenance on media files.
- **Content hash:** a short fingerprint (here SHA-256) that changes if any byte of a file changes.
- **DTSA:** US Defend Trade Secrets Act of 2016.
- **Flow-down:** passing contract terms from a customer to a supplier's own sub-tier suppliers.
- **FRE 902(14):** US federal evidence rule letting hash-verified copies of data be
  self-authenticating with a qualified person's certificate.
- **NDA:** non-disclosure (confidentiality) agreement.
- **Need-to-know:** access limited to people whose work requires the information.
- **Reasonable steps / measures:** what a trade-secret owner must show it did to keep the
  information secret.
- **RFC 3161:** the internet standard for trusted timestamps.
- **Robust vs fragile watermark:** robust survives editing; fragile breaks on any change (and so
  shows tampering).
- **Shape signature:** compact numbers describing a shape, used to find similar shapes.
- **Trade secret:** valuable information kept secret by reasonable steps.

## Open questions

1. Which mesh and B-rep watermarking methods are free of live patents? [Lawyer]
2. Does the EU design reform's file right cover native CAD and STEP files, or only print-ready
   meshes? Commentators differ.
3. Should know-how markings (CAM templates, feeds and speeds) be shareable with the tool
   library's field allow-list [HS 12], or always blocked?
4. Is a per-recipient fingerprint personal data under GDPR when the recipient is a sole trader?
   [Lawyer]
5. What minimum NDA fields matter most to small shops? (Ask shop owners before building.)
6. Would maintainers accept a `confidentiality.rs` pending format file alongside the markings file
   proposed in [HS 3], or one combined file?
7. Is a design-around note helpful or harmful evidence in the US and in Germany? [Lawyer]
8. Which timestamp authorities are acceptable to courts in the main countries, and is a free one
   good enough?

## Sources

Trade secrets and NDAs:

- EU Directive 2016/943, Article 2: https://www.legislation.gov.uk/eudr/2016/943/article/2
- DTSA notice and enactment summary (Day Pitney): https://daypitney.com/article/print/4535
- DTSA notice clause note: https://terms.law/NDA/clause-library/dtsa-notice.html
- UK Trade Secrets (Enforcement, etc.) Regulations 2018: https://legislation.gov.uk/uksi/2018/597/made
- BAG 8 AZR 172/23 summary (Loschelder, February 2025): https://loschelder.de/fileadmin/redaktion/newsletter/pdf/arbeitsrecht/2025_02_Februar_1_Verschwiegenheit.pdf
- Japan UCPA trade-secret requirements and METI guidelines (secondary summary): https://timewell.jp/en/columns/unfair-competition-trade-secret-protection
- China AUCL amendment effective 15 October 2025 (Legal 500): https://www.legal500.com/intelligence/china/antitrust-competition-law/china's-anti-unfair-competition-law-amended-effective-october-15-2025

Design, copyright and patents:

- EU design reform and 3D printing (Hogan Lovells): https://ca.hoganlovells.com/en/publications/effects-of-the-eu-design-reform-on-3d-printing
- EU design reform, Regulation 2024/2822 and Directive 2024/2823 (IPKat): https://ipkitten.blogspot.com/2024/11/design-reform-reaches-its-finale-it-is.html
- Meshwerks v. Toyota, 10th Cir. 2008: https://www.ca10.uscourts.gov/sites/ca10/files/opinions/01011513284.pdf
- 3D printing and patent law (BSKB): https://bskb.com/our-firm/articles/article-detail/insights/2020/10/30/3d-printing-and-patent-law

Evidence and provenance:

- FRE 902(13) and 902(14) (Forensic Focus): https://www.forensicfocus.com/legal/new-federal-rule-of-evidence-to-directly-impact-computer-forensics-and-ediscovery-preservation-best-practices/
- C2PA Rust crate asset handlers: https://docs.rs/crate/c2pa/0.84.0/source/src/asset_handlers/README.md

Watermarking:

- Wang, Lavoué, Denis, Baskurt, "A Comprehensive Survey on Three-Dimensional Mesh Watermarking"
  (2008): https://liris.cnrs.fr/Documents/Liris-3360.pdf

Programs:

- Onshape, Share Documents: https://cad.onshape.com/help/Content/Collaboration/share_documents.htm
- Teamcenter IP classification (PLM Handbook): https://plmhandbook.blogspot.com/2020/08/ip-classification-in-teamcenter.html
- NextLabs DRM for Teamcenter: https://nextlabs.com/?p=3992
- SOLIDWORKS PDM vault access: https://help.solidworks.com/2017/English/EnterprisePDM/FileExplorer/c_Enterprise_PDM_Vault_Access.htm
- Supplier access to a PDM vault (forum question): https://3dswym.3dexperience.3ds.com/question/solidworks-user-forum/supplier-access-to-vault-with-read-permissions_9RjHuDCrRo6EA2ILWmCK7A
- Xometry Partner Terms and Conditions: https://www.xometry.com/partner-terms-and-conditions/
- Xometry NDA article: https://community.xometry.com/kb/articles/662-how-to-get-a-non-disclosure-agreement-nda-with-xometry
- Protolabs Network (Hubs) IP protection: https://www.hubs.com/fr/protection-de-la-propriete-intellectuelle/
