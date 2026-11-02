### Research: process physics and process data for 3D printing

**In one line:** the limits on how fast and how well each printing process can build, and a shared material and machine library that holds them, covering topics 8, 9, 10, 15 and 16 of this request.

**What it covers**
- **FDM limits:** hot-end melt capacity in mm³/s, pressure lag and layer cooling, so a print time estimate respects what the machine can actually melt.
- **Resin:** the working curve (`Dp`, `Ec`) and what it means for exposure and layer thickness.
- **Metal (LPBF and DED):** melt-pool criteria and the process windows behind them; this part may also help #1625.
- **A shared material and machine library:** published values plus a per-shop layer of measured ones, each with its source.
- **Open spool tags:** reading material data from tags such as OpenPrintTag.
- **Design checks and estimates:** where SolidCraft could warn while designing, and where it should just read a slicer's profile.
- **Export control:** the shared library and spool-tag data block sanctioned regions and screen contributors, and only generic process data is ever shared.

**This stays a hand-off to slicers, not a slicer** (#2595 sends 3MF; the in-app slicer in #6871 was closed). It gives SolidCraft the numbers to warn early and estimate honestly.

**Research file:** attached (MIT OR Apache-2.0), also on my fork: [3d-printing-research-nsnlookupjn-process-physics.md](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-jfhnp0/research/3d-printing-research-nsnlookupjn-process-physics.md)

Feedback welcome, especially on whether SolidCraft should keep this data at all, or only read it from a slicer profile or a tag at estimate time.
