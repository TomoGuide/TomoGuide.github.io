---
layout: default
title: "Cryo-ET Data Collection Workflow"
parent: "Welcome Page"
nav_order: 5
---

# Cryo-ET Data Collection Workflow

1. Sample preparation: FIB milled plunge frozen cells or HPFed sample or purified thing of interest.
1. Sample Mounting: The sample is transferred into the cryo TEM (kept below -180°C).
1. Tilt Series Acquisition: The stage is tilted incrementally (e.g., usually ±60° or ±40° with a 2 or 3° increment) and images are recorded at each tilt angle. This generates a tilt series of 2D projections.
1. Alignment: The projections are aligned to correct for stage shifts, beam-induced motion, and other distortions.
1. Tomogram Reconstruction: A 3D volume is reconstructed from the aligned tilt series using algorithms such as weighted back-projection or simultaneous iterative reconstruction (SIRT).
1. Post-processing and further analyses: Depending on the scientific question, the tomogram may be segmented to highlight specific cellular structures, or 1.  sub-tomogram averaging can be performed if repeated complexes exist within the volume.

