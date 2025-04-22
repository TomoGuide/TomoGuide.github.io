---
layout: default
title: "Datasets, Software, and Resources"
nav_order: 8
---

# Datasets, Software, and Resources

## **Dataset: EMPIAR-11830**
For this tutorial, we compiled a subset of **33 tomograms** from the "Chlamy dataset" (EMPIAR-11830) (R. Kelley, et al, *Towards community-driven visual proteomics with large-scale cryo-electron tomography of Chlamydomonas reinhardtii.* bioRxiv [Preprint] (2024). [https://doi.org/10.1101/2024.12.28.630444](https://doi.org/10.1101/2024.12.28.630444)).

Even if already reconstructed tomograms are available for this dataset (originally reconstructed using TOMOMAN, automatically aligned with AreTomo), this tutorial aims at teaching you how to reconstruct tomograms from scratch and get into subtomogram averaging (STA).  
We believe this subset can also be of interest for people benchmarking different software.

You can directly download the subsets from EMPIAR using this command:

`wget -r -N -np -nH --cut-dirs=4 ftp://ftp.ebi.ac.uk/empiar/world_availability/EMPIAR-code/`

<!-- For Engel people it's here:  
`/scicore/home/engel0006/GROUP/pool-visprot/Florent/Folder_Newpipeline/Frames/Set_Cytoribo` -->

This folder should contain:
- The raw frames  
- The .mdoc for each tilt series (6 + 27)  
- The gain references (2)  
- Two text files with thickness measurements for automated AreTomo TS alignment  
- Templates and masks for Template Matching  

The 33 tilt-series (TS) can be divided into two groups: 6 associated with GainRef1 and 27 with GainRef2.  
If you are just interested in learning the basics of tomogram reconstruction and STA, we recommend using only the 6 TS associated with GainRef1 as processing will be faster.  
If you want to push resolution (try classification etc.), process the entire set of TS.

**General info about the dataset:**

- **Detector:** Falcon4i with SelectrisX energy filter, using Tomo5 on a generation 4 Titan Krios(es)  
- **Pixel size:** 1.91 (originally 1.96, but 1.91 is closer to reality)  
- **Voltage:** 300  
- **Spherical aberration:** 2.7  
- **Tilt axis:** 95 (use -95 to get the proper tomogram handedness)  
- **Defocus handedness:** -1 in RELION if starting from scratch (+1 if using the TOMOMAN preprocessed project)  
- **Dose:** 3.5 e-/A² per tilt

---

## **Software**

You need to have access to a GPU-powered machine running on Linux. It can be a local machine or a computing cluster. In our case, we work on a computing cluster with a SLURM system.
You will also need to have appropriate CUDA drivers (this means you need to have NVIDIA GPUs) and a python installation.

[Scipion](https://scipion.i2pc.es/){: .btn } <br>
[IMOD](https://bio3d.colorado.edu/imod/){: .btn } <br>
[RELION 5](https://relion.readthedocs.io/en/release-5.0/){: .btn } <br>
[AreTomo3](https://github.com/czimaginginstitute/AreTomo3){: .btn } <br>
[ChimeraX](https://www.cgl.ucsf.edu/chimerax/){: .btn } <br>
[ArtiaX](https://github.com/FrangakisLab/ArtiaX){: .btn } <br>
[pytom-match-pick](https://github.com/SBC-Utrecht/pytom-match-pick){: .btn } <br>
