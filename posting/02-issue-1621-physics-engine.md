### Research: the physics engine behind CAM, and running it on GPUs

**In one line:** how the CAM Physics System could actually be built: exact geometry plus a fast stock proxy, standard force models at every toolpath step, a CPU reference that works everywhere, and GPUs (local, shop or cloud) as an optional accelerator.

**What it covers**
- **Two representations:** the exact B-rep from `crates/kernel` for boundaries and finishing, and a stock proxy (sparse voxels, dexels or a narrow-band SDF) for engagement at every step.
- **Force model:** linear edge-force or Kienzle per flute and axial slice, summed into force, torque and power, with the feed scaled to the tightest limit of tool, spindle, machine, workholding and stock.
- **CPU first, GPU optional:** a wasm-safe CPU core below the UI layer defines the answers; a `wgpu` path in layer 5 runs the same maths faster and must match it within a stated tolerance.
- **Process planning:** manufacturing feature tags linked to the parametric timeline, and a graph of stock states from billet to part, so a design change re-runs only what it affects.
- **Where the GPU comes from:** the user's own card, other GPUs in the shop, or rented cloud GPUs, through one headless worker program.
- **Closed-loop data contracts from day one:** calibration profiles, live machine data and inspection deviations, so the CAM can later tune itself.
- **Export control:** a controlled design (ITAR, EAR, CUI) never goes to a GPU or library outside an approved boundary; under government mode only GPUs inside the boundary are allowed, and calibration data is shared only through a field allow-list.

**Everything is a SolidCraft command,** so the UI, scripts, CLI and agents over MCP use the same engine. This is the technical companion to my CAM Physics System comment above.

**Research files:** attached (MIT OR Apache-2.0), also on my fork: [cam_physics_architecture.md](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-jfhnp0/research/cam_physics_architecture.md) and [cam-research-nsnlookupjn-gpu-compute.md](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-jfhnp0/research/cam-research-nsnlookupjn-gpu-compute.md)

Feedback welcome, especially on the stock proxy (voxels, dexels or SDF), and on whether a remote worker could ever use CUDA (for FEA, #1620) or should stay on `wgpu`.
