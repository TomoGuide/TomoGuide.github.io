---
layout: default
title: "Preprocessing in RELION5"
parent: "Tutorial"
nav_order: 3
---

# Preprocessing and Tomogram Reconstruction in RELION5

Below is a basic guide on how to import frames and reconstruct tomograms directly in RELION5.

---

## Import Frames

1. Prepare a directory named **rawdata** in your RELION working folder.
2. Copy (or make symlinks to) the .eer/.tiff frames, .mdoc files, and gain references into **rawdata**.
3. In RELION, use the **Import** job to point to these raw frames and the corresponding .mdoc files.

> **(Placeholder: screenshot of RELION import dialog)**  
> `![RELION Import](path/to/relion_import.png)`

- If you have multiple gain references (e.g., 6 TS use GainRef1, 27 TS use GainRef2), you might import them as separate 
  groups until you get to the STA step.

### Tilt-Axis & Defocus Handedness

RELION also wants to know:
- **Tilt Axis** sign: if your real axis is +95°, and your data came out mirrored, you might need to enter -95.  
- **Defocus Handedness**: if you get it wrong, high-resolution refinements can be mirrored. For this dataset, 
  “Invert defocus handedness” is `YES (-1)`.

> **Tip:** If you’re unsure, reconstruct a few tomograms first and visually confirm the orientation.

---

## Tomogram Reconstruction in RELION5

Similar to Scipion, you’ll do:
- **Motion Correction** for each tilt (e.g., RELION’s internal motioncorr or external motioncor2).  
- **Tilt-Series Alignment** (with RELION’s own alignment or AreTomo integration).  
- **CTF Estimation**  
- **Tomogram Assembly** and possible dose filtering.

Once your tomograms are reconstructed, you can further refine them, check for mirrored orientation, etc. 
At that point, you’d proceed to [STA in RELION5](/08-tutorial/04-sta-in-relion5/).

