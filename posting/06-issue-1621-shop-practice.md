### Research: milling best practice and ten shop-floor CAM features

**In one line:** turn what good machinists do by habit into rules CAM applies and the simulation checks, and add ten features shops keep asking for.

**What it covers**
- **Best practice as rules:** coolant timing for carbide (dry or constant flood, never switched mid-cut), roll-in and roll-off, lead-ins with a smoothly ramped feed, and datums and the tool setter.
- **Chip clearing where the simulation predicts trouble:** air, fans or coolant inserted where chips would be recut or land on finished faces.
- **Cleaning as standard operations:** washdown, part wash and dry, camera and tool cleaning.
- **Finish-first planning:** roughing derived from what the finishing tools need.
- **New tool types and processes:** angle heads in standard tool changers, grinding on mills, burnishing, flexible hones and abrasive brushes, and 4th-axis turning.
- **Smarter motion:** stock-aware rapids instead of clearance planes, and global G2/G3 arc fitting.
- **Serial engraving and tool setting:** live text fields for serials, and tool-setter placement with a safe length search instead of a tape-measured rough length.
- **Export control:** milling practice is local only; the sharing and online parts of the features file follow the region-block and screening rules in its export section.

**Every rule is advice a pro can override and a novice gets by default,** and each one adds a check to the simulation gate described in my CAM Physics System comment.

**Research files:** attached (MIT OR Apache-2.0), also on my fork: [cam-research-nsnlookupjn-milling-best-practices.md](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-jfhnp0/research/cam-research-nsnlookupjn-milling-best-practices.md) and [cam-research-nsnlookupjn-general-features.md](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-jfhnp0/research/cam-research-nsnlookupjn-general-features.md)

Feedback welcome, especially on whether cleaning steps should be CAM timeline operations or post-processor options, and which of the ten features matters most.
