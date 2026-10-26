# CAM research: the CAM Physics System, or "physics from the ground up"

Research for [#1621](https://github.com/tangency-labs/solidcraft/issues/1621) (Research Request
for Plugin: CAM). This is the **summary topic** for a set of five research files that together
describe machining as **one physical system**. It does not repeat their physics; it shows how
they fit together, what they share, and what SolidCraft's CAM would need to be built on so that
all of it can work. It ends with a request: **make this physical system the cradle of
SolidCraft's CAM**, so that every toolpath is computed from physics from the first version on,
instead of having physics added later on top of preset feeds and speeds.

The five topics:

| # | Topic | File |
|---|---|---|
| 1 | Shared cloud tool library, cutting data and tool tags | [cam-research-nsnlookupjn-tool-library.md](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-tool-library/research/cam-research-nsnlookupjn-tool-library.md) |
| 2 | Cutting tool physics and the conservative-to-aggressive slider | [cam-research-nsnlookupjn-cutting-physics.md](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-cutting-physics/research/cam-research-nsnlookupjn-cutting-physics.md) |
| 3 | Workholding physics, workholding library and the force-vector negotiation | [cam-research-nsnlookupjn-workholding.md](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-workholding/research/cam-research-nsnlookupjn-workholding.md) |
| 4 | Toolpath optimisation by the structural integrity of the stock during the cycle | [cam-research-nsnlookupjn-stock-integrity.md](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-stock-integrity/research/cam-research-nsnlookupjn-stock-integrity.md) |
| 5 | Machine library: axis stiffness and available force, the machine model | [cam-research-nsnlookupjn-machine-library.md](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-machine-library/research/cam-research-nsnlookupjn-machine-library.md) |

A sixth file closes the loop on the shop floor:
[part certification by in-process inspection](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-y2w06n/research/cam-research-nsnlookupjn-in-process-inspection.md)
(probing, gauging, per-part records). It is not part of the force system but is how the system's
predictions get checked against real parts.

**What lives where.** The load-path table, the series compliance chain and the agreed
interfaces between the five topics are kept in one place, the workholding file's section
["The complete physical system"](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-workholding/research/cam-research-nsnlookupjn-workholding.md#the-complete-physical-system).
Each topic file owns its own physics and its side of each interface. This file owns only the
system-level parts: the shared setup, the solver loop that combines every limit, the one slider
across the whole chain, the closed loop with measurement, the phased plan, and the request.

Licence: MIT OR Apache-2.0, the same as SolidCraft (see the Licence section of CONTRIBUTING.md).

## About this research

- **Author:** Jason ([@nsnlookupjn](https://github.com/nsnlookupjn)). The system idea, the
  real-time force-vector negotiation, the stock integrity topic, the machine library and the
  request to make physics the cradle of the CAM are Jason's; source research and writing done
  with an AI assistant (Claude) in October 2026.
- **How it was produced:** a synthesis of the five topic files above, which cite their public
  sources. Claims new to this file cite their own sources below. No commercial CAD/CAM program
  was driven or scripted, no GPL/LGPL/AGPL source was read, and no text is copied beyond short
  quotes.
- **Units:** millimetres, newtons, newton-metres, watts, hertz, radians; stiffness in N/µm.
- **Marking:** "(unverified)" marks a claim not confirmed against a source. **Fact**,
  **opinion** and **Jason's idea** are labelled where they mix. The architecture in "What users
  need" is opinion throughout: a proposal for discussion, not a decision.

> **[TODO Jason: personal experience]** One or two jobs where the tool, the hold, the stock and
> the machine interacted: for example a part that was fine in roughing and moved or chattered in
> finishing once the walls were thin, or a cut that was safe on one machine and not on another.
> Label this as personal experience.

## Summary

1. **A cut is one load path, not separate features.** Force runs
   **spindle → holder → tool → chip → stock → workholding → table → machine axes**. At every
   instant the weakest link sets how hard the tool may work, and stiffness adds up as a series
   chain (`1/k_total = Σ 1/k_link`, per direction). Today's CAM programs model the tool well,
   the machine as kinematics, the fixture as geometry to avoid and the stock as geometry to
   remove; in the public sources reviewed none treats all links as one structure (details in
   each topic file).
2. **Five topics, one model.** The tool library (1) and the machine library (5) and the
   workholding library (3) supply the **parts** of the load path from shared, collaborative
   catalogues. Cutting physics (2) turns tool, material and engagement into a **force vector**
   at each step. Stock integrity (4) computes how stiff and strong the **in-process stock** is as
   material is removed. All five read and write one **setup**: one machine, one or more holds,
   the stock, and the tools.
3. **The negotiation runs along the toolpath (Jason's idea).** As the cutter moves through
   three dimensions it pushes and pulls the part in changing directions. The workholding is
   strong in some directions and weak in others; the machine is stiffer in some axes than others;
   the stock weakens as it is cut. So the allowed tool load is not one number per operation but a
   function of **direction and time in the cycle**, and the feed (and where possible the
   direction of cut) is adjusted at each step to keep the load at the best value the weakest link
   allows in that direction.
4. **Sequence is part of the physics.** As stock is machined away it becomes weaker and can
   deform. Planning the **order of cuts** so the stock stays stiff while forces are high, and
   balancing stock integrity against tool force and holding force, is part of the same
   optimisation (Jason's idea, topic 4).
5. **One slider for the whole chain.** Jason's conservative ("I don't trust this") to aggressive
   ("I don't care if I break a tool or two") slider moves the safety factors on **every** link
   at once, never below hard floors, and missing data pulls a link toward conservative.
6. **Every result names its limit.** The system's most useful output for a novice is not the feed
   but the sentence beside it: "limited by the vise: the part would slip along the jaws above
   this feed". That puts an experienced machinist's knowledge into the program.
7. **The simulation enforces the physics (Jason's point).** The physics model is built into
   the toolpath simulation, so the same rules are followed through the whole program before it
   reaches the machine. A novice's toolpath that would break a tool, move the part, stall the
   spindle or overload the machine is caught in simulation, shown at the move where it happens,
   and blocked from posting until it is fixed or deliberately accepted.
8. **The request: physics as the cradle.** The cheapest moment to build this is before the CAM
   exists. If SolidCraft's CAM starts with a setup that carries physical data, toolpath
   generators that report engagement per segment, and one limits engine, then each topic can
   land as an increment. If CAM starts from presets, the same work later means reworking every
   generator and file format. The concrete asks are in "The request to the maintainers" below.

## Fundamentals

### The load path and the compliance chain

Fact (mechanics): a milling cut is a closed force loop. The tool pushes on the stock with the
cutting force `F`; the stock passes that force to the workholding, the workholding to the table,
the table through the axes and the frame back to the spindle, which pushes on the tool. Every
element in that loop deflects, and in a series loop the deflections add:

`δ = F / k_total`, with `1/k_total = 1/k_spindle+holder+tool + 1/k_machine + 1/k_fixture + 1/k_stock`

evaluated **per direction**, because each link is stiffer in some directions than others.
Similarly each link has a **strength** limit (the force or moment it can take before it slips,
yields, stalls or breaks), and the loop as a whole can only take the **smallest** of them.

The table of links, what each supplies and the agreed interface quantities is in the
workholding file, section
["The complete physical system"](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-workholding/research/cam-research-nsnlookupjn-workholding.md#the-complete-physical-system),
with the cutting ↔ workholding interface in the section that follows it there and in the cutting
physics file's "Outputs for workholding".

### Three kinds of limit

Each link limits the cut in three ways, and the system checks all three at every step:

| Limit | Question | Example per link |
|---|---|---|
| **Strength** (force, moment, power) | Does anything slip, stall, yield or break? | tool breakage and holder torque (2); spindle power and torque at this RPM, axis thrust (5); slip, lift, tip, turn of the hold (3); stock wall yield (4) |
| **Stiffness** (deflection) | Is the part still inside tolerance under load? | tool deflection (2); machine static stiffness (5); fixture compliance (3); thin-wall deflection (4) |
| **Dynamics** (vibration) | Does anything ring? | tool-tip stability lobes (1, 2); machine modes (5); part-plus-fixture mode (3); thin-wall mode, which drops as the wall is cut (4) |

A fourth kind, **thermal**, affects accuracy (spindle growth, part heating) and changes slowly.
One thermal rule is in scope from the start (Jason's point): **warm the machine up before going
to maximum torque**. The machine library holds the warm-up routine and the derating that
applies until it has run; the rest of thermal behaviour is an open question.

### Time scales: planning, along the path, on the machine

Jason's point is that the negotiation between tool force and workholding force is "realtime
during cutting". There are three ways to read that, and the system needs all three in order:

1. **Planned, per path step (offline).** Walk the toolpath in small steps over the in-process
   stock, compute the force vector at each step, check every link, set the feed per segment.
   This is what physics-based feed optimisers already do for the tool and spindle (the cutting
   physics file covers them); the system extends it to the hold, the stock and the machine. The
   result is real-time in the sense that it is *computed for each instant of the cycle*, but it
   is computed before the cycle starts.
2. **Planned, per operation and per setup.** Choices that cannot change mid-cut: which tool,
   which hold and torque, which machine, which order of operations. These are chosen by trying
   candidates and running (1) for each.
3. **Measured, on the machine (later).** Adaptive control: the control or an edge device reads
   spindle load (or force from a sensor) during the cut and trims the feed. The planned model is
   then the **starting point and the safety net** (the measured value is only allowed to raise
   feed where the model says every link still has margin). Spindle load is available from many
   controls through MTConnect or OPC UA (see the in-process inspection file's table of
   interfaces). Fact: commercial adaptive feed control products exist for CNC controls
   (unverified which controls and how they decide limits).

### Why the stock changes everything

The stock is the only link whose properties change during the cycle, and it changes because of
the cutting itself. Fact (mechanics): the bending stiffness of a wall scales with the cube of
its thickness, so a wall cut from 6 mm to 3 mm becomes about eight times more compliant; its
lowest natural frequency drops at the same time. A vacuum-held plate loses sealed area as it is
cut free; a thin part in a vise distorts more under the same clamp force once its centre is
pocketed. So every check in this system runs on the **in-process stock**, and the order of
cuts decides which checks pass. Topic 4 owns that model; the workholding file explains how the
hold's limits change with it.

### Why the machine closes the loop

The machine is where the load path ends. Its spindle power and torque curve cap the cut at each
RPM; its axis drives have a maximum thrust; its structure and guideways have a static stiffness
and dominant modes that add to the deflection and chatter budget; on a 4th or 5th axis the rotary
brake or clamp torque is part of the workholding. Without a machine model the system can only
check the tool and the part; with it, the same cut can be judged differently on a light router,
a VMC and a horizontal (topic 5).

## Accuracy

- **Each link has its own uncertainty**, set out in each topic file: cutting coefficients
  (typically ±10–20 % from tabulated `kc1.1` data, better when calibrated; see topic 2),
  friction coefficients at the jaws (wide spread; topic 3), machine stiffness (rarely published
  by makers, often has to be measured; topic 5), and in-process stock stiffness (beam estimates
  vs FEA; topic 4).
- **Errors combine.** A limit computed from the weakest link inherits that link's uncertainty;
  a deflection computed from the series chain inherits all of them. This is the physical reason
  for the slider and for labelling assumed values: the program cannot claim more accuracy than its
  worst input (opinion).
- **Measured feedback narrows it.** Spindle load vs predicted power, probed wall thickness vs
  predicted deflection, and inspection results vs tolerance (the in-process inspection file) are
  the data that would let a shop move the slider with confidence and refine coefficients.

## Programs

From public pages; see each topic file for details and citations.

- **Physics-based feed optimisation** exists in commercial verification software (the cutting
  physics file covers it): it computes engagement from a stock model and sets feed per segment
  for tool and spindle limits. In the public material reviewed, the hold, the stock's changing
  stiffness and the machine's directional stiffness are not part of that optimisation
  (unverified for every product).
- **Machining-dynamics tools** (stability lobes from tap tests or receptance coupling) treat the
  tool-holder-spindle link in depth, separately from CAM (topics 1 and 2).
- **Fixture design tools** treat the hold as geometry and, in a few research and fixture-maker
  tools, as a static force check (topic 3).
- **Thin-wall machining** research (deflection compensation, cut sequence for stiffness) exists
  in the academic literature and in some CAM strategies (topic 4).
- **Machine simulation** in CAM uses the machine model for kinematics and collisions, not for
  stiffness or force (topic 5).

**The gap (opinion, from the above):** each link has been studied, and each has tools, but they
are not joined. Joining them is mostly a data and architecture problem, which is why it matters
what SolidCraft's CAM is built on.

## Complaints and requests

Collected in the topic files; the system-level themes are:

- **"The CAM said it was fine."** Programs that check only the tool give feeds the hold, the
  part or the machine cannot take, and the programmer finds out at the machine (topics 2, 3).
- **Presets do not travel.** A feed that works on one machine with one vise fails on another
  (topics 1, 5).
- **Thin parts move and spring.** Clamping harder to stop slipping distorts thin parts;
  finishing a thin wall first-then-roughing-near-it makes it chatter (topics 3, 4).
- **Novices cannot see the limits.** Jason's observation, repeated in each file: the knowledge
  that keeps experienced machinists out of trouble is not visible in CAM today.

## What users need

Everything in this section is opinion: a proposal for the maintainers to accept, change or
reject.

### The shared setup

One object that every part of the system reads and writes, so that no topic needs its own copy
of the world:

```text
Setup
├─ machine        → machine library entry (catalogue + shop instance)       topic 5
│                   kinematics, envelope, spindle torque/power curves per gear range, warm-up
│                   routine and cold derating, axis thrust, stiffness, modes, rotary clamp torque
├─ machine state  → cold / warming / warm (gates full torque)                  topic 5
├─ holds[]        → workholding library entries + their pose on the machine topic 3
│                   capacity envelope per direction, clamp force(torque), contacts, stiffness
├─ stock          → initial stock + material + in-process state              topic 4
│                   material (ISO 513 group, kc1.1/mc or edge coefficients, E, yield)
├─ tools[]        → tool library assemblies (tool + holder + stick-out)      topic 1
│                   geometry, coefficients, limits, tool-tip FRF if known
├─ confidence     → the slider position (one per setup, override per operation)  topic 2
└─ operations[]   → toolpaths with per-segment engagement and the limit that set each feed
```

Fact about SolidCraft today: the open CAM work (RFC #1153 for 2.5D CAM, PR #4158 with a basic
local tool library) has a per-machine tool file and no machine, hold or stock physics yet (from
the tool library file's "Fit with SolidCraft's current CAM work"). The setup above would grow
from that work rather than replace it.

### The solver: three nested loops

```text
outer   per setup     choose machine, holds and torque windows, operation ORDER   (topics 3, 4, 5)
middle  per operation choose tool, strategy, entry side, climb/conventional        (topics 1, 2, 3)
inner   per step      engagement → force wrench → check every link → feed scale    (all five)
```

The inner loop, per step `i` of a toolpath, on the in-process stock:

1. **Engagement** from the toolpath and the in-process stock: radial and axial engagement, chip
   thickness (topic 2; the stock model is topic 4).
2. **Force wrench**: force vector `F` and moment `M` in the tool frame, transformed to the part
   and machine frames at the contact point (topic 2).
3. **Check every link**, each returning the largest allowed scale on the load in that
   direction, after its safety factor from the slider:
   `s_tool`, `s_holder`, `s_spindle(n)`, `s_axis(u)`, `s_hold(u, M)`, `s_stock(u, i)`, and the
   deflection and stability margins for each. The spindle link is the machine library's
   **torque and power curves** read at this step's speed `n` (and gear range, where the spindle
   has one), `M_spindle(n)` and `P_spindle(n)`, not a single nameplate rating; a spindle drive
   can have far less torque at low speed than its rated power suggests. Any change of speed (for
   example to move out of a chatter zone) re-reads the curve (Jason's point).
4. **Feed scale** `s = min(all of them)`, converted to a feed through the force-vs-chip-thickness
   relation (the workholding file gives `s_feed ≈ s^(1/(1 − m_c))`), capped by the hard limits.
5. **Machine state gate (Jason's point).** Until the machine has run its warm-up routine
   (held in the machine library), spindle torque and power, and speed, are derated; the cut is
   not allowed to reach maximum torque on a cold machine. The slider cannot override this gate.
6. **Record the binding link** (which link and which direction set `s`) on the segment, so the
   result can explain itself.

The middle loop tries alternatives where the inner loop found a weak direction (cut toward the
fixed jaw, ramp instead of plunge, a stiffer tool) and keeps the one with the shortest time that
passes. The outer loop tries operation orders and holds (topic 4 plans the order so the stock
stays stiff while forces are high; topic 3 gives each hold's torque window), and ranks feasible
setups by: passes every check, then cycle time, then predicted distortion, then setup effort
(the order proposed in the workholding file).

**Cost.** The inner loop is cheap per step (closed-form force and beam formulas) but runs over
every step of every toolpath; the stock model is the expensive part. Opinion: start with a
dexel or voxel in-process stock and beam/plate estimates for stiffness (no B-rep booleans, which
also keeps this out of the paused kernel work), and allow an FEA check from the FEA research
(#1620) as an optional, slower, more accurate stock model later.

### Physics in the simulation: the gate before G-code (Jason's point)

The rules above are only useful if nothing gets past them. Jason's requirement: the physics
model is **built into the simulation**, so the rules are followed through to the end and a
novice's toolpath error cannot kill the work, the tools or the machine.

What that means in practice (opinion on the method):

- **One model, two uses.** The simulation that removes material and checks collisions uses the
  same in-process stock, setup and limits engine as the feed calculation. It does not keep its
  own simplified copy, so what is simulated is exactly what is checked.
- **Every move is checked, including moves the optimiser did not write.** Hand edits, imported
  toolpaths, rapids, plunges, linking moves, a feed typed over the calculated one, a tool or
  hold changed after the toolpath was computed: the simulation re-runs the inner loop over the
  final program, so a later change cannot silently break a limit.
- **The checks are physical, not only geometric.** Besides gouges and collisions, the
  simulation flags, at the move where it happens: tool or holder overload, spindle torque or
  power above the curve at that speed, full torque on a cold machine, axis thrust above the
  machine's limit, slip, lift, tip or turn of the hold, a wall that deflects out of tolerance or
  rings, a full-width slot or plunge the tool cannot take, minimum chip thickness (rubbing).
- **Severity, with the binding limit named.** *Stop* (would break or move something: tool,
  part, hold, machine), *Warn* (out of tolerance, chatter risk, short tool life), *Info*
  (assumed data used). Each item says which link, which direction and by how much, and offers
  the fix the solver found (lower feed on these moves, cut toward the fixed jaw, re-order these
  operations, raise vise torque to X N·m).
- **A gate before posting.** Posting G-code requires a simulation with no open *Stop* items.
  **Any** *Stop* item, including one on a hard limit, can be overridden (Jason's decision), but
  only item by item, after a **disclaimer** that names the limit, the step and the margin by
  which it is exceeded, and with a **recorded approval**: who, when, which limit, by how much
  and the reason. The record is stored in the design with the program and printed on the setup
  sheet and part traveller (inspection file). The slider changes where *Warn* starts; it never
  removes a *Stop* by itself.
- **Soft findings are fixed automatically where possible.** *Warn* items the feed optimiser can
  solve (slow these moves, split this cut) are fixed and listed; only what needs a person's
  choice is left open. The setup sheet carries the assumptions the simulation used (friction,
  torque, machine state, estimated data).
- **Novice default.** New users start conservative, with the gate on; the gate cannot be
  switched off globally, only accepted item by item.

In SolidCraft terms this is one more command (a physics-checked simulation whose output is the
list of findings), and the post-processor command refuses, with a clear error and not a panic,
when there are unaccepted *Stop* findings.

### One slider for the whole chain

The cutting physics file defines the slider and its table for the tool and spindle; the
workholding file adds its safety factors. The system rule is that **one** slider position sets
every link's margin together, with these properties (from those files):

- each link has its own scale from conservative to aggressive (e.g. force vs tool rating 50 → 90 %,
  friction-held workholding safety factor 3.0 → 2.0);
- **hard limits never move**: maximum RPM of tool, holder and machine; the spindle torque and
  power curves at the step's speed; the cold-machine derating until warm-up has run; machine feed
  maxima; finishing tolerance; minimum chip thickness; collisions; the hold's floor (1.5);
- **missing data pulls toward conservative**: an estimated machine stiffness, a default friction
  value or an unmeasured tool FRF caps that link at the conservative setting and is shown as
  assumed;
- the slider shows its effect as a trade-off ("conservative: 40 N·m, 12 min; aggressive:
  25 N·m, 9 min").

Machine and stock links (topics 4 and 5) add their own rows to that table in their files.

### The closed loop: Self-tuning CAM Operations (Jason's name)

Jason calls this loop **Self-tuning CAM Operations**: the machine's measured behaviour, lined
up with the program, tunes the libraries the CAM plans from, so each run of a job (and each
new job on the same machine, hold and material) is planned from better data.

The system makes predictions that the shop can check:

Summary of what Self-tuning CAM Operations measures against the plan:

| Prediction | Measured by | Feeds back into |
|---|---|---|
| spindle torque and power per segment, against the curve at that speed | spindle load (MTConnect, OPC UA, control load meter) | material `kc1.1`, the slider's comfort level |
| wall deflection, part distortion | in-process probing of walls and datums | stock model, clamp torque window |
| finished size and form | in-process and final inspection (per-part record) | the whole chain's accuracy |
| chatter-free depth | sound or vibration, surface finish | tool-tip FRF, machine modes |

**Real-time feedback from the edge box (Jason's idea).** The Raspberry Pi edge device from the
in-process inspection file could also read **vibration, spindle load, axis loads and torques in
real time** from machines whose control publishes them (through MTConnect, OPC UA or the
control maker's interface; which signals each control publishes belongs in the machine library).
**Aligned with the G-code and the timeline (Jason's point).** Every sample is stored against
the program it belongs to: the program and its version, the **block (line) number** and the
toolpath segment that line came from, the operation, the tool, and a timestamp on the cycle
timeline. Most controls publish the executing block or line number alongside load data
(unverified per control; MTConnect defines a block/line data item). With that, an event (a
load spike, a vibration peak, a stall) is located on the exact move in the program, shown on the
simulation's timeline next to the predicted value for that move, and tied to the operation and
setup in the design. Without alignment the data shows *that* something happened but not
*where*.

Uses, in order of risk (opinion):

1. **Record and compare** (first): measured vs predicted torque, power and force per segment,
   stored with the part record; the gap corrects the shop instance (material coefficients,
   friction at the jaws, the spindle curve).
2. **Detect and alarm**: an unexpected jump in load or a vibration peak near a predicted part,
   fixture or wall mode suggests slip, lift or chatter; the box flags it and, where the control
   allows, can request a feed hold through the control's own safe input (unverified per control).
3. **Adjust feed** (later): only through the control maker's adaptive-control interface, inside
   the envelope the planned model allows, after a safety review. The box never bypasses the
   control's own limits.

The in-process inspection file covers the probing, gauging and per-part record on the same
device. Opinion: feedback should **update the shop instance**
(this machine, this vise, this material lot), never silently change the shared catalogue; a shop
can choose to contribute measured values back, labelled as measured.

### The libraries, one pattern

Tools, workholding and machines all use the pattern from the tool library file: a **shared
catalogue** entry (what the product is, described once, collaboratively, from public data or
measurement), a **shop instance** (this one, with its wear, measured runout, measured stiffness,
tag), and optional **tags** to identify it at the machine. Material data (cutting coefficients,
modulus, yield) is shared by all three and could be a fourth catalogue (open question). Using one
pattern means one sync, one offline cache, one review process and one licence policy for all
shared data.

### How it fits SolidCraft's architecture

Facts from the repository's own rules (AGENTS.md, docs/architecture.md): everything a user does
is a command in `crates/engine/src/cmd/*`; layers run `geom` → `sketch`/`kernel`/`render` →
`doc` → `io` → `engine` → `ui-egui` → apps; everything below the apps builds for wasm; no panics;
units are millimetres and radians.

Opinion on placement:

- The physics (force models, link checks, the solver loops) is plain numeric code with no UI or
  kernel dependency, so it can sit in its own crate low in the stack, beside the CAM core, build
  for wasm and be tested with analytic values (the repository's oracle approach fits: e.g. a
  cantilever deflection or a vise slip limit has a closed-form expected value).
- The libraries are data, versioned in a public schema, stored with the design (shop instances
  used by a setup) and cached from the shared catalogue.
- Each check, each recommendation and each library action is a command with its own SolidCraft
  id, input and output schema, so the toolbar, scripts, CLI, control channel and MCP reach the
  same physics. A result's binding limit is part of the command's output, so agents and the UI
  can explain it the same way.
- Input-derived numbers (library entries are shared data) are treated as hostile: finite checks,
  ranges, caps, as the repository requires.

### Phased plan

Each phase is useful on its own and none needs rework when the next lands, **if** phase 0 is
in the CAM core from the start.

| Phase | Adds | Topics | Useful because |
|---|---|---|---|
| 0. Cradle | the setup object with machine, holds, stock, tools, material and slider fields; toolpath generators that emit engagement per segment; a limits engine with "binding limit" output; units and library schemas | all | everything below plugs in without changing generators or file formats |
| 1. Per operation | tool and spindle checks from the tool library and Kienzle, against the spindle torque and power curves at the operation's speed; warm-up gate; the slider | 1, 2, 5 (spindle curves and warm-up only) | catches broken tools and stalled spindles for any engagement, which presets cannot |
| 2. Per segment | feed per segment from engagement on a simple in-process stock | 2, 4 (stock geometry only) | the feed-optimiser milestone |
| 3. Workholding | capacity envelope per direction, torque window, best-hold suggestion, direction-aware feed | 3 | the novice's biggest blind spot; the real-time force-vector negotiation |
| 4. Machine | axis thrust and stiffness, rotary clamp torque, machine modes | 5 | the same job judged correctly on different machines |
| 5. Stock integrity | stock stiffness as it is cut, deflection and wall modes, cut-order planning, re-clamp before finishing | 4 (+ FEA #1620) | thin parts; balancing stock against tool and hold |
| (every phase) Simulation gate | each phase's checks run over the final program in simulation, and *Stop* findings block posting | all | novice errors are caught before they reach the machine, from the first phase on |
| 6. Self-tuning CAM Operations | live load, torque and vibration (edge box), aligned to G-code lines and the timeline, plus probing, feeding back into shop instances | inspection file | lets the shop trust and move the slider |

### The request to the maintainers

Concretely, the request is that SolidCraft's CAM (#1153, PR #4158 and what follows) be
designed around this system from the start:

1. **A setup that carries physics.** The CAM document holds the machine, the holds and their
   pose, the stock and its material, the tools as assemblies, and the slider, even while most
   fields are optional and unused by the first operations.
2. **Engagement from every toolpath generator.** Each generator reports, per segment, the
   radial and axial engagement and the contact point (or enough for a stock model to compute
   them). This is the hook every physics check needs, and the most expensive thing to add later.
3. **One limits engine.** All limits (tool, holder, spindle, axes, hold, stock) go through one
   place that returns the allowed scale and names the binding limit, instead of each feature
   adding its own clamp on feed.
4. **One library pattern** for tools, workholding and machines (catalogue, shop instance, tag),
   with a public schema, so contributors can add products the way the topic files describe.
5. **Explain every number.** Every computed feed, torque or recommendation carries the limit
   that set it and whether its inputs are measured, catalogue or assumed.
6. **Physics in the simulation, as a gate.** The toolpath simulation uses the same setup, stock
   and limits engine, checks every move of the final program (including hand edits and linking
   moves), and posting G-code requires no open *Stop* findings. Any finding, hard limits
   included, can be overridden item by item after a disclaimer, with the approval recorded in
   the design and on the setup sheet.

None of this requires the physics itself to be finished first; it asks that the first CAM code
leave room for it in the right places.

## Glossary

- **Binding limit:** the link and direction that set the allowed load at a step.
- **Capacity envelope:** the allowed force and moment in each direction for a hold (topic 3) or
  a link generally.
- **Compliance:** inverse of stiffness (µm/N); compliances add in a series load path.
- **Self-tuning CAM Operations:** Jason's name for the closed loop in which live machine data,
  aligned with the G-code lines and the cycle timeline, tunes the shop's library entries.
- **Gate (simulation gate):** the rule that a program cannot be posted while the physics-checked
  simulation has unaccepted *Stop* findings.
- **Engagement:** how much of the tool is in the material (radial width `ae`, axial depth `ap`,
  entry and exit angles).
- **In-process stock:** the stock as it is at a given point in the cycle.
- **Link:** one element of the load path (tool assembly, spindle, machine axes, hold, stock).
- **Load path:** the closed chain through which cutting force flows back to the spindle.
- **Shop instance:** a specific physical item a shop owns, linked to a catalogue entry.
- **Slider (confidence slider):** Jason's conservative-to-aggressive control of all safety
  factors at once.
- **Wrench:** a force and moment taken together at a reference point.

## Open questions

1. Is a separate **material catalogue** needed, or does material data live with cutting data in
   the tool library?
2. How fine must the inner loop's steps be to catch direction changes in corners and ramps
   without making long programs slow to compute?
3. Which in-process stock representation (dexel, voxel, mesh) is accurate enough for stiffness
   estimates, and when must topic 4 fall back to FEA (#1620)?
4. Should the slider be one value per setup, or per link with one master control?
5. Beyond the warm-up gate, is thermal behaviour (spindle growth, part heating, release
   distortion after unclamping) inside this system or a later topic?
6. Where should real-time adaptive control live: in the control's own option, in an edge device
   (the inspection file's Pi), or not in SolidCraft at all?
7. Does lathe and mill-turn work fit the same setup (chuck as hold, turret as tool holder), or
   does it need its own model?

## Sources

This file is a synthesis; each topic file lists its own sources. Additional references used here:

- Altintas, Y., *Manufacturing Automation: Metal Cutting Mechanics, Machine Tool Vibrations, and
  CNC Design*, 2nd ed., Cambridge University Press, 2012: cutting force models, machine and
  tool dynamics, and adaptive control in one framework (unverified chapter numbers).
- Schmitz, T. L. and Smith, K. S., *Machining Dynamics: Frequency Response to Improved
  Productivity*, 2nd ed., Springer, 2019: tool-holder-spindle and workpiece dynamics, thin-wall
  and fixture compliance as parts of one system (unverified edition year).
- [MTConnect Institute, MTConnect standard](https://www.mtconnect.org/) (accessed 2026-10-26):
  read-only machine data including spindle load.
- [OPC Foundation, OPC 40501 Machine Tools companion specification](https://opcfoundation.org/)
  (accessed 2026-10-26): machine tool monitoring information model.
- ISO 230 series (test code for machine tools), from its public catalogue entries: the standard
  vocabulary for measuring machine geometry, stiffness-related and thermal behaviour (see the
  machine library file).
- SolidCraft repository: AGENTS.md and docs/architecture.md (layers, commands, wasm, no panics);
  issues #1153, #1620, #1621 and PR #4158 (current CAM and FEA work).
