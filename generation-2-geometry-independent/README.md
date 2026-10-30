# Winding-Detection Algorithm — Generation 2: Geometry-Independent Model

Jupyter notebook that locates every turn of the wound electrode stack of a
lithium-ion cell in a neutron-tomography slice, without assuming any model of
the spiral. It is the second of the two algorithms described in the internship
report *Neutron Imaging and Tomography of Lithium Batteries* (Institut
Laue-Langevin, 2026), Section 4.3 and Appendix B.

> **Status.** The notebook locates the windings and traces each turn; it does
> **not** yet detect fissures or fusions. It has been verified visually on a few
> slices of one cell and has not yet been applied to a whole volume (see
> *Limitations* below).

## What the notebook does

Instead of predicting where each winding should be, as the [first generation](../generation-1-archimedean-spiral)
did with an Archimedean-spiral model, this notebook finds where each winding
is. Every slice is processed in four steps:

1. **Centre and outer radius.** A rough centre is taken from a thresholded
   mask. The outer boundary of the cell is then located on 180 rays as the
   strongest intensity edge, outlying rays are discarded, and the centre is
   corrected by fitting the first harmonic of the boundary radii. This is
   repeated four times.
2. **Inner radius.** The radius at which the winding begins is found from the
   variation of the intensity between angles: inside the hollow core it
   reflects only the noise, while within the windings it rises to a plateau.
3. **Polar binarization without pitch correction.** The slice is resampled on a
   plain (angle, radius) grid with 1440 angles. No shift is applied, so each
   turn appears as a band that ends one spacing further out than it begins.
   The image is binarized by background subtraction and Otsu thresholding in
   radial windows, and the hollow core is set to background.
4. **Winding extraction by connected components.** The binarized image is
   padded cyclically by a quarter of a turn, so that the 0°/360° seam joins
   true neighbours, and labelled with 8-connectivity. Every region that covers
   at least 90% of the angles is accepted as one turn. Its radius at every
   angle gives the trajectory of the turn, and small gaps are filled by
   circular interpolation.

Because connectivity is preserved by the polar transformation, the extraction
depends neither on the exact centre nor on the pitch of the spiral; the centre
only has to lie inside the hollow core.

## The code is explained

The notebook is meant to be read as well as run. It opens with an
introduction, a description of the model and a list of its limitations, and
every code cell is preceded by a text cell that explains what the cell does
and why. The functions have docstrings, and the code is commented line by line
where the reasoning is not obvious.

| Phase | Cells | Content |
|-------|-------|---------|
| 1. Setup and geometry detection | 1–4 | Imports and input data, radial sampling, centre and outer radius, inner radius |
| 2. Polar unwrapping | 5 | Polar binarization without a pitch model |
| 3. Winding extraction | 6–7 | Tracing each turn as a connected region; filling small gaps |
| 4. Visualization | 8 | Overlay of the outer and inner radii and the traced turns on the slice |
| 5. Demonstration | 9 | The full pipeline on several slices, with the same parameters |

## Setup

**Requirements:** Python 3.10 or newer (developed with Python 3.12) and the packages below.

```bash
# 1. Get the code
git clone https://github.com/genariturri/winding-detection-algorithms.git
cd winding-detection-algorithms/generation-2-geometry-independent

# 2. Create and activate a virtual environment (recommended)
python -m venv .venv
source .venv/bin/activate          # on Windows: .venv\Scripts\activate

# 3. Install the dependencies
pip install numpy scipy scikit-image matplotlib tifffile jupyterlab

# 4. Start Jupyter and open the notebook
jupyter lab
```

## Input data

The tomography data are **not** included in this repository. The notebook
works on individual slices, each stored as a single 2D TIFF image of a denoised
reconstruction (for example `slice_0200_n2i.tif`). Before running, edit the
input settings at the end of **Cell 1**: the folder that contains the slices,
and the slices to analyse, each given as a label for the output and the name
of its file in that folder:

```python
DATA_DIR = "/path/to/your/slices"   # folder that contains the slices

SLICE_FILES = {
    "Slice 200": "slice_0200_n2i.tif",
    "Slice 350": "slice_0350_n2i.tif",
    "Slice 500": "slice_0500_n2i.tif",
    "Slice 650": "slice_0650_n2i.tif",
}
```

## Running

Run the cells in order (*Run → Run All Cells*). Cells 1–8 set the input data
and define the functions; Cell 9 runs the whole pipeline on every slice listed
in `SLICE_FILES`.

## Outputs

For every slice, the notebook prints the detected centre, the inner and outer
radii and the number of turns traced, and it displays an overlay of the slice
with the outer radius (blue circle), the inner radius (dashed cyan circle) and
the traced turns (coloured curves). The figures are shown in the notebook and
are not saved automatically; add `plt.savefig(...)` in `plot_sanity_check`
(Cell 8) to keep them.

## Main parameters

All parameters are keyword arguments of the functions, with the defaults used
in the report. They were set on the dataset of the report and will probably
need to be adjusted for another cell or another pixel size:

| Function (cell) | Parameter | Default | Meaning |
|-----------------|-----------|---------|---------|
| `detect_max_radius` (3) | `n_iterations` | 4 | Passes of the centre correction |
| | `angle_step_deg` | 2 | Angular spacing of the rays (180 rays) |
| | `inner_margin` | 0.3 | Fraction of the search range excluded near the centre |
| | `outlier_tolerance` | 60 px | Rays farther than this from the median radius are discarded |
| `detect_spiral_inner_radius` (4) | `threshold_fraction` | 0.5 | Level between the noise floor and the plateau |
| | `safety_margin` | 15 px | Margin added to the detected inner radius |
| `build_polar_binary` (5) | `n_angles` | 1440 | Angular samples of the polar grid |
| | `background_sigma` | 15 px | Smoothing used to estimate the background |
| | `threshold_window`, `threshold_step` | 40 px, 10 px | Width and spacing of the Otsu windows |
| `trace_windings_via_components` (6) | `min_angular_coverage_fraction` | 0.9 | Minimum fraction of a turn covered by an accepted region |
| | `min_area` | 200 px | Smallest region considered |

## Limitations

- A turn that is interrupted or merged with a neighbour does not form a single
  region over 90% of the angles, so it is lost or mistraced instead of being
  reported as a defect. Detecting fissures and fusions on the traced turns is
  the next step.
- The binarization can split a winding where the dip inside its repeat unit is
  resolved, and it can accept the bright outer ring or the casing as extra
  regions.
- The outer boundary is assumed to be the strongest edge along most rays, and
  the windows used to find the inner radius are tied to the geometry of the
  cell of the report.
- The algorithm fails where the windings are interrupted over a wide range of
  angles, for instance by a large void near the end of the electrode stack.

## Citation

If you use this code, please cite the internship report:

> G. Iturri Bello, *Neutron Imaging and Tomography of Lithium Batteries: 3D
> Microstructural Analysis and a Geometry-Independent Spiral-Unwrapping
> Algorithm*, internship report, Institut Laue-Langevin, Grenoble (2026).

## Author

Genar Iturri Bello ([@genariturri](https://github.com/genariturri) on GitHub).
Contact details are given in the *Attribution* section at the top of the
notebook.
