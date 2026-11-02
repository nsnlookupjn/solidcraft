### Research: talking to the machine, and SolidCraft Edge on every brand

**In one line:** what today's CNC, additive and CMM controls let outside software read and do, a ranked plan for connecting to the machines shops actually own, and **SolidCraft Edge**, a small Rust device at the machine that closes the loop back to CAM.

**What it covers**
- **What each control allows:** Haas, Fanuc, Siemens, Heidenhain, Okuma, Mazak and Brother, and how live inspection works with a Haas today.
- **Ranked by installed base:** CNC controls, additive brands and CMMs, with control generations and software versions; Haas as the worked example, generation by generation.
- **Open standards first:** MTConnect and OPC UA/umati before vendor APIs; every connection route, and a read-only plug-in module with no outputs in hardware.
- **Limits in the control, not in polling:** the post writes per-tool load limits into the control's own monitor, because polled data is too slow to stop a breaking tool.
- **Every factory feature, every cut:** smoothing, adaptive feed, chatter avoidance and probing, switched on and explained for novices, visible and overridable for pros.
- **An MCP profile for machines:** read-only first, layered on MTConnect and OPC UA; software can make the machine safer but has no tool to start or move it.
- **Edge as open hardware:** a Raspberry Pi 5 class device, Rust services only, with the board and enclosure designed in SolidCraft itself.
- **Export control:** Edge is in scope as an asset in a controlled shop, remote dashboards follow government mode, and the open hardware design stays generic.

**None of this needs CAM code first.** The tables seed the machine library's capability model. Hosting Edge here raises questions only maintainers can answer: whether it lives in this repo, how the wasm rule applies to firmware, and which open hardware licence to use (CERN-OHL-P-2.0 suggested).

**Research files:** attached (MIT OR Apache-2.0), also on my fork: [cam-research-nsnlookupjn-machine-interfaces.md](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-jfhnp0/research/cam-research-nsnlookupjn-machine-interfaces.md) and [cam-research-nsnlookupjn-edge-machine-integration.md](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-jfhnp0/research/cam-research-nsnlookupjn-edge-machine-integration.md)

Feedback welcome, especially on hosting Edge here and on the open-standards-first order.
