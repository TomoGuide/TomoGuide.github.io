---
layout: default
title: "Preprocessing in AreTomo3"
parent: "Tutorial"
nav_order: 2
---

# Preprocessing in AreTomo3

[AreTomo3](https://github.com/czimaginginstitute/AreTomo3) is a software which can take you from raw frames/movies to 3D-CTF corrected tomograms in under 10 minutes. If you are not familiar with AreTomo3 you can also read their latest [publication](https://www.biorxiv.org/content/10.1101/2025.03.11.642690v1). Below we will show you how to AreTomo3 using a SLURM submission:

```bash
#!/bin/bash
# Standard output and error:
#SBATCH -o aretomo3.out%j
# Initial working directory:
#SBATCH -D ./
# Job Name:
#SBATCH -J aretomo3
# Queue (Partition):
#SBATCH --partition=emgpu
# Number of nodes and MPI tasks per node:
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=4
#SBATCH --gres=gpu:1
#
#SBATCH --mail-type=none
#SBATCH --mem 64G
#
# Wall clock limit:
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
fm_dose=0.2 # dose per frame! so 2 e/A^2 per 10 frames here
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
    -FmInt 1 \
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

Make sure you check the meaning of all the flags with `AreTomo3 --help`.
How this command now works is that it will find all frames corresponding to their .mdocs. The tomograms will be reconstructed with CTF correction and also ODD and EVEN frames are generated. Those can be used for denoising for example. Again, if you don't use SLURM you can just adopt the command and submit/run it your way. Additionally, AreTomo3 will output many files with useful alignment information (e.g. IMOD format).

## Creating a RELION5 project

Since the alignment and CTF estimation/correction from AreTomo3 seems quite powerful we can use this for [template matching](/03-tutorial/04-template-matching/#at3tm) or [subtomogram averaging](/03-tutorial/05-sta-in-relion5/). 




