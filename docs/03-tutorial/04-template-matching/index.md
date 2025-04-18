---
layout: default
title: "Particle Picking"
parent: "Tutorial"
nav_order: 4
---

# Particle Picking by Template Matching

## Create boundary masks of your tomograms

To restrict particle extraction (or detection) to your lamella only, and not the entire tomogram, you might want to use a "slab mask" or a boundary mask. This is particularly useful on difficult targets no minimize the amount of false positive picks.

In our case with ribosomes, a slab mask is actually not really necessary.
You can automatically create a boundary masks using [Slabify](https://github.com/CellArchLab/slabify-et). This works best on denoised tomograms. For detailed instruction check the Slabify [Wiki](https://github.com/CellArchLab/slabify-et/wiki).

If you know where you particle should be (e.g mitochondria, nucleus etc) you can further restrict your search by creating appropriate masks with software like [Amira](https://www.thermofisher.com/ch/en/home/electron-microscopy/products/software-em-3d-vis/amira-software/cell-biology.html?cid=msd_vds_ls_none_amr_123456_gl_pso_gaw_escchz&gad_source=1&gbraid=0AAAAADxi_GQP_HTwG5UvF75JpF-4AyxPP&gclid=Cj0KCQjwzYLABhD4ARIsALySuCSqID5sMy5zoaTDmyL_ScEreSQ_k5zh6D2_RKo_Dhr_trSPq0WOU7caAjPIEALw_wcB), [Napari](https://napari.org/stable/), [MITK](https://github.com/MITK/MITK) - or using automated membrane segmentations tools like [MemBrain-seg](https://github.com/teamtomo/membrain-seg). 

<a href="/imgs/35_slab.png" data-lightbox="image-gallery">
  <img src="/imgs/35_slab.png" alt="Processing Workflow" style="width:60%;">
</a>

You can make a slab_mask folder inside the tomograms folder (e.g RELION5/Tomograms/jobXXX/tomograms/) for example.


## Use pytom-match-pick for template matching

Make sure you have [pytom-match-pick](https://github.com/SBC-Utrecht/pytom-match-pick) installed. Set up a template matching folder in root RELION working directory (e.g RELION5/project/template_matching).

You will need to have:
- A **template** at the correct pixel size. Here, we first use a SPA structure with matching pixel size, correct box size, and inverted contrast (black density on white background)
- A **template mask** with the same properties. It can also just be a sphere but it is not recommended here.

For more details check the [Documentation](https://sbc-utrecht.github.io/pytom-match-pick/). Generally, we recommend to use a template that was generated from the data. This gives (much) better results. Hence, template matching is also an iterative process. For challenging targets you want to rerun everything once you obtain a "low-res" structure from your data.

### Scaling a template using RELION or EMAN

Two examples of how to scale an `input.mrc` file from the [EMDB](https://www.ebi.ac.uk/emdb/) for example using either `e2proc3d.py` from [EMAN2](https://blake.bcm.edu/emanwiki/EMAN2) or `relion_image_handler` from [RELION](https://github.com/3dem/relion/tree/ver5.0):

```
e2proc3d.py input.mrc output.mrc --scale=0.2 --process=filter.lowpass.gauss:cutoff_freq=0.1 --clip=84,84,84
```
```
relion_image_handler --i input.mrc --o output.mrc --rescale_angpix 7.64 --new_box 84 --force_header_angpix 7.64
```

- **`--scale=0.2`**: Rescale the pixel size by a facor of input/output so here 5 for example.
- **`process=filter.lowpass.gauss:cutoff_freq`**: Lowpass filter as 1/Resolution. Here 10 Å for example. Best to just filter to the binnings Nyquist resolution for "high-res TM".
- **`new_box`** or **`clip`**: Boxsize (in pixels)
- **`force_header_angpix`**: There might be some discrepancy between the desired and actual pixel size from the way RELION scales the image. We force the header to the desired pixel size.

To run template matching in batch (on all the tomograms in your RELION folder) we will use this **[batch_pytom_wRln5](https://github.com/Phaips/batch_pytom_wRln5)** script we wrote. It is intended to create `bash.sh` files for SLURM submission.  It will read all necessary informations like the defocus per tilt and exposure values from the `tilt-series.star` files in your RELION `Tomograms/jobXXX/` folder. For each tomogram those values will be provided to the `pytom_match_template.py` script. 

Additionally, for ribosomes good results were obtained using:

- Angular search: **10°**
- Enable random-phase correction: **YES**
- Enable per-tilt-weighting: **YES**
- Are the tomograms CTF corrected: **YES**
- Enable non-spherical mask: **YES**
- High-pass filter: **400**

An example how to run the script:

```
python batch_pytom.py \
  --mrc-dir path/to/Tomograms/jobXXX/tomograms \
  --star-dir /path/to/Tomograms/jobXXX/ \
  -t /path/to/template.mrc \
  -m /path/to/mask.mrc \
  [--particle-diameter 140 | --angular-search 7] \ # either or - here --angular-search 10 is good
  -s 2 2 1 \
  --voxel-size 7.64 \
  -g 0 \
  --random-phase-correction \
  --rng-seed 69 \
  --per-tilt-weighting \
  --non-spherical-mask \
  --tomogram-ctf-model phase-flip \
  --high-pass 400 \
  --tomogram-mask path/to/slab_mask.mrc # if you have one \
  [--dry-run] # to just create the submission scripts without launching them
```

> Note: many flags are set to default values like the voltage **300** kV, amplitude contrast **0.07**, and SLURM settings like the partition **emgpu** (name of our SLURM partition). So make sure you check `batch_pytom.py --help` for informations about the input flags. 

An example output of one of the generated submission files might look like this:

```
#!/bin/bash -l
#SBATCH -o pytom.out%j
#SBATCH -D ./
#SBATCH -J pytom_1
#SBATCH --partition=emgpu
#SBATCH --ntasks=1
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=4
#SBATCH --gres=gpu:1
#SBATCH --mail-type=none
#SBATCH --mem=128G
#SBATCH --qos=emgpu
#SBATCH --time=05:00:00

ml purge
ml pytom-match-pick

pytom_match_template.py \
  -v job017/tomograms/rec_Position_1.mrc \
  -a submission/tomo_1/1.tlt \                               # read from .star file
  --dose-accumulation submission/tomo_1/1_exposure.txt \     # read from .star file
  --defocus submission/tomo_1/1_defocus.txt \                # read from .star file
  -t tmpl.mrc \
  -d submission/tomo_1 \
  -m mask.mrc \
  --angular-search 10 \
  -s 2 2 1 \
  --voxel-size-angstrom 7.64 \
  -r \
  --rng-seed 69 \
  -g 0 \
  --amplitude-contrast 0.07 \
  --spherical-aberration 2.7 \
  --voltage 300 \
  --per-tilt-weighting \
  --tomogram-ctf-model phase-flip \
  --non-spherical-mask \
  --high-pass 400 \
  --tomogram-mask masks/bmask_1.mrc                           # from Slabify for example
```

and of course if you don't have an HPC or don't use SLURM, you can just run the regular [pytom-match-pick](https://sbc-utrecht.github.io/pytom-match-pick/) similar to the command above. You can run `batch_pytom.py` with the flag `--dry-run` in order to generate all the input commands and flags to then run it the way you like while still having all the tilt, defocus, and exposure informations read from the RELION .star files.

> Some numbers: ~40 min per subvolume (tomogram is split in 4) so 2.5 to 3h per tomo with 7° angular sampling at bin4 on rtx4090 node (we could have ask for more resources of course).
1.5h when you use the same parameters but a 10° (testing 15000 angles) angular sampling instead of 7° (testing 50000 angles). Random-phase correction will basically double the computation time but we recommend using it - especially for more challenging targets.

You can check the `_scores.mrc` file in IMOD for example to already see if template matching was successful. If you open the `tomogram.mrc` and `_scores.mrc` at the same time you should see bright dots at the center of each of your particles of interest. Later in this section, we'll show you how to visualize your particles first with IMOD, and then more appealingly using ChimeraX & ArtiaX.


## Extract particles

Once you have succesfully run template matching, you can extract your particles with `pytom_extract_candidates.py`. This will create the `particles.star` in RELION5 format to then use for [subtomogram averaging](/03-tutorial/05-sta-in-relion5/). Again for detailed explanation check the documentation or `--help` of the pytom script.

You can extract from multiple tomos for example via SLURM like:

```
#!/bin/bash -l
#SBATCH -o pytomextract.out%j
#SBATCH -D ./
#SBATCH -J pytom
#SBATCH --partition=emgpu
#SBATCH --ntasks=1
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=4
#SBATCH --gres=gpu:1
#SBATCH --mail-type=none
#SBATCH --mem 64G
#SBATCH --qos=emgpu
#SBATCH --time=06:00:00
 
ml purge
ml pytom-match-pick
 
pytom_extract_candidates.py -j submission/tomo_24/rec_tomo24_job.json -r 20 -n 800 -c -1 --relion5-compat
pytom_extract_candidates.py -j submission/tomo_25/rec_tomo25_job.json -r 20 -n 800 -c -1 --relion5-compat
pytom_extract_candidates.py -j submission/tomo_34/rec_tomo34_job.json -r 20 -n 800 -c -1 --relion5-compat
pytom_extract_candidates.py -j submission/tomo_35/rec_tomo35_job.json -r 20 -n 800 -c -1 --relion5-compat
```


Automatically select X best positions (determined cutoff by pytom) with a maximum number of 5000 particles:
```
pytom_extract_candidates.py -j submission/tomo_24/rec_tomo24_job.json -r 20 -n 5000 --relion5-compat
```

Force select the top 800 positions:
```
pytom_extract_candidates.py -j submission/tomo_24/rec_tomo24_job.json -r 20 -n 800 -c -1 --relion5-compat
```

or you can investigate the `.svg` file that was generated from the extraction job. Based on this you can tweak your `-c` value.


## Check your particle positions with IMOD

We wrote a script called **[rln2mod](https://github.com/Phaips/rln2mod)** which will create IMOD `.mod` point models from your `particle.star` files. You will need to have IMOD loaded since it will run `point2model`. The script will output `.mod` files for all `.star` files in the directory it is run from:

```
python rln2mod.py --x 1024 --y 1024 --z 512
```

Just give the tomogram dimensions in pixels as input. Then you can open your tomogram.mrc and .mod together in IMOD. A trick for better visualization is to go to: <kbd>Edit > Object > Type > Sphere radius for point</kbd> and increase this value! Here, an example from reconstruction in AreTomo with refined thickeness and with default pytom extraction parameters using a high-pass filter of 400. Shown in in IMOD XYZ mode (<kbd>ctrl + X</kbd>):

<a href="/imgs/36_mod.png" data-lightbox="image-gallery">
  <img src="/imgs/36_mod.png" alt="Processing Workflow" style="width:60%;">
</a>

A total of **546** positions were extracted. The particle picking is nearly perfect, with only a few false positives occurring on thylakoid membranes or chloroplastic ribosomes, which closely resemble cytosolic ribosomes. The boundary mask was applied for selection, you can see that nothing has been picked outside of the tomogram volume.

Common problems that can occur: 
- Particles extracted miss true positive particles: can be fixed by (slightly) increasing the number of particles `-n` and forcing this number with `-c -1`.
- Particles extracted include false positives: this is more often the case. Because membranes, ice contamination, or other high contrast object cross-correlate with a high score as well.

The latter might not pose a problem if you believe you can easily trash them though classification, in the later stages of **[STA](/03-tutorial/05-sta-in-relion5/)**.


You can select your positions with a mask only covering the cytosol, and excluding the chloroplast, instead of a simple boundary mask as we did. This is the best, but requires you to create masks, and can easily be tidieous when you work with tens or hundreds of tomos.
You can also try to play with the high-pass parameter at the TM step (we used a 400 HP by default, you can use smaller values, but be careful at some point you will lose true positives)
The number of particles that you want is always tricky to decide. Of course, you ideally want to pick everything.
In reality, if you want to pick everything, you will probably have to "overpick" and include false positives. Luckily you will be able to clean them out in the later stages.
If you underpick, there's high chances that you will minimise the number of false positives and will only select your particles of interest. This is useful when you want to quickly generate an average from your own data.
This is ALWAYS a good idea when doing template matching, because using a template generated from your data will always give better results than an template coming from e.g molmap in Chimera from a PDB (worst) or an SPA map.



Then, once you are satisfied with the TM results, you can generate a "master" .star file that will contain all the particle positions for all the tomos.

To do so, load pytom and use pytom_merge_stars.py , it will automatically merge all the .star files present in the directory where you execute the script.

Just be careful, when importing to RELION, remove "rec_" from the TomoName in the merged .star file ...










