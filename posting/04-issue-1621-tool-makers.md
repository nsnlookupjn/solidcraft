### Research: tool makers and the physics data behind tool choice

**In one line:** find out which tool makers publish data that physics-based CAM can use, so SolidCraft can help anyone **choose tools by physics**, for tools they already own and when buying new ones.

**What it covers**
- **Five kinds of tool data physics needs:** geometry, cutting material and coating, force coefficients, stiffness and dynamics, and limits and life.
- **What makers publish today:** geometry is well covered by ISO 13399 / GTC, force data exists mainly as per-material `kc1`/`mc` tables, and stiffness and dynamics are almost never published.
- **Maker-by-maker table:** where the data lives, its format, what each maker's adviser computes and where to buy, covering Sandvik, Kennametal, Iscar, Seco, Walter, Mitsubishi, Harvey/Helical, Haas Tooling, MachiningCloud and ToolsUnited.
- **Combining sources:** ISO 13399 geometry, published material tables, shop tap tests and shop results, each with its source and confidence.
- **Tools you own vs tools to buy:** rank candidates by predicted time within limits, show the binding limit, and list where to buy.
- **Export control:** maker data and where-to-buy links bring in vendor accounts, so hosted parts block sanctioned regions and screen buyers and sellers; only generic tool data is ever shared from a controlled job.
- **Guiding new users:** a novice view (Best / Good / Safe in shop language) and a pro view (every tool, every limit, every source), on the same physics engine.

**Choosing tools by physics is new to most programmers and operators,** so the data has to be complete and the explanations plain. This builds on my tool library research above. It is not in any way complete: I'll keep adding to it, and I encourage others, **tool makers especially,** to correct and extend the table.

**Research file:** attached (MIT OR Apache-2.0), also on my fork: [cam-research-nsnlookupjn-tool-makers-physics-data.md](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-jfhnp0/research/cam-research-nsnlookupjn-tool-makers-physics-data.md)

Feedback welcome, especially from tool makers on the table, and on whether the novice view should be on by default.
