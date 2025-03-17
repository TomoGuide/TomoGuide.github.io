---
layout: default
title: "Preprocessing in RELION5"
parent: "Tutorial"
nav_order: 3
---

# Preprocessing and tomogram reconstruction in RELION5

Below is how to import frames and reconstruct tomograms directly in RELION5.

---

## Import frames

1. Create a **rawdata** folder in your RELION working directory.
2. Copy or symlink the `.eer/.tiff` frames, `.mdoc` files, and gain references into **rawdata**.
3. Use the **Import** job in RELION to point to these raw frames and `.mdoc` files.

> **(Placeholder for screenshot: RELION import)**

If you have multiple gain references (like 6 TS with GainRef1 and 27 TS with GainRef2), you might import them separately 
until you reach the STA step.

### Tilt axis & defocus handedness

RELION also wants to know:
- **Tilt axis** sign (95 vs -95). If your real axis was +95° but the data is mirrored, you may need -95.  
- **Defocus handedness**. If you get it wrong, high-resolution refinements might be mirrored. For this dataset, 
  “Invert defocus handedness” is YES (-1).

If you’re unsure, reconstruct a couple tomograms first and check orientation visually.

---

## Tomogram reconstruction

Like Scipion, you’ll:
- Run **motion correction** on each tilt
- Do **tilt-series alignment** (RELION’s internal or AreTomo)
- Estimate **CTF**
- Assemble final tomograms (dose filtering, etc.)

Once done, confirm they’re not mirrored. If they are, you may need to adjust your tilt-axis sign. After that, you’re 
ready for [STA in RELION5](/08-tutorial/04-sta-in-relion5/).
