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
- **Renishaw gets its own pitch**, because its probes sit on almost every brand in the ranking:
  one documented probe-result record on every control, and Renishaw Central serving the same
  read-only profile (see [A pitch to Renishaw](#a-pitch-to-renishaw)).
- **"Make everything standard" with MCP:** the machine interfaces file already sketches a
  machine-tool MCP tool set with no motion tools. This file adds a **read-only-first "machine MCP
  profile"**, mapped onto MTConnect and umati names so makers can serve it from data they already
  have, and a per-maker pitch in ranking order (see [One standard: a machine MCP profile](#one-standard-a-machine-mcp-profile)).

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
| **C9** | **Brother CNC-C00** (SPEEDIO compact machines) | not found (large unit counts in electronics manufacturing; unverified) | medium | MTConnect and FTP-based options (unverified) | sensor kit |
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

### Rust software on the Edge (sketch, opinion)

- One **connector per interface**: `haas-mdc` (TCP and serial), `mtconnect` (HTTP client and an
  agent for republishing), `dprnt` (TCP and serial listener), `opcua` (client, read-only), `focas`
  (out of process, because the Fanuc library is a native binary), `io-module` (SPI ADC and GPIO).
- Each connector declares which **signals it can give and at what rate**, and fills the machine
  library's "Signals the machine publishes" for that machine.
- **Crates must pass SolidCraft's licence check** (`cargo xtask licences --check`); permissive crates
  for serial ports, GPIO and SPI exist, but each must be checked before use. Nothing under GPL, so
  no LinuxCNC, Klipper or OctoPrint code.

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

## One standard: a machine MCP profile

Jason's goal is to suggest an MCP project to every maker and make everything standard. The machine
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
4. **Should SolidCraft ship a hardware design at all**, or only a specification that third parties
   build? An open hardware licence (for example CERN-OHL-P, which is permissive) would be needed
   for the board; that is a maintainer decision.
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

### CMM and probing

- Modern Machine Shop, Renishaw data platform (September 2023): https://www.mmsonline.com/products/renishaw-data-platform-promotes-connected-shopfloor
- Modern Machine Shop, process control with machine shop connectivity (August 2023): https://mmsonline.com/articles/process-control-leveraging-machine-shop-connectivity-in-real-time
- Hexagon, PC-DMIS CMM supported interfaces: https://docs.hexagonmi.com/pcdmis/2020.2/en/helpcenter/mergedProjects/miim/CMM_Supported_Interfaces.htm
- VDW, I++ DME interface standard (July 2025): https://vdw.de/en/length-measurement-technology-interface-standard-boosts-automation/
- GM Insights, CMM market share: https://gminsights.com/industry-analysis/coordinate-measuring-machine-market/market-share

### Law

- Finnegan, final rule in the ninth triennial §1201 proceeding (October 2024): https://www.finnegan.com/en/insights/ip-updates/final-rule-issued-in-the-us-copyright-offices-ninth-triennial-section-1201-proceeding.html
