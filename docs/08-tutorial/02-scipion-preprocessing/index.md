---
layout: default
title: "Preprocessing in Scipion"
parent: "Tutorial"
nav_order: 2
---

# Preprocessing and tomogram reconstruction in Scipion

In this section, we will go through how to reconstruct your tomograms using Scipion, and at the end, how to export the 
information to perform STA in RELION5. If you want to do everything in RELION5, you can click "here" (LINK to the page below).

Scipion is a freely available and open-source software that mainly acts as a wrapper for other programs, allowing you to 
organize your projects and have different software interact almost seamlessly. It can be used for both SPA and tomography, 
but it is particularly handy for tomography since the field currently lacks a “simple” software that performs all the 
operations necessary for tomogram reconstruction and STA, while also keeping track of your work.

> **(Placeholder for a screenshot of Scipion interface)**

Here's an example of the workflow that we are going to follow:

> **(Placeholder for a small workflow graphic)**

---

## Create a new project

From the base Scipion interface click on **Create Project**, give it a name and a location.  
From there you are in your Scipion project, which is for now empty.

---

## Import frames

The first step is going to import the frames. You can press on the top left or press <kbd>Ctrl + f</kbd> and look for 
“Add tomo - import tilt-series”.

Here, you need to specify the directory that contains the movies (e.g., `.eer`) and the `.mdoc` files that contain the 
information about each TS.

> **Note**: We work with Tomo5 mdocs (TFS acquisition software). In your case you might be working with SerialEM mdocs.  
> Scipion is smart enough to read the info from the mdoc files, but I recommend overriding these values if you know them, 
  since they are often wrong, notably the Tilt axis angle.  
> If you collect your own data on a "classic" Titan G4 with Falcon4i and SelectrisX in `.eer`, the tilt axis will probably be 
  the same as here. If in doubt, ask your facility manager, or check the output of AreTomo.  
> A wrong tilt axis angle might result in a wrong-handed tomogram (mirrored), so it's really important to be sure.

---

## Motion correction

This step aligns each frame of the movies and corrects for beam-induced motion.

> **(Placeholder for screenshot of Scipion motion correction settings)**

Tick **Yes** for “Split & sum odd/even frames” if you later want to denoise your tomograms using cryoCare.

In the “Motioncor params” tab, since we are dealing with really low dose per tilts and frames (as opposed to SPA, where 
the dose is usually about 10 times higher), we will perform **full frame motion correction** instead of dividing them into patches.

Because this dataset was collected as **.eer**, specify how you want to group frames. For instance:

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

## Cleaning the stack

Once motion correction is done, you want to remove “bad” tilts from your TS. Bad tilts are tilts that are either:
- Strongly shifted compared to the others (more than 15% of the FOV)
- Partially or fully blacked out
- Blurred because motion correction was not sufficient

---

## TS alignment

Can be done either with **AreTomo** or with **IMOD patch tracking**. Here we automatically do it with AreTomo.  
AreTomo tries to find the optimal back projection scheme based on a theoretical lamella thickness. It also refines the 
tilt axis angle, starting from the one you provided at import.

> **(Placeholder for screenshot of AreTomo alignment)**

- For the initial reconstruction, we use an estimated lamella thickness of ~1000 unbinned pixels (~190 nm).  
- We don’t reconstruct odd/even tomograms for denoising at this stage.  
- A little note: check your tilt axis sign (±95°). If it’s reversed, you’ll end up with mirrored tomograms.

### Refining AreTomo TS alignment

Open the bin8 tomograms from your first AreTomo run. Flip/Rotate volumes as needed. Measure lamella thickness 
(e.g., in IMOD). Provide these thickness measurements to AreTomo in the advanced options if you want more accurate 
reconstructions.

---

## CTF estimation

You can use AreTomo to estimate CTF automatically or pick alternatives like **CTFFIND5**. We found AreTomo was better 
than older CTFFIND4.

---

## Tomogram reconstruction

Perform dose filtering, apply transforms, then reconstruct tomograms. They might still look noisy.  
To help make them more visually interpretable:

### Generating denoised tomograms

- Quick approach: **dimifilter**  
- More advanced: **cryoCare** (if you split odd/even frames earlier)

### Generating CTF corrected tomograms

---

## Exporting data to RELION5 for STA

Scipion can export the final alignment, tomograms, or pick coordinates into RELION5. In the next section, we’ll see how 
to do the entire pipeline in RELION5.
