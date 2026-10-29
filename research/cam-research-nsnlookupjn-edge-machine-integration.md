# CAM research: connecting SolidCraft Edge to every machine, ranked by how many shops have one

Research for [#1621](https://github.com/tangency-labs/solidcraft/issues/1621) (Research Request
for Plugin: CAM). This file covers one part of that request: **how SolidCraft Edge (the shop-floor
box at the machine) gets status and real-time data out of CNC machines, additive machines and
CMMs, brand by brand, ranked by installed base so the work reaches the most users first**. Haas is
the worked example, in depth, across every Haas control generation.

Jason's questions, in his words:

> Research how a solidcraft edge mini-pc or PI could extract data from all generations of Haas
> machines to get status and realtime data

> Research if we can design an electornic module to plugin to the Haas controllers to get
> realtime data. This is a read-only interface

> Research who at Haas may be an engineering contact to cooperate on a factory integrated
> interface

> Research which CNC machines are most popular in the world by installed user base. Do the same
> analysis for integrating edge as a bolt-on or factory installed for each machine. [...] Format
> the the results by brand and machine with references to the popularity rank so we can
> prioritize our work to cover the most users quickly. Consider additive machines as well.
> Consider CMM and Inspection machines as well.

> The edge can connect to the machine controller via network, serial, industrial bus and direct
> connection to the controller electronics. Consider also tapping into existing cables or
> connections to do inspection of existing data and memory modules

> Maybe we could fit a plug on top of a microchip or cpu to directly read data from the
> motherboard?

> Ideally we would suggest a MCP project to each machine manufacturer and make everything
> standard

It builds on a set of files from the same author and does not repeat them:

| Topic | File | What this file takes from it |
|---|---|---|
| Machine interfaces and MCP | [cam-research-nsnlookupjn-machine-interfaces.md](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-fz0g0g/research/cam-research-nsnlookupjn-machine-interfaces.md) | what each control maker's interface exposes (FOCAS, OPC UA, MTConnect, Heidenhain DNC, Okuma THINC and others), rate tiers A–D, the write safety model, the MCP-on-the-machine tool set |
| In-process inspection and SolidCraft Edge | [cam-research-nsnlookupjn-in-process-inspection.md](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-y2w06n/research/cam-research-nsnlookupjn-in-process-inspection.md) | the SolidCraft Edge device definition (Raspberry Pi 5, Rust, five functions), the retrofit sensor kit, aligning data with G-code blocks |
| Machine health | [cam-research-nsnlookupjn-machine-health.md](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-9w3gq9/research/cam-research-nsnlookupjn-machine-health.md) | what a load or vibration signal means |
| Machine library | [cam-research-nsnlookupjn-machine-library.md](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-machine-library/research/cam-research-nsnlookupjn-machine-library.md) | "Signals the machine publishes" in each machine entry |
| Additive process physics | [3d-printing-research-nsnlookupjn-process-physics.md](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-j72y6h/research/3d-printing-research-nsnlookupjn-process-physics.md) | what an additive machine's signals mean (melt pool, flow, temperatures) |
| High-security work | [security-research-nsnlookupjn-high-security.md](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-2fa4g1/research/security-research-nsnlookupjn-high-security.md) | SolidCraft Edge on a segmented OT network, wired only, no inbound ports |

**Ownership (so nothing is said twice):** this file owns **the popularity ranking, the per-brand
integration plan (bolt-on or factory-installed), the Haas deep dive by control generation, the
connection routes into a controller (network, serial, industrial bus, electronics, cable taps,
memory) and the read-only plug-in module**. What each control's software interface exposes in
detail stays in the machine interfaces file; this file links to it and adds only what changes the
ranking or the plan.

Licence: MIT OR Apache-2.0, the same as SolidCraft (see the Licence section of CONTRIBUTING.md).

## About this research

- **Author:** Jason ([@nsnlookupjn](https://github.com/nsnlookupjn)). Ideas and direction by
  Jason; source research and writing done with an AI assistant (Claude) in October 2026.
- **How it was produced:** web searches of public sources (Haas service pages, maker product
  pages, Fanuc and EOS press releases, CONTEXT market reports, the VDW press office, monitoring
  vendors' connectivity guides, Hexagon's public PC-DMIS help, the US Copyright Office), each claim
  checked against the cited page where possible. Several Haas pages refused automated fetching;
  where a Haas fact comes from a third-party page instead, it says so. No commercial CAD/CAM
  program was run, no vendor SDK was downloaded, no machine was connected, and no text is copied
  beyond short quotes.
- **GPL code was not read.** Some community Haas collectors are GPL (for example the TrakHound
  "MTConnect Agent For Haas", GPL-2.0 per its repository); only their README summaries were seen,
  and no source. The same applies to Klipper/Moonraker, OctoPrint and LinuxCNC (Tormach
  PathPilot), which are named here as things a machine may run, never as code to read.
- **Market numbers are estimates.** No paid market report was bought. Shares come from press
  releases (fact for what the maker states), free summaries of analyst reports, and trade articles
  that do not cite their data (marked). The ranking is good enough to order work, not to quote as
  a market study.
- **Units:** millimetres, seconds, milliseconds, hertz, kilohertz (kHz), volts.
- **Marking:** "(unverified)" marks a claim not confirmed against a source. **Fact**, **opinion**
  and **estimate** are labelled where they mix.

**Author's experience (personal experience, not a cited source):**

- **Shop floor since 1987:** a machine shop apprenticeship from 1987 to 1991, starting at 16 on
  manual machines, then writing G-code at the control on Milacron-era machines with green
  screens.
- **CAD/CAM over about 35 years:** at Ford, Unisurf, Siemens, Sketchpad and NX, with programs on
  punch cards, then tape, then floppy disks; later CATIA, AutoCAD and Pro/Engineer. Currently
  fluent in Fusion, Mastercam, SOLIDWORKS and NX.
- **Engineering and software:** BTEC HND in Electrical and Mechanical Engineering, certified
  welder, Masters in Engineering, PhD in Materials Science. Programming since age 8 (machine code,
  BASIC; later C++, C#, assembler, web and databases).
- **Current work:** owns several engineering and software companies, focused on engineering
  military systems and making and supplying military spare parts; has produced flight-safety
  components, nuclear submarine parts, satellite parts and weapons-system parts.

## Summary

**The fastest way to reach the most machines is two open standards plus one universal bolt-on
kit, then Fanuc and Haas, then everyone else.** In order (opinion, from the ranking below):

1. **MTConnect and OPC UA (umati) readers** in SolidCraft Edge. They are free to read, already
   built into many recent controls (Haas NGC, Mazak, Okuma, Hurco, Mitsubishi, Siemens and
   Heidenhain options, DMG Mori gateways, Stratasys F900), and one reader covers many brands.
2. **A universal, read-only bolt-on sensor module** (current sensors, vibration, stack-light and
   relay inputs) that works on **any** machine of any age. It is the only route for the very large
   number of older machines that have no usable data port, and the only route to kHz-rate data
   on most machines.
3. **Fanuc** (the largest installed base by far: 5 million CNCs built by February 2022, Fanuc
   press release) through FOCAS2 over Ethernet, or MTConnect/OPC UA where the option is fitted.
4. **Haas** (over 200,000 machines by about 2018; the most common brand in US job shops, which is
   where SolidCraft's early CAM users are likely to be) through Machine Data Collection, MTConnect
   and DPRNT, generation by generation (deep dive below).
5. **Siemens SINUMERIK** (OPC UA option), then **Mazak, Okuma, Heidenhain, Mitsubishi**, then
   Chinese domestic controls, then legacy and small brands.
6. **Additive:** Bambu Lab, Prusa and Formlabs first (most units; documented local interfaces),
   then the Klipper-based printers through their network API, then industrial machines (EOS,
   Stratasys, HP, metal PBF) through their OPC UA/MTConnect/REST options.
7. **CMMs and inspection:** read **result files** (QIF, DMIS output, Q-DAS/AQDEF, maker reports)
   from Hexagon, Zeiss and Mitutoyo software first. That covers most CMMs read-only with no
   electronics at all. Live CMM state through I++ DME needs care, because I++ DME is a control
   protocol.

**On Jason's hardware questions:**

- **A read-only plug-in module for Haas is practical**, as long as it plugs into the documented
  points: the Ethernet or RS-232 data port, the user and status relays, and clamp-on sensors on
  the motor leads. Software alone cannot make the Haas data port read-only, because it also
  accepts a write command (`?E`). So the module enforces read-only itself, with an allow-list of
  query commands and no output hardware at all (see
  [The read-only plug-in module](#the-read-only-plug-in-module)).
- **Tapping existing cables works well on some links and badly on others.** It is safe and useful
  on RS-232 drip-feed (DNC) lines, Ethernet links through a mirror port, RS-485/Modbus links to
  peripherals, and analog load outputs. It is risky on servo feedback (encoder) lines and fibre
  links. A clip on the CPU or memory chip is **not recommended**: the parts are BGA packages with
  no pins to clip, the buses run at hundreds of MHz, and a probe can crash a live servo control.
  It also means reverse engineering the maker's firmware. Reading the control's own **backup
  files** gets most of the same data safely (see [Connection routes](#connection-routes-into-a-controller)).
- **A factory-integrated interface needs Haas.** The ask is small and specific: a documented,
  read-only, high-rate stream (spindle load, axis loads, actual feed, block number) on the
  existing Ethernet port. The route in is the local Haas Factory Outlet first, then Haas Automation
  engineering in Oxnard (see [Working with Haas](#working-with-haas-on-a-factory-integrated-interface)).
- **SolidCraft Edge as an open hardware project inside SolidCraft** (Jason's proposal): an open
  board, bare-metal embedded Rust firmware for the real-time I/O and read-only enforcement, and a
  minimal Linux image running only Rust services on the Pi (see
  [SolidCraft Edge as an open hardware project](#solidcraft-edge-as-an-open-hardware-project-inside-solidcraft)).
- **CAM Physics sets the limits, the Edge learns** (Jason): the post writes per-tool load limits
  for the control's own stop or autofeed, and the Edge's fast load and vibration data teach
  SolidCraft each tool and setup, so the next program is better. The Edge never writes to the
  control (see [From CAM physics to the control and back](#from-cam-physics-to-the-control-and-back-load-limits-and-learning)).
- **Every factory feature, every cut** (Jason): novices get the machine's best features switched
  on and explained by default; pros see and override all of them (see
  [Every factory feature, every cut](#every-factory-feature-every-cut)).
- **Keeping up with control software** (Jason): versions tracked as catalogue data fed by Edge
  discoveries and release notes, and an **upgrade advisor** that re-runs the shop's own recent
  jobs with a new feature on and shows hours saved, cost and payback (see
  [Keeping up with control software](#keeping-up-with-control-software-and-a-reason-to-upgrade)).
- **Sensors that connect to the Edge** (Jason): from direct analog, Modbus and IO-Link to PoE,
  Wi-Fi, Bluetooth LE, Zigbee and LoRaWAN, with the rate each really gives, an industrial and a
  budget choice per signal, a **pricing and where-to-buy table** with example kits, and a wired-only profile for secure shops (see
  [Sensors that connect to the Edge](#sensors-that-connect-to-the-edge)).
- **Machine tables** (Jason): control generations and software versions with their monitoring
  and control features, builder and model tables, user-base estimates by continent, and the
  capability record the machine library needs (see
  [Machine tables](#machine-tables-controls-versions-features-and-user-base)).
- **Renishaw gets its own pitch**, because its probes sit on almost every brand in the ranking:
  one documented probe-result record on every control, and Renishaw Central serving the same
  read-only profile (see [A pitch to Renishaw](#a-pitch-to-renishaw)).
- **"Make everything standard" with MCP (Model Context Protocol):** MCP is the right universal
  standard for *AI agents*; MTConnect and OPC UA stay the data model underneath it, because makers
  already ship them. The machine interfaces file already sketches a
  machine-tool MCP tool set with no motion tools. This file adds a **read-only-first "machine MCP
  profile"**, mapped onto MTConnect and umati names so makers can serve it from data they already
  have, and a per-maker pitch in ranking order (see [One standard: an MCP profile for machines](#one-standard-a-model-context-protocol-mcp-profile-for-machines)).

## Fundamentals

### Bolt-on or factory-installed: the two ways in

| | **Bolt-on** (shop fits it, no help from the maker) | **Factory-installed** (the maker fits or supports it) |
|---|---|---|
| What it is | SolidCraft Edge on a documented port, plus add-on sensors | a maker option turned on (MTConnect, OPC UA, FOCAS), an app on the control, or a module the maker fits in the cabinet |
| Data rate | port polling 1–10 per second; add-on sensors to 50 kHz | whatever the maker exposes; usually 1–10 per second, rarely 500 Hz+ (Siemens Edge) |
| Data meaning | add-on sensors need calibrating (amps are not "spindle load %") | the control's own values with the control's units |
| Cost to the shop | the box and sensors; an electrician's hour for clamps | sometimes a paid software option per machine |
| Warranty | documented ports: no effect; anything inside the cabinet: ask the dealer | covered |
| Best for | old machines, and fast signals on any machine | new machines, and anything SolidCraft should rely on long term |

**Rule of thumb (opinion):** use the factory interface for *what the machine is doing* (program,
block, tool, mode, alarms, offsets), and bolt-on sensors for *how hard it is working, fast*
(current, vibration). Join the two on the Edge's clock.

### Connection routes into a controller

Jason lists four routes plus cable taps and memory. Here is each one, with what it gives, how safe
it is and whether SolidCraft should build it (opinion, except where cited).

| Route | Examples | Gives | Read-only? | Risk to the machine | Build it? |
|---|---|---|---|---|---|
| **1. Network (Ethernet)** | MTConnect HTTP, OPC UA, Haas MDC TCP, FOCAS2, Heidenhain DNC, maker REST/MQTT | state, program, tool, alarms, counters, often loads | depends on the protocol (MTConnect is read-only by design; Haas MDC and FOCAS also accept writes) | low, if the Edge sits on an OT segment and polls gently | **Yes, first** |
| **2. Serial (RS-232 / RS-422 / RS-485)** | Haas classic MDC (RS-232), DNC drip-feed, old Fanuc/Fadal/Milacron, Modbus RTU peripherals | the same as network on older controls; block-by-block progress when the Edge *is* the DNC drip-feeder | queries need a transmit line; enforce read-only in the module | low | **Yes** |
| **3. Industrial bus (fieldbus)** | PROFINET, EtherCAT, PROFIBUS, Modbus TCP, CAN; maker buses (Fanuc FSSB fibre and I/O Link, Siemens DRIVE-CLiQ, Mitsubishi SSCNET fibre, Heidenhain HSCI) | I/O states, drive data, sometimes positions at ms rates | **passive taps only** (mirror port, Ethernet tap, receive-only RS-485) | medium: a wrong active device on a fieldbus can stop the machine | **Open buses yes, as passive taps; maker buses no** without the maker's help |
| **4. Direct to the controller electronics** | user and status relays, M-code outputs, stack-light lamps, analog load outputs on drives, clamp-on current sensors on motor leads | run/idle/alarm, part pulses, spindle and axis current at kHz | yes, if inputs are isolated and there are no outputs | low with isolated inputs and a qualified electrician; never touch safety circuits | **Yes** (this is the bolt-on kit) |
| **5. Taps on existing cables** | Y-cable on a DNC RS-232 line (receive only), Ethernet tap between HMI and NC, RS-485 sniffing, high-impedance receivers on encoder lines | whatever already travels on that cable | yes, if the tap is electrically passive | low on serial and Ethernet; **high on encoder and fibre lines** (see below) | **Serial and Ethernet yes; encoder and fibre no** |
| **6. Memory: backup files** | the control's own backup to USB/network (Haas backup, Fanuc SRAM/parameter backup to a memory card, Siemens archive, Heidenhain backup) | parameters, offsets, tool tables, programs, alarm history, counters | yes (a copy) | none, done with the control's own function | **Yes**: parse documented or plain-text formats |
| **7. Memory: chips and the CPU** | clips or probes on SRAM, flash, the CPU, JTAG | in theory, everything | yes, electrically | **high** | **No** (see next section) |

#### Why not a clip on the CPU or memory chip

Jason asks whether a plug fitted over a microchip or CPU could read data straight off the
motherboard. It is a fair engineering question, and on a 1990s board it was physically possible.
For SolidCraft it is **not recommended**, for six reasons (opinion, with the facts behind it):

1. **No pins to clip.** Modern controller CPUs, memory and FPGAs are ball-grid-array (BGA)
   packages with the contacts underneath. Test clips fit only older leaded packages (DIP, PLCC,
   QFP).
2. **Speed.** Memory buses on a modern control run at hundreds of MHz to GHz. Capturing them needs
   a lab logic analyser, not a Raspberry Pi, and the probe itself loads the signal.
3. **It can crash the machine.** Any extra load or noise on the CPU or memory bus of a running
   motion controller can cause a watchdog reset or servo fault mid-cut. That is a broken tool or a
   crash, the opposite of what the Edge is for.
4. **The data is not labelled.** Raw memory is the maker's internal layout, which changes with
   every software version. Making sense of it means reverse engineering the maker's firmware. That
   conflicts with SolidCraft's clean-room ground rule of independence, and may break the software
   licence the shop agreed to.
5. **Law.** In the US, getting past a technical protection measure is restricted by DMCA §1201.
   The 2024 rulemaking (final rule 25 October 2024) renewed repair exemptions for vehicles,
   vessels and medical devices and added commercial food-preparation equipment, but **no exemption
   for industrial machine tools** (Finnegan summary of the final rule). The EU Software Directive
   (2009/24/EC, Article 6) allows decompiling for interoperability only under narrow conditions.
   This is not legal advice; a shop or SolidCraft would need its own.
6. **Warranty, service and certification.** Opening the logic boards voids support, and for
   defence and aerospace shops it can break the machine's qualified state.

**What to do instead:** the same information is available, safely, through routes 1, 2, 4 and 6.
On boards that have them, **debug or service serial headers** that the maker documents for
service (some drives and I/O boards print diagnostics on a UART) may be worth asking the maker
about. That should be a cooperation item, not a bolt-on.

#### Cable taps: which ones are safe

- **DNC drip-feed line (RS-232), safe and very useful.** On legacy controls that run long programs
  by drip-feed from a PC, the Edge can either sniff the line (receive-only Y-cable) or *become*
  the DNC server. Either way it knows which block has been sent. Allowing for the control's buffer
  (a few hundred bytes to a few kB, per control), that is close to the block being cut, which
  these old controls cannot report any other way (opinion; buffer sizes unverified per control).
- **Ethernet between the HMI PC and the NC, safe to copy.** A managed switch's mirror (SPAN) port,
  or a passive Ethernet tap, copies traffic without joining the network. Open protocols
  (PROFINET, EtherCAT, Modbus TCP, OPC UA) can be decoded; maker-private payloads cannot be decoded
  without reverse engineering (see above), so this is useful mainly for open fieldbus I/O.
- **RS-485/Modbus to peripherals, safe.** Chillers, coolant systems, chip conveyors, bar feeders,
  some spindle drives on older machines: receive-only sniffing gives their states.
- **Servo encoder lines, risky.** Older machines use incremental A/B/Z encoders on RS-422 line
  drivers; a high-impedance differential receiver can in principle listen at MHz rates and give
  true axis position. But any fault in the tap (a short, a ground loop, noise) shows up as an
  encoder alarm or a following-error fault, and on serial absolute encoders (EnDat, BiSS, Fanuc and
  other maker protocols) the data is framed and often proprietary. **Not a bolt-on**; only with the
  maker's engineering approval.
- **Fibre (Fanuc FSSB, Mitsubishi SSCNET), no.** An optical splitter takes a share of the light,
  and the links are designed with little margin; expect link alarms (inference).

### Real-time: what rate each route gives

From the machine interfaces file's tiers (A polling, B high-rate trace, C add-on sensor, D inside
the control), applied to the routes here:

| Route | Typical rate | Tier |
|---|---|---|
| Haas MDC / FOCAS / MTConnect / OPC UA polling | 1–10 values per second per signal | A |
| DNC drip-feed position (legacy) | per block | A (with block-level alignment) |
| Siemens Edge trace, maker servo traces | 500 Hz+ | B |
| Bolt-on current and vibration sensors | 1–50 kHz | C |
| Control's own tool-load monitor (Haas Setting 84, others) | servo cycle | D |

**At 10,000 rev/min with 4 teeth, the tooth-passing frequency is 667 Hz.** Seeing single teeth
needs Tier C sampling (5 kHz or more). Comparing each CAM segment's average load with the
prediction needs only Tier A.

## Popularity ranking

### How the ranking was made

- **Installed base, in units, worldwide**, from public figures where they exist, and estimates
  where they do not (marked). Revenue rankings (where DMG Mori has led since 2015, hpo forecasting
  "Top 100" preview) are a poor guide to *how many shops have one*, so units come first.
- **Two lists for CNC:** by **control family** (what the Edge actually talks to) and by
  **machine builder** (what the shop calls its machine). One control family covers many builders:
  "Fanuc" means a DN Solutions (Doosan), Hyundai WIA, Makino, Matsuura, Kitamura, many Taiwanese
  builders and many others.
- **A "SolidCraft audience" column** adjusts for who is likely to use SolidCraft CAM first: small
  and mid-size job shops, many in the US and Europe, including defence suppliers (opinion; Jason's
  view of the market is that Haas is the most common brand for entry- to mid-level shops).

### CNC controls by installed base

| Rank | Control family | Installed base (evidence) | SolidCraft audience | Main factory interface | Bolt-on route |
|---|---|---|---|---|---|
| **C1** | **Fanuc** (0i, 30i/31i/32i, older 0/16/18/21; on DN Solutions, Hyundai WIA, Makino, Matsuura, Kitamura, other Japanese and most Taiwanese builders) | 5 million CNCs built, February 2022 (Fanuc press release, fact); "50–60 % of CNC machines worldwide" (CNC Machines article, uncited estimate); about 40 % of global CNC systems and 35–40 % of China (industry report, estimate) | very high everywhere | FOCAS2 over Ethernet (library licence); MTConnect and OPC UA options on recent controls | serial DNC on old controls; sensor kit |
| **C2** | **Siemens SINUMERIK** (808/828/840D sl/ONE; on DMG Mori, Hermle, Grob, Chiron, many European and Chinese builders) | 20–25 % (two uncited estimates agree) | high in Europe, medium in the US | OPC UA server option; SINUMERIK Edge (500 Hz capture); MTConnect via adapter | sensor kit; PROFINET I/O passive tap |
| **C3** | **Chinese domestic** (GSK, Huazhong (HNC), KND and others) | about 50 % of CNC units sold in China; GSK near 100,000 sets a year; Huazhong over 300,000 cumulative (industry report, estimates) | low for US/EU and defence shops (sourcing and ITAR concerns); high in China | maker protocols; China's NC-Link standard (not researched further) | sensor kit |
| **C4** | **Heidenhain** (TNC 640, TNC7, iTNC 530 and older; on DMG Mori, Hermle, Spinner, Quaser, many 5-axis and toolroom machines) | 200,000th control in June 2007 (press, fact); 5–15 % share (estimates vary) | medium–high in mould, toolroom and 5-axis | OPC UA NC Server option; Heidenhain DNC (RemoTools SDK) | sensor kit |
| **C5** | **Mitsubishi Electric** (M70/M80/M800; on many Taiwanese and Japanese lathes and mills, EDM and laser) | about 5–12 % (estimates) | medium | MTConnect and maker API (see machine interfaces file) | sensor kit |
| **C6** | **Haas** (own control, Haas machines only) | over 200,000 Haas machines by about 2018 (machinery.co.uk, 200,000th machine; fact); 125,000th CNC in February 2012 (American Machinist, fact) | **very high in US job shops** (Jason; uncited article: "dominates the US job shop market") | NGC: MDC over Ethernet, MTConnect (port 8082), DPRNT; classic: MDC over RS-232 | sensor kit; status relays |
| **C7** | **Mazak Mazatrol** (SmoothX/G/Ai, Matrix and older; Mazak machines) | about 5–7 % (uncited estimate) | high in mid-size production shops | MTConnect built in on Smooth controls; SmartBox gateway | sensor kit |
| **C8** | **Okuma OSP** (Okuma machines) | not found (estimate: a few %) | medium–high in US production | THINC API on the Windows HMI; MTConnect adapter | sensor kit |
| **C9** | **Brother CNC-D00** (SPEEDIO compact machines; older CNC-C00) | not found (large unit counts in electronics manufacturing; unverified) | medium | OPC UA, Ethernet, fieldbus slave options (Brother product page, machine interfaces file) | sensor kit |
| **C10** | **Others**: Hurco WinMax (MTConnect built in, per machine interfaces file), Fagor, Tormach PathPilot (LinuxCNC-based; do not read its source), Centroid, FANUC-licensed and older brands | small each | Hurco and Tormach matter for small US shops | varies | sensor kit |
| **C11** | **Legacy** (pre-network): Fadal CNC88, Cincinnati Milacron Acramatic, early Fanuc 0/6/11, early Haas, Bridgeport/BOSS, Allen-Bradley | large in total, unknown | **high**: many small shops still run them (Jason's own career started on these) | none | DNC drip-feed line; sensor kit |

**Builder view (for the shop's own words).** Shops say "a Doosan" or "a DMG", not "a Fanuc". A
SolidCraft machine library entry should store both builder and control, so that the Edge picks the
connector from the control (opinion). Builders whose controls differ by model or region (DMG Mori:
Siemens, Heidenhain, Fanuc, Mitsubishi; Doosan/DN Solutions: mostly Fanuc, some Siemens; Mazak:
Mazatrol, some Fanuc and Siemens) are the reason.

### Additive machines by installed base

| Rank | Brand | Installed base (evidence) | Factory interface | Bolt-on route |
|---|---|---|---|---|
| **A1** | **Bambu Lab** (X1, P1, A1, H2 series) | 37 % of entry-level (under US$2,500) printer shipments in 2025; entry-level shipments up 26 % in 2025 (CONTEXT, April 2026) | LAN mode and **Developer Mode** (LAN only, no new authorisation check); Bambu Connect; Bambu Local Server (on-premises HTTP API, Windows x86-64 for now); SDK access by application (Bambu Lab wiki) | camera; smart plug power |
| **A2** | **Creality, Elegoo, Anycubic** (FDM and resin) | among the top entry-level vendors, no shares given (CONTEXT); Chinese vendors over 90 % of entry-level units | Klipper-based models expose a network API through Moonraker (GPL project: use only its network interface, never its source); cloud apps; resin printers vary | camera; power |
| **A3** | **Prusa Research** (MK4, Core One, XL) | not in the CONTEXT summary; large in Europe and among makers | **PrusaLink** local REST API on the printer; Prusa Connect (cloud) | camera |
| **A4** | **Formlabs** (Form 4, Form 4L, Fuse) | **38 % of professional (US$2,500–20,000) shipments in 2025** (CONTEXT) | **Formlabs Local API** (needs a server application on a PC on the same network); Formlabs Web API / Developer Platform (cloud) | camera |
| **A5** | **UltiMaker** (S-series, Method) | not found | local REST API on S-series printers (community-documented; unverified); Digital Factory (cloud) | camera |
| **A6** | **Stratasys** (F-series, Fortus, PolyJet J-series, Origin) | among the midrange unit leaders (CONTEXT); a large industrial installed base (not quantified here) | **MTConnect** on F900 and Fortus 450mc; **Connectivity SDK** for Fortus, F900, F123 and PolyJet J3/J5/J7/J8 (Stratasys connectivity page) | stack light; power |
| **A7** | **HP Multi Jet Fusion** | midrange powder-bed growth noted (CONTEXT) | maker software (not researched; unverified) | power; stack light |
| **A8** | **Metal powder bed fusion**: BLT, Eplus3D, Farsoon, ZRapid lead **units**; **EOS** and Nikon SLM Solutions lead **revenue** (CONTEXT, Q4 2025) | EOS: 5,000th industrial machine installed (EOS blog, March 2025) | **EOS: EOSCONNECT Core (OPC UA live data and events; REST Web API; MQTT from Core 1.6; build start over OPC UA needs a Control licence)**; others: OPC UA common (unverified per maker) | cabinet I/O; power |
| **A9** | **3D Systems, Markforged, others** | smaller | maker clouds and APIs (not researched) | camera; power |

**Why additive is easier than CNC (opinion):** most printers already run Linux and a web server, so
the data is a network call away. The hard parts are policy (Bambu's 2025 authorisation
change; cloud-only APIs) and meaning (the additive process file).

### CMMs and inspection machines by installed base

| Rank | Brand | Installed base (evidence) | Factory interface | Bolt-on / read-only route |
|---|---|---|---|---|
| **M1** | **Hexagon** (Brown & Sharpe, Leitz, DEA, Sheffield heritage; PC-DMIS software) | Hexagon and Zeiss together over 23 % of the CMM market in 2023 (GM Insights summary; no per-company split); likely the largest installed base of bridge CMMs (estimate) | PC-DMIS talks to 21 controller types, including **I++ DME client**, Leitz, DEA, Sheffield, Zeiss, Mitutoyo GPIB, Wenzel (Hexagon PC-DMIS help) | **result files**: PC-DMIS reports, DMIS output, QIF, Q-DAS (AQDEF) export |
| **M2** | **Zeiss** (CONTURA, PRISMO, ACCURA; CALYPSO software; PiWeb) | see above | I++ DME server on Zeiss controllers (Zeiss drives I++ DME, VDW); PiWeb | result files (table files, PiWeb, QIF) |
| **M3** | **Mitutoyo** (Crysta-Apex, Strato; MCOSMOS; MeasurLink) | major player (not quantified) | I++ DME (Mitutoyo drives I++ DME, VDW); GPIB on older controllers | result files; MeasurLink database |
| **M4** | **Nikon, Wenzel, LK, Aberlink, Coord3, Keyence, Creaform, FARO** (arms, trackers, optical) | named major players, no shares (GM Insights) | varies; I++ DME on several | result files |
| **M5** | **On-machine probes** (Renishaw, Blum, Marposs, Heidenhain TS on CNCs) | on most modern CNCs | results land in the control's macro variables (read over the CNC interface or DPRNT; inspection file) | as for the CNC |
| **M6** | **Renishaw Equator, gauges, UCC controllers** | not quantified | I++ DME on UCC (Renishaw joined the I++ DME 3.0 work, VDW) | result files |

**I++ DME is a control protocol, not a monitor.** The client sends motion and measurement commands
to the CMM server (VDW: the server handles hardware and calibration; the client handles sequences,
evaluation and programs). Version 2.5 is VDMA standard sheet 8722 (extended 2024); version 3.0,
with Renishaw and OGP joining Zeiss, Hexagon and Mitutoyo, is due at the end of 2026 (VDW press
release, July 2025). A CMM accepts one client at a time in practice (inference), so the Edge
should **not** connect as a second I++ client to a CMM that PC-DMIS or CALYPSO is driving. The
read-only route for CMMs is their **result files**, plus a stack light and power sensing for
state. Live CMM position for a dashboard is a cooperation item for I++ DME 3.0 (opinion).

## The Haas deep dive: every generation

### Telling the generations apart

| Generation | Years (approximate) | How to tell at the control | Data port |
|---|---|---|---|
| **H1. Early Haas control** | 1988 (VF-1 at IMTS '88) to about 2000 | no Setting 143 on the I/O settings page | RS-232 for programs (DNC) only |
| **H2. Classic Haas Control (CHC) with MDC** | about 2001 ("Q codes" standard from 2001, per a MachineMetrics commenter quoting Haas) to about 2016–2017 | Settings, I/O page: RS-232 settings near the top, **Setting 143 "DATA COLLECT" ON/OFF** (MachineMetrics) | RS-232 Machine Data Collection (Q commands); Ethernet on later machines for file sharing |
| **H3. Next Generation Control (NGC)** | began production 2016 (MachineMetrics); MachineMetrics treats "2017 and later" as NGC for data | Settings, Network, Wired Connection page exists; **Setting 143 takes a number (the TCP port)** (MachineMetrics); manual cover says "Next Generation Control" | Ethernet MDC, MTConnect, DPRNT over the network, HaasConnect |

Fact for H1: the 1988 VF-1 control had 128 KB of program memory and processed 20 blocks per second;
the 2012 VF-1 had 1 MB and up to 1,000 blocks per second (American Machinist, February 2012).
Fact for H3: NGC has 1 GB program memory as standard, wired Ethernet standard and Wi-Fi optional,
and Net Share for programs (West Manufacturing News, December 2016).

### H3. NGC (2016 and later): the richest, and documented by Haas

From Haas's own "Machine Data Collection – NGC" page (fact unless marked):

- **Setting 143** turns MDC on and **sets the TCP port** (Haas's example: 5051; monitoring vendors
  use 4001 or 8000). **Do not use 8082 or 9090–9999**, which the control uses for other things.
- **The control is a TCP server**; any TCP terminal can connect. **Up to 2 connections at a time,
  and a reply to one connection goes to all of them.** So two collectors on one Haas will see
  each other's answers. **The Edge should be the only MDC client** and share the data onward
  (opinion; this also explains the "too many connections" reports, MachineMetrics).
- **Query format:** `?Q###` with a newline, upper case. Replies start with `>` and are
  comma-separated. An unknown command returns `?, ?Q105` (for example). The first `?Q100` after
  connecting may need sending twice.
- **Queries** (each with Haas's example reply): `?Q100` serial number, `?Q101` software version
  (e.g. `100.17.000.2037`), `?Q102` model, `?Q104` mode, `?Q200` total tool changes, `?Q201` tool
  in use, `?Q300` power-on time, `?Q301` motion time, `?Q303` last cycle time, `?Q304` previous
  cycle time, `?Q402`/`?Q403` M30 counters, `?Q500` program, status and parts in one reply.
- **`?Q600 nnnn` reads any macro or system variable.** This is how the Edge reads what the Q list
  lacks: positions, spindle speed, spindle load, probe results. Variable numbers must be checked
  against the Haas macro manual for the machine: a community macro uses **`#1132` as spindle load
  in %** (eMastercam forum, marked "unproven" by its author; unverified), and a community MTConnect
  adapter reads absolute Z with `Q600 5023` (HaasMTConnect README).
- **`?Exxxx yyyyyy.yyyyyy` writes a variable**, limited to **#1–#33 and #10000–#10999**
  (#10550–#10580 unavailable with probing). Haas warns that wrong system variable values can
  damage the machine. **For a read-only Edge, this command must never be sent** (see the module
  below).
- **MTConnect on port 8082:** `http://<ip>:8082/<model>/probe`, `/current`, `/sample`. Haas
  recommends software **100.20.000.1200 or later** for best MTConnect function; machine names with
  a "/" needed a patch (Haas note dated 18 August 2024). One secondary source says HTTPS is on
  port 8083 (unverified). MachineMetrics adds that some newer NGC machines offer the agent at
  `http://<ip>:8082/current`.
- **Status relays:** factory **Setting 315.26 STATUS RELAYS = TRUE** turns 8M relay outputs 90–97
  into machine status outputs. It needs the 8M relay option and a service key (Haas). Which relay
  means which state was not found (unverified). With the 8M option fitted, this is a **factory
  status output** a read-only module can wire to without touching anything else.
- **User relays:** the NGC I/O board has user relays M21, M22 (TB3A), M23, M24 (TB3B, reserved for
  the Haas robot package without the User Relay Kit) and M25 (TB2); M59/M69 Pnnn switch them and
  M29/M129 Pnnn wait for a finish signal; 8M banks add eight relays each (Haas "M-Code Relay
  Function – NGC"). A program posted by SolidCraft can **pulse a relay at each operation
  change** as a hardware sync marker for the bolt-on sensors (opinion). The relay is driven by
  the program, not by the Edge, so the Edge stays read-only.
- **DPRNT over the network:** NGC can send DPRNT output over TCP. FactoryWiz's test uses TCP 8001
  for DPRNT and 8000 for MDC, and names Settings 261 and 263 as the ones to check (FactoryWiz
  knowledge base). The Haas DPRNT page could not be fetched; exact setting meanings unverified.
  **DPRNT is the best event channel:** a posted program can print "op 3 start, tool 7, segment
  1042" at the exact block, and probe results after each measurement.
- **HaasConnect:** phone and e-mail alerts of status, alarms and operator overrides, standard on
  Haas VMCs and HMCs with NGC (Haas). No API is published.
- **In-control tool load monitoring (Tier D):** per-tool load limits on the Current Commands load
  page; **Setting 84** chooses alarm, feed hold, beep or **autofeed** (feed steps down to a minimum
  when load passes the limit, and back when it falls below 95 % of it, in 0.1 s steps; since
  software 9.43; not in tapping) (mfgrobots summary of Haas material). This is where SolidCraft's
  predicted loads should end up (machine interfaces file).

**Software version caveat (fact):** NGC software numbers look like `100.20.000.1200`; features such
as MTConnect depend on them. **The Edge should read `?Q101` first and pick its feature set from
the version** (opinion).

### H2. Classic Haas Control (about 2001–2017)

- **MDC on RS-232 port 1.** MachineMetrics settings: 38,400 baud, even parity, 7 data bits, 1 stop
  bit, XON/XOFF; Setting 143 ON. Reply framing from a forum copy of Haas documentation: STX
  (0x02), comma-separated reply, ETB (0x17), CR/LF, then `>` (CNC Arena; check on a real machine).
- **The Pi needs no Moxa box.** Monitoring vendors use a serial-to-Ethernet converter because their
  computer is elsewhere. SolidCraft Edge sits at the machine, so an isolated USB-to-RS-232 adapter
  or an RS-232 board on the Pi is enough (opinion).
- **Same Q commands**, including `Q600` for macro variables (TrakHound's list of planned work
  mentions adding Q600 for Haas; supported on classic per the HaasMTConnect README, which reads
  over the serial port). Whether the classic control accepts the `E` write command was not found
  (unverified); the read-only module never sends it anyway.
- **One port, two jobs.** On machines where port 1 also carries DNC program transfer, MDC and DNC
  share it (inference). The Edge can take over DNC too (it already has the programs from
  SolidCraft).
- **Model support varies** (MachineMetrics comments, not official): mills like VF-YT, EC, TM, ST
  and VM likely support it; older VF, SL, HL and Mini Mill depend on year and upgrades. **Check
  for Setting 143 on the machine.**
- **Ethernet on later classic machines** is for file sharing (Net Share/FTP); MDC stays on serial
  (MachineMetrics treats all pre-NGC machines as serial).
- **No MTConnect agent on the control.** The Edge can run its own MTConnect agent and publish the
  classic Haas data as MTConnect, so the rest of SolidCraft sees one interface for every Haas
  (opinion). The MIT-licensed MTConnect.NET-Haas modules show this is a known pattern; SolidCraft
  would write its own in Rust.

### H1. Early Haas control (1988 to about 2000)

- **No MDC.** The RS-232 port carries programs only.
- **Bolt-on only:** stack light or lamp inputs, a user relay pulse at M30 for part counts (if the
  machine has one), a current sensor on one spindle motor conductor for run/idle and load,
  vibration on the spindle housing (MachineMetrics' "Haas I/O Integration" uses exactly a relay
  pulse and a spindle CT for these machines).
- **DNC drip-feed:** if the shop drip-feeds programs, the Edge as the DNC server knows which block
  has been sent. That is the only "where in the program" signal these machines can give.
- **Haas machines do not supply 24 V for accessories** (MachineMetrics), so the module needs its
  own power: a DIN-rail supply from the cabinet's service outlet, or PoE.

### What "real-time" means on a Haas, by generation

| Signal | H1 early | H2 classic | H3 NGC |
|---|---|---|---|
| Run / idle / alarm | lamps, spindle current | `Q500`, `Q104` | `Q500`, MTConnect, status relays |
| Program, tool, part count | relay pulse; DNC | `Q500`, `Q201`, `Q402` | same, plus MTConnect |
| Block / where in the CAM program | DNC sent-block | DNC sent-block, `Q600` of a variable the post updates | **DPRNT markers** from the post; MTConnect block (if published; check `/probe`) |
| Spindle load | **CT on spindle conductor (kHz)** | `Q600` load variable (≈1–10 per second; number unverified) + CT | MTConnect or `Q600` (≈1–10 per second) + CT |
| Axis load | Hall sensors on servo conductors (kHz; cabinet work) | same | same; or MTConnect if published |
| Probe results | none | DPRNT over serial | DPRNT over TCP; `Q600` |
| Tool wear, chatter | vibration sensor | same | same |

**Calibration trick (opinion):** on H2 and H3, read the control's own spindle load % slowly
through `Q600` or MTConnect, and the CT current fast. Fit one to the other during the first few
cuts. From then on the fast current signal is reported in the control's units.

## The read-only plug-in module

### What it is

A small board that plugs onto the Raspberry Pi 5 (or a Compute Module 5 industrial carrier),
DIN-rail mounted in or next to the electrical cabinet, called here the **SolidCraft Edge I/O
module** (working name). It is designed around Haas, and it is the same module for every brand,
because the plug-in points are generic. Everything below is a design proposal (opinion), with
facts cited where they constrain it.

| Block | Purpose | Spec (proposal) |
|---|---|---|
| **Data ports** | MDC, MTConnect, FOCAS, OPC UA, DNC | 2 × Ethernet (one to the machine, one to the shop network; **no routing between them**); 1 × isolated RS-232; 1 × isolated RS-485 (receive-only jumper) |
| **Digital inputs** | stack light, status relays (Haas 8M outputs 90–97), user relays, door switch, M30 pulse | 8 × opto-isolated inputs for dry contacts, 24 V DC and 120/230 V AC lamp circuits (selectable per channel); 2.5 kV or more isolation |
| **Analog inputs** | spindle and axis current, drive analog load outputs | 4–8 channels, simultaneous-sampling 24-bit ADC at 8–50 kS/s per channel; inputs for split-core CTs and **closed-loop Hall sensors** (Hall sensors pass DC and low frequency; CTs lose accuracy at low spindle speed); 0–10 V inputs for drive load outputs |
| **Vibration** | tooth-level wear, chatter | 2 × IEPE accelerometer inputs at 25.6 kS/s, or a MEMS accelerometer on a short cable |
| **Shop infrastructure** | power quality, compressed air, ambient conditions (Jason; what each problem does to the physics system and its thresholds belong in the [machine health file](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-9w3gq9/research/cam-research-nsnlookupjn-machine-health.md)) | **read the machine's own sensors first** (Jason: many machines have them built in, for example thermal-compensation temperature sensors (Okuma Thermo-Friendly, Mazak Intelligent Thermal Shield), low air pressure, low-voltage or phase-loss alarms and coolant level; exposed as MTConnect, OPC UA or variables where the control publishes them, and recorded as capability flags); add-ons only where a machine has none: a DIN-rail three-phase power-quality meter on the machine's supply read over the RS-485 port (Modbus RTU), so the module never touches mains voltage itself; 2 × 4–20 mA inputs for an air pressure transducer and a dew-point sensor; an ambient temperature and humidity sensor (and optionally one on the machine casting) on a short digital bus |
| **Time** | aligning fast sensor data with control events | PTP/NTP on the shop network; hardware timestamps on every sample block |
| **Power** | Haas does not supply 24 V for accessories (MachineMetrics) | 24 V DC in from a DIN-rail supply, or PoE |
| **Outputs** | none | **no relay, transistor or analog outputs exist on the board** |

### How it stays read-only

Jason specifies a read-only interface. "Read-only" has to be enforced at every layer, because some
of the ports it uses also accept writes:

1. **No outputs in hardware.** The board has nothing that can drive a machine signal.
2. **Receive-only taps.** RS-485 and sniffing RS-232 connections have the transmit line physically
   open (a jumper or a receive-only cable).
3. **A query allow-list in the protocol layer.** On a Haas data port, the Edge firmware may send
   only `?Q100`–`?Q600` queries. **`?E` (write variable) is not in the code at all**, rather than
   behind a flag. The same applies to FOCAS (read functions only), OPC UA (Read, Browse and
   Subscribe only; no Write or Call) and Bambu or Prusa APIs (status calls only).
4. **One client.** The Edge is the machine's only data client and republishes data (MTConnect,
   MQTT), so other software never needs a writable connection (opinion; Haas NGC allows only two
   connections and broadcasts replies to all).
5. **Shown on the box.** The dashboard says "read-only" and the build hash of the allow-list, so an
   auditor can check it (high-security file).

When SolidCraft later wants to write (offsets, staged programs, load limits), that is a separate,
opt-in module and a separate safety case (the machine interfaces file's write tiers). The
read-only module never grows that ability.

### Installing it safely

- **Documented ports first** (Ethernet, RS-232): no cabinet work, no warranty question.
- **Anything in the cabinet** (relay terminals, motor conductors): a qualified electrician,
  lockout/tagout, and the machine's own wiring diagrams. Machine electrical standards apply
  (NFPA 79 in the US, IEC 60204-1 in Europe). The machine's DC bus and motor leads carry hundreds of
  volts, and drive capacitors stay charged after power-off.
- **Current sensors go around one conductor**, at the drive terminals inside the cabinet. A clamp
  around a whole three-phase motor cable reads nearly zero, because the phase currents cancel.
- **Never connect to E-stop, door interlock or other safety circuits**, even as an input.
- **Check with the dealer** before any work inside the cabinet; for Haas, the local Haas Factory
  Outlet.

### Every factory feature, every cut

Jason's aim: **use every factory feature of every machine to optimise every cut**, and SolidCraft's
audience: **novices who should make parts like pros, and pros who get every tool they ever asked
for.** Both need the machine library to know the features, and the post to use them. Seed list of
cut-optimising factory features (K unless marked; names are the makers' own; each becomes a flag
in the machine library's "Motion and smoothing", "Load monitor and autofeed" or "Probing" group):

| Control | Smoothing / high-speed | Load and adaptive feed | Chatter, thermal, collision | Probing and tool setting |
|---|---|---|---|---|
| Haas NGC | G187 (smoothness and corner tolerance) | per-tool load monitor; Setting 84 autofeed (F) | – | WIPS probing macros, VPS templates (F) |
| Fanuc 0i/30i | AI contour control (G05.1 Q1), AICC II, nano smoothing | optional adaptive control and tool-load monitoring via builder/partner options | servo learning, thermal compensation options | probe macros (Renishaw and builder) |
| Siemens 840D sl/ONE | CYCLE832 high-speed settings; Top Surface | ACM (adaptive control and monitoring) option; Edge apps (F) | collision avoidance option | measuring cycles |
| Heidenhain TNC 640/TNC7 | Cycle 32 TOLERANCE | AFC adaptive feed control option | ACC active chatter control, DCM dynamic collision monitoring options | touch-probe cycles, KinematicsOpt |
| Okuma OSP | Super-NURBS, Hi-Cut Pro | load monitor | Machining Navi (chatter avoidance using sound), Thermo-Friendly Concept, Collision Avoidance System | probing cycles |
| Mazak Smooth | Smooth machining configuration | Intelligent Machining functions | Active Vibration Control, Intelligent Thermal Shield, Intelligent Safety Shield | probing, tool eye |
| Mitsubishi M8 | SSS (super smooth surface) control | load monitor | chatter suppression (on some) | probing |
| Brother CNC-D00 | high-accuracy mode | – | thermal compensation | – |

**How SolidCraft uses them (opinion):**

- **Novice:** they plug in the Edge, it discovers the machine, and CAM switches the right features
  on by default. Smoothing tolerance comes from the part tolerance, load limits from CAM Physics,
  probing where the model asks for it, and chatter avoidance where the control has it. Everything
  is explained in plain words on the setup sheet ("G187 P2 E0.02 used because the tightest surface
  tolerance in this operation is 0.05 mm").
- **Pro:** every feature is visible and overridable per operation, every raw signal from the Edge
  is available (plots, export, MCP), and nothing is hidden behind the defaults. An override is
  recorded with its reason, like the risk slider.
- **Both:** a feature is used only when the machine library says this machine has it (verified
  or discovered for anything unattended), with a stated fallback otherwise.

### Keeping up with control software, and a reason to upgrade

Jason's point: makers keep releasing control software, updating is often a paid service-tech job,
so SolidCraft should **keep up with new revisions and give the shop a reason to upgrade**: "you
can have this feature and run this much faster if you upgrade to this". A design (opinion):

**1. Track versions as data, not code.** Each control generation in the catalogue lists its
software versions and what each adds or fixes (a feature flag, a new MTConnect data item, a
changed setting). Sources, best first:

- **Edge discoveries:** every Edge reads the version (`?Q101`, `cnc_sysinfo`, OPC UA) and which
  capabilities actually work, so a new version shows up in the catalogue the day a shop installs it
  (opt-in, anonymous).
- **Maker release notes and service pages** where published: for example Haas's NGC software
  revision pages, and Haas's own advice that MTConnect works best from 100.20.000.1200.
- **Shops and dealers** correcting entries through ordinary reviewed changes to the catalogue.

The capability key already carries a software version range (machine library file), so a new
version is a new range, not a code change.

**2. The upgrade advisor.** SolidCraft knows three things no one else has together: the
machine's current version (Edge), what newer versions and paid options add (catalogue), and **the
shop's own jobs** (CAM, with real cycle times from the Edge). So it can price an upgrade in the
shop's own numbers:

| Step | What SolidCraft does |
|---|---|
| Find the gap | "This VF-2 runs 100.17; 100.20.000.1200+ adds full MTConnect data; option X adds feature Y." |
| Re-run the shop's history | Re-post the last 90 days of jobs on this machine **with the feature turned on** (a smoothing mode, adaptive feed, a faster block rate) and simulate cycle time with CAM Physics. No machine time is needed. |
| Show the benefit | "On your last 90 days: 31 hours saved (8 %), 4 tool breaks that the load monitor would have caught, live load data for self-tuning." |
| Show the cost | the shop enters the dealer's quote (service visit, option price, downtime); SolidCraft computes payback ("about 7 weeks at your shop rate") |
| Show the risk | requalification (aerospace and defence shops may need a first-article or process re-approval after a control software change; compliance file), program compatibility, and a **backup of the control first** (connection route 6) |
| Check afterwards | after the upgrade, the Edge verifies the new capabilities and compares real cycle times with the prediction, so the next advisor estimate is more trustworthy |

**3. The same advisor for paid options and hardware:** a control option (for example an
adaptive feed or high-speed smoothing option), a probe or tool setter, through-spindle coolant or
a faster spindle. "Adding a tool setter removes 14 manual tool touch-offs a week."

**Rules (opinion):**

- **Neutral and honest.** No sponsored recommendations, and estimates show their confidence and
  method. A recommendation that turned out wrong is shown as wrong. Shops will trust it only if
  makers cannot buy a place in it.
- **Never pushed during a job;** advice appears in planning and the machine's page, not on the
  machine screen mid-shift.
- **Pros see the working** (which jobs, which feature, what the simulation changed); novices see
  one sentence and the payback.
- **Aggregated, anonymous demand** ("2,300 Haas machines on SolidCraft would gain from feature Y")
  is useful evidence in the conversations with makers above (opinion; only with shops' opt-in).

### From CAM physics to the control and back: load limits and learning

Jason likes CAM Physics telling the machine the load limit for each tool, for automatic stop and
perhaps automatic feed control, and still wants vibration and load analysis so SolidCraft can
"learn" setups and tools and optimise the cutting. That is a closed loop that **never needs the
Edge to write to the control** (opinion; it builds on the machine interfaces file's "Posting limits
into the control", the machine health file's tool condition layers and the inspection file's
self-tuning CAM operations):

1. **CAM Physics predicts the load.** For each tool, the cutting physics file gives cutting power
   per segment. Divided by the spindle's available power at that speed (the machine library's
   power curve), that gives a predicted spindle load in %, the unit the control uses.
2. **The post writes two limits per tool:**
   - an **upper limit** (the heaviest predicted segment plus the margin set by the risk slider),
     which the control uses for automatic stop or feed hold;
   - a **lower limit** check: in segments where the tool should be cutting, load near idle means a
     broken or missing tool. On a Haas this is a short macro at a safe point (the community macro
     cited above does this), because the built-in monitor watches only the upper limit (inference).
3. **The control reacts in milliseconds (Tier D).** On a Haas, Setting 84 picks alarm, feed hold,
   beep or **autofeed**. Autofeed (feed steps down while load is over the limit, then back) fits
   roughing with variable engagement. It is not for tapping or thread milling (Haas), and it can
   hide a dulling tool, so the Edge logs every slowdown as evidence. How the post can set a Haas
   tool's load limit from the program, rather than on the load page, is still unverified (machine
   interfaces file, open question 2).
4. **The Edge measures what really happened (Tier C).** The read-only module records spindle
   current at kHz and vibration at 25.6 kS/s, aligned to CAM segments by the DPRNT markers the post
   prints. For each segment it keeps measured over predicted load (the residual), and the vibration
   at the tooth-passing frequency and any other strong peak (chatter).
5. **SolidCraft learns, per combination of tool, holder, stick-out, setup and material lot:**
   - **tools:** the real specific cutting force and how the residual grows with wear (tool life);
   - **setups:** stiffness and the frequencies that chatter (chatter peaks that move with spindle
     speed fit the stability model in the cutting physics file; peaks that stay put point at the
     workholding or the part, per the workholding file);
   - **the machine:** idle power and the spindle's real power curve (machine health file).
6. **The next post is better.** Updated limits and suggested feeds and speeds come back to the
   programmer as a proposal (Stage tier; the programmer accepts). The improved numbers reach the
   machine in the next program, not through a live write. So the Edge can stay strictly read-only
   and the loop still closes.

**Learning a new tool or setup quickly (opinion):** an optional short "fingerprint" cut at the start
of a job. That is an air cut (idle power at each speed), then two light cuts at known engagement.
It calibrates the current sensor against the control's own load %, and gives a first specific
cutting force before the real cuts begin. The machine library's warm-up cycle is the natural place
for the air cut.

### Sensors that connect to the Edge

Jason: research the sensors that can connect to the Edge, from industrial-rated Modbus sensors to
cheap commercial Wi-Fi, Bluetooth and PoE ones. This section covers **how each kind connects and
what rate and reliability it gives**. What a signal means stays in the machine health file, and the
retrofit sensor kit in the inspection file. Facts are cited; **K** marks general industry
knowledge not checked here.

#### By connection type

| Connection | Typical sensors | Rate the Edge gets | Power | Cost | Good for | Not good for |
|---|---|---|---|---|---|---|
| **Direct analog into the I/O module** (IEPE, CT, Hall, 4–20 mA, 0–10 V, thermocouple/RTD) | accelerometers, current sensors, pressure transducers, temperature probes | **1–50 kHz** (the module's ADC) | from the module | low per signal | tooth-level vibration, spindle and axis current, anything fast | many signals spread around a shop |
| **Modbus RTU (RS-485) / Modbus TCP** | power-quality meters, temperature and humidity transmitters, flow meters, chillers, coolant units, VFDs | polled, 1–10 values per second (K) | own supply | low | robust, cheap, every industrial maker supports it | fast signals; Modbus has no security of its own, so keep it on the machine segment (K) |
| **IO-Link** (point-to-point, 24 V, three-wire; IEC 61131-9, K) | vibration sensors with built-in analysis, pressure, flow, temperature, level, RFID | COM1 4.8, COM2 38.4, COM3 230.4 kbaud (ifm); millisecond cycles (K) | from the IO-Link master | medium | **industrial plug-and-play**: each device is described by an IODD file; masters publish data as **REST, MQTT or OPC UA in JSON** (ifm), so the Edge needs no IO-Link stack of its own | raw kHz waveforms (devices usually send computed values) |
| **IO-Link Wireless** | the same devices, wireless | up to 40 devices, 5 ms latency, packet error rate better than 10⁻⁹ (IO-Link community, 2018) | battery or local | medium–high | rotating or moving parts, retrofits without cable runs | secure wired-only shops |
| **PoE (Power over Ethernet)** | IP cameras, multi-sensor nodes, PoE I/O modules | network rate; video | over the Ethernet cable | medium | cameras (slip, chips, door, operator view), sensor nodes a cable run away; **wired, so acceptable in secure shops** | very cheap single sensors |
| **USB to the Pi** | USB accelerometers, measurement microphones, USB DAQ, cameras | kHz (DAQ, audio) | from the Pi | low | a sensor right next to the Edge; sound for chatter (Okuma's Machining Navi uses sound, K) | long distances |
| **Wi-Fi** (ESP32-class boards, smart plugs and energy meters; MQTT or HTTP) | temperature, humidity, energy, door, light-on-stack-light sensors | about 1 value per second or slower (K) | mains or USB | **very low** | quick, cheap shop-wide coverage for novices; prototypes | fast signals; noisy RF near VFDs; **not allowed in many secure shops** (high-security file) |
| **Bluetooth LE** | battery vibration and temperature sensors | bursts: a stored sample or summary every few minutes; one example sends complete samples and lasts about 25,000 measurements on one 3,400 mAh battery (Nordic case study) | battery | low–medium | **condition trends** on spindles, pumps, compressors, fans | tooth-level or continuous monitoring; secure shops |
| **Zigbee / Thread / Matter** | building temperature, humidity, door, leak sensors | slow (K) | battery | very low | room conditions | machine signals |
| **LoRaWAN** | tanks, compressors, outdoor and yard sensors | a few values an hour or faster (K) | battery | low | long range (compressor room, chip bins, coolant tanks) | anything fast |

#### By signal: industrial choice and budget choice

| Signal | Industrial-rated choice | Budget choice | Note |
|---|---|---|---|
| Spindle vibration (tooth-level, chatter) | IEPE accelerometer into the module | MEMS accelerometer on a short cable into the module | wireless sensors cannot do tooth-level (rate) |
| Spindle and axis load | closed-loop Hall or split-core CT on one conductor, into the module | split-core CT (e.g. SCT-013 class, IMD guide) | calibrate against the control's own load % |
| Machine state | status relays and stack-light lamp circuits into opto inputs | light sensors stuck on the stack-light lamps (no cabinet work) | the control's own data is better where it exists |
| Ambient and casting temperature | Modbus or IO-Link transmitter | Wi-Fi/BLE sensor | read the machine's built-in sensors first |
| Compressed air pressure and dew point | IO-Link or 4–20 mA transmitter | 4–20 mA transducer into the module | low air affects tool changes, clamping and probes |
| Power quality | Modbus power-quality meter on the machine supply | Wi-Fi energy meter (no sub-cycle events) | sags and phase loss show as drive alarms |
| Coolant concentration, level, temperature | inline refractometer (Modbus or 4–20 mA), level and temperature transmitters | manual refractometer reading typed into the Edge | machine health file has the meaning |
| Sound | measurement microphone (USB) | USB or MEMS microphone | chatter frequency without touching the machine |
| Vision | PoE industrial camera | Raspberry Pi camera | slip, chips, door, tool presence |

#### Sensor pricing and where to buy (seed for a sensor library)

Jason: make a pricing table for sensors and where to buy them, so SolidCraft can build a sensor
library later and give users the choices they need to instrument a machine. **Prices are as
listed on the cited seller page when checked on 27 October 2026, unless marked as an older launch
price.** They change often, vary by country and include or exclude VAT as noted. A future library
should store the price with its date and seller and refresh it, never treat it as fixed. No seller
is endorsed. The list is a starting set chosen to cover each signal at an industrial and a budget
level.

| Signal | Product (maker, model) | Class | Interface to the Edge | Price seen | Where (seller seen) | Notes |
|---|---|---|---|---|---|---|
| Edge computer | Raspberry Pi 5, 4 GB / 8 GB | both | — | US$110 / US$175 (official prices, July 2026 snapshot; raised three times Dec 2025 – Apr 2026 due to memory costs) | Raspberry Pi approved resellers | travis.media price guide |
| Spindle/axis current | YHDC SCT-013-000 split-core CT, 100 A | budget | analog into the module | US$12.66 | Walmart marketplace; also eBay, OpenEnergyMonitor shop | AC only; weak at low spindle speed |
| Spindle/axis current | LEM HLSR series Hall-effect current sensors | industrial | analog into the module | price not captured | RS Components, Digi-Key | passes DC and low frequency; PCB-mount, needs a carrier |
| Vibration (tooth-level) | Analog Devices EVAL-ADXL355-PMDZ (MEMS accelerometer board) | budget | SPI into the module or Pi | price not captured | Digi-Key, Richardson RFPD | low-noise MEMS; short cable only |
| Vibration (tooth-level) | PCB Piezotronics 603C01 IEPE accelerometer | industrial | IEPE input on the module | from £99 (**2016 launch price**; check current) | PCB Piezotronics and distributors | industrial ceramic shear sensor |
| Vibration (condition) | ifm VVB001 IO-Link vibration sensor | industrial | IO-Link master → REST/MQTT/OPC UA | €632.39 incl. VAT | Bürklin | sends computed values and raw data blocks |
| IO-Link gateway | ifm AL1350 IO-Link master, 4 ports, IoT | industrial | Ethernet (JSON over REST/MQTT/OPC UA per ifm's IoT masters; check this model) | €316.41 excl. VAT | Eltra Trade; RS Components | needs 24 V |
| Power quality | Eastron SDM630MCT Modbus three-phase meter | industrial (budget end) | Modbus RTU on RS-485 | £124.17 excl. VAT (CTs extra, from about £37.50 each) | Bimble Solar | energy, voltage, current, power factor; not a sub-cycle event recorder |
| Power (energy) | Shelly Pro 3EM V2 with three 120 A CTs | budget | Wi-Fi, Bluetooth, LAN | US$154.99 | Shelly US store | per-phase V, A, PF; 1-minute history; LAN keeps it wired |
| Air pressure | ProSense SPTD25 pressure transmitter, 4–20 mA | industrial (budget end) | 4–20 mA into the module | from US$99 (**2016 launch price**) | AutomationDirect | low air affects tool change, clamping, probes |
| Air dew point | Vaisala DMT143 dew-point transmitter (−70 to +60 °C) | industrial | analog output into the module | price on request | Vaisala distributors (e.g. AxFlow) | wet air damages spindles and pneumatics |
| Ambient temperature, humidity | RS-485 Modbus SHT30 temperature/humidity sensor | budget | Modbus RTU | price not captured | KinCony shop, Core Electronics | read the machine's built-in sensors first |
| Ambient temperature, humidity, pressure, motion | RuuviTag Bluetooth sensor | budget | Bluetooth LE | €39.99 | Verkkokauppa | battery for years (user review); not for secure shops |
| Sound (chatter) | miniDSP UMIK-1 USB measurement microphone | both | USB audio, 48 kHz, 24-bit | US$139.98 | Parts Express | individually calibrated |
| Vision | Reolink RLC-510A 5 MP PoE camera | budget | PoE (RTSP/ONVIF not confirmed on the seller page) | US$39.99 (sale) | 123SecurityProducts | slip, chips, door, operator view |
| Machine state | the machine's own status/user relays into the module's opto inputs | both | digital inputs | wiring only | — | free where the relays exist (Haas 8M option) |

**Example kits (estimates from the prices above; module and enclosure not yet priced):**

- **Novice starter kit:** Pi 5 4 GB, one SCT-013 CT, one MEMS accelerometer board, one RuuviTag,
  the machine's own data port. About **US$200–350** plus the I/O module.
- **Pro wired kit (secure-shop friendly):** Pi 5 8 GB, one IEPE accelerometer, an IO-Link master
  with a vibration sensor, a Modbus power meter with CTs, an air pressure transmitter and a
  measurement microphone. About **US$1,800–2,600** plus the I/O module.

**Fields for the future sensor library (opinion):** maker, model, class (industrial / budget),
signal, range, accuracy, bandwidth and rate, interface, power, IP rating, approvals, mounting,
wired-only safe (yes/no), Edge driver available (yes/no), datasheet link, and price entries (price,
currency, VAT included or not, seller, country, date checked). Each sensor in a shop then links
to its library entry plus its own calibration date, as in the capability model.

#### Rules for sensors on the Edge (opinion)

- **Every sensor is a capability** with signal, unit, rate, accuracy, location, calibration date and
  provenance, in the same machine library model as the control's signals. CAM uses it only as far
  as its rate and accuracy allow.
- **Read-only both ways:** the Edge reads sensors and never drives an output. Sensor config pages
  (smart plugs with relays, IO-Link actuators) are disabled or left unused.
- **Time:** fast sensors are timestamped by the module; network sensors by the sender where possible
  (PTP or NTP), otherwise on arrival with the delay noted.
- **Secure shops get a wired-only profile** (direct, Modbus, IO-Link wired, PoE, USB): no Wi-Fi,
  Bluetooth, Zigbee or LoRa radios on the Edge or its sensors (high-security file).
- **Novices get kits, pros get a catalogue.** A novice buys a "basic kit" (spindle CT, MEMS
  accelerometer, stack-light inputs, ambient sensor) that the Edge recognises without setup. A pro
  can add any Modbus register map, IO-Link device (by its IODD) or MQTT topic, and name its unit
  and rate.
- **Drivers in Rust, licence-checked:** Modbus, MQTT and BLE crates exist, but each must pass
  `cargo xtask licences --check` before use; no GPL code.

### Rust software on the Edge (sketch, opinion)

- One **connector per interface**: `haas-mdc` (TCP and serial), `mtconnect` (HTTP client and an
  agent for republishing), `dprnt` (TCP and serial listener), `opcua` (client, read-only), `focas`
  (out of process, because the Fanuc library is a native binary), `io-module` (SPI ADC and GPIO).
- Each connector declares which **signals it can give and at what rate**, and fills the machine
  library's "Signals the machine publishes" for that machine.
- **Crates must pass SolidCraft's licence check** (`cargo xtask licences --check`); permissive crates
  for serial ports, GPIO and SPI exist, but each must be checked before use. Nothing under GPL, so
  no LinuxCNC, Klipper or OctoPrint code.

### SolidCraft Edge as an open hardware project inside SolidCraft

**Jason's proposal:** an open hardware design, with its embedded software and operating system in
Rust, hosted as a project within SolidCraft. What that could look like (opinion; the structure is
a proposal for maintainers):

**Two processors, each doing what it is good at.**

| Part | Runs | Job |
|---|---|---|
| **I/O co-processor** on the module (a microcontroller such as Raspberry Pi's RP2350) | **bare-metal embedded Rust** (for example the Embassy async framework, MIT OR Apache-2.0; licence to be confirmed by `cargo xtask licences`) | samples the ADC and digital inputs on a fixed clock, timestamps every block, and enforces read-only in firmware: the serial and RS-485 transmit lines are driven only by this chip, and its firmware can send nothing except allow-listed queries |
| **Raspberry Pi 5** (or Compute Module 5) | a **minimal Linux image** with every user-space program in Rust | network connectors (MTConnect, OPC UA, Haas MDC, FOCAS), the dashboard, report printing, the link to SolidCraft CAM (REST, MQTT, WebSocket), updates |

**Why not a Rust operating system on the Pi too (opinion):** the Linux kernel is GPL, but running
it is not reading it, and it is the only practical way today to get the Pi 5's Ethernet, USB,
display and Wi-Fi drivers. Rust-written operating systems (for example Redox, MIT) do not support
the Pi 5's hardware well enough yet (unverified for late 2026). Bare-metal Rust is the right
choice for the real-time part, where it shines. The Linux layer should be small, read-only
(squashfs root, A/B updates) and built reproducibly. Image build tools also need a licence check:
Yocto and Buildroot are themselves GPL tools, which the project would run but never copy into the
repository.

**Repository layout (sketch):**

- `edge/firmware/` – the co-processor firmware (embedded Rust, `no_std`).
- `edge/apps/` – the Pi user-space services (Rust): connectors, dashboard, reports, CAM link.
- `edge/os/` – the image recipe and update scheme (configuration only).
- `edge/hardware/` – the board and enclosure as SolidCraft designs (schematic, PCB layout, bill of
  materials, enclosure), plus exported Gerber X2 / IPC-2581 for manufacture.
- `edge/docs/` – the installation and safety guide (electrician steps, lockout/tagout, standards).

**How it fits SolidCraft's rules (facts from AGENTS.md; consequences opinion):**

- **"Application code is Rust only and builds for wasm."** The firmware and the Pi services cannot
  build for wasm. Edge would need its own Cargo workspace or an agreed exception, with the shared
  data types (signals, machine library entries, results) in a crate that does build for wasm, so
  the desktop and web apps read Edge data with the same code.
- **Layering:** Edge services would sit beside `apps`, depending on `doc` and `io` types but never
  on `ui-egui`; `cargo xtask layers` would need a rule for them.
- **No GPL code read or copied**, which rules out reusing LinuxCNC, Klipper or GPL MTConnect
  agents; MIT/Apache crates only, checked by `cargo xtask licences`.
- **Licences:** software MIT OR Apache-2.0 like the rest of SolidCraft; hardware under a
  permissive open hardware licence such as **CERN-OHL-P-2.0**. The PCB tool matters: KiCad is a
  GPL application, but designs made with it are the designer's own (running a tool is not reading
  its code). Maintainers should confirm that reading confirms this.
- **Name:** "SolidCraft Edge" is still a working name.

**Designed in SolidCraft itself (Jason's proposal).** The Edge hardware should be designed with
SolidCraft, using a new **electronics module**, so the project proves the tool on its own product.
SolidCraft has no electronics design today (the repository has mechanical CAD, CAM research and an
"electronics enclosure" demo built over MCP, `docs/mcp.md`, but no schematic or PCB tools). A
staged path (opinion):

1. **Now: mechanical side in SolidCraft.** Enclosure, DIN-rail clip, connector cut-outs, heat
   path and cable glands, as a normal SolidCraft design (the MCP enclosure demo shows this already
   works).
2. **Board as a part.** Board outline, mounting holes, keep-outs and 3D component placement in
   SolidCraft, so the enclosure and board are checked together for fit and clearance.
3. **Electronics module.** Schematic capture, PCB layout, electrical and design rule checks, and
   manufacturing output in open formats: **Gerber X2** (open specification), **IPC-2581** (open
   IPC standard) and a bill of materials, plus a STEP model of the populated board. Symbols and
   footprints generated from the IPC-7351 land-pattern rules and maker datasheets, not copied from
   another tool's libraries (clean-room ground rule).
4. **Edge built with it.** Each Edge board revision is a SolidCraft design in the repository, in
   the same reviewable format as other designs ("designs in git", checking skill).

Until stage 3 exists, an external open tool could draw the first board, but that would be a
stop-gap. The electronics module is a large topic of its own and deserves its own research file
(opinion).

**Why host it inside SolidCraft (opinion):** one repository means the CAM side, the machine library
and the Edge share one signal vocabulary and one release. The read-only allow-list is reviewed in
the same open PRs as the code that depends on it. For defence and aerospace shops, an auditable,
open, read-only device is easier to approve than a closed vendor box (high-security file).

## Working with Haas on a factory-integrated interface

**What to ask for (opinion), in order of size:**

1. **Publish what already exists:** the full NGC MTConnect data-item list by software version, the
   DPRNT network settings, the status relay mapping (Setting 315.26), and the spindle and axis
   load variable numbers for `?Q600`.
2. **A read-only MDC mode:** a setting that makes the Setting 143 port refuse `?E`, so a shop can
   connect monitoring without opening a write path. This is cheap for Haas, and good for defence and
   aerospace shops.
3. **Faster, richer MTConnect:** spindle load, axis loads, actual feed, block number and tool in
   the `/sample` stream at 100 Hz or more, with timestamps.
4. **A factory option:** a Haas-fitted cabinet slot or connector for a third-party read-only
   module, with 24 V power, the status relays and the encoder or drive data Haas chooses to expose,
   so no shop ever has to clamp onto motor leads.
5. **The machine MCP profile** (below), served by the NGC itself or by a Haas gateway.

**Who to contact.** Haas Automation is privately owned by Gene Haas and based in Oxnard,
California (Wikipedia). It sells and services machines through its network of **Haas Factory
Outlets (HFOs)**, which are the first contact for any customer engineering question. No public
developer or partner programme was found (the HaasConnect page offers none). Suggested route
(opinion):

1. **The local HFO's applications or service engineer**, with a written one-page request (items
   1–3 above) and the shop's machines and software versions.
2. **Haas Automation headquarters in Oxnard**, asking the HFO to forward the request to the
   control software and electrical engineering groups.
3. **Existing integration partners** who already work with Haas data: Autodesk (Fusion Machine
   Connect lists "HAAS (Classic and Next Generation)") and monitoring vendors (MachineMetrics and
   others). Haas knows them, so they show how Haas handles third-party requests.
4. **Standards bodies:** the MTConnect Institute, since Haas ships an MTConnect agent. Asking there
   for a richer Haas MTConnect profile helps every Haas user, not only SolidCraft.

Named individuals are deliberately not listed in this public file.

## One standard: a Model Context Protocol (MCP) profile for machines

Jason's goal is to get every maker to implement **MCP, the Model Context Protocol** (the open
protocol AI agents use to discover and call tools; SolidCraft already ships an MCP server), unless
there is a better universal standard.

**Is there a better universal standard? (opinion, with the facts behind it)** For *machine data*,
two open standards already exist and are widely shipped: **MTConnect** (read-only, HTTP/XML;
built into Haas NGC, Mazak, Hurco and others) and **OPC UA for Machine Tools** (umati, OPC
40501-1; driven by the German and Swiss builders and the control makers). Both describe machines
better than MCP does: units, data types, rates, history. Neither is designed for an AI agent. They
do not give self-describing tools with JSON Schema inputs and outputs, a way to ask the human for
confirmation, or agent-style authorisation. MCP gives all three and describes no machine at all.
So the answer is **both, layered**: makers keep MTConnect or OPC UA as the data model and add an
MCP server on top that exposes that data as agent tools under a fixed safety profile. Asking
makers to replace OPC UA or MTConnect with MCP would be refused. Asking them to add a thin MCP
layer over what they already ship is cheap, and the OPC Foundation has already started down that
road with its OPC UA MCP package. That is the profile below.

The machine
interfaces file already proposes the tool set and safety tiers (Record, Advise, Stage, Request,
Commit, Never; dangerous tools absent, not hidden; confirmation on the machine). This file adds
three things (opinion):

**1. Read-only first.** Version 1 of the profile is **Record only**: identity, capabilities,
state, signal subscriptions, offsets, tools and results. Makers can ship that without any safety
review of writes, and it is all SolidCraft Edge needs today.

**2. Mapped onto names makers already use.** Every MCP resource maps to an MTConnect data item and
an OPC UA for Machine Tools (OPC 40501-1, umati) node, so a maker with either can serve it with a
thin adapter, and the Edge can serve it *on the maker's behalf* for older machines:

| MCP resource or tool (profile v1) | MTConnect | OPC UA for Machine Tools (umati) |
|---|---|---|
| `machine.identity` | `Device` attributes, `/probe` | `Identification` |
| `machine.state` | `Execution`, `ControllerMode`, `Program`, `Block`, `Line` | `Monitoring` (channel, spindle, stacklight) |
| `signals.subscribe` (load, speed, feed) | `Load`, `RotaryVelocity`, `PathFeedrate`, `/sample` | spindle and axis monitoring variables |
| `tools.read` | `CuttingTool` assets | `Equipment` / tools |
| `alarms.read` | `Condition` items | `Notification` / messages |
| `results.read` (probe, CMM) | `Measurement`-type assets (unverified) | (no common node found; QIF file reference) |
| `capabilities` (which signals, which rates) | `/probe` + sample intervals | `Capabilities` |

**3. A pitch per maker, in ranking order.** Each line is the one-sentence ask (opinion):

| Rank | Maker | The ask |
|---|---|---|
| C1 | Fanuc | Serve profile v1 from the existing FOCAS/OPC UA data on the control or a FIELD gateway, read-only, without a separate library licence. |
| C2 | Siemens | Serve profile v1 from the SINUMERIK OPC UA server, and let Edge apps publish high-rate signals through it. |
| C3 | GSK, Huazhong, KND | Map NC-Link to profile v1, so domestic controls join the same tooling. |
| C4 | Heidenhain | Serve profile v1 from the OPC UA NC Server; operator confirmations already exist in DNC. |
| C5 | Mitsubishi | Serve profile v1 from the MTConnect/API layer. |
| C6 | **Haas** | Serve profile v1 from the NGC (MTConnect is already there), with the read-only MDC mode above; Haas is the most likely early adopter in the US job-shop market. |
| C7–C8 | Mazak, Okuma | Serve profile v1 from SmartBox / THINC. |
| A1–A4 | Bambu Lab, Creality, Prusa, Formlabs | Serve profile v1 locally on the printer, no cloud account required. |
| A6, A8 | Stratasys, EOS | Map MTConnect (Stratasys) and EOSCONNECT OPC UA (EOS) to profile v1. |
| M1–M3 | Hexagon, Zeiss, Mitutoyo | Serve `results.read` as QIF and state as profile v1, alongside the I++ DME 3.0 work due at the end of 2026. |
| M5–M6 | **Renishaw** | Serve `results.read` for on-machine probe cycles and CMM/Equator results, and state for its controllers, from Renishaw Central; see the pitch below. |

### A pitch to Renishaw

Jason asks for a separate pitch to Renishaw, because Renishaw leads in on-machine probes and is a
major CMM and gauging maker (Jason's assessment; no market-share source was checked). Renishaw is
the one maker that sits **across every machine in the ranking**: its probes and tool setters are
fitted to Fanuc, Siemens, Heidenhain, Haas (WIPS is Renishaw-based, inspection file), Mazak and
Okuma machines; it makes CMM controllers (UCC), probe heads (REVO), the Equator gauge and metal
additive machines (RenAM). Facts on Renishaw's software: **Renishaw Central** (launched 2023) is
its process-control and data platform, which "collects metrology, status and alarm data from
connected devices across the shop floor" (Modern Machine Shop, September 2023), and Renishaw
joined the I++ DME 3.0 work (VDW, July 2025). Renishaw uses its own UK factories as the proving
ground (RAMTIC cells, MMS, August 2023). Which protocols and APIs Renishaw Central exposes to third
parties was not found (unverified).

**The pitch (opinion, a draft for Jason to adapt):**

1. **One probe result format on every control.** Today a probe result lives in a different macro
   variable on every control, and leaves the machine by DPRNT, FOCAS or a vendor connector.
   Renishaw writes the probing macros for most of those controls, so Renishaw could print every
   result as one documented record (feature, nominal, measured, deviation, tool, work offset,
   time), ideally as QIF-compatible fields. That works on any control that supports DPRNT, and on
   every brand at once.
2. **Renishaw Central as a profile v1 server.** Serve the read-only machine MCP profile
   (`machine.state`, `results.read`, `tools.read`) from Renishaw Central, so SolidCraft Edge,
   or any open tool, reads Renishaw data without a per-machine connector.
3. **Model-based probing from open CAD.** SolidCraft carries tolerances, characteristic importance
   and probing intent in the model (inspection file). A documented, open way to hand Renishaw's
   cycle generators the features and tolerances (QIF plan in, QIF results out) lets Renishaw
   probing follow the design automatically. Renishaw gains an open, free front end that sends
   more shops to its probes.
4. **I++ DME 3.0 read-only monitoring.** Renishaw is in the 3.0 group: ask for a standard,
   read-only monitoring connection to a CMM controller, so an Edge dashboard can show CMM state
   and position without being the motion client.
5. **Why Renishaw gains (opinion):** probing is where in-process quality starts. An open CAD/CAM
   that generates probing and reads results the same way on every brand makes Renishaw hardware
   the default choice for small shops that do not run Fusion or Mastercam today.

**Route in (opinion):** Renishaw's local applications engineers and its machine tool probe
product line, with the one-page ask (items 1–2), then the I++ DME group for item 4. As with Haas,
no named individuals are listed in this public file.

**Where to propose it:** the MTConnect Institute and the umati / OPC Foundation working groups,
with SolidCraft (and other open CAM projects) as a user voice. The OPC Foundation already ships an
OPC UA MCP package (machine interfaces file), so a "machine MCP profile, read-only v1, no motion
tools ever" companion is a natural, small next step (opinion).

## Accuracy

- **Market numbers** are the weakest part of this file. Fanuc's 5 million, Haas's 200,000,
  Heidenhain's 200,000 (2007), EOS's 5,000 and the CONTEXT shares are firm; the control-share
  percentages are uncited estimates that disagree with each other (Fanuc 40 % or 50–60 %;
  Heidenhain 5–10 % or 12–15 %). The *order* (Fanuc first, then Siemens) is consistent across all
  of them.
- **Haas NGC facts** come from Haas's own MDC page and M-code relay page and are firm. The spindle
  load variable number, DPRNT settings 261/263, the status relay mapping and HTTPS on 8083 are
  unverified.
- **Classic and early Haas years** are approximate and come from monitoring vendors and forums.
- **Polling rates** (1–10 per second) are typical figures from the machine interfaces file, not
  measured on a Haas here. A real NGC should be measured: round-trip time per `?Q` query, and how
  many queries per second the control sustains without slowing the HMI.
- **Module specs** are a design proposal, not a tested design.

## Programs

| Program or product | What it does with these machines | Licence / notes |
|---|---|---|
| MachineMetrics | Haas NGC (Ethernet MDC), classic (serial via Moxa), I/O integration (relay pulse + spindle CT) for old machines | commercial |
| FactoryWiz | Haas NGC MDC and DPRNT over TCP | commercial |
| Litmus Edge | Haas over MTConnect and DPRNT (CSV stream) | commercial |
| Autodesk Fusion Machine Connect | program transfer and monitoring, "HAAS (Classic and Next Generation)", MTConnect, OPC UA, FOCAS, DCOM | commercial, Windows service |
| MTConnect.NET-Haas (TrakHound) | Haas Ethernet and RS-232 modules for the MTConnect.NET agent | MIT |
| MTConnect Agent For Haas (TrakHound, older) | Haas MDC to MTConnect | **GPL-2.0: do not read** |
| HaasMTConnect (community) | serial adapter reading Q600 variables | "public domain" per README; no licence file |
| Hexagon PC-DMIS | drives 21 CMM controller types incl. I++ DME | commercial |
| EOSCONNECT | EOS machine data over OPC UA, REST, MQTT | commercial |
| Stratasys Connectivity SDK, MTConnect | Stratasys printers | commercial |
| Bambu Local Server, Developer Mode | Bambu printers on the LAN | free; Windows x86-64 for the server |
| PrusaLink | Prusa printers, local REST | free |
| Formlabs Local API | Formlabs printers via a local server app | free |

## Complaints and requests

- **"Too many connections" on Haas MDC**, fixed only by resetting Setting 143 (MachineMetrics
  comment). Haas allows two connections and sends replies to both (Haas), so two collectors fight.
- **Collectors ask shops to turn off HaasConnect, HaasDrop and Remote Net Share** (Chatter, in the
  machine interfaces file).
- **Fewer MTConnect data items on older NGC software** (Haas recommends 100.20.000.1200+).
- **No load in the Q list**, so every vendor adds a current sensor on older Haas machines
  (MachineMetrics I/O guide).
- **Bambu's authorisation change (early 2025; date unverified)** broke third-party tools until Developer Mode and
  Bambu Connect were offered (Bambu Lab wiki).
- **Cloud-only APIs** on some printers and CMM software block air-gapped defence shops (high-security
  file).

## What users need

**A prioritised work plan (opinion), each step reaching more users than the next:**

| Step | Work | Reaches |
|---|---|---|
| 1 | MTConnect client + agent in the Edge; OPC UA read-only client | all MTConnect/umati machines: Haas NGC (newer software), Mazak, Okuma, Hurco, Siemens/Heidenhain with options, Stratasys F900, EOS |
| 2 | Read-only I/O module + sensor kit, with calibration to control units | **every** CNC, printer and CMM, any age (state, counts, current, vibration) |
| 3 | Haas MDC connector (TCP + serial), DPRNT listener, version-aware feature set | every Haas since about 2001 |
| 4 | FOCAS2 read-only connector (out of process) | the largest control base (Fanuc C1) |
| 5 | Bambu, PrusaLink, Formlabs local connectors | the top additive brands A1–A4 |
| 6 | CMM result-file watcher (QIF, DMIS output, Q-DAS) | most CMMs (M1–M4), with no electronics |
| 7 | DNC server / sniffer for legacy RS-232 | legacy machines C11, early Haas H1 |
| 8 | Requests to makers (Haas first) and the MCP profile proposal | long-term, everyone |

**What SolidCraft should record for every machine (opinion):** builder, model, control family,
software version, which connector, which signals at what rate, read-only or not, and whether the
data path is bolt-on or factory. This goes into the machine library entry and decides what the
Edge can promise the CAM side.

## Machine tables: controls, versions, features and user base

Jason asked for a starting set of tables: machines, controllers and software versions with the
monitoring and control features each one offers, and an estimate of the user base per machine,
by continent. These tables are **a starting point to be corrected** by maintainers, makers and
shops. They also seed the machine library: see
[The machine library's capability record](#the-machine-librarys-capability-record).

**Confidence legend:** **F** = fact from a cited source in this file or the machine interfaces
file; **K** = general industry knowledge, not checked against a source here (verify before
relying on it); **E** = estimate made for this file, with its method stated. Years for control
generations are approximate (K) unless marked F.

### Table 1. CNC control generations and what each can give the Edge

Features: **Mon** = monitoring (state, program, tool, alarms); **Load** = spindle/axis load
readable through the interface; **Blk** = current block or line; **Off** = tool/work offsets
readable; **Wr** = the interface also accepts writes (so the Edge must allow-list); **Std** =
open standard (MTConnect or OPC UA) available.

| Maker | Generation (approx. years) | Interface on that generation | Mon | Load | Blk | Off | Wr | Std | Conf. |
|---|---|---|---|---|---|---|---|---|---|
| **Haas** | Early control (1988 – about 2000) | RS-232 DNC only | – | – | DNC sent-block | – | – | – | F/K |
| Haas | Classic (about 2001 – 2017), MDC Setting 143 ON/OFF | RS-232 MDC `?Q` queries, `Q600` variables | yes | via `Q600` (variable unverified) | via variable | via `Q600` | `E` unverified | – (Edge can republish as MTConnect) | F |
| Haas | NGC 100.xx (2016 –), MDC Setting 143 = port | Ethernet MDC, DPRNT over TCP, HaasConnect | yes | via `Q600` | via DPRNT markers | yes | **yes (`?E` #1–33, #10000–10999)** | MTConnect :8082 (best ≥ 100.20.000.1200) | F |
| **Fanuc** | 0 / 0-C, 15, 16/18/21 (1985 – late 1990s) | RS-232 DNC; some 16/18 with HSSB or Ethernet boards | DNC only, or limited | – | DNC | – | – | – | K |
| Fanuc | 16i/18i/21i, 0i-A/B/C (late 1990s – 2008) | FOCAS1 on Ethernet option board or HSSB | yes | yes (`cnc_rdspload`) | yes | yes | yes (FOCAS writes) | – | K |
| Fanuc | 0i-D/F, 0i-F Plus, 30i/31i/32i-A/B, -B Plus (2008 –) | FOCAS2 on embedded Ethernet (port 8193); MT-LINKi; MTConnect and OPC UA options | yes | yes | yes | yes | yes (password-gated parameters) | options | F (FOCAS) / K (years) |
| **Siemens** | System 3/8, 810/820/850/880 (1970s – 1990s) | RS-232 DNC | – | – | DNC | – | – | – | K |
| Siemens | 840C, 810D/840D powerline (1990s – 2000s) | HMI-based (Windows HMI on 840D), RS-232 | limited | via HMI tools | yes | yes | yes | – | K |
| Siemens | 840D sl, 828D, 808D (2006 –), SINUMERIK ONE (2019 –) | Access MyMachine /OPC UA (licensed, port 4840); SINUMERIK Edge (500 Hz) | yes | yes | yes | yes | yes (OPC UA methods) | **OPC UA** | F (interface) / K (years) |
| **Heidenhain** | TNC 1xx–4xx (1980s – 1990s) | RS-232 | – | – | DNC | – | – | – | K |
| Heidenhain | iTNC 530 (about 2004 –) | HEIDENHAIN DNC (option 18), Ethernet | yes | limited | yes | yes | **yes (start/stop programs)** | – | F (DNC) / K |
| Heidenhain | TNC 620 (81760x-08+), TNC 640 (34059x-10+), TNC7 (81762x-16+) | OPC UA NC Server (options 56–61, up to 6 clients) + DNC | yes | via builder extensions | yes | yes | yes (program control) | **OPC UA** | F |
| **Mazak** | Mazatrol M-series, M-Plus, Fusion 640 (1981 – early 2000s) | RS-232 | – | – | DNC | – | – | – | F (1981) / K |
| Mazak | Matrix, Matrix Nexus (2000s – 2010s) | Ethernet, MTConnect via adapter (K) | yes | partly | yes | partly | – (no public write) | MTConnect | K |
| Mazak | Smooth X/G/Ai/Ez (2013 –) | **MTConnect native**, SmartBox gateway | yes | yes | yes | yes | no public write | **MTConnect** | F |
| **Okuma** | OSP-U/E series (1990s – 2000s) | RS-232; MTConnect adapters on some | limited | – | DNC | – | – | – | K |
| Okuma | OSP-P100/P200 (2000s), P300 (2013), P500 (2020s) | **THINC API** on the Windows HMI (6,000+ data points on P300), MTConnect; OPC UA on newer | yes | yes | yes | yes | **yes (THINC writes)** | MTConnect, OPC UA (newer) | F (P300, THINC) / K |
| **Mitsubishi** | M500/M600, M60S (1990s – 2000s) | RS-232, Ethernet on later | limited | – | DNC | – | – | – | K |
| Mitsubishi | M70/M700, M80/M800, M800V (2005 –) | Ethernet, maker API library, MTConnect | yes | yes | yes | yes | yes (API) | MTConnect | K |
| **Brother** | CNC-B00/C00 (2000s – 2010s) | Ethernet, RS-232 | yes | limited | yes | yes | unknown | – | K |
| Brother | CNC-D00 (current) | OPC UA, Ethernet, fieldbus slave options | yes | yes (K) | yes | yes | "computer remote" (undocumented) | **OPC UA** | F |
| **Hurco** | Ultimax (1980s – 2000s) | RS-232 | – | – | DNC | – | – | – | K |
| Hurco | WinMax (2000s –) | Ethernet, MTConnect (unverified) | yes | unknown | yes | yes | unknown | MTConnect | K |
| **Chinese domestic** | GSK 980/928, KND K1000, Huazhong HNC-808/818/848 | maker protocols; NC-Link on newer Huazhong (K) | varies | varies | varies | varies | varies | NC-Link | F (models) / K |
| **Tormach** | PathPilot (LinuxCNC-based) | network interfaces not checked (do not read LinuxCNC source) | unknown | unknown | unknown | unknown | unknown | – | K |
| **Legacy others** | Fadal CNC88, Milacron Acramatic, Allen-Bradley, Bridgeport BOSS | RS-232 DNC, or tape | – | – | DNC | – | – | – | K |

**Every row's "–" is where the bolt-on module does the work:** stack light and relay inputs for
state, current sensors for load, the DNC line for program position.

### Table 2. CNC machine families: builder, model, control, Edge route

| Rank (control) | Builder | Model families | Controls fitted | Edge route, best first |
|---|---|---|---|---|
| C6 | **Haas** | VF vertical mills, VM, UMC 5-axis, EC horizontals, TM toolroom, Mini/Super Mini Mill, DT/DM drill-tap, ST/SL lathes, TL toolroom lathes, DS twin-spindle, Office/Desktop mill | Haas (early, classic, NGC) | NGC: MTConnect + MDC + DPRNT; classic: serial MDC; early: module |
| C1 | **DN Solutions** (formerly Doosan) | DNM, DVF, Puma, Lynx | Fanuc mostly; some Siemens (K) | FOCAS2 / MTConnect option |
| C1 | **Hyundai WIA**, Hwacheon, Smec | KF, L-series; Vesta, Hi-TECH | Fanuc, some Siemens/Hwacheon own (K) | FOCAS2 |
| C1 | **Makino, Matsuura, Kitamura, Nakamura-Tome, Takisawa** | PS/a-series, MAM, Mycenter, WT | Fanuc (Makino Pro on Fanuc), some Mitsubishi (K) | FOCAS2 |
| C1 | **Taiwanese builders** (Hartford, Feeler, Victor, Leadwell, YCM, Awea, Quaser) | many | Fanuc, Mitsubishi, Siemens, Heidenhain (K) | per control |
| C2/C4 | **DMG Mori** | DMU, DMC, CMX, NLX, NTX, CTX | Siemens, Heidenhain, Fanuc, Mitsubishi by model and region (K) | OPC UA / umati (CELOS, IoTconnector) |
| C2/C4 | **Hermle, Grob, Chiron, Spinner, Index** | C-series, G-series, FZ/DZ, VC, G-series lathes | Heidenhain or Siemens (K) | OPC UA |
| C7 | **Mazak** | VCN, VCU, Variaxis, Integrex, QT, Quick Turn, HCN | Mazatrol; some Fanuc, Siemens (K) | MTConnect (native) |
| C8 | **Okuma** | Genos, MB, MU, LB, LU, Multus | OSP | THINC (read-only use) / MTConnect |
| C9 | **Brother** | SPEEDIO S/R/M/U series | CNC-D00 | OPC UA |
| C10 | **Hurco** | VM, VMX, BX, TM lathes | WinMax | MTConnect |
| C10 | **Tormach** | 440, 770, 1100, 1500 | PathPilot | module first; network API to be checked |
| C5 | **Citizen, Star, Tsugami** (Swiss-type) | L, M, SR, SB | Mitsubishi or Fanuc (K) | per control |
| C11 | **Legacy** | Fadal VMC, Bridgeport, Cincinnati, Milltronics | own or legacy | module + DNC |

### Table 3. Additive machines

| Rank | Brand | Models | Software / firmware version notes | Monitoring interface | Control interface (for read-only Edge: do not use) | Conf. |
|---|---|---|---|---|---|---|
| A1 | Bambu Lab | X1, P1, A1, H2 | authorisation-gated commands after the 2025 firmware change; **Developer Mode** (LAN only) behaves as before | LAN status (Developer Mode), Bambu Local Server HTTP API | print control in the same channels | F |
| A2 | Creality / Elegoo / Anycubic | K1, K2, Ender (Creality); Centauri, Neptune, Saturn (Elegoo); Kobra, Photon (Anycubic) | Klipper-based on many FDM models; resin printers vary | Moonraker network API where Klipper (GPL project: network use only) | same API | K |
| A3 | Prusa | MK4/MK4S, Core One, XL, Mini | PrusaLink built into recent firmware | PrusaLink local REST | same API | F/K |
| A4 | Formlabs | Form 3, Form 4/4L, Fuse 1 | Local API needs a server app on a PC | Local API; Web API (cloud) | job send | F |
| A5 | UltiMaker | S3/S5/S7, Factor 4, Method | local REST API (community-documented) | REST | REST | K |
| A6 | Stratasys | F123, F900, Fortus 450mc, J-series PolyJet, Origin | Connectivity SDK on Fortus/F900/F123/J3–J8 | **MTConnect** (F900, Fortus 450mc), SDK | SDK (two-way) | F |
| A7 | HP | MJF 4200/5200/5420 series | not researched | maker software | – | K |
| A8 | EOS | M 290, M 300-4, M 400/400-4, P 396, P 500 | EOSCONNECT Core 1.0 (M 290, M 400, M 400-4), 1.2 (M 300-4, P 500), MQTT from 1.6 | **OPC UA**, REST Web API, MQTT | build start via OPC UA with Control licence | F |
| A8 | BLT, Eplus3D, Farsoon, ZRapid, Nikon SLM | BLT-S, EP-M, FS, iSLM; NXG, SLM 280/500 | not researched | OPC UA likely (unverified) | – | K |

### Table 4. CMMs and inspection

| Rank | Brand | Machines | Controller / software | Read-only Edge route | Conf. |
|---|---|---|---|---|---|
| M1 | Hexagon | Global, Optiv, Leitz PMM/Reference, Tigo | DC-series controllers; PC-DMIS (21 controller interfaces incl. I++ DME client) | PC-DMIS reports, DMIS output, QIF, Q-DAS export; stack light | F (interfaces) / K |
| M2 | Zeiss | CONTURA, PRISMO, ACCURA, O-INSPECT, DuraMax | Zeiss controllers (I++ DME server); CALYPSO; PiWeb | CALYPSO table files, PiWeb, QIF | F/K |
| M3 | Mitutoyo | Crysta-Apex, Strato-Apex, MiSTAR | UC controllers (I++ DME; GPIB on older); MCOSMOS; MeasurLink | MCOSMOS results, MeasurLink database | F/K |
| M4 | Nikon, Wenzel, LK, Aberlink, Coord3, Keyence, FARO, Creaform | bridge CMMs, arms, trackers, handheld scanners | own software; I++ DME on several | result files | K |
| M5 | Renishaw (on-machine) | OMP/RMP probes, NC4 and OTS tool setters | probing macros on the CNC; Renishaw Central | DPRNT records; CNC interface | F/K |
| M6 | Renishaw (CMM and gauging) | Equator, REVO heads, UCC controllers; RenAM (additive) | UCC (I++ DME); MODUS | result files; Renishaw Central | F/K |

### Table 5. User base estimates by continent

**Method (E):** units *still in service* in 2026 = cumulative units built (fact where cited,
extrapolated otherwise) × a survival share (assumed 50–75 %, since machine tools run 20–30 years
and many are resold rather than scrapped). The continent split is an estimate from where each
maker sells, anchored on 2025 machine tool consumption by country: China US$26.4 billion, the US
US$11.4 billion, Germany US$5.13 billion (Gardner World Machine Tool Survey, 2026 edition). Asia
is therefore the largest market for most makers, and the US the second. **Every number in this
table is an order of magnitude, not a count.** Ranges are deliberately wide.

| Rank | Control family / brand | Built to date (basis) | In service 2026 (E) | Americas | Europe | Asia | Rest | Conf. |
|---|---|---|---|---|---|---|---|---|
| C1 | Fanuc CNC | 5 M by Feb 2022 (F); about 250 k/year 2018–22 (F, from milestones) → about 6 M by 2026 (E) | 3.5–4.5 M | 20 % | 20 % | 58 % | 2 % | medium |
| C2 | Siemens SINUMERIK | no figure; share about half of Fanuc's (E from Table 1 sources) | 1.5–2.5 M | 12 % | 45 % | 40 % | 3 % | low |
| C3 | Chinese domestic (GSK, Huazhong, KND …) | Huazhong 300 k cumulative; GSK near 100 k/year (report, E) | 1.5–3 M | 1 % | 1 % | 97 % | 1 % | low |
| C4 | Heidenhain TNC | 200 k by June 2007 (F); about 15–25 k/year since (E) | 0.35–0.55 M | 12 % | 60 % | 26 % | 2 % | low |
| C5 | Mitsubishi M-series | no figure | 0.5–1 M | 12 % | 8 % | 78 % | 2 % | low |
| C6 | **Haas** | 125 k by Feb 2012, 200 k by about 2018 (F) → about 12 k/year → about 300 k by 2026 (E) | **0.2–0.27 M** | **65 %** | 22 % | 11 % | 2 % | medium (total) / low (split) |
| C7 | Mazak (Mazatrol) | no figure | 0.15–0.3 M | 30 % | 25 % | 43 % | 2 % | low |
| C8 | Okuma OSP | no figure | 0.1–0.2 M | 35 % | 15 % | 48 % | 2 % | low |
| C9 | Brother CNC | no figure | 0.1–0.3 M | 10 % | 10 % | 79 % | 1 % | very low |
| C10 | Hurco, Tormach, Fagor, Centroid, others | no figure | 0.1–0.3 M together | 50 % | 35 % | 13 % | 2 % | very low |
| C11 | Legacy, pre-network (all makers) | — | 0.5–1.5 M | 30 % | 30 % | 38 % | 2 % | very low |
| A1–A2 | Entry-level printers (Bambu Lab 37 % of 2025 entry-level shipments, F; Creality, Elegoo, Anycubic) | several million units a year in total (E) | 10–20 M, Bambu Lab several million (E) | 30 % | 30 % | 37 % | 3 % | low |
| A3–A5 | Prusa, Formlabs (38 % of 2025 professional shipments, F), UltiMaker | — | 0.5–1.5 M together (E) | 40 % | 40 % | 18 % | 2 % | low |
| A6–A9 | Industrial polymer and metal (EOS 5,000 installed by March 2025, F; Stratasys, HP, 3D Systems, Chinese metal PBF) | — | 50–150 k (E) | 30 % | 35 % | 33 % | 2 % | low |
| M1–M4 | CMMs (Hexagon, Zeiss, Mitutoyo, others) | — | 150–400 k (E) | 25 % | 35 % | 38 % | 2 % | very low |

**Haas by model family (E, very low confidence):** VF and VM vertical mills about 45 %; ST/SL
lathes about 25 %; Mini/Super Mini Mill, TM and TL about 12 %; UMC 5-axis about 6 %; EC
horizontals, DT/DM and others about 12 %. By generation in service in 2026 (E, from about 12 k
machines a year since 2016): NGC about 45–50 %, classic with MDC about 40–45 %, early controls about
5–10 %. **So a classic serial connector matters almost as much as the NGC connector.**

### How to firm up these numbers

1. **Edge installs count themselves.** Every Edge reads `?Q100`/`?Q102` (Haas), `cnc_sysinfo`
   (Fanuc), the MTConnect `/probe` header or the OPC UA `Identification` node. With the shop's
   opt-in, an anonymous count by model, control and software version is the best data anyone
   could have about real machines in small shops (opinion).
2. **Ask the makers.** Fanuc, Heidenhain and Haas publish milestones; the "machine MCP profile"
   pitch is a natural moment to ask for installed-base figures by region.
3. **Used-machine listings** as a proxy for model mix: sample a few thousand listings per region
   and count by builder, model and control.
4. **Buy the Gardner data** (consumption by country and type) if the maintainers want a firmer
   continent split.

### The machine library's capability record

Jason's point: **the machine library has to know which features and capabilities each machine
has.** The machine library file owns that model: its
[Capability model](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-machine-library/research/cam-research-nsnlookupjn-machine-library.md#capability-model-which-features-each-machine-has-and-how-we-know-jasons-requirement) keys a capability set on (machine model, control family and generation,
software version range, options fitted), groups the flags (data interfaces, monitoring signals,
motion and smoothing, load monitor and autofeed, probing, machine options, physics data) and
records provenance per flag (discovered, verified by test, maker-declared, user-declared, unknown
= treated as absent). **The tables in this section are seed data for that model.** This file adds
only how the Edge fills each group on its own (opinion):

**Granular to the individual machine (Jason):** "We must be granular through manufacturer
libraries to extract the most performance from every individual machine configuration and
version." The machine library file's Capability model handles this with **three layers, from
general to individual** (maker library, configuration, individual machine; see its
[Capability model](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-machine-library/research/cam-research-nsnlookupjn-machine-library.md#capability-model-which-features-each-machine-has-and-how-we-know-jasons-requirement)). The Edge's part is to fill the individual layer: discovered and verified
flags and the machine's own measured physics (real spindle power curve, block rate, chatter
frequencies).

| Capability group (machine library) | How the Edge discovers it | Example (Haas VF-2, NGC) |
|---|---|---|
| Identity and version (the key) | `?Q100`/`?Q101`/`?Q102`; FOCAS `cnc_sysinfo`; MTConnect `/probe` header; OPC UA `Identification` | VF-2, serial 1234567, software 100.20.000.1200 |
| Data interfaces | port scan of known ports on the machine segment only (MDC port in Setting 143, 8082, 4840, 8193), then a harmless identity query on each | `haas-mdc-tcp` 5051, `mtconnect` 8082, `dprnt-tcp` 8001 |
| Monitoring signals | MTConnect `/probe` data items; OPC UA browse; a test read of each candidate variable; module channel map | spindle load % at 1 Hz (MTConnect); spindle current at 10 kHz (module) |
| Writes the interface allows | known per connector, **recorded even though the Edge never uses them**, so auditors see the exposure | `?E` on #1–33 and #10000–10999 |
| Built-in environment sensors | MTConnect `/probe` (temperature, pressure, voltage items), OPC UA browse, known alarm numbers for low air or power | casting/spindle temperature if published; low-air alarm (declared) |
| Motion, load monitor, probing, options | from the version via Table 1 (maker-declared), then **verified by test** (a G187 block accepted, a DPRNT line received, a probe cycle result, the fingerprint cut) | G187 yes (verified); Setting 84 autofeed (declared); WIPS probe (verified) |

## Export control, sanctions and government use

*Added 2026-10-29 from the [government use research](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-qk5riz/research/gov-research-nsnlookupjn-government-use.md). Opinion and policy proposals, not legal advice.*

**What in this file crosses a border:** the SolidCraft Edge box, its REST/MQTT/WebSocket links to CAM, remote dashboards, brand integrations and MCP pitches to machine makers; Edge is an in-scope asset in a controlled shop, and the open hardware design itself must stay generic.

- **Hosted parts block sanctioned regions and screen people.** Any service this file proposes that the project would run blocks comprehensively sanctioned regions, plus Russia and Belarus (US, EU and UK rules restrict CAD/CAM software and related services there), and screens accounts, payers, contributors and marketplace parties against the US Consolidated Screening List and the UK, EU and UN lists.
- **No support there.** No support, consulting or custom work for users in sanctioned regions or for listed parties.
- **Government mode.** At the Controlled level (CUI and allied equivalents) online parts are off or point to the organisation's own mirror; at the Isolated level they arrive only as signed packs on media; a design flagged Export controlled (ITAR/EAR or national equivalent) never leaves through them.
- **Only generic data is shared.** Nothing derived from a controlled design (geometry, toolpaths, part or drawing numbers, test or inspection results) goes into a shared library, report or issue. Tool data stays shareable through the field allow-list in the high-security research.
- **Code stays public.** Every build the project ships is public, which keeps it "published" software outside the EAR. The rules are tracked by the standing export and sanctions watch.
- **Checked for controlled data (2026-10-29):** this file contains no controlled technical data. Its examples are public sources, generic values or made-up stand-ins, and the author's experience is stated in general terms.

## Glossary

- **BGA (ball grid array):** a chip package with solder balls underneath instead of pins.
- **CT (current transformer):** a clamp sensor for AC current; poor at very low frequency.
- **Closed-loop Hall sensor:** a current sensor that measures DC and AC, good on drive outputs.
- **DNC / drip-feed:** sending a program to the control over RS-232 while it runs.
- **DPRNT:** a macro command that prints text and variable values from a running program to a
  port.
- **FOCAS2:** Fanuc's library for reading (and writing) control data over Ethernet.
- **HFO (Haas Factory Outlet):** Haas's local sales and service outlets.
- **I++ DME:** the open protocol between CMM software (client) and the CMM controller (server).
- **MDC (Machine Data Collection):** Haas's query interface on Setting 143 (`?Q` and `?E`
  commands).
- **MTConnect:** an open, read-only HTTP/XML standard for machine data.
- **NGC:** Haas's Next Generation Control (2016 and later).
- **OPC UA for Machine Tools (OPC 40501-1, umati):** the VDW/OPC Foundation information model for
  machine tools.
- **PTP (Precision Time Protocol):** network clock synchronisation to microseconds.
- **Q-DAS / AQDEF:** a widely used file format for measurement data and SPC.
- **QIF:** the Quality Information Framework, an ISO standard for inspection plans and results.
- **SPAN / mirror port:** a switch port that copies another port's traffic.
- **Status relays (Haas):** 8M relay outputs 90–97 used as machine status outputs when Setting
  315.26 is on.

## Open questions

1. **Haas variable numbers:** which system variables give spindle load, axis loads and the current
   block on NGC and classic, and how fast `?Q600` answers on a real machine.
2. **Haas DPRNT network settings** (261, 262, 263) and the status relay mapping for outputs 90–97.
3. **Does the classic control accept `E` writes?** It does not matter to the read-only module, but
   it matters for the security argument.
4. **Hosting Edge inside SolidCraft:** Jason proposes an open hardware design with embedded Rust
   software and OS as a SolidCraft project (see
   [SolidCraft Edge as an open hardware project](#solidcraft-edge-as-an-open-hardware-project-inside-solidcraft)).
   Maintainers decide whether it lives in this repository, how the wasm rule applies to it, and
   which open hardware licence to use (CERN-OHL-P-2.0 suggested).
5. **NC-Link** (China's machine data standard): worth a connector, or out of scope?
6. **I++ DME read-only:** is there a safe way to read CMM state without being the motion client?
   Worth asking the I++ DME 3.0 group.
7. **Name:** SolidCraft Edge is a working name; Siemens uses "SINUMERIK Edge" (machine interfaces
   file). Maintainers decide.
8. **Market data:** a paid report (or the makers) would firm up the control shares; is that worth
   it, or is the order enough?

## Sources

### Haas

- Haas, Machine Data Collection – NGC: https://www.haascnc.com/service/troubleshooting-and-how-to/how-to/machine-data-collection---ngc.html
- Haas, M-Code Relay Function – NGC (RD0041): https://www.haascnc.com/service/troubleshooting-and-how-to/reference-documents/m-code-relay-function---ngc.html
- Haas, HaasConnect: https://www.haascnc.com/productivity/control/haas-connect.html
- Haas, Communication with External Devices – DPRNT (not fetched): https://www.haascnc.com/service/troubleshooting-and-how-to/how-to/communication-with-external-devices---dprnt.html
- MachineMetrics, Haas serial MDC: https://support.machinemetrics.com/hc/en-us/articles/229050328-How-to-Collect-data-using-HAAS-Serial-Machine-Data-Collection
- MachineMetrics, How to determine if a Haas is Next Gen, Classic Serial or I/O: https://support.machinemetrics.com/hc/en-us/articles/1500005987301-How-to-Determine-if-my-HAAS-Machine-is-Next-Gen-Classic-Serial-or-I-O
- MachineMetrics, Haas I/O integration: https://support.machinemetrics.com/hc/en-us/articles/360037481193-HAAS-I-O-Integration
- CNC Arena forum, machine data collection (Q list): https://en.cncarena.com/forum/thread/155953-machine-data-collection/
- FactoryWiz, Haas NGC connection and DPRNT test: https://kb.factorywiz.com/portal/en/kb/articles/haas-ngc-connection-and-dprnt-test
- eMastercam forum, spindle load macro test: https://www.emastercam.com/forums/topic/114599-spindle-load-macro-test/
- mfgrobots, Haas autofeed and tool load monitoring: https://www.mfgrobots.com/Article/equipment/cnc/46178.html
- Industrial Monitor Direct, DIY OEE monitoring (protocol vs I/O): https://industrialmonitordirect.com/hu/blogs/knowledgebase/diy-oee-monitoring-with-esp32-protocol-based-vs-io-collection-methods
- TrakHound, MTConnect.NET-Haas (MIT): https://github.com/TrakHound/MTConnect.NET-Haas
- TrakHound, MTConnect Agent For Haas (GPL-2.0; README only): https://github.com/TrakHound/MTConnect-Agent-For-Haas
- HaasMTConnect (README only): https://github.com/nombreinvicto/HaasMTConnect
- Litmus, Haas via MTConnect and DPRNT: https://litmus.io/solutions/connecting-to-haas-cnc-controller
- American Machinist, "24 years, 125,000 CNCs" (February 2012): https://www.americanmachinist.com/machining-cutting/24-years-125000-cncs
- machinery.co.uk, Haas installs 200,000th machine: https://www.machinery.co.uk/content/news/haas-installs-200-000th-machine
- West Manufacturing News, Improved CNC Control (December 2016): https://mfgnewsdigital.mydigitalpublication.com/article/Improved+CNC+Control/2648094/361279/article.html
- Wikipedia, Haas Automation: https://en.wikipedia.org/wiki/Haas_Automation

### Market and other makers

- Modern Machine Shop, 58th World Machine Tool Survey (2025 consumption by country): https://www.mmsonline.com/articles/58th-annual-world-machine-tool-survey-report-and-data-now-available-2
- Okuma, History of the P-Control: https://okuma.com/blog/history-of-the-p-control

- Fanuc, 5 million CNCs (17 March 2022): https://www.fanuc.co.jp/en/profile/pr/newsrelease/2022/notice20220317.html
- machinery.co.uk, Heidenhain 200,000th control: https://www.machinery.co.uk/content/news/heidenhain-ships-200-000th-machine-control-unit
- CNC Machines, top CNC controls (uncited estimates): https://cncmachines.com/top-cnc-controls-history-market-share-pros-cons
- China CNC controller report 2026 (estimates): https://faxiangongchang.com/en/reports/china-cnc-controller-2026
- GM Insights, CNC controller market: https://www.gminsights.com/industry-analysis/cnc-controller-market
- all-about-industries, Top 100 machine tool report 2025 (hpo forecasting): https://www.all-about-industries.com/the-100-largest-machine-tool-manufacturers-in-the-world-a-241ea3b70de92afdbd35f0f24c29dc2c/
- Sirris, umati becomes the universal smart factory language: https://www.sirris.be/en/inspiration/umati-becomes-universal-smart-factory-language

### Additive

- CONTEXT, Q4 2025 3D printer shipments (15 April 2026): https://www.contextworld.com/quarterly-industrial-3d-printer-shipments-rebound-amid-continued-record-breaking-entry-level-growth
- Bambu Lab wiki, third-party integration: https://wiki.bambulab.com/en/software/third-party-integration
- Formlabs Local API: https://formlabs.com/uk/support/Formlabs-Local-API/
- EOS, 5,000 machines installed: https://www.eos.info/content/blog/2025/5000-machines-installed
- EOS, EOSCONNECT: https://uk.eos.info/en-gb/industrial-3d-printer/software/software-releases/eosconnect
- Stratasys, connectivity and IIoT: https://www.stratasys.com/en/software/connectivity-and-iiot/
- PrusaLink (community API reference): https://hexdocs.pm/prusa_link/api-reference.html

### Sensors

- ifm, IO-Link in detail: https://www.ifm.com/fr/en/shared/io-link/technology/io-link-in-detail
- IO-Link community, IO-Link Wireless specification completed (April 2018): https://io-link.com/en/Global/newsmeldung.php?showId=567
- Nordic Semiconductor, wireless sensor reports industrial asset health (July 2020): https://www.nordicsemi.com/Nordic-news/2020/07/Wireless-sensor-reports-industrial-asset-health-to-technicians

- Raspberry Pi 5 prices (travis.media, July 2026 snapshot): https://travis.media/blog/buying-your-first-raspberry-pi-5/
- SCT-013-000 (Walmart listing): https://www.walmart.com/ip/1663482259
- Shelly Pro 3EM V2: https://us.shelly.com/products/shelly-pro-3em-v2
- Eastron SDM630MCT (Bimble Solar): https://www.bimblesolar.com/EnergyMeter-3-phase-SDM630MCT
- ifm VVB001 (Bürklin): https://buerklin.com/en/p/ifm-electronic/motion-sensors/vvb001/82P8793
- ifm AL1350 (Eltra Trade): https://eltra-trade.com/products/ifm-electronic-al1350
- PCB 603C01 price (instrumentation.co.uk, 2016): https://instrumentation.co.uk/pcb-piezotronics-reduces-pricing-for-industrial-accelerometers-shear-mode-quality-only-%c2%a399/
- ProSense SPTD25 (2016 launch): https://www.plantautomation-technology.com/industrialtrends/automationdirect-announces-prosense-sptd25-pressure-transmitters
- Vaisala DMT143 (AxFlow): https://www.axflow.com/catalog/products/instruments/humidity-instruments/vaisala-dmt143
- RuuviTag (Verkkokauppa): https://www.verkkokauppa.com/en/product/613490/RuuviTag-Multi-Functional-Bluetooth-Sensor
- miniDSP UMIK-1 (Parts Express): https://www.parts-express.com/miniDSP-UMIK-1-Omni-directional-USB-Measurement-Calibrated-Microphone-230-332
- Reolink RLC-510A (123SecurityProducts): https://www.123securityproducts.com/security-cameras/bullet-cameras/rlc-510a.html
- EVAL-ADXL355-PMDZ (Digi-Key): https://www.digikey.com/es/products/detail/analog-devices-inc/EVAL-ADXL355-PMDZ/7324256
- LEM HLSR series (Digi-Key): https://www.digikey.com/en/product-highlight/l/lem-usa/hlsr-series-industrial-current-sensors
- RS-485 SHT30 sensor (KinCony): https://shop.kincony.com/products/rs485-temperature-and-humidity-sensor-modbus-rtu-use-by-sht30

### CMM and probing

- Modern Machine Shop, Renishaw data platform (September 2023): https://www.mmsonline.com/products/renishaw-data-platform-promotes-connected-shopfloor
- Modern Machine Shop, process control with machine shop connectivity (August 2023): https://mmsonline.com/articles/process-control-leveraging-machine-shop-connectivity-in-real-time
- Hexagon, PC-DMIS CMM supported interfaces: https://docs.hexagonmi.com/pcdmis/2020.2/en/helpcenter/mergedProjects/miim/CMM_Supported_Interfaces.htm
- VDW, I++ DME interface standard (July 2025): https://vdw.de/en/length-measurement-technology-interface-standard-boosts-automation/
- GM Insights, CMM market share: https://gminsights.com/industry-analysis/coordinate-measuring-machine-market/market-share

### Law

- Finnegan, final rule in the ninth triennial §1201 proceeding (October 2024): https://www.finnegan.com/en/insights/ip-updates/final-rule-issued-in-the-us-copyright-offices-ninth-triennial-section-1201-proceeding.html
