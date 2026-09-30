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
  output, classified by `PersonType_Label` (uncontrolled) and by age group (controlled) at the
  block level, then aggregated to school district / county via exact block-level crosswalks.

Box Elder County is only partially covered by the MPO/TDM region, so it is validated last using
the model boundary clipped at the tract level.

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
