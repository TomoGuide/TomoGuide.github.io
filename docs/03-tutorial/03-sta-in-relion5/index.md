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

To improve your average, you have a few key options—mainly **alignment** and **classification**. You can perform both simultaneously or run them separately, depending on your goal.

Just like in single-particle analysis, achieving high resolution requires a homogeneous particle set. That’s where classification comes in—it helps you sort out different types of particles or complexes (e.g., ribosomes vs. HSP60), remove junk or noise, and identify structural heterogeneity within a complex (e.g. different translational states of ribosomes).

The first step here is to clean out false positives that may have been picked during Template Matching.

A good approach is to **first align all particles** together using a quick local refinement. Since our particles already have initial orientations from pyTOM, a fast alignment with local searches only should be sufficient at this stage.

Once you've done that, you can proceed to *classification without alignment* to separate good particles from bad ones, or to distinguish between different structural states.

## Fast alignment using 3D classification with a single class

To align your particles, you have two option, **3D Refinement** and **3D Classification**. 3D Refinement is the best but is a bit more time consuming, because of how it handles the particle, than 3D classification. At this first step we can use 3D classification, which is a bit less accurate, but faster. 

Let's launch an alignment with the 3D classification using only one class.


<a href="/imgs/26_class.JPG" data-lightbox="image-gallery">
  <img src="/imgs/26_class.JPG" alt="Processing Workflow" style="width:60%;">
</a>
<a href="/imgs/26_class2.JPG" data-lightbox="image-gallery">
  <img src="/imgs/26_class2.JPG" alt="Processing Workflow" style="width:60%;">
</a>

### I/O Tab

- Input images STAR file: Use the particles.star file generated by the PseudoSubtomo job.

- Reference map: Use the merged.mrc file produced by the ReconstructParticleTomo job.

### Reference Tab

- **Initial low-pass filter:** Start with **50 Å**. Later, adjust this value to be slightly above the estimated resolution from the previous step. This is important to low-pass filter your reference to avoid over-fitting.
- Example: If 3D Classification gave you a resolution of **22 Å**, set this to **25 Å** in **3D Refinement**.
- **Symmetry**: We don't work with a symmetric complex. If you do, I would recommend to initially **not apply symmetry**. Apply it in later stages.

### CTF Tab

- **CTF Correction**: Always **enable** this.
- **Ignore CTFs until first peak**: It can be usefull to turn on at the beginning, especially for initial alignment and classification. This option acts like a low-pass filter.

### Optimization Tab

- **Number of classes**: Set to the number of desired classes. Current case: We are using **1** class.
- **T parameter**: This is a parameter that you will have to play with. Usually launching multiple jobs with different T values is smart (e.g 0.5, 1, 2 and 4)
- **Number of iterations**: Start with the default of **25**.
- **Mask diameter**: Should be about **90% of the box size**. 
Example: For box size = 84 px, pixel size = 1.91 Å, binning = 4: 
84 × 1.91 × 4 × 0.9 = ~570 Å
- **Limit resolution E-step to**: To avoid overfitting and noisy reconstructions, set this to the **Nyquist resolution** at your current binning.
Example: 1.91 × 4 × 2 = ~15 Å

## Sampling Tab

- **Perform alignment**: Set to **Yes** (since we are aligning). If not aligning, set to **No**.
- **Off search range and step**. This define the translation parameters, how much you let your thing move in the box. Our particles should be centered thanks to TM and we are high binning, so best is use small values like a range of 2 range and a step of 1.
- **Local searches only**: Enable this, and use the suggested parameters. This overrides **Angular sampling interval**. We only want to perform local searches because our particles are already well aligned from TM.

### Helix Tab

- Not applicable in our case. Leave it untouched. This is only if you are working with something with helical symmetry, e.g. a filament.

### Compute Tab

- Configure settings as shown.
- **Enable GPU acceleration only** if performing image alignment. If not aligning, **disable GPU acceleration**.



## 3D classification without alignment

Your particles should now be all aligned more or less the same way. The next step would be to run classification without alignment, but this might be different for you.

Let's again launch a Class3D job.


<a href="/imgs/26_class3.JPG" data-lightbox="image-gallery">
  <img src="/imgs/26_class3.JPG" alt="Processing Workflow" style="width:60%;">
</a>
<a href="/imgs/26_class4.JPG" data-lightbox="image-gallery">
  <img src="/imgs/26_class4.JPG" alt="Processing Workflow" style="width:60%;">
</a>


### I/O Tab

- **Input images STAR file**: Use the *_data.star file generated by the previous **3Dclass** job. E.g, if you liked iteration 9 you have to use __run_it009_data.star__ as input.

Reference map: Use the run_it009_class001.mrc file created by the 3Dclass job.  

Reference Tab

Initial low-pass filter: You can now set the low pass filter slightly above the resolution given by the iteration you reached in the 3DClass with 1 class

Optimization Tab

Number of classes: This now depends on the heterogeneity you expect and the number of particles you have. Start maybe with 8 classes, but this parameter should be changed and tested.

T parameter: This is a parameter that you will have to play with. Usually launching multiple jobs with different T values is smart (e.g 0.5, 1, 2 and 4).

Sampling Tab

Perform alignment: Set to No this time

Compute Tab

Turn off GPU acceleration

To check the progress of the classification, can check each iterations in the ~/Class3D/job00X and open the volumes run_it00x_class001.mrc in ChimeraX or IMOD.
You can also follow it in RELION. Go to File > Display (or Alt+D)  and open Class3D > job00X > run_it000_model.star.

This would show you a 2D slice of all your classes, which can sometime be helpful to judge about the quality of the classes.

Like this for example:







