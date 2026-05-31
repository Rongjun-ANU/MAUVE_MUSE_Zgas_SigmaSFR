# MAUVE_MUSE_Zgas_SigmaSFR

Analysis pipeline, plotting notebooks, calibration tables, run logs, and figure
products associated with:

> MAUVE--MUSE: When Metallicity Follows or Fights Star Formation -- A
> Mass-Dependent Inversion in Virgo Galaxies

This repository is the public code-release location cited in the paper:

```text
https://github.com/Rongjun-ANU/MAUVE_MUSE_Zgas_SigmaSFR
```

The scripts and notebooks were copied from the local working tree
`/Users/Igniz/Desktop/ICRAR/extended` on 2026-05-31. They preserve the original
file names and mostly use relative paths from the repository root, so run the
scripts and notebooks from this directory unless a command explicitly says
otherwise.

## Scope

This repository contains the analysis code and derived plotting products needed
to document the paper workflow. It does not contain the full MAUVE--MUSE
datacubes or full nGIST value-added products. Those FITS products are large and,
as described in the paper, are expected to be released with the MAUVE public data
release. Before that release, access is controlled by the MAUVE collaboration.

The bundled files are therefore intended for:

- documenting the exact scripts and plotting notebooks used for the paper;
- preserving calibration/support tables needed by the scripts;
- preserving run logs that record the paper-generation runs;
- preserving final and referee-stage figure products;
- allowing reruns when the required MAUVE/nGIST FITS products are available
  locally.

## Repository Contents

Core pipeline scripts:

- `Foreground_star.py`, `Foreground_star.sh`  
  Builds foreground-star masks from MAUVE kinematic maps.
- `Mass.py`, `Mass.sh`  
  Builds extended spatial-binning products with stellar mass surface-density
  maps, R-band photometry, error maps, and inclination corrections.
- `SFR+Z.py`, `SFR.sh`  
  Builds Balmer-decrement-corrected Halpha SFR maps, BPT classifications,
  metallicity maps, and integrated metallicity summaries.
- `proxy_EWHa.py`, `proxy_EWHa.sh`  
  Builds continuum-cube proxy EW(Halpha) maps and legacy pseudo-EW comparison
  products.
- `sec34_null_test.py`, `sec34_null_test_20260408.py`  
  Helper modules used by `20260412_Plot_Paper_1_referee.ipynb` for the section
  3.4 null-test and simulation figures.

Plotting and analysis notebooks:

- `20260413_Plot_Paper_1_EW.ipynb`  
  Main paper plotting notebook for EW-selected figures and comparison panels.
- `20260412_Plot_Paper_1_referee.ipynb`  
  Referee-response / robustness plotting notebook, including SNR200 and null
  tests.
- `20260413_Moran_MAUVE14_EW.ipynb`  
  Local Moran-like map notebook for the 14-galaxy EW-selected sample.
- `20260521_BPT_diagram_tab20.ipynb`  
  BPT diagnostic diagram plotting notebook.
- `SNR200/20260412_Plot_Paper_1_SNR200.ipynb`  
  SNR200 robustness notebook. The large SNR200 FITS products are deliberately
  not tracked here.

Support tables and helper data:

- `BaSTI+Chabrier.dat`  
  MILES/BaSTI Chabrier-IMF mass-to-light table used by `Mass.py`.
- `MAUVE_Inclination.dat`  
  Galaxy inclinations used for projected-to-face-on surface-density corrections.
- `new_redshifts`  
  Galaxy redshift table used by `proxy_EWHa.py`.
- `Sanchez2017TableB1.csv`  
  CALIFA comparison table used by the referee / FMR comparison notebooks.
- `mauve_master_loader.py`, `mauve_master_wiki.fits`  
  Small local loader and master-sample FITS table used by the integrated
  comparison cells in the plotting notebooks.
- `MILES_safe/`  
  Compact SSP template subset used by the exact `Mass.py`
  `light_norm_to_r` conversion path when the original MILES template library is
  unavailable.

Run logs and figure products:

- `mass_logs/`, `sfr_logs/`, `proxy_ewha_logs/`  
  Per-galaxy logs from the paper analysis runs.
- `SNR200/mass_logs/`, `SNR200/sfr_logs/`  
  Logs from the SNR200 robustness reruns.
- `Plot_Paper_1/`  
  Main paper figure products.
- `Plot_Paper_1_referee/`  
  Referee-stage figure products. Private report/letter documents and the
  duplicate manuscript PDF were intentionally excluded.
- `Moran_MAUVE14/`  
  Per-galaxy local Moran-like map products.

## Files Intentionally Not Included

- `SNR200/*.fits` files were omitted intentionally. They are large intermediate
  products and were excluded per the release-preparation decision.
- Full MAUVE datacubes, continuum cubes, nGIST gas maps, SFH maps, spatial
  binning maps, and kinematic maps are not included.
- Local macOS metadata (`.DS_Store`) and notebook checkpoints are not included.
- Private or non-code referee documents (`*.docx`) and the duplicate manuscript
  PDF inside `Plot_Paper_1_referee/` are not included.

## Runtime Environment

The original local runs used the ICRAR science Python environment. A matching
environment needs Python 3 plus:

- `numpy`
- `scipy`
- `astropy`
- `matplotlib`
- `speclite`
- `ppxf`
- `jupyter` / `ipykernel` for notebooks

Install a lightweight environment from the provided requirements file:

```bash
python -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

On systems where a project Conda environment already exists, activate that
environment instead and run the scripts from the repository root.

The shell wrappers also use standard Unix tools. `SFR.sh` and `proxy_EWHa.sh`
use GNU `parallel` if available and fall back to `xargs`.

## Expected Input Data Layout

The pipeline scripts expect MAUVE data either under the MAUVE project root:

```text
/arc/projects/mauve/
  cubes/v3.0/<GALAXY>/
  products/v0.6/<GALAXY>/
```

or in the repository root as fallback inputs. The most important expected
patterns are:

```text
<GALAXY>_DATACUBE_FINAL_WCS_Pall_mad_red_v3.fits
<GALAXY>_CONTcube.fits
<GALAXY>_KIN_maps.fits
<GALAXY>_KIN_maps_extended.fits
<GALAXY>_SPATIAL_BINNING_maps.fits
<GALAXY>_SPATIAL_BINNING_maps_extended.fits
<GALAXY>_SFH_maps.fits
<GALAXY>_sfh_weights.fits
<GALAXY>_gas_BIN_maps.fits
<GALAXY>_gas_BIN_maps_extended.fits
<GALAXY>_proxy_EW_maps.fits
```

The 14-galaxy paper sample is:

```text
IC3392
NGC4064
NGC4192
NGC4293
NGC4298
NGC4330
NGC4383
NGC4396
NGC4419
NGC4457
NGC4501
NGC4522
NGC4694
NGC4698
```

## Core Workflow

The wrappers are configured for the paper sample and can also accept a custom
galaxy list.

1. Build foreground-star masks where needed:

   ```bash
   ./Foreground_star.sh
   ./Foreground_star.sh NGC4064 NGC4298 NGC4694
   ```

2. Build stellar mass surface-density products:

   ```bash
   ./Mass.sh
   ./Mass.sh NGC4064 NGC4192
   ```

   Useful environment overrides:

   ```bash
   PYTHON_BIN=/path/to/python ./Mass.sh NGC4064
   MASS_NCPUS=4 ./Mass.sh NGC4064
   MASS_DISABLE_STAT=1 ./Mass.sh NGC4064
   ```

3. Build SFR, BPT, and metallicity products:

   ```bash
   ./SFR.sh
   ./SFR.sh NGC4064 NGC4192
   ```

4. Build proxy EW(Halpha) maps:

   ```bash
   ./proxy_EWHa.sh
   ./proxy_EWHa.sh NGC4064 NGC4192
   ```

5. Recreate paper figures from the notebooks:

   ```bash
   jupyter lab
   ```

   Open the notebooks from the repository root so their relative paths resolve
   correctly. The notebooks write into `Plot_Paper_1/`, `Plot_Paper_1_referee/`,
   `Moran_MAUVE14/`, and `SNR200/` depending on the notebook.

## SNR200 Robustness Products

The SNR200 folder currently contains only:

```text
SNR200/20260412_Plot_Paper_1_SNR200.ipynb
SNR200/mass_logs/
SNR200/sfr_logs/
```

The corresponding SNR200 FITS products are intentionally absent from this
repository. To rerun SNR200 plotting cells that read FITS files, regenerate or
place the following products in `SNR200/`:

```text
NGC4396_v3tk_SNR200_spatial_binning_maps_extended.fits
NGC4396_v3tk_SNR200_gas_bin_maps_extended.fits
NGC4396_v3tk_SNR200_sfh_maps.fits
NGC4396_v3tk_SNR200_sfh_weights.fits
NGC4501_v3tk_SNR200_spatial_binning_maps_extended.fits
NGC4501_v3tk_SNR200_gas_bin_maps_extended.fits
NGC4501_v3tk_SNR200_sfh_maps.fits
NGC4501_v3tk_SNR200_sfh_weights.fits
```

These names are ignored by `.gitignore` so they can be used locally without
accidentally committing large intermediate products.

## Output Naming

The scripts preserve the working-tree naming convention:

- `Mass.py` writes `<GALAXY>_SPATIAL_BINNING_maps_extended.fits`.
- `SFR+Z.py` writes `<GALAXY>_gas_BIN_maps_extended.fits`.
- `proxy_EWHa.py` writes `<GALAXY>_proxy_EW_maps.fits`.
- `Foreground_star.py` writes `<GALAXY>_KIN_maps_extended.fits`.

The notebooks save PNG/PDF/SVG products into the figure directories listed
above.

## Notes For Reuse

- The bundled logs are useful provenance records, but rerun logs will reflect
  local file paths and installed package versions.
- The scripts are research-code snapshots for this paper, not a general MAUVE
  public pipeline release.
- The figure notebooks contain executed outputs from the source working tree;
  rerunning them requires the relevant local FITS products.
- The public data release should be treated as the source of science data once
  available. Until then, the repository documents the analysis pipeline and the
  derived public-facing products bundled here.

## Citation

If this repository is useful for reproducing or extending the work, cite the
associated MAUVE--MUSE paper and this GitHub repository URL.
