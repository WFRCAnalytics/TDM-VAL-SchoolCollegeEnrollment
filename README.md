# TDM-VAL-SchoolCollegeEnrollment

Validates the [PopulationSim](https://github.com/WFRCAnalytics/TDM-INP-PopulationSim) synthetic
population against real K-12 and college enrollment data for the Wasatch Front region (Box Elder,
Weber, Davis, Salt Lake, and Utah counties).

The synthetic population is not controlled on school enrollment (`SCHG`), so this project checks
it two ways: against real enrollment directly, and against the age-group population as a sanity
bound.

## Data sources

- **School enrollment / geography**: [TDM-INP-K-12-Enrollment](https://github.com/WFRCAnalytics/TDM-INP-K-12-Enrollment)
  and UGRC school district boundaries (state-preferred source), compared at the **school district**
  level.
- **College enrollment / geography**: [TDM-INP-College-Enrollment](https://github.com/WFRCAnalytics/TDM-INP-College-Enrollment)
  (USHE primary, IPEDS fallback), compared at the **county** level.
- **Synthetic population**: [TDM-INP-PopulationSim](https://github.com/WFRCAnalytics/TDM-INP-PopulationSim)
  output, classified by school grade (`SCHG`, uncontrolled) and by age group (controlled), then
  aggregated to school district / county through MAZ polygons (`_data/raw/geometry/`).

Box Elder County is only partially covered by the MPO/TDM model region. Synthetic persons exist only
inside model MAZs, so the real-enrollment side is clipped to the same footprint before comparison.

## PopulationSim pipeline order

The synthetic population comes from the sibling `TDM-INP-PopulationSim` repo. Its notebooks and a
PopulationSim run must happen in this order. PopulationSim itself runs outside that repo.

1. `0_DownloadInput.ipynb`: fetch Census inputs.
2. `1_InputPreperation_GeoCrosswalk.ipynb`: build the block, MAZ, and TAZ crosswalks.
3. `2_InputPreperation_Seed2.ipynb`: write `seed_households.csv` and `seed_persons.csv` for both PopulationSim projects.
4. `3_InputPreperation_CollegeStudents_Controls.ipynb`: write the dorm-student controls and crosswalk into `StudentsInDorm/data/`.
5. `4_InputPreperation_NonGQ_Controls.ipynb`: write the non-group-quarters controls into `NonGQhouseholds/data/`.
6. **PopulationSim (outside both repos).** Run each project once, `NonGQhouseholds` and then `StudentsInDorm`, in a scenario folder other than `Base_sample` so the WFRC model's base run isn't overwritten.
7. Copy the three outputs into `_data/raw/populationsim/` in this repo, keeping the subfolders:
   - `NonGQhouseholds/synthetic_NonGQhouseholds.csv`
   - `NonGQhouseholds/synthetic_NonGQpersons.csv`
   - `StudentsInDorm/synthetic_StudentsInDorms.csv` (singular here; see the note below)
8. `5_PostProcessCombine.ipynb` (in `TDM-INP-PopulationSim`) combines the two segments into the final ABM files. It is not needed for this repo's validation.

The input folder is `StudentsInDorm` (singular) and the output folder is `StudentsInDorms` (plural).
Both names are intentional and match that repo's `_config.yaml`.

## Pipeline

| File | Purpose |
|---|---|
| `1_geography.qmd` | School district boundaries, college campus locations, county assignment, and the MPO/TDM model boundary; builds the block-to-district and block-to-county crosswalks. |
| `2_enrollment.qmd` | Real K-12 (by district) and college (by county) enrollment counts. |
| `3_synthetic.qmd` | PopulationSim synthetic population aggregated to district/county, by enrollment status and by age group. |
| `4_validation.qmd` | Joins real vs. synthetic aggregates and reports the comparison. |

Raw/intermediate data (including PopulationSim's synthetic population microdata) stays local and
is not committed; only aggregated comparison tables and charts are published.

## Python environment

This project uses uv with a repo-local virtual environment in `.venv/`.

Set up or refresh the environment:

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

## Site

Rendered with Quarto and published via GitHub Pages.
