---
layout: default
title: "Preprocessing in AreTomo3"
parent: "Tutorial"
nav_order: 2
---

# Preprocessing in AreTomo3

[AreTomo3](https://github.com/czimaginginstitute/AreTomo3) is a software which can take you from raw frames/movies to 3D-CTF corrected tomograms in under 10 minutes. If you are not familiar with AreTomo3 you can read the [publication](https://www.biorxiv.org/content/10.1101/2025.03.11.642690v1) as well as the [user manual](https://github.com/czimaginginstitute/AreTomo3/tree/main/docs). Below we will show you how to run AreTomo3 using a SLURM submission:

```bash
#!/bin/bash
#SBATCH -o aretomo3.out%j
#SBATCH -D ./
#SBATCH -J aretomo3
#SBATCH --partition=emgpu
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=4
#SBATCH --gres=gpu:1
#SBATCH --mail-type=none
#SBATCH --mem 64G
#SBATCH --qos=emgpu
#SBATCH --time=00-00:30:00
 
files="path/to/frames" # All frames with mdocs in this folder will be processed. Can also specify only one Position_X tilt series.
gain_ref="GainReference.gain"
outdir=aretomo_output
apix=1.91
tilt_axis=-95.75
dark_tol=0.7
vol_z=2048
amp_con=0.1
fm_dose=0.14 # dose per frame! so 3.5 e/A² per 25 slices here
lowpass=15
# align_z=1400 # Not really needed since AreTomo3 can estimate it automatically
 
ml purge
ml IMOD
ml AreTomo3
 
mkdir -p ${outdir}
 
time AreTomo3 \
    -InPrefix "${files}" \
    -Insuffix ".mdoc" \
    -Gain "${gain_ref}" \
    -OutDir "${outdir}" \
    -FlipGain 1 \
    -Gpu ${CUDA_VISIBLE_DEVICES} \
    -PixSize ${apix} \
    -McBin 1 \
    -McPatch 1 1 \
    -FmInt 12 \
    -FmDose ${fm_dose} \
    -SplitSum 1 \
    -VolZ ${vol_z} \
    -TiltAxis ${tilt_axis} \
    -AtBin 4 \
    -OutXF 1 \
    -OutImod 1 \
    -Wbp 1 \
    -FlipVol 1 \
    -TiltCor 1 \
    -Patch 0 0 \
    -CorrCTF 1 \
    -DarkTol ${dark_tol} \
    -CorrCTF 1 ${lowpass} \
    -Kv 300 \
    -Cs 2.7 \
    -AmpContrast ${amp_con}
```

Make sure you check the meaning of all the flags with `AreTomo3 --help`. Important things to get right are `FmInt` (default `15`) which corresponds how you want your eer frames to be grouped **[check this](/03-tutorial/01-scipion-preprocessing/#motion-correction/)** and `FmDose` is defined as **dose per frame**! Here for example **3.5 e/Å² per 25 frames** would result in the `FmDose=0.14`. This depends on your EER grouping - in TIFF format this will change and `FmInt 1` should be used.
{: .note }

The way the command works now is that it will find all frames corresponding to their .mdocs in the given path. **Thus, make sure your mdocs and frames are in the same folder!** The tomograms will be reconstructed with CTF correction in bin4 and also ODD+EVEN volumes are generated. Those can be used for denoising for example. If you don't use SLURM you can just adopt the above command and submit/run it your way. Additionally, AreTomo3 will also output alignment information (e.g. IMOD format).

## Creating a RELION5 project

Since AreTomo3 directly output alignment and CTF estimation/correction, we can use this for [template matching](/03-tutorial/04-template-matching/#at3tm) or [subtomogram averaging](/03-tutorial/05-sta-in-relion5/). Here, we provide a **[script](https://github.com/Phaips/aretomo3torelion5)** allwoing you to go directly from AreTomo3 to RELION5. It reads all the AreTomo3 output informations and writes a `tomogram.star` and corresponding `tilt-series.star` file in RELION5 format. To simply create the RELION5 files for all tomograms in the AreTomo3 directory you can run:

```python
aretomo3torelion5.py /path/to/aretomo_output/ --dose 3.5
```
We included the `--dose` flag since there is -- to our knowledge -- no way to read this information from the AreTomo3 outputs. AreTomo3 itself reads the exposure informations from the .mdocs, which could be an issue, since .mdoc files often do not contain correct values unfortunately! That is why we decided to just prompt the user for this value and calculate the **cumulative dose/exposure** based on the acquisition parameters.

If you have successfully generated your `.star` files you can run a `relion_tomo_reconstruct_tomogram_mpi` job using the `tomogram.star` file to check the alignments are correct. Here an example in bin4:

```
relion_tomo_reconstruct_tomogram_mpi --t tomograms.star --o Tomograms/job001/ --w 4096 --h 4096 --d 2048 --binned_angpix  --only_do_unfinished  --j 12 --Fc --SNR 100 --pre_weight --ctf --pipeline_control Tomograms/job001/
```

Or you can directly follow the [STA in RELION5](/03-tutorial/05-sta-in-relion5/) once you have a particle list from e.g. [template matching](/03-tutorial/04-template-matching/).






