# CAM research: tool holding physics, or "how hard can this holder be pushed?"

Research for [#1621](https://github.com/tangency-labs/solidcraft/issues/1621) (Research Request
for Plugin: CAM). This file covers one part of that request: **the link between the cutting tool
and the machine spindle** (collets and nuts, chucks, tapers, pull studs, drawbars, coolant
seals, and holders for tools that do not rotate) and **the physics that limits it**: maximum
speed, balance, clamping torque and tool slip, pull-out, runout, stick-out and stiffness,
vibration, drawbar and pull stud loads, fastener torques, through-spindle coolant, and
alignment of non-rotating tools such as broaches. Other CAM topics are out of scope here.

It is one part of a set that together describes machining as one physical system. The holder
sits in the middle of the load path, between the cutter and the spindle:

```
  spindle (drawbar, taper) ─> pull stud ─> holder body ─> clamp (collet / shrink / hydraulic / screw) ─> tool shank ─> cutting edge
```

Related files in the same fork:

- [CAM Physics System overview](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-vgqo3g/research/cam-research-nsnlookupjn-physics-system.md):
  the shared setup, the "lowest limit wins" solver, the one slider, and the simulation gate that
  the holder limits below feed into.
- [Shared cloud tool library](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-tool-library/research/cam-research-nsnlookupjn-tool-library.md):
  where holder catalogue items and assemblies (cutter + holder + stick-out) live. This file adds
  the **physics fields** a holder item needs.
- [Cutting tool physics](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-cutting-physics/research/cam-research-nsnlookupjn-cutting-physics.md):
  the source of cutting torque `Mc`, forces `Ft/Fr/Fa`, tooth-passing frequency and the
  conservative-to-aggressive slider. Its section on runout is extended here.
- [Machine library](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-machine-library/research/cam-research-nsnlookupjn-machine-library.md):
  spindle taper, drawbar force, maximum speed, torque curves, spindle receptances (RCSA) and
  the compliance chain the holder is one link of.
- [Workholding physics](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-workholding/research/cam-research-nsnlookupjn-workholding.md):
  the same friction-hold and form-hold physics, on the part side instead of the tool side.
- [Machine health](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-9w3gq9/research/cam-research-nsnlookupjn-machine-health.md)
  and [in-process inspection](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-y2w06n/research/cam-research-nsnlookupjn-in-process-inspection.md):
  drawbar force checks, pull stud replacement, and measuring tool length to catch a tool that
  has slipped.
- [Gear cutting, splines and broaching](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-gyhruq/research/cam-research-nsnlookupjn-gear-spline-broach.md):
  the broaching *process* (tooth load, stroke force, pass plans). This file owns the *holder*
  side of non-rotating tools; the split is described in "Non-rotating tools" below.

Licence: MIT OR Apache-2.0, the same as SolidCraft (see the Licence section of CONTRIBUTING.md).

## About this research

- **Author:** Jason ([@nsnlookupjn](https://github.com/nsnlookupjn)). Idea and direction by Jason;
  source research and writing done with an AI assistant (Claude) in October 2026.
- **How it was produced:** web searches of public sources (holder and machine makers' public
  tables and articles, trade press, practitioner forums), each claim checked against the cited
  page where possible. No commercial CAD/CAM program was driven or scripted, no GPL/LGPL/AGPL
  source was read, and no text is copied beyond short quotes. Standards (ISO 21940-11, ASME
  B5.50, ISO 12164, ISO 15488 / DIN 6499, ISO 26623) are described from public summaries, not
  their paid text. The worked examples are textbook mechanics computed for this file; where
  they rest on assumed coefficients, the assumptions are stated.
- **Units:** millimetres, newtons, newton-metres, rev/min, bar, grams-millimetres (g·mm) for
  unbalance, unless a source is quoted in other units (then converted: 1 ft·lb = 1.356 N·m,
  1 lbf = 4.448 N, 1 psi = 0.0689 bar).
- **Marking:** "(unverified)" marks a claim not confirmed against a source. **Fact**,
  **opinion**, **estimate** and **marketing** are labelled where they mix.

> **[TODO Jason: personal experience]** Which holders you run (ER, shrink, hydraulic, side-lock,
> milling chucks), on which tapers, how you torque ER nuts and pull studs today (by feel,
> torque wrench, TORCO-style fixture), and any tool you have seen slip, pull out, walk down,
> chatter from stick-out, or any pull stud or drawbar problem. Label this as personal
> experience.

## Summary

0. **Why this matters: the holder is an invisible limit.** Jason's starting point: maximum RPM
   and tool retainer torque should be **limited by the tool holding style**. An ER32 collet, a
   shrink-fit chuck and a side-lock holder hold the same 12 mm end mill very differently, but a
   CAM program sees them as three shapes to avoid in simulation. A novice cannot see that the
   ER32 nut needs about 136 N·m to grip a 20 mm shank properly, that the holder's balance
   decides how fast it may spin, or that a worn drawbar halves the force keeping the holder in
   the spindle. If CAM knows the holder's physics, it can show these limits and keep the
   toolpath inside them.
1. **The holder is a chain of joints, and each joint has a limit.** The spindle taper is held by
   the drawbar through the pull stud; the tool is held in the holder by friction (collet,
   shrink-fit, hydraulic, milling chuck) or by form (Weldon flat, whistle notch, Safe-Lock-style
   grooves). Each joint can **slip** (torque), **pull out** (axial force), **open up** (bending
   moment), **fly apart or lose grip** (speed), or **leak** (coolant). Checking a holder means
   comparing each cutting load with each joint's capacity, the same way the workholding file
   checks a vise.
2. **Speed is limited by several things at once.** The holder's rated speed, its balance quality
   at the assembly's real mass (ISO 21940-11), the nut's speed rating, the taper interface
   (ASME B5.50 steep tapers are described as suitable "up to 25,000 rpm"; practitioners keep
   40-taper holders well below that), grip loss from centrifugal expansion, the tool's own limit
   and the spindle. The allowed speed is the **lowest** of these, and it is a hard limit that no
   slider moves.
3. **Clamping torque is a property of the holder, the nut, the collet bore and the shank
   condition.** Rego-Fix publishes ER nut torques per collet size and bore: for ER32 with a
   3 to 22 mm bore, 136 N·m (100 ft·lb) on a standard nut, less for small bores, and different
   values for each nut type. A ball-bearing (low-friction) nut turns the same wrench torque into
   more clamping force, so it needs **less** torque for the same grip. An oily shank can lose
   about 40 % of a hydraulic chuck's torque capacity (900 vs 520 N·m quoted for one 20 mm chuck).
4. **Pull-out is not only a static force check.** Helical cutters pull themselves out of friction
   holders. Static margins look large, yet tools still walk out in heavy roughing because the
   load rotates and flexes the shank every revolution. Holder makers answer with form-locking
   shanks; CAM should flag high-risk combinations (high helix, high radial engagement, friction
   holder, long stick-out) rather than trust a static number.
5. **Runout costs tool life and changes chip load per tooth.** A 0.012 mm runout on a 2-flute
   cutter fed at 0.025 mm/tooth makes one tooth cut about 0.037 mm and the other about
   0.013 mm. A holder maker states the rule of thumb "one tenth (.0001") of runout will
   increase or decrease tool life by 10 %". CAM can use the holder's rated runout (or a measured
   value) to raise the effective peak chip load used in force and tool-life checks.
6. **Stick-out is the strongest lever on stiffness.** Tool deflection grows with the **cube** of
   stick-out: 48 mm instead of 36 mm is 2.4 times the deflection. The holder sets the minimum
   stick-out (its nose shape and reach) and the clamped length.
7. **The spindle side has limits too.** Drawbar force (CAT40 about 10 kN suggested; Haas lists
   6.5 to 10.2 kN for most of its 40-taper mills and 12.6 to 18 kN for HSK-A63), pull stud
   tightening torque (Rego-Fix gives a **maximum** of 50 N·m for SK/BT/CAT 40; another tooling
   seller quotes 45 to 55 ft·lb, about 61 to 75 N·m, so sources disagree and the maker's value
   must win), pull stud life (about three years, or 6,000 to 8,000 hours per Haas guidance quoted
   by MSC), and a drawbar that has lost about half its force signals broken disc springs.
8. **Through-spindle coolant is a pressure joint.** Pressures of 20 to 140 bar need a sealed
   path: rotary union, pull stud bore, holder (coolant form), collet seal or sealing disc, tool.
   Sealed ER collets are rated to about 138 bar by one maker, while users report leaks at 17 bar
   with other brands. Pressure also **pushes the tool out of the holder** (about 2.2 kN on a
   20 mm shank at 70 bar, estimate) and the holder out of the spindle.
9. **Non-rotating tools need orientation and alignment, not speed.** Broaching, shaping and
   static lathe tools need the spindle oriented and held (M19 and a brake or lock), the cutting
   edge aligned to the feature, and the edge on centre height (one guideline: within 0.1 mm,
   tighter on small diameters). The holder must resist the twist that the stroke force creates.
10. **The idea: a physics-based holder library.** Each holder catalogue item carries its limits
   (speed, balance grade and residual unbalance, clamping torque vs shank diameter, pull-out
   resistance, runout, stiffness, coolant pressure, nut and fastener torques). The CAM solver
   takes the lowest limit along the chain, the simulation gate refuses toolpaths that break a
   hard limit, and the setup sheet tells the operator which torque to apply.

## Fundamentals

### 1. The joints in the chain

| Joint | Held by | Fails by | What decides the limit |
|---|---|---|---|
| Spindle taper ↔ holder | Drawbar force through the pull stud; friction on the taper (and face, for dual-contact) | Holder lifting or rocking under bending moment, fretting, pulling out under axial load | Drawbar force, taper type (steep, dual-contact, HSK, Capto), speed |
| Pull stud ↔ holder | Thread | Fatigue fracture, thread failure, over-torque swelling the taper | Stud material and age, tightening torque |
| Holder ↔ tool (friction) | Radial pressure from collet, shrink fit, hydraulic membrane, milling chuck | Slip in rotation (torque), slip along the axis (pull-out), walking out under cyclic load | Clamping pressure × friction × contact area, shank tolerance and condition |
| Holder ↔ tool (form) | Weldon flat, whistle notch, threaded shank, grooves | Screw or key failure, runout, unbalance | Screw size and torque, geometry |
| Holder body | Steel | Bending, vibration, burst at overspeed | Diameter, length, material, balance |
| Coolant path | Seals, O-rings, sealing discs | Leak, pressure loss, dry-running seal damage | Pressure rating of each seal, rotary union rating |

The same two kinds of hold described in the workholding file appear here: **friction holds**
(collets, shrink-fit, hydraulic, milling chucks, the taper itself) keep the tool only while the
load is below `friction coefficient × clamping force`, and **form holds** (Weldon flats,
whistle notch, positive-lock grooves, drive keys) carry load through geometry.

### 2. Holder styles

| Style | How it grips | Strengths | Weaknesses | Typical runout |
|---|---|---|---|---|
| ER collet chuck (ISO 15488, formerly DIN 6499) | Slotted collet with 8° back taper squeezed by a threaded nut | Cheap, flexible: each collet covers about a 1 mm range | Grip depends on nut torque and friction; nut adds mass and unbalance; less torque than hydraulic or shrink | Standard ≤ 15 µm at 3×D; precision ≤ 10 µm; premium ≤ 5 µm ([AIMS Industrial](https://aimsindustrial.com.au/en-nz/blogs/product-guides/er-collet-guide)) |
| ER with ball-bearing (low-friction) nut | Same collet; a bearing ring between nut and collet removes most face friction | More clamping force per N·m of wrench torque | Different torque table; bearing can wear | As ER |
| Shrink-fit | Bore about 0.01 mm smaller than the shank; heated to expand, cools to grip ([CTE](https://www.ctemag.com/articles/shrink-fit-toolholder-fundamentals)) | Very low runout, symmetric, slim; "inherently balanced" | Needs a heater; grip falls as centrifugal force expands the bore; life 20,000 to 50,000 cycles | Low (few µm) |
| Hydraulic expansion | Oil pressure expands a thin membrane onto the shank | Low runout, damping, high torque on dry shanks | Shank must be clean and dry; torque falls with oil (900 → 520 N·m quoted for one 20 mm chuck, [Manufacturers' Monthly](https://www.manmonthly.com.au/product/hydraulic-expansion-toolholders/)) | Low |
| Milling chuck (roller / power chuck) | Needle rollers or a sleeve squeeze the bore | High grip for roughing | Heavier, larger nose | Moderate |
| Side-lock (Weldon, DIN 1835-B / DIN 6535-HB) | Set screw on a flat | Positive drive, no pull-out | Pushes the tool off centre: worse runout and balance ([CTE, Haimer case study](https://ctemag.com/?p=100021)) | Poor |
| Whistle notch (DIN 1835-E / 6535-HE) | Inclined flat and screw | Positive, resists pull-out | Asymmetric: runout and balance suffer | Poor |
| Friction + form (e.g. Safe-Lock grooves) | Grip plus drive keys in helical grooves on the shank | Friction runout with form-lock against pull-out | Needs special shanks | < 3 µm claimed (marketing, same source) |
| Shell mill arbor, face mill adapter | Pilot, drive keys, centre screw | Carries large cutters | Screw torque and key fit matter | n/a |
| Tapping holder (rigid or floating) | Square drive, tension/compression float | Absorbs pitch mismatch | Float adds compliance | n/a |

Spindle-side interfaces:

| Interface | Contact | Notes |
|---|---|---|
| CAT (ASME B5.50), BT (JIS/MAS) steep taper 7:24 | Taper only | Spindle growth at speed can pull the holder up into the taper; taper-only holders "tend to bellmouth the spindle over time" ([CTE](https://ctemag.com/?p=96096)). Haimer describes the ASME B5.50 interface as "primarily suitable for applications with a spindle speed of up to 25,000 rpm" ([Haimer](https://haimer.com/us/products/tool-holders/steep-taper-ASME-B5.50-CAT40-CAT50)); one expert in the CTE article advises under 20,000 rpm for 40-taper. |
| Dual contact steep taper (e.g. BIG-PLUS) | Taper + face | Face contact stops the holder being drawn up; a CV40 dual-contact holder deflected about 60 µm at 70 mm under 500 kg radial load, roughly half a taper-only holder (CTE, maker data). |
| HSK (ISO 12164) | Short hollow taper + face | The hollow shank expands with the spindle at speed; HSK63 is cited for about 20,000 to 30,000 rpm, HSK32 near 45,000 rpm (CTE). |
| Polygon taper (Capto, ISO 26623) | Polygon + face | Torque carried by the polygon without keys (CTE). |

### 3. Speed limits and balance

**Balance.** An unbalanced holder spins with its centre of mass offset by `e` from the axis.
The unbalance is `U = m · e` (g·mm) and the force it creates is `F = U · ω²`, so force grows
with the **square** of speed. ISO 21940-11 (formerly ISO 1940-1) sets a balance grade `G`
(mm/s), and the permissible unbalance is

```
U_per [g·mm] = G · 9549 · m [kg] / n [rpm]
```

([CTE, "Toolholder and tool balance"](https://ctemag.com/?p=96427)). The same article gives G6.3
for general-purpose, G2.5 for high-speed and G1.0 for precision machines, shows a 5 g·mm
assembly making about 1.4 N at 5,000 rpm and about 88 N at 40,000 rpm, and suggests better
balance is worth it when the unbalance force reaches about 5 % of the cutting force. That last
point matters for finishing, where cutting forces are small.

*Worked example (computed for this file).* A holder plus tool of 1.2 kg at 20,000 rpm and G2.5:
`U_per = 2.5 · 9549 · 1.2 / 20,000 ≈ 1.43 g·mm`, an offset `e ≈ 1.2 µm`, and a force
`F = 1.43e-6 kg·m · (2π·20,000/60)² ≈ 6.3 N`. The same formula run backwards gives the
**speed limit from balance**: if the measured assembly unbalance is 5 g·mm, then
`n_max = G · 9549 · m / U = 2.5 · 9549 · 1.2 / 5 ≈ 5,700 rpm` for G2.5. A holder marked "G2.5 at
25,000 rpm" only stays G2.5 at that speed if nothing added (nut, side-lock screw, asymmetric
tool) moves the assembly's unbalance. That is why the library must hold the **residual
unbalance in g·mm**, not only the grade and a speed.

**Other speed limits.** The lowest of these wins:

| Limit | Source of the number |
|---|---|
| Holder rated speed (often marked on the holder) | Holder maker |
| Balance speed from `U` and `m` (above) | Holder maker's residual unbalance, or a balancing machine measurement of the assembly |
| Nut rating (standard vs high-speed balanced ER nuts) | Nut maker |
| Taper interface (steep, dual-contact, HSK, Capto) | Standard and holder maker; practitioner guidance lower |
| Grip loss at speed (shrink-fit and hydraulic bores expand with centrifugal force; CTE notes shrink-fit grip "weakens as spindle speed rises") | Holder maker's speed-dependent torque, if published (often not) |
| Tool's own maximum (`RPMX` in ISO 13399) | Tool maker |
| Spindle maximum and current gear range | Machine library |
| Rotary union and coolant (see section 9) | Machine library |

### 4. Clamping force, torque and slip

**From wrench torque to clamping force (ER nut).** The wrench torque on the nut is spent on
thread friction and on friction where the nut presses the collet:

```
T_nut = F_a · ( r_2 · tan(λ + ρ') + μ_c · r_c )
```

where `F_a` is the axial force the nut puts on the collet, `r_2` the thread pitch radius, `λ`
the lead angle, `ρ'` the thread friction angle, `μ_c` the friction between nut and collet and
`r_c` its effective radius. The collet's 8° back taper then turns `F_a` into radial force on the
shank: `F_N ≈ F_a / tan(α + ρ)` with `α = 8°` and `ρ` the taper friction angle. Part of `F_N` is
used up closing the collet slots, so the grip on the shank is less than this.

*Worked example (estimate; assumed coefficients).* ER32 nut thread M40×1.5 (unverified),
`r_2 ≈ 19.5 mm`, `λ ≈ 0.7°`, thread friction 0.12 (`ρ' ≈ 7.9°`), `μ_c = 0.15`, `r_c ≈ 15 mm`. At the
recommended 136 N·m: the thread term is about 2.95 mm, the collet-face term about 2.25 mm, so
`F_a ≈ 136 / 0.0052 ≈ 26 kN`. With taper friction 0.10, `F_N ≈ 26 / tan(13.7°) ≈ 107 kN` (gross,
before slot-closing losses). On a 20 mm shank with shank friction 0.15, the slip torque is at
most `0.15 · 107,000 · 0.010 ≈ 160 N·m` and the axial pull-out resistance at most about
`16 kN`. These are **upper bounds**; the maker's rated torque should always replace them.

**Why ball-bearing nuts need less torque.** Replace `μ_c ≈ 0.15` with a bearing (about 0.01) and
the collet-face term nearly vanishes: the same 26 kN now needs about `26,000 · 0.0031 ≈ 81 N·m`
instead of 136 N·m. Published tables point the same way: Rego-Fix lists different torques per
nut type for the same collet (ER32, 3–7.5 mm bore: 136 N·m on the ER nut, 128 on ERB, 104 on
ERAX; [Rego-Fix torque table](https://usa.rego-fix.com/rego_torque.pdf)), and a collet guide
gives ER32 "100–130 Nm standard, 55–75 Nm ball-bearing" ([AIMS Industrial](https://aimsindustrial.com.au/en-nz/blogs/product-guides/er-collet-guide)).
**Opinion:** applying the standard-nut torque to a bearing nut over-tightens the collet; the
library must key nut torque by **nut type**, not only collet size.

**Published ER nut torques (fact).** From the Rego-Fix table (ER/ERC standard nut column), in
N·m: ER11 3–7 mm 24; ER16 5–10 mm 56; ER20 7–13 mm 80; ER25 8–17 mm 104; **ER32 3–22 mm 136**
(2–2.5 mm only 24); ER40 3–26 mm 176; ER50 6–36 mm 240. Small bores get far less torque to avoid
crushing small shanks, so the limit is a function of **collet size, bore and nut type**. Rego-Fix
recommends a torque wrench or its TORCO-BLOCK fixture.

**Shank condition.** Friction holds depend on clean, dry, round shanks with a tight tolerance
(h6 is common for shrink and hydraulic chucks, unverified per holder). Oil cuts friction: the
900 → 520 N·m example above is a 42 % loss.

**The check.** Per move, from the cutting physics: `Mc ≤ T_slip(holder, shank, condition) / SF_slip`.
Torque slip shows up as a tool that spins in the holder, scores the shank and the bore, and
loses its length offset.

### 5. Pull-out and walking out

A helical cutter's flutes push the chip up and pull the tool **down out of the holder**. A
first estimate of the axial pull is `F_a,tool ≈ F_t · tan(β)` per engaged tooth, with `β` the
helix angle (estimate; the cutting physics file owns the better model). For a 45° helix and
600 N tangential force per engaged tooth this is about 600 N, small next to a static pull-out
resistance of several kN. Yet tools do pull out. A Haimer case article explains that in heavy
roughing "high pulling forces combine with high cutting forces", the tool flexes in the holder,
and precision friction holders (shrink-fit, hydraulic, milling chucks) are "especially
affected"; trochoidal paths using the full flute length raise the axial force
([CTE](https://ctemag.com/?p=100021)). The bending load rotates with the spindle, so the shank
is flexed in every direction once per revolution; the joint can creep a little each turn even
when the static margin looks large (mechanism described as "micro-creep" by some sources,
unverified). Form-locking shanks (Weldon, whistle notch, helical grooves with drive keys) stop
this; side-lock solutions cost runout and balance (same source).

**Opinion: CAM should flag, not compute, walk-out risk.** A static check (`F_a,tool + F_coolant
≤ F_pullout / SF`) catches gross errors. Walk-out needs a rule-based flag: friction holder +
helix ≥ 40° + radial engagement and depth above a threshold + long stick-out → recommend a
form-lock holder or lower axial engagement, and ask the in-process loop to re-measure tool length
after the operation.

### 6. Runout

Runout is the radial offset between the tool's cutting edges and the spindle axis. It comes from
the spindle, the taper fit, the holder, the collet and the tool itself. With total indicated
runout `t` on a 2-flute cutter, one edge sits `t` further out than the other, so the chip
thickness per tooth becomes about `fz + t` and `fz − t`.

*Worked example (computed).* `fz = 0.025 mm`, `t = 0.012 mm` (a standard ER collet's 15 µm class
is close to this): the high tooth cuts about 0.037 mm (48 % more than programmed), the low
tooth about 0.013 mm. When `t ≥ fz`, the low tooth stops cutting and the high tooth takes the
whole `2·fz`. For more flutes the pattern is similar: a holder maker describes a six-flute cutter
with more than 0.001" (25 µm) runout where "only about three teeth" do the work, and gives the
rule "one tenth (.0001") of runout will increase or decrease tool life by 10 %"
([BIG DAISHOWA](https://www.bigdaishowa.com/node/1498); rule of thumb, marketing source). Trade
press reaches the same conclusion ([Modern Machine Shop](https://www.mmsonline.com/articles/the-real-cost-of-runout)).

What CAM can do with this: use the holder's rated runout (or the measured value from a tool
setter, see in-process inspection) to compute the **peak** chip thickness for force, chip-load
and tool-life checks, and to warn when runout approaches the programmed `fz` for small tools.

### 7. Stick-out and stiffness

A tool sticking out of a holder is a cantilever. Its tip stiffness is
`k = 3·E·I / L³` with `I = π·d_eff⁴ / 64`; the fluted part makes the effective diameter smaller
(about 0.8·D is a common first estimate, unverified).

*Worked example (computed).* 12 mm carbide end mill (`E ≈ 600 GPa`), `d_eff = 9.6 mm`,
`I ≈ 417 mm⁴`. At 36 mm stick-out `k ≈ 16 N/µm`; at 48 mm `k ≈ 6.8 N/µm`. Under a 300 N radial
force the tip moves about 19 µm vs 44 µm: a third more stick-out gives 2.4 times the
deflection. The tool's first bending frequency also falls roughly with `1/L²`, which moves
stable chatter-free speeds (cutting physics, machine library).

The holder adds its own compliance in series (`1/k_total = 1/k_tool + 1/k_holder + 1/k_taper +
1/k_spindle + …`, as in the machine library's compliance chain). The holder's nose diameter and
reach decide how short the stick-out can be while still clearing the part, which is why the tool
library already stores the holder profile. **Clamped length** matters too: a shank gripped over
too short a length is less stiff and grips with less torque (minimum insertion lengths are in
holder makers' data; not collected here).

### 8. Vibration

Three vibration sources pass through the holder:

1. **Unbalance:** a force at once per revolution, `F = U·ω²` (section 3).
2. **Runout:** uneven chip loads repeat once per revolution (section 6).
3. **Chatter:** self-excited vibration at the tool-holder-spindle natural frequencies. The
   tool-tip frequency response depends on the holder's mass, stiffness and damping, which is
   why receptance coupling (RCSA) measures the spindle once and models tool and holder as beams
   (tool library and machine library files). Hydraulic chucks and purpose-built damped holders
   add damping (maker claims; magnitude not collected here).

The holder library therefore needs a **beam model** of the holder (profile, material, mass) and,
where known, measured damping, so the chatter model can couple it.

### 9. The spindle side: drawbar, taper and pull studs

**Drawbar force (fact).** Suggested drawbar pull for CAT tapers: CAT30 5.3 kN, CAT40 10.2 kN,
CAT45 17.8 kN, CAT50 22.2 kN; recommended HSK-A/C clamping force: HSK50 11 kN, HSK63 18 kN,
HSK80 28 kN, HSK100 45 kN ([Tooling & Accessories Crockford](https://tacrockford.com/product/typical-power-drawbar-clamping-forces/);
it notes machine builders design higher or lower). Haas lists drawbar force ranges per machine:
for most 40-taper VMCs 6.5 to 10.2 kN (1,460 to 2,300 lbf), for 50-taper 12.5 to 19.6 kN, for
HSK-A63 spindles 12.6 to 18 kN, and states that a loss of about 50 % of nominal clamp force
points to a cracked or broken disc spring
([Haas drawbar force table](https://www.haascnc.com/service/troubleshooting-and-how-to/reference-documents/drawbar-force-reference-table.html)).

**What the drawbar must resist.**

- *Axial pull down* (tool and holder pulled out of the spindle): helical milling, coolant
  pressure behind the pull stud, the holder's weight on heavy tools. Pushing loads (drilling
  thrust) press the holder into the taper and do not need the drawbar.
- *Bending moment* at the gauge line: radial cutting force × distance from the gauge line. When
  the moment is large enough the taper opens on one side, the holder rocks and frets.
  *Order-of-magnitude estimate (opinion, ignores taper wedge and friction):*
  `M_open ≈ F_drawbar · r_gauge`. CAT40 gauge diameter 44.45 mm, at 10 kN: about 220 N·m, so a
  radial force at 100 mm from the gauge line of about 2.2 kN; with a worn drawbar at 6.5 kN,
  about 1.4 kN. Dual-contact and HSK interfaces raise this (the BIG-PLUS figure in section 2).
  This should be replaced by maker data or a measured interface stiffness when available.
- *Torque* at the taper: carried mostly by taper friction, with drive keys as backup (unverified
  split; maker data needed).

**Pull studs (retention knobs).** The pull stud transmits the whole drawbar force into the
holder at every tool change.

- *Tightening torque.* Rego-Fix lists **maximum** pull stud tightening torques of 25 N·m for
  SK/BT/CAT 30, 50 N·m for 40 and 100 N·m for 50, warning that more can deform the taper and
  spoil runout ([Rego-Fix](https://usa.rego-fix.com/rego_torque.pdf)). A tooling seller lists
  20–25, 45–55 and 75–90 ft·lb (27–34, 61–75 and 102–122 N·m) for the same sizes
  ([Maritool FAQ](https://www.maritool.com/faq.php?cID=5)). The two disagree by about 1.5×, which
  is the clearest reason for **per-holder, per-stud maker values** in the library and a
  conservative default when unknown.
- *Over-torque.* Over-tightening or installing with a hammer "can cause the knob to bulge out the
  small end of the taper" ([BIG DAISHOWA, "Pull Studs: Under Pressure"](https://www.bigdaishowa.com/en/node/1241)),
  which spoils the taper fit, causes fretting and vibration, and loads the tool changer
  ([MSC](https://www.mscdirect.com/betterMRO/metalworking/look-after-retention-knobs-pull-studs)).
- *Fatigue and life.* Pull studs fatigue with each clamp cycle and corrode in coolant. Guidance:
  replace about every three years (BIG DAISHOWA; Maritool), yearly for multi-shift or heavy use
  (Maritool), or every 6,000 to 8,000 hours (Haas, via MSC). MSC suggests a drawbar force gauge
  check every month or two and date-coding each knob. A fractured stud at speed is described as
  a "spectacular crash" (MSC).
- *Matching.* Pull stud profile (angle, length, thread: CAT40 5/8"-11, CAT50 1"-8 per Haimer)
  must match the machine's gripper. Grip forces have reportedly risen from about 1,600 lb to
  about 2,500 lb (7 to 11 kN) over 20 years (BIG DAISHOWA).

These steep-taper concerns do not apply to HSK, Capto or KM-style interfaces, which have no
separate pull stud (MSC).

### 10. Holder fasteners

Every screw in the tool assembly has a tightening torque that the maker specifies: ER nuts
(section 4), pull studs (section 9), shell mill and face mill arbor screws, boring head clamp
screws, insert screws, side-lock screws, coolant set screws on flange-fed holders. Too little
torque lets parts move or leak; too much distorts seats, strips threads or (for pull studs)
swells the taper. A practitioner notes that on flange-fed through-coolant holders the flange set
screws must be tight "or coolant may be forced into the spindle" ([eMastercam forum](https://www.emastercam.com/forums/topic/87606-thru-spindle-sealed-collets/)).
The physics here is ordinary bolted-joint mechanics; what CAM can add is **putting the right
value in front of the person holding the wrench** (setup sheet, tool tag) and recording that it
was applied (opinion; values per fastener were not collected for this file).

### 11. Through-spindle coolant

**The path.** Coolant pump → rotary union at the back of the spindle → drawbar bore → pull
stud bore (sealed by an O-ring on the stud pilot; a 0.01" oversize pilot can stop it sealing,
[Maritool](https://www.maritool.com/faq.php?cID=5)) → holder (ASME B5.50 Form AD: through the
pull stud; Form AF: through the flange collar, [Haimer](https://haimer.com/us/products/tool-holders/steep-taper-ASME-B5.50-CAT40-CAT50))
→ collet or chuck seal → tool coolant holes.

**Pressure ratings (fact).** One rotary union is rated 103 bar (1,500 psi) and 20 l/min
([Deublin via MROSupply](https://www.mrosupply.com/products/96769/as-markdown/)). Rego-Fix rates
its metallic sealed ER collets and coolant nuts to 2,000 psi (about 138 bar), and its sealing
discs to "over 2000 psi" ([Rego-Fix coolant flyer](https://www.productivity.com/wp-content/uploads/2020/04/Rego-Fix-Coolant-Flyer-3-26-20-Interactive.pdf));
sealed collets need nominal-diameter shanks (0.5 mm range). Users report slight leaks at
250 psi (17 bar) with some brands, run up to 1,200 psi (83 bar) with others, and stress that
"the tool has to go to the bottom of the collet or it won't seal" (eMastercam forum).

**Physics that CAM should know (estimates computed for this file; not found stated in sources):**

- *Push-out force on the tool:* coolant pressure acts on the sealed shank end, `F = p · A`. At
  70 bar (7 N/mm²) on a 20 mm shank (314 mm²) this is about **2.2 kN** pushing the tool out of
  the holder, more than three times the helical pull estimate in section 5. It adds directly to
  the pull-out check.
- *Push-out force on the holder:* the same pressure on the pull stud seal area works against the
  drawbar. With a 16.6 mm pilot (217 mm²) at 70 bar: about 1.5 kN, about 15 % of a 10 kN CAT40
  drawbar.
- *Centrifugal pressure rise:* in radial passages the pressure grows by `ρ·ω²·r²/2`; at
  20,000 rpm and 20 mm radius about 9 bar. Small, but it adds to the seal load at speed.
- *Dry running:* some rotary unions must not spin without coolant (dry-running unions exist,
  e.g. [US6308734](https://patents.google.com/patent/US6308734)). The machine library should
  store whether the union allows dry running or MQL, and the simulation gate should refuse a
  through-coolant-off high-speed move where it does not (opinion).

### 12. Non-rotating tools: orientation and alignment

Some tools are held in the spindle (or a turret) but must **not** rotate: keyway and slot
broaching on mills and lathes, shaping (stroke) tools, static lathe tools, single-point boring
bars used with oriented retract. For these, speed and balance stop mattering; **orientation,
alignment and twist** take over.

**Split with the gear and broaching file (agreed).** That file owns the *process*: broach tooth
load, rise per tooth, teeth in cut, stroke force, gullet capacity, stroke-shaping pass plans,
rotary broach feed and thrust. It hands this file a per-operation force vector and the
alignment tolerances the process needs. This file owns the *holder*: spindle orient and lock,
angular alignment of the cutting edge, centre height, twist, holder stiffness, rotary broach
holders and their centring.

**Spindle orientation and lock.** On a mill the spindle is stopped and oriented (`M19`, or
`M19 R<angle>` on some controls) before a broaching stroke ([GoEngineer, CAMWorks broaching](https://www.goengineer.com/en-us/blog/camworks-broaching-turn-setup-and-mill-setup)).
The stroke force acts at the edge, offset from the spindle axis, so it twists the spindle with
`M = F · r_edge`. *Example (computed):* 2 kN at 10 mm offset is 20 N·m that the orient servo or
spindle brake must hold. Whether the spindle can hold this is a **machine library** field
(orient holding torque, mechanical lock yes/no) that public spec sheets rarely give; when unknown
CAM should require a confirmed value or a mechanical lock (opinion).

**Angular alignment of the edge.** The edge must be at a known angle to the machine axes. The
angle of the edge relative to the spindle's orient position depends on the holder: steep-taper
holders have drive slots 180° apart, so whether an assembly can seat in two orientations depends
on the holder and the tool changer (unverified per interface); HSK and Capto have their own
coding. The library should record each interface's **seating symmetry** and the assembly's
measured edge angle, which belongs with the tool offsets.

**Centre height (lathes).** A turning or static broaching edge must sit on the spindle
centreline. A Sandvik guideline quoted by Tormach is within 0.1 mm; an academic rule is
`0.005·R` of the workpiece radius, so small parts need tighter settings
([Tormach](https://tormach.com/articles/tool-centerline-tolerances-turning-expert-techniques)).
The effective rake changes by about `asin(h/R)`: *(computed)* 0.1 mm off on a 10 mm diameter is
about 1.1°; on a 2 mm diameter about 5.7°. Tormach also notes a tool below centre "digs in" as it
flexes. Parting tools should be at or slightly above centre (same source).

**Twist.** A stroke force offset from the holder's axis twists the tool in its holder and the
holder in the spindle or turret. Twist rotates the edge, which moves the centre height and the
slot position; the gear and broaching thread cites a source that slight tool twist shifts centre
height and that the tool should retract about 0.4 mm on the back stroke (their citation; see
their file). The holder check is `M_twist ≤ T_slip(holder)` with a margin, plus a twist
stiffness estimate for the slot-position error.

**Rotary broaching holders.** A rotary broach holder tilts the broach 1° from the spindle axis so
only part of the form cuts at a time; the broach has a little more draft than the tilt. On a
lathe the broach is driven by the rotating part; on a vertical mill an attachment on the
non-rotating part of the spindle stops the broach turning and fixes its orientation. Centring is
set by adjustment; the article gives a final runout of about 0.002" (0.05 mm), and the gear
thread cites 0.0008" (0.02 mm) from another article. Holder speed limits: 8 mm shank holders up
to 3,400 rpm, 1/2" and 3/4" shank holders 800 to 1,200 rpm ([Production Machining](https://www.productionmachining.com/articles/the-basics-of-rotary-broaching)).
Misalignment causes spiralling of the form (same source).

**Static and driven turret tools.** VDI (ISO 10889) and BMT turret holders locate static tools
against the turret face; adjustable holders correct centre height and squareness. Driven (live)
tool holders are a rotating holder in a static holder: they bring their own speed limit, torque
limit (gear ratio), and coolant path, all of which belong in the library (opinion; values not
collected here).

## Accuracy

- **Published holder data is mostly limits, not models.** Makers publish maximum speeds,
  balance grades, nut torques and sometimes transmittable torque, but rarely torque vs speed,
  pull-out force, or stiffness. The friction models above are useful for missing values and for
  explaining limits; they are not accurate to better than perhaps a factor of 2 because friction
  coefficients vary with oil, finish and wear (opinion).
- **Sources disagree.** Pull stud torque for a 40 taper is a 50 N·m maximum in one source and
  61–75 N·m in another; ER32 bearing-nut torque appears as 55–75 N·m in a guide and differs by
  nut type in the maker's table. The library must record *whose* value it holds.
- **Measured beats rated.** A holder's rated runout and balance describe the holder alone. The
  assembly's real runout and unbalance (tool, nut, collet, pull stud, spindle) can be measured on
  a presetter and balancing machine, and those values should override the catalogue.
- **Static checks miss walk-out.** Pull-out under rotating bending is not predicted by the static
  model; treat it as a flagged risk with a measured check after the cut.
- **Drawbar force drifts.** Disc springs wear; a 50 % loss is a failure. The capacity used by CAM
  should come from the last measured value where the shop has a gauge.

## Programs

- **CAM tool libraries** treat holders mainly as geometry for collision checking and simulation
  (see the tool library file: holders as profile point lists; users drawing holders by hand
  because catalogue items lack geometry). In the public sources reviewed for this file, no CAM
  program checks clamping torque, pull-out, balance speed, drawbar capacity or coolant pressure
  against the toolpath.
- **ISO 13399** has property symbols for some holder data (for example a maximum speed and a
  torque value, as noted in the cutting physics file), so vendor packages can carry limits; how
  many vendors fill them for holders was not checked (open question).
- **Holder makers** publish torque tables (Rego-Fix), torque fixtures (TORCO-BLOCK), coolant
  pressure ratings, and balance grades; **machine makers** publish drawbar force ranges and
  measurement procedures (Haas). These are not linked to CAM.
- **Presetters and balancing machines** measure runout, length and unbalance of assemblies; the
  in-process inspection file describes how measured values could flow back.

## Complaints and requests

1. **Tools pulling out in heavy roughing** with precision friction holders, and side-lock holders
   that stop pull-out but ruin runout and tool life ([CTE](https://ctemag.com/?p=100021)).
2. **Coolant collets leaking** at moderate pressure, and chips packing behind sealed collets on a
   turning centre ([eMastercam forum](https://www.emastercam.com/forums/topic/87606-thru-spindle-sealed-collets/)).
3. **Pull studs installed with a hammer or breaker bar**, swelling tapers and failing by fatigue
   ([MSC](https://www.mscdirect.com/betterMRO/metalworking/look-after-retention-knobs-pull-studs);
   [BIG DAISHOWA](https://www.bigdaishowa.com/en/node/1241)).
4. **Taper-only holders bellmouthing spindles** on older machines at speed ([CTE](https://ctemag.com/?p=96096)).
5. **Runout quietly eating carbide** ([BIG DAISHOWA](https://www.bigdaishowa.com/node/1498);
   [Modern Machine Shop](https://www.mmsonline.com/articles/the-real-cost-of-runout)).
6. **No dedicated broaching operation in CAM**; users build it from contour or turning cycles
   with custom post processors ([GoEngineer](https://www.goengineer.com/en-us/blog/camworks-broaching-turn-setup-and-mill-setup)).

> **[TODO Jason: personal experience]** Which of these you have hit, and any you would add.

## What users need

### Jason's request

- **Limit maximum RPM by tool holding style.** Every holder style has its own speed limit, and
  the toolpath must never exceed the lowest one in the assembly.
- **Know the retainer torque.** ER32 nut torque (and the different value for bearing nuts),
  pull stud torque, holder fastener torques: CAM should state them, not leave them to feel.
- **Cover the whole physics of tool holding:** vibration, runout, tool slip, pull-out, stick-out,
  spindle pull force, pull stud torque, fastener torque, through-spindle coolant, and alignment
  of non-rotating tools such as broaches.

### A physics-based holder library

Holder entries follow the same pattern as the tool, workholding and machine libraries: a
shared catalogue item with the maker's values and sources, a shop's measured overrides, and an
assembly that combines them.

```
HolderItem       id, vendor, part_number, style (er_collet | shrink | hydraulic | milling_chuck |
                 side_lock | whistle_notch | form_lock | arbor | tapping | rotary_broach |
                 static_turret | driven_turret | ...),
                 spindle_interface (CAT40 | BT40 | BIG-PLUS 40 | HSK-A63 | C6 | ...),
                 profile (for collision and beam model), mass_kg, material,
                 rated_rpm, balance_grade_G, balance_rated_rpm, residual_unbalance_gmm,
                 runout_um_at (value, measured at L),
                 coolant_form (none | through_stud | flange | both), coolant_max_bar,
                 bore_or_collet_range_mm, min_clamp_length_mm, shank_tolerance,
                 torque_capacity: [(shank_d, dry_Nm, oily_Nm, rpm)],      # maker data if any
                 pullout_capacity: [(shank_d, N)],                          # maker data if any
                 form_lock (none | weldon | whistle | grooves),
                 stiffness: k_radial_N_per_um at L, damping (optional),
                 fasteners: [(name, size, torque_Nm, source)],
                 seating_symmetry (1 | 2 | ...), sources[]

NutItem          collet_series (ER32 ...), type (standard | bearing | mini | high_speed | coolant),
                 rated_rpm, unbalance, coolant_max_bar,
                 torque_table: [(bore_range_mm, Nm, source)]

PullStudItem     interface, profile (gripper angle, length), thread, max_torque_Nm (source),
                 coolant_bore, installed_date, serial, cycles (optional)

Assembly         (from the tool library) cutter + holder + nut/collet + pull stud + stick-out,
                 plus measured: runout_um, unbalance_gmm, length, applied_nut_torque_Nm,
                 applied_stud_torque_Nm, edge_angle_deg (non-rotating), who and when
```

### The limits the holder gives the solver and the simulation gate

| Limit | Formula or source | Type |
|---|---|---|
| `n_max_holder` | min(rated rpm, nut rpm, interface rpm, `G·9549·m/U`, tool `RPMX`, spindle, union) | **Hard** (slider never moves it) |
| Torque | `Mc ≤ T_slip / SF_slip` | Soft (slider moves SF within floors) |
| Pull-out | `F_axial,tool + p·A_shank ≤ F_pullout / SF_pull` | Soft, plus a walk-out **flag** |
| Taper opening | `F_radial · L_gauge ≤ M_open / SF_taper` | Soft |
| Drawbar | `F_axial,down + p·A_stud ≤ F_drawbar,measured / SF_db` | Soft |
| Deflection | stick-out beam + holder + chain (machine library) | Soft (finishing tolerance is hard) |
| Runout | peak chip `fz + t` used in force and tool-life checks | Model input |
| Coolant | `p ≤ min(union, stud, holder, collet/seal)`; no dry spin if union forbids | **Hard** |
| Fastener torques | maker values to the setup sheet | Instruction |
| Non-rotating | orient holding torque ≥ `F·r_edge · SF`; centre height and angle within the process tolerance | **Hard** for the orient check |

The CAM Physics System's solver takes the **lowest** limit along the chain (tool, holder,
spindle, machine axes, workholding, stock) and reports which link is binding. For the holder
that report should read like: "Feed limited by ER32 collet slip torque (estimated 160 N·m,
SF 2.0) at 80 N·m cutting torque; a shrink-fit holder or a bearing nut at the rated torque
would lift this."

### The slider

Following the cutting physics file's conservative-to-aggressive slider: it moves the soft
safety factors (slip, pull-out, taper opening, drawbar) between a conservative value and a floor
it never relaxes past (suggested starting points, opinion: SF 2.5 → 1.5 for slip and pull-out,
3.0 → 2.0 for drawbar and taper opening, because a holder leaving the spindle is far worse than
a scrapped part). Speed, balance, coolant pressure and fastener torques are **hard limits**.

### Novice-safe defaults

- Unknown holder → assume a standard ER collet with standard-class runout (15 µm), G6.3 balance,
  the lowest common interface speed, and the conservative slider end.
- Unknown nut torque → show "torque unknown: use the maker's table", never guess a value the
  operator might apply.
- Unknown pull stud torque → show the lower of the published values (Rego-Fix maximums) as the
  ceiling.
- Unknown drawbar force → use the low end of the machine maker's range, and suggest a gauge
  check (machine health file).

### On the setup sheet and the tool tag

- The torque to apply to each nut, stud and fastener, the wrench or fixture, and a field for the
  operator to confirm it was applied (the same recorded-approval pattern as the workholding
  overrides).
- The assembly's maximum speed and which limit sets it.
- Coolant: required pressure, whether the collet must be sealed, "tool to the bottom of the
  collet".
- Non-rotating tools: orient angle, centre height check, retract on back stroke.

### Closing the loop

- Tool length measured before and after an operation: a change means slip or pull-out
  (in-process inspection file).
- Runout and unbalance measured on a presetter or balancing machine replace catalogue values.
- Drawbar force gauge readings and pull stud age feed the machine health file; a drawbar below
  50 % of nominal blocks the job.

## Glossary

| Term | Also called | Meaning | Note |
|---|---|---|---|
| Holder | tool holder, chuck | The part that connects a cutting tool to the spindle | |
| Collet | ER collet | A slotted sleeve squeezed onto the shank by a nut | ER sizes ER8 to ER50 |
| Bearing nut | ball-bearing nut, low-friction nut | An ER nut with a bearing between nut and collet | Needs less wrench torque for the same grip |
| Shrink-fit | heat-shrink | A holder whose bore grips by interference after heating and cooling | |
| Hydraulic chuck | hydro-expansion chuck | A holder that grips via oil pressure on a thin membrane | |
| Side-lock | Weldon | A holder that drives the tool by a screw on a flat | |
| Runout | TIR | Radial wobble of the tool edges about the spindle axis | |
| Unbalance | imbalance | Mass × offset of the centre of mass, in g·mm | |
| Balance grade | G-grade | ISO 21940-11 quality grade, in mm/s | G2.5 common for high speed |
| Stick-out | gauge length, projection | Length of the tool beyond the holder | Deflection ∝ L³ |
| Pull-out | walk-out, creep | A tool moving axially out of the holder during cutting | |
| Drawbar | | The spindle mechanism that pulls the holder into the taper | |
| Pull stud | retention knob | The stud on a steep-taper holder that the drawbar grips | |
| Dual contact | face contact, BIG-PLUS | Taper and flange face both touch the spindle | |
| Through-spindle coolant | TSC, CTS | Coolant delivered through the spindle and tool | |
| Rotary union | coolant union | The seal that passes coolant into the rotating spindle | |
| Spindle orient | M19 | Stopping the spindle at a set angle | Needed for non-rotating tools |
| Centre height | centreline | The edge's height relative to the lathe spindle axis | |

## Open questions

1. Do holder makers publish **torque vs speed** for shrink-fit and hydraulic chucks, so the grip
   loss at speed can be a curve instead of a guess?
2. Which holder data fields in ISO 13399 / GTC packages are actually filled for holders (speed,
   torque, balance, coolant pressure)?
3. Which pull stud torque should be the default when maker data is missing, given the 50 N·m vs
   61–75 N·m disagreement for 40-taper?
4. How should walk-out risk be quantified: is there public test data on pull-out vs helix,
   engagement and holder type?
5. Is the `M_open ≈ F_drawbar · r_gauge` estimate good enough as a fallback, or should taper
   opening wait for maker stiffness data?
6. Which machines publish spindle orient holding torque, and is a mechanical lock common?
7. Should coolant push-out (`p·A`) be applied in full, or do seal designs relieve part of it?
8. Minimum clamped lengths per holder style, and how much grip is lost when a shank is clamped
   short.

> **[TODO Jason: personal experience]** Answers to any of these from your shop, especially
> questions 3, 4 and 6.

## Sources

- [Rego-Fix: recommended tightening torque for ER clamping nuts and steep-taper pull studs](https://usa.rego-fix.com/rego_torque.pdf)
- [Rego-Fix coolant flyer (via Productivity Inc.)](https://www.productivity.com/wp-content/uploads/2020/04/Rego-Fix-Coolant-Flyer-3-26-20-Interactive.pdf)
- [AIMS Industrial: ER collet guide](https://aimsindustrial.com.au/en-nz/blogs/product-guides/er-collet-guide)
- [Cutting Tool Engineering: Toolholder and tool balance](https://ctemag.com/?p=96427)
- [Cutting Tool Engineering: Alternatives to steep tapers](https://ctemag.com/?p=96096)
- [Cutting Tool Engineering: Haimer pull-out case article](https://ctemag.com/?p=100021)
- [Cutting Tool Engineering: Shrink-fit toolholder fundamentals](https://www.ctemag.com/articles/shrink-fit-toolholder-fundamentals)
- [Haimer: steep taper ASME B5.50 CAT40/CAT50 technical specifications](https://haimer.com/us/products/tool-holders/steep-taper-ASME-B5.50-CAT40-CAT50)
- [Haimer: Safe-Lock](https://haimer.com/us/products/solutions/safe-lock)
- [Fabricating & Metalworking: Prevent tool pull-out](https://fabricatingandmetalworking.com/prevent-tool-pull-out-in-high-performance-machining-applications/)
- [Manufacturers' Monthly: SCHUNK TENDO E compact](https://www.manmonthly.com.au/product/hydraulic-expansion-toolholders/)
- [BIG DAISHOWA: The one tenth = 10 % rule](https://www.bigdaishowa.com/node/1498)
- [BIG DAISHOWA: Pull studs, under pressure](https://www.bigdaishowa.com/en/node/1241)
- [Modern Machine Shop: The real cost of runout](https://www.mmsonline.com/articles/the-real-cost-of-runout)
- [Tooling & Accessories Crockford: typical power drawbar clamping forces](https://tacrockford.com/product/typical-power-drawbar-clamping-forces/)
- [Haas: drawbar force reference table](https://www.haascnc.com/service/troubleshooting-and-how-to/reference-documents/drawbar-force-reference-table.html)
- [MSC: Why you should look after your retention knobs](https://www.mscdirect.com/betterMRO/metalworking/look-after-retention-knobs-pull-studs)
- [Maritool: pull stud FAQ](https://www.maritool.com/faq.php?cID=5)
- [eMastercam forum: thru-spindle sealed collets](https://www.emastercam.com/forums/topic/87606-thru-spindle-sealed-collets/)
- [Deublin 1101-235-238 rotary union (MROSupply listing)](https://www.mrosupply.com/products/96769/as-markdown/)
- [US6308734B1: dry running coolant union](https://patents.google.com/patent/US6308734)
- [Production Machining: The basics of rotary broaching](https://www.productionmachining.com/articles/the-basics-of-rotary-broaching)
- [GoEngineer: CAMWorks broaching, turn and mill setup](https://www.goengineer.com/en-us/blog/camworks-broaching-turn-setup-and-mill-setup)
- [Tormach: tool centreline tolerances for turning](https://tormach.com/articles/tool-centerline-tolerances-turning-expert-techniques)
