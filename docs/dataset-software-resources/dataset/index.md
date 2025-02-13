---
layout: default
title: "Dataset"
---

# Dataset

For this tutorial, we compiled a subset of 33 tomograms from the “Chlamy dataset” (EMPIAR-11830).  
(R. Kelley, et al, *Towards community-driven visual proteomics...* (2024), [https://doi.org/10.1101/2024.12.28.630444](https://doi.org/10.1101/2024.12.28.630444))

Even though reconstructed tomograms are available, this tutorial teaches you how to reconstruct them from scratch and do sub-tomogram averaging (STA).

**Download** (Linux):
wget -r -N -np -nH --cut-dirs=4 ftp://ftp.ebi.ac.uk/empiar/world_availability/EMPIAR-code/

Folder contains raw frames, .mdoc files, gain refs, thickness files, templates, and 33 tilt-series. For quick tests, use the 6 TS with GainRef1; for deeper analysis/classification, process all 33.

General info:
- Falcon4i + SelectrisX on Titan Krios
- Pixel size: 1.91 Å
- Tilt axis: 95°, but -95° for correct handedness
- Defocus handedness: -1 in RELION if starting fresh
- Dose: 3.5 e-/Å^2 per tilt
