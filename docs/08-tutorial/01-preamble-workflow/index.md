---
layout: default
title: "Preamble and General Workflow"
parent: "Tutorial"
nav_order: 1
---

# Preamble

For this tutorial, we compiled a subset of 33 tomograms from the "Chlamy dataset" (EMPIAR-11830) (R. Kelley, et al, *Towards 
community-driven visual proteomics with large-scale cryo-electron tomography of Chlamydomonas reinhardtii.* bioRxiv 
[Preprint] (2024). [https://doi.org/10.1101/2024.12.28.630444](https://doi.org/10.1101/2024.12.28.630444))

Even if already reconstructed tomograms are available for this dataset (originally reconstructed using TOMOMAN and 
automatically aligned with AreTomo), this tutorial aims at teaching you how to reconstruct tomograms from scratch and get 
into subtomogram averaging (STA). We believe this subset can also be of interest for people benchmarking different software.

In this tutorial, software that you will need are:
- **Scipion** (link)
- **RELION5** (link)
- **IMOD** (link)
- **pytom-match-pick** (link)

We like to start our tomogram preprocessing and reconstruction in Scipion, but we will also describe how to do the 
complete RELION5 pipeline. We recommend that you also download ChimeraX (link) and the ArtiaX (link) plug-in for 
particle map-backs.

Start by visiting [Datasets, Software, and Resources](/07-dataset-software-resources/) to ensure you download all the 
necessary software as well as the dataset.

### Dataset Contents
The downloaded dataset should contain:
- The raw frames
- The .mdoc for each tilt series (6 + 27)
- The gain references (2)
- Two text files with thickness measurements for automated AreTomo TS alignment
- Templates and masks for Template Matching

The 33 tilt-series (TS) can be divided into two groups: 6 associated with **GainRef1** and 27 with **GainRef2**.  
- If you’re just interested in learning the basics of tomogram reconstruction and STA, we recommend using the 6 TS 
  associated with **GainRef1** as processing will be faster.
- If you also want to try to push resolution or do classification, process the entire set of TS.

---

# General Workflow

Cryo-ET data processing can be summarized by this simple chart. If you’re familiar with single-particle data processing, 
you’ll notice that most of the pre-processing steps are very similar. In this tutorial, we’ll walk through each step following 
data acquisition.

> **(Placeholder for a workflow diagram)**  
> `![Workflow Diagram](path/to/workflow_diagram.png)`

### Quick Summary of Steps

1. **Pre-processing**  
   - **Import** frames and specify data acquisition parameters  
   - **Motion Correction** to account for beam-induced motion  
   - **CTF Estimation**  
   - **Tilt-Series Curation** (removing poor-quality tilts)

2. **Tilt-Series Alignment**  
   Stacking each tilted image so features line up correctly.

3. **Dose Filtering**  
   Adjust images that have high or low electron dose.

4. **Tomogram Reconstruction**  
   Combine aligned tilt-series images into a 3D volume.

5. **Post-processing**  
   - **Deconvolution / Denoising**  
   - **CTF Correction**  

