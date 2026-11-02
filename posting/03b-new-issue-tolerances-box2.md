**What it covers**
- **Tolerances on dimensions and parameters:** ±, +a/−b, limits, ISO 286 fit codes (H7, g6), and a general tolerance per design or body from a process profile (ISO 2768 f/m/c/v and similar).
- **Thread classes on Thread and Hole:** 6H/6g, 2B/2A (`docs/thread-catalogue.md` lists classes as out of scope today).
- **GD&T with datums:** ISO 1101 or ASME Y14.5, on faces and features, stored by persistent face names.
- **Per face:** surface texture (Ra/Rz), finish spec (for example anodize type and class, masking) and hardness.
- **Importance per characteristic:** critical, key, standard, minor, don't care, so effort goes where it matters.
- **Permitted deviations:** maximum internal corner radius, edge breaks per ISO 13715, drill points allowed, per part and per feature.
- **A "release for production" check:** lists what is missing, only when the user asks.

**It must stay optional.** A design with none of this works exactly as today: no new required fields, no new warnings, **no default changes**. Novices never see it unless they want it; pros get every control.

**Used by:** Drawings (#860, #6782, #7257) for dimensions and annotations, STEP AP242 semantic PMI export, CAM (#1153, #1621) for target sizes and toolpath accuracy, and in-process inspection. Tolerances and inspection results of controlled parts are controlled data, so they follow the same export rules as the geometry.

**Research file:** attached (MIT OR Apache-2.0), section "Engineering intent in the model", also on my fork: [cam-research-nsnlookupjn-in-process-inspection.md](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-jfhnp0/research/cam-research-nsnlookupjn-in-process-inspection.md)

Feedback welcome, especially on whether this should be a design-side milestone of its own and which pending format file it should use.
