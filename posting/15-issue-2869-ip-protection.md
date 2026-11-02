### Research: protecting designs from IP theft, from NDA to finished part

**In one line:** make SolidCraft the CAD/CAM program that best **tracks and helps prevent IP theft**, with layered digital fingerprints in every file that leaves, signed provenance, NDA-aware sharing with suppliers, and evidence a lawyer can use. Novices get it with one "Confidential" switch; pros get every control.

**What it covers**
- **Honest about open source:** no program can stop a determined thief, and DRM doesn't work. So the aim is to make the owner's legal position strong, make leaks traceable, make authorship provable, and help honest users stay honest.
- **"Reasonable steps" made provable:** trade-secret laws in the US (DTSA), EU (Directive 2016/943), UK, Germany, Japan and China only protect owners who kept information secret. A German court refused protection in 2024 (BAG 8 AZR 172/23) because the employer couldn't show its measures. Markings, need-to-know, NDAs linked to designs and a share log give users that record.
- **Layered digital fingerprints:** visible notice, signed metadata, a fragile per-recipient fingerprint hidden in choices that don't change the part (mesh order, tessellation, G-code segmentation), an optional sub-tolerance geometric fingerprint (never on critical features, off by default), and a hashed, timestamped provenance record.
- **"Who leaked this?" in one command:** drop in a suspicious STL, STEP, 3MF, G-code or PDF; SolidCraft matches fingerprints to the owner's share ledger, or falls back to shape-similarity search. Leads, not accusations.
- **NDA register linked to designs:** parties, purpose, term, permitted recipients, sub-tier flow-down, and return-or-destroy reminders. Shares outside an NDA are refused.
- **Share the minimum:** a share wizard sends a quote, a machine or an inspector only what that purpose needs, never the feature history.
- **Evidence bundles:** design history, hashes, audit log and share records in one sealed archive for the owner's lawyer.
- **Not infringing others:** licence tags on library parts, origin records on imports, and an opt-in patent and design watch. The EU design reform (Regulation 2024/2822, Art. 19(2)(d)) now covers files that record a protected design.
- **Export control and team rules:** fingerprints and share records never send anything home; hosted parts follow the same region blocks and screening as every other file, and the team rules (no one goes to jail, help users comply, prevention first) apply.
- **Country table** covering trade secrets, designs and copyright in the US, EU, Germany, UK, China, Japan, Korea, India, Canada and Australia, plus a comparison of Onshape, Teamcenter, SOLIDWORKS PDM, Xometry and Protolabs Network. **None of them traces a leak once a file has left.**

**Never spyware:** every fingerprint is made by the design owner on their own outgoing files, shown to anyone who opens the file in SolidCraft, and nothing ever phones home. **Nothing changes a default.** The first steps are small: a Confidential switch and banner, notice metadata on exports, and a share ledger. It builds on my high-security, compliance and government-use research and fits the PDM work in #2869.

**Research file:** attached (MIT OR Apache-2.0), also on my fork: [ip-research-nsnlookupjn-ip-protection.md](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-jfhnp0/research/ip-research-nsnlookupjn-ip-protection.md)

Feedback I'd most like: should confidentiality markings share one design-format file with the government markings, or have their own?
