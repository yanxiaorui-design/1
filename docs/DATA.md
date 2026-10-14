# Data dictionary and preparation

All CSVs use UTF-8. IDs must be nonmissing strings and unique where indicated. Process each city separately unless the stage explicitly accepts a multi-city covariate table.

| Table | Required columns | Interpretation |
|---|---|---|
| flows.csv | home_polyid, work_1km, ses_group, age_type, gender_type, count | Aggregated commuter records. ses_group is Q1–Q4; count is nonnegative integer sample count. Optional homecity and work_polyid. |
| homes.csv | home_polyid, x, y, count | One residential 250 m centroid per row, metre coordinates, total sampled commuters residing there. |
| communities.csv | community_name, x, y, price | Community average selling price per m²; one city, same projected CRS as homes. Invalid/nonpositive prices removed; duplicate name+coordinates removed. |
| grids.csv | polyid, work_1km, lon_gcj02, lat_gcj02 | All 250 m city-grid centroids and their unique 1 km parent IDs; optional city_code. Coordinates for routing are GCJ-02 degrees. |
| routes.csv | home_polyid, work_polyid, status, duration_s, distance_m | Complete directed matrix for every occupied eligible home and every other city grid. status is ok or no_route. API output also records endpoint, coordinates, timestamp and CRS. |
| covariates.csv | city_code, region, workplace_segregation, commuting_distance, Dis2Center, nearest_dis, roadLength, total_count, POI_count, POI_entropy, Elite Business & Finance, Industrial & Logistics, Public & Knowledge | One retained work grid per row. Distances and road length in metres; worker and POI counts in consistent units. region is East/Central/West/Northeast. |

The original age codes 3–5 map to 25–39, 6–8 to 40–54, and 9–10 to 55–64; gender codes 1/2 follow the original scripts. The manuscript's five broad age categories are a reporting aggregation, not the input encoding. Flow preparation assumes the source product has already passed the ≥10 observed home-days and ≥10 work-days filters. The package does not have raw day-level observations to repeat that filter.

Calculate 250 m centroids after projection, not directly in longitude/latitude. Construct the parent 1 km mapping by spatial join, checking boundary points and requiring exactly one parent per centroid. Apply official coordinate conversion to Amap inputs if their original datum differs. Keep projected IDW coordinates separate from GCJ-02 routing coordinates. Clip housing observations and grids to the chosen city boundary before analysis. The geographic study boundary and its historical version are author-supplied inputs.

Regression covariates are prepared upstream. Commuting distance in the current manuscript is Euclidean; only the counterfactual simulation is replaced by network routing. POI entropy uses consistent category definitions and natural logarithms. Sector counts and nearest subway/center distances must match the original covariate definitions. OSM contributes road geometry for road provision; Amap supplies walking routes. They are different inputs.

## Synthetic demonstration

All six demo tables were generated with `scripts/make_demo.py`. No row is copied, perturbed, sampled or anonymized from the confidential research dataset. Grid coordinates and prices are artificial. `routes_synthetic.csv` contains toy costs, not live routing responses. Covariates form a separate toy regression sample of 420 grids across 12 fictional cities. The demo cannot substantiate any result about real people, cities or the 2024 study.


## Data sources and access

The mobile-phone mobility data are not publicly available to protect user privacy. Research access may be requested from the study's corresponding author through the contact supplied with the article, subject to data-provider approval and applicable data-use restrictions. No real mobile-phone records are included in this repository.

Residential housing-price data originate from [Anjuke](https://www.anjuke.com/). Point-of-interest data originate from [Amap](https://lbs.amap.com/). Road-network geometry originates from [OpenStreetMap](https://www.openstreetmap.org/), © OpenStreetMap contributors, under the [Open Database License](https://www.openstreetmap.org/copyright). Walking route distances and times are obtained separately from the [Amap route-planning API](https://lbs.amap.com/api/webservice/guide/api/direction) using an authorized Web Service key.

Provider links identify the original sources; they are not archived copies of the study's 2024 extracts. Historical extracts and restricted inputs are not redistributed. Provider terms apply independently of the software license.

The files in `data/demo/` are fully synthetic and generated without reading, sampling or perturbing real mobility records. They support execution of the documented methods. `routes_synthetic.csv` contains artificial network costs and is not an API extract. Demo outputs must not be interpreted as research findings.
