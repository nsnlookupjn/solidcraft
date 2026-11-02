### Research: SolidCraft inside government (DoD, NASA, allies, and unfriendly governments)

**In one line:** what it takes for government engineers to use SolidCraft in house, and one opt-in **government mode** that would make it adoptable for every kind of government work, from public projects to isolated classified networks, in the US and allied countries.

**What it covers**

- **Policy is on our side:** the DoD CIO's 2022 memo says open source is commercial software that must get *"equal consideration"*, under an "Adopt, Buy, Create" order. NASA, Canada (open source first since 2018) and Germany's Bundeswehr (openDesk, 2025) point the same way. BRL-CAD (Army) and OpenVSP (NASA) are open design tools already used inside government.
- **What a government employee must watch:** approved software lists and ATOs, Section 508 accessibility (free software isn't exempt), keeping government designs out of the public browser app, commercial AI endpoints and bug reports, ethics rules, and how public-domain contributions from federal employees fit our sign-off rules.
- **An honest answer on DoD and NASA:** not as their production CAD today; realistic soon for labs, depots, test ranges, schools, shipboard additive work and isolated networks, *if* we supply evidence. The DoD's 2025 Software Fast Track says open source provenance is its main worry.
- **Government mode with four levels:** Public, Controlled (CUI and allied equivalents), Export controlled (a per-design flag) and Isolated, set by an administrator's policy file. A table shows what each level switches on and off and which requirement it meets, plus an **offline build** with networking compiled out for classified enclaves.
- **Country profiles as data, not code:** marking labels, supplier cyber standards, crypto approval and export regime names for the US, UK, Canada, Australia, NATO and the EU.
- **An eight-item evidence pack** an agency sponsor can file (security report, SBOM, licence check, STIG-mapped hardening guide, accessibility report, advisories, governance, network statement), plus a **government LTS release line**.
- **Unfriendly governments:** no open-source licence can exclude them. The real risks are controlled data leaking into public libraries and reports, defence services, sanctioned access to hosted services, and hostile contributions. It includes a worked example (can Russia use SolidCraft to design drones?) against the 2024 US CAD/CAM export and services rules and the EU and UK bans.
- **Team rules:** **no one goes to jail** (no in-house counsel, so the most cautious reading, refuse rather than warn), **help users comply**, and **prevention first**. Questions only a lawyer can answer are marked [Lawyer], and the file lists free official help and pro bono routes.
- **A standing export and sanctions watch:** a weekly automated scan of the Federal Register and EUR-Lex, monthly triage, a quarterly written review and a yearly adviser check.

**None of this changes a default.** The smallest first steps are documents and CI jobs: a network behaviour statement, SBOM and provenance on releases, a hardening guide, and an accessibility self-assessment. It extends the high-security profile in this issue, and relates to #787, #1621, #2869, #7426 and #8648.

**Research file:** attached (MIT OR Apache-2.0), also on my fork: [gov-research-nsnlookupjn-government-use.md](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-jfhnp0/research/gov-research-nsnlookupjn-government-use.md)

Feedback I'd most like: who hosts solidcraft.io and the browser app, and under which country's law? That decides which sanctions rules the project's own services must follow.
