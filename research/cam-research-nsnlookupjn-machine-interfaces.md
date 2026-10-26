# CAM research: what CNC machines let software see and do, and a proposal for MCP on the machine

Research for [#1621](https://github.com/tangency-labs/solidcraft/issues/1621) (Research Request
for Plugin: CAM). This file covers one part of that request: **the monitoring and control
interfaces CNC machine makers provide today, how a CAM program talks to a Haas mill for
in-process control and probing, a proposal that machine makers add an MCP (Model Context
Protocol) interface with a safety model, and SolidCraft running at the machine on a small mini
PC**.

Jason's questions, in his words:

> Research what interfaces for monitoring and control do cnc machine manufacturers currently
> provide.

> Also consider the abilities for fusion machine connector to a haas vf mill that runs over
> ethernet back to fusion for machine control in-process as well as in-process probing and final
> inspection. We would investigate the Haas (historically most common brand for entry to mid
> level cnc shops). We would suggest all CNC machine makers add an MCP interface to their
> machines to allow monitoring and machine control by our edge device. [...] We could use these
> [cameras] to detect work or workholding slip (in utopia).

> Can we read spindle torque and axis load in realtime to detect/predict tool failure and or
> setup problems or generally bad cutting behaviour. Use PI box? Can solidcraft run at the
> machine on a small mini pc with a simple screen and mouse to really integrate the process
> right into the machine and feedback to the design and CAM process.

It builds on a set of files from the same author and does not repeat them:

| Topic | File | What this file takes from it |
|---|---|---|
| In-process inspection and SolidCraft Edge | [cam-research-nsnlookupjn-in-process-inspection.md](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-y2w06n/research/cam-research-nsnlookupjn-in-process-inspection.md) | **SolidCraft Edge** (the Raspberry Pi box) (hardware, software, retrofit sensor kit), aligning data with G-code blocks and CAM segments, cameras for probing tasks, "corrections stay in the control" |
| Machine health | [cam-research-nsnlookupjn-machine-health.md](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-9w3gq9/research/cam-research-nsnlookupjn-machine-health.md) | what the signals *mean*: residual = measured / predicted per segment, tool condition (chipped, broken, worn), trends, work orders |
| Cell automation | [cam-research-nsnlookupjn-cell-automation.md](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-g08nas/research/cam-research-nsnlookupjn-cell-automation.md) | three control layers (safety / real-time handshake / supervision), write tiers Record → Advise → Stage → Commit, MTConnect Interfaces as handshake vocabulary |
| Workholding | [cam-research-nsnlookupjn-workholding.md](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-workholding/research/cam-research-nsnlookupjn-workholding.md) | slip and lift physics and detection from load signals |
| Machine library | [cam-research-nsnlookupjn-machine-library.md](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-machine-library/research/cam-research-nsnlookupjn-machine-library.md) | "Signals the machine publishes": each machine entry lists its signals, units and rates |
| High-security work | [security-research-nsnlookupjn-high-security.md](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-2fa4g1/research/security-research-nsnlookupjn-high-security.md) | SolidCraft Edge in a controlled shop: wired only, no inbound ports, segmented OT network |
| Aging eyes UI | [ui-research-nsnlookupjn-aging-eyes.md](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-kc0dit/research/ui-research-nsnlookupjn-aging-eyes.md) | large-target, high-contrast screens for the machine station |
| ITAR and quality compliance | [compliance-research-nsnlookupjn-itar-quality.md](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-ymp2ma/research/compliance-research-nsnlookupjn-itar-quality.md) | how an accepted change from the shop floor becomes a new revision |

**Naming:** the Raspberry Pi computer at the machine is called **SolidCraft Edge** (Jason's name: it sits on the edge of CAM and integrates the machine and automation back into CAM for closed-loop process improvement). Older files call it "the Pi box" or "the edge box".

**Ownership (so nothing is said twice):** this file owns **which controls expose which signals
and commands, at what rate, through which interface**, the **safety model for anything that
writes to a control** (messages, offsets, feed hold, stop requests), the **MCP proposal**, and
**SolidCraft at the machine on a mini PC**. SolidCraft Edge hardware stays in the inspection file;
what a load or vibration pattern means for the tool, the machine or the hold stays in the machine
health and workholding files.

Licence: MIT OR Apache-2.0, the same as SolidCraft (see the Licence section of CONTRIBUTING.md).

## About this research

- **Author:** Jason ([@nsnlookupjn](https://github.com/nsnlookupjn)). Idea and direction by Jason;
  source research and writing done with an AI assistant (Claude) in October 2026.
- **How it was produced:** web searches of public sources (control makers' product pages and
  brochures, Autodesk's public help pages and product page, the OPC Foundation's online reference
  and NuGet listings, the MTConnect Institute's document pages, connectivity guides published by
  monitoring vendors, a KIT research dataset record, the MCP project's blog), each claim checked
  against the cited page where possible. Several Haas pages could not be fetched from the session
  (the site refused the request), so Haas details come from third-party connectivity guides and
  are marked. No commercial CAD/CAM program was run, driven or scripted (Fusion was not run for
  this file), no vendor SDK was downloaded, no GPL/LGPL/AGPL source was read (LinuxCNC, which
  some machines use, is GPL and was not read), and no text is copied beyond short quotes.
- **SolidCraft facts** were checked in the repository at commit `2016ba7` (October 2026): the MCP
  server (`solidcraft-cli mcp`, `docs/mcp.md`), the local control channel
  (`docs/control-protocol.md`), Linux ARM64 packages and the browser build (`docs/downloads.md`).
  There is no CAM code on `main` yet; see #1153 (2.5D CAM RFC) and PR #4158 (2.5D CAM core).
- **Units:** millimetres, micrometres (µm), seconds and milliseconds, hertz.
- **Marking:** "(unverified)" marks a claim not confirmed against a source. **Fact**,
  **opinion** and **marketing** are labelled where they mix.

> **[TODO Jason: personal experience]** Which controls are in your shop (Haas NGC or classic,
> Fanuc, Siemens, others), whether they are networked today, whether you have used Fusion's
> live inspection connection or Machine Connect and what went wrong, whether you use Haas tool
> load monitoring, and which crashes or broken tools you would most like the machine to catch.
> Label this as personal experience.

## Summary

**Every major control maker offers a monitoring interface, almost none offer a safe, general
way for outside software to act on the machine, and none offer an AI-agent interface yet.** The
landscape in October 2026, in one table (details and sources in
[What each control maker offers](#what-each-control-maker-offers)):

| Maker / control | Monitoring | Writing / control from outside | Notes |
|---|---|---|---|
| **Haas** NGC (2016+), classic | Machine Data Collection "Q commands" over RS-232 (classic) or Ethernet (NGC, Setting 143 = port); MTConnect on recent NGC software; HaasConnect phone alerts | macro variables (limited, unverified); files over Net Share / HaasDrop; no remote cycle start found | Q commands give status, counters and cycle times, not spindle load; load needs a macro variable read or an add-on sensor |
| **Fanuc** 0i/30i series | FOCAS2 library over Ethernet/HSSB; MT-LINKi monitoring (servo/spindle currents, overrides, alarms) | FOCAS writes macro variables, tool and work offsets, parameters (password gated) | Library is Windows-documented, Linux build reported; option bits; port 8193 |
| **Siemens** SINUMERIK 840D sl, 828D, ONE | Access MyMachine /OPC UA (licensed); SINUMERIK Edge apps capture NC data at 500 Hz | OPC UA methods for files, tools and magazine; Edge apps run beside the NC | Highest-rate control data found in this research |
| **Heidenhain** TNC 620/640/TNC7 | OPC UA NC Server (options 56–61, up to 6 clients); HEIDENHAIN DNC (option 18, RemoTools SDK, Windows COM) | program transfer, tool data, control of the current program; DNC can start, pause and stop program sequences and show messages on the control | Strongest security story: SignAndEncrypt, X.509 user certificates |
| **Okuma** OSP-P300 and later | THINC API (free, standard, over 6,000 data points), MTConnect | THINC reads and writes code, parameters, variables and tool data | Windows on the control: apps run *on* the machine |
| **Mazak** (Mazatrol, Mitsubishi-based) | MTConnect, SmartBox gateway, Smooth Monitor AX, Sensor Box retrofit | no public write interface found | SmartBox is a security gateway ("secures the machine from the network") |
| **Brother** CNC-D00 | OPC UA, Ethernet, fieldbus slave options | "computer remote" (undocumented publicly) | MTConnect not listed |

**For Jason's Haas VF question:** Fusion's **live connection for inspection** (part of the paid
Manufacturing Extension) brings probe results back to Fusion in real time through a per-machine
**Machine Connector** application on a Windows PC; a Haas needs Machine Data Collection enabled.
The separate **Fusion Machine Connect** product adds program transfer, run/idle/stop monitoring
and history across Haas, MTConnect, Siemens OPC UA, Fanuc FOCAS and Heidenhain DCOM machines,
through one always-on Windows service PC, per-machine subscription. **Neither public page
describes controlling the machine**: the operator still presses Cycle Start, and correction
happens in the probing macros inside the control. That is the right split, and SolidCraft should
copy it (opinion).

**For "can we read spindle torque and axis load in real time":** yes, at three very different
speeds. Control polling (FOCAS, MTConnect, OPC UA, Haas Q commands) gives a value every 100–1,000
ms: enough to compare each CAM segment's average load with its prediction, set per-tool load
limits and spot a dull tool or a setup that is fighting the cut, **too slow to save a tool that
breaks now**. High-rate control traces (SINUMERIK Edge at 500 Hz) see individual events. Add-on
sensors on SolidCraft Edge (spindle current clamp, accelerometer at kHz) see each tooth. So the box
**predicts and records**, and the stop that saves the tool must come from **inside the control**:
for example the control's own tool-load monitor with a limit SolidCraft computes from the
physics and posts with the program (Haas has a per-tool load limit with an overload action;
unverified per software version). See [Signal rates](#signal-rates-what-real-time-really-means).

**The MCP proposal.** Makers should expose an **MCP server on the control (or its gateway)** as a
thin, self-describing layer over the information model they already have (OPC UA 40501
Machine Tools, MTConnect), with the safety enforced *underneath* MCP, in the control: read tools
open to an authorised client; "ask" tools (show a message, request feed hold, request stop after
this tool, stage a program, propose an offset) that the control applies only within limits or
after the operator confirms on the control's own screen; and **no tool at all** for cycle start,
jog, override increase, parameters or safety I/O. "The edge device may always ask the machine to
be safer; it may never make it move." The OPC Foundation's own MCP package for OPC UA (preview,
October 2026) exposes Write and Call by default, which shows why the limits have to live in the
machine, not in the agent. See [A proposal: MCP on the machine](#a-proposal-mcp-on-the-machine).

**SolidCraft at the machine.** Yes, on a small mini PC (opinion: an x86 N100-class box with a
touch screen and a mouse, or a Pi 5 for a lighter, view-only station): SolidCraft already builds
for Linux ARM64 and the browser and has an MCP server. A **machine station** mode would show the
current operation on the model, planned versus measured load, the next inspection step, and a
"report back" button that attaches a note, photo and measurements to the design feature and the
CAM operation, so the design and the program improve from the shop floor. It stays out of the
control loop. See [SolidCraft at the machine on a mini PC](#solidcraft-at-the-machine-on-a-mini-pc).

**Cameras for slip (stretch goal).** A fixed camera that checks fiducial marks on the part and
jaws at safe pauses (spindle stopped, coolant off, air blast) could catch movement of about
0.05–0.1 mm on a 200 mm field (opinion, from pixel arithmetic); continuous detection during the
cut is the "utopia" part. See [Cameras that see slip](#cameras-that-see-slip-a-stretch-goal).

## Fundamentals

### Three questions every interface answers

For each control, three questions decide what SolidCraft or SolidCraft Edge can do (opinion; the
same split as the cell automation file's three layers):

1. **What can I read, and how fast?** State, program and block, positions, feed and speed,
   overrides, loads, alarms, offsets, tool life, variables. Rate decides what is possible
   (see [Signal rates](#signal-rates-what-real-time-really-means)).
2. **What can I write, and what guards it?** Files, offsets, macro variables, parameters,
   messages. Guards are options and licences, passwords, modes (only in MDI, only when stopped),
   and whether the operator sees and confirms.
3. **Can I make it act?** Start, hold, stop, select a program. This is where machine safety
   (ISO 16090-1 for machining centres, unverified clause content) and liability live, and where
   makers are, rightly, the most careful.

### The two open standards underneath

- **MTConnect** (ANSI/MTC1.4; version 2.5 published as parts 1–5 on docs.mtconnect.org) is a
  **read-only** vocabulary and HTTP/XML (and JSON) service: an adapter talks to the control, an
  agent serves `/probe` (what the machine has), `/current` and `/sample` (values with timestamps
  and sequence numbers). Data items are SAMPLE (positions, speeds, loads, temperatures), EVENT
  (mode, program, block, tool) and CONDITION (normal, warning, fault). Part 5 **Interfaces**
  defines request/response handshakes between pieces of equipment (used in the cell automation
  file), which is the closest MTConnect comes to control. Monitoring vendors describe it as
  "a read-only protocol for shop-floor monitoring" (Industrial Monitor Direct).
- **OPC UA**, with the **OPC 40501-1 Machine Tools** companion specification (umati), defines an
  information model "for monitoring the machine tool and giving an overview of the jobs on it"
  for MES, SCADA, ERP and analytics (OPC Foundation reference, scope section). OPC UA itself can
  write variables and call methods; which ones a machine allows is the maker's choice. OPC UA has
  real security (signed and encrypted sessions, certificate users, roles).

Both are **maker-neutral models**. Neither is aimed at an AI agent. That gap is where the MCP
proposal sits.

### What each control maker offers

#### Haas (NGC 2016 and later; classic control)

Haas matters most for this file: Jason calls it the most common brand for entry- to mid-level
shops (personal experience; no market-share source was checked).

- **Machine Data Collection (MDC, "Q commands").** Standard since 2001 on the classic control
  over RS-232 (MachineMetrics support article, a comment quoting Haas). The control answers a
  short query with a comma-separated reply. Listed commands (CNC Arena forum, from Haas
  documentation): **Q100** serial number, **Q101** software version, **Q102** model, **Q104**
  mode (list, MDI, memory, jog), **Q200** tool changes, **Q201** tool in use, **Q300** power-on
  time, **Q301** motion time, **Q303** last cycle time, **Q304** previous cycle time,
  **Q402/Q403** M30 parts counters, **Q500** program, status and parts count in one reply, and
  **Q600** for macro variables (the inspection file reads macro variables this way; exact syntax
  and allowed ranges unverified).
  - Classic: Setting 143 "Machine Data Collection" ON; serial 38400 baud, even parity, 7 data
    bits, 1 stop bit, XON/XOFF (MachineMetrics). A serial-to-Ethernet converter (e.g. a Moxa in
    TCP server mode) puts it on the network.
  - **NGC: over Ethernet, and Setting 143 is the TCP port number** (Chatter sets 5051;
    MachineMetrics uses 4001). Chatter's guide also says to **disable HaasConnect, HaasDrop and
    Remote Net Share** for its collector (local Net Share can stay), which suggests these
    services compete for the same connection (inference). MachineMetrics reports a "Too many
    connections. Disconnecting." error whose only workaround is to reset Setting 143.
  - **What Q commands do not give:** spindle load, axis load and actual feed are not in the list
    above. Haas macros have system variables for spindle load (commonly cited as `#1098` on the
    classic control, with a different number on NGC; unverified), which Q600 could read about
    once a second. For a dependable load signal on a Haas, the retrofit sensor kit's spindle
    current clamp is the safer plan (inspection file).
  - **Writing:** some Haas documentation describes writing macro variables over MDC (an "E"
    command; unverified, not found in the sources read). If it exists, it is the cleanest
    handshake channel (a reserved block of `#500`-range variables), see
    [Writing to the control](#writing-to-the-control-the-safety-model).
- **MTConnect on NGC.** NGC "may have MTConnect capability", and software older than
  100.20.000.1200 gives "fewer data points" (MachineMetrics Haas page). Which items (spindle
  load, axis loads, block number) a given machine fills must be checked in its `/probe`
  response (machine library rule).
- **DPRNT** (print from a macro) to the serial port or the network: how probe results leave the
  machine for Fusion's live inspection and for SolidCraft Edge (inspection file). Autodesk forum
  titles such as "Surface Inspection DEPRINT Error on Haas NGC" show it is also where things
  break.
- **HaasConnect** (phone notifications of machine state and alarms), **HaasDrop** (send files
  from a phone), **Net Share** (network file share for programs) and **MyHaas** (Haas's
  customer portal). The Haas pages could not be fetched from this session; descriptions are from
  memory of Haas's public marketing and are (unverified). As far as the sources show, **none of
  them starts or stops the machine remotely.**
- **In-control tool load monitoring.** The NGC tool offset table has a per-tool load limit (% of
  spindle load), with a setting for what happens on overload (alarm, feed hold, beep or
  "autofeed", which reduces feed) (unverified per software version; Haas pages could not be
  fetched). This matters: it is a fast, deterministic reaction **inside the control** that
  SolidCraft can feed with numbers from its physics model (see
  [What users need](#posting-limits-into-the-control)).
- **Probing.** Haas machines use a Renishaw-based wireless probe (WIPS) with Haas's own probing
  macros and Visual Quick Code templates; results land in macro variables and offsets inside the
  control (inspection file).

#### Fusion and a Haas VF: what the public pages show

Fusion was not run. This is from Autodesk's help pages and product page.

- **Live connection for inspection** (Fusion Manufacturing Extension). "Import inspection
  results in real-time over a live connection" for surface inspection, geometric inspection
  (Probe WCS, Probe Geometry) and live part alignment. Needs the **Machine Connector application
  for your CNC machine**, which does not run on a Mac (install it on a Windows PC on the network
  instead). The machine connects to that PC by Ethernet, serial or wireless; the connector's IP
  and port must match the Machine Definition. For Haas, **Machine Data Collection must be
  enabled.** The flow: post the probing program, start Live Connection in Fusion, press **Cycle
  Start on the machine**, results appear on the model, save a report.
- **Fusion Machine Connect** (separate subscription, per machine): NC program transfer, run /
  idle / stopped monitoring from web devices, history export for trends and maintenance.
  Supports "HAAS (Classic and Next Generation)", MTConnect machines, Siemens via OPC UA, Fanuc
  via FOCAS, Heidenhain via DCOM. One always-on **Windows** computer runs the service for all
  machines. The page **does not describe machine control.**
- **What this means for "machine control in-process":** in Fusion today, in-process control is
  the **probing macros running inside the Haas** (update the work offset, a wear offset, or stop
  on out-of-tolerance), and Fusion receives results and draws them. Nothing on the public pages
  lets Fusion change the running program, hold the feed or start a cycle. Jason's experience
  (inspection file) is that the connector setup is clunky; forum titles ("Live Inspection Fusion
  360 and my Machine wont connect", "Fusion 360 (machine connect) supported on tormach?") agree.
- **What SolidCraft can do better (opinion):** no per-machine connector app (SolidCraft Edge or the
  machine station speaks MDC/DPRNT/MTConnect directly), no Windows-only service, no
  subscription; the same probing-in-the-control split, but with the results tied to the design's
  tolerances and the part record (inspection file), and with load data aligned to the CAM
  segments (machine health file).

#### Fanuc (0i-F/0i-F Plus, 30i/31i/32i-B and later)

- **FOCAS2** library (Fanuc America product page): Ethernet or HSSB (fibre); access to axes and
  spindle, tool life management, CNC files, **servo and spindle data**, diagnostics, alarms and
  operator messages. Pricing by enquiry; the page mentions Windows only. Secondary sources:
  TCP port 8193; an Ethernet/FOCAS option may need enabling; read functions include
  `cnc_rddynamic` (positions, feed, speed in one call), `cnc_rdservo`, `cnc_rdmacro`,
  `cnc_rdtofs`, `cnc_rdwkofs`, `cnc_rdalarm`, `cnc_rdspload` (spindle load); **writes** include
  `cnc_wrmacro`, `cnc_wrtofs`, `cnc_wrwkofs` and `cnc_wrparam` (password gated); a Linux shared
  library (`libfwlib32.so`) exists (Industrial Monitor Direct; Cybus; parameter and option numbers
  unverified). Typical polling: `cnc_rddynamic` every 250 ms, macros every 1 s.
- **MT-LINKi** (Fanuc's monitoring software): one to 2,000 machines; alarm history, macro
  variables and signal history, **servo and spindle motor currents and temperatures**, battery
  and fan status, feed overrides; OPC UA to robots and PLCs; MongoDB storage; "monitoring" only,
  no control described (Fanuc America page).
- Fanuc also sells an MTConnect adapter and an OPC UA server option on recent controls
  (unverified).
- **Licence caution for SolidCraft:** FOCAS is a proprietary library under Fanuc's terms; a
  Rust edge service would call it through FFI, which brings `unsafe` and a closed binary. That
  belongs in a separate, optional connector process, not in SolidCraft's own crates (opinion;
  AGENTS.md forbids `unsafe` outside an allowlist).

#### Siemens (SINUMERIK 828D, 840D sl, ONE)

- **Access MyMachine /OPC UA**: a licensed OPC UA server on SINUMERIK Operate for 840D sl and
  828D; read machine state, spindle data, tool and magazine management, events; **methods for
  file read/write/list/delete** and access rights; default port 4840 (Cybus SINUMERIK connector
  docs; Siemens configuration manual exists but was not read in full).
- **SINUMERIK Edge**: an industrial PC beside the NC that runs Siemens and third-party apps.
  The *Analyze MyWorkpiece /Capture* app exports NC data at **500 Hz** in JSON; a KIT research
  dataset recorded SINUMERIK 840D signals at 500 Hz on a DMG Mori CMX 600V and added a force
  platform and a spindle accelerometer at 10 kHz (KIT dataset record). Other Edge apps
  (Analyze MyWorkpiece /Monitor, Optimize MyMachining /Trochoidal) do process-parallel quality
  monitoring and optimisation (trade press). This is the clearest example of a maker shipping
  the "edge device" Jason describes, closed to Siemens's app ecosystem.

#### Heidenhain (TNC 620, TNC 640, TNC7; older iTNC 530)

- **OPC UA NC Server** (brochure): software options 56–61, **up to six OPC UA clients**; TNC7
  (81762x-16+), TNC 640 (34059x-10+), TNC 620 (81760x-08+); 90-day trial. Reads job status,
  interruptions, run times, machine messages (program end or abort, maintenance, faults,
  custom NC messages) and, with machine-builder extensions, temperatures, fill levels and
  counters. **Writes:** transfer of NC programs, **control of the current program**, tool data
  from a presetter, database synchronisation. Security: SignAndEncrypt, Basic256Sha256, user
  authentication with X.509 certificates; a Connection Assistant sets up certificates.
- **HEIDENHAIN DNC** (software option 18, part of the RemoTools SDK; a COM component on
  Windows): reads the active program, run times, interruptions, **feed and spindle override
  history**, errors and tool status; exchanges programs, datums, tools and images; **starts,
  pauses and stops program sequences**; shows messages and operator prompts on the control;
  uses machine-specific PLC functions (Heidenhain product page; Cybus guide: Windows service,
  `getState`, `getPlcData`, `getToolTableRow`, `transmitFile`).
- Heidenhain is the one maker in this list whose public pages show **remote start/stop** and
  **operator prompts on the control** as product features. The operator-prompt feature is
  exactly the "confirm on the control's own screen" step the MCP proposal needs.

#### Okuma (OSP-P300 and later)

- **THINC API**: free and standard on OSP-P300; custom PC applications "read and write code,
  set parameters and variables, including tool information", over 6,000 data points (Okuma
  OSP-P300 white paper, 2013). The control runs **Windows**, so applications install **on the
  control**. Okuma is a founding MTConnect supporter. Newer OSP controls add OPC UA (secondary
  source; port 4840 unverified).
- Okuma is the existing proof that "software at the machine, on the control" works; it is also a
  reminder that a general PC inside the control needs the same patching and hardening as any PC
  (high-security file).

#### Mazak (Mazatrol SmoothG/Ai/Ez, Mitsubishi-based)

- **MTConnect** natively; **SmartBox**, a security gateway with multiple MTConnect agents that
  exports data through one encrypted port ("secures the machine from the network", "secures the
  network from the machine"); **Smooth Monitor AX** dashboards (status, 24-hour history,
  tool utilisation, maintenance, energy); **Sensor Box** retrofit (stack light, coolant
  temperature and level, power) turns older equipment into MTConnect sources (Mazak monitoring
  page). No public write or control interface was found; secondary sources say the internal NC
  link is for Mazak engineering only (Industrial Monitor Direct).
- SmartBox is a useful pattern for the MCP proposal: the agent-facing server lives in a
  **gateway** the maker controls, not in the motion controller.

#### Brother (SPEEDIO, CNC-D00)

- Product page lists **OPC UA**, Ethernet, optional RS-232C, an optional built-in PLC, fieldbus
  slave options (CC-Link, PROFIBUS DP, DeviceNet, PROFINET, EtherNet/IP), "computer remote" and
  "auto notification" (not described further). MTConnect is not listed.

#### Others, briefly

- **DMG MORI** (CELOS, on Siemens, Heidenhain or Fanuc controls): umati/OPC UA and MTConnect
  through its connectivity products; in-machine camera systems for chip monitoring are marketed
  (page could not be fetched; unverified).
- **Hurco** (WinMax): MTConnect adapter (unverified).
- **Tormach** (PathPilot, built on LinuxCNC): LinuxCNC is GPL-2.0, so its code must not be read
  for SolidCraft work (AGENTS.md); talking to a running machine over a documented network
  interface is fine, but which interfaces PathPilot exposes was not checked. A Tormach forum
  thread asks whether Fusion's machine connect works with it at all.
- **Retrofit monitoring vendors** (MachineMetrics, Chatter, FactoryWiz, Cybus) all build per-
  control connectors for the same handful of interfaces above, which shows the long tail is
  support work, not invention (inspection file's conclusion).

### Signal rates: what "real time" really means

Jason's question is whether spindle torque and axis load can be read "in realtime" to catch tool
failure, setup problems and bad cutting. "Real time" means three different things here (fact for
the rates where cited; the consequences are opinion):

| Tier | Source | Rate | Sees | Can it save a breaking tool? |
|---|---|---|---|---|
| **A. Control polling** | Haas Q commands, FOCAS, MTConnect, OPC UA | 1–10 per second (100–1,000 ms; FOCAS `cnc_rddynamic` at 250 ms typical; MTConnect adapters 100–1,000 ms) | the average load of a segment, tool by tool; feed and speed actually used; overrides; alarms | **No.** At 10,000 rev/min and 3 teeth, 250 ms is 125 tooth passes. By the time a poll sees the spike, the tool is gone. |
| **B. High-rate control trace** | SINUMERIK Edge capture (500 Hz); maker servo-tuning traces (offline tools) | 100–1,000 per second | single events: an entry shock, a step in load when a tooth chips, a stall | Sometimes, if the reaction is inside the control. Over a network, latency is not guaranteed. |
| **C. Add-on sensor on the box** | spindle current clamp, accelerometer, microphone (inspection file's retrofit kit) | 1–50 kHz | each tooth; chatter frequency; the spectrum | Detects fastest, but the box can only **ask** the control to stop (a wired input), and the control must act on it. |
| **D. Inside the control** | the control's own tool-load monitor, adaptive feed option, or a maker partner (e.g. Caron TMAC, Siemens Edge apps) | servo cycle (ms) | the same load, with a guaranteed reaction | **Yes.** This is the only tier that reliably stops in time. |

**What each tier is good for (opinion):**

- **Tier A is enough for most of what Jason wants:** compare each CAM segment's measured average
  load with its prediction (the machine health file's residual), find the tool that pulls 30 %
  more than it should (dull, wrong grade, wrong material lot), the setup that fights the cut
  (vibration-free but loads high: wrong stick-out, worn holder), the operator override that
  hides a bad program, and the gradual wear that predicts the next failure. It is also enough to
  **set the limits** for Tier D.
- **Tier C adds chatter and tooth-level wear** and is the realistic way to get a load signal on
  a Haas, where the Q commands do not include load.
- **Tier D must do the stopping.** SolidCraft's job is to give the control good limits: a load
  ceiling per tool and per operation computed from the predicted cutting force with a margin,
  posted with the program (see [Posting limits into the control](#posting-limits-into-the-control)).
  SolidCraft Edge then checks, after the fact, whether the limits were right and suggests better ones
  for the next run.

Timestamps and the G-code block matter as much as rate: a value that cannot be placed on the
toolpath is half as useful. The inspection file's alignment section and the machine library's
"best alignment the control allows" field cover this; this file adds only the rate per
interface above.

### Writing to the control: the safety model

Everything above can be read safely. Writing is where a mistake breaks tools, parts or people.
This model applies to SolidCraft Edge, the machine station, an agent, and any future maker MCP server
(opinion, strongly held; it extends the cell automation file's write tiers and the inspection
file's "corrections stay in the control"):

**The rule:** *outside software may always ask the machine to be safer; it may never make the
machine move.* Safer means: show a message, hold the feed, stop at the next safe point, refuse to
start the next cycle. Moving means: cycle start, jog, override increase, program change mid-run,
parameter changes.

| Tier | Examples | Who confirms | Default |
|---|---|---|---|
| **Record** | read state, loads, alarms, offsets, results | nobody | on |
| **Advise** | message or prompt on the control's screen; stack-light colour; a note in the part record | nobody (it changes nothing) | on |
| **Stage** | upload a program to a staging folder (not the active one); propose an offset change; propose a load limit for a tool | the operator, on the control's own screen, before it takes effect | on, with confirmation |
| **Request (safer)** | feed hold; stop after this tool; do not start the next cycle; optional-stop flag the program checks at a safe point | nobody (it is always safe to stop), but the request and its reason are logged and shown | off by default, opt-in per machine |
| **Commit** | write a wear offset within a small limit (for example ≤ 0.02 mm per change and ≤ 0.05 mm in total per shift) without a prompt | configured by the shop, limits enforced by the control | **off** by default |
| **Never** | cycle start, jog, override increase, parameter writes, safety I/O, door interlocks | not offered | not offered |

Rules that go with the tiers (opinion):

1. **The control enforces, not the client.** Limits (which variables, which value range, how
   much an offset may change, how often) live in the control or the maker's gateway. A client
   that is wrong, compromised or confused cannot exceed them. MCP tool annotations
   (`readOnlyHint`, `destructiveHint`) are **hints**; the MCP project's own blog post "Tool
   Annotations as Risk Vocabulary: What Hints Can and Can't Do" (March 2026) is about exactly this
   limit.
2. **Mode interlocks.** Staging and commits only when the control is in a state where they cannot
   surprise anyone: offsets between cycles, never mid-cut; programs only into a staging area.
3. **The operator sees every write**, on the control's screen and in the part record, with who
   or what asked and why ("T7 wear offset −0.008 mm proposed by inspection, bore 4 measured
   +0.008 mm").
4. **Handshakes through reserved variables.** Where a control only offers macro-variable writes
   (Haas, Fanuc), reserve a small block (for example 20 variables in the `#500` range) as the
   mailbox, and let a macro in the program decide at a safe point what to do with a request
   (stop, skip, continue). This keeps the decision in the program the operator proved out.
5. **Wired stop requests, if any, are designed by the integrator** (a dry contact into a control
   input that the program or PLC handles), as in the inspection file.
6. **Network loss is safe.** If the box disappears, the machine keeps doing what its program
   says; nothing in the program waits forever on the box (time-outs default to "continue" for
   advice and "stop" for a pending safety request; opinion).
7. **Everything is logged** with the same audit chain as the high-security file.

### Cameras that see slip: a stretch goal

The camera hardware (sealed housing, air blast, Pi camera, OpenCV) is in the inspection file's
"Cameras instead of, or beside, the probe"; slip and lift physics and load-signature detection
are in the workholding file. What a camera could add (opinion):

- **Check frames at safe pauses.** At tool changes or a programmed pause (spindle stopped,
  coolant off, a short air blast), the camera takes a frame; the box measures the position of
  fiducial marks on the part (or its edges) relative to marks on the jaws or fixture. Movement
  beyond a threshold means slip.
- **Resolution arithmetic:** a 1920-pixel-wide image across a 200 mm field is about 0.10 mm per
  pixel; fiducial markers are commonly located to a fraction of a pixel, so 0.03–0.05 mm
  movement may be detectable (opinion, not tested here; lighting, focus and thermal drift of the
  mount set the real limit). That is enough to catch a part that has rotated or walked in the
  jaws, not enough to replace probing.
- **Program side:** the post inserts the check at chosen points; the program waits a short time
  for a "clear" in a reserved variable (safety model rule 4) and stops if it gets "moved", or
  continues on time-out with a warning (shop's choice).
- **Continuous detection during the cut** (the "utopia" part): chips and coolant hide the part.
  Research exists on event cameras for slip sensing in automation (Khalifa University project
  CIRA-2018-55) and patents on camera-based slip and position-loss detection, but no shop-ready
  product was found. During the cut, load and vibration signatures (workholding file) remain the
  practical detector.

## Accuracy

What the numbers in this file can and cannot support (opinion unless cited):

- **Polled loads are averages.** A Tier A load is the control's own filtered value, often in
  "% of rated" with an unstated rating (S1 or S6, which gear) and unknown filter time. Compare it
  with the prediction only after calibrating its meaning per machine (machine library's signal
  entry), and compare like with like: the average over a segment, not a peak.
- **Spindle load ≠ cutting torque.** It includes friction, acceleration of the spindle, and
  motor losses; at low speed these are a large part. Subtract an air-cut baseline from the same
  speed (machine health file's fingerprint cycle).
- **Axis load ≠ cutting force.** It includes friction, gravity on Z, acceleration and ballscrew
  efficiency; it shows large changes (a crash, heavy engagement) well and small ones poorly.
- **Timestamps.** Each interface stamps differently (agent time, poll time, control time); clock
  offsets of tens of milliseconds are normal before correction (inspection file).
- **Interface coverage is per software version and option.** Every claim of "control X exposes
  Y" in this file needs checking on the real machine (`/probe` for MTConnect, browse for OPC UA,
  function return codes for FOCAS). The machine library stores the verified result per machine
  model and software version.
- **Camera slip numbers** are arithmetic, not measurements.

## Programs

Existing programs and products in this space (descriptions from their public pages; none was run):

| Program | What it does | Interfaces | Control? |
|---|---|---|---|
| Fusion live inspection + Machine Connector (Autodesk) | probe results into Fusion live; reports | per-machine connector app, Haas MDC | no |
| Fusion Machine Connect (Autodesk) | program transfer, monitoring, history | Haas, MTConnect, OPC UA (Siemens), FOCAS, DCOM (Heidenhain) | no (as described) |
| MT-LINKi (Fanuc) | monitoring of up to 2,000 machines, currents and temperatures | Fanuc Ethernet, OPC UA to robots/PLCs | no |
| Smooth Monitor AX, SmartBox (Mazak) | dashboards, secure gateway | MTConnect | no |
| SINUMERIK Edge apps (Siemens) | 500 Hz capture, quality monitoring, optimisation | SINUMERIK internal | some apps optimise programs (as described) |
| HEIDENHAIN DNC / RemoTools SDK | custom Windows apps for monitoring and remote operation | COM | yes: start, pause, stop, messages |
| THINC API (Okuma) | apps on the control | native | yes: read and write data |
| MachineMetrics, Chatter, FactoryWiz, Cybus | monitoring platforms with per-control connectors | MDC, FOCAS, MTConnect, OPC UA | mostly no |
| MTConnect C++ Agent (MTConnect Institute; Apache-2.0, unverified) | reference agent | adapters | no (read-only) |
| TrakHound MTConnect Agent for Haas (open source, RS-232; licence not stated) | Haas MDC to MTConnect | serial Q commands | no |
| OPC UA MCP server (OPC Foundation, `OPCFoundation.NetStandard.Opc.Ua.Mcp`, 2.0.1, October 2026) | exposes OPC UA Part 4 services as MCP tools for AI assistants; Robotics and Vision preview packages | OPC UA | **the default `full` profile includes Write and Call**; no read-only mode described |
| Community OPC UA MCP servers (glama.ai and others) | the same idea, hobby/early | OPC UA | varies |

The OPC Foundation package is the most important finding for the MCP proposal: the standards body
itself is building the agent bridge (fact), and its default exposes writes (fact), so the case
for **limits enforced by the machine** is not hypothetical (opinion).

## Complaints and requests

From public forum titles, vendor setup guides and Jason (thread titles only where pages could
not be read):

- **Connector friction:** "Live Inspection Fusion 360 and my Machine wont connect"; "Surface
  Inspection DEPRINT Error on Haas NGC"; "Fusion 360 (machine connect) supported on tormach?"
  (Autodesk and Tormach forums). Jason (inspection file): the Machine Connector "was always
  clunky".
- **Windows-only pieces:** Fusion's Machine Connector does not run on a Mac; Machine Connect's
  service needs an always-on Windows PC; Heidenhain DNC is a Windows COM component; FOCAS is
  documented for Windows.
- **Paywalls and options:** live inspection needs the Manufacturing Extension; Machine Connect
  is per machine; FOCAS, Siemens OPC UA and Heidenhain OPC UA/DNC are paid options.
- **Services that fight:** collectors ask Haas users to turn off HaasConnect, HaasDrop and Remote
  Net Share; "Too many connections" on Haas MDC (Chatter, MachineMetrics).
- **Data that depends on software version:** fewer MTConnect data points on older Haas NGC
  software (MachineMetrics).
- **No load in Haas Q commands** (from the command list), so shops add sensors or read macro
  variables.
- **Jason:** threading and tapping are where he breaks the most tools "because no other software
  does the physics" (threading thread); a load limit per tap posted into the control is one
  concrete answer (see below; the threading module itself is a separate topic).

## What users need

### SolidCraft Edge: Jason's five functions, mapped to the interfaces

**Jason's definition (2026-10-26, summarised; the in-process inspection file owns the full
definition):** SolidCraft Edge is an edge computer on a **Raspberry Pi 5, programmed in Rust**,
for connecting CNC machines and physical sensors. It has five functions: (1) **data
acquisition** from machines (spindle speed, tool position, temperature) and sensors over serial,
**Modbus** or Ethernet machine interfaces; (2) **machine communication**, sending commands and
status requests for coordinated operation; (3) **data transmission to the CAM station** over
REST, MQTT or WebSocket; (4) a **visual dashboard** on a connected monitor (machine state,
alerts, production metrics); (5) **report printing** (production logs, maintenance summaries,
quality checks). The operator uses a **keyboard and mouse**. It is built for reliability, low
latency and safe continuous operation.

What each function needs from this file (opinion where not cited above):

| Function | Interfaces it uses (from [What each control maker offers](#what-each-control-maker-offers)) | Limits from this file |
|---|---|---|
| 1. Data acquisition | Haas MDC (serial or Ethernet) and DPRNT; MTConnect; OPC UA (40501); FOCAS (separate process); Heidenhain OPC UA/DNC; **Modbus RTU/TCP for add-on sensors, coolant monitors and small PLCs** (Modbus is not a CNC interface on the controls surveyed; it fits the retrofit sensor kit) | polled control data at 100–1,000 ms, add-on sensors at kHz; see [Signal rates](#signal-rates-what-real-time-really-means) |
| 2. Machine communication | status requests: all of the above. **Commands: supervision level only**, per [Writing to the control](#writing-to-the-control-the-safety-model): Advise (messages), Stage (programs, offset and load-limit proposals the operator confirms on the control), Request (feed hold, stop after this tool, block next start); Commit off by default | **never** cycle start, jog, override increase, parameters or safety I/O. "Coordinated, real-time operation" means coordination at safe points through handshakes (reserved variables, MTConnect Interfaces), not motion control; the cell automation file's three layers keep the Edge out of the safety and real-time handshake layers |
| 3. Transmission to the CAM station | REST for records and reports, MQTT for live values (publish per machine topic), WebSocket for the live dashboard in SolidCraft; all outbound from the Edge, no inbound ports on the plant side (high-security file) | records carry the alignment keys (program, block, CAM segment, tool, timestamp) so the CAM station can line measured up with predicted |
| 4. Visual dashboard | the [machine station](#solidcraft-at-the-machine-on-a-mini-pc) screens, drawn by the Edge itself on its monitor | big targets and contrast (aging-eyes file); keyboard and mouse as Jason specifies, touch optional |
| 5. Report printing | none from the control; reports come from the part record (inspection file) | report levels as in the inspection file |

**"Low latency" (opinion):** the Edge should keep its own loop fast (collect, timestamp, show
and forward within about a second), but nothing that must happen within milliseconds (stopping a
breaking tool) is the Edge's job. That stays in the control, fed with limits SolidCraft posts
(see [Posting limits into the control](#posting-limits-into-the-control)). Written this way, the
Edge stays safe when its network, its software or its power fails: the machine carries on under
its own program and its own monitors.

**MCP on the Edge (opinion):** the Edge is the natural place for SolidCraft's side of the MCP
proposal. It is an **MCP client** of a maker's machine MCP server where one exists, and it can
itself be an **MCP server** to the CAM station and agents, exposing the same tiers (read,
advise, stage, request; no motion tools) over whatever interface the machine actually has.
That way an agent sees one safe tool set whether the machine is a Haas on MDC or a Heidenhain on
OPC UA.

### A connector layer in SolidCraft (opinion)

- **One connector per interface, not per machine:** Haas MDC (serial and Ethernet), DPRNT
  capture, MTConnect client, OPC UA client (40501 where present), FOCAS (separate optional
  process, closed library), Heidenhain DNC/OPC UA, Okuma THINC (on the control's Windows). Each
  reports what it found (`/probe`, browse) into the machine's shop record in the machine
  library.
- **Runs in SolidCraft Edge (or a machine-station PC)**, not in the desktop app; the desktop app reads
  the recorded data through the part record and sync (inspection file).
- **Rust, separate crate or process**, so SolidCraft's own crates keep the no-`unsafe`,
  wasm-buildable rules (AGENTS.md); closed vendor libraries only behind a process boundary.
- **Read-only by default.** Write tiers per the safety model, each switched on per machine and
  logged.

### Posting limits into the control

The most valuable "control" SolidCraft can offer is not remote control at all, but **better
numbers inside the control** (opinion):

- **Per-tool load limits.** From the cutting physics prediction for each tool's heaviest
  segment, plus a margin from the confidence slider, the post writes the control's tool-load
  limit (Haas tool offset table load column via a `G10`-style setting or an operator-checked
  setup sheet, unverified syntax; Fanuc and Siemens through their monitoring options). The
  control then stops or holds in milliseconds when a tool is overloaded, which no network box can.
- **Taps and thread mills especially:** a tap's torque rises sharply with wear and chip packing;
  a limit just above the predicted torque turns a broken tap into an alarm (links to the
  threading topic, in progress).
- **After the run,** the box compares measured loads with the limits and proposes new limits for
  the next run (Stage tier: the programmer accepts).

### A proposal: MCP on the machine

**What Jason proposes:** that all CNC makers add an MCP interface so an edge device can monitor
and control machines.

**Why MCP and not "just OPC UA" (opinion).** OPC UA and MTConnect already describe machines well.
What they lack is the thing MCP was built for: **self-describing tools with JSON Schema inputs
and outputs that an AI agent can discover and call**, with structured results, a way for the
server to ask the human for input (elicitation, MCP 2025-06-18), and OAuth-based authorisation
(Cisco developer blog summary of 2025-06-18). SolidCraft already speaks MCP (`solidcraft-cli
mcp`, `docs/mcp.md`): a SolidCraft agent that plans a job could talk to the machine with the same
protocol it uses to talk to the model. So the proposal is **an MCP facade over the maker's
existing OPC UA / MTConnect model**, not a new data model.

**Where it runs:** in the control's HMI PC (Okuma, Siemens, Heidenhain already run apps there)
or, better, in a maker gateway like Mazak's SmartBox or Siemens's Edge, which keeps the agent
traffic away from the motion controller (opinion).

**A minimal tool set (sketch, opinion):**

| MCP tool or resource | Tier | Notes |
|---|---|---|
| `machine.identity`, `machine.capabilities` (resource) | Record | model, control, software version, options, which signals at what rate (feeds the machine library) |
| `machine.state` | Record | mode, program, block, tool, positions, feed, speed, overrides, alarms |
| `signals.subscribe` | Record | loads, temperatures, with timestamps and block numbers; rate stated per signal |
| `offsets.read`, `tools.read`, `results.read` | Record | work and tool offsets, tool table and life, probe results |
| `operator.message` | Advise | text and an optional picture on the control's screen; never blocks the machine |
| `program.stage` | Stage | upload to a staging folder; activation only by the operator |
| `offset.propose`, `toolload.propose` | Stage | shown on the control with the reason; the operator accepts or rejects; optional Commit within control-enforced limits |
| `cycle.request_hold`, `cycle.request_stop_after_tool`, `cycle.block_next_start` | Request (safer) | always allowed to make things safer; logged with the reason |
| *(none)* | Never | no cycle start, jog, override increase, parameter or safety I/O tools exist on the server at all |

**Safety model:** the tiers and rules from [Writing to the control](#writing-to-the-control-the-safety-model)
apply unchanged, with these MCP-specific points (opinion):

- **Absent, not hidden.** Dangerous tools are not merely annotated `destructiveHint: true`; they
  are not on the server. Annotations are hints a client may ignore.
- **Confirmation on the machine, not in the chat.** MCP elicitation asks the *client's* user; a
  staged change on a machine must be confirmed by the operator *at the machine*, on the control's
  screen (Heidenhain's DNC already shows operator prompts). The MCP result says "pending operator
  confirmation" until then.
- **Scopes per tier** in the OAuth token (`machine:read`, `machine:advise`, `machine:stage`,
  `machine:request`), issued by the shop's own authority, short-lived, bound to the machine
  (resource indicators); mutual TLS on the OT network; no inbound internet path (high-security
  file).
- **Rate and value limits** in the server (for example one offset proposal per tool per cycle,
  offsets within ±0.05 mm).
- **Audit:** every call, result and operator decision is recorded on the machine and sent to the
  part record.
- **Liability is clearer this way:** the machine maker keeps control of motion and safety, the
  shop decides which tiers are on, and the agent only advises and requests. This is likely the
  only form makers would accept (opinion).

**How to suggest it (opinion; for maintainers and Jason):** a short public note from SolidCraft
(and other open CAM projects) describing the tool set and safety tiers above, offered to the
MTConnect Institute and the OPC Foundation / umati working groups, would carry more weight than
asking makers one at a time. The OPC Foundation already ships an OPC UA MCP package, so the
natural place for the safety profile is a companion to it: "a machine-tool MCP profile with no
motion tools".

### SolidCraft at the machine on a mini PC

**Jason's idea:** run SolidCraft at the machine on a small mini PC with a simple screen and mouse,
to integrate the process into the machine and feed back to design and CAM.

**SolidCraft Edge already has a screen.** Jason's definition gives the Edge its own monitor, keyboard and mouse and a live dashboard, so the simplest machine station is the Edge's own dashboard. Running the full SolidCraft app (the model, the toolpath, report-back on the design) at the machine is the richer option below, either on the Edge's Pi 5 or on a separate mini PC beside it.

**Can it run there? (fact for SolidCraft, opinion for hardware).** SolidCraft ships Linux ARM64
packages (AppImage, .deb, .rpm), Windows ARM64 and x86, and a browser build (`docs/downloads.md`);
it renders with the GPU (wgpu) and has a headless CLI and an MCP server. So:

| Option | For | Notes (opinion; prices unverified, they move) |
|---|---|---|
| **x86 mini PC** (Intel N100/N150 class, 8–16 GB RAM, fanless) | full machine station: view the model and toolpath, live charts, inspection entry | about USD 150–300; mature GPU drivers; fanless and sealed matters near coolant mist |
| **Raspberry Pi 5** (8 GB) | lighter station, or SolidCraft Edge itself with a screen | GPU driver support for wgpu on the Pi's Vulkan driver should be tested before promising it (unverified); fine for a browser view of a station page served by the box |
| **The control's own PC** (Okuma, some Siemens and Heidenhain HMIs) | no extra box | SolidCraft for Windows on the HMI; the maker's rules on installing software apply |
| **Operator's tablet** | browser build or a page from the box | no hardware at the machine at all |

**What the station shows (opinion; one screen, big targets per the aging-eyes file):**

1. **This operation on the model:** the setup, the stock, the toolpath of the running segment
   highlighted from the block number (alignment from the inspection file).
2. **Planned versus measured:** spindle and axis load for the current tool against the
   prediction and the limit, in plain words ("T7 pulling 18 % above plan for 3 parts").
3. **Next check:** the inspection step due now, with limits and a picture (inspection file's UX
   goals), entered with a gauge, a tap, or read from the probe.
4. **Report back:** one button attaches a note, a photo from the camera, the measured values and
   the machine data of the last minute to **the design feature and the CAM operation** it belongs
   to. In the office, the programmer and designer see it on the model; an accepted change becomes
   a new revision (compliance file workflow).
5. **No editing of the design at the machine by default** (a view and comment station), so the
   released revision cannot change by accident on the shop floor; an engineer can unlock it.

**Not in the control loop.** The station advises and records; writes follow the safety model.
It must keep working, and the machine must keep running, when the station is off or the network
is down (offline-first; sync later).

### What contributors could build first (opinion, smallest useful steps)

1. A **Haas MDC reader** (Ethernet, Setting 143 port) and a **DPRNT capture** that write
   timestamped records: the cheapest real data from the most common machine.
2. An **MTConnect client** that reads `/probe` and fills the machine library's signal entry.
3. A **"post the load limits"** option in the CAM post (once CAM exists, #1153 / PR #4158): one
   load ceiling per tool from the physics, printed on the setup sheet even where the control
   cannot read it from the program.
4. A **read-only machine station page** served by SolidCraft Edge and opened in SolidCraft's browser
   build.
5. A **public draft of the machine MCP profile** (tool set and tiers above) for discussion.

## Glossary

- **Adapter / agent (MTConnect):** the adapter talks to one control; the agent serves the
  standard HTTP interface to clients.
- **DNC (Heidenhain):** Heidenhain's remote-access interface (option 18), not the older meaning
  "drip-feed numerical control".
- **DPRNT:** a macro statement that prints text to a port; used to send probe results.
- **SolidCraft Edge:** Jason's name for the Raspberry Pi computer at the machine (formerly "the Pi box"): it sits on the edge of CAM and brings the machine and automation back into CAM for closed-loop process improvement.
- **Edge device:** any small computer at the machine that collects data; SolidCraft Edge and SINUMERIK Edge are examples.
- **Elicitation (MCP):** a server asking the client's user for input during a tool call.
- **FOCAS:** Fanuc's library for reading and writing CNC data over Ethernet or HSSB.
- **HSSB:** High-Speed Serial Bus, Fanuc's fibre link to a PC.
- **Modbus (RTU/TCP):** a simple industrial register protocol, common for sensors, coolant monitors and small PLCs; not a CNC control interface on the controls surveyed here.
- **MCP (Model Context Protocol):** an open protocol for AI applications to discover and call
  tools and read resources on servers, with JSON Schema inputs and outputs.
- **MDC (Haas Machine Data Collection):** the Q-command query interface (Setting 143).
- **OPC UA / OPC 40501 / umati:** the industrial communication standard, its machine-tool
  companion specification, and the VDMA initiative that promotes it.
- **Q command:** a Haas MDC query such as Q500.
- **Staging:** placing a change where it does nothing until the operator accepts it.
- **Tool-load monitor:** a control function that compares spindle load against a per-tool limit
  and reacts (alarm, feed hold, reduced feed).

## Open questions

1. **Haas details** that need a Haas manual or a real NGC: the exact Q600 syntax and variable
   ranges, whether NGC supports writing macro variables over MDC, the NGC spindle and axis load
   variable numbers, which MTConnect items NGC fills on current software, and the exact tool-load
   monitor settings. Jason's machines are the best test bench.
2. **Can a post set Haas tool-load limits from the program**, or only through the offset page or
   a file? (Determines whether "post the load limits" is automatic or a setup-sheet step.)
3. **Is the OPC Foundation's MCP package aiming at a safety profile**, and would umati accept a
   "no motion tools" machine-tool profile? Worth asking on their channels.
4. **Should SolidCraft ship connectors for closed libraries** (FOCAS, DNC) at all, or only the
   open protocols (MTConnect, OPC UA, Haas MDC) and leave closed ones to third parties? A licence
   and maintenance question for maintainers.
5. **wgpu on the Raspberry Pi 5:** does the SolidCraft ARM64 build render acceptably on the Pi's
   Vulkan driver? A quick test would settle the station hardware choice.
6. **Camera slip check:** what movement can a sealed camera really detect in a coolant-wet
   enclosure? Needs a bench test.
7. **Name check for "SolidCraft Edge":** Siemens sells SINUMERIK Edge and Industrial Edge, and other vendors use "Edge" too. "SolidCraft Edge" is distinct as a whole, but the maintainers should check it against their trademark plans before it appears in the product.
8. **Who owns the machine station's "report back" data model:** this file, the inspection file
   (part record) or the compliance file (revisions)? Suggest: record in the inspection file's
   model, revision flow in the compliance file.

## Sources

### Standards and specifications (public pages)

- OPC Foundation, OPC 40501-1 *Machine Tools: Monitoring and Job Management*, scope section:
  https://reference.opcfoundation.org/specs/OPC-40501-1/1.1
- MTConnect Institute, MTConnect Standard 2.5, Parts 1–5: https://docs.mtconnect.org/MBSD_MTConnect_Part_1_2-5-0.pdf
  (and Part 5 Interfaces: https://docs.mtconnect.org/MBSD_MTConnect_Part_5_2-5-0.pdf)
- Model Context Protocol blog, "Tool Annotations as Risk Vocabulary: What Hints Can and Can't Do"
  (16 March 2026): https://blog.modelcontextprotocol.io/tags/security/
- Cisco developer blog, "What's New in MCP 2025-06-18: Human-in-the-Loop, OAuth, Structured
  Content": https://blogs.cisco.com/developer/whats-new-in-mcp-elicitation-structured-content-and-oauth-enhancements
- OPC Foundation, `OPCFoundation.NetStandard.Opc.Ua.Mcp` 2.0.1 (NuGet, October 2026):
  https://www.nuget.org/packages/OPCFoundation.NetStandard.Opc.Ua.Mcp ; Robotics preview:
  https://www.nuget.org/packages/OPCFoundation.NetStandard.Opc.Ua.Mcp.Robotics/

### Control makers

- Fanuc America, FOCAS2 Library: https://www.fanucamerica.com/products/software/cnc/focas2-library
- Fanuc America, MT-LINKi: https://www.fanucamerica.com/products/software/cnc/mt-link-i
- Heidenhain, OPC UA NC Server brochure (ID 1355797): https://www.heidenhain.us/wp-content/uploads/2023/02/BR_OPC_UA_NC_Server_ID1355797_en.pdf
- Heidenhain, HEIDENHAIN DNC: https://www.heidenhain.com/products/software/heidenhain-dnc
- Okuma, OSP-P300 white paper (2013): https://www.okuma.com/files/documents/p300_whitepaper_final_2013_11_22.pdf
- Mazak, Monitoring & Analysis: https://www.mazak.com/us-en/technology/monitoring-analysis/
- Brother, CNC-D00: https://machinetool.global.brother/en-ap/cnc-d00/index.aspx
- Haas, Machine Data Collection – NGC (could not be fetched from the session):
  https://haascnc.com/ar/service/troubleshooting-and-how-to/how-to/machine-data-collection---ngc.html

### Autodesk Fusion (help and product pages; Fusion not run)

- Live connection for inspection: https://help.autodesk.com/cloudhelp/ENU/Fusion-CAM/files/MFG-LIVE-CONNECTION-INSPECTION-OVERVIEW.htm
- Import inspection results in real time: https://help.autodesk.com/cloudhelp/ENU/Fusion-CAM/files/MFG-HOW-TO-LIVE-CONNECTION-INSPECTION.htm
- Fusion Machine Connect: https://www.autodesk.com/products/fusion-machine-connect/overview

### Connectivity guides and secondary sources

- Chatter, Haas NGC integration: https://support.chatter.dev/docs/integration/haas-ngc
- MachineMetrics, Haas serial MDC: https://support.machinemetrics.com/hc/en-us/articles/229050328-How-to-Collect-data-using-HAAS-Serial-Machine-Data-Collection
- MachineMetrics, Haas connectivity: https://www.machinemetrics.com/connectivity/machines-controls/haas
- CNC Arena forum, Haas machine data collection (Q command list): https://en.cncarena.com/forum/thread/155953-machine-data-collection/
- TrakHound / namizich, MTConnect Agent for Haas: https://github.com/namizich/MTConnect-Agent-For-Haas
- Industrial Monitor Direct, "CNC Data Collection Deep Dive: FOCAS, MTConnect, and NC Protocols":
  https://industrialmonitordirect.com/blogs/knowledgebase/cnc-data-collection-deep-dive-focas-mtconnect-and-nc-protocols
- Cybus, SINUMERIK connector: https://docs.cybus.io/connectors/shop-floor-connectors/sinumerik
- Cybus, Heidenhain DNC: https://docs.cybus.io/guides/machine-connectivity/connecting-a-machine-via-heidenhain-dnc-interface
- KIT, multimodal CNC milling dataset (SINUMERIK Edge 500 Hz, sensors 10 kHz): https://publikationen.bibliothek.kit.edu/1000185172
- Modern Machine Shop, Siemens Analyze MyWorkpiece /Monitor: https://mmsonline.com/products/siemens-analyzemyworkpiece-monitor-enables-process-parallel-quality-control
- Khalifa University, CIRA-2018-55 slip sensing with dynamic vision sensors: https://khazna.ku.ac.ae/en/projects/cira-2018-55-intelligent-slip-sensing-system-using-dynamic-active/

### Community sources (thread titles)

- Autodesk forum, "Surface Inspection DEPRINT Error on Haas NGC": https://forums.autodesk.com/t5/fusion-support-forum/surface-inspection-deprint-error-on-haas-ngc/td-p/9575587
- Autodesk forum, "Live Inspection Fusion 360 and my Machine wont connect": https://forums.autodesk.com/t5/fusion-manufacture-forum/live-inspection-fusion-360-and-my-machine-wont-connect/m-p/12336384
- Tormach forum, "Fusion 360 (machine connect) supported on tormach?": https://forums.tormach.com/t/fusion-360-machine-connect-supported-on-tormach/872

### SolidCraft

- `docs/mcp.md`, `docs/control-protocol.md`, `docs/downloads.md`, `AGENTS.md` at commit `2016ba7`.
