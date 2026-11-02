**What it covers**
- **An opt-in profile set by a machine-wide policy file:** no file means today's behaviour, so **no default changes**.
- **Egress inventory:** every network path (update check, agent chat, web app and cloud storage, future library sync), enforced by an xtask check like `cargo xtask layers`, with `network.egress = "deny"` and an allow-list.
- **Markings as design data:** CUI banner and category, distribution statement A–F, ITAR/EAR, with a high-water mark when a marked part is inserted, carried onto viewport banners, drawings, STEP/3MF metadata, screenshots and renders. One pending format file.
- **A per-design "Export controlled" switch that locks in:** it carries into copies, assemblies and exports, blocks paths that would release the design (unapproved AI endpoints, public libraries, non-approved cloud storage), and clears only with a logged reason or a compliance officer's approval.
- **Tool data stays shareable on defense jobs:** only design, test and inspection data is controlled, so tool records can still reach a shared library through a field allow-list that strips anything derived from the design.
- **Audit log, FIPS build and evidence:** an append-only, hash-chained log from the command dispatcher (so MCP and UI are covered alike), a FIPS build flavour, SBOM and build provenance on releases, and a `security.report` command.
- **Signed offline library packs** that cross an air gap on media, shared by the tool, workholding, machine and NSN library ideas.
- **Marketing from controlled designs:** a scrubbed marketing derivative with its own release gate, never the real geometry.

**Our team rules shape all of it:** **no one goes to jail** (no in-house counsel, so the most cautious reading, refuse rather than warn), **help users comply** (plain-language stops, never legal advice) and **prevention first** (risky features stay off until their checks pass). It's research, not legal advice, and software isn't "certified" by itself; organisations' systems are.

**Smallest first steps, none user-visible:** the egress inventory and its check, then SBOM and provenance on releases. Related: #787, #1621, #1221 (plugin provenance), #8648 (hosted AI connector).

**Research file:** attached (MIT OR Apache-2.0), also on my fork: [security-research-nsnlookupjn-high-security.md](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-jfhnp0/research/security-research-nsnlookupjn-high-security.md)

Feedback welcome, especially on whether markings should share one design-format file with the confidentiality markings in my IP protection research.
