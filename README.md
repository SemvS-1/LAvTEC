# Finding the Ionospheric Vertical Total Electron Content Using LOFAR LBA VLBI Calibration Solutions

This repository contains the code used in my thesis, *"Finding the
Ionospheric Vertical Total Electron Content Using LOFAR LBA VLBI
Calibration Solutions,"* for obtaining the ionospheric vertical total
electron content (vTEC) from LOFAR LBA VLBI calibration solutions.

## Overview

The pipeline reconstructs vTEC per station, time step,
and assumed ionospheric height shell from two LOFAR calibration products —
differential rotation measure (dRM) and differential slant TEC (dsTEC) solutions — combined with the
geometry (line-of-sight magnetic field and airmass) of each station-source
line of sight at a stack of assumed shell heights.

| File | Purpose |
|---|---|
| `Calibration_Pipeline_(CP).py` | Core pipeline: loads geometry/dRM/dsTEC, forms differentials against a reference station, aligns time grids, and solves for vTEC (`solve_T`). Also contains the diagnostic plotting functions used across multiple reference stations. |
| `CP_Example.py` | Example driver that loops over every candidate reference station across two datasets and produces the full set of diagnostic figures for each. |
| `Geometry_File_Computation.py` | Computes the per-station, per-time, per-height airmass and line-of-sight magnetic field (`b_parallel`) needed by the pipeline, and saves them to an HDF5 geometry file. |
| `Single_Reference_Station_Diagnostics.py` | Simplified driver for a single, fixed reference station per dataset — produces the validity check and the two figures (B/ΔB per height, vTEC height profiles) used directly in the thesis. |

## Requirements

- **Python 3.9+**
- The packages listed in [`requirements.txt`](requirements.txt):

  ```
  numpy>=1.24
  scipy>=1.10
  h5py>=3.8
  matplotlib>=3.7
  astropy>=5.3
  losoto>=2.2
  lofarantpos>=0.5
  spinifex>=1.0
  ppigrf>=2.0
  ```

  Install everything with:

  ```bash
  pip install -r requirements.txt
  ```

  A few of these are domain-specific and worth calling out:
  - **[`losoto`](https://github.com/revoltek/losoto)** reads/writes the
    `h5parm` calibration-solution format used for the dRM and dsTEC solutions
    (`cal-fr.h5`, etc.).
  - **[`lofarantpos`](https://github.com/lofar-astron/lofarantpos)**
    provides LOFAR station positions, used in `compute_geometry.py` to
    build each station's `EarthLocation`.
  - **[`spinifex`](https://git.astron.nl/RD/spinifex)** computes
    ionospheric pierce points, airmass, and (via its magnetic-model
    wrapper) line-of-sight magnetic field at each pierce point.
  - **[`ppigrf`](https://github.com/IAGA-VMOD/ppigrf)** provides the IGRF
    geomagnetic field model that `spinifex`'s magnetic model calls
    internally; it isn't imported directly in this repo's code but must
    be installed for `compute_geometry.py` to run.

  `astropy`, `h5py`, `numpy`, `scipy`, and `matplotlib` are the general
  scientific-Python stack used for coordinates/time handling, HDF5 I/O,
  interpolation/filtering, and plotting respectively.

## Data layout

The scripts expect a data directory (configurable via the `BASE_DIR`
variable at the top of each script) containing the geometry, dRM, and dsTEC
files for each dataset/epoch, plus any external comparison data (GNSS
TEC, legacy reference TEC) used for the diagnostic plots. See the module
docstring at the top of `run_example.py` and `compute_geometry.py` for the
exact expected file layout.

## Usage

1. Run `Geometry_File_Computation.py` (once per dataset/epoch) to produce the
   geometry HDF5 file consumed by the pipeline.
2. Run `Calibration_Pipeline_(CP).py` to loop over candidate reference stations and
   generate the full diagnostic figure set, **or** run
   `Single_Reference_Station_Diagnostics.py` for a fixed reference station and the
   thesis figures directly.

Update the configuration block at the top of each script (`BASE_DIR`,
station lists, reference stations, heights, etc.) to match your own data
before running.
