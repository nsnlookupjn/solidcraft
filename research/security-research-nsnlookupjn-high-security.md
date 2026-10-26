# Security research: SolidCraft in government, sensitive and high-security work

A data model and deployment model that lets SolidCraft run where defense and aerospace design
work happens: **fully local and air-gapped**, or **hosted in a government cloud (AWS GovCloud
(US), Azure Government)**, with data security that an assessor can **verify** against the US
rules for Controlled Unclassified Information (CUI), export-controlled technical data and DoD
cloud work.

This is a cross-cutting topic. It touches every other research file from the same author,
because each of them proposes data that leaves the user's computer (a shared library, a cloud
sync, an edge device at the machine):

- [NSN military/aerospace part library](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-nc3c94/research/parts-research-nsnlookupjn-nsn-library.md) (#787)
- [In-process inspection and the Raspberry Pi box](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-y2w06n/research/cam-research-nsnlookupjn-in-process-inspection.md) (#1621)
- [Shared cloud tool library](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-tool-library/research/cam-research-nsnlookupjn-tool-library.md),
  [workholding library](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-workholding/research/cam-research-nsnlookupjn-workholding.md),
  [machine library](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-machine-library/research/cam-research-nsnlookupjn-machine-library.md) (#1621)
- [Machine health](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-9w3gq9/research/cam-research-nsnlookupjn-machine-health.md)
  and [cell automation](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-g08nas/research/cam-research-nsnlookupjn-cell-automation.md) (#1621)

Licence: MIT OR Apache-2.0, the same as SolidCraft (see the Licence section of CONTRIBUTING.md).

## About this research

- **Author:** Jason ([@nsnlookupjn](https://github.com/nsnlookupjn)). Idea and direction by Jason;
  source research and writing done with an AI assistant (Claude) in October 2026.
- **How it was produced:** web searches of public sources (Federal Register summaries from law
  firms, DoD and NIST pages, cloud providers' compliance pages, crate documentation), each claim
  checked against the cited page where possible. SolidCraft's own code and docs were read to fit
  the proposal to what exists: `docs/book/src/guide/updates.md`, `docs/releasing.md`,
  `docs/crash-reports.md`, `docs/mcp.md`, `docs/agent-chat.md`, `docs/web-runtime.md`,
  `docs/plugins.md`, `SECURITY.md`, `deny.toml`, `flake.nix` and the `Cargo.toml` files. No GPL
  code was read and no commercial CAD program was driven.
- **Not legal advice, and not a compliance claim.** Software is never "CMMC certified" or
  "ITAR compliant" by itself: an organisation's *system* (people, computers, networks, policies)
  is assessed. What a program can do is (1) not undermine that system and (2) make the
  organisation's evidence easy to produce. This file is about those two things.
- **Classified work** (Confidential, Secret, Top Secret) is out of scope except to say what the
  program must not do (phone home, need a network) so that it can run on an accredited,
  air-gapped classified system under that system's own rules.
- **Marking:** "(unverified)" marks a claim not confirmed against a source. **Fact**,
  **opinion** and **marketing** are labelled where they mix. Dates are as of 2026-10-26.

> **[TODO Jason: personal experience]** Where you have seen CAD/CAM used in controlled
> environments (CUI enclaves, ITAR shops, air-gapped labs, GovCloud), what got in the way (licence
> servers that need the internet, update checks, cloud-only tools, USB bans), and what an
> assessor or security officer asked for. Label this as personal experience.

## Summary

1. **The goal.** A shop or engineer doing DoD, NASA, DOE or aerospace work can install
   SolidCraft, switch on a **high-security deployment profile** set by an administrator, and then
   *show* that it sends nothing anywhere, keeps every design and library on approved storage,
   marks controlled designs and everything exported from them, logs who did what, uses validated
   cryptography where cryptography protects CUI, and is the exact build the project published.
2. **SolidCraft already starts well (fact).** It is open source, builds from source (a Nix flake
   and a `vendor/` folder exist), keeps crash reports on the computer and never sends them, sends
   only an anonymous update check that an environment variable turns off
   (`SOLIDCRAFT_UPDATE_CHECK=0`), verifies update downloads by SHA-256 and (once the key is set)
   a minisign signature, binds its control channel to `127.0.0.1` with a per-launch token, and
   runs `cargo deny` on advisories, bans and sources in CI. The Flatpak has no network permission
   at all. Most commercial CAD/CAM needs a licence server or cloud sign-in; SolidCraft does not.
3. **What is missing (opinion).** One switch that an administrator, not the user, controls; a
   complete inventory of every network path (agent chat, update check, web app, cloud storage,
   future libraries and the Pi box); **markings in the design model** that follow parts into
   assemblies and onto every export; an append-only **audit log**; a **FIPS 140-3 build** (today
   TLS uses rustls with the `ring` provider, which is not a validated module); **SBOM and build
   provenance** attached to releases; and signed **offline library packs** that cross an air gap
   on removable media.
4. **The rules that matter (fact).** CUI work for DoD runs under DFARS 252.204-7012 (NIST SP
   800-171 Rev 2's 110 requirements; 72-hour incident reports; cloud providers at FedRAMP
   Moderate *equivalent*). CMMC 2.0 turns that into a condition of contract award: phase 1 began
   10 Nov 2025 (self-assessment) and **phase 2 begins 10 Nov 2026** (third-party Level 2
   certification), two weeks from this writing. Export-controlled technical data (ITAR/EAR) adds
   US-person and location limits, softened since 2020 by an **end-to-end encryption carve-out**.
   FIPS 140-2 certificates moved to the historical list in September 2026, so new work should
   point at **FIPS 140-3** modules.
5. **Two deployment shapes.** **Air-gapped desktop** (the baseline: everything works offline;
   libraries arrive as signed packs) and **GovCloud-hosted** (the web app and a headless engine in
   AWS GovCloud or Azure Government, identity through the agency's or company's single sign-on
   with CAC/PIV, storage encrypted with FIPS endpoints). A hybrid, desktop plus an on-premises or
   GovCloud **library mirror**, covers most shops.
6. **Other topics in secure mode.** The NSN library runs from the offline DLA PUB LOG pack, with
   online lookups off (lookups reveal programs) and contributions of controlled items blocked.
   The Pi box becomes an in-scope CUI asset: hardened image, wired-only, pull-only sync to the
   shop's own server, encrypted storage, the same hash-chained log. **Tool data can still be
   shared** on defense jobs (cutting results per tool part number), because only part, design,
   test and inspection data is controlled. A field allow-list makes sure nothing derived from the
   design (geometry, toolpaths, part numbers, job names) rides along in a tool record.
7. **A per-design ITAR switch.** Online manufacturers that take ITAR work (Xometry, Jiga,
   RapidMade) all do the same few things: the customer flags the job, controlled files go to a
   separate US-only store (Xometry and Jiga say AWS GovCloud), only vetted US persons and
   ITAR-registered US suppliers see them, and partners are bound by contract not to release them.
   SolidCraft can do the same at the source: one **Export controlled (ITAR)** switch per design
   that, once on, **sticks**. It follows the design into copies, assemblies and every export, it
   blocks the paths that would release it, and it can only be turned off by a logged, reasoned
   decision. On the desktop the switch is a guardrail plus evidence. In a future collaboration
   cloud the server enforces it, the way the marketplaces do.

## Fundamentals

### The regimes, in one table

| Regime | Who it binds | What it means for SolidCraft | Source |
|---|---|---|---|
| **FCI** (Federal Contract Information), FAR 52.204-21 | Anyone with a federal contract | 15 basic safeguards (CMMC Level 1). Ordinary hygiene. | Cooley summary of the CMMC rule |
| **CUI**, 32 CFR 2002; DoD: DoDI 5200.48 | Information the government creates or a contractor creates for it that a law or policy says to protect | Designs, drawings, models and part data made under a DoD contract are often **CTI** (Controlled Technical Information), a CUI category. Must be **marked** and protected. | CUI marking guides (Secureframe, DTIC) |
| **DFARS 252.204-7012** | DoD contractors and subcontractors handling covered defense information | NIST SP 800-171 on contractor systems; report cyber incidents within 72 hours; cloud services that hold CUI must meet **FedRAMP Moderate equivalent**. | DoD CIO memo of 21 Dec 2023 (Secureframe summary) |
| **NIST SP 800-171** | Contractor systems holding CUI | Rev 2: 110 requirements in 14 families (access control, audit, configuration, identification, media, system and communications protection...). Rev 3 (May 2024) exists, but a DoD class deviation keeps **Rev 2** as the DoD reference for now. | NSF summary of the class deviation |
| **CMMC 2.0**, 32 CFR 170 (effective Dec 2024) and DFARS 252.204-7021 (effective 10 Nov 2025) | DoD contractors | L1 = FAR 15 safeguards, self-assessed. L2 = 800-171 Rev 2, self or **C3PAO** assessed. L3 = L2 + 24 requirements of NIST SP 800-172, assessed by DIBCAC. Phases: 10 Nov 2025 self; **10 Nov 2026 C3PAO**; 10 Nov 2027 L3 and options; 10 Nov 2028 full. Annual affirmation by an "affirming official". | Cooley, Sept 2025 |
| **FedRAMP** (Low/Moderate/High) | Cloud services used by federal agencies | A SolidCraft *hosted* service for an agency would need an authorization; a contractor's cloud for CUI needs Moderate or equivalent. | FedRAMP, DoD memo |
| **DoD Impact Levels** (Cloud Computing SRG) | Cloud used by DoD | IL2 public/non-critical; IL4 CUI; IL5 higher-sensitivity CUI and national security systems; IL6 Secret. GovCloud and Azure Government hold IL2/4/5; IL6 needs the separate Secret regions. | Knox Systems comparison; AWS DoD SRG page |
| **ITAR** (22 CFR 120–130) and **EAR** (15 CFR 730–774) | Anyone handling defense articles / dual-use items | Technical data (models, drawings) for USML items may only reach US persons unless licensed. Since 25 Mar 2020, **end-to-end encrypted** data sent or stored abroad is not an export if it meets 22 CFR 120.54(a)(5) / 15 CFR 734.18(a)(5). | AFS Law alert |
| **FIPS 140-3** | Federal systems; 800-171 requirement 3.13.11 | Cryptography that protects CUI's confidentiality must be a **validated module**. FIPS 140-2 certificates went historical on 22 Sep 2026: still usable in existing systems, not for new procurements. | Traefik summary of the CMVP transition |
| **Software supply chain** (EO 14028, NIST SSDF SP 800-218, SBOM) | Software sold to government | SBOMs in SPDX or CycloneDX; CISA's 2025 minimum elements draft adds component hash, licence, tool name and generation context. | Greenberg Traurig, Oct 2025 |
| **Technical data packages**, MIL-STD-31000 (B, revision under way) | DoD acquisition | Native CAD is the "authoritative master"; STEP/IGES for exchange; viewables are information only. Markings come from DoDI 5230.24 (distribution statements). | NIST MBE summit slides, 2024; DoDI 5230.24 |

### Markings a CAD model actually carries

A controlled model or drawing commonly needs all of these at once (fact, from the DoD CUI and
distribution-statement guidance; exact wording comes from the contract and the CUI Registry):

- **CUI banner** at top and bottom of every page or view: `CUI`, optionally followed by
  category and dissemination controls (examples given in guides: `CUI//EXPORT CONTROLLED//NOFORN`).
- **Designation indicator block** on the first page: *Controlled by* (organisation and office),
  *CUI category* (for example CTI, export controlled), *dissemination / distribution control*,
  *point of contact*.
- **Distribution statement** A to F (DoDI 5230.24), with reason and date and controlling office.
  Statement A is "approved for public release"; B to F narrow who may receive it.
- **Export-control warning** (ITAR or EAR) and, where known, the USML category or ECCN.
- **Portion marks** such as `(CUI)` and `(U)` are optional for CUI unless the contract asks.
- **Electronic files:** "CUI" in headers and footers, and many organisations also put it in file
  and folder names; screenshots carry a corner mark.

Legacy markings (FOUO, "Proprietary", old ITAR stamps) are still common on inherited drawings and
are treated as CUI until reviewed (DoDI 5200.48, via the Secureframe guide).

### What 800-171 asks of an application (not the whole system)

Most of the 110 requirements are about the organisation (training, physical security, personnel
screening). The ones an application can help with or hurt (opinion, grouped by family):

| Family (Rev 2 numbering) | What the app must do or not do |
|---|---|
| 3.1 Access control | Respect OS identity and file permissions; in a hosted deployment, enforce roles and attributes (US person, need-to-know); limit CUI flow (3.1.3) so a marked design cannot be sent to an unapproved destination. |
| 3.3 Audit and accountability | Produce events for who opened, saved, exported, printed, changed markings; protect the log from tampering (3.3.8); timestamps from the system clock (3.3.7). |
| 3.4 Configuration management | Let an administrator lock settings (3.4.2); disable non-essential functions such as update checks and cloud features (3.4.6, "least functionality"); control user-installed software and plugins (3.4.9). |
| 3.5 Identification and authentication | Desktop: use the OS login (which can be CAC/PIV). Hosted: single sign-on with multifactor (3.5.3). |
| 3.8 Media protection | Mark exported media (3.8.4); support encrypted removable media for packs and transfers (3.8.6). |
| 3.13 System and communications protection | No unexpected egress (3.13.1 boundary protection, 3.13.6 deny by default); FIPS-validated crypto for CUI (3.13.11); TLS for anything in transit (3.13.8); protect CUI at rest (3.13.16). |
| 3.14 System and information integrity | Fix flaws quickly (3.14.1: the "latest release only" policy in `SECURITY.md` conflicts with an air gap; see "Open questions"); treat imported files as hostile (already a SolidCraft rule). |

## Accuracy

What can be relied on, and what cannot:

- **Dates of rules** (CMMC phases, FIPS 140-2 sunset, ITAR carve-out) were checked against law
  firm and vendor summaries of the Federal Register, not the Register itself. Phase dates can
  move; a search result title mentions a newer DFARS class deviation "2026-O0025, Revision 3"
  and a "Phase 2 pause" (unverified; the page did not load). Maintainers should re-check before
  quoting dates.
- **800-171 Rev 2 vs Rev 3:** DoD still points at Rev 2 through a class deviation (May 2024);
  Rev 3 is expected later. A design that maps to both is safest.
- **FIPS claims are narrow.** "Uses a FIPS 140-3 validated module" is checkable (certificate
  number, operational environment in the security policy). "FIPS compliant" or "FIPS ready" is
  marketing and assessors know it. rustls's FIPS mode is covered by AWS-LC certificate #4816 for
  the environments in that certificate's security policy only.
- **Cloud authorizations are per service and per region.** "GovCloud is FedRAMP High" does not
  mean every service in it is; each service has its own scope (fact, AWS's own service-in-scope
  lists). The same applies to AI models: Claude 3.5 Sonnet v1 and Claude 3 Haiku on Amazon
  Bedrock in GovCloud were approved for FedRAMP High and IL4/5 in May 2025 (AWS announcement);
  Anthropic's public-sector FAQ lists Bedrock as IL5 accredited and the route for ITAR data, and
  a Secret-region (IL6) offering. Newer models' status must be checked at the time.
- **Software alone is never "certified".** Any line in SolidCraft's docs that says otherwise
  would be wrong (opinion, but universal among assessors).

## Programs

How existing tools handle this (from their public pages; no product was run):

| Kind | Examples | How they handle high-security work (public claims) |
|---|---|---|
| Cloud-native CAD | Onshape, Fusion | Separate government offerings or regions are advertised by some vendors (marketing; details unverified). Ordinary tenants are commercial cloud, so CUI use depends on the tenant's FedRAMP status. |
| Desktop CAD with licence servers | Most traditional CAD/CAM | Run air-gapped with an offline or on-premises licence server; update checks and telemetry are switched off by IT; PDM vaults are hosted on-premises or in GovCloud (general knowledge; unverified per product). |
| Secure file sharing for CUI/ITAR | PreVeil, Virtru, Kiteworks, Tresorit | Market **end-to-end encryption** with FIPS-validated modules to use the ITAR 120.54 carve-out (marketing; they are the authors of several of the guides cited). |
| Collaboration clouds | Microsoft GCC High, AWS GovCloud, Azure Government | US-person-only administration, US-only data residency, FedRAMP High, IL4/5. GCC High is the Microsoft 365 tier usually needed for CUI-Specified (Knox Systems summary). |
| Online manufacturing marketplaces | Xometry, Jiga | Customer flags ITAR/EAR/DFARS at quote or by enabling the account; controlled files stored in **AWS GovCloud**; access limited to US persons and ITAR-registered US partners; contract flow-down to partners. Details in "What users need", section 13. |
| ITAR-registered shops | RapidMade (Portland, Oregon) | ITAR registration and "secure U.S.-based operations and strict controls for handling export-controlled technical data"; no public detail on how files are submitted or stored. |
| Open source CAD | FreeCAD, others | Run offline by nature; no published security profile found. (FreeCAD is GPL; only its public website was considered, no code.) |

**Gap (opinion):** no CAD program found publishes an **egress inventory**, a **machine-readable
security report**, an **SBOM with build provenance**, *and* **model-level markings that follow
parts into exports**. An open-source program can publish all four because nothing is hidden.

## Complaints and requests

Common themes in public forums and vendor material (unverified as individual quotes; opinion):

- "The CAD needs internet to start" (cloud licensing) is the first blocker in an air gap.
- Update checkers, crash uploaders and "usage statistics" are found by the network team, then
  blocked by firewall, then the software misbehaves; administrators want one documented switch.
- Markings live in the title block of a drawing but not in the 3D model, the STEP export, the
  STL sent to a printer or the screenshot pasted in an email; that is how CUI leaks.
- Moving data into an air gap means scanning USB media; libraries that update daily are painful.
- AI assistants are attractive and banned at the same time, because nobody can say where the
  prompt and model go.
- Small shops (the majority of the defense industrial base) cannot afford a PDM server and a
  consultant to get to CMMC Level 2.

## What users need

### 1. A deployment profile the administrator controls

**Policy file.** A read-only, machine-wide file that the app reads at start, before anything else,
and that the user cannot override from Preferences:

| Platform | Suggested location (opinion) |
|---|---|
| Linux | `/etc/solidcraft/policy.toml` |
| Windows | `%ProgramData%\SolidCraft\policy.toml` (also a Group Policy registry key, later) |
| macOS | `/Library/Application Support/SolidCraft/policy.toml` (also a configuration profile, later) |

Example:

```toml
profile = "high-security"          # or "standard" (default: no file means today's behaviour)
network.egress = "deny"            # deny all; only `network.allow` hosts may be reached
network.allow = ["mirror.shop.local", "bedrock-runtime-fips.us-gov-west-1.amazonaws.com"]
updates.check = false
crypto.require_fips = true         # refuse to start TLS without the FIPS provider
agents.allowed = ["ollama-local", "bedrock-govcloud"]
mcp.enabled = true                 # localhost control channel; every call audited
plugins.allowed = []               # no plugins unless listed by hash
marking.required = true            # every new design must be given a marking before save
marking.default = "CUI"            # the organisation's default
audit.enabled = true
audit.sink = ["file", "syslog"]
libraries.sources = ["pack:/opt/solidcraft/packs", "https://mirror.shop.local/packs"]
libraries.contribute = false
crash_reports.draft = false        # no "Report..." GitHub draft
web_storage = false                # (web app) refuse browser-local storage of designs
```

- **No file, no change.** Today's defaults stay as they are; this is an opt-in profile, so it
  does not trigger the "changes a default" rule.
- **Visible.** Preferences shows locked items with a lock and "Set by your organisation".
- **A command.** `policy.inspect` (with input and output schemas, per `skills/solidcraft-commands`)
  returns the effective policy and its file path and hash, so MCP clients and auditors see the same
  thing as the UI.

### 2. An egress inventory, enforced in CI

Every place the code can open a network connection, listed in one file, with what it sends and
which policy key turns it off. From reading the current code and docs (fact as of this commit):

| Path | Where | Sends | Off by |
|---|---|---|---|
| Update check | `crates/ui-egui/src/update.rs`, `apps/solidcraft` (ureq + rustls) | GET to api.github.com, then release files | Preference, `SOLIDCRAFT_UPDATE_CHECK=0`, `updates.check` |
| Control channel / MCP bridge | `apps/solidcraft/src/control_server.rs`, `crates/mcp` | localhost only, token per launch | `mcp.enabled` |
| Agent chat | `docs/agent-chat.md` | **the design and prompts** to the chosen agent or endpoint | `agents.allowed` |
| Crash report draft | `docs/crash-reports.md` | opens a GitHub issue page in the browser (no design; screenshot only if the user pastes it) | `crash_reports.draft` |
| Web app | `apps/solidcraft-web` service worker | loads the app; designs stay in the browser unless cloud Open/Save is used | hosting choice |
| Cloud Open/Save | `docs/web-runtime.md` (craft-cloud / CollabCraft) | design files to the cloud | `network.allow` |
| Future: library sync, NSN lookups, tool/workholding/machine libraries, Pi box sync | proposals linked above | catalogue requests, possibly shop data | `libraries.sources`, `libraries.contribute` |

**Check:** a `cargo xtask egress` (opinion; same shape as `cargo xtask layers`) that fails CI when
a crate that is not on the list depends on `ureq`, `reqwest`, `std::net` connect calls or
`web_sys::fetch`. The list then doubles as the documentation an assessor reads.

### 3. Markings in the design model

The heart of "data model for sensitive work": markings are **data in the design**, not text on a
drawing.

```text
Marking (one per design; optional one per component, body, drawing sheet)
  classification:  "U" | "CUI" | (classified levels: stored, shown, never enforced)
  categories:      ["CTI", "EXPORT CONTROLLED", ...]       # from the CUI Registry
  dissemination:   ["NOFORN", "FEDCON", ...]
  distribution:    { statement: "A".."F", reason, date, controlling_office }
  export:          { regime: "ITAR" | "EAR" | "none", usml_or_eccn?: string }
  designation:     { controlled_by, poc }
  source:          "user" | "inherited:<component id>" | "library:<pack id>"
  set_by, set_at                                            # who and when (for the audit log)
```

Behaviour (opinion):

- **High-water mark.** A design's effective marking is the strictest of its own and its
  components' markings. Inserting a CTI part into an unmarked design asks once: "This part is
  marked CUI//CTI. The design will be marked CUI. Continue?" The answer is in the audit log.
- **Downgrade is deliberate.** Lowering a marking (removing CUI, changing distribution to A) is
  its own command (`marking.set` with `reason`), never a side effect of deleting a component, and
  is logged. Many organisations require a second person for decontrol; that belongs to their
  process, but the log entry makes it visible.
- **Markings travel with every export:**
  - viewport: top and bottom banner bars in the marking's colour (convention: green for
    unclassified, purple for CUI; the colours come from theme tokens so High Contrast and the
    colour-vision modes still work; see the aging-eyes research);
  - drawings and PDFs: banners on every sheet, the designation indicator and distribution
    statement in a title-block field;
  - STEP: marking in the file header and as a product property (AP242 has property mechanisms
    for this; exact entity choice is an open question);
  - 3MF, STL, OBJ: 3MF metadata; for STL/OBJ (no metadata) a side-car `.marking.json` and a
    file-name suffix if the policy asks;
  - screenshots and renders (UI and MCP `screenshot`): a corner mark burned into the image;
  - crash reports and agent prompts: refused or redacted for marked designs (below).
- **Older designs** open as "unmarked" with no change; per the repository's format rule, adding
  markings is ONE new file in `crates/doc/src/format/pending/` (for example `markings.rs`).
- **Libraries carry markings.** A pack entry can be marked; NSN items get theirs from FLIS DEMIL
  and from the organisation's own pack, never from a guess.

### 4. Flow control: CUI does not leave by accident

- **Agent chat.** Each agent endpoint gets a **trust level** in policy (for example
  `public`, `cui`, `itar`). A design marked CUI can only be sent to endpoints at `cui` or above;
  ITAR-marked only to `itar`. Air gap: a local model (Ollama, LM Studio on the same computer).
  Cloud: an endpoint inside the boundary, for example Claude on Amazon Bedrock in AWS GovCloud
  through its FIPS endpoint (FedRAMP High and IL4/5 for the models AWS lists; check the model).
  The panel shows the endpoint's trust level next to its name.
- **MCP / control channel.** Already local-only with a token (good). In the profile: every call
  is audited; `screenshot` and `export` honour the marking; `--allow-path` folders must be inside
  approved storage.
- **Plugins.** Only allowed by hash in policy; plugins in `docs/plugins.md` already have a
  provenance field (manifest hash, build id) that fits.
- **Clipboard and drag-out** of geometry from a marked design carry the marking in the clipboard
  payload where the format allows (opinion; low priority).

### 5. Audit log

- **Append-only, hash-chained JSON lines** (each line has the SHA-256 of the previous one), the
  same structure proposed for the Pi box part record, so one verifier checks both.
- **Events:** start/stop with policy hash; open, save, save-as, close; export, print, screenshot;
  marking set or changed; insert of a marked component; agent send (endpoint, size, not content);
  MCP connect and each command name; library pack import with signature result; policy load.
- **Who:** OS user (and on a hosted service the SSO subject). **When:** system clock, UTC.
- **Where:** a local file under the data dir, protected by OS permissions; optional forward to
  syslog / journald / Windows Event Log, so the organisation's SIEM collects it.
- **Commands:** `audit.list {since?, kind?}`, `audit.verify` (recomputes the chain and reports the
  first broken line). Because everything in SolidCraft is a command, the log falls out of the
  command dispatcher in one place rather than being sprinkled through the UI.
- **Off by default** outside the profile, for ordinary users' privacy.

### 6. Encryption

- **At rest: rely on the operating system first.** BitLocker, FileVault, LUKS with validated
  modules are what assessors already accept for 3.13.16. SolidCraft's job is not to write
  temporary copies outside approved storage: recovery files, thumbnails, render caches and crash
  folders should all live under one configurable data root (`SOLIDCRAFT_CRASH_DIR` already
  exists; extend the idea to every cache).
- **Optional file-level encryption** (opinion, later): an encrypted design container for sending
  over untrusted channels, built on AES-256-GCM from a validated module, keys from the OS key
  store or a smart card. This is exactly what the ITAR/EAR end-to-end carve-out rewards: data
  that stays encrypted from sender to recipient with no third party holding keys is not an
  export when stored or passing abroad (22 CFR 120.54(a)(5); not to §126.1 countries or Russia).
- **In transit:** today TLS is rustls with the `ring` provider. A **FIPS build** swaps in the
  `aws-lc-rs` provider with rustls's `fips` feature (`rustls::crypto::default_fips_provider()`;
  AWS-LC FIPS 3.0, certificate #4816, FIPS 140-3) and refuses to connect when
  `ClientConfig::fips()` is false. Trade-offs: aws-lc is C and needs a C toolchain and CMake; it
  does not build for wasm, so the FIPS build is desktop-only; the browser's own TLS covers the
  web app. Update signatures (minisign, Ed25519) and integrity hashes (`sha2`) protect integrity,
  not CUI confidentiality, so they are not what 3.13.11 is about; still, a FIPS build could use
  AWS-LC's SHA-256 too, so the security report can say "all cryptography from module #4816".

### 7. Verifiable integrity: proving it is the published build

| Piece | State in SolidCraft today (fact) | Proposal (opinion) |
|---|---|---|
| Checksums | `SHA256SUMS.txt` on every release | keep |
| Signatures | minisign signing of the checksum list designed; app enforces once `PUBLIC_KEY` is set (`docs/releasing.md`); Windows SignPath and macOS Developer ID being set up | finish; publish the public key in the repo, README and release notes |
| Dependency policy | `cargo deny` checks advisories, bans, sources in CI | add a licence and source allow-list report to each release |
| SBOM | none attached | CycloneDX or SPDX per package (for example `cargo cyclonedx`), with component hashes and licences to meet CISA's 2025 draft minimum elements; also embed the dependency list in the binary (`cargo auditable`) so a scanner can read it from the installed file |
| Provenance | none | GitHub artifact attestations (SLSA build provenance) for each release file |
| Reproducible builds | Nix flake exists; `vendor/` folder | publish a recipe that rebuilds a release bit-for-bit (path remapping, fixed timestamps), and have CI check that two builds match |
| Offline build | `vendor/` | document building from a source tarball with vendored crates and no network, for organisations that must build inside the boundary |
| Security report | none | `security.report` command and Help ▸ About ▸ Security: version, commit, build provenance, SBOM hash, FIPS provider and certificate (or "not FIPS"), policy file and hash, egress list in force, audit status. Export as JSON or PDF as evidence for a System Security Plan. |

**Patching in an air gap.** `SECURITY.md` supports only the latest release. Air-gapped sites
update on a schedule, sometimes quarterly. A small step: publish security advisories with the
affected version range and the fixed version, so a site can judge whether its build is affected
without upgrading (opinion).

### 8. Air-gapped operation

The baseline every other mode builds on: **nothing requires a network.**

- **Install** from a package with verifiable signature and SBOM, carried in on scanned media.
- **Libraries as packs.** Every shared library (appearances and physical materials already have a
  local catalogue format, `docs/book/src/dev/local-libraries.md`; tools, workholding, machines,
  NSN items proposed) is exported as a **pack**: a single file holding entries, a manifest with
  version, date, source, licence, marking and per-entry hashes, and a detached signature by the
  publisher's key. Import (`library.import_pack`) checks the signature and hashes before reading
  any entry, treats contents as hostile input (size and count caps, as for imports), and logs the
  result. Packs are deltas or full snapshots; a monthly full snapshot keeps the process simple.
- **Organisation keys.** A site can re-sign packs it has reviewed with its own key and set policy
  to trust only that key: the "review then admit" step most security officers want.
- **Designs reference library entries by ID and copy what they use** (the existing library
  contract), so designs open without the pack present.

### 9. GovCloud-hosted operation

For organisations that want browser access or central storage inside a FedRAMP High / IL4-5
boundary (opinion on architecture; service availability per region must be checked):

```text
User (CAC/PIV) --SSO (OIDC/SAML, MFA)--> web app (static wasm)  ---\
                                                                     \-- API --> storage
Desktop app ---- TLS (FIPS) ----> library mirror (packs)                        (S3 / Blob,
                                   headless engine (solidcraft-cli)              SSE with KMS,
                                   audit collector -> SIEM                       FIPS endpoints)
Everything inside one GovCloud / Azure Government account boundary; US-person administration.
```

- **Web app** served from inside the boundary (not from a public CDN unless the CDN is in the
  authorization; whether CloudFront can be used for GovCloud workloads is unverified).
- **Identity:** the customer's identity provider (Entra ID in Azure Government, IAM Identity
  Center or a SAML IdP for AWS) with CAC/PIV; SolidCraft never stores passwords.
- **Attributes for ITAR:** the IdP asserts "US person" and program membership; the storage layer
  enforces access to export-controlled designs by attribute (ABAC), not the client.
- **Storage:** design files and packs in the provider's object store with server-side encryption
  under customer-managed keys; versioning on (doubles as backup and history).
- **The current CollabCraft deployment** described in `docs/web-runtime.md`
  (`craft.quasarke.net`) is commercial hosting and must not hold CUI; the cloud tier has to be
  **self-hostable** for this to work.
- **AI in the boundary:** Bedrock in GovCloud (Claude models listed for FedRAMP High and IL4/5) or
  Azure OpenAI in Azure Government (requires onboarding) as the only agent endpoints.
- **Incident reporting:** a hosted operator under DFARS 7012 must report within 72 hours and keep
  images for 90 days (fact, DFARS 7012 text as summarised by the sources); logs above make that
  possible.

### 10. The Raspberry Pi box in a controlled shop

From the [inspection research](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-y2w06n/research/cam-research-nsnlookupjn-in-process-inspection.md):
the box reads probe results from the control, keeps a hash-chained per-part record, prints
traveller labels and syncs reports. In a CUI shop it is **in scope** (it stores serials and
measurements of CTI parts). What changes (opinion):

- **Wired only.** No Wi-Fi or Bluetooth (many facilities ban them; disable in firmware and
  remove from the image).
- **Network:** on the machine (OT) network, segmented from the office network per NIST SP 800-82
  practice; no inbound ports; it **pushes to** the shop's own server only (on-premises or
  GovCloud), never to a vendor cloud. Its sync client uses the same pack and signature format.
- **Hardened image:** read-only root, signed updates, no default accounts, SSH off or key-only,
  USB limited to the barcode scanner by device ID.
- **Storage encrypted** (LUKS). The Pi has no TPM, so the key needs a home: entered at boot,
  fetched from the server at boot, or a TPM add-on board (unverified which is practical); Pi 5 /
  CM5 secure boot support is reported but not checked here.
- **Crypto:** the OS's validated module if the distribution offers one for arm64 (for example a
  FIPS-validated OpenSSL provider in a commercial Linux subscription; unverified for Raspberry Pi
  OS). If none, an industrial PC with a TPM and a validated OS is the "high-security tier" of the
  box.
- **Labels:** serial numbers and DataMatrix codes are normally not CUI, but the traveller report
  may be; it prints with the job's marking.
- **Same audit chain** as the desktop; one `audit.verify` checks both.

### 11. The NSN library in a controlled environment

From the [NSN library research](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-nc3c94/research/parts-research-nsnlookupjn-nsn-library.md):

- **Offline PUB LOG pack is the baseline** (it already was): DLA publishes it as public data,
  so the pack itself is not CUI and can cross into an air gap.
- **Online lookups are off** in the profile. Which NSNs a user looks up can reveal which system
  they are working on (the earlier research already flagged this); even an anonymous query
  stream is aggregated program data.
- **FED LOG** (DLA's fuller offline catalogue for .mil/.gov and CAC holders) could be a second
  local source on government machines; whether its licence allows conversion into a pack is an
  open question.
- **Contributions blocked** for marked designs and for items whose DEMIL code is not "A". An
  organisation can run its own **private contribution pack**: models made in-house for NSNs,
  reviewed and signed by the site, never leaving it.
- **Marking from data:** a contributed model inherits the strictest of the design's marking and
  the item's DEMIL/export screen.

### 12. Tool, workholding and machine libraries in secure mode: what may be shared

**The rule (Jason's experience in defense work, labelled as such):** on defense jobs, **tool
data can be shared**. What a tool did (cutting results per tool part number) isn't controlled.
Only **part, design, test and inspection data** can't be shared. That matches how the rules are
written (opinion, not legal advice). ITAR controls *technical data* "required for the design,
development, production... of defense articles". A vendor's catalogue end mill, and how it
behaves in 6061 at a given speed and feed, is general machining knowledge, not a description of
the article. The risk is not the tool data. It's **design-derived data hiding inside a tool
record**. Some contracts are stricter than the regulations, so the policy can still turn sharing
off for a program.

So the profile doesn't block tool-library sharing. It **filters** it, field by field, with an
**allow-list** (only listed fields leave; anything new is blocked until it's reviewed):

| Leaves the shop (allowed) | Never leaves (blocked) |
|---|---|
| Tool vendor, catalogue/part number, grade, coating, geometry from the catalogue | Part geometry, stock model, fixture model, sketches, any mesh or B-rep |
| Holder and stick-out (length), runout measured at setup | Toolpaths, G-code, NC program names, CAM operation names |
| Workpiece material **by standard designation** (e.g. 6061-T6, Ti-6Al-4V) and hardness | Part number, NSN, drawing number, revision, CAGE of the part, customer, program or contract name |
| Operation **class** (slotting, adaptive, finishing wall, drilling) and engagement: ae, ap, chip thickness, speed, feed | Feature dimensions that describe the part (pocket depth, wall height, hole positions), tolerance values |
| Result: tool life (minutes or metres cut), wear mode, failure mode, spindle-load %, chatter yes/no | Inspection results, probe data, test data, serial numbers, part photos or screenshots |
| Machine **model** and spindle taper | Machine serial, shop location, operator, exact timestamp (coarsened to the month) |
| Coolant type | Free-text notes (they're where part names leak). They can only be shared as a fixed vocabulary, never as free text. |

How it works (opinion):

- **One export path.** Every shared tool record passes through one command,
  `tool_data.prepare_share {records}`. It builds the outgoing record from the allow-listed
  fields only. It never copies the internal record and then deletes fields, so a field added
  later can't leak by default.
- **Provenance is cut.** The outgoing record has no link back to the design or job. The local
  record keeps that link, so the self-tuning loop in the cutting-physics research still works
  at home.
- **Preview and log.** The user, or a policy-set reviewer, sees exactly what will leave. The
  audit log keeps a hash of each record that left, with the job it came from, kept **locally
  only**, so an assessor can confirm nothing else went.
- **The ITAR switch doesn't block tool data.** A tool record from an ITAR design is shared
  through the same filter, plus a check: a value that looks like a part identifier (an NSN
  pattern, the design's part number, drawing number or file name) anywhere in an allowed field
  is refused.
- **Small-population leaks.** Odd combinations can still hint at a program: a rare alloy,
  a special form tool, a burst of records in one week. Coarsening timestamps to the month, and
  publishing community aggregates only once a minimum number of shops have contributed
  (k-anonymity, k of 5 or more), reduces that. A **custom or special tool** (made to the part's
  shape, such as a form tool ground to a profile) is design-derived. Its profile is not shared;
  only its class is.
- **Policy keys:** `libraries.share_tool_data = true | false` (default `true` in the profile,
  following Jason's experience) and `libraries.share_review = "none" | "user" | "reviewer"`.
- **Pull is always fine:** vendor catalogues, fixture specs and machine envelopes come in from
  the organisation's mirror or signed packs.
- **Workholding and machine-health data** follow the same pattern. Vise model, clamping force
  and slip results can be shared. Fixture geometry made for the part can't. Machine-health
  trends are shared by machine model only, never with the machine serial or the job.

### 13. A per-design ITAR switch that locks in restrictions

Jason's idea: a switch on each design that, once set, **locks in** ITAR restrictions, so the
restrictions don't depend on everyone remembering them. It matters most if SolidCraft ever has a
collaboration cloud, because a cloud can share a design with a click.

#### How three manufacturers handle ITAR designs (public pages, October 2026)

**Xometry** (online manufacturing marketplace; the customer uploads CAD and the job is routed to
independent partner shops):

- **Flagging:** buyers mark ITAR, DFARS or EAR projects with **Add Certifications** in the
  instant-quote site. The site also asks about this at the start of the quote and again at
  checkout (Xometry help article 666). Their drawing guidance asks for explicit export wording on
  drawings, because unclear drawings delay restricted jobs (help article 853, preview only).
- **Storage:** "ITAR-controlled data is stored on AWS GovCloud". All customer data is "encrypted
  in transit and at rest". Access "is strictly controlled and logged". Employees use MFA, and
  customers can turn MFA on (Xometry data-protection page).
- **Company status:** ITAR registered (registration letter on the legal page, expiring
  30 June 2027). CMMC Level 2 certified by a C3PAO, with a public announcement in March 2025 and
  a certificate expiring 13 Feb 2028. Also on the JCP (DD 2345) list. AS9100D.
- **Routing:** when an order needs a certification, Xometry says it routes the work "only to
  verified shops with that active certification" (certifications page). Partners apply with
  their certifications, and ITAR registration opens more job types (supplier page). Whether ITAR
  jobs are hidden from non-registered or foreign partners is not stated publicly (unverified).
- **Partner contract** (Partner Terms §2.4): data may be "accessible only by persons authorized
  under the U.S. Trade Laws", possibly limited to "U.S. citizens, lawful U.S. permanent
  residents". Partners must not release it to foreign nationals without written authorization,
  must keep procedures to prevent release, must report suspected violations, and must flow these
  terms down to anyone they use. Finishing may be subcontracted only on orders not subject to
  trade laws (§2.1). Files must be returned or destroyed within 18 months of the order (§5.3).
  The terms say nothing about CUI, DFARS 7012 or NIST 800-171.

**Jiga** (a similar marketplace, included for contrast):

- **ITAR is set per account, not per part.** An account must be enabled by Jiga support first.
  Uploading an export-controlled file to an account that isn't enabled shows a warning.
- On an enabled account, only **ITAR-registered, US-based** suppliers are shown. Jiga says it
  checks that they don't use foreign nationals or unauthorized subcontractors.
- Files are kept in **AWS GovCloud**. Only the US persons the customer designates can open them,
  with two-factor authentication. **Email is not an approved channel** for ITAR files.

**RapidMade** (an ITAR-registered contract manufacturer in Tualatin, near Portland, Oregon:
3D printing (MJF, SLA, FDM, metal), thermoforming, CNC and engineering):

- Says it is "an ITAR-registered manufacturing partner" with "secure U.S.-based operations and
  strict controls for handling export-controlled technical data". All work is done at its own
  facility. ISO 9001:2015.
- Publishes **no detail** on how controlled files are submitted, stored or deleted, and nothing
  on CMMC or NIST 800-171 (rapidmade.com home and ITAR pages). That is typical of single-site
  shops: their control is "it never leaves our building and our people are US persons".
  SolidCraft's markings and audit log would help exactly this kind of shop produce evidence.

**What they have in common (opinion):** (1) a person declares the job controlled, because the
software can't classify it; (2) once declared, the files move into a **separate, US-only store**;
(3) **who** can see the files is limited by verified attributes (US person, ITAR-registered
company); (4) **contracts flow down** to every party that touches the data; (5) access is
**logged**; (6) there's a **retention** limit. None of them describes what happens to markings
inside the CAD file itself. Their control starts at upload. SolidCraft can start it at the first
sketch.

#### The switch on the desktop

- **One control:** Design Properties ▸ **Export controlled (ITAR)**, or the command
  `export_control.set {regime: "ITAR", usml_category?, note?}`. It sets the design's marking
  (section 3) to `export: {regime: "ITAR"}` plus the organisation's chosen CUI category and
  banner text. EAR works the same way with an ECCN. SolidCraft never decides whether something is
  ITAR. The person, or their export compliance officer, decides. SolidCraft only records the
  decision and enforces it.
- **Suggested, never automatic:** SolidCraft offers to turn the switch on when it sees a clue:
  an imported STEP/3MF/PDF whose metadata or notes contain ITAR or "22 CFR" wording, an inserted
  component that is already ITAR, an NSN part whose DEMIL code is not "A", or a library pack entry
  marked ITAR. The answer is logged.
- **Sticky ("locked in"):**
  - **Turning it off** is a separate command, `export_control.clear {reason}`. It is never a side
    effect of anything else. When the policy names an export compliance role, clearing needs a
    second person's signed approval (a small approval file signed with their key or smart card,
    imported on the desktop). Otherwise it needs a typed reason. Either way it is in the audit log.
  - **Inheritance:** Save As, copy and paste of geometry, linked-design paste
    (`docs/linked-design-paste.md`), configurations, derived parts, and inserting the design as a
    component into another design all carry the flag. The receiving design is raised to ITAR
    (high-water mark) after a one-time prompt.
  - **Every export says it:** an export-control warning statement on drawings and PDFs, a
    `.marking.json` side-car or 3MF/STEP metadata with the flag, banners on screenshots and
    renders, and optionally an `-ITAR` file-name suffix.
- **What it blocks or guards** while on:
  - agent chat, except endpoints the policy marks `itar` (a local model, or Bedrock in GovCloud,
    which Anthropic's FAQ lists as the route for ITAR data);
  - crash-report drafts, contributions of the design or anything made from it to public
    libraries (NSN models, custom tools, fixtures), and any
    cloud Open/Save to storage not marked `itar` in policy;
  - "send to manufacturer" or slicer hand-off (#2595) integrations, unless the destination is
    marked ITAR-capable. If SolidCraft ever links to a service such as Xometry or Jiga, it should
    pass the flag so the service's own ITAR setting is turned on (Xometry's Add Certifications,
    a Jiga ITAR-enabled account), rather than relying on the user to tick it again.
- **Optional end-to-end encryption:** with ITAR on, Save can write an encrypted container whose
  keys are held only by the named US persons. That fits the 22 CFR 120.54(a)(5) carve-out
  (section 6), so a laptop crossing a border, or a backup in a commercial cloud, holds only
  ciphertext.
- **Honest limits:** a desktop app can't stop a determined user who edits the JSON or takes a
  photo of the screen. On the desktop the switch is a **guardrail and evidence**: it prevents
  accidents and shows an assessor what happened. Hard enforcement needs a server.

#### The switch in a future collaboration cloud

`docs/web-runtime.md` already plans cloud Open/Save through craft-cloud and CollabCraft. If that
becomes a place where people share designs, the ITAR flag becomes **server-enforced**, following
what Xometry and Jiga do:

| Rule | How (opinion) | Precedent |
|---|---|---|
| Flag lives on the server | The server stores the flag and the marking. The client's claim is never trusted. Changing it is an API call with role checks and an audit record. | Xometry logs access; Jiga sets ITAR per account |
| Separate US-only partition | ITAR designs, their thumbnails, previews, versions, comments and search index live only in a GovCloud or Azure Government partition with US-person administration. A workspace must be **ITAR-enabled** before it can hold them. Uploading a flagged design to a workspace that isn't enabled is **refused**, not just warned, because SolidCraft knows the flag. | Xometry and Jiga use GovCloud; Jiga enables per account |
| Who can open it | Only members whose identity provider asserts **US person** (and, for companies, a current ITAR registration on file), named on the design or workspace, with MFA. Every share checks those attributes at the moment it happens, not only when the invitation was sent. | Xometry Partner Terms §2.4; Jiga "designated US persons" + 2FA |
| No public paths | No public links, no publishing to a community gallery, no "fork to my account" outside the partition, no email notifications that contain thumbnails or geometry. | Jiga: email isn't an approved channel |
| Sharing with suppliers | Sharing to another organisation requires them to accept flow-down terms (no release to foreign persons, report violations, return or destroy), recorded with the share. | Xometry Partner Terms §2.1, §2.4, §5.3 |
| Retention | Each share has an expiry date and a "return or destroy" step when the job ends. The owner sees which copies are still out. | Xometry: 18 months after the order |
| Support access | Only US-person staff, with access logged and visible to the customer. | GovCloud administration is US-only |
| Turning it off | Only an org's **export compliance** role can clear the flag, with a reason, and every past share is listed so it can be reviewed. | (no public precedent found) |
| Access from abroad | A warning and policy option when an ITAR design is opened from outside the US. Whether that is lawful depends on who the person is and on the encryption carve-out, which is the organisation's call. | 22 CFR 120.54 |

**Why at the source matters (opinion):** marketplaces protect a design from the moment it's
uploaded. A design-side flag protects it from the first sketch, and it hands the flag to the
marketplace at upload instead of relying on someone remembering to tick a box at checkout.

### Proposed commands (sketch)

| Command | Input | Output |
|---|---|---|
| `policy.inspect` | — | effective policy, file path and SHA-256, locked keys |
| `marking.set` | `{target?, marking, reason?}` | new effective marking (with high-water mark) |
| `marking.inspect` | `{target?}` | marking and where it came from |
| `audit.list` | `{since?, kind?, limit?}` | events |
| `audit.verify` | `{path?}` | `{ok, lines, first_bad_line?}` |
| `export_control.set` | `{regime: "ITAR" \| "EAR", usml_category?, eccn?, note?}` | effective marking; designs raised by inheritance |
| `export_control.clear` | `{reason, approval?}` | cleared marking, or refusal naming the missing approval |
| `tool_data.prepare_share` | `{records}` | outgoing records (allow-listed fields only) and refused values with reasons |
| `library.import_pack` | `{path}` | `{signer, verified, entries, warnings}` |
| `security.report` | `{format?: "json" \| "pdf"}` | the report above |

All dialog-free, all with schemas, all audited when the profile is on.

### What contributors could build first (opinion, smallest useful steps)

1. The **egress inventory** document and `cargo xtask egress` check (no user-visible change).
2. **SBOM and provenance** on releases (release workflow only).
3. The **policy file** with only `updates.check`, `agents.allowed` and `network.egress`.
4. **Markings** in the model (one pending format file) with viewport banners and drawing banners.
5. **Audit log** in the command dispatcher.
6. **FIPS build** as a separate release flavour.
7. Signed **library packs** (shared with the library research topics).
8. The **ITAR switch** on top of markings (4): set, inherit, clear-with-reason, export statements.
9. The **tool-data share filter** (section 12), so tool sharing works on defense jobs from day one.

## Glossary

| Term | Meaning |
|---|---|
| ABAC | Attribute-based access control: access decided by attributes such as "US person". |
| Air gap | A computer or network with no connection to the internet; data crosses on media. |
| C3PAO | CMMC Third-Party Assessment Organization. |
| CAC / PIV | DoD Common Access Card / federal Personal Identity Verification card (smart cards). |
| CMMC | Cybersecurity Maturity Model Certification, DoD's contractor assessment program. |
| CMVP | NIST's Cryptographic Module Validation Program, which issues FIPS 140 certificates. |
| CTI | Controlled Technical Information, a CUI category covering military technical data. |
| CUI | Controlled Unclassified Information. |
| DEMIL code | FLIS code saying whether an item needs demilitarisation; a first export screen. |
| DFARS | Defense Federal Acquisition Regulation Supplement. |
| Distribution statement | A–F marking on DoD technical documents saying who may receive them. |
| EAR / ITAR | Export Administration Regulations (Commerce) / International Traffic in Arms Regulations (State). |
| FCI | Federal Contract Information. |
| FedRAMP | US government cloud authorization program (Low, Moderate, High). |
| FIPS 140-3 | US standard for cryptographic modules; validated modules have certificate numbers. |
| GovCloud | AWS GovCloud (US): isolated US regions for regulated workloads. |
| IL2/4/5/6 | DoD cloud Impact Levels from the Cloud Computing SRG. |
| SBOM | Software Bill of Materials (SPDX, CycloneDX). |
| SLSA | Supply-chain Levels for Software Artifacts; build provenance framework. |
| SSP | System Security Plan, the document an organisation is assessed against. |
| TDP | Technical Data Package (MIL-STD-31000). |

## Open questions

1. **Does the project want a "high-security profile" at all?** It is opt-in and changes no
   default, but it is a lasting maintenance surface. Who maintains the egress list?
2. **Where do markings live in STEP?** Which AP242 entities carry a marking so other CAD and
   PDM systems read it? (Needs the AP242 public documentation, not commercial CAD.)
3. **Patching policy for air gaps:** "latest release only" vs. advisories with version ranges.
4. **FIPS build cost:** is a C dependency (aws-lc) acceptable for a separate desktop flavour,
   given "application code is Rust only" applies to SolidCraft's own code, and `ring` already
   contains C and assembly?
5. **Self-hosting the cloud tier:** can craft-cloud / CollabCraft be deployed by a customer in
   GovCloud, and under what licence?
6. **FED LOG and contributed geometry:** licence terms for packs built from FED LOG; whether
   the project ever hosts contributed geometry for controlled items (probably never).
7. **Marking colours:** the purple/green banner convention vs. the colour-vision and
   high-contrast themes; always show text, never colour alone.
8. **Classified systems:** is there anything beyond "works fully offline, no egress, markings
   stored and shown" that the project should promise? (Probably not.)
9. **Rule dates:** re-check CMMC phase 2 (10 Nov 2026) and any newer DFARS class deviation before
   quoting.
10. **ITAR switch scope:** should clearing the switch ever be allowed on a desktop without an
    approval file, or only with a typed reason? Should EAR (dual-use) use the same switch with an
    ECCN, or a separate one?
11. **Marketplace hand-off:** do Xometry, Jiga or others offer an API field for "export
    controlled" that a design tool could set at upload? (Not found in public docs.)
12. **What exactly marketplaces hide:** whether ITAR jobs are hidden from non-registered or
    foreign partners in Xometry's partner job board is not stated publicly; RapidMade publishes
    no file-handling detail. Asking them directly would fill the gap.

## Sources

Accessed 2026-10-26 unless noted.

- Cooley, *DoD Releases Long-Awaited Final Rule Implementing CMMC Contract Clause* (Sept 2025):
  <https://www.cooley.com/news/insight/2025/2025-09-25-dod-releases-long-awaited-final-rule-implementing-cybersecurity-maturity-model-certification-contract-clause>
- Secureframe, *FedRAMP Equivalency for CMMC: the DoD memo explained*:
  <https://secureframe.com/blog/fedramp-equivalency-cmmc>
- NSF, *NIST 800-171 Rev 3 class deviation*:
  <https://www.nsf.org/ca/en/knowledge-library/nist-800-171-rev-3-class-deviation-controlled-unclassified-information>
- Kiteworks, *CMMC Phase 2 Pause: DFARS Revision 3 Explained* (search result only; page did not load):
  <https://www.kiteworks.com/cmmc-compliance/cmmc-dfars-revision-3-compliance/>
- Traefik, *FIPS 140-2 sunsets in September 2026*:
  <https://traefik.io/blog/fips-140-2-sunsets-in-september-2026-are-you-ready-for-fips-140-3>
- rustls manual, *FIPS*: <https://docs.rs/crate/rustls/0.23.43/source/src/manual/fips.rs>
- Arent Fox Schiff, *ITAR defense controls catch up with modern IT, allowing exporters to use encryption*:
  <https://www.afslaw.com/perspectives/alerts/itar-defense-controls-catches-modern-it-allowing-exporters-use-encryption>
- Secureframe, *A practical guide to marking CUI*: <https://secureframe.com/blog/cui-marking>
- DTIC, *CUI information*: <https://discover.dtic.mil/cui-information/>
- DoDI 5230.24, *Distribution Statements on Technical Documents*:
  <https://www.esd.whs.mil/Portals/54/Documents/DD/issuances/dodi/523024p.pdf>
- NIST MBE Summit 2024, Windham, *MIL-STD-31000 Technical Data Packages*:
  <https://www.nist.gov/document/2024-mbe-summit-windham-mil-std-31000-technical-data-packages-acquisition>
- Knox Systems, *Azure Government vs. AWS GovCloud*:
  <https://knoxsystems.com/resources/azure-government-vs-aws-govcloud>
- AWS, *GovCloud DoD SRG*: <https://aws.amazon.com/govcloud-us/dodsrg>
- AWS, *Amazon Bedrock models get FedRAMP High and DoD IL-4/5 approval in AWS GovCloud (US)* (23 May 2025):
  <https://aws.amazon.com/about-aws/whats-new/2025/05/amazon-bedrock-models-fedramp-high-dod-il-4-5-govcloud/>
- Xometry, *ITAR, DFARS, and export restrictions* (help article 666):
  <https://community.xometry.com/kb/articles/666-itar-dfars-and-export-restrictions>
- Xometry, *Drawing discrepancies in ITAR, EAR and CUI* (help article 853, preview only):
  <https://community.xometry.com/kb/articles/853-drawing-discrepancies-in-itar-ear-and-cui>
- Xometry, *Data protection*: <https://www.xometry.com/capabilities/data-protection/>
- Xometry, *Partner Terms and Conditions* (§1.2, §2.1, §2.4, §5.3):
  <https://www.xometry.com/partner-terms-and-conditions/>
- Xometry, *Legal* (ITAR registration letter, CMMC certificate): <https://www.xometry.com/legal/>
- Xometry, *Certifications*: <https://www.xometry.com/certifications/>
- Xometry, *Aerospace & defense supplier*: <https://www.xometry.com/aerospace-defense-supplier/>
- Xometry, *Xometry is CMMC Level 2 certified* (11 Mar 2025): <https://www.xometry.com/resources/blog/xometry-is-cmmc-certified/>
- Jiga, *ITAR and compliance*:
  <https://help.jiga.io/trust-and-compliance/8dT4ZL2u22PQXTya1r8RGD/itar-and-compliance/8aKkcfjdUEaFD5QDj7yCLc>
- RapidMade, home page and *ITAR certification*: <https://rapidmade.com>,
  <https://rapidmade.com/itar-certification/>
- Anthropic, *Public sector FAQs*: <https://support.claude.com/en/articles/13756069-public-sector-faqs>
- Greenberg Traurig, *Software bill of materials guidance for government contractors* (Oct 2025):
  <https://www.gtlaw.com/en/insights/2025/10/software-bill-of-materials-guidance-for-government-contractors>
- CISA, *2025 Minimum Elements for a SBOM* (draft): <https://www.cisa.gov/resources-tools/resources/2025-minimum-elements-software-bill-materials-sbom>
- SolidCraft repository: `SECURITY.md`, `docs/book/src/guide/updates.md`, `docs/releasing.md`
  ("Update signing"), `docs/downloads.md` ("Code signing"), `docs/crash-reports.md`, `docs/mcp.md`,
  `docs/agent-chat.md`, `docs/web-runtime.md`, `docs/plugins.md`,
  `docs/book/src/dev/local-libraries.md`, `deny.toml`, `.github/workflows/ci.yml`, `flake.nix`,
  `apps/solidcraft/Cargo.toml` (ureq, rustls with `ring`), `crates/ui-egui/Cargo.toml` (`minisign-verify`).
