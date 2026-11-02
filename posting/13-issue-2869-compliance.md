### Research: what regulated shops need from versioned designs

**In one line:** most of what ISO 9001, AS9100, ISO 13485, IATF 16949 and ITAR shops need from design-to-machine records builds directly on the versioning this RFC proposes.

**What it covers**
- **A content hash per released revision,** recorded on every derived file (drawing, analysis, CAM program, inspection plan), so anything made from an old revision is flagged stale.
- **Release states and signed approvals,** with who approved what and when.
- **Change impact:** which drawings, programs and inspection plans a change touches.
- **Corrective-action links** from a nonconformance back to the revision and the change that fixed it.
- **Integrations:** QMS, PLM and ERP systems, supplier records and a future collaboration server.
- **A US defence profile** that uses the government mode's country profiles rather than hard-coding one country.
- **Export control:** hosted and collaboration parts block sanctioned regions and screen people, and controlled designs follow the ITAR flag in my high-security research.

**All of it is off by default.** A hobbyist sees nothing new; a regulated shop switches on only what its standard asks for.

**Research file:** attached (MIT OR Apache-2.0; research, not legal advice; my defence and aerospace background is in the file), also on my fork: [compliance-research-nsnlookupjn-itar-quality.md](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-jfhnp0/research/compliance-research-nsnlookupjn-itar-quality.md)

Feedback welcome, especially on whether the content hash should be part of the document server's own revision record.
