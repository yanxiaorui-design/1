# Workplace socioeconomic segregation

Code for calculating workplace and worker-experienced socioeconomic segregation, interpolating residential housing prices, fitting city fixed-effects regression models, and simulating a 15-minute walking scenario. Version 1.0.3.

Socioeconomic status (SES) is proxied by residential housing prices and classified into city-specific quartiles. The repository includes a fully synthetic demo; confidential mobility data are not distributed. The demo demonstrates the core methods rather than reproducing empirical results for the 36 study cities.

## Layout

```text
workplace-socioeconomic-segregation/
├── main.py                     # Run the demo or an individual analysis stage
├── config.json                 # Analysis thresholds, IDW parameters and random seeds
├── requirements.txt            # Exact tested dependencies
├── LICENSE                     # MIT license
├── CITATION.cff                 # Software citation and version
├── src/
│   ├── housing.py              # IDW housing prices and commuter-weighted SES quartiles
│   ├── segregation.py          # Workplace scores and worker-experienced segregation
│   ├── regression.py           # Regional city fixed effects and LMG importance
│   ├── routing.py              # Amap walking API requests and route caching
│   └── simulation.py           # Feasible destinations and random workplace assignment
├── scripts/
│   ├── make_demo.py            # Regenerate the synthetic input tables
│   └── verify_demo.py          # Compare demo outputs with reference results
├── data/
│   ├── demo/                  # Six synthetic input tables
│   └── expected_results.json  # Reference results for 17 output files
├── tests/
│   ├── test_core.py            # Mathematical and statistical checks
│   └── test_pipeline.py        # End-to-end demo test
└── docs/
    ├── METHODS.md              # Detailed methods and assumptions
    └── DATA.md                 # Input fields, units, preparation and access
```

## System requirements

Tested on Windows with Python 3.12. Dependencies include NumPy, pandas, SciPy, statsmodels, requests, pyproj and pytest; exact versions are listed in `requirements.txt`. Linux and macOS have not been tested. No specialized hardware is required; a standard laptop with approximately 8 GB RAM is sufficient for the demo. Research-data requirements depend on city size and the number of routing pairs.

## Installation

From the folder containing `main.py`:

```bash
python -m venv .venv
# Windows PowerShell: .venv\Scripts\Activate.ps1
# Linux/macOS: source .venv/bin/activate
python -m pip install -r requirements.txt
```

Estimated installation time: 2–10 minutes with binary wheels and a normal internet connection. If PowerShell activation is unavailable, use `.venv\Scripts\python.exe` instead of `python`.

## Demo

```bash
python main.py demo
python scripts/verify_demo.py --output results/demo
```

The demo runs housing interpolation → SES assignment → segregation, plus regional regression and ten counterfactual simulations. It uses artificial routing costs and a separate synthetic regression sample. No API key or internet access is needed after installation.

Outputs are written to `results/demo/`. The observed synthetic example contains 6,400 workers and four retained work grids:

```text
workplace_median = 0.25
workplace_mean = 0.25
worker_experienced_mean = 0.25
Verified 17 synthetic reference outputs
```

The warmed pipeline runs in a few seconds; allow approximately 1–2 minutes for cold imports and tests. Run all nine automated checks with:

```bash
python -m pytest -q
```

## Run with research data

Place authorized, prepared inputs under `data/private/`. Run each city separately for housing, segregation and simulation; regression accepts a combined multi-city covariate table. See [data definitions](docs/DATA.md) for required fields and units.

```bash
# Interpolate housing prices and assign SES groups
python main.py housing --communities data/private/communities.csv --homes data/private/homes.csv --projected-crs EPSG:32650 --output results/city/housing

# Calculate observed workplace and worker-experienced segregation
python main.py segregation --flows data/private/flows.csv --output results/city/observed

# Estimate regional city fixed-effects models and relative importance
python main.py regression --covariates data/private/covariates.csv --output results/regression
```

Use the appropriate projected CRS in metres for the city; EPSG:32650 is an example. Before calculating segregation, join the housing output fields `house_pric` and `ses_group` to commuter flows by `home_polyid`. Raw mobile-signal processing, home/work inference and geographic covariate preparation occur upstream.

For the walking counterfactual, set `AMAP_API_KEY` in your local environment and provide all city 250 m grid centroids in GCJ-02 coordinates with their 1 km parent IDs:

```powershell
$env:AMAP_API_KEY = 'YOUR_KEY'
```

```bash
python main.py routes --flows data/private/flows.csv --grids data/private/grids.csv --cache cache/city.sqlite --output results/city/routing --max-requests 1000
python main.py simulate --flows data/private/flows.csv --grids data/private/grids.csv --routes results/city/routing/routes.csv --output results/city/simulation
```

The simulation selects destinations with Amap walking time ≤900 seconds, excludes the home grid, and assigns workers uniformly among feasible alternatives. It uses network travel time, without a Euclidean-radius fallback. Route failures, missing pairs and homes with no feasible destination stop execution. The private cache supports resuming requests; `--offline` requires a complete cache. Review pair counts and API quotas before increasing `--max-requests`. Current API queries do not recover a historical routing snapshot.

## Parameters and outputs

Edit `config.json` or pass `--config path/to/settings.json`. Defaults are: at least 30 workers per retained work grid, at least 10 POIs for regression, IDW with 12 neighbors and power 2, a 900-second walking threshold, and ten replicates starting with seed 20261012.

Workplace segregation is summarized by the city median of retained grid scores. Worker-experienced segregation is a worker-count-weighted mean. Regression outputs contain coefficients, HC1 robust inference and LMG contributions. Simulation outputs contain baseline scores, simulated flows, replicate changes and Monte Carlo summaries. Each run also records configuration and input hashes. See [methods](docs/METHODS.md) for details; use `python main.py --help` for command-line options.

## Data availability and license

Mobile-phone data are not publicly available to protect privacy. Research access may be requested from the corresponding author identified in the article, subject to provider approval and applicable restrictions. Housing prices originate from [Anjuke](https://www.anjuke.com/), POIs from [Amap](https://lbs.amap.com/), and road geometry from [OpenStreetMap](https://www.openstreetmap.org/) (© OpenStreetMap contributors; [ODbL](https://www.openstreetmap.org/copyright)). Walking routes are obtained separately through the [Amap API](https://lbs.amap.com/api/webservice/guide/api/direction).

This package covers the five core methods, not every manuscript figure or upstream processing step. Tests use synthetic data and mocked API responses; empirical and live-API validation requires authorized research inputs. Code and synthetic data use the [MIT license](LICENSE); third-party data retain their own terms. Citation metadata are provided in [CITATION.cff](CITATION.cff).
