---
layout: default
title: "Tutorial"
nav_order: 9
has_children: true
---

# Preamble

For this tutorial, we compiled a subset of 33 tomograms from the "Chlamy dataset" (EMPIAR-11830) (R. Kelley, et al, *Towards 
community-driven visual proteomics with large-scale cryo-electron tomography of Chlamydomonas reinhardtii.* bioRxiv [Preprint] 
(2024). [https://doi.org/10.1101/2024.12.28.630444](https://doi.org/10.1101/2024.12.28.630444).)

Even if already reconstructed tomograms are available for this dataset (originally reconstructed using TOMOMAN and 
automatically aligned with AreTomo), this tutorial aims at teaching you how to reconstruct tomograms from scratch and get 
into subtomogram averaging (STA). We believe this subset can also be of interest for people benchmarking different software.

In this tutorial, software that you will need are: **Scipion** (link), **RELION5** (link), **IMOD** (link), and **pytom-match-pick** (link). 
We like to start our tomogram preprocessing and reconstruction in Scipion, but we will also describe how to do the complete 
RELION5 pipeline. We recommend that you also download ChimeraX (link) and the ArtiaX (link) plug-in for particle map-backs.

Start by going to the **Dataset, Software and Resources** tab and make sure you download all the necessary software as well 
as the dataset.

The downloaded dataset should contain:

- The raw frames  
- The .mdoc for each tilt series (6 + 27)  
- The gain references (2)  
- Two text files with thickness measurements for automated AreTomo TS alignment  
- Templates and masks for Template Matching  

The 33 tilt-series (TS) can be divided into two groups: 6 associated with **GainRef1** and 27 with **GainRef2**.  
If you are just interested in learning the basics of tomogram reconstruction and STA, we recommend just using the 6 TS 
associated with **GainRef1** as processing will be faster.  
If you also want to push resolution or try classification, process the entire set of TS.

---

# General workflow

Cryo-ET data processing can be summarized by this simple chart. If you’re familiar with single-particle data processing, 
you’ll notice that most of the pre-processing steps are very similar.

In this tutorial, we’ll walk through all the steps following data acquisition.

> **(Placeholder for a workflow diagram)**

Here’s a quick summary:

## Pre-processing

- **Import**: Load raw images into the analysis software and specify all the data acquisition parameters.  
- **Motion Correction**: During imaging, the sample may drift slightly because it’s very sensitive to the electron beam. 
  Motion correction detects and compensates for sample drifting caused by the electron beam, ensuring each image is as 
  sharp and stable as possible.  
- **CTF Estimation**: The electron microscope’s lenses can cause distortions in the images. The “Contrast Transfer Function” 
  (CTF) describes those distortions. By estimating the CTF, you gather important information that helps correct these 
  effects later on.  
- **Tilt-Series Curation**: Because you take a series of images at various tilt angles, some may be poor quality (e.g., too 
  much drift or damage). Tilt-series curation is where you review the images and remove or flag the bad ones, ensuring only 
  the best data moves forward.

## Tilt-Series Alignment

Each tilted image is lined up properly with the others (kind of like stacking photos exactly on top of each other). Alignment 
makes sure that the features in each tilt angle match up in the correct positions to reconstruct the most accurate 3D volume.

## Dose Filtering

High doses of electrons can damage the sample, and lower doses might lead to noisy images. Dose filtering involves adjusting 
images that are overly damaged or too low in signal to be useful. This helps maintain quality in the final 3D reconstruction.

## Tomogram Reconstruction

Once all the tilt-series images are aligned and cleaned up, the software combines them to form a 3D “tomogram,” which is 
essentially a 3D picture of your sample.

## Post-processing

- **Deconvolution / Denoising**: Specialized methods refine the 3D data by enhancing details and reducing noise, similar to 
  using a ‘sharpen’ or ‘denoise’ filter on a standard image—only adapted for three dimensions. This is used to make the 
  tomograms pleasant to “human” eyes.  
- **CTF Correction**: You use the previously estimated CTF information to correct for the lens distortions in your final 3D 
  tomogram, improving the overall quality and clarity. This is especially important for STA and automated particle picking.
