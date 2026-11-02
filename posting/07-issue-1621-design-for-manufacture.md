### Research: design for manufacture, physical properties and buying the stock

**In one line:** check a design against the shop's real tools and machines while it is being drawn, carry hardness and heat treatment from the model into CAM, and size and buy the blank from the model.

**What it covers**
- **DFM check while designing:** what instant-quote services flag (thin walls, sharp inside corners, deep pockets, hole and thread depth, undercuts, draft, sheet-metal relief), checked against the shop's own tools, holders and machines, each finding with a reason, a fix and a cost or time delta.
- **Design-to-CAM cost optimiser:** small named design moves (bigger corner radius, snap to stock size, standard drill and thread sizes) tried in copies; never touches functional or toleranced faces; an accepted change becomes a new revision.
- **Physical properties in the model:** hardness, heat treatment with case depth (ISO 18203), coatings and "left as cast" surfaces, per part or per face region.
- **Hardness in CAM:** stock carries hardness by region and depth, so each move uses what is actually being cut (cast skin, scale, flame-cut edges, hardened zones).
- **Derived stock:** design once at final size, then derive the casting, forging, printed or bar blank and the before and after heat-treatment states.
- **Buying the stock:** a supplier-neutral stock request quoted with several metal suppliers; the mill certificate is checked against the order (including DFARS specialty-metal melt origin) and its heat number follows every part.
- **Export control:** supplier quotes, uploads and the marketplace screen every buyer and seller and block sanctioned regions; DFARS and country-of-origin rules apply to the material itself.

**Phase 1 of the DFM check needs no CAM** and builds on what SolidCraft already has (`inspect.accessibility`, `inspect.draft`, `inspect.parameter_sweep`, the Hole feature). Rules of thumb are only ever advice.

**Research files:** attached (MIT OR Apache-2.0), also on my fork: [cam-research-nsnlookupjn-dfm-check.md](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-jfhnp0/research/cam-research-nsnlookupjn-dfm-check.md), [design-research-nsnlookupjn-physical-properties.md](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-jfhnp0/research/design-research-nsnlookupjn-physical-properties.md) and [cam-research-nsnlookupjn-stock-marketplace.md](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-jfhnp0/research/cam-research-nsnlookupjn-stock-marketplace.md)

Feedback welcome, especially on whether the design-side DFM phase should get its own issue, and whether supplier connectors belong in this repository.
