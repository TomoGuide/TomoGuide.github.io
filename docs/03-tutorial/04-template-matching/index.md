---
layout: default
title: "Particle Picking"
parent: "Tutorial"
nav_order: 4
---

# Particle Picking by Template Matching

> "Template matching is a technique in digital image processing for finding small parts of an image which match a template image" 
    --Wikipedia, 2025

## Create boundary masks of your tomograms

To restrict particle extraction (or detection) to your lamella only, and not the entire tomogram, you might want to use a "slab mask" or a boundary mask. This is particularly useful on difficult targets no minimize the amount of false positive picks.

In our case with ribosomes, a slab mask is actually not really necessary.
You can automatically create a boundary masks using [Slabify](https://github.com/CellArchLab/slabify-et). This works best on denoised tomograms. For detailed instruction check the Slabify [Wiki](https://github.com/CellArchLab/slabify-et/wiki)
If you know where you particle should be (e.g mitochondria, nucleus etc) you can further restrict your search by creating appropriate masks with software like [AMIRA](https://www.thermofisher.com/ch/en/home/electron-microscopy/products/software-em-3d-vis/amira-software/cell-biology.html?cid=msd_vds_ls_none_amr_123456_gl_pso_gaw_escchz&gad_source=1&gbraid=0AAAAADxi_GQP_HTwG5UvF75JpF-4AyxPP&gclid=Cj0KCQjwzYLABhD4ARIsALySuCSqID5sMy5zoaTDmyL_ScEreSQ_k5zh6D2_RKo_Dhr_trSPq0WOU7caAjPIEALw_wcB), [Napari](https://napari.org/stable/), [MITK](https://github.com/MITK/MITK) - or using automated membrane segmentations tools like [MemBrain-seg] (https://github.com/teamtomo/membrain-seg). 

<a href="/imgs/35_slab.png" data-lightbox="image-gallery">
  <img src="/imgs/35_slab.png" alt="Processing Workflow" style="width:60%;">
</a>

You can make a slab_mask folder inside the tomograms folder (e.g RELION5/Tomograms/jobXXX/tomograms/) for example.






