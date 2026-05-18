# XL-Calibur Analysis Projects

This repository collects two small XL-Calibur research-analysis projects. They focus on two related parts of the instrument-analysis workflow: back-looking camera calibration and polarimetry-oriented detector response / maximum-likelihood modeling.

## Project Overview

### 1. Back-Looking Camera Calibration

Folder: [`01-back-looking-camera-calibration`](01-back-looking-camera-calibration)

This project analyzes XL-Calibur back-looking camera alignment packets. It extracts valid alignment records from observation runs, measures image-center drift and rotation over time, and classifies observation timestamps into spatial-offset cases.

Key work:

- Parsed XL-Calibur binary observation files with the `xlcalibur` Python data-access package.
- Filtered alignment packets using fit-valid flags, finite center coordinates, and nonzero scale values.
- Converted packet timestamps into elapsed seconds and MJD timestamps for run-level and observation-section analysis.
- Normalized camera center offsets into physical displacement units and plotted X/Y drift over time.
- Converted rotation angles from radians to degrees to inspect alignment stability.
- Classified observation points into 9 spatial offset cases based on X/Y displacement thresholds.
- Exported MJD time ranges and visualization results for Crab and Cygnus X-1 observation sections.

Example run-level alignment stability plot:

![Run-level center and rotation plot](01-back-looking-camera-calibration/calibration%20result/Crab%20Section%200/Run013448.png)

Example spatial distribution classification:

![Spatial distribution example](01-back-looking-camera-calibration/calibration%20result/spatial%20distribution/Crab%20Sec0%20visual.png)

### 2. Polarization ML and Detector Simulation (Keep updating)

Folder: [`02-polarization-ml-and-simulation`](02-polarization-ml-and-simulation)

This project explores a polarimetry analysis branch for XL-Calibur. It includes an event-level maximum-likelihood estimator for polarization degree / angle and a simplified detector-response simulation for a beryllium scattering bar and surrounding detector geometry.

Key work:

- Built a maximum-likelihood scan for polarization degree (PD) and polarization angle (PA).
- Used event-dependent modulation factors, `mu[i]`, mapped from detector row/layer calibration values.
- Read ON/OFF FITS event files and estimated source/background fractions from exposure-scaled counts.
- Simulated a 3D beryllium scattering bar and square detector response.
- Modeled energy-dependent attenuation and scattering probability.
- Visualized unfolded detector response and azimuthal modulation.

## Repository Structure

```text
.
├── 01-back-looking-camera-calibration/
│   ├── Python Analysis.pdf
│   ├── XL-CALI Alignment data.ipynb
│   ├── timespot classification.ipynb
│   └── calibration result/
├── 02-polarization-ml-and-simulation/
│   ├── maximum_likelihood_event_mu.ipynb
│   └── beryllium_bar_detector_simulation.ipynb
├── README.md
└── .gitignore
```

## Notebooks

### `01-back-looking-camera-calibration/XL-CALI Alignment data.ipynb`

This notebook performs run-level camera alignment analysis:

1. Opens an XL-Calibur run file through `dataclasses.XDataFile`.
2. Scans `X_PKT_ALIGNMENT_DATA` packets.
3. Deserializes packet records into alignment measurements.
4. Removes invalid records:
   - failed camera fits
   - non-finite center coordinates
   - zero scale values
5. Computes mean-centered X/Y offsets.
6. Converts timestamps into elapsed seconds from the start of the run.
7. Plots relative X/Y center position and camera rotation angle over time.
8. Saves per-run calibration figures.

### `01-back-looking-camera-calibration/timespot classification.ipynb`

This notebook converts alignment measurements into observation time categories:

1. Processes a sequence of run IDs.
2. Calculates normalized X/Y center offsets for each valid alignment record.
3. Converts timestamps into MJD.
4. Classifies each timestamp into one of 9 spatial zones:
   - center-stable region
   - positive/negative X shift
   - positive/negative Y shift
   - four diagonal shift regions
5. Exports case-by-case MJD lists.
6. Generates spatial distribution plots for observation sections.

### `02-polarization-ml-and-simulation/maximum_likelihood_event_mu.ipynb`

This notebook estimates polarization parameters using event-level modulation factors instead of a single global modulation value.

Workflow:

1. Reads ON and OFF XL-Calibur FITS event files (From HEASARC-NASA).
2. Extracts event-level columns such as `Q`, `U`, `PI`, `WEIGHT`, and detector row/layer information.
3. Converts Stokes-like event quantities into azimuthal angle:

   ```text
   phi = 0.5 * atan2(U, Q)
   ```

4. Maps each event to a detector-row modulation factor:

   ```text
   mu[i] = mu_table[row_i]
   ```

5. Applies a selected energy cut using the `PI` column.
6. Estimates source and background fractions from exposure-scaled ON/OFF counts.
7. Scans over polarization degree and polarization angle using a two-stage coarse-to-fine grid.
8. Plots likelihood contours using Wilks-style confidence levels.

Core per-event model:

```text
p_i = 1 + xiS * PD * mu[i] * cos(2 * (phi_i - PA))
```

where `PD` is polarization degree, `PA` is polarization angle, `mu[i]` is the event-dependent modulation factor, `xiS` is the estimated source fraction, and `phi_i` is the reconstructed event azimuthal angle.

### `02-polarization-ml-and-simulation/beryllium_bar_detector_simulation.ipynb`

This notebook builds a simplified geometric response model for a beryllium-scattering rod within a square detector boundary.

Workflow:

1. Defines a cylindrical Be scattering bar.
2. Builds a 3D scatter-point grid inside the bar.
3. Defines a four-wall detector layout with 32 axial layers and 32 global azimuthal columns.
4. Traces rays from scatter points to the detector boundary.
5. Applies a Gaussian PSF-like weighting around each hit point.
6. Includes energy-dependent attenuation and scattering fractions.
7. Combines responses across tested energies using a power-law source spectrum.
8. Visualizes the unfolded 32x32 detector response map and azimuthal response integrated over detector depth.

## Data Notes

The original raw run files and FITS event files are not included in this repository because they are large observation data products and are expected to live in the XL-Calibur data environment. Example local paths used during analysis include:

```text
/data/xlcal/data/2024_convData-P-ns/Run013685.dat
/Volumes/DATA/XL-Calibur-Crab/.../event/*.evt.gz
```

Generated calibration plots and MJD time-spot CSV files are included for the camera-calibration project so the output can be reviewed without the full raw-data environment. The detector simulation notebook is self-contained apart from standard Python packages.

## Requirements

- Python
- Jupyter Notebook or JupyterLab
- `numpy`
- `pandas`
- `matplotlib`
- `astropy`
- XL-Calibur internal Python packages for the camera-calibration notebooks:
  - `xlcalibur.dataclasses`
  - `xlcalibur.housekeeping`
  - `xlcalibur.dataaccess`
  - `xlcalibur.xcom.Packets`
  - `xlcalibur.systems.Systems`

## How To Run

1. Clone this repository.
2. Open the desired notebook in Jupyter.
3. For camera-calibration notebooks, update the run-file path to match the local XL-Calibur data environment.
4. For maximum-likelihood analysis, update the ON/OFF FITS event-file paths.
5. Run the detector simulation notebook directly if `numpy` and `matplotlib` are installed.

<!--  Interview Talking Points

- Built reproducible scientific Python workflows for instrument calibration and polarimetry analysis.
- Used domain-specific packet parsing to extract alignment telemetry from raw observation runs.
- Designed data-quality filters before computing physical offset metrics.
- Converted continuous camera offsets into discrete spatial cases so observation periods could be compared across sources and run sections.
- Implemented an event-level likelihood model instead of relying on a single average modulation factor.
- Connected detector calibration values to per-event physics parameters through `mu[i]`.
- Used ON/OFF exposure scaling to estimate source and background fractions.
- Used a coarse-to-fine likelihood scan to make parameter estimation computationally manageable.
- Built a geometric response simulation that traces scattering directions into a detector layout.
- Incorporated energy-dependent attenuation and source-spectrum weighting into the detector response.-->

## Possible Improvements

This project is ongoing and still being updated.
