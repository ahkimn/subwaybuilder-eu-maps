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

## 0.1.2 (2026-05-02)

### Updated Cities

- **Czechia**
  - `BRQ` - Brno
  - `HKP` - Hradec Králové - Pardubice
  - `OLO` - Olomouc
  - `OSR` - Ostrava
  - `PLZ` - Plzeň
  - `PRG` - Praha
  - `UCH` - Ústí nad Labem - Chomutov

### New Features

- All Czechia map boundaries expanded to include more of the surrounding area
- RÚIAN replaces Overture as the canonical source of CZ buildings, with nearly full tag + height coverage for ~4.22 million buildings nationwide
  - Better tagging enables more accurate point placement, with residential points only allowed on buildings tagged as residential, and worker points only allowed on buildings that could palusibly contain workplaces
- Worker points are now seeded separately from residential points, and are snapped to building polygons to improve placement in industrial estates
- Recalibrated building floor area job density priors against per-bundle empiric evidence and SLDB-NACE classifications, to better distribute workers amongst the seeded points

### Known Issues

- Obce on the outskirts of the map boundary see high levels of short commutes due to the constraint that all commutes must start and end within the map boundary.

### Bugfixes

- Fixed a bug where the cross-ref mesh for residential mass was essentially non-functional due to an ID formation mismatch.

## 0.1.1 (2026-04-26)

### New Cities

- **Czechia**
  - `HKP` - Hradec Králové - Pardubice
  - `OLO` - Olomouc
  - `UCH` - Ústí nad Labem - Chomutov

### Updated Cities

- **Czechia**
  - `BRQ` - Brno
  - `OSR` - Ostrava
  - `PLZ` - Plzeň
  - `PRG` - Praha

### New Features

- Augmented the GHS-POP resident/population raster with Overture building information to reduce noise caused by smoothing
  - Known industrial areas now "mask" the raster to prevent resident placement in industrial estates
  - Areas with no Overture coverage also "mask" the raster to prevent resident placement in entirely rural areas
- Updated worker raster to take into account worker totals from non-intersecting cells in the Overture mesh cross-ref
- Removed same-node workers entirely, resulting in a very small ~0.2-0.6% drop in total modeled demand for each city in the bundle

### Known Issues

- The new metropolitan area boundaries are a bit strange and will need some expansion. Targeting that in a 0.2.0 for each Czech map

## 0.1.0 (2026-04-25)

### Initial Cities

- **Czechia**
  - `BRQ` - Brno
  - `OSR` - Ostrava
  - `PLZ` - Plzeň
  - `PRG` - Praha

### New Features

- First release of the Czechia map pack.
- Sub-municipal (ZSJ-díl) resident and worker placement for all four bundles, calibrated against Census 2021 tables and the GHS-POP 2020 raster.
- Phase E special demand for airports, universities/colleges, and cultural attractions on all four bundles.
- COVID-era self-commute correction applied to all Czech bundles; aggregate self-commute share brought from ~27–34% (published census) down to ~3–8% per bundle via gravity-calibrated redistribution.
- Overture-derived buildings and OSRM routing included.

### Known Issues

- Some maps contain points that have residents live/work at the same location, which is an artifact of two distinct causes:
  - Some ZSJ-díl only contain one worker point, so any residents at that point who also work in the boundary must have a null commute
  - Some very small ZSJ-díl have only one point total, so any residents who also work in the boundary must have a null commute
- Residential point location is noisy, and not entirely distanced from total "activity" density due to the smoothness of the GHS-POP raster.
  - This sometimes placing residents near industrial locations

# Planned Updates

- Poland — boundary, census, and commute data in research.
- Additional Czech regional bundles as source-data coverage expands.
- Continued refinement of attraction coverage for Czech bundles.
