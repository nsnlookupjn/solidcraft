# UI research: designing SolidCraft for aging eyes and low vision

Research on **how the interface should look for people whose eyes have aged or who have a visual
disability**: what happens to the eye with age and with common eye disease, what published
research and standards say about text size, contrast, colour, brightness and light versus dark
modes, what is different on a shop floor, and **concrete settings and presets SolidCraft could
offer**, for the CAD workspaces and for a future CAM interface.

The motivation comes from Jason's own experience: as he gets older he needs certain colours,
brightness and sizes to work comfortably, and many machinists are older. A CAD/CAM program that
is only comfortable for 25-year-old eyes leaves out a large part of the people who make parts.

It pairs with the CAM notes that mention user-facing controls, mainly the conservative-to-aggressive
slider in
[cutting tool physics](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-cutting-physics/research/cam-research-nsnlookupjn-cutting-physics.md)
and the
[workholding](https://github.com/nsnlookupjn/solidcraft/blob/research/cam-workholding/research/cam-research-nsnlookupjn-workholding.md)
note, and with the part traveller in the in-process inspection note.

Licence: MIT OR Apache-2.0, the same as SolidCraft (see the Licence section of CONTRIBUTING.md).

## About this research

- **Author:** Jason ([@nsnlookupjn](https://github.com/nsnlookupjn)). Idea and direction by Jason;
  source research and writing done with an AI assistant (Claude) in October 2026.
- **How it was produced:** web searches of public sources (peer-reviewed vision science and
  ophthalmology papers and their abstracts, public-health pages, W3C accessibility documents,
  standards' public catalogue pages, type foundry pages), each claim checked against the cited
  page where possible. SolidCraft's own code and docs were read for the "What SolidCraft already
  has" section; the contrast ratios there were computed for this file from the colours in
  `assets/styles/ui/palettes.rs` with the WCAG 2 formula. No commercial CAD/CAM program was driven,
  scripted or looked at, no GPL/LGPL/AGPL source was read, and nothing is copied beyond short
  quotes. Standards with paid text (ISO 9241-303, EN 12464-1) are described from public summaries
  only.
- **This is not medical advice.** It summarises research so the software can be designed well.
  Anyone whose sight is changing should see an optometrist or ophthalmologist; many of the
  problems below (presbyopia, cataract) are very treatable.
- **Marking:** "(unverified)" marks a claim not confirmed against a source in this session.
  **Fact**, **opinion** and **marketing** are labelled where they mix.

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
- **Why this topic:** after more than 35 years at screens and machines, the author needs
  particular colours, brightness and text sizes to work comfortably, as many experienced
  machinists do.

> **[TODO Jason: personal experience]** What changed for you and roughly when: which text became
> hard to read first, whether you prefer light or dark screens now (and whether that differs in
> the shop and at home), which colours are hard to tell apart, how glare from shop lights or
> windows affects the screen, what glasses you use at the computer and at the machine (readers,
> progressives, computer/office lenses, safety glasses), and any CAM or control screens that are
> good or bad for you. Label this as personal experience.

## Summary

1. **This is most users, not a niche.** About 1.8 billion people (25% of the world) had
   presbyopia in 2015 (Fricke et al. 2018). By age 80 more than half of Americans have, or have had
   surgery for, cataract (Prevent Blindness). In the US, machinists' average age is 45.7 and about
   38% are aged 50–64 (Data USA, 2024). Designing for aging eyes is designing for the core
   audience of a CAD/CAM program used in shops.
2. **Older eyes need more light, more contrast and bigger detail, and suffer more from glare.**
   A typical 60-year-old gets about one third of the retinal light of a 20-year-old (Lighting
   Research Center). Contrast sensitivity for fine detail falls steeply with age while coarse
   detail is spared (Owsley et al. 1983, modelled by Shao et al. 2024). Light scattered inside the
   eye roughly doubles by 65 and triples by 77 (van den Berg et al. 2007, as cited by Łabuz et al.
   2015). The yellowing lens makes blue–yellow differences the hardest colour differences to see
   (Wuerger 2013).
3. **No single theme is best for everyone.** Dark text on a light background reads better for
   younger *and* older adults with clear eyes (Piepenbrock et al. 2013), and older adults showed
   more mental load with dark mode in bright rooms (Sethi & Ziat 2023). But people with cloudy
   eye media (cataract) read 10–50% faster with light text on dark (Legge et al. 1985). So
   SolidCraft should offer **both, done well**, and help people pick.
4. **Size should follow viewing distance.** At SolidCraft's default 100% scale, body text on a
   common 24-inch 1080p monitor is about 10 arcminutes of x-height at a 60 cm desk and about 6
   arcminutes at 1 m in front of a machine. That is small for aging eyes. A **"How far are you
   from the screen?"** setting could compute the scale instead of making people guess.
5. **SolidCraft already has good foundations:** a High Contrast theme, Protan/Deutan/Tritan colour
   vision modes with stripes on failing faces, a 50–250% UI scale with Ctrl/Cmd +/-, a font
   override and contrast checks on some token pairs. The gaps are text size separate from UI
   scale, minimum text sizes, line weights and point sizes in the 3D view, contrast of dim text,
   borders and some sketch colours, and a **one-click vision preset** that bundles it all.
6. **Recommendation:** add a **Vision** section (or Preferences page) with five presets (Standard,
   Clear, Clear Dark, Shop Floor, Large & Bold), each a bundle of existing and new settings that
   users can then fine-tune; a contrast and size floor that every theme must pass; and rules for
   the CAM interface (confidence slider, status colours, setup sheets) so they never depend on
   colour or fine detail alone.

## Who this is for

| Group | Size of the group | Source |
| --- | --- | --- |
| Presbyopia (loss of near focus, from about 40–45) | ~1.8 billion people worldwide in 2015 (25%); 826 million with near vision impaired because it was uncorrected or under-corrected | Fricke et al. 2018, *Ophthalmology* 125(10) |
| Cataract | More than half of Americans by age 80 have cataract or have had cataract surgery | Prevent Blindness infographic (2024) |
| Congenital colour vision deficiency | About 8% of men and 0.5% of women of Northern European ancestry, mostly red–green (deutan more common than protan) (unverified: widely cited figure, not checked against a primary source this session) | — |
| Machinists (US) | Average age 45.7; about 38% aged 50–64 | Data USA, SOC 51-4041, 2024 data |

Most older users have **several** of these at once (for example presbyopia plus early cataract
plus a weaker blue sense), and they use the program for hours at a time, so small costs in
legibility add up to fatigue.

## What happens to the eye with age (the medical side)

Each row is a change, what the research says, and what it means for the screen.

| Change | What the research says | What it means for the UI |
| --- | --- | --- |
| **Presbyopia** (lens stiffens, near focus lost) | Most people need reading glasses by about 45; by about 65 changing focus is nearly impossible (Lighting Research Center). | A screen sits at *intermediate* distance (50–100 cm). Reading glasses are set for ~40 cm and progressives have a narrow intermediate zone, so users tilt their head or lean in. Bigger text lets the eye work at a distance its glasses already serve. |
| **Less light reaches the retina** (smaller pupil, "senile miosis", and a denser, yellower lens) | A typical 60-year-old receives "about one-third the retinal illuminance of a 20-year old" (Lighting Research Center). | Dim themes and low-luminance text (dark grey on black, mid-grey "dim" labels) lose much more for older users. Their effective screen is a third as bright. Brightness and luminance contrast matter more than hue. |
| **Lower contrast sensitivity for fine detail** | In Owsley et al.'s data (ages 20–80), sensitivity at 4 cycles/degree falls from about 251 in the 20s to 56 in the 80s, and at 16 cycles/degree from 29 to 5; coarse detail (0.5 cycles/degree) stays about the same (Shao et al. 2024, modelling Owsley et al. 1983). Shao et al. suggest a contrast of at least 0.02 and avoiding detail finer than about 8 cycles/degree for visualisations that every age can see. | Thin lines, small icons, hairline borders and small text are exactly the "high spatial frequency" content that fades first. Thicker strokes and larger features help more than stronger colours. Low-contrast decorative borders become invisible. |
| **More light scattered inside the eye (glare)** | Intraocular straylight "increases 2-fold at 65 years and is tripled by the age of 77" in eyes with good acuity (van den Berg et al. 2007, cited in Łabuz, Reus & van den Berg 2015). Scatter adds a "luminous veil" that lowers contrast and saturation (Lighting Research Center). | Large bright areas (white panels, a white 3D background, overhead lights reflected on the screen) wash out everything near them. A bright-white viewport with thin dark edges is the worst case. Offer off-white backgrounds, a dim-but-light theme, and dark themes that are not pure black-on-white inverted. |
| **Yellowing lens: weaker blue sense** | In 185 observers aged 18–75, colour discrimination fell with age on all three axes, most strongly on the tritan (blue–yellow) axis: slope 0.254 versus 0.004 (protan) and 0.003 (deutan) (Wuerger 2013, *PLoS ONE*). Perceived hue stays fairly stable, which suggests the brain compensates. | Blue-vs-purple, yellow-vs-white, and blue-on-dark-grey distinctions are the ones older users lose. Dark blue text or lines on a dark background are doubly bad (low luminance and the weak axis). SolidCraft's **Tritan** mode is relevant to many older users who would never call themselves colour blind. |
| **Cataract** (cloudy lens) | Readers with cloudy ocular media read 10–50% faster with white-on-black text (rate ratios 1.10–1.52); readers with clear media showed little difference (0.97–1.11). The authors attribute it to scatter in the eye lowering the contrast of black-on-white more (Legge, Rubin, Pelli & Schleske 1985, *Vision Research* 25(2)). | This is the strongest evidence **for** a dark theme, but only for this group. The dark theme should then have high-luminance text (near white, not mid-grey) and avoid large bright patches. |
| **Macular degeneration (AMD) and other central field loss** | Readers with intact central fields read fastest at intermediate character sizes; readers with central field loss needed much larger print (best at 12′–24′ in that study) and read much more slowly (median 25 versus 130 words/minute) (Legge et al. 1985). | These users need a magnifier and very large text, and benefit from information placed where they expect it. A "Large & Bold" preset and a good zoom of the whole UI (already present) help; they may also use the operating system's magnifier, so SolidCraft should not break it. |
| **Glaucoma and other peripheral field loss** (unverified: general clinical knowledge, not sourced this session) | Side vision narrows while central vision can stay sharp. | Messages that appear briefly in a corner (SolidCraft's status chip at the bottom left) can be missed. Important messages should also appear near where the user is working, and stay until dismissed. |
| **Slower adaptation between bright and dark** (unverified) | Older eyes take longer to recover after looking at a bright light or a bright screen. | Switching between a bright shop and a dark screen (or a white dialog popping up over a dark viewport) costs more. Keep the brightness of dialogs close to that of the workspace in a given theme. |
| **Dry eye and fatigue** (unverified) | Screen work reduces blinking; more common with age. | Larger text and less squinting reduce strain; nothing specific for the UI beyond that. |

**Fact:** the changes above are well documented. **Opinion:** the "what it means" column is this
note's design interpretation of them.

## What the design research says

### Light or dark (display polarity)

- **Positive polarity (dark text on light) generally wins for clear eyes, at every age.**
  Piepenbrock, Mayr, Mund & Buchner (2013, *Ergonomics*) titled their finding "Positive display
  polarity is advantageous for both younger and older adults" (title only seen in this session).
  Buchner and colleagues later explained the advantage by the higher overall display luminance,
  which shrinks the pupil and sharpens the image (Uni Mannheim record "The advantage of positive
  text-background polarity is due to high display luminance"; abstract not fetched, unverified).
- **Older adults showed more mental load with dark mode in bright surroundings**, younger adults
  with dark mode in dim surroundings, and older adults tended to prefer positive polarity (Sethi &
  Ziat 2023, *Ergonomics* 66(12), as summarised in a later paper).
- **Cataract reverses this** (Legge et al. 1985, above).
- **Conclusion (opinion):** the default for a lit shop or office should be a light theme with
  slightly off-white panels; a well-built dark theme with bright text is the right choice for
  people with cataract, strong glare sensitivity, or who work in a dim room. SolidCraft's existing
  **Viewport: Light/Dark** option (the 3D view in one polarity while the window uses the other)
  is useful here: for example light panels for reading text and a mid-dark viewport so a bright
  3D background does not flood the eye.

### Text size

- **Normal readers have a "critical print size"**: below it, reading slows sharply; above it,
  bigger text does not help much. Low-vision readers' critical size is several times larger
  (Legge et al. 1985: intact central fields peaked at small to intermediate sizes, central field
  loss at 12′–24′). (The common rule of thumb that critical print size for normal adults is
  about 0.2° is from Legge's later work; unverified, the PDF could not be fetched.)
- **What SolidCraft shows at 100% scale** (computed for this note; assumes a 24-inch 1920×1080
  monitor, pixel pitch 0.277 mm, x-height 0.52 of the font size, which is typical of sans fonts
  like IBM Plex Sans but unverified for Plex specifically):

  | UI scale | x-height of 12-point body text at 60 cm (desk) | at 100 cm (standing at a machine) |
  | --- | --- | --- |
  | 100% | 9.8′ | 5.9′ |
  | 125% | 12.3′ | 7.4′ |
  | 150% | 14.7′ | 8.8′ |
  | 200% | 19.6′ | 11.8′ |
  | 250% (current maximum) | 24.6′ | 14.7′ |

  For comparison, a 20/20 letter is 5′ tall in total (standard optometry definition). The
  smallest tokens in `palettes.rs` (`font_triad` 8, `font_badge` 9.5, `font_micro` 10) are two
  thirds of body text, so at 100% on a shop-floor display they are close to the acuity limit of
  a well-corrected young eye.
- **W3C low-vision user need:** "Users can change the text size (font size) of all text, without
  zooming the entire interface" (W3C *Accessibility Requirements for People with Low Vision*,
  First Public Working Draft 2016). Zooming everything costs screen space for the 3D view, which
  matters a lot in CAD; enlarging only text does not.

### Contrast

- **WCAG 2.2** (W3C Recommendation; levels from the specification): 1.4.3 Contrast (Minimum),
  AA, 4.5:1 for text and 3:1 for large text; 1.4.6 Contrast (Enhanced), AAA, 7:1 and 4.5:1;
  1.4.11 Non-text Contrast, AA, 3:1 for the boundaries and states of controls and for graphics
  needed to understand content; 1.4.1 Use of Color, A, colour is never the only way information
  is shown; 1.4.4 Resize Text, AA, 200%; 1.4.12 Text Spacing; 2.4.13 Focus Appearance (AAA); 2.5.8
  Target Size (Minimum), AA, 24×24 CSS px; 2.5.5 Target Size (Enhanced), AAA, 44×44 px. WCAG is
  written for web content, but it is the reference for desktop software through EN 301 549 in
  Europe and Section 508 in the US (unverified for the exact mapping).
- **IBM's accessibility toolkit** applies these to components: 4.5:1 for labels; 3:1 for input
  fields, icons and data-visualisation marks; focus indicators at least 2 px thick and 3:1 against
  adjacent colours.
- **APCA** (Advanced Perceptual Contrast Algorithm, Myndex) is the candidate contrast method for
  WCAG 3. It accounts for text size and weight and for light-on-dark versus dark-on-light, which
  WCAG 2's ratio handles poorly. Its guidance: Lc 90 preferred for body text, Lc 75 minimum for
  body text, Lc 60 for larger text, Lc 45 for large headings, Lc 30 the floor for any text, Lc 15
  the invisibility point for some people. **It is not part of WCAG 2 and is still in development**;
  its licence was not confirmed in this session, so SolidCraft should not adopt its code without
  checking. The ideas (contrast should depend on size and weight; dark mode needs its own check)
  are still useful.
- **For aging eyes, 4.5:1 is a floor, not a target (opinion).** It was set for typical 20/40
  vision. With one third of the retinal light and doubled scatter, a 60–70-year-old sees much
  less of that ratio. The "Clear" presets below aim for 7:1 for all text.

### Colour

- Never colour alone (WCAG 1.4.1; W3C low-vision need "Color is not the only visual means of
  conveying information").
- With age the **blue–yellow** axis weakens most (Wuerger 2013); congenitally, **red–green** is the
  common deficiency. A palette that must work for both should separate meanings by **lightness**
  first, then by shape or pattern, and only then by hue. Blue/orange pairs are the usual advice for
  red–green deficiency, but blue and yellow both lose strength with lens yellowing, so the blue in
  such a pair should be a light or saturated blue against a light-enough background, not navy on
  dark grey (opinion based on the above).
- Saturation also drops for older viewers ("reds look more like pinks", Lighting Research Center).
  Status colours need strong saturation and a lightness difference from their background.

### Fonts

- **Distinguishable letterforms** matter more than style: 1/l/I, 0/O, 5/S, 8/B, rn/m. For CAD/CAM
  the digits are most important, because dimensions, feeds and offsets are read as numbers.
- **Atkinson Hyperlegible** was designed by the Braille Institute of America with Applied Design
  Works (2019) for low-vision readers, with exaggerated, deliberately different letterforms; it is
  under the SIL Open Font License, and **Atkinson Hyperlegible Next** (2025) adds variable weights
  and a monospaced version (Wikipedia, Google Fonts repository). SolidCraft already bundles IBM
  Plex (also OFL) and lets users choose any installed font via `ui_font`, so a user can install
  Atkinson today; bundling it would be a separate decision with an ATTRIBUTION.md row.
- **Weight helps** low-contrast vision: thin and light weights disappear first (fine detail,
  above). A preset can move body text from regular (400) to medium (500) without changing
  sizes; Plex is variable and already ships 400–700.

### Brightness and the room

- W3C low-vision needs include "Users can set the overall brightness of a display" and, for light
  sensitivity, a darker background.
- Shop lighting is bright on purpose: EN 12464-1 values summarised by DIALux list 300 lx for rough
  and medium-accuracy machining (tolerance above 0.1 mm), 500 lx for precision machining (below
  0.1 mm), 750 lx for marking, checking and measuring, and 1000 lx for toolmaking and precision
  assembly (DIALux knowledge base; described there as service illuminance). A screen next to a
  machine competes with that light and its reflections.

### Targets, pointers and focus

- WCAG 2.5.8 asks 24×24 px minimum; 2.5.5 asks 44×44 px. SolidCraft's `control_height` is 28 at
  100%. For gloved or shaky hands, industrial push buttons settle at 22.5 mm, "easy to operate –
  even with gloved hands" (EAO HMI design guide); touch sizes for gloves were not found in a
  primary source this session.
- W3C low-vision needs include larger mouse pointers and text cursors, with pointers that do not
  hide tooltips.

## The shop floor is different from the office

(Opinion, from the sources above and general shop practice; **[TODO Jason]** confirm or correct.)

- **Distance:** a CAM station or a control-side viewer is often read standing, at arm's length or
  farther (about 0.7–1.2 m). The x-height table shows text is roughly half the angular size there.
- **Glasses:** reading glasses blur at 1 m; progressives make the user tilt their head back;
  safety glasses may be scratched, tinted or fogged by coolant mist. A "Shop Floor" preset should
  assume the user's glasses are not ideal for the screen.
- **Light and glare:** 500–1000 lx overhead plus windows and reflective surfaces. Matte screens,
  a light theme with off-white panels, or a mid-grey viewport all reduce veiling glare.
- **Hands:** gloves, oily fingers and touchscreens on newer controls. Large targets, no hover-only
  information, and confirmations that do not depend on precise clicks.
- **Glance reading:** the operator looks up from the machine for a second. Key numbers (current
  tool, stock-to-leave, feed override, confidence setting) should be readable at a glance: big,
  high-contrast, in a fixed place.

## What SolidCraft already has

From the repository at the time of writing (October 2026):

| Feature | Where | Comment |
| --- | --- | --- |
| Light, Dark, System and 20 built-in themes, including **High Contrast** | Interface › Theme; `assets/styles/ui/themes/` | High Contrast is a dark theme (black dialogs, white text, cyan accent). There is no high-contrast **light** theme, which the polarity research favours for most older users with clear eyes. |
| **Viewport** Light/Dark independent of the theme | Interface › Theme › Viewport | Good for glare control (see polarity above). |
| **Colour vision** modes Protan, Deutan, Tritan; failing faces in Draft and Accessibility Analysis also striped | Graphics › Colour vision (#778) | Already follows "never colour alone" for analysis faces. Tritan is relevant to age-related blue loss. |
| **UI scale** 50–250%, quick buttons 75–200%, Ctrl/Cmd +/- and 0 | Interface › Scale | Scales everything, including panels, so the viewport shrinks. |
| **Interface font** override (`ui_font`) | Interface | Lets users pick Atkinson Hyperlegible or another legible font today. |
| Contrast checks in code (`theme::contrast_ratio`, 4.5:1 on some token pairs) | `ui_kit`, theme audit (#5007) | Applied to workspace ink and some status colours, not yet to every text/background pair or to non-text 3:1. |
| Font sizes come only from tokens (`font_literals_tests`) | `docs/fonts.md` | **This makes a text-size setting easy**: one multiplier on the tokens changes all text. |

**Contrast of some default light/dark tokens** (computed for this note with the WCAG 2 formula from
`assets/styles/ui/palettes.rs`; the sketch colours are measured against white as an
approximation of a light viewport):

| Pair | Ratio | WCAG 2.2 |
| --- | --- | --- |
| Light: `text` on `panel` | 13.9:1 | passes AAA |
| Light: `text_dim` on `panel` | 4.53:1 | just passes AA, fails AAA |
| Light: `text_dim` on `toolbar` | 4.38:1 | **fails AA (4.5:1)** |
| Light: `form_dim` on dialog body | 7.27:1 | passes AAA |
| Light: `accent` on `panel` (as text) | 4.24:1 | fails AA for small text; passes 3:1 for graphics |
| Light: `border` on `panel` | 1.43:1 | **fails 3:1 non-text contrast** if the border is what shows where a control is |
| Light: `sketch_construction` orange on white | 2.77:1 | **fails 3:1** for graphics |
| Light: `sketch_locked` green on white | 4.36:1 | passes 3:1 for graphics |
| Light: `sketch_point_free` blue on white | 6.27:1 | passes |
| Dark: `text` on `panel` | 10.9:1 | passes AAA |
| Dark: `text_dim` on `panel` | 5.07:1 | passes AA, fails AAA |

These are spot checks, not an audit, and whether a border "identifies a control" depends on the
control (many controls also have a fill). They show where an aging-eyes preset would need to
change colours, and that a test that checks **every** text/background token pair in **every**
theme would be a natural extension of the existing contrast checks.

**Not found** in a search of `crates/ui-egui/src` (may exist under another name): a separate text
size setting, a viewport line-weight or edge-width setting, a sketch point or handle size setting,
a mouse pointer size setting, and a "reduce motion" setting.

## Recommendations for SolidCraft

All of this is opinion built on the research above. Everything below would be ordinary
preferences reached through `ui.prefs`, so scripts and the control channel can set them like any
other preference, and presets are only bundles of those preferences.

### 1. A Vision section with presets

A **Vision** section (in Interface, or its own Preferences page) with one choice at the top and
the individual settings below it. Choosing a preset sets the values; changing any value afterwards
shows "Custom". First run could ask once: "How do you like your screen?" with the five previews.

| Setting | Standard (today) | **Clear** (aging eyes, lit room) | **Clear Dark** (cataract, glare-sensitive, dim room) | **Shop Floor** (1 m away, standing) | **Large & Bold** (low vision) |
| --- | --- | --- | --- | --- | --- |
| Theme | Light/Dark/System | new **High Contrast Light** (off-white panels, near-black text) | High Contrast (dark) with near-white text | High Contrast Light | user's choice of either high-contrast theme |
| Viewport | Follow theme | mid-light grey, not white | mid-dark grey, not black | mid-grey | follow theme |
| UI scale | 100% | 125% | 125% | 150% | 200% |
| **Text size** (new, multiplies text tokens only) | 100% | 115% | 115% | 130% | 130% |
| **Smallest text** (new floor in points, after scaling) | none | 12 | 12 | 14 | 16 |
| Text weight | Regular | Medium | Medium | Medium | Semibold |
| Text contrast target | 4.5:1 | 7:1 | 7:1 | 7:1 | 7:1 |
| Control borders | theme | ≥ 3:1, 1.5 pt | ≥ 3:1, 1.5 pt | ≥ 3:1, 2 pt | ≥ 3:1, 2 pt |
| **Edge and sketch line weight** (new) | 1× | 1.5× | 1.5× | 2× | 2× |
| **Point, handle and glyph size** (new) | 1× | 1.25× | 1.25× | 1.5× | 2× |
| Focus ring | theme | 2 pt, ≥ 3:1 | 2 pt, ≥ 3:1 | 3 pt | 3 pt |
| Dim/secondary text | theme's dim | darker dim (≥ 7:1) | brighter dim (≥ 7:1) | no dim text | no dim text |
| Messages | status chip, few seconds | chip + stays until dismissed for errors | same | large banner near the work, stays | same as Shop Floor |
| Colour vision | Off | Tritan offered (age-related blue loss) | Tritan offered | Off unless set | Off unless set |
| Animation | on | on | on | reduced | reduced |

Numbers are starting points for testing with real users, not results.

### 2. Size by distance

Add an optional "How far are you from the screen?" (40 cm to 2 m) and use the monitor's physical
size (which the operating system usually reports) to **suggest** a UI scale that gives body text a
chosen visual size, for example 12′ of x-height for Clear and 20′ for Large & Bold. The formula is
plain geometry:

`scale = target_arcmin × distance_mm × π / 10800 / (x_height_ratio × font_body_px × pixel_pitch_mm)`

The user still sees and can change the result. This replaces guessing between 125% and 150%.

### 3. Text size separate from UI scale

Because every font size already comes from `theme::Tokens` (enforced by `font_literals_tests`),
a **Text size** multiplier on the font tokens alone, with layout that grows to fit, would meet the
W3C low-vision need and keep more of the screen for the 3D view. A **minimum text size** clamps
the small tokens (`font_triad`, `font_badge`, `font_micro`) so no text gets smaller than the floor.

### 4. A contrast floor every theme must pass

Extend the existing contrast tests from selected pairs to every text-on-background token pair and
every control boundary, in every shipped theme: 4.5:1 for text and 3:1 for non-text in all themes,
7:1 for text in High Contrast and High Contrast Light. Imported themes get a warning, not a
rejection. The spot checks above show the light theme's `text_dim` on `toolbar`, control borders
and construction orange as first candidates.

### 5. The 3D view

- **Line weight and silhouettes:** older eyes lose fine detail first, so 1-pixel edges on a light
  background fade. A line-weight multiplier and an option for a stronger silhouette edge help more
  than colour changes.
- **Points, handles, constraint glyphs and dimension text:** separate size multipliers, and
  dimension text following the Text size setting.
- **Selection and pre-selection:** show them with an outline or thicker edge as well as a tint, so
  they survive reduced contrast and colour vision modes.
- **Sketch states** (fully constrained, free, construction, projected, lost): add a second cue
  (line style, point shape) to each colour, as the analysis stripes already do.
- **Background:** avoid pure white and pure black; a mid-light or mid-dark grey keeps edges visible
  while limiting glare and halation.

### 6. Messages where they are seen

Errors and warnings that matter should appear near the pointer or the active dialog and stay until
dismissed, in addition to the status chip, so users with peripheral field loss or who looked away
do not miss them.

### 7. Developer tooling: an "aging eye" preview

The colour vision modes already simulate red, green and blue deficiencies for checking designs. A
developer-only preview that **blurs slightly, lowers contrast, adds a veil and yellows the image**
(roughly a 70-year-old with early cataract) would let contributors check a screen in a second, the
same way they check dark mode. UI screenshots in PRs could include it for new panels.

## Recommendations for the CAM interface

CAM puts safety-relevant numbers in front of people who are, on average, older and often standing
at the machine.

1. **The conservative-to-aggressive confidence slider** (cutting physics and workholding notes):
   - Words, not just colour: positions labelled, for example, "Safe start · Conservative ·
     Balanced · Productive · Aggressive", and the current word shown large next to the slider.
   - A number as well (for example the safety factor in use).
   - If coloured green → amber → red, each band also changes the **pattern or icon** (for example
     plain → hatched → cross-hatched, or ✓ → ! → ⚠) so it survives red–green deficiency and the
     age-related loss of saturation. Lightness should also step clearly along the slider.
   - A thumb at least 44 px at the chosen scale, and keyboard steps.
2. **Status colours** (OK / check / problem) everywhere in CAM: each with a shape icon and a word,
   distinct in lightness, tested in all three colour vision modes and in the aging-eye preview.
   Avoid red-vs-green as the only difference (deutan is the commonest congenital deficiency) and
   avoid blue-vs-purple or yellow-vs-white (age-related tritan loss).
3. **Glance panel for the shop floor:** current operation, tool number and description, stock to
   leave, confidence setting and warnings in a large, fixed panel that follows the Shop Floor
   preset, readable from 1–2 m.
4. **Setup sheets and the part traveller** (in-process inspection note): printed and on-screen
   versions follow the chosen text size; tool numbers and critical dimensions in a
   distinguishable monospaced font (slashed or dotted zero), never smaller than 12 pt on paper.
5. **Simulation colours** (stock, part, gouge, excess, fixture collision): a palette chosen for
   lightness differences, with the colour vision modes and stripes applied the same way as Draft
   analysis.
6. **No hover-only information** for anything a gloved or touch user needs.

## Design checklist for contributors

A short list that could go into `docs/ui-kit.md` (opinion):

- [ ] Text uses tokens and still fits at Text size 130% and UI scale 200%.
- [ ] No text below the minimum text size at any scale.
- [ ] Text ≥ 4.5:1 on its background in every theme (≥ 7:1 in the high-contrast themes).
- [ ] Control boundaries, icons and graphics needed to understand them ≥ 3:1.
- [ ] Nothing is shown by colour alone; check Protan, Deutan and Tritan.
- [ ] No blue-on-dark or yellow-on-white for meaning.
- [ ] Lines that carry meaning respect the line-weight multiplier.
- [ ] Errors stay until dismissed and appear near the work.
- [ ] Targets ≥ 24 px at 100% (≥ 44 px for primary CAM controls).
- [ ] Screenshot checked in light, dark, High Contrast and (once it exists) the aging-eye preview.

## Export control, sanctions and government use

*Added 2026-10-29 from the [government use research](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-qk5riz/research/gov-research-nsnlookupjn-government-use.md). Opinion and policy proposals, not legal advice.*

**Our three rules:**

- **No one goes to jail.** Our team has no in-house counsel. So wherever a rule is unclear we take the most conservative reading and refuse rather than warn. Questions that need an outside export-control lawyer are marked as such, and free official help (BIS, DDTC, OFAC) is the first call.
- **Help users comply.** SolidCraft stops and explains in plain language, using rule packs for each jurisdiction that the administrator picks. It flags conflicts between countries' laws, never resolves them, and never gives legal advice.
- **Prevention first.** Nothing in this file that is hosted, shared, paid or networked ships until its compliance gates are in place and checked: region blocks, screening, ITAR flag handling, government-mode behaviour, and an outside lawyer's sign-off for any hosted or paid service.

**What in this file crosses a border:** local display and accessibility settings; accessibility also matters for US government use (Section 508).

- **No hosted service is proposed here.** If one is added later, it blocks comprehensively sanctioned regions, plus Russia and Belarus, and screens accounts and payers against the US Consolidated Screening List and the UK, EU and UN lists.
- **No support there.** No support, consulting or custom work for users in sanctioned regions or for listed parties.
- **Government mode.** At the Controlled level (CUI and allied equivalents) online parts are off or point to the organisation's own mirror; at the Isolated level they arrive only as signed packs on media; a design flagged Export controlled (ITAR/EAR or national equivalent) never leaves through them.
- **Only generic data is shared.** Nothing derived from a controlled design (geometry, toolpaths, part or drawing numbers, test or inspection results) goes into a shared library, report or issue. Tool data stays shareable through the field allow-list in the high-security research.
- **Code stays public.** Every build the project ships is public, which keeps it "published" software outside the EAR. The rules are tracked by the standing export and sanctions watch.
- **Checked for controlled data (2026-10-29):** this file contains no controlled technical data. Its examples are public sources, generic values or made-up stand-ins, and the author's experience is stated in general terms.

## Open questions

1. Should the presets live in Interface or get their own **Vision** (or **Accessibility**)
   Preferences page?
2. Is a **High Contrast Light** theme wanted as a built-in? The research favours it for most older
   users with clear eyes.
3. Should SolidCraft bundle **Atkinson Hyperlegible Next** as an optional interface font, or only
   document that users can install and choose it?
4. Should the first-run screen ask about vision, or should it stay in Preferences only?
5. Who could help test with real older users and machinists? Shops, community colleges and
   low-vision organisations are possible partners (**[TODO Jason]**: anyone you know).

## References

Medical and vision science

- Fricke TR, Tahhan N, Resnikoff S, Papas E, Burnett A, Ho SM, Naidoo KS. *Global Prevalence of
  Presbyopia and Vision Impairment from Uncorrected Presbyopia: Systematic Review, Meta-analysis,
  and Modelling.* Ophthalmology 125(10):1492–1499 (2018). https://pubmed.ncbi.nlm.nih.gov/29753495/
  (summary via https://research.sightsavers.org/?p=28154)
- Lighting Research Center, Rensselaer. *Lighting for Older Adults: The Aging Eye.*
  https://www.lrc.rpi.edu/programs/lightHealth/AARP/healthcare/lightingOlderAdults/agingEye.asp
- Shao S, Li Y, Meso AI, Holliman NS. *Does empirical evidence from healthy aging studies predict a
  practical difference between visualizations for different age groups?* EG UK CGVC 2024.
  https://arxiv.org/abs/2407.21767 (models Owsley C, Sekuler R, Siemsen D, *Contrast sensitivity
  throughout adulthood*, Vision Research 1983)
- Wuerger S. *Colour Constancy Across the Life Span: Evidence for Compensatory Mechanisms.*
  PLoS ONE 8(5):e63921 (2013). https://pmc.ncbi.nlm.nih.gov/articles/PMC3648508/
- Łabuz G, Reus NJ, van den Berg TJTP. *Ocular straylight in the normal pseudophakic eye.* (2015),
  citing van den Berg TJTP et al. (2007) for phakic eyes.
  https://repub.eur.nl/pub/102424/102424_Chapter_3-Ocular_straylight_in_the_normal_pseudoph.pdf
- Legge GE, Rubin GS, Pelli DG, Schleske MM. *Psychophysics of reading: II. Low vision.* Vision
  Research 25(2):253–265 (1985).
  https://legge.dl8.umn.edu/sites/legge.psych.umn.edu/files/2020-08/Psychophysics%20of%20Reading%20II%20Low%20Vision%20Legge%20Rubin%20Pelli%20Schleske%201985.pdf
- Prevent Blindness. *By age 80 more than half of Americans will have cataracts* (2024).
  https://preventblindness.org/document/by-age-80-more-than-half-of-americans-will-have-cataracts

Display polarity

- Piepenbrock C, Mayr S, Mund I, Buchner A. *Positive display polarity is advantageous for both
  younger and older adults.* Ergonomics 56(7) (2013) (title only seen in this session).
- Buchner A et al. *The advantage of positive text-background polarity is due to high display
  luminance.* https://madoc.bib.uni-mannheim.de/26118/ (not fetched)
- Sethi T, Ziat M. *Dark mode vogue: Do light-on-dark displays have measurable benefits to users?*
  Ergonomics 66(12):1814–1828 (2023) (as summarised in
  https://www.alexandria.unisg.ch/server/api/core/bitstreams/62e4b3c1-0521-4d3a-bbad-e6e5eb81795e/content)

Standards and guidance

- W3C. *Web Content Accessibility Guidelines (WCAG) 2.2*, Understanding documents.
  https://www.w3.org/WAI/WCAG22/Understanding/
- W3C. *Accessibility Requirements for People with Low Vision*, First Public Working Draft (2016).
  https://www.w3.org/TR/low-vision-needs/
- IBM Accessibility. *Visual design.* https://www.ibm.com/able/toolkit/design/visual
- Myndex. *APCA introduction.* https://github.com/Myndex/apca-introduction
- ISO 9241-303:2011, *Requirements for electronic visual displays* (contents page only:
  luminance contrast, image polarity, character height).
  https://webstore.ansi.org/preview-pages/ISO/preview_ISO+9241-303-2011.pdf
- EN 12464-1 values as summarised by DIALux.
  https://dialux4.support-en.dial.de/support/solutions/articles/9000080318-standard-values-for-the-metalworking-industries
- Silktide. *WCAG 2.5.8 Target Size (Minimum).*
  https://silktide.com/accessibility-guide/the-wcag-standard/2-5/input-modalities/2-5-8-target-size-minimum/
- EAO. *HMI System Design* (technical article). https://www.mouser.lt/pdfDocs/EAO_TA_HMI_System_Design_EN-2.pdf

Fonts and workforce

- *Atkinson Hyperlegible.* https://en.wikipedia.org/wiki/Atkinson_Hyperlegible ;
  https://github.com/googlefonts/atkinson-hyperlegible
- Data USA. *Machinists* (2024 data). https://la.datausa.io/profile/soc/514041

SolidCraft

- `assets/styles/ui/palettes.rs`, `assets/styles/ui/themes/high-contrast.json`,
  `crates/ui-egui/src/prefs/interface.rs`, `docs/fonts.md`, `docs/theme-colour-audit.md`,
  `docs/book/src/guide/ui-tour.md` (Colour themes; Colour vision), `docs/control-protocol.md`
  (`ui.prefs`).
- Related issues: #778 (colour vision modes), #2030 (ribbon redesign dark mode and colour-vision
  pass), #5007 (theme colour audit), #1621 (CAM research).
