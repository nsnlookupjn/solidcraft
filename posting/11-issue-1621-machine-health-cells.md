### Research: machine health, and running a machining cell

**In one line:** since CAM knows what the load of every move should be, every cut becomes a health check, and the same data lets SolidCraft plan and supervise a cell with robot loading and tool setting.

**What it covers**
- **Predicting failure:** measured spindle torque, axis load and vibration compared with the CAM prediction, with baselines from a daily warm-up check, alarm thresholds and trends.
- **Catching bad tools:** chipped, broken and worn tools, with "stop and re-measure on the tool setter" when in doubt.
- **Predicting bad work:** inspection data and coolant condition, turned into maintenance work orders the CAM also reacts to.
- **Cell automation:** robot part loading and unloading and tool setting, planned from SolidCraft CAM (grips, robot reach and clearance, tool budget, the handshake).
- **Safety stays where it belongs:** the Edge supervises, queues and records, but never sits in the safety chain or the real-time robot and machine handshake.
- **Presetter offsets without typing:** staged as a `G10` program and checked by the machine's own tool setter.
- **Export control:** health data, remote monitoring, shared baselines and cell templates block sanctioned regions and screen users; nothing from a controlled job leaves the shop.

**Both build on SolidCraft Edge** from the previous comment and on the machine library already posted here.

**Research files:** attached (MIT OR Apache-2.0), also on my fork: [cam-research-nsnlookupjn-machine-health.md](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-jfhnp0/research/cam-research-nsnlookupjn-machine-health.md) and [cam-research-nsnlookupjn-cell-automation.md](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-jfhnp0/research/cam-research-nsnlookupjn-cell-automation.md)

Feedback welcome, especially on where machine health data should live, and whether cell planning belongs in CAM or a separate workspace.
