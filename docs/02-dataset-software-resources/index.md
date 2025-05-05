---
layout: default
title: "Datasets, Software, and Resources"
nav_order: 8
---

# Datasets, Software, and Resources

## **Dataset: EMPIAR-11830**
For this tutorial, we compiled a subset of **33** tomograms from the **Chlamy dataset** (EMPIAR-11830), originally published in as R. Kelley, et al, Towards community-driven visual proteomics with large-scale cryo-electron tomography of Chlamydomonas reinhardtii [bioRxiv Preprint (2024)](https://doi.org/10.1101/2024.12.28.630444).

**General info about the dataset:**

- **Detector:** Falcon4i with SelectrisX energy filter, using Tomo5 on a generation 4 Titan Krios(es)  
- **Pixel size:** 1.91 (microscope defined 1.96, calibrated by STA to be 1.91)  
- **Voltage:** 300  
- **Spherical aberration:** 2.7  
- **Tilt axis:** -95, be aware that the tilt axis angles indicated in the mdocs are usually wrong
- **Defocus handedness:** -1 in RELION if starting from scratch (+1 if using the TOMOMAN preprocessed project)  
- **Dose:** 3.5 e-/A² per tilt

Although reconstructed tomograms from this dataset are already available (processed using TOMOMAN and automatically aligned with AreTomo), this tutorial is designed to guide you through the full tomogram reconstruction workflow from scratch, leading into **subtomogram averaging (STA)**.

This subset may also serve as a useful benchmark for testing and comparing different software tools.

The 33 tilt series are divided into two groups:
- 6 tilt series using GainRef1
- 27 tilt series using GainRef2

If you're just getting started and want to learn the fundamentals of tomogram reconstruction and STA, we recommend beginning with the **6 GainRef1 tilt series** for faster processing.

If you're interested in pushing resolution, trying classification, or running advanced workflows, you can process the full set. GainRef1 alone is about 33 Gb while GainRef1 and 2 is about 170Gb.

You can download the datasets directly from EMPIAR. We compiled some scripts to help you in that process.

From a Linux terminal, in your desired directory, run the following command to download the 6 tilt series associated with **GainRef1:**

Make the script executable:

```bash
chmod +x bash_download_gain1.sh 
./bash_download_gain1.sh
```
and run `./bash_download_gain1.sh`:

<details>
  <summary><strong>bash_download_gain1.sh</strong></summary>
  <pre><code class="bash">
#!/bin/bash

# Base URL
BASE_URL="ftp://ftp.ebi.ac.uk/empiar/world_availability/11830/data/chlamy_visual_proteomics"

# List of gain1 target entries
ENTRIES=(
  "06042022_BrnoKrios_Arctis_grid5_Position_12"
  "02122021_BrnoKrios_Arctis_lam2_pos2"
  "02122021_BrnoKrios_Arctis_lam1_pos8"
  "02122021_BrnoKrios_Arctis_lam1_pos6"
  "01122021_BrnoKrios_arctis_lam3_pos31"
  "01122021_BrnoKrios_arctis_lam3_pos29"
  "01122021/gainref"
)

for ENTRY in "${ENTRIES[@]}"; do
  wget -r -N -np -nH --cut-dirs=4 \
       --accept "*.eer,*.mdoc,*.gain" \
       "$BASE_URL/$ENTRY/"
done
  </code></pre>
</details>


To download the 27 tilt series associated with **GainRef2**:

<details>
  <summary><strong>bash_download_gain2.sh</strong></summary>
  <pre><code class="bash">
#!/bin/bash

# Base URL
BASE_URL="ftp://ftp.ebi.ac.uk/empiar/world_availability/11830/data/chlamy_visual_proteomics"

# List of gain2 target entries
ENTRIES=(
    "06042022_BrnoKrios_Arctis_grid7_Position_19"
    "06042022_BrnoKrios_Arctis_grid7_Position_21"
    "06042022_BrnoKrios_Arctis_grid7_Position_24"
    "15042022_BrnoKrios_Arctis_grid9_Position_25"
    "15042022_BrnoKrios_Arctis_grid9_Position_32"
    "15042022_BrnoKrios_Arctis_grid9_Position_35"
    "15042022_BrnoKrios_Arctis_grid9_Position_65"
    "27042022_BrnoKrios_Arctis_grid9_hGIS_Position_12"
    "27042022_BrnoKrios_Arctis_grid9_hGIS_Position_15"
    "27042022_BrnoKrios_Arctis_grid9_hGIS_Position_25"
    "27042022_BrnoKrios_Arctis_grid9_hGIS_Position_29"
    "27042022_BrnoKrios_Arctis_grid9_hGIS_Position_42"
    "27042022_BrnoKrios_Arctis_grid9_hGIS_Position_44"
    "27042022_BrnoKrios_Arctis_grid9_hGIS_Position_51"
    "02052022_BrnoKrios_Arctis_grid_hGIS_Position_15"
    "02052022_BrnoKrios_Arctis_grid_hGIS_Position_19"
    "02052022_BrnoKrios_Arctis_grid_hGIS_Position_47"
    "02052022_BrnoKrios_Arctis_grid_hGIS_Position_49"
    "02052022_BrnoKrios_Arctis_grid_hGIS_Position_59"
    "02052022_BrnoKrios_Arctis_grid_hGIS_Position_63"
    "02052022_BrnoKrios_Arctis_grid_hGIS_Position_64"
    "02052022_BrnoKrios_Arctis_grid_hGIS_Position_77"
    "02052022_BrnoKrios_Arctis_grid_hGIS_Position_78"
    "02052022_BrnoKrios_Arctis_grid_hGIS_Position_80"
    "02052022_BrnoKrios_Arctis_grid_hGIS_Position_83"
    "12052022_BrnoKrios_Arctis_grid_newGISc_Position_13"
    "12052022_BrnoKrios_Arctis_grid_newGISc_Position_29"
    "06042022/gainref"
)

# Loop over each entry and download only .eer and .mdoc files
for ENTRY in "${ENTRIES[@]}"; do
    wget -r -N -np -nH --cut-dirs=4 \
         --accept "*.eer,*.mdoc,*.gain" \
         "$BASE_URL/$ENTRY/"
done
  </code></pre>
</details>

From there, because the original files have annoying names, run this `prepare_clean_rename.sh` script. It will clean, rename and organize the files. (Run the script the same way you did for the download part):

<details>
  <summary><strong>prepare_clean_rename.sh</strong></summary>
  <pre><code class="bash">
#!/bin/bash

BASE_DIR="chlamy_visual_proteomics"

GAIN1=(
    "06042022_BrnoKrios_Arctis_grid5_Position_12"
    "02122021_BrnoKrios_Arctis_lam2_pos2"
    "02122021_BrnoKrios_Arctis_lam1_pos8"
    "02122021_BrnoKrios_Arctis_lam1_pos6"
    "01122021_BrnoKrios_arctis_lam3_pos31"
    "01122021_BrnoKrios_arctis_lam3_pos29"
)

GAIN2=(
    "06042022_BrnoKrios_Arctis_grid7_Position_19"
    "06042022_BrnoKrios_Arctis_grid7_Position_21"
    "06042022_BrnoKrios_Arctis_grid7_Position_24"
    "15042022_BrnoKrios_Arctis_grid9_Position_25"
    "15042022_BrnoKrios_Arctis_grid9_Position_32"
    "15042022_BrnoKrios_Arctis_grid9_Position_35"
    "15042022_BrnoKrios_Arctis_grid9_Position_65"
    "27042022_BrnoKrios_Arctis_grid9_hGIS_Position_12"
    "27042022_BrnoKrios_Arctis_grid9_hGIS_Position_15"
    "27042022_BrnoKrios_Arctis_grid9_hGIS_Position_25"
    "27042022_BrnoKrios_Arctis_grid9_hGIS_Position_29"
    "27042022_BrnoKrios_Arctis_grid9_hGIS_Position_42"
    "27042022_BrnoKrios_Arctis_grid9_hGIS_Position_44"
    "27042022_BrnoKrios_Arctis_grid9_hGIS_Position_51"
    "02052022_BrnoKrios_Arctis_grid_hGIS_Position_15"
    "02052022_BrnoKrios_Arctis_grid_hGIS_Position_19"
    "02052022_BrnoKrios_Arctis_grid_hGIS_Position_47"
    "02052022_BrnoKrios_Arctis_grid_hGIS_Position_49"
    "02052022_BrnoKrios_Arctis_grid_hGIS_Position_59"
    "02052022_BrnoKrios_Arctis_grid_hGIS_Position_63"
    "02052022_BrnoKrios_Arctis_grid_hGIS_Position_64"
    "02052022_BrnoKrios_Arctis_grid_hGIS_Position_77"
    "02052022_BrnoKrios_Arctis_grid_hGIS_Position_78"
    "02052022_BrnoKrios_Arctis_grid_hGIS_Position_80"
    "02052022_BrnoKrios_Arctis_grid_hGIS_Position_83"
    "12052022_BrnoKrios_Arctis_grid_newGISc_Position_13"
    "12052022_BrnoKrios_Arctis_grid_newGISc_Position_29"
)

declare -A RENAME_MAP=(
    ["01122021_BrnoKrios_arctis_lam3_pos29"]="tomo24"
    ["01122021_BrnoKrios_arctis_lam3_pos31"]="tomo25"
    ["02122021_BrnoKrios_Arctis_lam1_pos6"]="tomo34"
    ["02122021_BrnoKrios_Arctis_lam1_pos8"]="tomo35"
    ["02122021_BrnoKrios_Arctis_lam2_pos2"]="tomo37"
    ["06042022_BrnoKrios_Arctis_grid5_Position_12"]="tomo50"
    ["06042022_BrnoKrios_Arctis_grid7_Position_19"]="tomo69"
    ["06042022_BrnoKrios_Arctis_grid7_Position_21"]="tomo71"
    ["06042022_BrnoKrios_Arctis_grid7_Position_24"]="tomo74"
    ["15042022_BrnoKrios_Arctis_grid9_Position_25"]="tomo216"
    ["15042022_BrnoKrios_Arctis_grid9_Position_32"]="tomo224"
    ["15042022_BrnoKrios_Arctis_grid9_Position_35"]="tomo227"
    ["15042022_BrnoKrios_Arctis_grid9_Position_65"]="tomo260"
    ["27042022_BrnoKrios_Arctis_grid9_hGIS_Position_12"]="tomo297"
    ["27042022_BrnoKrios_Arctis_grid9_hGIS_Position_15"]="tomo300"
    ["27042022_BrnoKrios_Arctis_grid9_hGIS_Position_25"]="tomo310"
    ["27042022_BrnoKrios_Arctis_grid9_hGIS_Position_29"]="tomo314"
    ["27042022_BrnoKrios_Arctis_grid9_hGIS_Position_42"]="tomo329"
    ["27042022_BrnoKrios_Arctis_grid9_hGIS_Position_44"]="tomo331"
    ["27042022_BrnoKrios_Arctis_grid9_hGIS_Position_51"]="tomo339"
    ["02052022_BrnoKrios_Arctis_grid_hGIS_Position_15"]="tomo355"
    ["02052022_BrnoKrios_Arctis_grid_hGIS_Position_19"]="tomo359"
    ["02052022_BrnoKrios_Arctis_grid_hGIS_Position_47"]="tomo378"
    ["02052022_BrnoKrios_Arctis_grid_hGIS_Position_49"]="tomo380"
    ["02052022_BrnoKrios_Arctis_grid_hGIS_Position_59"]="tomo391"
    ["02052022_BrnoKrios_Arctis_grid_hGIS_Position_63"]="tomo396"
    ["02052022_BrnoKrios_Arctis_grid_hGIS_Position_64"]="tomo397"
    ["02052022_BrnoKrios_Arctis_grid_hGIS_Position_77"]="tomo411"
    ["02052022_BrnoKrios_Arctis_grid_hGIS_Position_78"]="tomo412"
    ["02052022_BrnoKrios_Arctis_grid_hGIS_Position_80"]="tomo415"
    ["02052022_BrnoKrios_Arctis_grid_hGIS_Position_83"]="tomo418"
    ["12052022_BrnoKrios_Arctis_grid_newGISc_Position_13"]="tomo423"
    ["12052022_BrnoKrios_Arctis_grid_newGISc_Position_29"]="tomo440"
)

clean_directory() {
    local entry_path="$1"
    echo "Cleaning $entry_path"
    rm -rf "$entry_path"/{AreTomo,ctffind4,metadata,tiltctf}
    if [ -d "$entry_path/frames" ]; then
        mv "$entry_path"/frames/*.eer "$entry_path/" 2>/dev/null
        rmdir "$entry_path/frames" 2>/dev/null
    fi
}

# Step 1: Clean all folders
for entry_path in "$BASE_DIR"/*/; do
    clean_directory "$entry_path"
done

# Step 2: Make gain directories
mkdir -p "$BASE_DIR/gain1" "$BASE_DIR/gain2"

# Step 3: Move gain1 folders
for folder in "${GAIN1[@]}"; do
    if [ -d "$BASE_DIR/$folder" ]; then
        mv "$BASE_DIR/$folder" "$BASE_DIR/gain1/"
    fi
done

# Step 4: Copy overlapping gain1 folders into gain2, move the rest
for folder in "${GAIN2[@]}"; do
    if [ -d "$BASE_DIR/gain1/$folder" ]; then
        cp -r "$BASE_DIR/gain1/$folder" "$BASE_DIR/gain2/"
    elif [ -d "$BASE_DIR/$folder" ]; then
        mv "$BASE_DIR/$folder" "$BASE_DIR/gain2/"
    fi
done

# Step 5: Copy and rename gain references
GAINREF1_SRC="$BASE_DIR/01122021/gainref"
GAINREF2_SRC="$BASE_DIR/06042022/gainref"

GAINREF1_FILE=$(find "$GAINREF1_SRC" -type f -name "*.gain" | head -n 1)
GAINREF2_FILE=$(find "$GAINREF2_SRC" -type f -name "*.gain" | head -n 1)

if [ -f "$GAINREF1_FILE" ]; then
    cp "$GAINREF1_FILE" "$BASE_DIR/gain1/gainref1.gain"
    echo "→ gainref1.gain copied to gain1/"
    rm -rf "$BASE_DIR/01122021"
else
    echo "No .gain file found in $GAINREF1_SRC"
fi

if [ -f "$GAINREF2_FILE" ]; then
    cp "$GAINREF2_FILE" "$BASE_DIR/gain2/gainref2.gain"
    echo "→ gainref2.gain copied to gain2/"
    rm -rf "$BASE_DIR/06042022"
else
    echo "No .gain file found in $GAINREF2_SRC"
fi

# Step 6: Rename folders and files
for GAIN_DIR in "$BASE_DIR/gain1" "$BASE_DIR/gain2"; do
    for original_name in "${!RENAME_MAP[@]}"; do
        new_name="${RENAME_MAP[$original_name]}"
        src_folder="$GAIN_DIR/$original_name"
        dest_folder="$GAIN_DIR/$new_name"

        if [ -d "$src_folder" ]; then
            echo "Renaming folder: $original_name → $new_name"
            mv "$src_folder" "$dest_folder"

            # Rename .mdoc
            old_mdoc=$(find "$dest_folder" -maxdepth 1 -name "*.mdoc" | head -n 1)
            if [ -f "$old_mdoc" ]; then
                mv "$old_mdoc" "$dest_folder/${new_name}.mdoc"
                echo "Renamed .mdoc to ${new_name}.mdoc"
            fi

            # Rename all .eer files
            for eer_file in "$dest_folder"/*.eer; do
                if [[ -f "$eer_file" ]]; then
                    angle=$(echo "$eer_file" | sed -E 's/.*\[(.*)\]_EER\.eer/\1/')
                    mv "$eer_file" "$dest_folder/${new_name}[${angle}]_EER.eer"
                    echo "Renamed .eer to ${new_name}[${angle}]_EER.eer"
                fi
            done
        fi
    done
done

echo "Everything is cleaned, organized, and renamed."
  </code></pre>
</details>

Finally run the `python_organise.py` script below. It will modify the .mdoc and create gain1_link and gain2_link folders that contains soft links to .eer and .mdoc.

Make sure you have python loaded and run:
```bash
python python_organise.py chlamy_visual_proteomics
```


<details>
  <summary><strong>prepare_clean_rename.sh</strong></summary>
  <pre><code class="python">
import os
import re
import sys

def parse_mdoc(mdoc_path):
    subframe_pattern = re.compile(r"SubFramePath\s*=\s*(.+)")
    bracket_pattern = re.compile(r"\[(.+?)\]")

    mdoc_data = []
    subframe_paths = []

    with open(mdoc_path, 'r') as mdoc_file:
        for line in mdoc_file:
            mdoc_data.append(line)
            match = subframe_pattern.match(line.strip())
            if match:
                old_path = match.group(1).strip()
                bracket_value = bracket_pattern.search(old_path)
                if bracket_value:
                    subframe_paths.append((old_path, bracket_value.group(1)))
    return mdoc_data, subframe_paths

def find_matching_files(folder_path):
    bracket_pattern = re.compile(r"\[(.+?)\]")
    eer_files = []

    for filename in os.listdir(folder_path):
        if filename.endswith(".eer"):
            match = bracket_pattern.search(filename)
            if match:
                bracket_value = match.group(1)
                full_path = os.path.join(folder_path, filename)
                eer_files.append((filename, bracket_value, full_path))
    return eer_files

def update_mdoc_in_place(mdoc_path, mdoc_data, subframe_paths, eer_files):
    updated_data = []
    for line in mdoc_data:
        updated_line = line
        for old_path, bracket_value in subframe_paths:
            if old_path in line:
                matching_files = [file for file in eer_files if file[1] == bracket_value]
                if matching_files:
                    new_filename = matching_files[0][2]
                    updated_line = line.replace(old_path, new_filename)
                break
        updated_data.append(updated_line)

    with open(mdoc_path, 'w') as mdoc_file:
        mdoc_file.writelines(updated_data)

def process_folder(folder_path):
    eer_files = find_matching_files(folder_path)

    for filename in os.listdir(folder_path):
        if filename.endswith(".mdoc"):
            mdoc_path = os.path.join(folder_path, filename)
            mdoc_data, subframe_paths = parse_mdoc(mdoc_path)
            update_mdoc_in_place(mdoc_path, mdoc_data, subframe_paths, eer_files)
            print(f"Updated: {mdoc_path}")

def create_symlinks(base_dir, gain_name):
    gain_path = os.path.join(base_dir, gain_name)
    links_dir = os.path.join(os.path.dirname(base_dir), f"{gain_name}_links")

    os.makedirs(links_dir, exist_ok=True)

    for folder in os.listdir(gain_path):
        folder_path = os.path.join(gain_path, folder)
        if not os.path.isdir(folder_path):
            continue
        for file in os.listdir(folder_path):
            if file.endswith(".eer") or file.endswith(".mdoc"):
                target_file = os.path.join(folder_path, file)
                link_path = os.path.join(links_dir, file)

                # Create relative path for symlink
                rel_target = os.path.relpath(target_file, links_dir)

                try:
                    if os.path.exists(link_path) or os.path.islink(link_path):
                        os.remove(link_path)
                    os.symlink(rel_target, link_path)
                    print(f"Linked: {file} → {rel_target}")
                except Exception as e:
                    print(f"Failed to link {file}: {e}")

def main(base_dir):
    for gain_dir in ['gain1', 'gain2']:
        full_gain_path = os.path.join(base_dir, gain_dir)
        if not os.path.isdir(full_gain_path):
            continue
        for entry in os.listdir(full_gain_path):
            entry_path = os.path.join(full_gain_path, entry)
            if os.path.isdir(entry_path):
                process_folder(entry_path)
        create_symlinks(base_dir, gain_dir)

if __name__ == "__main__":
    if len(sys.argv) != 2:
        print("Usage: python update_subframe_paths.py <chlamy_visual_proteomics>")
    else:
        main(sys.argv[1])
  </code></pre>
</details>

At the end you should have a folder named `chlamy_visual_proteomics` containing fours folders. `gain1` and `gain2` folders contains the raw data and the gain references, `gain1_links` and `gain2_links` contain links to the `.eer` and the `.mdoc` in a single folder.

<a href="/imgs/40_gain1.png" data-lightbox="image-gallery">
  <img src="/imgs/40_gain1.png" alt="Processing Workflow" style="width:50%;">
</a>
<a href="/imgs/40_gain2.png" data-lightbox="image-gallery">
  <img src="/imgs/40_gain2.png" alt="Processing Workflow" style="width:50%;">
</a>

Additionally you can directly download these files here:

- Two text files with thickness measurements for automated AreTomo TS alignment: https://github.com/TomoGuide/TomoGuide.github.io/tree/main/docs/data/Z_height/
- Templates and masks for Template Matching: https://github.com/TomoGuide/TomoGuide.github.io/tree/main/docs/data/TM/

## Software

You need to have access to a GPU-powered machine running on Linux. It can be a local machine or a computing cluster. In our case, we work on a computing cluster with a SLURM system.
You will also need to have appropriate CUDA drivers (this means you need to have NVIDIA GPUs) and a Python installation.

Click on the buttons below to get more information about the main software used in that tutorial and download it:

[Scipion](https://scipion.i2pc.es/){: .btn } <br>
[IMOD](https://bio3d.colorado.edu/imod/){: .btn } <br>
[RELION 5](https://relion.readthedocs.io/en/release-5.0/){: .btn } <br>
[AreTomo3](https://github.com/czimaginginstitute/AreTomo3){: .btn } <br>
[ChimeraX](https://www.cgl.ucsf.edu/chimerax/){: .btn } <br>
[ArtiaX](https://github.com/FrangakisLab/ArtiaX){: .btn } <br>
[pytom-match-pick](https://github.com/SBC-Utrecht/pytom-match-pick){: .btn } <br>
