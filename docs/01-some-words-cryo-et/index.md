﻿---
layout: default
title: "Some Words about Cryo-Electron Tomography"
nav_order: 2
parent: "Welcome to Tomo101"
---

# Some Words about Cryo-Electron Tomography

Cryo-electron tomography (cryo-ET) is an imaging method used to obtain three-dimensional (3D) reconstructions of biological specimens in a close-to-native, vitrified state. Unlike single-particle analysis (SPA), which aims to determine the structure of repeated, purified macromolecular complexes by averaging thousands or even millions of identical particles, cryo-ET focuses on imaging large and complex biological samples. These samples can be entire cells, subcellular organelles, or thick cellular sections (lamellae) produced by focused ion beam (FIB) milling. The main difference between SPA and cryo-ET, is that the data produced by cryo-ET is much more information-rich. You will see much more than what you are targeting, or sometimes it will be difficult to find what you are targeting. Additionally, you mostly work in 3D (even if the raw data are 2D projections of your sample). This means that you want to look at your tomograms to understand what’s in them (all of them!) contrary to look at SPA micrographs where you have the same things thousands of time.


<pre>
+------------------+           +--------------+
|     CRYO-ET      |           |     SPA      |
+------------------+           +--------------+
        ||                            ||
        ||                            ||
       \||/                          \||/
 ┌────────────┐                 ┌────────────┐
 |   *   *    |                 |   -    -   |
 |     ^      |                 |     _      |
 |    \_/     |                 |    ---     |
 └────────────┘                 └────────────┘
      /  \                            /  \
     /    \                          /    \
"Wow, so much detail!"        "Meh, these all look the same..."
</pre>






---
## Sample Types

Cryo-electron tomography (cryo-ET) requires thin samples (around 100–200 nm) so the electron beam can pass through (we are using transmission electron microscopes (**TEM**)). For whole cells, a focused ion beam (FIB) is used to mill thin lamellae while preserving native structure. However, if you’re studying purified organelles (e.g., isolated mitochondria), you can sometimes skip or reduce milling if they’re already thin enough. People are also now going towards more complex samples (e.g small multicellular organisms, or biopsies) which involve high-pressure freezing (**HPF**) of the sample instead of the more classical plunge-freezing.

(image idea for illustration: slice of a cell/tissue and organelle)


A critical philosophical difference is that cryo-ET aims to observe macromolecules in their natural cellular context. Meanwhile, SPA requires extensive purification steps, removing the protein complexes from their native milieu but achieving higher resolution in exchange for losing cellular context.

### FIB-Milled Cellular Lamellae
- **Preserves In Situ Context:** You see organelles and macromolecules in their native environment (cell membranes, cytoskeleton, neighboring structures).
- **Complex and Heterogeneous:** More challenging to prepare and interpret because you capture the full cellular complexity.

### Purified sample (e.g organelles, large viruses, etc)
- **Simpler Prep:** Often no milling needed if the organelle is inherently small or can be fractionated to the right thickness.
- **Less Context:** You lose interactions with other cellular components—what you gain in simplicity, you lose in native spatial relationships.

In short, cellular lamellae capture the true cellular environment at the expense of more complex data collection and processing, whereas purified organelles simplify tomography but lose the broader context.

---
## Cryo-ET Data Collection Workflow


1. **Sample preparation**  
   FIB-milled plunge-frozen cells, HPFed samples, or purified macromolecules of interest.

2. **Sample Mounting**  
   The sample is transferred into the cryo-TEM (kept below –180°C).

3. **Tilt Series Acquisition**  
   The stage is tilted incrementally (e.g., usually ±60° or ±40°) with 2–3° increments, collecting 2D projections at each tilt angle.

4. **Alignment**  
   Projections are aligned to correct for stage shifts, beam-induced motion, etc.

5. **Tomogram Reconstruction**  
   A 3D volume is reconstructed from aligned tilt series using weighted back-projection or SIRT.

6. **Post-processing & Further Analyses**  
   You can segment specific structures in the tomogram or perform sub-tomogram averaging if you have repeated complexes.


---
## Challenges in Cryo-ET

## **Dose and Radiation Damage**
Electron radiation damages biological samples. The total electron dose must be strictly limited, leading to a lower signal-to-noise ratio (SNR).

## **Missing Wedge Artifact**
Due to incomplete tilt sampling (typically ±60°), 3D reconstructions contain anisotropic distortions, known as the **missing wedge** effect.

## **Beam-Induced Motion**
Even small movements in the sample can degrade resolution, especially at higher tilt angles.



---
## Differences Between Cryo-ET and SPA


| Aspect                 | Cryo-ET                                                                                   | Single-Particle Analysis (SPA)                                      |
|------------------------|-------------------------------------------------------------------------------------------|----------------------------------------------------------------------|
| **Sample Type**        | Unique cellular or subcellular specimens (e.g., FIB-milled lamellae or intact viruses/cells) | Purified, isolated macromolecular complexes or particles             |
| **3D Reconstruction**  | 3D volume from multiple tilt images of the *same* specimen                                | 3D reconstruction from averaging many 2D projections of *multiple copies* of a particle in different orientations |
| **Resolution**         | Typically lower (5–10 nm for cellular tomography, though can reach sub-nm with sub-tomogram averaging) | Can reach near-atomic resolutions (2–4 Å) for well-behaved particles and large data sets |
| **Structural Context** | *In situ*: preserves spatial relationships within the cell                                | *In vitro*: particles are studied in isolation, removed from their native environment |
| **Data Requirement**   | Single tilt series per region of interest, capturing unique structures                    | Thousands to millions of particle images needed for high-resolution averaging |
| **Throughput**         | Lower – time-consuming data collection per specimen                                      | Potentially higher – automated data collection and well-optimized pipelines |


---
## Sub-Tomogram Averaging

Cryo-ET can achieve higher resolution through **sub-tomogram averaging (STA)**, where repeated structures (e.g., ribosomes, viral spikes) are extracted from tomograms and aligned.

## **Steps in STA**
1. **Particle Picking** - Extract candidate regions from tomograms.
2. **Alignment & Classification** - Align similar particles and classify.
3. **Averaging** - Enhance resolution by averaging.

