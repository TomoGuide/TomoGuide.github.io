---
layout: default
title: "STA in RELION5"
parent: "Tutorial"
nav_order: 4
---

# Subtomogram Averaging (STA) in RELION5

After you’ve reconstructed your tomograms in RELION5 (or exported them from Scipion), you can proceed with subtomogram 
averaging to reach higher resolution on repeated structures (like ribosomes, viral spikes, etc.).

> **(Placeholder for STA workflow diagram)**  
> `![STA Workflow Diagram](path/to/sta_workflow.png)`

## Steps in STA

1. **Particle Picking**  
   You identify coordinates of repeated particles in your tomograms (manually or via template matching).  

2. **Extraction**  
   Subvolumes are extracted from the tomograms at each coordinate.  

3. **Alignment & Classification**  
   RELION aligns these subvolumes in 3D space and classifies them by structural features.

4. **Averaging**  
   The aligned subvolumes are averaged to improve SNR and resolution.  

5. **Refinement & Validation**  
   Additional rounds of alignment, classification, and post-processing steps (masking, B-factor sharpening, etc.) may be 
   performed to push resolution.

---

## Defocus Handedness & Tomogram Orientation

Be sure your tomograms aren’t mirrored. If you see an inverted hand in your final 3D structure, revisit your tilt-axis sign or 
defocus handedness.  

## Exporting STA Results

Once your STA job finishes, you can analyze your final maps in ChimeraX or any 3D visualization tool. If needed, you can 
export back to Scipion or other software for more specialized tasks.

