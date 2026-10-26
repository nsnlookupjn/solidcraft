# CAM research: GPU acceleration, local GPU networks and GPU-as-a-service

Research for [#1621](https://github.com/tangency-labs/solidcraft/issues/1621) (Research Request
for Plugin: CAM). This file covers **how SolidCraft's CAM could use graphics processors (GPUs) to
compute stock simulation, collision checks, physics and toolpaths faster**, and where those GPUs
can be: in the user's own computer, on other computers in the same shop ("a local GPU network"),
or rented from a cloud provider ("GPU-as-a-service"). It also covers what must stay on the CPU,
how to keep results identical whichever device computed them, what remote compute costs, and how
to keep controlled design data on approved machines only.

It belongs to the set of files that treat CAM as one physical system. The
[CAM Physics System overview](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-vgqo3g/research/cam-research-nsnlookupjn-physics-system.md)
describes the solver loop that checks every toolpath segment against the tool, machine,
workholding and stock limits; this file is about **making that loop fast enough to run on every
segment of every program**, and fast enough to try many alternatives (feeds, slider positions,
cut orders) instead of one. The
[stock integrity](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-stock-integrity/research/cam-research-nsnlookupjn-stock-integrity.md)
file is the heaviest user: it needs the in-process stock, its stiffness and its vibration modes
updated as material is removed.

Licence: MIT OR Apache-2.0, the same as SolidCraft (see the Licence section of CONTRIBUTING.md).

## About this research

- **Author:** Jason ([@nsnlookupjn](https://github.com/nsnlookupjn)). The idea (use local GPUs,
  local GPU networks and standard GPU compute services to speed CAM simulation and toolpath
  calculation) is Jason's; source research and writing done with an AI assistant (Claude) in
  October 2026.
- **How it was produced:** web searches of public sources (published papers' abstracts and open
  full texts, vendor announcements and trade press, cloud providers' public pages, standards
  mailing lists), each claim checked against the cited page where possible, plus a reading of
  SolidCraft's own `crates/gpu` crate to see what already exists. No commercial CAD/CAM program
  was driven or scripted, no GPL/LGPL/AGPL source was read (this rules out, for example,
  OpenCAMLib and PyCAM, which are LGPL/GPL), and no text is copied beyond short quotes. The
  sizing estimates are arithmetic done for this file and are marked as estimates.
- **Units:** millimetres, micrometres (µm), seconds, bytes (MB = 10⁶ bytes, MiB = 2²⁰ bytes),
  US dollars per hour for prices, unless a source is quoted in other units.
- **Marking:** "(unverified)" marks a claim not confirmed against a source. **Fact**,
  **opinion**, **estimate** and **marketing** are labelled where they mix. Prices change often;
  every price here is "as published on the date shown" and only an order of magnitude.

**Author's experience (personal experience, not a cited source):**

- **Shop floor since 1987:** a machine shop apprenticeship from 1987 to 1991, starting on manual
  machines, then writing G-code at the control on early CNC machines (Milacron and similar, with
  no way to save a program).
- **CAD/CAM over about 35 years:** most major packages. In automotive work: Unisurf, Siemens,
  Sketchpad and NX, with programs stored on punch cards, then tape, then floppy disks; later
  CATIA, AutoCAD and Pro/Engineer. Currently fluent in Fusion, Mastercam, SOLIDWORKS and NX.
- **Engineering and software:** BTEC HND in Electrical and Mechanical Engineering, MEng, PhD in
  Materials Science, certified welder. Programming since childhood (machine code, BASIC, C++,
  C#, assembler, web and database technologies). Owns engineering and software companies.
- **Current work:** engineering and manufacturing spare parts for defence and aerospace,
  including flight-safety components.

**This research is not complete.** It is a starting point: the author will keep researching
and adding to it, and others are encouraged to do the same, correct it and contribute their own
findings and shop experience.

> **[TODO Jason: personal experience]** CAM jobs you have waited on (which operations, how long,
> on what computer), whether your CAM uses the GPU or all CPU cores, whether a shop you know has a
> spare GPU workstation or server that sits idle overnight, and any rules you work under about
> where design data may be processed (ITAR, CUI, customer contracts). Label this as personal
> experience.

## Summary

1. **Most of the slow part of CAM is the same small operation repeated millions of times.**
   Placing a tool at a point and asking "what material is under it, how much does it remove, does
   anything collide, how much force does that make" is independent at each point. That is exactly
   the kind of work a GPU, with thousands of simple cores, does well. Stock (material removal)
   simulation, collision and gouge checking, "drop-cutter" surface toolpaths, offsetting, and the
   per-segment engagement and force calculation that the CAM Physics System needs all fit.
2. **Commercial CAM components have moved there in the last two years (fact, marketing
   numbers).** ModuleWorks announced GPU material removal simulation in August 2024, claiming up
   to 10 times faster than CPU on a mid-range card and over 20 times on a high-end one;
   MachineWorks shipped "MachineWorks GPU" with version 8.7 in July 2025 for material removal and
   collision detection, claiming 10 to 25 times for the stock model; TopSolid announced it would
   ship the MachineWorks GPU simulation. An MIT-licensed browser project, `raster-path`, generates
   3-axis raster toolpaths with WebGPU compute shaders and reports 20 to 100 times over its own
   WebAssembly CPU version. Research papers report 5 to 50 times for dexel simulation and voxel
   offsetting.
3. **Some parts stay on the CPU.** Adaptive clearing is a chain of decisions where each step
   depends on the stock left by the previous one, so it parallelises **inside** a step (evaluate
   many candidate directions at once) and **across** independent regions or levels, but not along
   the path. Exact B-rep work, G-code output and the final pass/fail verdict are cheap and need
   double precision. **Opinion:** the clean split is "the GPU proposes and screens, the CPU
   certifies": every result that ends up in a program is re-checked in double precision on the
   CPU, so a GPU (which in WebGPU has no 64-bit floats, and on consumer cards has no error-
   correcting memory) can never silently make a program wrong.
4. **SolidCraft already has the right GPU foundation.** Its `crates/gpu` crate uses `wgpu` (the
   Rust implementation of WebGPU) with WGSL compute shaders for display tessellation and for a GPU
   path tracer, runs on Vulkan, Metal, DX12 and in the browser, and is `forbid(unsafe_code)`.
   **Opinion:** CAM kernels should be written once in WGSL through `wgpu`, not in CUDA, OpenCL or
   SYCL. That keeps the project's rules (Rust only, builds for wasm, no unsafe, any GPU vendor)
   and lets the same kernels run in the desktop app, in the browser and on a headless worker.
5. **One worker program for local GPUs, local GPU networks and the cloud (Jason's idea, design
   is opinion).** A small headless SolidCraft compute worker runs the same Rust code and the same
   GPU kernels. On the user's computer it is just a thread; on another shop PC or a GPU server it
   listens on the local network; in the cloud it runs on a rented GPU virtual machine from any
   provider (AWS, Azure, Google Cloud, Lambda, RunPod and so on). SolidCraft sends it a
   self-contained, hashed job, gets back results, and certifies them locally. No provider-specific
   API is needed beyond starting a machine.
6. **Remote GPUs pay off for batch work, not for dragging a slider.** Uploading a job is small
   (an estimated few megabytes) and a cloud GPU costs roughly $1 to $4 per hour on demand in late
   2026 (a few minutes of compute costs cents). But a cold cloud machine takes time to start, and
   every round trip adds tens of milliseconds. Interactive work stays on the local GPU; long
   verifications, whole-program physics checks, and "try 200 cut orders and keep the best" go to
   the network or the cloud.
7. **Controlled data never leaves approved machines.** The default is **local only**. A design
   can carry a data class (for example ITAR, CUI or customer-confidential), and the engine itself,
   not the user interface, refuses to send a job to any endpoint not approved for that class.
   AWS GovCloud (US) offers GPU instances (G6 with NVIDIA L4, in US-West since November 2024 and
   US-East since August 2025), and Azure Government has offered GPU virtual machines since 2017,
   so a controlled-data shop can still rent GPUs inside an approved boundary, or run the worker on
   its own hardware only. This follows the markings and endpoint trust levels proposed in the
   [high-security research](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-2fa4g1/research/security-research-nsnlookupjn-high-security.md).
8. **The CPU path is never removed.** Every GPU kernel has a CPU twin (using `rayon` threads,
   already a SolidCraft dependency) that gives the same answer within a stated tolerance; tests
   compare them; a missing GPU, a browser without WebGPU, or a GPU lost in the middle of a job
   falls back to the CPU and the user is told.

## Fundamentals

### Why GPUs help (fact)

A CPU has a few to a few dozen powerful cores. A GPU has thousands of simpler ones, grouped so
that many threads run the same instructions on different data at once, and it has much higher
memory bandwidth. Work is fast on a GPU when it is:

- **data-parallel**: the same calculation on millions of independent items (grid cells, tool
  positions, triangles, toolpath segments);
- **arithmetic- or bandwidth-heavy** relative to the data that has to be copied in and out;
- **regular**: few branches that send neighbouring threads different ways, and memory read in
  neighbouring blocks.

It is slow when each step depends on the result of the step before (a long serial chain), when
the work per item is tiny compared with the cost of copying the data to the GPU, or when the
algorithm needs a lot of irregular pointer-chasing (as in exact B-rep topology).

MachineWorks' own explanation of its GPU release says the same thing in one line: data-parallel
operations benefit because GPUs have thousands of cores against tens in a CPU.

### Which CAM work is data-parallel

| Workload | What repeats | Parallel over | GPU fit | Who uses it in this file set |
|---|---|---|---|---|
| **Stock (material removal) simulation**, dexel or voxel | Subtract the tool's swept shape from each grid column or cell | Grid cells × tool positions | **Very good** (fact: ModuleWorks, MachineWorks, papers) | All; the in-process stock model |
| **Collision and gouge checking** (holder, shank, machine parts against stock, part and fixture) | Distance or overlap test of a shape against stored geometry | Tool positions × geometry elements | **Very good** (fact: MachineWorks GPU does collision) | Machine library, workholding |
| **Drop-cutter / raster surface toolpaths** (3-axis finishing on a height map) | Lowest safe tool height over a grid of XY points | XY points × triangles or height-map cells | **Very good** (fact: raster-path; GPU z-map papers) | Toolpath generation |
| **Offsetting** (contour and 3D offsets as morphological dilation of a voxel model) | A small filter applied at every cell | Cells | **Very good** (fact: Georgia Tech HDT paper, ~50× vs 8 cores at 512³) | Toolpath generation, rest machining |
| **Engagement per segment** (radial and axial depth, engagement angle, chip thickness) | Intersection of tool footprint with current stock at each step | Segments | **Good**: a by-product of the stock update | Cutting physics, CAM Physics System |
| **Force, torque, power per segment** from engagement and cutting coefficients | A short formula per tooth angle per segment | Segments × tooth angles | **Very good**, cheap once engagement is known | Cutting physics, machine library (torque curves) |
| **Limit negotiation per segment** (minimum over tool, machine, fixture, stock limits along the force direction) | Compare force vector with each link's limit | Segments | **Good** (simple), but the result drives feed changes that change later engagement | CAM Physics System solver loop |
| **Stock stiffness and modes** (beam/plate estimates on recognised features; later FE models) | Formula per feature; or a sparse linear solve | Features; matrix rows | Formulas: trivial on CPU. FE solve: **moderate** (fact: COMSOL reports 2× on a structural model, up to 5× on multiphysics with NVIDIA's sparse solver) | Stock integrity, FEA request #1620 |
| **Adaptive clearing path search** | Choose the next step from the stock left by the last | **Within** a step: candidate directions; **across** independent regions and Z levels | **Partial** (opinion) | Toolpath generation |
| **Exploring alternatives** (feeds, slider positions, cut orders, setups) | Run the whole check again with a different choice | Candidates | **Excellent for many GPUs or machines**: each candidate is independent | Stock integrity sequence planner, risk slider |

The last row is the one that makes **remote** GPUs worth having. A single program check is
usually fast enough on one good local GPU. Trying hundreds of cut orders, or the whole risk
slider range, is not, and those runs are fully independent, so they spread across as many GPUs
as the shop or the cloud can offer.

### Stock models that suit a GPU (fact, with estimates)

- **Z-map (height map):** one height per XY grid cell. Simple and fast; only for 3-axis work
  without undercuts. Used by GPU drop-cutter and raster methods.
- **Dexel:** a column per grid cell holding a list of solid intervals along the column, so
  undercuts and through-holes work. **Tri-dexel** keeps three grids along X, Y and Z, which
  restores accuracy on walls that are parallel to one grid direction; the KU Leuven paper below
  uses an adaptive tri-dexel grid, denser near the cut. Commercial simulation (ModuleWorks says
  its simulation is dexel-based) uses this family.
- **Voxel / sparse voxel trees:** a 3D grid of filled or empty cells, often stored as a tree so
  empty and full regions cost little. The Georgia Tech "hybrid dynamic tree" is an example, run
  at up to 2048³ on one GPU.
- **Signed distance fields (SDF):** each cell stores the distance to the nearest surface. Good
  for collision and clearance queries ("how far is the holder from the stock?"), and widely used
  for GPU collision in robotics and physics simulation.

**Estimate (this file's arithmetic, opinion on the conclusion):** a 200 × 150 × 80 mm block on a
0.05 mm Z-dexel grid has 4000 × 3000 = 12 million columns. With up to four intervals per column
stored as 32-bit integers that is about 12 M × 4 × 2 × 4 bytes ≈ 384 MB, plus counts, which fits
in an 8 GB GPU but not in one WebGPU storage buffer at the default limit (128 MiB per binding,
256 MiB per buffer; adapters can raise these). So the stock is **tiled**: split into blocks that
each fit a buffer, and only blocks the tool can touch in a batch are processed. A 10 mm tool's
footprint covers about 31,000 columns at 0.05 mm; a 100 m toolpath sampled every 0.05 mm is
2 million positions; the naive total is about 6 × 10¹⁰ column updates. That is minutes on a few
CPU cores and seconds on a GPU, which matches the order of the vendor claims. Updating by the
swept area of each segment instead of every sample cuts the work several times on both.

### Precision, determinism and trust

This is the part a CAM GPU design has to get right, because a toolpath that is fast but wrong
breaks tools and parts.

- **Single precision is enough for the grid, if coordinates are local (estimate).** A 32-bit
  float has a 24-bit significand: between 512 and 1024 mm, adjacent values are about 0.06 µm
  apart. Stock grids at 5 to 50 µm resolution are far coarser. The danger is subtracting large,
  nearly equal numbers far from the origin, so each tile stores coordinates relative to its own
  origin.
- **WebGPU has no 64-bit floats (fact).** WGSL has `f32` and optional `f16`, and no `f64`. A
  2022 discussion on the W3C GPU mailing list says `f64` might come later as an optional
  extension and that Metal (Apple GPUs) lacks it, so Apple implementations would not offer it.
  Libraries emulate double precision with pairs of floats ("double-single" or `df64`) at several
  times the cost.
- **Consumer GPUs are slow at double precision and have no ECC (fact).** On NVIDIA GeForce
  cards double precision runs at a small fraction (about 1/32 or less) of single precision, for
  example about 1.3 TFLOPS on an RTX 4090 against about 9.7 on an A100; GeForce memory has no
  error correction or detection, while data-centre cards correct single-bit errors and report
  double-bit ones.
- **Use integers where results must be exact (opinion).** Dexel interval ends can be stored as
  32-bit integers in a fixed unit (for example 0.1 µm, covering ±214 mm per tile). WGSL has
  integer atomics (`atomicMin`, `atomicMax`) but no float atomics, so "lowest safe height" and
  "deepest cut" reductions are exact and give the **same answer in any order**. That makes GPU
  results deterministic: the same job gives bit-identical stock on any GPU and on the CPU twin.
- **Floating-point sums are order-dependent (fact).** A parallel sum of floats can differ in the
  last bits from run to run. Force and power totals should use a fixed reduction order (or
  integer fixed-point accumulation) so a regenerated program does not change for no reason.
- **"GPU proposes, CPU certifies" (opinion).** Anything that becomes part of a program (a cutter
  location, a feed override, a collision verdict near the clearance limit) is re-checked on the
  CPU in double precision, only where the GPU result is within a margin of a limit. Most segments
  are far from every limit and need no re-check; the few close ones are cheap. The certificate
  (what was checked, by which method, at what tolerance) goes into the program record, in the
  same spirit as the in-process inspection file's per-part records.

### GPU programming interfaces

| Interface | What it is | Vendors / platforms | Fits SolidCraft's rules? |
|---|---|---|---|
| **WebGPU / WGSL via `wgpu`** | Modern cross-platform GPU API; `wgpu` is its Rust implementation and also runs natively on Vulkan, Metal and DX12 | NVIDIA, AMD, Intel, Apple, mobile; browsers (Chrome and Edge since version 113) | **Yes.** Already used by `crates/gpu`; Rust only; builds for wasm; safe Rust |
| **Vulkan compute** (directly) | Low-level cross-platform API; `wgpu` already sits on it | All desktop vendors except Apple (via a translation layer) | Possible but duplicates `wgpu` and needs `unsafe` |
| **CUDA** | NVIDIA's own platform with the richest libraries (sparse solvers, sorting, BVH builders) | NVIDIA only | Not as the main path: vendor lock-in, no browser, Rust bindings (`cudarc`, Rust-CUDA) are `unsafe` at the edges. **Possible** as an optional backend of a remote worker (open question) |
| **OpenCL** | Older cross-vendor compute standard | Broad but stagnating; Apple deprecated it | No: little reason to start new work on it |
| **SYCL / oneAPI, HIP / ROCm** | C++ single-source GPU programming (Intel; AMD) | Mostly Intel / AMD, with NVIDIA back-ends | No: C++, against "application code is Rust only" |
| **CubeCL** | Rust crate for writing GPU kernels in Rust that compile to several back-ends, including `wgpu`, CUDA and others | Multi-vendor | **Worth watching** (opinion): kernels in Rust instead of WGSL; still pre-1.0 in 2026 |
| **OpenGL compute** | Older graphics API with compute shaders | Broad | No: `wgpu` replaces it. (ModuleWorks says it wrote its GPU simulation natively on OpenGL) |

**Fact, from SolidCraft's code:** `crates/gpu` already evaluates display tessellation plans in a
WGSL compute shader and runs a GPU path tracer with cancellation, admission checks (does the job
fit the device's memory?) and bounded read-back. The `render.backends` command discovers render
devices without opening a window or changing the document. A CAM compute layer can copy all of
these patterns rather than invent new ones.

### Where the GPU can be

| Place | Example | Round trip | Start-up | Cost | Data leaves the computer? |
|---|---|---|---|---|---|
| **This computer's GPU** | Laptop or workstation GPU, integrated GPU, the browser's WebGPU | Microseconds (copy over PCIe) | None | Already paid | No |
| **This computer's CPU** | All cores via `rayon` | None | None | Already paid | No |
| **A GPU on the local network** | A second workstation, a GPU server in the office, an idle CAM seat overnight | About 1 ms on wired LAN (estimate) | Seconds if the worker is running | Already paid; electricity | Stays in the building |
| **A cluster of local GPUs** | Several shop PCs or a rack, each running the worker | As above | As above | As above | Stays in the building |
| **A commodity GPU cloud** | Lambda, RunPod, Vast.ai and similar | Tens of ms (estimate) | Seconds for "serverless" workers with a warm image; minutes for a fresh virtual machine (unverified) | About $1 to $4 per hour for A100 / H100 class, on demand, late 2026 | Yes, to the provider (and, on community marketplaces, to another person's machine) |
| **A hyperscaler** | AWS EC2 G and P instances, Azure N-series, Google Cloud GPUs | Tens of ms | Minutes for a VM (unverified) | Similar or higher; spot prices lower but can be taken away | Yes, to the provider, inside the account's region |
| **A government cloud region** | AWS GovCloud (US) G6 (NVIDIA L4, 24 GB); Azure Government N-series | Tens of ms | Minutes | Usually higher than commercial regions (unverified) | Stays inside the authorised boundary |

## Design for SolidCraft (opinion)

This section is the proposal. It builds on what SolidCraft already does for rendering.

### One compute layer, many devices

1. **Kernels.** Each data-parallel CAM operation (stock update, drop-cutter, engagement, force,
   collision, offset) is one WGSL compute kernel plus a CPU twin in Rust. The CPU twin is the
   reference; tests run both on the same inputs and compare (bit-identical for integer stock,
   within a stated tolerance for float results). This follows SolidCraft's existing pattern:
   `tess.wgsl` is described in the code as "the GPU twin of `TessPlan::evaluate`".
2. **A compute job.** A job is a self-contained, serialisable description: the stock tile(s), the
   tool and holder shapes, the toolpath segments, the cutting data and limits from the physics
   files, the kernel version, and the tolerance. Its identity is the hash of its contents, so a
   result can be cached and reused, and a remote result can be checked to belong to the job that
   was sent.
3. **A device list.** A `compute.backends` command, modelled on `render.backends`, lists the
   devices SolidCraft can use without starting any work: CPU, local GPUs (name, memory, whether
   it supports the needed features), configured network workers, configured cloud endpoints, and
   for each its availability and the data classes it is approved for. Like every SolidCraft
   action it is a command, so agents and the MCP tools can use it, and programmatic calls never
   open dialogs.
4. **A scheduler.** It splits work into jobs (by stock tile, by toolpath chunk, by candidate),
   sends each to the best allowed device, and merges results. Interactive work (preview while
   editing) prefers the local GPU; batch work may fan out. It honours cancellation (the user
   presses Stop) and runs beside the window thread, as SolidCraft's performance rules require
   for any long work.
5. **Fallback.** If no GPU is available, the browser has no WebGPU, the device is lost mid-job
   (`wgpu` reports this as an error, which SolidCraft already logs instead of crashing), or a
   remote worker stops answering, the job runs on the CPU twin. The program is the same either
   way; only the time differs. The user sees which device computed what.

### The compute worker: local network and cloud use the same program

- **What it is.** A headless SolidCraft binary (for example `solidcraft-compute-worker`) built
  from the same crates. It accepts jobs over HTTPS, runs them on its GPU (or CPU), and returns
  results. It holds no designs: it keeps a job only while computing it, unless the operator turns
  on a result cache.
- **On the local network.** Started on any machine with a GPU. SolidCraft finds it by an address
  the user types, or (opt-in) by local-network discovery. Pairing uses a one-time code shown on
  the worker, after which both sides use mutual TLS, so another computer on the network cannot
  pose as a worker or submit jobs.
- **A local GPU network.** Several workers form a pool; the scheduler sends independent jobs to
  each (candidate cut orders, toolpath chunks, slider positions) and gathers results. No cluster
  software is required for this; a shop that already runs one (Slurm, Kubernetes) can start
  workers through it.
- **In the cloud.** The same worker runs in a container image on a rented GPU machine. SolidCraft
  needs only an address and credentials; a small optional helper can start and stop machines on
  a given provider when jobs are queued and stop them after an idle time so billing stops. No
  provider-specific GPU API is involved, so moving between AWS, Azure, Google Cloud, Lambda,
  RunPod or a GovCloud account changes configuration, not code.
- **In the browser.** The web app can use the local GPU through WebGPU, and can reach a worker
  over HTTPS if the worker allows the app's origin. A browser cannot discover workers on the
  local network by itself, so the address is typed or provided by the organisation.

### What a job sends and gets back (estimate)

- **Sent:** the stock tiles the job touches (compressed; tens of MB for a large fine stock, much
  less as a coarse start plus the toolpath that made it, since the worker can rebuild the stock),
  the toolpath segments (a 1-million-segment path is about 24 MB as three 64-bit coordinates per
  point before compression), tool and holder profiles, and limits (kilobytes).
- **Returned:** per-segment results (engagement, force vector, the binding limit and the allowed
  feed), collision events, and optionally the final stock tile. Tens of bytes per segment.
- **Bandwidth:** at a shop uplink of 50 Mbit/s, 10 MB takes under 2 seconds. So for any job that
  takes more than about 30 seconds locally, transfer is not the bottleneck; start-up time and
  queueing are.

### When to send work away (opinion, with a formula)

Send a job to a remote device only when

`T_local > T_upload + T_start + T_queue + T_remote + T_download + T_certify`

and, for paid devices, when the cost `price_per_hour × billed_time` is below what the user
allowed. The scheduler measures each device's speed on small jobs and estimates the rest.

**Worked estimate.** A whole-program physics check that takes 6 minutes on the user's laptop GPU
might take 30 seconds on a data-centre GPU. On a warm cloud worker that is 30 s plus a few
seconds of transfer, billed at about $3 per hour, about 3 cents. On a cold virtual machine add a
few minutes of start-up, and the gain is gone for a single check but not for a batch of 200 cut
orders, which is 200 × 6 min = 20 hours locally against about 25 minutes across ten rented GPUs
(about $12 at $3 per GPU-hour). These numbers are illustrative; the structure (batch work gains,
single interactive checks do not) is the point.

### Keeping controlled data where it belongs

Shops that make defense, aerospace or customer-confidential parts often may not send design data
to arbitrary computers. ITAR (22 CFR 120–130) controls the technical data of defense articles;
DoD contracts require controlled unclassified information (CUI) to be protected; customer NDAs
add their own limits. A part's geometry, its toolpath and its stock model are all technical data
about the part, even if no B-rep is sent. So the design treats every job as carrying the design's
data class.

These rules reuse the model in the
[high-security research](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-2fa4g1/research/security-research-nsnlookupjn-high-security.md)
(markings in the design, an administrator's policy file, endpoint trust levels, an audit log)
rather than inventing a second one. Compute workers are one more kind of endpoint in it.

- **Default: local only.** Remote compute is off until someone turns it on. Under the
  high-security profile's policy file, a worker's address must also be in the network allow list.
- **Markings on the design.** The design's effective marking (the strictest of its own and its
  components', the high-security file's "high-water mark") decides where its jobs may go. A
  marking is lowered only by its own deliberate, logged command, never by a compute setting.
- **Endpoints carry trust levels.** Each configured worker or cloud endpoint has a trust level in
  policy, the same levels the high-security file gives agent endpoints (for example `public`,
  `cui`, `itar`). A local-network server might be `itar`; a GovCloud account `cui` or `itar` if
  the organisation has decided so; a commodity GPU marketplace `public` only. An organisation
  ships this as part of its locked policy.
- **The engine enforces it.** The scheduler refuses, with an explanation, to send a job to an
  endpoint below the design's marking, whatever asked for it (a person, an agent, a plugin). The
  refusal is a normal command error, never a crash.
- **Every send is recorded.** Endpoint, job hash, size and time (not the content) go into the
  high-security file's hash-chained audit log, and each worker address is a row in its egress
  inventory.
- **In transit and at rest.** Mutual TLS in transit; the worker keeps nothing after the job by
  default; cloud images can use encrypted disks.
- **What this does not do.** Software settings do not make a cloud account compliant; that
  depends on the organisation's agreements, the region and who administers it. This feature only
  makes sure SolidCraft never sends data somewhere the organisation has not approved.

The high-security file owns the wider rules (the policy file, markings, air-gapped and
GovCloud-hosted operation, encryption, the audit log). This section is only the compute part. In
an air-gapped shop, the local GPU network is the only remote compute, and the worker needs no
internet access at all.

### How this serves the CAM Physics System

The CAM Physics System's solver loop checks every segment against the minimum of the tool,
machine, workholding and current-stock limits along the force direction, then adjusts feed or
engagement. With GPU compute:

- **Stock update and engagement come together.** The kernel that removes material at a segment
  already knows how much it removed and where, which is the engagement the cutting physics needs.
  One pass gives the new stock and the input to the force model.
- **Force and limits per segment are cheap** once engagement is known, so the whole program can be
  checked at every segment rather than at sampled points.
- **Feed changes are iterative.** A lower feed at one segment does not change the stock (the
  geometry is the same) but can change chip thickness and so force; a changed **path** does
  change later engagement. The loop runs the cheap force check on the GPU many times and reruns
  the stock update only where the path changed.
- **The risk slider can be shown live.** Precomputing the result for a range of slider positions
  (each independent) lets the user move the slider and see cycle time and the binding limit
  change immediately.
- **The stock integrity planner can search.** Each candidate cut order is a full independent run,
  so a local GPU network or rented GPUs can evaluate many at once and return a ranked list.
- **Stock stiffness updates.** Beam and plate formulas on recognised features are cheap on the
  CPU. When an FE model replaces them (the FEA request #1620), its sparse solves are the work a
  GPU worker can take, at the 2 to 5 times speed-ups vendors report for direct sparse solvers.
- **Machine and warm-up gates are unaffected.** Compute speed never changes a limit. The
  warm-up gate and every other hard rule in the physics files still apply to the final program.

### A phased plan

1. **CPU reference first.** Write each CAM kernel as a clear, tested, multi-threaded CPU
   function, deterministic, with integer stock. Measure. (The SolidCraft rule not to rewrite the
   B-rep kernel for speed does not apply here: CAM stock is its own data structure.)
2. **Local GPU.** Port the hottest kernels (stock update with engagement, drop-cutter, collision
   against an SDF) to WGSL in a CAM compute module that follows `crates/gpu`'s patterns: admission
   check, cancellation, bounded read-back, GPU-versus-CPU tests. Ship with automatic fallback.
3. **Device list and jobs.** Add `compute.backends`, the hashed job format, and the scheduler,
   still local only.
4. **Local network workers.** Ship the headless worker, pairing and mutual TLS, data classes and
   endpoint approvals, and the send log. Turn on fan-out of batch jobs.
5. **Cloud workers.** Publish a container image of the worker; document starting it on common
   providers, including a GovCloud region; optional start/stop helper with a spending cap.
6. **Search at scale.** Use the pool for cut-order search, slider sweeps and, when it exists,
   FE-based stock stiffness.

## Accuracy

- **Vendor speed-ups are marketing numbers measured by the vendor.** ModuleWorks' "up to 10×"
  is on its own benchmarks in Aachen with a Ryzen 7 3700X CPU and an RTX 4070 GPU, with the
  biggest gains on simulations of over a million toolpath points; MachineWorks' "10 to 25×"
  depends on the GPU and, in its words, is "subject to the constraints of accuracy". Neither
  states its accuracy or its CPU baseline's thread count in the pages reviewed.
- **Paper speed-ups depend on the baseline.** The Georgia Tech offsetting paper reports about
  50× over an eight-core CPU at 512³ but about 4.5× at 1024³, and 254× against a single core: the
  same method looks very different against different baselines and sizes. The KU Leuven dexel
  paper reports about 5× and excludes pre-processing from the timing.
- **`raster-path` numbers** (about 0.8 s against 20 to 80 s for 48,000 points; about 2 s
  against about 280 s for 1.2 million points) are from its own README, on one example model,
  against its own WebAssembly CPU version, not against an optimised native multi-threaded CPU
  implementation.
- **The estimates in this file** (memory per stock, work per toolpath, transfer times, cost per
  job) are order-of-magnitude arithmetic, not measurements. SolidCraft's performance rules ask for
  measured medians on real cases before claiming any speed-up; these numbers should be replaced by
  such measurements.
- **Accuracy of GPU stock simulation** is set by the grid resolution, not by the GPU: a dexel or
  voxel model with 0.05 mm cells cannot show a 0.01 mm step whatever device computes it. Adaptive
  grids (finer near the cut) and the CPU certification step address this.
- **Cloud prices** are from a comparison article dated August 2026 and updated October 2026; the
  article itself warns prices change. Start-up times for cloud machines were not confirmed
  against provider documentation (unverified).

## Programs

### Commercial CAM components and CAM programs (public pages only)

- **ModuleWorks** (component supplier to many CAM programs): GPU-accelerated material removal
  simulation announced August 2024 for testing with release 2024.12; "up to 10×" on a mid-range
  NVIDIA card, "more than 20×" on high-end cards; the time-critical part was rewritten natively on
  the GPU using OpenGL rather than ported; an extra licence fee and a suitable GPU are required.
  Simulation uses a dexel model.
- **MachineWorks** (simulation component supplier): "MachineWorks GPU" first released with
  MachineWorks 8.7 (July 2025); runs material removal and collision detection on the GPU; large
  3- and 5-axis jobs "in seconds", mid-sized under a second; earlier press said 10 to 25× on the
  stock model, benchmarked on NVIDIA RTX 2000 Ada and GeForce RTX.
- **TopSolid** (Missler Software): announced it would ship MachineWorks' GPU-based CNC
  simulation.
- **What the public pages do not show:** none of the pages reviewed describes a CAM program that
  sends toolpath or simulation work to a shop's other GPUs or to rented cloud GPUs as an
  integral feature, or one that enforces data classes on such remote compute. Some CAM programs
  offer cloud computation of toolpaths as part of a cloud product (unverified; not checked
  against vendor pages for this file).

### Open source and research

- **`raster-path`** (GridSpace, MIT): browser-based 3-axis toolpath generation with WebGPU
  compute shaders: planar raster (rasterise tool and terrain onto a shared XY grid, find the
  lowest safe height at each position), radial (for cylindrical parts) and tracing along given
  polylines. Shows that WebGPU compute is enough for production-style drop-cutter work in a
  browser. (Only its README was read.)
- **Kukreja, Dhanda, Pande (IIT Kanpur), J. Manuf. Sci. Eng. 144(1) 2022:** adaptive iso-planar
  3-axis toolpaths on a voxel part model on the GPU: gouge-free cutter locations from the voxel
  model, then side and forward steps; gouge detection by image processing of "shadow" voxels.
- **Georgia Tech (Kurfess group), J. Manuf. Sci. Eng. 140(4) 2018 (authors not confirmed on the
  page read):** voxel offsetting as morphological dilation and erosion with a CUDA convolution
  filter on a "hybrid dynamic tree" (dense top grid, octrees, dense leaf grids), up to 2048³;
  validated by machining on turning, 3-axis and mill-turn machines; large offsets split into
  smaller ones trade accuracy for speed.
- **Ozcan, Kushwaha, Qian, Reynaerts (KU Leuven), euspen 2024:** adaptive tri-dexel material
  removal simulation, ray–triangle and ray–cylinder intersection on the GPU (MATLAB with CUDA on
  an RTX 3070), about 5× faster at the highest resolution.
- **GPU collision with signed distance fields** is mature in robotics and physics simulation
  (for example RWTH Aachen's GPU SDF collision for multibody dynamics, and the Eurographics
  "Intersection Distance Field Collision for GPU" paper). The same ideas apply to holder and
  machine-component clearance checks.
- **GPU finite elements:** COMSOL reports direct sparse solvers on NVIDIA's cuDSS at up to 5× on
  multiphysics models and 2× on a structural model (RTX 5000 Ada against a workstation CPU). This
  is the relevant ceiling for stock stiffness by FE.

### Compute providers (public pages)

- **AWS:** G6 instances (NVIDIA L4, 24 GB per GPU, up to 8 GPUs) available in GovCloud (US-West)
  from November 2024 and in GovCloud (US-East) from August 2025; many more GPU types in commercial
  regions.
- **Azure Government:** NC-series GPU virtual machines announced as available in Azure Government
  in August 2017; current N-series availability there was not checked (unverified).
- **Lambda and RunPod** (October 2026 article): H100 SXM about $2.99 per hour on RunPod's Secure
  Cloud and $3.99 to $4.29 on Lambda; A100 about $1.19 to $1.99. Lambda bills per minute and
  charges no egress; RunPod's serverless workers bill per second of active time. "Community"
  tiers run on machines owned by third parties, which matters for controlled data.

## Complaints and requests

Paraphrased from the sources above and from general knowledge of shop practice; forum threads
were not separately collected for this topic (TODO).

- **Waiting.** Simulation and verification of long 5-axis programs, and fine 3D finishing
  toolpaths, take minutes to hours; programmers skip full simulation to save time, which is when
  crashes and gouges get through. (General, unverified for any specific program.)
- **Idle hardware.** Workstations bought with a powerful GPU for display use it only to draw the
  screen while the CPU does the CAM work. (General.)
- **Paying twice.** GPU simulation from at least one component vendor needs an extra licence on
  top of the CAM licence and a suitable GPU (fact, ModuleWorks per Digital Engineering).
- **Vendor lock-in.** GPU features that require a specific vendor's GPU leave Apple, AMD and Intel
  users with the slow path. (Opinion; whether specific CAM GPU features are NVIDIA-only was not
  confirmed.)
- **"Is it the same answer?"** Users and vendors both worry that a faster simulation is a less
  accurate one; MachineWorks' own phrasing ("subject to the constraints of accuracy") shows the
  trade-off is real.
- **Cloud and confidentiality.** Shops with defense or customer-confidential work cannot use
  cloud computation that sends design data outside an approved boundary. (General; see the
  high-security topic.)

> **[TODO Jason: personal experience]** Add complaints you have heard in shops or met yourself.

## What users need

1. **It just gets faster.** With a capable GPU, CAM uses it automatically; with none, nothing
   breaks and the user is not asked anything.
2. **The same program either way.** Whether computed on a laptop, a GPU server or a cloud GPU,
   the G-code and the physics results are the same, and the program record says how they were
   checked.
3. **Use what the shop already owns.** Point SolidCraft at another PC's GPU, or a GPU server,
   with a pairing code, no cluster expertise needed.
4. **Rent only when it pays.** An estimate of time and cost before a remote batch runs, a
   spending cap, and machines that stop when idle.
5. **Never send controlled data to the wrong place.** Local only by default; data classes on the
   design; endpoints approved per class; a refusal with a reason; an audit log.
6. **Any GPU.** NVIDIA, AMD, Intel, Apple and the browser, because the kernels are written once
   in WebGPU.
7. **Stop means stop.** Cancelling a long check stops local and remote work and leaves the design
   unchanged.
8. **See where time goes.** Which device computed which part, how long it took, and why work
   was or was not sent away.

### Interface with the other files

| File | What it gives this one | What this one gives it |
|---|---|---|
| CAM Physics System | The solver loop: per-segment minimum over all limits; the slider; the phased plan | Fast per-segment evaluation; fan-out for slider sweeps; "GPU proposes, CPU certifies" |
| Cutting physics | Force model per tooth angle; cutting coefficients | Engagement per segment from the stock update; force kernels |
| Stock integrity | Candidate cut orders; stiffness and mode models | Parallel evaluation of candidates; worker capacity for FE solves |
| Workholding | Fixture geometry and holding limits per direction | Collision and clearance checks against fixtures |
| Machine library | Axis limits, torque and power curves, machine geometry | Machine collision checks; per-segment torque and power checks |
| Tool library | Tool and holder geometry | Tool and holder shapes for collision and stock update |
| In-process inspection | Measured results that recalibrate models | A record of how each program was computed and certified |
| [High-security](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-2fa4g1/research/security-research-nsnlookupjn-high-security.md) | Markings, policy file, endpoint trust levels, audit log, air-gapped and GovCloud operation | Compute workers as endpoints: trust levels, refusal, audit entries, egress inventory rows |

## Glossary

- **GPU:** graphics processing unit; a processor with thousands of simple cores, used here for
  general computation ("GPU compute" or "GPGPU").
- **Compute shader / kernel:** a small program run on the GPU once per item, in many threads at
  once.
- **WebGPU / WGSL:** the modern web and cross-platform GPU API, and its shading language.
- **`wgpu`:** the Rust implementation of WebGPU that SolidCraft already uses; runs on Vulkan,
  Metal, DX12 and in browsers.
- **CUDA:** NVIDIA's proprietary GPU computing platform.
- **Dexel / tri-dexel:** a stock model of columns, each holding the solid intervals along it; a
  tri-dexel model has columns in three directions.
- **Z-map / height map:** a grid of heights; a dexel model with one interval per column.
- **Voxel:** a cell of a 3D grid, filled or empty.
- **SDF (signed distance field):** a grid of distances to the nearest surface, negative inside.
- **Drop-cutter:** finding, for an XY position, the lowest tool height that does not cut into
  the part.
- **Engagement:** how much of the tool is in the material at a step (radial and axial depth,
  engagement angle).
- **Determinism:** the same input always giving bit-identical output.
- **ECC:** error-correcting code memory, which corrects or reports memory bit errors.
- **f32 / f64:** 32- and 64-bit floating-point numbers (single and double precision).
- **Worker:** the headless SolidCraft program that runs compute jobs for another SolidCraft.
- **GPU-as-a-service:** GPUs rented by the hour or second from a cloud provider.
- **GovCloud:** AWS's isolated US regions for regulated government and defense workloads; Azure
  Government is Microsoft's equivalent.
- **CUI / ITAR:** controlled unclassified information; the US International Traffic in Arms
  Regulations.
- **Mutual TLS:** encrypted connection where both ends prove their identity with certificates.

## Open questions

1. Should the first GPU kernels target a Z-map (3-axis only, simplest) or go straight to
   tri-dexel so 5-axis and undercuts work from the start?
2. Is single precision with local tile origins and integer interval ends accurate enough for all
   stock and engagement work, with double precision only in CPU certification? What tolerance
   should a certificate state?
3. Would maintainers accept an optional CUDA back-end in the remote worker only (never in the
   app), for example to use NVIDIA's sparse solvers for FE, given that those libraries are
   proprietary? Or should everything stay on `wgpu` / CubeCL?
4. What wire format and protocol should jobs use: the design format's own serialisation, a
   separate versioned job schema (SolidCraft commands already have JSON input and output
   schemas), or a binary format for large stock tiles?
5. Should the worker be part of `solidcraft-cli` (one binary, a `serve-compute` mode) or a
   separate app?
6. Should a compute job ever be allowed to a lower-trust endpoint if it carries only
   discretised stock and tool envelopes, not the design? (Proposed answer: no, it is still
   technical data about the part; the high-security file's marking applies unchanged.)
7. Is local-network discovery acceptable in a CAD app by default, or should workers always be
   added by address?
8. Should SolidCraft ship a start/stop helper for specific cloud providers at all, or only
   document how to run the worker image, leaving provider automation to the user?
9. How should cost be shown and capped for rented compute, and who sets the cap in an
   organisation?

## Sources

All accessed 2026-10-26.

### Vendor announcements and trade press

- ModuleWorks, "Precision Meets Performance With GPU-Accelerated Simulation" (August 2024).
  [Page](https://www.moduleworks.com/precision-meets-performance-with-gpu-accelerated-simulation/)
- Digital Engineering, "CAM Embraces GPU Acceleration" (September 2024, sponsored).
  [Page](https://www.digitalengineering247.com/article/cam-embraces-gpu-acceleration)
- Digital Engineering, "MachineWorks GPU 1.0 Now Available" (July 2025).
  [Page](https://www.digitalengineering247.com/article/machineworks-gpu-1.0-now-available)
- MachineWorks, "TopSolid to release MachineWorks GPU-based CNC simulation".
  [Page](https://www.machineworks.com/news/topsolid-to-release-machineworks-gpu-based-cnc-simulation/)
- COMSOL, "GPU Acceleration". [Page](https://www.comsol.com/technologies/gpu-acceleration)

### Papers

- Kukreja A., Dhanda M., Pande S. S., "Voxel-based adaptive toolpath planning using graphics
  processing unit for freeform surface machining", J. Manuf. Sci. Eng. 144(1) 011013 (2022),
  doi:10.1115/1.4051535. [Record](https://researchportal.bath.ac.uk/en/publications/voxel-based-adaptive-toolpath-planning-using-graphics-processing-/)
- "A Graphics Processor Unit Accelerated Freeform Surface Offsetting Approach for High
  Resolution Subtractive Three-Dimensional Printing (Machining)", J. Manuf. Sci. Eng. 140(4)
  041012 (2018), Georgia Tech. [Abstract](https://asmedc.silverchair.com/manufacturingscience/article/140/4/041012/380522/A-Graphics-Processor-Unit-Accelerated-Freeform)
- Ozcan Y., Kushwaha S., Qian J., Reynaerts D., "Adaptive dexelisation approach for material
  removal simulation in milling", euspen 24th International Conference (2024).
  [PDF](https://euspen.eu/knowledge-base/ICE24301.pdf)
- "Intersection Distance Field Collision for GPU", Pacific Graphics short papers (2022).
  [Record](https://diglib.eg.org/handle/10.2312/pg20221242)
- RWTH Aachen MMI, "GPU-Accelerated Signed Distance Field Collision Detection for Multibody
  Dynamics Simulation". [Page](https://www.mmi.rwth-aachen.de/?p=17969)

### Software

- GridSpace, `raster-path` (MIT), README. [Repository](https://github.com/GridSpace/raster-path)
- CubeCL crate documentation. [docs.rs](https://docs.rs/crate/cubecl/0.11.0-pre.3)
- SolidCraft `crates/gpu` (WGSL tessellation and path tracing compute) and `docs/render-backends.md`
  (`render.backends` discovery), in this repository.

### Standards and hardware

- W3C public-gpu mailing list, "Double precision floats (IEEE-754 binary64)" thread, April 2022.
  [Message](https://lists.w3.org/Archives/Public/public-gpu/2022Apr/0006.html)
- Microway, "Comparison of NVIDIA GeForce GPUs and NVIDIA Tesla GPUs".
  [Page](https://www.microway.com/knowledge-center-articles/comparison-of-nvidia-geforce-gpus-and-nvidia-tesla-gpus/)

### Compute providers

- AWS, "Amazon EC2 G6 instances now available in the AWS GovCloud (US-West) Region" (November
  2024). [Page](https://aws.amazon.com/about-aws/whats-new/2024/11/amazon-ec2-g6-instances-aws-govcloud-us-west-region/)
- AWS, "Amazon EC2 G6 now available in the AWS GovCloud (US-East) Region" (August 2025).
  [Page](https://aws.amazon.com/about-aws/whats-new/2025/08/amazon-ec2-g6-now-available-govcloud-east-region/)
- Microsoft Azure Government blog, "Azure NC virtual machines availability (GPU compute)"
  (August 2017). [Page](https://devblogs.microsoft.com/azuregov/azure-nc-virtual-machines-availability-gpu-compute/)
- Spheron, "RunPod vs Lambda Labs 2026" (August 2026, updated October 2026).
  [Page](https://www.spheron.network/blog/runpod-vs-lambda-labs-2026/)

### Related research in this set

- Jason, high-security use of SolidCraft (markings, policy, audit log, GovCloud).
  [File](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-2fa4g1/research/security-research-nsnlookupjn-high-security.md)
