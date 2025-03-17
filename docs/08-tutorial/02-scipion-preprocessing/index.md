---
layout: default
title: "Preprocessing in Scipion"
parent: "Tutorial"
nav_order: 2
---

# Preprocessing and Tomogram Reconstruction in Scipion

In this section, we will go through how to reconstruct your tomograms using Scipion, and at the end, how to export the 
information to perform STA in RELION5. If you want to do everything in RELION5, jump to the [RELION5 section](/08-tutorial/03-relion-preprocessing/).

Scipion is free, open-source software that acts as a wrapper for other programs, letting you organize your projects and 
have different tools interact seamlessly. It is especially handy for tomography since the field lacks a single solution 
to do all the steps (reconstruction and STA) while tracking your work.

> **(Placeholder for a screenshot of Scipion interface)**  
> `![Scipion Screenshot](path/to/scipion_screenshot.png)`

---

## Create a New Project

1. From the base Scipion interface, click **Create Project**, give it a name and location.  
2. Now you have an empty Scipion project.

---

## Import Frames

Press the **Add tomo - import tilt-series** (or press <kbd>Ctrl+F</kbd> and type “import tilt-series”).  
- Specify the directory containing the movies (.eer or .tiff) and the corresponding .mdoc files.  
- Override the tilt-axis angle if you know it is different from what the .mdoc says (very important to avoid mirrored tomograms).

> **Note:** With Titan G4 + Falcon4i in .eer, the tilt-axis might be ±95°. If in doubt, confirm with the facility manager or check AreTomo output.

---

## Motion Correction

This step aligns each frame of the movies and corrects beam-induced motion.  
- For low-dose tomography data, you might choose **full-frame** motion correction (instead of multi-patch).  
- If you plan to use cryoCare for denoising, tick the box **Split & sum odd/even frames**.  

> **(Placeholder: screenshot or snippet of Scipion motion correction settings)**  
> `![MotionCorr Settings](path/to/motioncorr_settings.png)`

For **.eer** data, define how many frames to group. You can use a formula to find the desired frames per group based on 
dose per tilt, total number of frames, etc.

---

## Cleaning the Stack (Tilt Curation)

After motion correction, remove “bad” tilts (overly drifted, blacked out, etc.). In many workflows, this is done manually, 
though software like AreTomo can do automated curation.

---

## TS Alignment (AreTomo or IMOD)

We’ll show how to do it automatically with **AreTomo**. AreTomo refines tilt axis angle, can do patch tracking, etc.  
- Start with an approximate lamella thickness (e.g., 1000 unbinned pixels ~190 nm)  
- Reconstruct quick bin8 tomograms to measure thickness and refine.  
- Double-check your tilt axis sign (+95 vs -95) to avoid mirrored (inverted-hand) tomograms.

> **(Placeholder: screenshot of AreTomo alignment interface)**

---

## CTF Estimation

AreTomo can do it automatically. Alternatively, **CTFFIND5** or other options exist. But we found AreTomo gave better results 
than older CTFFIND4.

---

## Tomogram Reconstruction

After alignment, you can do dose filtering, apply transforms, and reconstruct your tomograms.  
Then, for better human interpretability:

### Generating Denoised Tomograms

- Quick “denoising” using **dimifilter** or 
- More advanced approach with **cryoCare** (if you splitted odd/even frames before).

### Generating CTF-corrected Tomograms

Finally, you can apply your CTF estimations to correct lens distortions in the 3D volume.

---

## Exporting Data to RELION5

Scipion can export the final tilt-series alignment parameters, reconstructed tomograms, or pick lists to RELION5 for STA.  
Check the “Export to RELION” protocol within Scipion or do it manually by copying the transformations and .star files.

