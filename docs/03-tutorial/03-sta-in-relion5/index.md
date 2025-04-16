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

<a href="/imgs/25_extract3.JPG" data-lightbox="image-gallery">
  <img src="/imgs/25_extract3.JPG" alt="Processing Workflow" style="width:60%;">
</a>
<a href="/imgs/25_extract4.JPG" data-lightbox="image-gallery">
  <img src="/imgs/25_extract4.JPG" alt="Processing Workflow" style="width:60%;">
</a>

Once the job is complete, you can open the resulting average in Chimera, ChimeraX, or IMOD. The reconstructed volume is typically saved as <kbd>data_merged.mrc</kbd> or <kbd>merged.mrc</kbd> inside the <kbd>~/ReconstructParticleTomo/job00X</kbd> directory. The result should resemble your expected structure—in this case, a low-resolution ribosome.

<a href="/imgs/26_reconstruct.png" data-lightbox="image-gallery">
  <img src="/imgs/26_reconstruct.png" alt="Processing Workflow" style="width:60%;">
</a>


This works well here because we used Template Matching, which provides initial orientation angles. If your particles were picked using a different method, you might need to generate an initial model using the <kbd>3D Initial Model</kbd> job instead.

This initial average will serve as a reference for the next steps, such as 3D classification or refinement.


## Refining and cleaning your particles

To improve your average, you have a few key options—mainly **alignment** and **classification**. You can perform both simultaneously or run them separately, depending on your goal.

Just like in single-particle analysis, achieving high resolution requires a homogeneous particle set. That’s where classification comes in—it helps you sort out different types of particles or complexes (e.g., ribosomes vs. HSP60), remove junk or noise, and identify structural heterogeneity within a complex (e.g. different translational states of ribosomes).

The first step here is to clean out false positives that may have been picked during Template Matching.

A good approach is to **first align all particles** together using a quick local refinement. Since our particles already have initial orientations from pyTOM, a fast alignment with local searches only should be sufficient at this stage.

Once you've done that, you can proceed to **classification without alignment** to separate good particles from bad ones, or to distinguish between different structural states.

## Fast alignment using 3D classification with a single class

To align your particles, you have two option, **3D Refinement** and **3D Classification**. 3D Refinement is the best but is a bit more time consuming, because of how it handles the particle, than 3D classification. At this first step we can use 3D classification, which is a bit less accurate, but faster. 

Let's launch an alignment with the 3D classification using only one class.


### I/O Tab

- **Input images STAR file**: Use the <kbd>particles.star</kbd> file generated by the PseudoSubtomo job.
- **Reference map**: Use the <kbd>merged.mrc</kbd> file produced by the **ReconstructParticleTomo** job.

<a href="/imgs/27_class.png" data-lightbox="image-gallery">
  <img src="/imgs/27_class.png" alt="Processing Workflow" style="width:60%;">
</a>

### Reference Tab

- **Initial low-pass filter:** Start with **50 Å**. Later, adjust this value to be slightly above the estimated resolution from the previous step. This is important to low-pass filter your reference to avoid over-fitting.
- Example: If 3D Classification gave you a resolution of **22 Å**, set this to **25 Å** in **3D Refinement**.
- **Symmetry**: We don't work with a symmetric complex. If you do, I would recommend to initially **not apply symmetry**. Apply it in later stages.

<a href="/imgs/27_class2.png" data-lightbox="image-gallery">
  <img src="/imgs/27_class2.png" alt="Processing Workflow" style="width:60%;">
</a>

### CTF Tab

- **CTF Correction**: Always **enable** this.
- **Ignore CTFs until first peak**: It can be usefull to turn on at the beginning, especially for initial alignment and classification. This option acts like a low-pass filter.

<a href="/imgs/27_class3.png" data-lightbox="image-gallery">
  <img src="/imgs/27_class3.png" alt="Processing Workflow" style="width:60%;">
</a>

### Optimization Tab

- **Number of classes**: Set to the number of desired classes. Current case: We are using **1** class.
- **T parameter**: This is a parameter that you will have to play with. Usually launching multiple jobs with different T values is smart (e.g 0.5, 1, 2 and 4)
- **Number of iterations**: Start with the default of **25**.
- **Mask diameter**: Should be about **90% of the box size**.<br>
 `Example: For box size = 84 px, pixel size = 1.91 Å, binning = 4`:<br>
'84 × 1.91 × 4 × 0.9 = ~570 Å'

- **Limit resolution E-step to**: To avoid overfitting and noisy reconstructions, set this to the **Nyquist resolution** at your current binning. <br>
'Example: 1.91 × 4 × 2 = ~15 Å'

<a href="/imgs/27_class4.png" data-lightbox="image-gallery">
  <img src="/imgs/27_class4.png" alt="Processing Workflow" style="width:60%;">
</a>

## Sampling Tab

- **Perform alignment**: Set to **Yes** (since we are aligning). If not aligning, set to **No**.
- **Off search range and step**. This define the translation parameters, how much you let your thing move in the box. Our particles should be centered thanks to TM and we are high binning, so best is use small values like a range of 2 range and a step of 1.
- **Local searches only**: Enable this, and use the suggested parameters. This overrides **Angular sampling interval**. We only want to perform local searches because our particles are already well aligned from TM.

<a href="/imgs/27_class5.png" data-lightbox="image-gallery">
  <img src="/imgs/27_class5.png" alt="Processing Workflow" style="width:60%;">
</a>

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

- **Input images STAR file**: Use the <kbd>*_data.star file</kbd> generated by the previous **3Dclass** job. E.g, if you liked iteration 9 you have to use <kbd>run_it009_data.star</kbd> as input.
- **Reference map**: Use the <kbd>run_it009_data.mrc</kbd> file created by the **3Dclass** job.  

### Reference Tab

- **Initial low-pass filter**: You can now set the low pass filter **slightly above the resolution** given by the iteration you reached in the 3DClass with 1 class

### Optimization Tab

- **Number of classes**: This now depends on the heterogeneity you expect and the number of particles you have. Start maybe with 8 classes, but this parameter should be changed and tested.
- **T parameter**: This is a parameter that you will have to play with. Usually launching multiple jobs with different T values is smart (e.g 0.5, 1, 2 and 4).

### Sampling Tab

- **Perform alignment**: Set to **No** this time

### Compute Tab

- Turn off GPU acceleration

To check the progress of the classification, can check each iterations in the <kbd>~/Class3D/job00X</kbd> and open the volumes <kbd>run_it00x_class001.mrc</kbd> in ChimeraX or IMOD.
You can also follow it in RELION. Go to <kbd>File > Display</kbd> (or Alt+D)  and open <kbd>Class3D > job00X > run_it000_model.star</kbd>.

This would show you a 2D slice of all your classes, which can sometime be helpful to judge about the quality of the classes.

Like this for example:

<a href="/imgs/26_class5.JPG" data-lightbox="image-gallery">
  <img src="/imgs/26_class5.JPG" alt="Processing Workflow" style="width:60%;">
</a>


By left clicking on the class you are selecting it. If you right-click you have different options, and statistics available.

When running refinement or classification in RELION, it's helpful to keep track of progress using a few key metrics in the output files.

You can run this command from the terminal:

- **Monitor Resolution Over Iterations**
   This will give you the resolution for each iteration. Ideally, it should improve (i.e., the number decreases) as refinement proceeds.
   ```
   grep _rlnCurrentResolution run_it???_model.star
   ```

- **Follow class population and resolution**
   If you're running classification, use <kbd>relion_star_printtable</kbd> to examine how the classes evolve:
   ```
   relion_star_printtable run_it001_model.star data_model_classes rlnClassDistribution rlnEstimatedResolution
   ```

- **Monitor Class Convergence (Optimum Changes)**
   Track how well classes are stabilizing. This number should start high (near 1) and trend toward 0 as the classes converge
   ```
   grep _rlnChangesOptimalClasses run_it???_optimiser.star
   ```

## Select good classes 

Assuming your classification worked and you want to select good class(es), and only proceed with these.

Let's launch a **Subset selection job**.

### I/O Tab

For the Input images STAR file, take the particles file that was created by the Class3D job, at the iteration that you want. For example, if you liked iteration 24 you have to use <kbd>Class3D/job00X/run_it024_optimiser.star </kbd> as input.

### Running Tab

Don't submit it to the queue, run it locally!

Press Run!

The Relion display GUI pops up, press Display! This will show you a 2D slice of all your classes. It's similar to the Display tab.

To select the one(s) you want, left click on it/them. Then press right click, and do Save selected classes. You also, BEFORE SAVING, you can right-click and do Show metadata this class, and it will show you the total number of particles you are selecting.

Close the windows, it RELION should tell Saved Select/job00X/particles.star with XXXXX selected particles.





## Creating a mask

bla bla



