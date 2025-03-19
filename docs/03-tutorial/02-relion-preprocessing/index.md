---
layout: default
title: "Preprocessing in RELION5"
parent: "Tutorial"
nav_order: 3
---

# Preprocessing and tomogram reconstruction in RELION5

---

## Import frames

To import your tilt series in RELION you need to have a folder containing your frames (as .eer or .tiff) and your .mdoc files pointing to your frames. You can also directly import motion corrected .mrc. In your RELION working folder, create a rawdata folder containing soft links to the frames and .mdoc (in the same directory) and the gainref(s) e.g:

```
ln -s ~/pool-visprot/Florent/Folder_Newpipeline/Frames/Set_Cytoribo/ ~/pool-visprot/Florent/Folder_Newpipeline/Tomo_reconstruction/RELION5/Cytoribosome_5.11.24_GOOD/rawdata
 
ln -s ~/pool-visprot/Florent/Folder_Newpipeline/Frames/Gain_refs/ ~/pool-visprot/Florent/Folder_Newpipeline/Tomo_reconstruction/RELION5/Cytoribosome_5.11.24_GOOD/rawdata

```

1. Create a **rawdata** folder in your RELION working directory.
2. Copy or symlink the `.eer/.tiff` frames, `.mdoc` files, and gain references into **rawdata**.
3. Use the **Import** job in RELION to point to these raw frames and `.mdoc` files.

Similarly to Scipion, if you are dealing with a dataset where tilt-series were collected with different Gain references, you should import them individually, grouped by gainrefs. 
In the case of the cytoribosome dataset, out of the 33 tomos, 6 tomos were collected with one gainref and 27 with another, so we split them in two different groups. You would have to run them as separate jobs until you go to STA.

You need to know the tilt axis and the defocus handedness for RELION (something Scipion didn't ask you for!). If you get the defocus handedness wrong, this will only have an effect during STA, especially when reaching high resolution. If you want to check it, you can use **[Ricardo's Defocusgrad](https://github.com/CellArchLab/cryoet-scripts/tree/main)**. With this dataset, _Invert defocus handedness_ should be set as **YES (-1)**.

You also need to know the handedness of you tomos. 
In our case they are flipped, so to get the proper hand, we need to import them with an inverted tilt axis. Instead of 95 we use -95.

Both defocus handedness and tomo handedness can be checked once the tomos are reconstructed, not before! If you don't know anything about your data, start by reconstructing 2 or 3 of them and check that first before trying to batch process over 100 tomos.
Be aware that even if data were collected on the same microscope, updates on the camera can result on flipped handedness. Data collected within the same session should be all the same.

<img src="/imgs/14_Import1.JPG" alt="Processing Workflow" style="width:70%;">
<img src="/imgs/15_Import2.JPG" alt="Processing Workflow" style="width:70%;">


TBD




