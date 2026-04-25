# Summary

Each map covers the metropolitan area around one or more major European cities. Metropolitan-area bounds are drawn from national cadastral or statistical-office boundary products; municipal-level population, employment, and commute data drive resident and worker placement below the municipality scale wherever the source data allows it.

## Features

- High level of detail, with sub-municipality population placement driven by country-specific rasters or settlement-unit grids.
- Spatial realism -- points are assigned in a manner that is aware of water features, administrative boundaries, and density-weighted placement surfaces.
- Special demand from several sources is modeled where country-specific open data is available.
  - **Czech Republic**
    - **Airports**
      - Demand based on annualized passenger statistics from the Czech Civil Aviation Authority, split by international & national travelers.
    - **Institutions of Learning**
      - Students in post-secondary (univerzity / vysoké školy) sized from official prezenční (in-person) enrollment datasheets published by MŠMT (Ministry of Education).
    - **Cultural Attractions**
      - Attendance figures sourced from the krajské (regional) tourism statistics published on tourdata.cz.
      - Zoos, botanical gardens, aquariums.
      - Art & history museums, castles, chateaux.
      - Major parks, spa towns, and natural landmarks.
      - Major religious sites (cathedrals, monasteries, pilgrimage churches).
  - **Poland**
    - Planned for a future release.
- Overture-derived buildings, OSRM routing, and export packaging are shared with the broader Subway Builder map pipeline.
- Building depth is default to -10m, with train-related infrastructure exempt.

## High-Level Methodology

Resident and commuter totals are estimated from country-specific census and employment tables, then distributed spatially using the best available mesh or raster guidance for each country. Population counts are conserved at municipality-level control totals, while workplace-side calibration and commute matrices keep resident and worker demand in balance.

A gravity model augmented with observed municipality-to-municipality commute flows reproduces macro-level commute patterns when sub-municipal O/D pairs are not directly published.

### Czech Republic

Czech bundles combine CZSO Census 2021 population and economic-activity tables with the EU JRC GHS-POP 2020 100m raster for within-municipality population weighting. For distributing jobs, a separate 100m raster layer built from Overture building volumes (and weighed by industry type) is constructed against the DojizdkovyProud workplace-side totals. Administrative boundaries come from RUIAN (the Czech cadastral registry). For resident-worker flows, the CZSO 2021 commute O/D matrix at sub-municipal (ZSJ-díl) granularity is used to provide macro-level flow, with randomization used to assign individual populations for each ZSJ-díl pair to points within their respective boundaries.

In addition, because the 2021 census was conducted during COVID, its published commute matrix inflates same-settlement (and therefore ZSJ-díl) self-commute flows well above pre-pandemic levels. A gravity-based correction pass redistributes the excess self-commute volume into cross-settlement flows. The parameters of the gravity model are calibrated per bundle, using the known log-normal commute-distance distribution from the CZSO 2021 commute O/D matrix. This correction pass brings aggregate self-commute back to plausible pre-pandemic levels without distorting the observed inter-settlement flow structure.

### Future countries

Additional European countries will be added as country-specific open-data pipelines come online. Each country follows the same conservation-and-calibration scheme above, with country-specific inputs substituted for boundaries, population, employment, commute matrices, and special-demand layers.

## Primary Data Sources

### Czech Republic

- ČÚZK RUIAN — cadastral boundaries for municipalities and basic settlement units (cuzk.cz)
- ČSÚ Sčítání 2021 — population, economic activity, age structure (csu.gov.cz)
- EU JRC GHS-POP 2023A / 2020 epoch — 100m population raster for within-municipality weighting (ghsl.jrc.ec.europa.eu)
- ČSÚ DojizdkovyProud 2021 — sub-municipal commute O/D matrix and workplace-side occupied-jobs tables
- MŠMT DSIA F21 — university and junior-college enrollment (msmt.cz)
- tourdata.cz — krajské attraction visitor statistics
- Úřad pro civilní letectví ČR (Czech CAA) — airport passenger statistics

### Future countries

To be populated as each country's pipeline is finalized.

## Issues/Questions

Please raise an issue on this repository for incorrect manifests, broken download links, or release-page problems. Suggestions are welcome — especially pointers to country-specific open datasets that could improve existing bundles or unlock new ones.

# Changelog

## 0.1.0 (2026-04-25)

### Initial Cities

- **Czech Republic**
  - `BRQ` - Brno
  - `OSR` - Ostrava
  - `PLZ` - Plzeň (Pilsen)
  - `PRG` - Praha (Prague)

### New Features

- First release of the Czech Republic map pack.
- Sub-municipal (ZSJ-díl) resident and worker placement for all four bundles, calibrated against Census 2021 tables and the GHS-POP 2020 raster.
- Phase E special demand for airports, universities/colleges, and cultural attractions on all four bundles.
- COVID-era self-commute correction applied to all Czech bundles; aggregate self-commute share brought from ~27–34% (published census) down to ~3–8% per bundle via gravity-calibrated redistribution.
- Overture-derived buildings and OSRM routing included.

### Known Issues

- Poland bundles are not yet included.

# Planned Updates

- Poland — boundary, census, and commute data in research.
- Additional Czech regional bundles as source-data coverage expands.
- Continued refinement of attraction coverage for Czech bundles.
