# Winding-Detection Algorithm — Generation 1: Archimedean-Spiral Model

Jupyter notebook that follows the windings of a wound (jelly-roll) lithium-ion
cell in neutron-tomography slices and looks for discontinuities in them. It is
the first of the two algorithms described in the internship report
*Neutron Imaging and Tomography of Lithium Batteries* (Institut Laue-Langevin,
2026), Section 4.2 and Appendix A.

> **Status.** This is the first generation and has been superseded by the
> second ([`generation-2-geometry-independent`](../generation-2-geometry-independent)). It is published because the
> report analyses it in detail, and because its failure modes motivated the
> second generation. Its detections should be read as diagnostic output, not as
> a validated defect count (see *Limitations* below).

## What the notebook does

Each slice of a denoised tomography stack is processed independently:

1. **Geometry of the slice.** A rough centre and size of the cell are obtained
   from a thresholded mask. The pitch of the spiral is measured from the
   spacing of the intensity maxima along six radial rays, and the centre is
   refined by fitting the first harmonic of the radius of the bright outer
   ring around the whole circumference.
2. **Polar unwrapping.** The slice is resampled in polar coordinates and every
   row is shifted by `p·θ/2π`, so that, if the winding were an Archimedean
   spiral of constant pitch `p`, every winding would become a horizontal band.
   The unwrapped image is then binarized (background subtraction and Otsu
   thresholding in radial windows).
3. **Discontinuity detection.** The bands are analysed as connected components,
   with cyclic padding so that the 0°/360° seam is handled correctly.
   Interrupted bands are reported as **fissures** and merged bands as
   **fusions**; duplicates created by the padding are removed, and abnormally
   thick regions (such as the bright outer ring) are filtered out.
4. **Mapping.** The detections are drawn on the slice as ribbons that follow
   the spiral and are coloured by area, and they are exported for the whole
   volume.

The notebook also contains a **sanity check** that anchors the model spiral on
a real winding and measures how far the real windings depart from it, which is
how the limitations of the constant-pitch model were quantified in the report.

## The code is explained

The notebook is meant to be read as well as run. It opens with an
introduction, a description of the model and a list of its limitations, and
every code cell is preceded by a text cell that explains what the cell does
and why. The functions have docstrings, and the code is commented line by line
where the reasoning is not obvious.

| Phase | Cells | Content |
|-------|-------|---------|
| 1. Setup and geometry estimation | 1–4 | Imports and paths, circle detection, pitch measurement, centre refinement |
| 2. Polar unwrapping | 5, 5b | Unwrapping under the constant-pitch model; sanity check of the model against the real windings |
| 3. Discontinuity detection | 6–8 | Connected components, seam-duplicate removal, fissures and fusions per slice |
| 4. Visualization utilities | 9 | Drawing helpers (ribbons, polar ruler, angle axis) |
| 5. Full-volume batch analysis | 10–11 | Processing every slice of the stack, aggregation and export |
| 6. Quality control | 12–13 | Check of whether the detections cluster near the bright outer ring |

## Setup

**Requirements:** Python 3.10 or newer (developed with Python 3.12) and the packages below.

```bash
# 1. Get the code
git clone https://github.com/genariturri/winding-detection-algorithms.git
cd winding-detection-algorithms/generation-1-archimedean-spiral

# 2. Create and activate a virtual environment (recommended)
python -m venv .venv
source .venv/bin/activate          # on Windows: .venv\Scripts\activate

# 3. Install the dependencies
pip install numpy scipy scikit-image matplotlib pandas tifffile imageio jupyterlab

# 4. Start Jupyter and open the notebook
jupyter lab
```

## Input data

The tomography data are **not** included in this repository. The notebook
expects one multi-page TIFF file with the denoised slices of the cell, cropped
so that the cell is roughly centred in the image:

```
<TIFF_DIR>/denoised_stack_cropped_centered.tif
```

Before running, edit the paths in **Cell 1**:

```python
TIFF_DIR = "/path/to/your/tiffs_battery_sequential"   # folder that contains the stack
NUM_SLICES_TO_SAVE = 960                              # number of slices of the full stack
```

## Running

Run the cells in order (*Run → Run All Cells*). The batch analysis of Phase 5
processes every slice of the stack and saves four images per slice, so it can
take a while on a full volume. Interactive plotting is switched off in Cell 1
to keep the memory use of this loop under control; figures that are meant to
be seen are shown explicitly.

## Outputs

All outputs are written inside `TIFF_DIR`:

| Output | Content |
|--------|---------|
| `fissures_only/`, `fusions_only/` | Each slice with the fissures or the fusions drawn as ribbons coloured by area |
| `binarized_raw/` | The unwrapped, binarized image of each slice, as analysed by the algorithm |
| `binarized_annotated/` | The same with markers (known issue: the marker scaling is unreliable; use `binarized_raw/`) |
| `slice_geometry_log.csv` | Centre, radius and pitch measured in every slice |
| `discontinuities_summary_overview.csv` | Every detection, with its slice, type, radius, area and angular limits |
| `spiral_sanity_check/` | Figures and tables of the sanity check (Cell 5b) |

## Main parameters

The parameters were tuned on the dataset of the report and will probably need
to be adjusted for another cell or another pixel size:

| Parameter | Cell | Default | Meaning |
|-----------|------|---------|---------|
| `MIN_RELIABLE_RADIUS` | 8 | 300 px | Detections closer to the centre are discarded |
| `MAX_WIDTH_RATIO_VS_MEDIAN` | 8 | 1.5 | Regions this many times thicker than a typical winding are rejected |
| `min_area_fissure`, `min_area_fusion` | 8 | 50 px, 400 px | Smallest region reported as a fissure or as a fusion |
| `SANITY_SLICES` | 5b | 100, 192, … , 830 | Slices used in the sanity check |
| `MIN_DEFECT_AREA`, `MAX_DEFECT_AREA` | 10 | 50, 500 px | Limits of the colour scale of the defect maps |

## Limitations

The explicit spiral model is only as accurate as its two global parameters:
the centre must be known to within about half the spacing of the windings, and
the pitch to within a few per cent. When they are estimated less accurately,
the unwrapped windings are not horizontal, and the distortion produces
spurious fissures and fusions in many consecutive slices. Detections near the
centre are excluded, and the bright outer ring can still contaminate the
results. These issues, measured in the report, are the reason for the second
generation.

The detections have not been compared with an independent reference set of
defects, so the numbers of fissures and fusions should not be quoted as a
defect count.

## Citation

If you use this code, please cite the internship report:

> G. Iturri Bello, *Neutron Imaging and Tomography of Lithium Batteries: 3D
> Microstructural Analysis and a Geometry-Independent Spiral-Unwrapping
> Algorithm*, internship report, Institut Laue-Langevin, Grenoble (2026).

## Author

Genar Iturri Bello ([@genariturri](https://github.com/genariturri) on GitHub).
Contact details are given in the *Attribution* section at the top of the
notebook.
