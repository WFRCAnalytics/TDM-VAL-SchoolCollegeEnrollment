# TDM-VAL-SchoolCollegeEnrollment

Validates the [PopulationSim](https://github.com/WFRCAnalytics/TDM-INP-PopulationSim) synthetic
population against real K-12 and college enrollment for the Wasatch Front region (Box Elder, Weber,
Davis, Salt Lake, and Utah counties). The rendered site is published with GitHub Pages from `docs/`.

The synthetic population is not controlled on school enrollment (`SCHG`), so the comparison runs two
ways: against real enrollment directly, and against the age-group population as a sanity bound.

- **K-12** is compared by **school district**, against Utah DOE (UGRC school counts) and NCES CCD.
- **College** is compared by **county**, against USHE and IPEDS.

## Status (October 2026)

| Page | Status |
|---|---|
| `1_geography.qmd` | Done. Builds the school-to-district and college-to-county crosswalks, with MAZ rows in both. Checks coverage and maps the study area. |
| `2_enrollment.qmd` | Done. Real enrollment by district and county, plus source comparison charts. |
| `3_synthetic.qmd` | Done, except the Box Elder clip (TODO). |
| `4_validation.qmd` | Districts and counties done. Box Elder (section 3) and the summary (section 4) are TODO. |
| `index.qmd` | Site home page: purpose, comparisons, page status, and limitations. |

## Data sources

- **Geography (Census TIGER, via pygris):** county boundaries and census blocks. Blocks are cached,
  but no crosswalk uses them.
- **Open SGID (UGRC):** school district boundaries; K-12 school locations and Utah DOE per-school
  enrollment counts (`ugrc_school`); higher-education locations (`ugrc_college`). Read over the network
  the first time, then from cache.
- **NCES:** EDGE public school locations with CCD enrollment; EDGE private school locations; EDGE
  postsecondary locations; the Private School Universe Survey (PSS) enrollment file; IPEDS Fall
  Enrollment (distance-education and full-/part-time files) and the institutional directory (HD).
- **WFRC:** the model (MPO) boundary; a draft MAZ microzone layer supplied for Utah (not yet a published
  WFRC layer); the WasatchFrontTAZ feature service (kept as a reference layer only; the joins use MAZ,
  not TAZ); and the USHE enrollment data request (Academic Year 2025, Term 2).
- **PopulationSim (local only):** the final persons output. See
  [`_data/raw/populationsim/README.md`](_data/raw/populationsim/README.md).

Box Elder County is only partly inside the MPO/TDM model region. Synthetic persons exist only inside
model MAZs, so the real-enrollment side needs clipping to the same footprint before a Box Elder
comparison. That clip is still TODO.

## Pipeline

The pages run in order, and each one reads the files the one before it wrote.

| File | Purpose |
|---|---|
| `index.qmd` | Site home page. Explains the comparison, lists the pages, and shows the data flow. |
| `1_geography.qmd` | Boundaries and school/college locations. Builds `crosswalk_school.csv` and `crosswalk_college.csv`, then checks coverage and maps the study area. |
| `2_enrollment.qmd` | Real K-12 enrollment by district (DOE, NCES public, NCES private) and college enrollment by county (USHE, IPEDS). Compares the sources. |
| `3_synthetic.qmd` | Rolls the PopulationSim persons up to district and county, by school grade (`SCHG`) and by age. |
| `4_validation.qmd` | Joins real and synthetic counts and reports the comparison in tables and charts. |

The validation tables compare the synthetic side with these real sources:

- K-12, by district: Utah DOE and NCES CCD public schools against synthetic `SCHG` 1-14 (any age), and
  the synthetic population aged 3-18.
- College, by county: USHE (public institutions) and IPEDS campus-based enrollment (optionally
  including private institutions) against synthetic `SCHG` 15-16 (any age), and the population aged
  18-24.

## Known limitations

- Box Elder needs the model-boundary clip (TODO).
- USHE is Academic Year 2025 and public only. IPEDS is fall 2023 and includes private institutions. The
  two are not like-for-like, and the sources differ in year.
- Synthetic college counts are by residence. IPEDS counts by institution location, and USHE by site.
- Charter schools are included in the district totals. Their attribution to resident districts is an
  assumption that has not been checked against DOE's published totals.
- Online-only schools are excluded from the district totals and reported separately.
- Private-school enrollment (PSS) is by school location and is not in the district validation table.

## PopulationSim inputs

The synthetic population comes from the sibling `TDM-INP-PopulationSim` repo. Its notebooks, and a
PopulationSim run that happens outside that repo, must run in this order:

1. `0_DownloadInput.ipynb`: fetch Census inputs.
2. `1_InputPreperation_GeoCrosswalk.ipynb`: build the block, MAZ, and TAZ crosswalks.
3. `2_InputPreperation_Seed2.ipynb`: write `seed_households.csv` and `seed_persons.csv` for both PopulationSim projects.
4. `3_InputPreperation_CollegeStudents_Controls.ipynb`: write the dorm-student controls into `StudentsInDorm/data/`.
5. `4_InputPreperation_NonGQ_Controls.ipynb`: write the non-group-quarters controls into `NonGQhouseholds/data/`.
6. **PopulationSim (outside both repos).** Run each project once, `NonGQhouseholds` and then `StudentsInDorm`, in a scenario folder other than `Base_sample`, so the WFRC model's base run isn't overwritten.
7. `5_PostProcessCombine.ipynb` (in `TDM-INP-PopulationSim`) combines the segment outputs into the final ABM files, including `Persons_Output4ABMwithStudentsInDorm.csv`. **This repo reads that file.** Copy it to `_data/raw/populationsim/ABM/1_Inputs/ScenarioBaseSample/1_PopulationSim/output/` under the same name.

The input folder is `StudentsInDorm` (singular) and the output folder is `StudentsInDorms` (plural).
Both names are intentional and match that repo's `_config.yaml`.

## Repository layout

| Path | What it is |
|---|---|
| `index.qmd`, `1_geography.qmd` to `4_validation.qmd` | The site pages, in pipeline order. |
| `_quarto.yml` | Site configuration: navigation sidebar, navbar, and theme. |
| `_extensions/WFRCAnalytics/wfrc-brand/` | WFRC brand extension (colors, fonts, logos). |
| `_data/raw/` | Inputs. Geometry, IPEDS, PSS, and USHE are committed. PopulationSim is local only. |
| `_data/intermediate/` | Files passed between pages. Rebuilt on each render, and gitignored. |
| `_outputs/` | Created by `2_enrollment.qmd`. Nothing is written there yet. |
| `docs/` | The rendered site, served by GitHub Pages. Committed. It includes copies of the intermediate CSVs and GeoJSON that the charts load. |
| `pyproject.toml`, `uv.lock` | Python dependencies. |
| `.env.example` | Template for `.env`. Not needed by the current pages; no page reads it. |

## Setup

### Python environment

This project uses uv with a repo-local virtual environment in `.venv/`. It requires Python 3.12.

```powershell
uv sync
```

If the uv-managed Python install location is not writable on your machine, keep it in this repo:

```powershell
$env:UV_PYTHON_INSTALL_DIR = ".uv-python"
uv sync --managed-python
```

Run Python or notebooks through the environment:

```powershell
uv run python
```

### Quarto

Install Quarto separately. Run it through the project environment so it uses this repo's `.venv`:

```powershell
uv run quarto --version
```

### Network

The first run of `1_geography.qmd` downloads boundaries and locations from Open SGID, NCES, WFRC's
ArcGIS services, and Census TIGER. After that, it reads the cached files in `_data/raw/`.

## Rendering the site

```powershell
uv run quarto render
```

This writes the site to `docs/`. To preview it locally while you edit, run:

```powershell
uv run quarto preview
```

## Data policy

- **Committed:** the raw inputs in `_data/raw/geometry/`, `_data/raw/ipeds/`, `_data/raw/pss/`, and
  `_data/raw/ushe/`, plus the rendered site in `docs/`.
- **Not committed:** `_data/intermediate/` (rebuilt on render), the PopulationSim folder except its
  README, `.env`, `.venv/`, `.uv-cache/`, and `.quarto/`.
