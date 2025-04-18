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

To run template matching in batch (on all the tomograms in your RELION folder) we will use this **[batch_pytom_wRln5](https://github.com/Phaips/batch_pytom_wRln5)** script we wrote. It is intended to create `bash.sh` files for SLURM submission.  It will read all necessary informations like the defocus per tilt and exposure values from the `tilt-series.star` files in your RELION `Tomograms/jobXXX/` folder. For each tomogram those values will be provided to the `pytom_match_template.py` function. 

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

Once you have succesfully run template matching, you can extract your particles with 

You can check the `_scores.mrc` file in IMOD for example to see if template matching was successful. If you open the `tomogram.mrc` and `_scores.mrc` at the same time you should see bright dots at the center of each of your particles of interest. 

If you want to visualize your particles even better then follow the next part:









