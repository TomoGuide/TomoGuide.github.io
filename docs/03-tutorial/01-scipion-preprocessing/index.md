---
layout: default
title: "Preprocessing in Scipion"
parent: "Tutorial"
nav_order: 2
---

# Preprocessing and tomogram reconstruction in Scipion

In this section, we will go through how to reconstruct your tomograms using Scipion, and at the end, how to export the 
information to perform STA in RELION5. If you want to do everything in RELION5, you can click **[here](/03-tutorial/03-sta-in-relion5/)**.

Scipion is a freely available and open-source software that mainly acts as a wrapper for other programs, allowing you to 
organize your projects and have different software interact almost seamlessly. It can be used for both SPA and tomography, 
but it is particularly handy for tomography since the field currently lacks a “simple” software that performs all the 
operations necessary for tomogram reconstruction and STA, while also keeping track of your work.


Here's **the** example workflow that we are going to follow:

<img src="/imgs/02_Workflow_Scipion.PNG" alt="Processing Workflow" style="width:40%;">

---

## Create a new project

From the base Scipion interface click on **Create Project**, give it a name and a location.  
From there you are in your Scipion project, which is for now empty.

---

## Import frames

The first step is going to import the frames. You can press <kbd>Add</kbd> on the top left or press <kbd>Ctrl + f</kbd> to select a protocol. Look for 
_“Add tomo - import tilt-series”_.

<img src="/imgs/03_image-2025-1-15_16-17-56.png" alt="Processing Workflow" style="width:70%;">

Here, you need to specify the directory that contains the movies (e.g., `.eer`) and the `.mdoc` files that contain the 
information about each tilt-series.

> **Note**: We work with Tomo5 mdocs (TFS acquisition software) here but you might be working with SerialEM mdocs. In any case, Scipion is smart enough to read the info from the mdoc files. However, we recommend overriding these values if you know them! Since they can be wrong in the mdoc file, notably the **Tilt axis angle**. If you collect your own data on a "classic" Titan G4 with Falcon4i and SelectrisX in `.eer`, the tilt axis will probably be the same as here. If you acquired in `.tiff` this value might be different. In doubt, ask your facility manager, or check the output of AreTomo (or IMOD) which can estimate the tilt axis angle. A wrong tilt axis angle might result in a wrong-handed tomogram (mirrored), so it's really important to be sure of that.

---

## Motion correction

This step aligns each frame of the movies and corrects for beam-induced motion.

<img src="/imgs/04_image-2025-1-15_16-18-38.png" alt="Processing Workflow" style="width:70%;">
<img src="/imgs/05_image-2025-1-15_16-19-1.png" alt="Processing Workflow" style="width:70%;">
<img src="/imgs/06_image-2025-1-15_16-19-55.png" alt="Processing Workflow" style="width:70%;">
<img src="/imgs/07_image-2025-1-15_16-19-23.png" alt="Processing Workflow" style="width:70%;">

Tick **Yes** for _“Split & sum odd/even frames”_ if you later want to denoise your tomograms using cryoCare.

In the _“Motioncor params”_ tab, since we are dealing with really low dose per tilts and frames (as opposed to SPA, where 
the dose is usually about 10 times higher), we will perform **full frame motion correction** instead of dividing them into patches.

Because this dataset was collected as `.eer`, specify how you want to group frames. For instance:

```
N_frames = desired_dose_per_group / ( dose_per_tilt * pixel_size² / total_EER_frames )
```
In general, we want **0.5** as the desired dose per grouped frames.

```
N_frames = 0.5 / ( 3.5 * 1.91² / 308  ) = 12
```

...which means, for GainRef1, we are dividing our EER exposure into 308 / 12 = ~ 25 slices

Note that tilts from GainRef1 and GainRef2 have different number of frames (on average), so edit that value accordingly!

```
N_frames = 0.5 / ( 3.5 * 1.91² / 350  ) = 13.7 ~ 14 frames
```

For `.tiff` frames this is not necessary/will not be considered.

## Cleaning the stack

Once motion correction is done, you want to remove “bad” tilts from your tilt stack. Bad tilts are tilts that are either:
- Strongly shifted compared to the others (more than 15% of the FOV)
- Partially or fully blacked out
- Blurred because motion correction was not sufficient

You can just open the motion correction output by double click and hold down <kbd>space bar</kbd> to deselect the bad tilts. Once done cleaning all bad tilts save the new tilt stack and use this as the import for the next TS alignment job.

---

## Tilt-series alignment

Can be done either with **AreTomo** or with **IMOD patch tracking**. Here we automatically do it with AreTomo. AreTomo tries to find the optimal back projection scheme based on a theoretical lamella thickness. It also refines the tilt axis angle, starting from the one you provided at import.

<img src="/imgs/08_Aretomo1_Scipion.PNG" alt="Processing Workflow" style="width:70%;">
<img src="/imgs/09_Aretomo2_Scipion.PNG" alt="Processing Workflow" style="width:70%;">


> **Note**: This is where having the correct tilt angle becomes crucial. If you use 95° instead of -95°, AreTomo will reconstruct tomograms with an inverted hand, causing them to be mirrored. Hence, in later steps—such as when performing Subtomogram Averaging (STA)—you could end up with mirrored structures. To check whether your tomograms are mirrored, you can run template matching with both properly oriented and inverted-hand templates (link).  

We will reconstruct bin8 tomograms first to quickly assess overall quality and to measure lamella thickness for Z-height refinement. For the initial reconstruction, we will use an estimated lamella  thickness of 1000 unbinned pixels (~190 nm).

At this step we will not reconstruct odd/even tomograms for denoising since we are going to do it at a later stage.

> **PLACEHOLDER: Here a little movie comparing unaligned with aligned TS**

### Refining AreTomo TS alignment

This step is optional but highly recommended if you want well-aligned tomograms. The better the tomogram alignment, the easier it will be to detect your target of interest, and the better resolution you can achieve in subsequent analyses.

1. Open the bin8 tomograms that resulted from your first AreTomo run.
2. Flip/Rotate the volumes as needed.
3. Measure the lamella thickness:
  - Place the yellow cross on one lamella edge by left-clicking.
  - Move your cursor to the opposite edge of the lamella.
  - Press <kbd>Q</kbd> to get the distance between the yellow cross and the position of your cursor.

> **PlACEHOLDER picture of how to measure thickness in IMOD**

Once you have done that for all the tomograms, you can provide this file as an input for the AreTomo job in the Advanced options.

<img src="/imgs/10_Aretomo3_Scipion.PNG" alt="Processing Workflow" style="width:70%;">

---

## CTF estimation

You can use different options to estimate CTF. However, you might have noticed that AreTomo can do it, and if you ran the previous job exactly as we did, you already have done it.
We found that CTFFIND4, that was an alternative that we were using, was not performing as well as AreTomo. CTFFIND5 appears to be performing better than CTFFIND4, so we recommand using this one if you don't want to use AreTomo. You can check the CTF estimate by opening the AreTomo CTF output. Your CTF values should not deviate much over the entire tilt-series except for the bad tilts. Take care of the Y-axis scaling! This can be missleading.

---

## Tomogram reconstruction

Perform dose filtering, apply transforms, then reconstruct tomograms.

<img src="/imgs/11_Dosefiltering_Scipion.PNG" alt="Processing Workflow" style="width:70%;">
<img src="/imgs/12_Transform_Scipion.PNG" alt="Processing Workflow" style="width:70%;">

We can now finally reconstruct our tomograms

<img src="/imgs/13_Recontruct_Scipion.PNG" alt="Processing Workflow" style="width:70%;">

However, those tomogram don't look particularly contrasty right? To help make it nicer to our human eyes we can denoise them

### Generating denoised tomograms

TBD 

- Quick approach: **dimifilter**  
- More advanced: **cryoCare** (if you split odd/even frames earlier)

### Generating CTF corrected tomograms

---

## Exporting data to RELION5 for STA

Scipion can export the final alignment, tomograms, or pick coordinates into RELION5. In the next section, we’ll see how 
to do the entire pipeline in RELION5.
