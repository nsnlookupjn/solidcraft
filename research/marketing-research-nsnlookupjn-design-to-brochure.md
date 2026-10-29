# Marketing research: from design to brochure, inside SolidCraft

A **marketing channel built into SolidCraft**: one path from a finished design to finished
marketing material. Photoreal stills, turntables, exploded views, animated controls, a
**web explorer** people can spin and click on a product page, video, spec sheets and printable
brochures and datasheets, all made from templates with the company's own branding, and all
**kept up to date when the design changes**.

Two rules shape everything below:

1. **Marketing output never carries enough detail to reverse engineer the product.** This holds
   for every product, not only export-controlled ones. Every output passes an automatic
   reverse-engineering risk check before it can be released (section 4).
2. **Controlled designs go through the release gate that already exists in the research.** The
   ITAR "marketing derivative" workflow is written up in
   [section 14 of the high-security research](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-2fa4g1/research/security-research-nsnlookupjn-high-security.md#14-marketing-an-itar-product-renders-animations-exploded-views-and-web-viewers),
   and digital fingerprints and leak tracing in the
   [IP protection research](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-2zu80u/research/ip-research-nsnlookupjn-ip-protection.md).
   This file links to them and does not repeat them.

Licence: MIT OR Apache-2.0, the same as SolidCraft (see the Licence section of CONTRIBUTING.md).

## About this research

- **Author:** Jason ([@nsnlookupjn](https://github.com/nsnlookupjn)). Idea and direction by Jason;
  source research and writing done with an AI assistant (Claude) in October 2026.
- **How it was produced:** web searches of public sources (vendor product pages, resellers'
  feature lists, vendor blogs, user forums, regulations on eCFR, DoD issuances, specifications),
  each claim checked against the cited page where possible. SolidCraft's own code and docs were
  read to fit the proposal to what exists: `ROADMAP.md`, `docs/render-workspace.md`,
  `docs/render-jobs.md`, `docs/render-path.md`, `docs/configurations.md`,
  `docs/book/src/guide/assemblies.md` (motion studies, exploded views, Export Animation),
  `docs/web-runtime.md`, `docs/drawing-foundations.md`, `docs/showcase.md` and
  `crates/io/src/formats.rs`. No GPL code was read and no commercial CAD or rendering program
  was driven.
- **Not legal advice.** Advertising law, export control and design-right law are touched on
  because the features must respect them. Points that need an outside lawyer are marked
  **[Lawyer]**.
- **Team rules on sanctions and export control:** no one on our team goes to jail; SolidCraft
  helps users comply; prevention first. We have no in-house counsel, so unclear rules get the
  most conservative reading and are marked for an outside lawyer. Details in the
  [government use research](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-qk5riz/research/gov-research-nsnlookupjn-government-use.md).
- **Guiding principle:** SolidCraft should let a novice produce marketing material that looks
  like a professional studio made it, and give professionals every tool they have asked for.
  Both are covered in each section.
- **Marking:** "(unverified)" marks a claim not confirmed against a source. **Fact**,
  **opinion** and **marketing** (vendor claims) are labelled where they mix. Dates are as of
  2026-10-29.

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
  including flight-safety components. That work is why the reverse-engineering and
  counterfeiting controls in this file matter: counterfeit spare parts are a real and costly
  problem, and marketing images are one place a counterfeiter can start.

**This research is not complete.** It is a starting point: the author will keep researching
and adding to it, and others are encouraged to correct it and add their own findings.

## Summary

1. **The gap.** Today a company designs in CAD, exports a mesh, renders it in a separate
   program, animates it in a third, builds a web viewer in a fourth and lays out the brochure in
   a fifth. Every design change means doing it all again by hand, and every export is a chance
   to leak the real geometry. No single tool covers the whole path, and none checks whether
   the marketing material gives away too much.
2. **SolidCraft already has most of the engine.** A Render workspace with CPU and GPU path
   tracing and PNG export, exploded views, motion studies exported as GIF or PNG frames,
   configurations that hold colour and material variants, a headless snapshot renderer, a wasm
   web app, and every one of these is a command an agent can drive. What is missing is the
   pipeline that joins them, plus video, a web explorer, layout and the safety checks.
3. **Proposal: a Present workspace.** A shot list made from the design (hero stills,
   turntable, explode, motion clips, hotspots, spec table), published to outputs (images,
   video, web explorer, brochure and datasheet PDF, social cuts, AR) from brand templates. It is
   associative: when the design changes, outputs go stale, re-render in the background, and
   wait for approval again.
4. **The web explorer in three tiers.** Tier 0 serves only pre-rendered images (orbit, zoom,
   hotspots, an explode slider and a colour configurator, with no geometry ever sent). It is the
   default. Tier 1 streams rendered pixels from a server. Tier 2 sends a scrubbed mesh and is
   allowed only after the reverse-engineering check passes, and for controlled designs only
   after a recorded government clearance.
5. **Never enough to reverse engineer.** Marketing work happens on a derived marketing
   variant, never the master. Fixed controls apply: a deviation floor, feature simplification,
   no internals, resolution and view-count caps, no orthographic views or measuring. An
   automatic **reverse-engineering risk check** compares every output with the master and
   refuses release when it is too close.
6. **Falsified scale and canary details.** The marketing variant can be deliberately off-scale
   and carry hidden canary details, like trap streets on a map, so a copy made from marketing
   material can be recognised. The limit: nothing shown may contradict a published spec or
   claim. So such visuals carry "illustrative, not to scale", and every number in the brochure
   comes from approved spec data, never from the altered geometry.
7. **Every claim has evidence.** Each spec and claim in a brochure links to its source (a
   property calculation, a test report, a standard). Unsupported claims are flagged, and stale
   ones block publishing.

## Fundamentals

### The kinds of marketing output, and what each needs from CAD

| Output | Used for | What it needs from the design | Typical tool today |
|---|---|---|---|
| Hero still | Website, catalogue, press | Appearance, lighting, camera, clean geometry | KeyShot, Visualize, Blender, Fusion/Onshape render |
| Turntable / 360 spin | Product page, trade show | A set of evenly spaced views | KeyShot XR, Sketchfab, photo turntables |
| Exploded view | Explaining what is inside, sales and service | Assembly structure, explode steps | Composer, Fusion Animation, KeyShot |
| Animated controls / motion | Showing how it works | Joints, motion study, timing | Fusion Animation, Composer, Blender |
| Interactive web viewer | Product page, configurator, email | A web mesh or image set, hotspots, variants | Sketchfab, model-viewer, Marmoset Viewer, Threekit, vendor share links |
| AR | Phone "see it in your room" | glTF and USDZ files at real size | model-viewer, Shapr3D, Sketchfab |
| Video | Website, social, trade show loops | Camera paths, motion, titles, music | Composer, KeyShot, After Effects, Premiere |
| Spec sheet / datasheet | Sales, distributors, procurement | Approved numbers: size envelope, mass, ratings, options | InDesign, Word, PDM report tools |
| Brochure | Print and PDF | Images plus copy plus spec table plus brand | InDesign, Affinity Publisher |

### Formats that matter

- **glTF 2.0 / GLB** (Khronos): the common web 3D format. The 2025 configurator survey cited
  below found it is the primary format across brand configurators. Extensions used for
  marketing: `KHR_materials_variants` (colour options in one file), mesh compression
  (Draco, meshopt), and physically based material extensions.
- **USDZ**: what iOS Quick Look AR opens. Google's `<model-viewer>` takes a glTF `src` and a
  separate `ios-src` USDZ for iOS.
- **Image sequences:** the basis of KeyShot XR-style 360 viewers and of most car-maker
  configurators (Porsche, Tesla in the survey). No geometry is sent.
- **Pixel streaming:** a server renders and sends video, as Ferrari reportedly does with Unreal
  Engine 5. Geometry stays on the server.
- **Video:** MP4 (H.264/H.265) is universal but those codecs carry patent licensing
  **[Lawyer]**. WebM with VP9 or AV1 is royalty-free by its licensors' statements (unverified
  for every jurisdiction). A pure-Rust AV1 encoder exists (rav1e, BSD-2-Clause, unverified for
  this use), which fits SolidCraft's Rust-only, wasm-buildable rule.
- **PDF:** PDF/X for print shops, PDF/UA for accessible documents. Typst is an Apache-2.0
  typesetting system written in Rust that produces PDF (licence checked on docs.rs), so it is a
  candidate layout engine that passes `cargo xtask licences`.
- **C2PA Content Credentials:** a signed provenance record inside images, video and PDF
  (specification 2.2 and later). It says who made a file, with what, and whether AI was
  involved.

### Why a web viewer can't keep a mesh secret (fact)

Sketchfab, the largest 3D web host, says preventing piracy is "not entirely possible" because
data sent to the graphics card can be read, a limit it says applies to all WebGL and WebXR
apps. It compresses data, never exposes originals, monitors for "rippers" and forbids reverse
engineering in its terms. The three.js forum reaches the same view: no client-side viewer is
100% safe; obfuscation, chunked delivery and hidden identifiers only make copying harder, and
only server-side rendering is truly safe. **Any mesh a browser displays must be treated as
published.** This is the reason for the three tiers in section 3.

### Images leak shape too (opinion, with a source on the method)

A turntable is a set of photographs from known angles, and that is exactly the input to
photogrammetry and newer methods such as Gaussian splatting (see the 2025 arXiv paper on
Gaussian splatting and photogrammetry accuracy in Sources). Tier 0 is far safer than a mesh, but
not free of risk. So image count, resolution, pixel density on the product, and the spread of
viewing angles are capped and checked as well (section 4).

## Accuracy

What "accurate enough" means is the opposite here from the rest of SolidCraft research:

- **Appearance must be honest.** Colour, finish and proportions as the customer will
  experience them, so the buyer is not misled (see section 5 on advertising law).
- **Geometry must be deliberately inaccurate where it does not matter to the buyer.** Fillets,
  hole sizes, threads, internal parts and exact dimensions are of no use to a buyer and of great
  use to a copier.
- **Numbers must be exact and sourced.** Every number shown (mass, envelope, ratings) comes
  from approved spec data and carries a tolerance or "typical" label.

Suggested defaults (opinion, to be tuned by users and maintainers; none is a standard):

| Control | Default for Tier 0 and 1 | Default for Tier 2 (mesh) |
|---|---|---|
| Minimum deviation from master (95th percentile surface distance) | 0.5% of the largest dimension | 1% of the largest dimension, and at least 0.5 mm |
| Smallest feature kept | 3% of the largest dimension | 5% of the largest dimension |
| Mesh size | n/a | at most 50,000 triangles |
| Image size | at most 2048 px on the long side | same |
| Turntable views | at most 36, one elevation | n/a |
| Orthographic views | never | never |

## Programs

**Fact** unless labelled. Vendor feature claims are **marketing** and were not tested.

| Product | What it does for marketing | Notes |
|---|---|---|
| **KeyShot Studio** (KeyShot was renamed in 2024; KeyShot DAM is the renamed Digizuite) | Photoreal rendering, animation. **KeyShot Web** bundles KeyShot XR (360 viewers: turntable, spherical, tumble, animation; progressive loading), Web Configurator (pre-rendered variants in a browser, needs Pro) and Web Viewer (share a scene by link, optional password, 10 GB cloud storage). | KeyShot XR is the best-known image-based viewer: the model of Tier 0. Separate program, so design changes mean re-import. |
| **SOLIDWORKS Composer** | Technical illustration: raster and vector images, exploded views, interactive animations and parts lists, "web templates"; free Composer Player. Associative to the CAD model. | The strongest example of "updates when the design changes". Aimed at documentation more than glossy marketing. |
| **SOLIDWORKS Visualize** | Photoreal rendering for SOLIDWORKS users (unverified detail; vendor pages not fetched). | Separate application. |
| **Fusion (Autodesk)** | Render workspace (local and cloud rendering), and an Animation workspace with storyboards, auto-explode one or all levels, manual explode, move, rotate, visibility, callout and camera actions, reverse storyboards; geometry changes carry over. Shares by public link or embed. | The 2014 deep-dive said publish-to-video was still in development; later versions publish video (unverified version). Forum threads complain about local rendering failing and output size limits. |
| **Onshape Render Studio** | Cloud rendering on NVIDIA Iray with RTX GPUs on AWS, AI denoising, MDL materials. Launched in beta (2022) for Professional and Enterprise. | No animation or explode in the launch post. Exploded views exist elsewhere in Onshape. |
| **Shapr3D Visualization** | Real-time PBR on iPad, Mac and Windows, 100+ materials, custom GLB materials, **Generative Render** (describe a scene in words, AI makes the image), AR, USDZ export, web viewer links that need no account. | Good novice benchmark. AI backgrounds raise the honesty question in section 5. |
| **Blender** | Free, very capable rendering, animation and video editing. | GPL: SolidCraft must never read or reuse its code. Users can still export to it. |
| **Adobe Substance 3D Stager / Illustrator 3D, InDesign** | Staging 3D products and packaging into marketing scenes; InDesign is the standard for brochure layout. | No link back to the CAD model. |
| **Sketchfab** | 3D hosting and embeds; lossy-compressed data, private and password models on Pro. | Openly says ripping cannot be fully prevented. |
| **Google `<model-viewer>`** | Open-source web component: orbit, AR through Scene Viewer (Android) and Quick Look (iOS, with a USDZ), hotspots and variant swapping in its examples. | Code samples Apache-2.0. A good target for "export a glTF to your own site". |
| **Marmoset Viewer** | Embeddable real-time viewer from Marmoset Toolbag, with a WordPress plugin. | Game-art focus. |
| **Threekit and similar** | Commercial 3D/AR product configurators for e-commerce (unverified detail). | Configurators for retail, not tied to CAD. |
| **Pixel streaming** (for example Unreal Engine Pixel Streaming) | Server-side rendering streamed as video. | The only option that keeps geometry entirely on the server; costs GPU servers per viewer. |

**What none of them do (opinion from the survey):** no product found checks whether a render,
viewer or brochure gives away enough to copy the part; none builds a marketing variant with
falsified scale or canary details; none links each brochure claim to its evidence; and only
Composer stays associative from CAD to output.

## Complaints and requests

From forums, vendor pages and the author's experience. **Opinion**, summarised.

- **"Every design change means redoing the marketing."** Export, re-import, re-apply
  materials, re-render, re-lay out. Composer and Fusion's Animation workspace are the exceptions
  people point to.
- **Too many programs.** CAD, a renderer, an animation tool, a web host, a layout program, a
  video editor: each with its own licence and learning curve.
- **Cloud credits and cloud-only rendering.** Fusion's cloud rendering historically used
  credits; Onshape Render Studio runs on vendor GPUs. Users with confidential designs don't want
  them on someone else's render farm.
- **Local rendering failures and size caps** (Fusion forum threads in Sources).
- **Viewers leak models.** Artists on the Unreal and Polycount forums report models ripped
  from embeds and resold.
- **Spec sheets drift.** The brochure says 2.4 kg, the product is now 2.6 kg. Nobody owns the
  numbers.
- **Novices can't light or frame a shot.** Results look like CAD screenshots.
- **Pros want control:** camera paths, depth of field, render passes, batch rendering per
  configuration, scripting, their own brand templates, export to their DAM.
- **Defence companies can't market at all without a review process**, and there is no tool
  support for building the release package (high-security research, section 14).
- **Counterfeiting** (author's experience): copies of spare parts built from catalogue
  pictures and a few measurements. The copier needs only the outside shape and a scale.

## What users need

### 1. A Present workspace fed directly by the design

A new workspace next to Design, Render and (later) Drawing. It holds a **marketing kit** for a
design:

- **A marketing variant** (section 4.1): the scrubbed design all marketing is made from.
- **A brand kit:** logo, colours, fonts, footer and legal text, owned by the user and stored as
  their own asset. SolidCraft ships one neutral original template so novices start somewhere;
  no template copies another company's look.
- **A shot list:** named shots, each one a camera, a render setup (existing `render.*` saved
  setups), a configuration row, and a kind: still, turntable, explode, motion clip, hotspot
  tour.
- **Copy:** product name, tagline, feature bullets, and a **claims register** (section 6).
- **Outputs:** images, video, a web explorer, brochure PDF, datasheet PDF, social cuts, AR
  files, each with a status: Draft, Stale, Pending review, Released.

**Novice path:** one button, **Make a marketing kit**. It picks a hero angle (three-quarter
view, the camera framing the existing home view uses), a turntable, an explode if the design
is an assembly, a spec table from the design's properties, and the neutral template. It shows
the reverse-engineering check result before anything can leave.

**Pro path:** every shot, camera path, lens, depth of field, light and render pass is editable;
shots batch across every configuration row; everything is a command, so a script or an agent
over MCP can build a 200-variant catalogue overnight.

### 2. Builds on what SolidCraft already has (fact, from the repo)

| Already in SolidCraft | Use in the pipeline | Gap |
|---|---|---|
| Render workspace: CPU raster, CPU ray tracing, GPU path tracing, studio lighting, HDR environments, saved setups, `render.preflight` / `render.start` / `render.export` (PNG) | Stills and turntable frames | Transparent PNG only from some methods; no EXR; render passes |
| Exploded views (`explode.create`, `explode.show`), animated by `motion.export_animation` with `view` (and `back: true`) | Explode clips and the explode slider | Explode of placeholder internals in a variant |
| Motion studies exported as GIF or PNG frames (`motion.export_animation`, up to 4096 px a side and 1000 frames), drawn by the `solidcraft-cli snapshot` renderer, not the path tracer | Animated controls | Video encoding; path-traced frames; camera paths |
| Configurations with parameter, material and appearance columns (`config.*`) | The configurator's colour and option variants | Map rows to web variants |
| glTF **import** (`crates/io/src/formats.rs`: glTF export is `None`) | n/a | A glTF writer is needed for Tier 2 and AR |
| Wasm web app with WebGPU viewport (`apps/solidcraft-web`, `docs/web-runtime.md`) | The basis of a Rust/wasm web explorer | An embeddable, read-only viewer build |
| Drawing workspace in progress (#860) | Datasheet dimension views later | Not finished |
| Rendering milestone and interface rework (#6281) | Where the Present workspace UI would land | In progress |

### 3. The web explorer (Jason's request)

An embeddable control a company drops into its product page or sends as a link, so anyone can
explore the product without installing anything. It is a key output of the pipeline.

**What the viewer can do (all tiers, where the tier allows):**

- **Orbit and zoom** within limits the publisher sets: a minimum distance, a perspective field
  of view that can't be narrowed toward orthographic, optional fixed elevation.
- **Hotspots with callouts:** a marker on the product that opens a short text, image or clip.
  Hotspots attach to named faces of the marketing variant, so they follow design changes.
- **Explode slider:** drag to separate the parts along the explode steps of the variant.
- **Configurator:** colour, finish and option buttons from the configuration table.
- **Animated controls:** buttons that play pre-approved motion clips ("open the lid", "extend
  the arm"). They play recorded clips; they never run live kinematics, so real joint limits
  and speeds never reach the browser.
- **Guided tours:** a sequence of camera moves and callouts, played or stepped through.
- **AR on phones:** Tier 2 only, because AR needs a mesh (section 3.3).
- **Accessibility:** keyboard control, text alternatives for every hotspot, captions on clips,
  and the contrast floor from the
  [aging-eyes research](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-kc0dit/research/ui-research-nsnlookupjn-aging-eyes.md).

**What the viewer never offers, in any tier:** measuring, section views, wireframe or edge
display, orthographic views, model download, or the feature tree. This is Jason's rule from
the top of this file.

#### 3.1 Tier 0: image-based explorer (default)

Everything is pre-rendered by SolidCraft's own renderer: a grid of views (azimuth by a small
number of elevations), explode positions as frames, each configuration as its own image set,
hotspot positions stored in screen coordinates per frame. The viewer swaps images as you drag.
This is how KeyShot XR and most car configurators work. **No geometry is ever sent.**

- Fits every product, including cleared defence products, and needs only static hosting.
- Caps from section 4 apply: views, resolution, product pixel density.
- Cost: many images per configuration. For example, 36 views by 3 elevations by 5 colours is
  540 images; at the default caps this is tens of megabytes (estimate, opinion).

#### 3.2 Tier 1: streamed explorer (hosted service, optional)

A server renders the marketing variant on request and streams pixels. Free camera movement and
lighting, still no geometry in the browser. It is a hosted service with running GPU costs, so
it comes under the export-control section below (region blocks, screening, an outside lawyer's
sign-off) and is off unless the project or a company chooses to run it. Controlled designs only
after release clearance.

#### 3.3 Tier 2: mesh explorer and AR (only after the check passes)

The viewer receives a glTF of the marketing variant: decimated, defeatured, off-scale if the
publisher chose (section 5), no internals, materials only. Enables AR, true real-time lighting
and variants through `KHR_materials_variants`. **Treated as published geometry**, so:

- the reverse-engineering check must pass at the stricter Tier 2 thresholds;
- for a design marked ITAR, EAR, CUI or confidential, Tier 2 is **off** unless the release
  profile allows it and a recorded clearance covers the exact file hash (high-security
  section 14, steps 4 to 6);
- AR shows true size, so either the variant is true-scale (and the check is stricter) or AR is
  disabled for off-scale variants. SolidCraft never presents an off-scale model as true-size AR.

#### 3.4 How the embed is built

- **Rust and wasm.** SolidCraft's rule is that application code is Rust and builds for wasm. The
  explorer is a small read-only wasm build that shares the web app's renderer, not a separate
  JavaScript stack. The embed is one `<iframe>` or script tag that loads it.
- **Self-hosted or SolidCraft-hosted.** `present.export_explorer` writes a static folder a
  company uploads to its own site. A hosted option is possible later, under the export-control
  section.
- **Open export for other viewers.** For Tier 2, users may also export plain glTF and USDZ for
  `<model-viewer>`, Sketchfab or their own site. The same check and gate apply to the file.

#### 3.5 Anti-scraping (deterrents; honest about limits)

None of these stop a determined copier; they raise the effort and leave a trail.

- **Signed, expiring URLs** for image tiles and meshes, locked to the publisher's domains.
- **Rate limits** per visitor on hosted explorers, and a cap on how many distinct views one
  session may fetch (stops scripted harvesting of every angle for photogrammetry).
- **Chunked and packed delivery** in a SolidCraft-specific container, not loose PNG or GLB.
- **Per-release fingerprints:** every published image set and mesh carries the visible notice
  and signed metadata layers, plus canary details, from the IP research
  ([section 5, layers L1 to L6](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-2zu80u/research/ip-research-nsnlookupjn-ip-protection.md#5-digital-fingerprints-in-layers)).
  Per-dealer or per-recipient variants make a leak traceable with
  [`ip.trace`](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-2zu80u/research/ip-research-nsnlookupjn-ip-protection.md#6-who-leaked-this-the-detector).
- **Terms of use** on the embed forbidding extraction and reverse engineering.

#### 3.6 Analytics, with privacy first

Publishers want to know which colours people try and which hotspots they open.

- **Aggregate only, by default:** counts of views, configurations tried, hotspots opened, clip
  plays, time on page. No cookies, no personal data, no IP address stored.
- **Anything more** (per-visitor journeys, linking to a CRM) is the publisher's choice, on the
  publisher's own analytics, with their consent banner. SolidCraft never sends viewer data to
  the SolidCraft project.
- **Security signals** for the publisher: sessions that fetch an unusual number of views, or
  from many addresses in a pattern, are reported as possible harvesting.

### 4. Never enough detail to reverse engineer (Jason's rule, all products)

**Requirement:** no marketing output, alone or combined with the other released outputs of the
same product, may carry enough information to manufacture a functional copy or a
look-alike that would pass as genuine. This applies to every design, not only ITAR or
confidential ones. It can be relaxed only by a named person, with a reason, and the relaxation
is recorded.

#### 4.1 Work only on a marketing variant

`marketing.derive` (shared with high-security section 14) makes a separate design from the
master by allow-list: only the chosen outer bodies come across; parameters, sketches, history,
PMI, tolerances, materials, part numbers and metadata do not. For an unmarked design this is the
same command with a lighter profile; for a marked design it carries the marking and goes
through the full release gate.

#### 4.2 Fixed controls (opinion; thresholds in a policy)

| Control | What it does | Why |
|---|---|---|
| **Deviation floor** | The variant must differ from the master by at least the policy's minimum surface distance (95th percentile), across the whole surface | A near-exact shell is a manufacturing model |
| **Feature simplification** | Removes or merges holes, threads, small fillets and chamfers, text, engraved part numbers and interface detail below a size threshold | Interfaces and small features are what a copier measures |
| **No internals** | Internals replaced by labelled placeholder blocks, or left out; explode shows covers lifting or placeholders | Internal arrangement is the product's know-how |
| **Scale obfuscation** (optional, section 5) | Non-uniform scale and altered proportions of non-claimed features | Removes a reliable scale |
| **Decimation** | Caps triangles and smooths curvature | Mesh resolution is measurable precision |
| **Image caps** | Long side, pixel density on the product, number of views, elevation spread | Limits photogrammetry |
| **No orthographic views** | Perspective only, field of view floor, no straight-on side or top views | Orthographic views are drawings |
| **No measuring tools** | Viewer has no ruler, section, wireframe or download | Obvious |
| **No scale references** | No rulers, grids, coins, hands or known objects next to the product, unless the publisher accepts the stricter check | Known objects give scale back |
| **Motion is illustrative** | Clips authored by hand in the variant; never real joint limits, speeds or sequences | Kinematics reveal function and performance |

#### 4.3 The automatic reverse-engineering risk check

`marketing.re_check` runs on every output before it can leave, and again on the full set of a
product's released outputs (because many safe pictures together can be unsafe). It compares
against the master inside SolidCraft; the master never leaves.

Measures (opinion; first version):

1. **Geometric closeness:** surface distance between the variant (or any mesh output) and the
   master, as a distribution. Refuse if the 95th percentile is under the floor.
2. **Feature survival:** how many of the master's holes, bosses, slots, threads and mating
   faces can still be found in the variant. Refuse if any feature marked critical or key
   survives (using the importance levels from the
   [in-process inspection research](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-y2w06n/research/cam-research-nsnlookupjn-in-process-inspection.md)).
3. **Scale truth:** whether the output, plus any published dimension, lets a viewer recover
   true scale.
4. **Image coverage:** for each image set, the angular coverage and pixel density on the
   product. A rough photogrammetry-readiness score from view overlap and baseline. Refuse above
   the cap.
5. **Text and labels:** no dimensions, tolerances, part numbers, materials or CAGE codes in
   captions, callouts or burned-in text, unless they come from the approved spec list.
6. **Combined release:** the union of everything already released for the product.

**Result:** pass, or refuse with plain reasons ("the mounting holes still match the master
within 0.1 mm; simplify them or raise the deviation"). A named approver can override with a
recorded reason for unmarked designs only. **Marked designs cannot be overridden here**; they
follow high-security section 14.

**Honest limits:** no automatic check can prove that something can't be copied. A determined
team with the real product in hand will copy it anyway. The check exists to stop marketing
material being the easy route, and to give the approver evidence.

### 5. Falsified scale and canary details (Jason's request), within advertising law

#### 5.1 What SolidCraft can do

- **Non-uniform scale:** the variant is scaled by slightly different factors per axis, chosen
  at random within a policy range and recorded privately with the variant (opinion: a few per
  cent, so it looks right but measures wrong).
- **Altered proportions** of features that are not part of any claim: rib spacing, vent
  pattern, fillet sizes, logo position.
- **Canary details:** small, plausible, deliberately wrong details unique to a release or a
  channel: a vent slot one fewer, a chamfer on one edge only, a boss slightly off-centre. Like
  trap streets on a map, a copy that reproduces them was made from marketing material. Each
  canary set is recorded in the IP research's provenance ledger and detected by `ip.trace`
  (see [IP research section 6](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-2zu80u/research/ip-research-nsnlookupjn-ip-protection.md#6-who-leaked-this-the-detector)).
  Per-dealer variants get different canaries, so the source of a copy can be traced.

**Never in manufacturing data.** Canaries and falsified scale live only in marketing variants,
which can't be used for manufacturing: `marketing.derive` marks the variant "marketing only",
and CAM, drawing and export-for-manufacture commands refuse it. This keeps the IP research's
rule that geometric fingerprints never touch critical, key or safety-critical features of a real
part.

#### 5.2 The limit: never contradict what is published

Altering geometry is acceptable only while the picture does not mislead buyers about the
product. **[Lawyer]** for each market.

- **US:** the FTC Act section 5 forbids deceptive acts or practices in commerce; the Lanham Act
  section 43(a) lets competitors sue over false or misleading advertising (unverified for
  applicability to these specific cases).
- **UK:** the Digital Markets, Competition and Consumers Act 2024 replaced the Consumer
  Protection from Unfair Trading Regulations 2008 for misleading actions and omissions
  (unverified effective date details).
- **EU:** the Unfair Commercial Practices Directive 2005/29/EC.
- **Government buyers:** a proposal or catalogue sent to a government customer can be a
  representation; false statements to the US government can bring False Claims Act liability.
  **[Lawyer]**
- **Design registrations and patents:** registered designs are defined by their
  representations, and patent drawings must be accurate. **Never use falsified marketing
  images in a design-right application, patent or trade-dress filing.** SolidCraft blocks
  marketing variants from those exports. **[Lawyer]**

So SolidCraft enforces:

1. **Every number comes from the spec data, never from the variant.** Dimensions, mass and
   ratings in a brochure or viewer are read from the approved spec list (section 6), with units
   and tolerances. The variant has no dimensions to read.
2. **"Illustrative, not to scale"** is added automatically to every output made from an
   off-scale variant (caption, viewer footer, video end card, PDF image caption), and can't be
   removed while the variant is off-scale.
3. **Claimed features are locked.** Any feature named in a claim ("fits a standard 19-inch rack",
   "four mounting points", "USB-C port") is protected from alteration and from canaries, and
   the check confirms the picture agrees with the claim.
4. **No "actual size" or true-scale AR** from an off-scale variant (section 3.3).
5. **Colours and finishes stay honest.** The appearance is the product's real appearance.

### 6. Spec sheets, datasheets and brochures that stay true

- **An approved spec list per design:** each entry is a value, unit, tolerance or "typical",
  source and approver. Sources can be live (mass and envelope from the
  [physical properties research](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-uxzal2/research/design-research-nsnlookupjn-physical-properties.md),
  rounded to a published precision) or documents (a test report, a standard, a certificate).
  The envelope is published only to the precision a buyer needs (opinion: whole millimetres or
  coarser), and the re-check counts it as a scale clue.
- **A claims register:** every marketing claim ("weatherproof to IP67", "30% lighter than the
  previous model") links to its evidence. Claims with no evidence are flagged for novices with
  a plain explanation; pros can require evidence before publishing.
- **Stale means blocked:** when the design changes and a live value moves outside its published
  tolerance, the spec entry, every output using it and its claims go Stale, using the content
  hash and stale-artefact idea from the
  [compliance research](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-ymp2ma/research/compliance-research-nsnlookupjn-itar-quality.md).
  Stale outputs can't be published until re-approved.
- **Layout:** brochure and datasheet templates rendered to PDF, candidate engine Typst
  (Apache-2.0, Rust). Templates are text files under version control; blocks bind to the
  shot list, spec list, configuration table and copy. Outputs: screen PDF, print PDF with bleed
  (PDF/X), accessible PDF (PDF/UA), and a QR code that opens the web explorer.
- **Translations:** copy and spec labels per language, so one design produces a brochure per
  market. Units follow the market (mm or inch) from the same spec data.

### 7. Video and social

- **Video encode** of motion, explode, turntable and tour shots, rendered by the path tracer
  where the hardware allows. Royalty-free WebM (AV1 or VP9) built in; MP4 (H.264) only through
  an encoder the user's system provides, because of codec patent licensing **[Lawyer]**. PNG
  frames remain for any editor.
- **Titles, captions and end cards** from the brand kit; the "illustrative, not to scale" card
  added automatically when it applies.
- **Social cuts:** square and vertical crops of the same shots.
- **Camera paths:** keyframed cameras with easing, shared by video and guided tours.

### 8. Associative updates and approvals

- Every output records the hash of the variant, the shot, the render setup, the spec list and
  the template it was made from.
- When any of them changes, the output goes **Stale**, a background job re-renders it (bounded,
  cancellable, off the window thread, as the performance skill requires), and it returns to
  **Pending review**.
- **Approvals are per file hash.** A re-render voids the approval, as in high-security section
  14, step 7. For unmarked designs the approver is a person the company names; for marked
  designs the full government release path applies.
- **Provenance on every file:** a C2PA Content Credential stating it was made in SolidCraft
  from a marketing variant (never the master's identity), whether AI was used, and the approval
  record. Plus the IP research's fingerprint layers.

### 9. AI in the pipeline (opinion)

- **Useful:** writing first-draft copy from the claims register, suggesting shots, generating
  background scenes (as Shapr3D's Generative Render does), translating.
- **Rules:** AI works only on the marketing variant and released images, never on the master. An
  AI step must not alter the product's geometry in the picture (it can invent a background,
  not a new feature), outputs are labelled as AI-assisted in their C2PA record, and claims it
  writes still need evidence. For marked designs, AI connectors follow the marking
  announcement in high-security section 4.

### Proposed commands (sketch)

All dialog-free, all with input and output schemas, all audited when a security profile is on.

| Command | Input | Output |
|---|---|---|
| `marketing.derive` | `{source, profile}` | marketing variant design, diff report (shared with high-security section 14) |
| `marketing.obfuscate` | `{variant, scale_range?, canaries?, channel?}` | updated variant, private record of scale and canaries |
| `marketing.re_check` | `{variant, outputs?, tier}` | `{pass, measures, reasons}` |
| `present.kit` | `{design, template?}` | a marketing kit with default shots (the novice button) |
| `present.shot` | `{kit, name, kind, camera?, setup?, config_row?, clip?}` | shot |
| `present.spec` | `{kit, entries}` | approved spec list, stale entries |
| `present.claim` | `{kit, text, evidence}` | claim record |
| `present.render` | `{kit, outputs?}` | job (uses `render.*` jobs) |
| `present.export_explorer` | `{kit, tier, path}` | static explorer folder, or refusal with reasons |
| `present.export_document` | `{kit, template, kind: "brochure" \| "datasheet", path}` | PDF path, or refusal |
| `present.export_video` | `{kit, shot, path, codec?}` | video path, or refusal |
| `present.status` | `{kit}` | every output with Draft / Stale / Pending / Released |
| `release.request`, `release.record` | as in high-security section 14 | review package, cleared hashes |

### What contributors could build first (opinion, smallest useful steps)

1. **Video encode** of the existing `motion.export_animation` frames to WebM.
2. **Turntable shot**: N path-traced views around the product through `render.*` jobs.
3. **Tier 0 explorer**: the read-only wasm viewer that plays an image grid with hotspots.
4. **glTF writer** in `crates/io` (export side of the existing glTF row), needed for Tier 2.
5. **`marketing.derive`** with the allow-list and defeature profile (shared with high-security).
6. **`marketing.re_check`** geometric closeness and feature survival measures.
7. **Spec list and datasheet template** with Typst.
8. **Configurator** from configuration rows, then explode slider, then guided tours.

## Export control, sanctions and government use

*Opinion and policy proposals, not legal advice. Builds on the
[government use research](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-qk5riz/research/gov-research-nsnlookupjn-government-use.md)
and [high-security section 14](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-2fa4g1/research/security-research-nsnlookupjn-high-security.md#14-marketing-an-itar-product-renders-animations-exploded-views-and-web-viewers).*

**Our three rules:**

- **No one goes to jail.** Our team has no in-house counsel. So wherever a rule is unclear we
  take the most conservative reading and refuse rather than warn. Questions that need an
  outside export-control lawyer are marked as such, and free official help (BIS, DDTC, OFAC) is
  the first call.
- **Help users comply.** SolidCraft stops and explains in plain language, using rule packs for
  each jurisdiction that the administrator picks. It flags conflicts between countries' laws,
  never resolves them, and never gives legal advice.
- **Prevention first.** Nothing in this file that is hosted, shared, paid or networked ships
  until its compliance gates are in place and checked: region blocks, screening, ITAR flag
  handling, government-mode behaviour, and an outside lawyer's sign-off for any hosted or paid
  service.

**What in this file crosses a border:** every published output (it is public by intent), the
hosted Tier 1 streamed explorer, any SolidCraft-hosted explorer, and outside marketing
studios or translators who receive the marketing variant.

- **Marketing material of a controlled product is technical data until cleared.** ITAR
  excludes only "basic marketing information on function or purpose or general system
  descriptions" (22 CFR 120.33(b)); images of how it is built, assembled or performs are not
  excluded, and public release needs government approval (120.34(a)(7); DoD review by DOPSR
  under DoD issuances, see high-security section 14). EAR items follow 15 CFR 734.7. So every
  output of a marked design stays marked, can't go to a public destination and has Tier 2 off
  until a recorded clearance covers its exact hash. **[Lawyer]**
- **Sending the variant to an outside studio or translator before release is an export** if
  the recipient is a foreign person or abroad. SolidCraft refuses to send a marked variant to a
  recipient not confirmed by the export compliance officer. **[Lawyer]**
- **Hosted parts block sanctioned regions and screen people.** Any explorer hosting or
  streaming service the project would run blocks comprehensively sanctioned regions, plus
  Russia and Belarus (US, EU and UK rules restrict CAD/CAM software and related services
  there), and screens publisher accounts and payers against the US Consolidated Screening List
  and the UK, EU and UN lists.
- **No support there.** No support, consulting or custom work for users in sanctioned regions
  or for listed parties.
- **Government mode.** At the Controlled level, the Present workspace exports only to approved
  local storage and hosted explorers are off; at the Isolated level only local outputs exist; a
  design flagged Export controlled never leaves through any of these paths before clearance.
- **Only generic data is shared.** Nothing derived from a controlled design goes into a shared
  template library, example kit or issue report. Shared brand and brochure templates contain
  no product data.
- **Code stays public.** Every build the project ships is public, which keeps it "published"
  software outside the EAR. The rules are tracked by the standing export and sanctions watch.
- **Checked for controlled data (2026-10-29):** this file contains no controlled technical
  data. Its examples are public sources, generic values or made-up stand-ins, and the author's
  experience is stated in general terms.

## Glossary

- **Canary detail:** a small deliberate difference in a marketing variant that identifies
  copies made from it, like a trap street on a map.
- **Configurator:** a viewer that lets a buyer switch colours and options.
- **C2PA / Content Credentials:** a signed provenance record embedded in media files.
- **Deviation floor:** the minimum distance by which a marketing variant must differ from the
  master.
- **DOPSR:** the Defense Office of Prepublication and Security Review, which clears DoD
  information for public release.
- **glTF / GLB:** the Khronos 3D transmission format used by most web viewers.
- **Gaussian splatting / photogrammetry:** methods that rebuild 3D shape from photographs.
- **Hotspot:** a clickable marker on a product in a viewer.
- **Marketing variant (marketing derivative):** a separate, scrubbed design made from the master
  only for marketing.
- **Pixel streaming:** rendering on a server and sending video to the browser.
- **PDF/X, PDF/UA:** PDF profiles for print production and for accessibility.
- **Stale:** an output whose sources changed since it was made or approved.
- **Turntable:** a set of views around a product, shown as a spin.
- **USDZ:** the 3D format iOS uses for AR Quick Look.

## Open questions

1. **Thresholds.** What deviation floor, feature size and image caps are right by product
   size and industry? Should they be set per company, per product, or per importance level?
2. **Combined-release check.** How to score many outputs together without a full
   photogrammetry run on every publish?
3. **Tier 2 for small products.** For a small part, a 1% deviation may be below what matters.
   Is Tier 2 ever acceptable for parts whose outer shape is the whole product (a bracket, a
   knob)? Possibly never by default.
4. **Off-scale AR.** Is there any honest way to offer AR for an off-scale variant, or should
   AR always require a true-scale variant with a stricter check?
5. **Canary legality.** Are canary details in a marketing image a problem in any market where
   the image could be read as a specification? **[Lawyer]**
6. **Codec licensing** for MP4 export in an open-source program. **[Lawyer]**
7. **Hosting.** Should the project ever host explorers, or only produce static folders
   companies host themselves? Hosting brings the whole compliance stack with it.
8. **Typst** as a dependency: size in the wasm build, and whether templates should be Typst
   files or a SolidCraft template format that compiles to it.
9. **Maintainers' view** on where this fits: a Present workspace, part of the Render workspace
   rework (#6281), or a plugin.

## Sources

Accessed 2026-10-29 unless stated.

**SolidCraft repository (main at 2016ba7):** `ROADMAP.md`; `docs/render-workspace.md`;
`docs/render-jobs.md`; `docs/render-path.md`; `docs/configurations.md`;
`docs/book/src/guide/assemblies.md`; `docs/web-runtime.md`; `docs/drawing-foundations.md`;
`docs/showcase.md`; `crates/io/src/formats.rs`.

**Related research by the same author:**
- [High-security research, section 14: marketing an ITAR product](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-2fa4g1/research/security-research-nsnlookupjn-high-security.md#14-marketing-an-itar-product-renders-animations-exploded-views-and-web-viewers)
- [IP protection research: fingerprints, leak tracing, provenance](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-2zu80u/research/ip-research-nsnlookupjn-ip-protection.md)
- [Government use research](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-qk5riz/research/gov-research-nsnlookupjn-government-use.md)
- [Compliance research (stale artefacts, release states)](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-ymp2ma/research/compliance-research-nsnlookupjn-itar-quality.md)
- [In-process inspection research (importance levels)](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-y2w06n/research/cam-research-nsnlookupjn-in-process-inspection.md)
- [Physical properties research](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-uxzal2/research/design-research-nsnlookupjn-physical-properties.md)
- [UI for aging eyes research](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-kc0dit/research/ui-research-nsnlookupjn-aging-eyes.md)

**Products:**
- KeyShot Web (KeyShot XR, Web Configurator, Web Viewer), Leap Australia reseller page: https://www.leapaust.com.au/product/keyshot-web
- KeyShot, "Discover the new KeyShot" (31 July 2024): https://keyshot.com/blog/the-next-steps-keyshot-company-evolution
- SOLIDWORKS Composer, Javelin reseller page: https://www.javelin-tech.com/3d/technology/solidworks-composer/
- Autodesk, "Fusion 360 Animation Deep-dive": https://www.autodesk.com/products/fusion-360/blog/fusion-360-animation-deep-dive/
- Autodesk forum, "Cannot render locally": https://forums.autodesk.com/t5/fusion-support-forum/cannot-render-locally/m-p/9035077
- Autodesk forum, output size question: https://forums.autodesk.com/t5/fusion-design-validate-document/i-must-be-dumb-turn-on-rendering-gt-only-doc-window-size/m-p/6201389
- Autodesk forum, "Fusion 360 and Cloud Credits": https://forums.autodesk.com/t5/fusion-design-validate-document/fusion-360-and-cloud-credits/m-p/5634068
- Onshape, "Onshape launches Render Studio": https://www.onshape.com/en/blog/onshape-launches-render-studio
- Onshape, "Introducing exploded views in Onshape": https://www.onshape.com/videos/introducing-exploded-views-in-onshape
- Shapr3D Visualization: https://www.shapr3d.com/features/visualization
- Adobe Substance 3D packaging use case: https://www.adobe.com/products/substance3d/use-cases/packaging-design/s3d-illustrator.html
- Marmoset Viewer: https://marmoset.co/viewer/
- Chrome Developers, "Augmented reality with model-viewer": https://developer.chrome.com/blog/model-viewer-ar?hl=en
- model-viewer example, swap model variants: https://modelviewer.dev/examples/scenegraph
- Epic Games, Pixel Streaming overview: https://dev.epicgames.com/documentation/en-us/unreal-engine/overview-of-pixel-streaming-in-unreal-engine
- DEV Community, "We reverse-engineered 12 product configurators": https://dev.to/podifai/we-reverse-engineered-12-product-configurators-heres-whats-actually-under-the-hood-12ld

**Viewer security:**
- Sketchfab Help, "Model security": https://help.sketchfab.com/en/articles/16152580-model-security
- three.js forum, "How to prevent ripping": https://discourse.threejs.org/t/how-to-prevent-ripping/12473
- Unreal Engine forum, models resold without permission: https://forums.unrealengine.com/t/warning-external-website-reselling-sketchfab-fab-artists-models-without-permission-deep3dsea/2708744
- arXiv 2508.07483, "Novel View Synthesis with Gaussian Splatting: Impact on Photogrammetry Model Accuracy and Resolution": https://arxiv.org/abs/2508.07483v1

**Formats and libraries:**
- Typst README (Apache-2.0): https://docs.rs/crate/typst/0.15.1/source/README.md
- C2PA Technical Specification 2.2: https://spec.c2pa.org/specifications/specifications/2.2/specs/C2PA_Specification
- Khronos, live glTF asset editing (variants): https://www.khronos.org/blog/live-gltf-asset-editing-in-your-browsereven-in-ar

**Regulation:**
- 22 CFR 120.33, technical data (eCFR): https://www.ecfr.gov/current/title-22/chapter-I/subchapter-M/part-120/subpart-C/section-120.33
- DoD Instruction 5230.29, "Security and Policy Review of DoD Information for Public Release" (2014, Change 1 2017): https://www.esd.whs.mil/Portals/54/Documents/DD/Issuances/dodi/523029p.pdf
- FTC Act section 5, Lanham Act section 43(a), UK DMCC Act 2024, EU Directive 2005/29/EC, US False Claims Act: cited from general knowledge, not fetched for this file (unverified).
