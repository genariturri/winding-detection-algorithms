# Winding-detection algorithms for neutron tomography of wound lithium-ion cells

Jupyter notebooks that locate the turns of the wound electrode stack (jelly
roll) of a cylindrical lithium-ion cell in neutron-tomography slices. They were
developed by Genar Iturri Bello during an internship at the Institut
Laue-Langevin (ILL), Grenoble, under the supervision of Dr. Bratislav Lukić,
and are described in Section 4 and Appendices A and B of the internship report
*Neutron Imaging and Tomography of Lithium Batteries: 3D Microstructural
Analysis and a Geometry-Independent Spiral-Unwrapping Algorithm* (2026).

## Contents

| Folder | Algorithm | Status |
|--------|-----------|--------|
| [`generation-1-archimedean-spiral`](generation-1-archimedean-spiral) | Unwraps each slice with an Archimedean spiral of constant pitch and reports fissures and fusions of the windings over a whole volume | Superseded by the second generation; published because the report analyses it |
| [`generation-2-geometry-independent`](generation-2-geometry-independent) | Traces each turn as a connected region of the polar image, without any model of the spiral | Locates and traces the windings; does not yet detect defects |

Each folder contains the notebook and a README that explains what it does, how
to set it up and run it, its main parameters and its limitations. The
notebooks are meant to be read as well as run: every code cell is preceded by
a text cell that explains what the cell does and why.

## Data

The tomography data are not included in this repository. The input expected by
each notebook is described in its README.

## Citation

If you use this code, please cite the internship report:

> G. Iturri Bello, *Neutron Imaging and Tomography of Lithium Batteries: 3D
> Microstructural Analysis and a Geometry-Independent Spiral-Unwrapping
> Algorithm*, internship report, Institut Laue-Langevin, Grenoble (2026).

## Author

Genar Iturri Bello ([@genariturri](https://github.com/genariturri) on GitHub).
Contact details are given in the *Attribution* section at the top of each
notebook.
