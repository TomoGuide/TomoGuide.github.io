---
layout: default
title: "Preprocessing in RELION5"
parent: "Tutorial"
nav_order: 3
---

# Preprocessing and tomogram reconstruction in RELION5

---

## Initialize RELION

Create a clean working folder and name it however you want.

In the folder you created, open a terminal.

Load RELION5 and launch the tomo branch.

```
ml RELION
 
relion --tomo &
```
A GUI pops-up, you are in RELION.

## Import frames

1. Create a **rawdata** folder in your RELION working directory.  
2. Copy or symlink the `.eer` frames, `.mdoc` files, and `.gain` references into **rawdata**.  
3. Use the **Import** job in RELION to point to these raw frames and `.mdoc` files.

```
ln -s ~/path/to/frames/ ~/path/to/RELION/raw/frames
 
ln -s ~/path/to/gain ~/path/to/RELION/raw/gain
```

<a href="/imgs/14_Import1.JPG" data-lightbox="image-gallery">
  <img src="/imgs/14_Import1.JPG" alt="Processing Workflow" style="width:60%;">
</a>
<a href="/imgs/15_Import2.JPG" data-lightbox="image-gallery">
  <img src="/imgs/15_Import2.JPG" alt="Processing Workflow" style="width:60%;">
</a>

Similarly to Scipion, if you are dealing with a dataset where tilt-series were collected with different Gain references, you should import them individually, grouped by gainrefs.  
In the case of the cytoribosome dataset, out of the 33 tomos, 6 tomos were collected with one gainref and 27 with another, so we split them in two different groups. You would have to run them as separate jobs until you go to STA.

You need to know the tilt axis and the defocus handedness for RELION (something Scipion didn't ask you for!). If you get the defocus handedness wrong, this will only have an effect during STA, especially when reaching high resolution. If you want to check it, you can use **[Defocusgrad](https://github.com/CellArchLab/cryoet-scripts/tree/main)**. With this dataset, _Invert defocus handedness_ should be set as **YES (-1)**.

You also need to know the handedness of your tomos.  
In our case they are flipped, so to get the proper hand, we need to import them with an inverted tilt axis. Instead of 95 we use -95.

Both defocus handedness and tomo handedness can be checked once the tomos are reconstructed, not before! If you don't know anything about your data, start by reconstructing 2 or 3 of them and check that first before trying to batch process over 100 tomos.  
Be aware that even if data were collected on the same microscope, updates on the camera can result in flipped handedness. Data collected within the same session should be all the same.

## Motion Correction and CTF estimation 

Run beam induced motion correction using RELION's implementation of motioncor2. 

For the EER fractionation value, refer to the motion correction part of the **[Scipion tutorial](/03-tutorial/01-scipion-preprocessing/)** where we show the calculations.

At this step you want to save the ODD and EVN frames for later steps, so tick **Save images for denoising**.
Don't forget to indicate the proper gainref in the Motion tab!

<a href="/imgs/16_Motion1.JPG" data-lightbox="image-gallery">
  <img src="/imgs/16_Motion1.JPG" alt="Processing Workflow" style="width:60%;">
</a>
<a href="/imgs/17_Motion1.JPG" data-lightbox="image-gallery">
  <img src="/imgs/17_Motion1.JPG" alt="Processing Workflow" style="width:60%;">
</a>

Once this is done, run CTF estimation with the default parameters. Just make sure that the executable is proper.

Make sure that CTF is properly estimated by checking the .pdf output. If it's not you can change the base settings.

> **Placeholder CTFFIND4 screenshot**

We recommend estimating CTF with AreTomo in the next step.

## Cleaning the stack

Opens Napari to select the tilts to exclude. Bad tilts are:

- Strongly shifted compared to the others (more than 15% of the FOV)
- Partially or fully blacked out - especially high tilts
- Blurred because motion correction was not sufficient

> **Placeholder NAPARI screenshot**  

Works, it's just slow and the contrast is bad ...

## TS alignment

Getting a properly aligned tilt series is very important for efficient particle detection and for tomogram "good looking-ness".

Similar to Scipion, in RELION you can either do it using AreTomo or with IMOD patch tracking.

For this dataset, and in general for quick alignment, AreTomo is good enough. This is why we are going to show you how to automatically do it with AreTomo.

AreTomo tries to find the optimal back projection scheme based on a theoritical lamella thickness. It will also find and refine the tilt axis angle, starting from the one you provided at the import step.

The way AreTomo works right now in RELION5 is a bit restrictive. In the GUI, you can only set a single estimated tomogram thickness. AreTomo performs best when the tomogram thickness is accurate, so having one value for all tomos is not optimal.  
Here, let's run a first alignment with a value of 200 nm.  
Set Correct Tilt Angle Offset to **YES**.

<a href="/imgs/18_ALI1.JPG" data-lightbox="image-gallery">
  <img src="/imgs/18_ALI1.JPG" alt="Processing Workflow" style="width:60%;">
</a>
<a href="/imgs/19_ALI1.JPG" data-lightbox="image-gallery">
  <img src="/imgs/19_ALI1.JPG" alt="Processing Workflow" style="width:60%;">
</a>

## Refining TS alignment:

Similar to Scipion, you can measure the tomogram thickness for each tomo and use this value to have a more accurate TS alignment.

Once you aligned your tomos the first time with a single estimated tomo thickness (here we did with 200 nm), you can refine your alignment using the real tomo thickness.  
To do this, you can measure the tomo thickness in IMOD by hand.

1. **Open the bin8 tomograms** that resulted from your first AreTomo run.  
2. Go to **Edit > Image > Flip/Rotate**.  
3. **Measure the lamella thickness:**
    - Place the yellow cross on one lamella edge by left-clicking (or right or middle)  
    - Move your cursor to the opposite edge of the lamella (DON'T CLICK).  
    - Press Q to get the distance between the yellow cross and the position of your cursor.

The distance will appear in the dialog box. Here 106 pixels, **81 nm**.

<a href="/imgs/20_ali1.JPG" data-lightbox="image-gallery">
  <img src="/imgs/20_ali1.JPG" alt="Processing Workflow" style="width:60%;">
</a>

Then, you need to edit _selected_tilt_series.star_ file in the _ExcludeTiltImages_ job, and **add a column named _rlnTomoTomogramThickness #9**. The column is thickness in **nm**.  
Here's an example of an edited file:

<a href="/imgs/21_ali1.JPG" data-lightbox="image-gallery">
  <img src="/imgs/21_ali1.JPG" alt="Processing Workflow" style="width:60%;">
</a>

Whenever you modify a file, create a copy of the original and name it *_ori, so that you keep track of what you have done and can easily revert back.

Then run the AreTomo job again with the edited selected_tilt_series.star and run the tomo reconstruction. Your tomo should look better.  
To be sure it was taken into account, check the .log and _AlignZ_ should be different for all tomos.

## Tomogram reconstruction

Fill in the info about your tomogram dimensions, here **4096 4096 2048**, and pixel size. Here **7.64** for bin4 tomos.

I would advise you to first reconstruct all the tomos with AreTomo (with corrected Z thickness). Then check the ones that are not properly reconstructed.

For the ones that are not properly reconstructed, align them using IMOD, and then at the Reconstruct step, only reconstruct the ones that were not properly reconstructed using AreTomo using the "Reconstruct only these tomo" option.

Tilt angle offset: If you reconstructed with AreTomo, you don't need to use this because AreTomo automatically flatens the tomo.  
IMOD is not doing it (you can correct it manually in etomo, but for some reason the current version of RELION is not reading it). Hence your tomo will be tilted. Here the tilt angle was about 10°, so you need to specify 10 to correct it.

<a href="/imgs/22_recons.JPG" data-lightbox="image-gallery">
  <img src="/imgs/22_recons.JPG" alt="Processing Workflow" style="width:60%;">
</a>
<a href="/imgs/23_recons.JPG" data-lightbox="image-gallery">
  <img src="/imgs/23_recons.JPG" alt="Processing Workflow" style="width:60%;">
</a>
<a href="/imgs/24_recons.JPG" data-lightbox="image-gallery">
  <img src="/imgs/24_recons.JPG" alt="Processing Workflow" style="width:60%;">
</a>

Additional flags to make your tomo look nicer and we advise to use:  
- ---**Fc** (binning by Fourier cropping)  
- ---**pre_weight** (pre-weighting in 2D instead of 3D weighting)  
- ---**ctf** (CTF correction by phase flip)  
- ---**SNR 100** (Wiener filter SNR value. Default 10. We found 100 looks "more IMOD-like")

We advise to always CTF correct tomograms especially for template matching and STA. Even denoising like [cryoCARE](https://github.com/juglab/cryoCARE_pip) and [DeepDeWedge](https://github.com/MLI-lab/DeepDeWedge) for example can benefit from this extra information of frequencies.

## Create denoised tomos

> TBD
