### Research: a shared machine library

**In one line:** give every machine model a physics-based library entry, so CAM treats the machine's real limits as hard limits at every cut step.

**What it covers**
- **Spindle:** torque and power against speed, per gear range, S1 and S6 ratings.
- **Axes:** thrust, feed, acceleration and jerk per axis, and rotary brake torque.
- **Stiffness:** tool-point stiffness and modes, so the machine counts in the deflection chain.
- **Kinematics, warm-up and signals:** the machine's kinematics, its warm-up routine, published control signals and a 3D model link.
- **The shop's own machine:** a measured instance on top of the model entry, optionally locked to the materials it cuts.
- **Realistic feeds:** predicting the feed the control really runs, not the programmed one.
- **Export control:** the shared library blocks sanctioned regions and screens contributors, and only generic machine data is shared.

**It completes my tool library, cutting physics, workholding and stock integrity research** and is the base the CAM Physics System builds on.

**Research file:** attached (MIT OR Apache-2.0), also on my fork: [cam-research-nsnlookupjn-machine-library.md](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-jfhnp0/research/cam-research-nsnlookupjn-machine-library.md)

Feedback welcome, especially on the kinematics format (STEP AP242 or ISO 14649-201).
