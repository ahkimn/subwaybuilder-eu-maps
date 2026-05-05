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
    - **Airports**
      - Demand based on annualized passenger statistics from the Civil Aviation Authority (Urząd Lotnictwa Cywilnego), split by international (ruch międzynarodowy) and national (ruch krajowy) travelers.
    - **Institutions of Learning**
      - Students in higher education sized from the POL-on RADON registry (Polish Ministry of Education) joined with GUS public per-institution enrollment statistics, and geocoded via the Polish national address geocoder GUGiK CAPAP. A composite stacjonarni (in-person) × in-person-attendance haircut is applied to estimate active on-site demand because GUS publishes only total enrollment, not the form-of-study split.
    - **Cultural Attractions**
      - Attendance figures sourced from the POT (Polska Organizacja Turystyczna) annual "Frekwencja w atrakcjach turystycznych" report.
      - Major museums, castles, palaces, and historic buildings.
      - Zoos, botanical gardens, arboreta, palmiarnie.
      - Water parks and thermal baths.
      - Catholic sanctuaries, cathedrals, basilicas, and monasteries.
      - UNESCO World Heritage sites (Auschwitz-Birkenau, Wieliczka/Bochnia salt mines).
      - National parks with **per-entrance demand disaggregation** for the largest parks (Tatrzański, Kampinoski, Karkonoski, Pieniński, Wielkopolski, PN Gór Stołowych) — the published park-level total is split across major trailheads / gateway entrances rather than a single park centroid.
    - **Sports Venues**
      - Annual spectator counts derived from official league statistics across four top-tier Polish leagues:
        - Ekstraklasa football (ekstrastats.pl).
        - Tauron Hokej Liga ice hockey (hokej.net + polskihokej.eu).
        - PlusLiga men's volleyball (plusliga.pl + sportowefakty.wp.pl).
        - ORLEN Superliga handball (orlen-superliga.pl).
    - **Convention & Exhibition Centers**
      - Annual visitor totals from the PIPT (Polska Izba Przemysłu Targowego) annual industry report and per-operator pages.
    - **Theatres, Philharmonics & Opera**
      - Per-season attendance from Instytut Teatralny (e-teatr.pl) for theatres; operator sprawozdania (annual reports) for philharmonics and opera houses.
    - **Multi-Purpose Arenas**
      - Non-sport event volume at major arenas (Tauron Arena Kraków, Spodek Katowice, Atlas Arena Łódź, Ergo Arena Gdańsk-Sopot, ORLEN Arena Płock, Arena Toruń, etc.) modeled separately from the sport-tenant rows so concert / family-event demand is captured alongside the league spectator demand.
- Overture-derived buildings, OSRM routing, and export packaging are shared with the broader Subway Builder map pipeline.
- Building depth is default to -10m, with train-related infrastructure exempt.

## High-Level Methodology

Resident and commuter totals are estimated from country-specific census and employment tables, then distributed spatially using the best available mesh or raster guidance for each country. Population counts are conserved at municipality-level control totals, while workplace-side calibration and commute matrices keep resident and worker demand in balance.

A gravity model augmented with observed municipality-to-municipality commute flows reproduces macro-level commute patterns when sub-municipal O/D pairs are not directly published.

### Czech Republic

Czech bundles combine CZSO Census 2021 population and economic-activity tables with the EU JRC GHS-POP 2020 100m raster for within-municipality population weighting. For distributing jobs, a separate 100m raster layer built from Overture building volumes (and weighed by industry type) is constructed against the DojizdkovyProud workplace-side totals. Administrative boundaries come from RUIAN (the Czech cadastral registry). For resident-worker flows, the CZSO 2021 commute O/D matrix at sub-municipal (ZSJ-díl) granularity is used to provide macro-level flow, with randomization used to assign individual populations for each ZSJ-díl pair to points within their respective boundaries.

In addition, because the 2021 census was conducted during COVID, its published commute matrix inflates same-settlement (and therefore ZSJ-díl) self-commute flows well above pre-pandemic levels. A gravity-based correction pass redistributes the excess self-commute volume into cross-settlement flows. The parameters of the gravity model are calibrated per bundle, using the known log-normal commute-distance distribution from the CZSO 2021 commute O/D matrix. This correction pass brings aggregate self-commute back to plausible pre-pandemic levels without distorting the observed inter-settlement flow structure.

### Poland

Polish bundles combine GUS NSP 2021 census tables (population at 1km / 250m / 125m hybrid resolution; per-rejon population from the 16-sheet voivodship workbook) with the GUGiK BDOT10k national buildings cadastre for within-municipality population and worker weighting. The metropolitan-area boundary unit is the FUA (Funkcjonalny Obszar Miejski), defined by the Eurostat URAU 2021 layer, with each FUA dissolved from member gmina polygons via the GUGiK PRG cadastral registry.

The sub-municipal base unit of population/worker modeling is the **BREC rejon statystyczny** (35,774 nationwide), GUS's official statistical enumeration area — the direct analog to the Czech ZSJ-díl. Per-rejon worker mass is derived from BDOT10k floor area weighted by `kodKst` (the Polish building-function classification; 9 worker classes), with per-bundle weights calibrated by NNLS fit against BDL PKD employment counts. Gmina-level `pracujący` (BDL) is the column-margin truth for worker totals.

Two PL-specific calibration corrections augment the JP-style municipal-weighted gravity Phase D. First, GUS strips intra-gmina commute flows at NSP 2021 publication. A **deterministic self-loop reconstruction** pass per gmina restores the diagonal as `BDL[g] − Σ inbound[g]`, recovering the intra-gmina worker mass that the model would otherwise lose.

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

### Poland

- GUGiK — PRG cadastral boundaries (voivodships, powiats, gminas), BDOT10k national building cadastre with PKOB → kodKst classification, and CAPAP national address geocoder (geoportal.gov.pl)
- GUS NSP 2021 / BREC / BDL — population at 1km / 250m / 125m hybrid grid, sub-gmina `rejon statystyczny` polygons + per-rejon populations, gmina-level pracujący employment, establishments, PKD-section workplace breakdown (P4457), per-institution student enrollment, and public-sector cultural attendance aggregates (stat.gov.pl, bdl.stat.gov.pl)
- Eurostat URAU 2021 — Functional Urban Area polygons for metropolitan-area definitions (gisco-services.ec.europa.eu)
- POL-on RADON — Polish Ministry of Education registry of higher-education institutions (radon.nauka.gov.pl)
- POT (Polska Organizacja Turystyczna) — annual `Frekwencja w atrakcjach turystycznych` PDF report
- ULC (Urząd Lotnictwa Cywilnego) — airport passenger statistics
- PIPT (Polska Izba Przemysłu Targowego) and Instytut Teatralny — convention / exhibition visitor statistics (polfair.pl) and per-season theatre attendance (e-teatr.pl)
- National sports leagues — Ekstraklasa football (ekstrastats.pl), Tauron Hokej Liga (hokej.net), PlusLiga volleyball (sportowefakty.wp.pl), ORLEN Superliga handball (orlen-superliga.pl)
- Municipal traffic studies — Warsaw Badanie Ruchu 2015 (WBR) and Wrocław Kompleksowe Badanie Ruchu 2018 (KBR), driving the optional rejon-level workplace prior and rejon × rejon OD overlay

### Future countries

To be populated as each country's pipeline is finalized.

## Issues/Questions

Please raise an issue on this repository for incorrect manifests, broken download links, or release-page problems. Suggestions are welcome — especially pointers to country-specific open datasets that could improve existing bundles or unlock new ones.

# Changelog

## 0.2.0 (2026-05-06)

### Initial Cities

- **Poland**
  - `BZG` - Bydgoszcz - Toruń
  - `GDN` - Gdańsk
  - `KRK` - Kraków
  - `LCJ` - Łódź
  - `WRO` - Wrocław

### New Features

- First release of the Poland maps.
- Sub-gmina (BREC rejon statystyczny) resident and worker placement for all four bundles, calibrated against NSP 2021 census tables and the GUGiK BDOT10k national buildings cadastre.
  - Per-bundle BDOT10k `kodKst` building-function weights are fit against BDL PKD employment.
- Phase E special demand for airports, universities, tourism attractions, sports venues across four top-tier leagues, convention/cultural centers, and multi-purpose arenas on all four bundles.
- Self-loop reconstruction applied to all four bundles
  - GUS strips intra-gmina commute flows at NSP 2021 publication (the published matrix records only cross-gmina worker flows).
  - The diagonal is reconstructed deterministically per gmina as `BDL[g] − Σ inbound_OD[dest=g]`.
- BDOT10k-derived buildings are used in place of Overture, standard OSRM routing (congruous with CZ) is included.

### Known Issues

- Per-college in-person attendance is not available to the same granularity as CZ and the resulting single-point campuses geocoded against the cadastre, leading to some awkward large point placement
- Residential point location is somewhat noisy — within-rejon population weighting uses a hybrid of the census grid and a GHS-POP 100m raster gated by the BDOT10k residential mask.
  - The mask is more selective than the CZ pipeline's RÚIAN binary mask, but GHS-POP smoothing still places some residents near light-industrial estates that BDOT10k does not classify as workplace.
- The `pracujący` count from BDL is the narrowest of four PL employment measures (excludes individual farmers on holdings <1 ha and small-employer agriculture)
  - The gap to broader measures (NSP 2021 census `pracujący` ~16.5M, GUS `pracujący ogółem` P3193 ~16.0M, vs BDL ~14.1M) is mostly small-farm employment, but the choice of measure may be revisited given the large rural areas covered in some bundles
- Self-loop reconstruction absorbs all 2021 → 2024 workforce growth (~5–7%) into intra-gmina mass, slightly inflating self-commute relative to a pure 2021 census view
- Gdańsk oceanic index is not fully constructed -- requires a follow up and will be fixed in the next iteration

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

- First release of the Czechia maps.
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

- Continued refinement of attraction coverage and point placement for Czech/Polish bundles.
- Additional Polish cities not yet included
- Addition of additional Central/Eastern European countries (Estonia/Hungary/Slovakia/Ukraine)
