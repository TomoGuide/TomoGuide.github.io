---
layout: default
title: "STA in RELION5"
parent: "Tutorial"
nav_order: 4
---

# Subtomogram Averaging (STA) in RELION5

After you’ve reconstructed your tomograms in RELION5 (or exported them from Scipion), you can proceed with subtomogram 
averaging to reach higher resolution on repeated structures (e.g. Ribosomes).

If you started in RELION you can directly go to Pick particles (link) otherwise if you come from Scipion or AreTomo3, you have to import them to RELION.

If you skipped the RELION reconstruction part, some info about the running tab.

The parameters you need to use in the Running tab will depend on your system, if you run RELION on a local machine, or through a computing cluster etc ...

To know whether our job is going to run on GPU or CPU here's a list:

- Make pseudo-subtomos (**CPU**)
- 3D initial model (**single GPU**)
- 3D classification (**GPU normally, but CPU if not doing alignments**)
- 3D auto-refine (**GPU**)
- 3D multi-body (**GPU**)
- Tomo reconstruct particle (**CPU**)
- Tomo CTF refinement (**CPU**)
- Tomo frame alignment (**CPU**)
- Local resolution (**CPU**)

## Extract your particles and check your average

The first step to begin Subtomogram Averaging (STA) is to run an Extract Subtomos job. This job will extract cropped subtomograms from all tilts, around the center of the particle positions we identified using Template Matching.

Typically, we start with a binning of 4 or sometimes even 8. We use high binning in the initial steps because the focus is on cleaning and roughly aligning the particles—high-resolution information is not essential at this stage.
In this example, we start at bin 4. You want the box size to be larger than your target—ideally at least ~1.5 times larger. Ribosomes are approximately 350 Å wide, which corresponds to about 46 pixels at bin 4.

In general, it's best to use box sizes that are powers of 2 or 3. These "magic numbers" optimize computational performance. You can find more about that here: https://blake.bcm.edu/emanwiki/EMAN2/BoxSize.
In our case, 46 is not ideal, and we want something about 1.5 to 2 times larger anyway. A box size of 72 or 84 pixels would be appropriate, let's use 84. Of course the larger the box, the longer the computation time.

You want to use the new RELION5 "Write out as 2D stacks" for faster processing. 
Here for inputs, I used fused particles from Gain1 and Gain2 and created a tomograms.star containing both Gain1 and Gain2 tomos

<a href="/imgs/25_extract.JPG" data-lightbox="image-gallery">
  <img src="/imgs/25_extract.JPG" alt="Processing Workflow" style="width:60%;">
</a>
<a href="/imgs/25_extract2.JPG" data-lightbox="image-gallery">
  <img src="/imgs/25_extract2.JPG" alt="Processing Workflow" style="width:60%;">
</a>

At this stage, you can check how the extracted particles look.

To do this, run a <kbd>Reconstruct Particle</kbd> job. In the **I/O** tab, use the <kbd>optimisation_set.star</kbd> file that was generated during the Extract Subtomo job.
In the **Average** section, make sure to set the **Box size** and **Binning factor** to exactly the same values you used in the extraction step.

#pictures job here

Once the job is complete, you can open the resulting average in Chimera, ChimeraX, or IMOD. The reconstructed volume is typically saved as <kbd>data_merged.mrc</kbd> or <kbd>merged.mrc</kbd> inside the <kbd>~/ReconstructParticleTomo/job00X</kbd> directory. The result should resemble your expected structure—in this case, a low-resolution ribosome.

#picture ribosome here

This works well here because we used Template Matching, which provides initial orientation angles. If your particles were picked using a different method, you might need to generate an initial model using the <kbd>3D Initial Model</kbd> job instead.

This initial average will serve as a reference for the next steps, such as 3D classification or refinement.


## Refining and cleaning your particles














