# SWAT+ COCOA Build Workflow

An R-based workflow to build a **SWAT+ model using the COCOA approach**.  
The pipeline:

1. builds/updates a **SWATbuildR** SQLite project,
2. writes **SWAT+ TxtInOut** via `write.exe`,
3. applies required patches (**landuse.lum**, **nutrients.sol**, **time.sim**),
4. generates management using **SWATfarmR**,
5. exports a portable **`clean_setup/`** ready for calibration/scenario runs,
6. creates a **routing QA shapefile** to visualize COCOA connectivity.

---

## What’s in this repo

### Core scripts
- **`setup_workflow.R`** — main orchestrator (runs the full pipeline)
- **`settings.R`** — all paths + scenario controls (your main config file)
- **`functions.R`** — helper functions used by the pipeline
- **`read_and_modify_landuse_lum*.R`** — rule-based edits to `landuse.lum`
- **`create_connectivity_line_shape.R`** — exports COCOA routing as line shapefile for QA

---

## Repository structure (expected)


---

## Prerequisites

### Software
- **R** (recent stable recommended)
- **SWAT+ executable** (your chosen build)
- **`write.exe`** compatible with your SWAT+ build

### R packages (used by the workflow)
- SWAT stack: `SWATbuildR`, `SWATprepR`, `SWATfarmR`
- common deps: `sf`, `dplyr`, `readr`, `DBI`, `RSQLite`, etc.

### Platform
- **Windows** is assumed (the pipeline uses `.exe` tools).
- On Linux/macOS you must adapt the SWAT+ execution and writer steps.

---

## Input datasets

### 1) SWATbuildR inputs (`Data/for_buildr/`)
Provide at minimum:
- DEM raster (e.g., `DTM5m.tif`)
- Soil raster (e.g., `soil.tif`)
- Soil lookup tables (e.g., `Soil_SWAT_cod.csv`, `usersoil_*.csv`)
- COCOA land objects layer (e.g., `land.shp`)
- Channel network (e.g., `river.shp`)
- Basin boundary/mask (e.g., `basin.shp`)

### 2) Meteorology (`Data/for_prepr/met_int.rds`)
- Pre-prepared meteo object used by:
  - `SWATprepR::prepare_wgn()`
  - `SWATprepR::add_weather()`

### 3) SWATfarmR inputs (`Data/for_farmr_input/`)
- `crops.shp` with a **`type`** field (land-use key)
- `mgt_crops.csv` and `mgt_generic.csv` (management schedules)

Optional:
- point sources layer (e.g., `WWTP.shp`) if you enable that block in the script.

---

## Configuration

Edit: `Scripts/workflow/settings.R`

Key parameters you typically change:

- **Paths**
  - `out_path` (repo-relative root for `Data/`, `Libraries/`, `Outputs/`)
- **Project**
  - `project_name` (used to locate `<project_name>.sqlite`)
- **Simulation**
  - `st_year`, `end_year`
- **SWAT binaries**
  - `swat_exe` (SWAT+ executable file name in `Libraries/bin/` or your configured location)
- **Routing control**
  - `frc_thres` (connectivity fraction threshold to drop weak connections)
- **Soils**
  - `lab_p` (labile P value written into `nutrients.sol`)
- **SWATfarmR**
  - land-use prefix settings and input file locations

> ⚠️ **Important:** `setup_workflow.R` deletes and recreates the results directory (`res_path`).  
> Do **not** point `res_path` to anything you want to keep.

---

## Quick start

1) Put inputs in:
- `Data/for_buildr/`
- `Data/for_prepr/` (must include `met_int.rds`)
- `Data/for_farmr_input/`

2) Update:
- `Scripts/workflow/settings.R`

3) Run from the repo root:
```r
source("Scripts/workflow/setup_workflow.R")
