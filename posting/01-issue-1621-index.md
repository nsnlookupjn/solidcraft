### Research: an index of my CAM research, and what's coming

**In one line:** I've posted eight CAM research files here so far and have more to come, so this comment lists them all in one place, says what changed, and spreads the rest out over the next week or so instead of flooding the thread.

**Already posted here**
- **Tool library:** a shared ISO 13399 catalogue, cutting data per material, shop assemblies.
- **Cutting physics:** chip thinning, force models, spindle limits, deflection, a conservative-to-aggressive slider.
- **Workholding:** checking a hold against the cutting forces and suggesting the best hold.
- **Stock integrity:** toolpaths that respect the stock getting weaker as it is cut away.
- **Machine library:** a physics entry per machine model, plus the shop's measured instance.
- **CAM Physics System:** ties the above into one limits engine and a physics-checked simulation.
- **In-process inspection:** tolerances in the model, probing, certification and SolidCraft Edge.
- **Tool holding:** RPM and torque limits per holding style, slip and pull-out checks.

**What changed since then:** every file now has an **Export control, sanctions and government use** section. It covers what in each file crosses a border, how hosted parts would block sanctioned regions and screen people, the government-mode levels, and a check that the file holds no controlled data. The team rules behind it are short: **no one goes to jail** (no in-house counsel, so the most cautious reading and refuse rather than warn), **help users comply** (plain-language stops, never legal advice), and **prevention first** (nothing hosted, shared or paid ships before its checks pass). The current copy of every file is in one folder of my fork: https://github.com/nsnlookupjn/solidcraft/tree/claude/project-thread-jfhnp0/research

**Coming next on this issue, a few at a time:** the physics engine architecture and GPU compute; tool makers' physics data; threads, gears, splines and broaching; milling practice and ten shop-floor features; DFM, physical properties and buying stock; machine interfaces and SolidCraft Edge; machine health and cell automation. Related work goes on #1626 (additive), #787 (NSN parts), #2869 (compliance and IP protection), and two new feature requests (tolerances in the model, and a high-security profile).

**All of it is MIT OR Apache-2.0,** none of it changes a default, and I don't write code for this project, so every idea is up for grabs. If you'd rather I post these somewhere else, or as fewer, bigger comments, just say and I'll change course.
