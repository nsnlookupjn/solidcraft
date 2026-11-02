# Technical research: closed-loop physics-driven CAM architecture for SolidCraft

**Target issue:** Research Request for Plugin: CAM (#1621)
**Author:** Jason ([@nsnlookupjn](https://github.com/nsnlookupjn)), with an architecture sketch from a computer engineer
**Date:** 27 October 2026
**Status:** incomplete, ongoing research; feedback and contributions welcome
**Licence:** MIT OR Apache-2.0
**Companion:** [CAM Physics System](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-vgqo3g/research/cam-research-nsnlookupjn-physics-system.md) (the full system, its limits and the request to the maintainers)

> This is an evolving architectural proposal. The equations, data models and integration
> ideas are open for discussion. Units are millimetres, newtons and radians, as in SolidCraft.

---

## 1. Architecture overview

The aim is to stop tool overloads and to evaluate cutting physics at every small step of every
toolpath, fast enough to be part of normal toolpath generation and simulation. The design keeps
the exact geometry and the fast physics evaluation separate:

```text
             design (B-rep from SolidCraft's kernel, crates/kernel)
                    │
        ┌───────────┴──────────────────────────────┐
        ▼                                          ▼
  exact geometry (CPU)                      physics core (CPU reference, wasm-safe)
  • feature tags                            • tool–workpiece engagement on a stock proxy
  • final boundaries, finishing             • force vectors per flute and step
  • G-code post-processing                  • feed scaling against every limit
        │                                          │  optional accelerator (layer 5):
        │                                          │  same maths on the GPU via wgpu
        └───────────────┬──────────────────────────┘
                        ▼
          manufacturing state graph (stock 0: billet → stock N: part)
```

### 1.1 Two representations

- **Exact B-rep** (SolidCraft's kernel, `crates/kernel`): the source of truth for design
  geometry, feature queries and finishing operations.
- **Stock proxy** (sparse voxels, dexels or a narrow-band signed distance field, SDF): the
  in-process stock and each intermediate state. Engagement and chip cross-sections are taken
  from the proxy at every step along a toolpath.

### 1.2 CPU reference first, GPU as an accelerator

SolidCraft's layering rules put `wgpu` only in layer 5 (`crates/gpu`, the UI) and require
everything below the apps to build for WebAssembly. So:

- the **physics core** (engagement, forces, limits, feed scaling) is plain Rust in a crate below
  layer 5, with no GPU dependency; it defines the correct answers and carries the tests
  (analytic reference values, as in SolidCraft's oracle tests);
- a **GPU path** in layer 5 runs the same maths in compute shaders for speed, behind a trait the
  core defines, and must give the same results as the CPU path within a stated tolerance;
- headless use (CLI, scripts, MCP, the browser build) always works on the CPU path.

---

## 2. Cutting force model

The cutter is divided into `N_z` axial disks of height `dz` (mm) (Altintas, 2012). For a flute
at immersion angle `φ` (rad) with feed per tooth `f_z` (mm), the uncut chip thickness is

```text
h(φ) = f_z · sin(φ)          (circular-path approximation; 0 outside the engagement arc)
```

Two standard models give the force on one disk of one flute. Either can be used; the tool
library stores the coefficients for the one it has data for.

**Linear edge-force (mechanistic) model** (Altintas, 2012): six coefficients per tool and
material pair, calibrated from cutting tests.

```text
dF_t = (K_tc · h + K_te) · dz
dF_r = (K_rc · h + K_re) · dz
dF_a = (K_ac · h + K_ae) · dz
```

`K_tc, K_rc, K_ac` (N/mm²) are cutting coefficients and `K_te, K_re, K_ae` (N/mm) are edge
coefficients.

**Kienzle model** (Kienzle, 1952): specific cutting force falls with chip thickness.

```text
dF_t = k_c1.1 · h^(1 − m_c) · dz        (h in mm, k_c1.1 in N/mm² at h = 1 mm)
```

Kienzle gives the tangential (cutting) force only; the radial and axial components are
estimated as ratios of it until calibrated values exist, and are marked as estimated.

### 2.1 From flute forces to the machine frame

Each disk's `(dF_t, dF_r, dF_a)` is rotated by its immersion angle into the tool frame and
summed over all disks and all engaged flutes, then transformed into machine coordinates
(ISO 841 axes) using the tool orientation. This gives the resultant force vector
`F = (F_x, F_y, F_z)`, the spindle torque `M_c = Σ dF_t · D/2` and the power
`P_c = M_c · 2π n / 60` for spindle speed `n` (rpm). The sum is a parallel reduction, which
is what the optional GPU path accelerates.

### 2.2 Feed scaling against every limit

The allowed load at a step is the smallest of all limits (tool and holder strength, spindle
torque and power curves at that speed, machine axis thrust, workholding capacity in the
direction of `F`, stock stiffness), each with the safety factor set by the
conservative-to-aggressive slider. With Kienzle, force grows with chip thickness as
`h^(1 − m_c)`, so the feed that brings the load to the limit is

```text
f_z,new = f_z · (F_allow / |F|)^(1 / (1 − m_c))
```

capped by the hard limits (maximum feed, minimum chip thickness, machine maxima). With the
linear model the same target is solved directly from the linear relation. Feed changes between
segments are rate-limited so the machine sees a steady load. The goal is steady maximum load:
raise feed wherever no limit binds, and remove short overload spikes at corners and entries.

---

## 3. Data structures for process planning

CAM data separates manufacturing intent from path geometry.

### 3.1 Feature tags

Design features carry manufacturing meaning, linked to SolidCraft's parametric timeline, so a
change in the design (a pocket depth, a wall draft) updates the operations that depend on it.
A sketch in Rust (illustrative; names and fields are open for discussion):

```rust
/// Manufacturing meaning of a design feature. Lengths in mm, angles in rad, roughness in µm.
pub enum ManufacturingFeature {
    Pocket { depth: f64, floor_radius: f64, draft: f64 },
    Hole { diameter: f64, depth: f64, thread_pitch: Option<f64> },
    Wall { height: f64, tolerance: f64, roughness_ra: f64 },
    Boss { height: f64, corner_radius: f64 },
}
```

Values from design files and libraries are treated as untrusted input: finite checks, ranges
and caps, errors returned as `Result`, never a panic.

### 3.2 Manufacturing state graph

Process planning works backward from the finished part to the stock and records the states in
a directed acyclic graph:

- **nodes**: in-process stock states (stock proxy snapshots);
- **edges**: operations (for example adaptive roughing, wall finishing) with tool, feeds per
  segment and predicted peak load.

The graph lets the planner compare cut orders (keeping the stock stiff while forces are high),
lets the simulation resume from any state, and lets a design change re-run only the operations
downstream of it.

---

## 4. Integration with SolidCraft

### 4.1 Crates and layers

- A CAM physics crate below layer 5: no UI or GPU dependency, builds for wasm, no panics.
- An optional GPU accelerator in layer 5 using `wgpu`, implementing the core's evaluation trait.
- Geometry queries through SolidCraft's kernel crate; the stock proxy is CAM's own.

### 4.2 Commands and automation

Everything is a SolidCraft command (with its own id and input/output schemas), so the toolbar,
scripts, CLI, control channel and the MCP server reach the same physics. Agents can evaluate
engagement, compare tools and simulate forces headlessly, without the UI.

### 4.3 Closed-loop data contracts

Defined from the start, even before anything produces the data, so shop-floor feedback can be
added without reworking the core:

- **Calibration profiles in:** cutting coefficients per tool and material (`K_tc … K_ae` or
  `k_c1.1, m_c`), machine stiffness per axis, measured spindle torque and power curves, each with
  its source and date. These update the shop's own library entries, never the shared catalogue
  silently.
- **Machine data in:** spindle load, torque and vibration during the cut, aligned with the
  G-code line and the cycle timeline (Self-tuning CAM Operations, through SolidCraft Edge).
- **Inspection results in:** probe or CMM deviations per feature (for example
  `ΔP = (dx, dy, dz)` at measured points, ideally exchanged as QIF), with measurement
  uncertainty and method. Where the predicted tool or wall deflection explains a deviation, the
  model is corrected first; the remainder adjusts finishing offsets or deflection compensation
  for the next part, within limits and logged, so the process stays inside tolerance across a
  production run.

---

## 5. Feedback wanted

- Stock proxy choice: sparse voxel octree, dexels or narrow-band SDF, for accuracy and speed on
  both CPU and GPU.
- Calibration of the force models for multi-flute carbide end mills.
- Schemas for importing calibration profiles and inspection deviation maps.

Comments and contributions are welcome on #1621.

---

## 6. Export control, sanctions and government use

*Added 2026-11-02, matching the section in every other research file in this set (see the
[government use research](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-qk5riz/research/gov-research-nsnlookupjn-government-use.md)).
Opinion and policy proposals, not legal advice.*

**Our three rules:**

- **No one goes to jail.** Our team has no in-house counsel. So wherever a rule is unclear we
  take the most conservative reading and refuse rather than warn. Questions that need an outside
  export-control lawyer are marked as such, and free official help (BIS, DDTC, OFAC) is the
  first call.
- **Help users comply.** SolidCraft stops and explains in plain language, using rule packs for
  each jurisdiction that the administrator picks. It flags conflicts between countries' laws,
  never resolves them, and never gives legal advice.
- **Prevention first.** Nothing in this file that is hosted, shared, paid or networked ships
  until its compliance gates are in place and checked: region blocks, screening, ITAR flag
  handling, government-mode behaviour, and an outside lawyer's sign-off for any hosted or paid
  service.

**What in this file crosses a border:** the closed-loop data contracts (calibration profiles,
machine data and inspection deviations flowing back to a library), and any remote GPU
accelerator that receives geometry.

- **Hosted parts block sanctioned regions and screen people.** Any service this file implies
  that the project would run blocks comprehensively sanctioned regions, plus Russia and Belarus,
  and screens accounts, payers and contributors against the US Consolidated Screening List and
  the UK, EU and UN lists.
- **No support there.** No support, consulting or custom work for users in sanctioned regions
  or for listed parties.
- **Government mode.** At the Controlled level (CUI and allied equivalents) calibration and
  feedback data stay in the organisation's own library; a remote GPU must be inside the
  boundary; a design flagged Export controlled (ITAR/EAR or national equivalent) never leaves
  through any of these paths.
- **Only generic data is shared.** Calibration coefficients may go to a shared library only
  through the field allow-list in the high-security research; nothing derived from a controlled
  design (geometry, toolpaths, stock states, inspection deviations) is shared.
- **Code stays public.** Every build the project ships is public, which keeps it "published"
  software outside the EAR.
- **Checked for controlled data (2026-11-02):** this file contains no controlled technical data.
  Its equations are from public textbooks and its examples are generic.

---

## 7. References

1. Altintas, Y. (2012). *Manufacturing Automation: Metal Cutting Mechanics, Machine Tool
   Vibrations, and CNC Design* (2nd ed.). Cambridge University Press.
2. Kienzle, O. (1952). Die Bestimmung von Kräften und Leistungen an spanenden Werkzeugen und
   Werkzeugmaschinen. *VDI-Zeitschrift* 94, 299–305.
3. ISO 841:2001. Industrial automation systems and integration — Numerical control of
   machines — Coordinate system and motion nomenclature.
4. Museth, K. (2013). VDB: High-resolution sparse volumes with dynamic topology. *ACM
   Transactions on Graphics* 32(3), article 27. https://doi.org/10.1145/2487228.2487235
5. ISO 23952:2020. Automation systems and integration — Quality information framework (QIF).
