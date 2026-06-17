# subwaybuilder-eu-maps

## Summary

Each map covers the metropolitan area around one or more major European cities. Metropolitan-area bounds are drawn from national cadastral or statistical-office boundary products; municipal-level population, employment, and commute data drive resident and worker placement below the municipality scale wherever the source data allows it.

## Features

- High level of detail, with sub-municipality population placement driven by country-specific rasters or settlement-unit grids.
- Spatial realism -- points are assigned in a manner that is aware of water features, administrative boundaries, and density-weighted placement surfaces.
- Special demand from several country-specific open-data sources is modeled — covering airports, ports, universities, hospitals, military installations, sports venues, cultural attractions, museums, libraries, and tourism sites. See [Special Demand Details](#special-demand-details) below for the per-country category breakdown.
- Buildings are sourced from each country's national cadastre (RÚIAN for Czechia, BDOT10k for Poland, EHR + ETAK for Estonia). OSRM routing data is shared with the broader Subway Builder map pipeline.
- Building depth is default to -10m, with train-related infrastructure exempt.

## High-Level Methodology

Resident and commuter totals are estimated from country-specific census and employment tables, then distributed spatially using the best available mesh or raster guidance for each country. Population counts are conserved at municipality-level control totals, while workplace-side calibration and commute matrices keep resident and worker demand in balance.

A gravity model augmented with observed municipality-to-municipality commute flows reproduces macro-level commute patterns when sub-municipal O/D pairs are not directly published.

#### Czech Republic

Czech bundles combine CZSO Census 2021 population and economic-activity tables with the EU JRC GHS-POP 2020 100m raster for within-municipality population weighting. For distributing jobs, a separate 100m raster layer built from Overture building volumes (and weighed by industry type) is constructed against the DojizdkovyProud workplace-side totals. Administrative boundaries come from RUIAN (the Czech cadastral registry). For resident-worker flows, the CZSO 2021 commute O/D matrix at sub-municipal (ZSJ-díl) granularity is used to provide macro-level flow, with randomization used to assign individual populations for each ZSJ-díl pair to points within their respective boundaries.

In addition, because the 2021 census was conducted during COVID, its published commute matrix inflates same-settlement (and therefore ZSJ-díl) self-commute flows well above pre-pandemic levels. A gravity-based correction pass redistributes the excess self-commute volume into cross-settlement flows. The parameters of the gravity model are calibrated per bundle, using the known log-normal commute-distance distribution from the CZSO 2021 commute O/D matrix. This correction pass brings aggregate self-commute back to plausible pre-pandemic levels without distorting the observed inter-settlement flow structure.

#### Poland

Polish bundles combine GUS NSP 2021 census tables (population at 1km / 250m / 125m hybrid resolution; per-rejon population from the 16-sheet voivodship workbook) with the GUGiK BDOT10k national buildings cadastre for within-municipality population and worker weighting. The metropolitan-area boundary unit is the FUA (Funkcjonalny Obszar Miejski), defined by the Eurostat URAU 2021 layer, with each FUA dissolved from member gmina polygons via the GUGiK PRG cadastral registry.

The sub-municipal base unit of population/worker modeling is the **BREC rejon statystyczny** (35,774 nationwide), GUS's official statistical enumeration area — the direct analog to the Czech ZSJ-díl. Per-rejon worker mass is derived from BDOT10k floor area weighted by `kodKst` (the Polish building-function classification; 9 worker classes), with per-bundle weights calibrated by NNLS fit against BDL PKD employment counts. Gmina-level `pracujący` (BDL) is the column-margin truth for worker totals.

Two PL-specific calibration corrections augment the JP-style municipal-weighted gravity model. First, GUS strips intra-gmina commute flows at NSP 2021 publication. A **deterministic self-loop reconstruction** pass per gmina restores the diagonal as `BDL[g] − Σ inbound[g]`, recovering the intra-gmina worker mass that the model would otherwise lose.

#### Estonia

Estonian bundles combine the 2021 Estonian census per-municipality and per-asustusüksus tables with Statistikaamet's INSPIRE-aligned 100 m / 250 m population grid for within-municipality population weighting. Workplace mass is derived from the Ehitisregister national building cache filtered through the three-tier KAOS use-code taxonomy (residential / workplace / mixed-use), with per-firm employee totals from Estonian Commercial Register annual reports providing the workplace anchor. Administrative boundaries come from the EHAK (Eesti haldus- ja asustusjaotuse klassifikaator) registry; the five largest cities — Tallinn, Tartu, Pärnu, Narva, Kohtla-Järve — are further subdivided into asum / kvartal / linnaosa neighborhoods via Maa-amet's official sub-municipal boundary layer (Tallinn 92 polygons, Tartu 51, Pärnu 22, Narva 19, Kohtla-Järve 5).

Commute flows are calibrated against the 2021 census commute matrix, which publishes only bin-marginal shares (within-municipality, within-county, cross-county, against Tallinn, against Tartu) rather than a full municipality × municipality cell matrix. A Generalized IPF pass with destination-pinning on the two anchor cities reconciles a gravity-decay model against those bin marginals; a symmetric containment-share guard prevents over-routing to the urban core when the decay would otherwise concentrate flows on Tallinn / Tartu.

### Future countries

Additional European countries will be added as country-specific open-data pipelines come online. Each country follows the same conservation-and-calibration scheme above, with country-specific inputs substituted for boundaries, population, employment, commute matrices, and special-demand layers.

## Primary Data Sources

#### Czech Republic

- ČÚZK RUIAN — cadastral boundaries for municipalities and basic settlement units (cuzk.cz)
- ČSÚ Sčítání 2021 — population, economic activity, age structure (csu.gov.cz)
- EU JRC GHS-POP 2023A / 2020 epoch — 100m population raster for within-municipality weighting (ghsl.jrc.ec.europa.eu)
- ČSÚ DojizdkovyProud 2021 — sub-municipal commute O/D matrix and workplace-side occupied-jobs tables
- MŠMT DSIA F21 — university and junior-college enrollment (msmt.cz)
- tourdata.cz — krajské attraction visitor statistics
- Úřad pro civilní letectví ČR (Czech CAA) — airport passenger statistics
- ÚZIS ČR — Lůžkový fond per-facility hospital bed counts and occupancy, with the NRPZS national health-provider register for facility addresses (uzis.cz)
- AČR installation roster — active-duty personnel counts curated from official unit pages and public references

#### Poland

- GUGiK — PRG cadastral boundaries (voivodships, powiats, gminas), BDOT10k national building cadastre with PKOB → kodKst classification, and CAPAP national address geocoder (geoportal.gov.pl)
- GUS NSP 2021 / BREC / BDL — population at 1km / 250m / 125m hybrid grid, sub-gmina `rejon statystyczny` polygons + per-rejon populations, gmina-level pracujący employment, establishments, PKD-section workplace breakdown (P4457), per-institution student enrollment, and public-sector cultural attendance aggregates (stat.gov.pl, bdl.stat.gov.pl)
- Eurostat URAU 2021 — Functional Urban Area polygons for metropolitan-area definitions (gisco-services.ec.europa.eu)
- POL-on RADON — Polish Ministry of Education registry of higher-education institutions (radon.nauka.gov.pl)
- POT (Polska Organizacja Turystyczna) — annual `Frekwencja w atrakcjach turystycznych` PDF report
- ULC (Urząd Lotnictwa Cywilnego) — airport passenger statistics
- PIPT (Polska Izba Przemysłu Targowego) and Instytut Teatralny — convention / exhibition visitor statistics (polfair.pl) and per-season theatre attendance (e-teatr.pl)
- National sports leagues — Ekstraklasa football (ekstrastats.pl), Tauron Hokej Liga (hokej.net), PlusLiga volleyball (sportowefakty.wp.pl), ORLEN Superliga handball (orlen-superliga.pl)
- Municipal traffic studies — Warsaw Badanie Ruchu 2015 (WBR) and Wrocław Kompleksowe Badanie Ruchu 2018 (KBR), driving the optional rejon-level workplace prior and rejon × rejon OD overlay
- RPWDL — Rejestr Podmiotów Wykonujących Działalność Leczniczą, the national medical-entity registry for hospital facilities and addresses (via dane.gov.pl)
- GUS BDL "Zdrowie i ochrona zdrowia" — per-voivodship hospital bed occupancy and outpatient visitation rates
- Military installation roster — active-duty personnel counts curated from Polish Armed Forces unit references and public sources

#### Estonia

- Statistikaamet (Statistics Estonia) — 2021 census (per-municipality and per-asustusüksus population, employment, demographics), INSPIRE-aligned 100 m / 250 m population grid, commute O/D bin-marginal matrix, employed-residents by economic activity, national tourism visitor statistics, EHAK administrative classifier (stat.ee)
- Maa-amet (Estonian Land Board) — ETAK topographic data (buildings + land-use + water + transport ROW polygons), KAOS building-use taxonomy, authoritative address geocoder, annual cadastral snapshot with siht1 zoning, sub-municipal asum / kvartal / linnaosa boundaries for the five largest cities (geoportaal.maaamet.ee)
- Ehitisregister (Estonian Building Registry) — national building cache with per-building KAOS use codes, floor counts, footprints, and unit counts (ehr.ee)
- Eesti Ametlik Ariregister (Estonian Commercial Register) — per-firm employee totals from majandusaasta aruanded (annual reports), used as the workplace anchor for sectoral employment (ariregister.rik.ee)
- Eesti Hariduse Infosüsteem (EHIS / HTM) — institutional metadata + per-institution enrollment for universities, junior colleges, and technical colleges (haridussilm.ee)
- Tallinna Lennujaam (Tallinn Airport) — terminal-level annual passenger statistics (tallinn-airport.ee)
- Tallinna Sadam (Port of Tallinn) — per-port annual passenger throughput at Old City Harbour (Helsinki / Stockholm / cruise terminals), Paldiski North / South, Paljassaare, plus the regional ports at Rohuküla, Sillamäe, and Saaremaa (ts.ee)
- Terviseamet (Estonian Health Board) — hospital facility registry + per-facility bed counts (terviseamet.ee)
- Kaitsevägi (Estonian Defence Forces) — garrison roster, active-duty personnel curated from official unit references (mil.ee)
- Eurostat / EMODnet Bathymetry — INSPIRE Population Grid epoch alignment + EMODnet DTM bathymetry for coastal bundles

### Future countries

To be populated as each country's pipeline is finalized.

## Issues/Questions

Please raise an issue on this repository for incorrect manifests, broken download links, or release-page problems. Suggestions are welcome — especially pointers to country-specific open datasets that could improve existing bundles or unlock new ones.

## Known Issues

### Czechia

- A residual set of ZSJ-díl with no inbound commute flow from anywhere in the 2021 census matrix (and no resident commuters either) remain without modelled workers.
- Obce on the outskirts of the map boundary see high levels of short commutes due to the constraint that all commutes must start and end within the map boundary.
- Residential point location is noisy, and not entirely distanced from total "activity" density due to the smoothness of the GHS-POP raster — sometimes placing residents near industrial locations.

### Poland

- 33 of the 65 covered higher-education institutions still use single-point placement at the rector's office. Multi-campus institutions outside the 32 already disaggregated are not yet per-faculty split.
- Per-college in-person attendance is not available at the same granularity as Czechia; the resulting single-point campuses geocoded against the cadastre lead to some awkward large point placement.
- Mixed-use buildings (ground-floor retail + residential upstairs) are classified by their predominant cadastral function and are not split between residential and worker meshes — a small (~2%) population of buildings is affected.
- Self-loop reconstruction absorbs both 2021 → 2024 workforce growth and the long-haul register commutes that the cordon depth gate now correctly removes from daily transit, slightly inflating self-commute share — most visibly in boundary gminas of high-OOB-commuter bundles (warsaw, gdansk, szczecin).

### Estonia

- The modeled commute-distance distribution under-represents very-short trips (≤1 km, ~6% modeled vs ~16% in the Statistics Estonia TT231 ground truth) and very-long trips (>100 km, ~0.1% modeled vs ~3% TT231). The shortfall on the short tail is a sub-grid limitation (the resident-mesh cell grain floors how close origin and destination can land); the long tail is a closed-system limitation (the model only places trip endpoints within the bundle).
- Cross-county commutes whose destination is neither Tallinn nor Tartu are currently modelled as in-bundle self-loops rather than redistributed across the remaining 13 county capitals (maakonna keskus). Affects ~6.8–6.9% of Pärnu and Ida-Viru bundle residents, ~1.5–3.8% of Tallinn / Tartu.

### Cross-country

- Faraway water, cross-border land, and cross-border inland water past the bundle's modeled extent can render as a no-data "grid" pattern at the lowest zoom levels because the supplemental water and earth layers are extracted against the bundle boundary plus a small buffer. Coverage is correct at gameplay zoom levels and beyond; only the lowest-zoom overview is affected. Most visible on Ida-Viru (Russian land east of Narva river; Lake Peipus), Tartu (Lake Peipus; Russian land beyond), Pärnu (Latvian land to the south), Szczecin (German land to the west), and Gdańsk (Kaliningrad to the north-east).
- Cross-border commute flows are absent from every country's national O/D matrix (CZ / PL / EE alike publish only within-country commutes). The model therefore reconstructs all worker demand from in-bundle residents only, which understates true labour-side demand in border bundles whose populations commute substantially to a neighbouring country. Most visible on Zielona Góra and Szczecin (Berlin / Brandenburg side), Ústí nad Labem – Chomutov, Liberec – Jablonec, and Ostrava (cross-border to Saxony / Slovakia / southern Poland respectively), and Ida-Viru (historically Narva – Ivangorod, now mostly closed).

### Resolved (historical)

#### Poland

- ~~Under a single shared EU-wide gravity decay curve, modeled mean cross-gmina commute distance ran roughly +2 to +4 km long across most PL bundles vs the NSP-2021 cordon-adjusted observed mean. Player-visible as phantom long-haul commuters spread across the map.~~ **(Resolved in 0.3.1 — per-bundle commute-distance calibration; mean over-inflation across the 19-bundle PL fleet drops from +2.60 km to −0.03 km, all 19 within ±1 km.)**
- ~~The cordon mechanism fabricated implausibly long (>80 km, register / weekly-commute skew) cross-boundary flows as in-bundle daily transit demand at the map edge, over-saturating boundary gateway gminas — most visibly Warsaw, where inbound concentration ran 0.72 with 12 over-saturated gateways.~~ **(Resolved in 0.3.1 — cordon depth gate; Warsaw inbound concentration drops to 0.36 with all 12 gateways cleared; implausible-depth share roughly halved on all 19 bundles in both directions.)**
- ~~Three building-function codes in the Polish national classification (freight handling, postal sorting, aircraft hangars) silently routed large logistics and retail halls into a high-density transport-active workplace class instead of warehousing, causing a ~20× per-gmina workplace-mass over-weight on the affected buildings and pushing jobs out of central districts into peripheral logistics belts.~~ **(Resolved in 0.3.1 — workplace classification fix.)**
- ~~PL's published commute O/D matrix is employer-seat keyed: a najemni worker's commute is attributed to their employer's registered seat (siedziba), not their physical workplace. National employers headquartered in Warsaw / Kraków therefore booked their geographically-dispersed workforce's jobs to the HQ city, over-concentrating jobs at corporate-HQ city-counties.~~ **(Resolved in 0.2.4 — workplace totals re-keyed toward NSP-2021 P4500 physical workplace location.)**
- ~~The `pracujący` count from BDL is the narrowest of four PL employment measures (excludes individual farmers on holdings <1 ha and small-employer agriculture). The gap to broader measures (NSP 2021 census `pracujący` ~16.5M, GUS `pracujący ogółem` P3193 ~16.0M, vs BDL ~14.1M) is mostly small-farm employment.~~ **(Resolved in 0.2.4 — broadened to the NSP 2021 census `pracujący`.)**
- ~~Gdańsk oceanic index is not fully constructed -- requires a follow up and will be fixed in the next iteration~~ **(Resolved in 0.2.1)**

#### Czechia

- ~~The new metropolitan area boundaries are a bit strange and will need some expansion. Targeting that in a 0.2.0 for each Czech map~~ **(Resolved in 0.2.0)**

## Changelog

### 0.3.2 (2026-06-15)

#### New Cities

- **Czechia**
  - `JIH` - Jihlava
  - `KVY` - Karlovy Vary
  - `ZLN` - Zlín

#### Updated Cities

- **Czechia**
  - `BRQ` - Brno
  - `CBS` - České Budějovice
  - `HKP` - Hradec Králové - Pardubice
  - `LBC` - Liberec - Jablonec nad Nisou
  - `OLO` - Olomouc
  - `OSR` - Ostrava
  - `PLZ` - Plzeň
  - `PRG` - Praha
  - `UCH` - Ústí nad Labem - Chomutov

#### New Features

- **Expanded metropolitan boundaries.** All twelve maps use redrawn metro boundaries curated to whole-municipality (obec) coverage, removing the prior double-coverage between adjacent metros and widening each commuter shed.

- **Fuller land use coverage.** All twelve maps now source land use polygons from RÚIAN/ZABAGED directly, giving much fuller coverage of park/wooded land.
  - Rendered landuse is now clipped to water, so no water features appear obscured by greenery when rendered.

- **Buildings sourced from the RÚIAN cadastre instead of Overture.** 3D footprints and heights now come from RÚIAN StavebniObjekt (height from recorded floor count), matching the cadastre-sourced buildings already used for Poland and Estonia and replacing the Overture footprints + building-volume height enrichment.

- **Symmetric cordon at the map boundary.** Cross-boundary commute flows the published commute matrix would otherwise collapse onto each boundary district's self-loop are redistributed across in-bundle gateway districts by gravity × in-bundle capacity, in both directions — de-conflating the boundary self-loop and improving central-area commute accuracy.

- **Shopping malls and special-function buildings reclassified.** ZABAGED building-type tags are now used to split the coarse RÚIAN "civic amenity" code that lumped malls in with schools, hospitals, and offices: malls move to retail density, schools / hospitals / administration to civic-facility, industry to production, and technical infrastructure to its own class, etc.

- **Expanded special-demand coverage.** Added across the fleet: libraries (multi-branch systems disaggregated per branch), national museums and galleries, and large commercial markets as a new **shopping-center** category (e.g. SAPA Praha) — plus full coverage for the three new maps.

- **Cleaner point placement.** Resident and workplace anchor points now snap to the nearest cadastre building footprint.

#### Bugfixes

- **Phantom residents in non-residential districts corrected.** Usual-residence population the census registers in industrial zones, dormitories, and institutions (which have almost no dwellings) is now capped to dwelling capacity, with the excess redistributed within the same municipality.

- **Spa demand de-duplicated.** Czech spa (lázně) facilities were appended to the hospital layer with a uniform placeholder, producing clusters of identical-demand points; spa demand now lives only in the curated hot-spring attractions.

### 0.3.1 (2026-06-12)

#### New Cities

- **Poland**
  - `CZE` - Częstochowa
  - `IEG` - Zielona Góra
  - `KIE` - Kielce
  - `LEG` - Legnica - LGOM (Legnicko-Głogowski Okręg Miedziowy)
  - `OPL` - Opole
  - `RDO` - Radom
  - `SZY` - Olsztyn

#### New Features

- **Per-bundle commute-distance calibration.** The commute-distance distribution in each bundle now matches the bundle-specific NSP-2021 cordon-adjusted observed mean. Mean modeled commute-distance over-inflation across the full 19-map PL set drops from **+2.60 km to −0.03 km**; all 19 land within ±1 km of their observed target.

- **Implausible long-haul cordon commutes redirected to self-loop.** Cross-boundary flows longer than ~80 km from origin to destination are no longer fabricated as in-bundle daily transit demand at the map edge — long flows are kept whole below 80 km, ramped down to zero at 160 km, and dropped beyond. The unretained mass reverts to local self-commutes via the existing mass-conserving reconstruction path.

- **Workplace classification fix for large logistics buildings.** Three building-function codes in the Polish national classification (freight handling, postal sorting, aircraft hangars) were silently routing large logistics and retail buildings into a high-density transport-active workplace class instead of warehousing — a roughly 20× per-gmina workplace-mass over-weight on the affected buildings. The remap shifts these buildings to warehousing density, redistributing workplace demand from peripheral logistics belts back towards dencer office-heavy urban cores.

- **Sparser-class workplace density anchored to national priors.** The per-bundle calibration of building-class job densities now blends against PL national priors. Previously-sparse classes (e.g. transport-active in non-station bundles) no longer fit to implausible jobs sqm floor area ratios via calibration noise.

- **Cleaner resident point placement on BDOT buildings.** Residents across all PL bundles now snap to nearest BDOT residential building polygons.

### 0.3.0 (2026-06-10)

#### Initial Cities

- **Estonia**
  - `TLL` - Tallinn
  - `TAY` - Tartu
  - `EPU` - Pärnu
  - `IDV` - Ida-Viru (Narva + Kohtla-Järve)

#### New Features

- First release of the Estonia maps.
- Sub-municipal (asustusüksus) resident and worker placement for all four bundles, calibrated against the 2021 Estonian census and Statistikaamet's INSPIRE-aligned 100 m / 250 m gridded population.
  - The five largest cities (Tallinn, Tartu, Pärnu, Narva, Kohtla-Järve) are further subdivided into asum / kvartal / linnaosa neighborhoods via Maa-amet's official sub-municipal boundary layer.
  - Per-asustusüksus worker mass uses the Ehitisregister national building cache (with the official KAOS use-code taxonomy filtering residential / workplace / mixed-use) and per-firm employee totals from Estonian Commercial Register annual reports as the workplace anchor.
- Demand points for airports, passenger ferry terminals, universities and colleges, cultural attractions, convention and exhibition centers, sports venues, libraries, religious sites, national parks and natural landmarks, spa resorts, and theme parks across all four bundles.
- Per-municipality commute calibration against the 2021 census commute matrix.
  - The published matrix releases only bin-marginal shares (within-municipality, within-county, cross-county, against Tallinn, against Tartu) rather than a full municipality-to-municipality cell matrix.
  - A Generalized IPF pass reconciles a gravity-decay model against those marginals with destination-pinning on the two anchor cities; a symmetric containment guard prevents over-routing to the urban core.
- ETAK + Ehitisregister-derived buildings used in place of Overture for spatial detail.
- Standard OSRM routing included.

### 0.2.4 (2026-06-01)

#### Updated Cities

- **Poland**
  - `WAR` - Warszawa
  - `KTW` - Katowice - GZM (Górnośląsko-Zagłębiowska Metropolia)
  - `KRK` - Kraków
  - `POZ` - Poznań
  - `WRO` - Wrocław
  - `GDN` - Gdańsk
  - `LCJ` - Łódź
  - `LUZ` - Lublin
  - `SZZ` - Szczecin
  - `BTK` - Białystok
  - `BZG` - Bydgoszcz - Toruń
  - `RZE` - Rzeszów

#### New Features

- **Fuller land use coverage.** All twelve maps now source land use polygons from BDOT directly, giving much fuller coverage of park/wooded land.
  - Rendered landuse is now clipped to water, so no water features appear obscured by greenery when rendered.

- **Broader employment basis.** The per-municipality worker control total now uses the broader GUS NSP 2021 census `pracujący` (an ILO-style residence-side measure) instead of the narrower BDL administrative `pracujący` (P4280) used previously.
  - This lifts worker totals across all 12 bundles, with the largest impact in agriculture-heavy rural gminas that the administrative measure under-counts

- **Symmetric cordon at the map boundary.** Cross-boundary commute flows that the published NSP 2021 matrix would otherwise have collapsed onto each boundary gmina's self-loop diagonal are now redistributed across in-bundle gateway gminas via a mass-conserving multinomial weighted by gravity × in-bundle capacity
  - Redirected diagonal self-loop share is then debited at the new gateway origin. \
  - The cordon pass operates symmetrically in both directions (outbound: in-bundle resident, out-of-bundle job; inbound: out-of-bundle resident, in-bundle job).
  - The result significantly improves CBD self-loop accuracy on monocentric bundles and de-conflates the boundary self-loop where the closed-matrix fabrication previously over-concentrated it; recovery is sharpest at the map boundary itself, declining monotonically toward the interior.

- New special demand categories added across all 12 bundles:
  - **Hospitals** -- daily commute demand at inpatient and outpatient hospital facilities. Per-facility bed counts are allocated from voivodship totals by BDOT10k building floor area, anchored to the RPWDL national medical-entity registry.

  - **Military bases** -- active-duty personnel demand at Polish Land Forces, Air Force, Navy, Special Forces, and Territorial Defence installations.

  - **Passenger-ferry terminals.** New demand points at Świnoujście, Gdynia, Gdańsk-Westerplatte, and the seasonal Hel passenger pier — sized from annual Port Monitor passenger statistics.

#### Other Features

- **Additional venue and attraction coverage.** Several smaller demand additions across all 12 bundles: I liga football spectator demand (cup-game inclusive) at the lower-tier clubs, non-league event volume at major multi-use stadiums, municipal sport-recreation facilities operated by local MOSiR / BOSiR offices, and additional Catholic sanctuaries and marquee tourism attractions.

#### Bugfixes

- **Corrected university faculty coordinates.** Wydział Neofilologii Uniwersytetu Warszawskiego (Powiśle, near BUW) and Wydział Lekarski Uniwersytetu Jagiellońskiego Collegium Medicum (Stare Miasto, Kraków) repositioned to their actual addresses.
  - The prior coordinates had snapped to the wrong same-name street via the national geocoder's candidate-disambiguation step.

### 0.2.3 (2026-05-19)

#### Updated Cities

- **Poland**
  - `WAR` - Warszawa
  - `KTW` - Katowice - GZM (Górnośląsko-Zagłębiowska Metropolia)
  - `KRK` - Kraków
  - `POZ` - Poznań
  - `WRO` - Wrocław
  - `GDN` - Gdańsk
  - `LCJ` - Łódź
  - `LUZ` - Lublin
  - `SZZ` - Szczecin
  - `BTK` - Białystok
  - `BZG` - Bydgoszcz - Toruń
  - `RZE` - Rzeszów

#### New Features

- **Worker destination distribution rebalanced.** Within each sub-municipal area that contains multiple worker points, the share of inbound commuters is now distributed across those points by their relative employment weight rather than concentrating on whichever sibling won the proportional-sampling lottery.

- **Worker point seeding improvements for industrial estates and dense urban blocks.** Building footprint area is now blended into the seeding signal alongside floor count and local workplace density.
  - Large warehouse parks holding a small number of very large buildings now should split into multiple worker points rather than concentrating on a single mega-point.
  - The clustering pass that places workers around seed locations is now refined to prevent the entire worker mass of a small block from snapping onto its largest building.

- **Building polygon coverage expanded.** The buildings layer now includes BDOT's stadiums, engineering polygons, and technical infrastructure polygons (transformer stations and similar).

- **Neighbourhood label coverage.** Place name labels at the dzielnice and miejscowość level now come from the GUS SIMC official locality registry joined against OSM `teryt:simc` tags.

- **Edge-of-bundle municipality reassignment.** A small set of peripheral gminas at the FUA boundary that were previously silently zeroed (due to source-registry mismatches or recent administrative boundary changes) now resolve correctly for both residence and worker mass.

#### Other Features

- **Bundle boundary refinements for KTW and KRK.** Katowice now folds in the Rybnik FUA on its southwestern edge as well as Oświęcim in the southeast. Kraków is also expanded slightly west to meet that new boundary.

- **Per-faculty placement extended to four additional institutions.** Politechnika Śląska, Śląski Uniwersytet Medyczny, Uniwersytet Rolniczy w Krakowie, and Uniwersytet Rzeszowski now have per-wydział spatial distribution instead of concentrating all students at the rector's office.

### 0.2.2 (2026-05-17)

#### New Cities

- **Czechia**
  - `CBS` - České Budějovice
  - `LBC` - Liberec - Jablonec nad Nisou

#### Updated Cities

- **Czechia**
  - `BRQ` - Brno
  - `HKP` - Hradec Králové - Pardubice
  - `OLO` - Olomouc
  - `OSR` - Ostrava
  - `PLZ` - Plzeň
  - `PRG` - Praha
  - `UCH` - Ústí nad Labem - Chomutov

#### New Features

- Building footprint area is now blended into the worker placement signal.
  - Industrial estates containing a small number of large warehouses should now be split into multiple anchored points rather than concentrating onto a single mega-point
- New special demand points are added across all bundles, including
  - **Libraries** -- with multi-branch library systems disaggregated per pobočka rather than concentrated at the central building. Per-branch visitor counts are derived from operator annual reports
  - **Hospitals** -- with daily commute demand at inpatient and outpatient hospital facilities sized from ÚZIS Lůžkový fond (per-facility bed counts and occupancy) joined with the NRPZS national provider register for addresses and coordinates.
  - **Military bases** -- with demand for active-duty soldiers stationed kasárna generated from a small set of named AČR installations
- Workplace point populations are now better balanced.
  - When a ZSJ-díl contains multiple worker points, workers are now actually distributed across those points rather than concentrating onto whichever point won the proportional sampling lottery.
- Restored worker inbound for ZSJ-díl mismatched by the COVID-era census.
  - ZSJ-díl that have residents and worker capacity on paper but show zero inbound flow in the published 2021 commute matrix now receive a small share of commutes.

#### Bugfixes

- Large floor area worker outlier inflation fixed.
  - The reconciliation pass that balances residence-side and workplace-side totals could over-assign workers to a single building in volume-heavy ZSJ-díl, yielding artifacts like a single distribution warehouse or shopping-mall building modelled as containing the worker mass of an entire district.
- Worker-point snap now respects ZSJ-díl boundaries.
  - When a worker point is snapped to a nearby building polygon, the snap now prefers a building within the same ZSJ-díl, falling back to any-polygon only when no in-ZSJ-díl candidate exists.
  - Previously the largest-building-in-radius rule could anchor a ZSJ-díl's entire worker mass on a neighbor's building, particularly at ZSJ-díl edges adjacent to industrial sites.

### 0.2.1 (2026-05-12)

#### New Cities

- **Poland**
  - `WAR` - Warszawa
  - `KTW` - Katowice - GZM (Górnośląsko-Zagłębiowska Metropolia)
  - `POZ` - Poznań
  - `LUZ` - Lublin
  - `SZZ` - Szczecin
  - `BTK` - Białystok
  - `RZE` - Rzeszów

#### Updated Cities

- **Poland**
  - `BZG` - Bydgoszcz - Toruń
  - `GDN` - Gdańsk
  - `KRK` - Kraków
  - `LCJ` - Łódź
  - `WRO` - Wrocław

#### New Features

- Point seeding rebuilt around BDOT residential geometry, replacing the GHS-POP × residential-mask hybrid with a per-cell footprint × floor-count signal. Resident points now sit on BDOT building polygons directly.
  - Large industrial estates that previously concentrated worker demand at a single overloaded point are now split into multiple workplace points, with tighter inter-point spacing.
  - Merged residence + workplace points now anchor to the location with the strongest combined residential and workplace signal, rather than defaulting to the residential side — fixes cases where a point carrying most of a sector's workers sat on a low-worker cell.
- Census-mesh conservation overhaul. The 125m + 250m + 1km NSP hybrid previously inflated gmina-level rollups by 15–23%.
  - Several fixes were made so that the rollup now conserves to ~1% of NSP truth across all 12 bundles, and rural residential pockets that were dropped by the over-aggressive strict-quadrant rule are now retained.
- Point agglomeration overhauled to be sensitive to per-boundary density, with rural cells receiving lighter agglomeration passes
- Special demand coverage refreshed across all 12 bundles
  - Airports recurated to 2025 ULC data (15 airports, with proper krajowy / międzynarodowy traveler split per port).
  - Universities now use per-institution in-person (stacjonarne) shares from POL-on `iKindName`, replacing the flat national-average haircut applied in 0.2.0.
  - Per-faculty placement for 32 of the 65 covered higher-education institutions (446K students across 315 points), replacing single-point placement at the rector's office for those institutions.
- Tourism attraction coverage expanded by ~25 famous PoIs missing from POT (Jasna Góra, Bazylika Mariacka Kraków / Gdańsk, Sanktuarium Licheń, Kalwaria Zebrzydowska, PGE Narodowy, Stadion Śląski, AmberExpo, several major Catholic sanctuaries and palaces). 17 POT classifier corrections (cable cars, orthodox cerkwie, mining tourist trails, narrow-gauge railways) and a sports-venue recalibration are also included.
- **Urban-palace ticketed/unticketed disaggregation** for the largest royal residence-park complexes (Łazienki Królewskie 5.0M; Wilanów 2.8M). Visitor demand is split between the ticketed pavilion(s) and the free park rather than concentrated at the palace coord, mirroring the per-trailhead pattern already used for the largest national parks.

#### Bugfixes

- Several miejsko-wiejska gminas across multiple bundles were previously silently zeroed in resident/worker mass due to administrative-code mismatches between source registries. All such gminas now resolve correctly.
- Areas affected by 2025 administrative gmina splits (e.g. Grabówka, east of Białystok) had their workplace demand silently zeroed because the new gmina codes did not yet exist in the BDL employment registry. Workplaces in such areas now receive their share of the parent gmina's employment.

### 0.2.0 (2026-05-06)

#### Initial Cities

- **Poland**
  - `BZG` - Bydgoszcz - Toruń
  - `GDN` - Gdańsk
  - `KRK` - Kraków
  - `LCJ` - Łódź
  - `WRO` - Wrocław

#### New Features

- First release of the Poland maps.
- Sub-gmina (BREC rejon statystyczny) resident and worker placement for all four bundles, calibrated against NSP 2021 census tables and the GUGiK BDOT10k national buildings cadastre.
  - Per-bundle BDOT10k `kodKst` building-function weights are fit against BDL PKD employment.
- Phase E special demand for airports, universities, tourism attractions, sports venues across four top-tier leagues, convention/cultural centers, and multi-purpose arenas on all four bundles.
- Self-loop reconstruction applied to all four bundles
  - GUS strips intra-gmina commute flows at NSP 2021 publication (the published matrix records only cross-gmina worker flows).
  - The diagonal is reconstructed deterministically per gmina as `BDL[g] − Σ inbound_OD[dest=g]`.
- BDOT10k-derived buildings are used in place of Overture, standard OSRM routing (congruous with CZ) is included.

### 0.1.2 (2026-05-02)

#### Updated Cities

- **Czechia**
  - `BRQ` - Brno
  - `HKP` - Hradec Králové - Pardubice
  - `OLO` - Olomouc
  - `OSR` - Ostrava
  - `PLZ` - Plzeň
  - `PRG` - Praha
  - `UCH` - Ústí nad Labem - Chomutov

#### New Features

- All Czechia map boundaries expanded to include more of the surrounding area
- RÚIAN replaces Overture as the canonical source of CZ buildings, with nearly full tag + height coverage for ~4.22 million buildings nationwide
  - Better tagging enables more accurate point placement, with residential points only allowed on buildings tagged as residential, and worker points only allowed on buildings that could palusibly contain workplaces
- Worker points are now seeded separately from residential points, and are snapped to building polygons to improve placement in industrial estates
- Recalibrated building floor area job density priors against per-bundle empiric evidence and SLDB-NACE classifications, to better distribute workers amongst the seeded points

#### Bugfixes

- Fixed a bug where the cross-ref mesh for residential mass was essentially non-functional due to an ID formation mismatch.

### 0.1.1 (2026-04-26)

#### New Cities

- **Czechia**
  - `HKP` - Hradec Králové - Pardubice
  - `OLO` - Olomouc
  - `UCH` - Ústí nad Labem - Chomutov

#### Updated Cities

- **Czechia**
  - `BRQ` - Brno
  - `OSR` - Ostrava
  - `PLZ` - Plzeň
  - `PRG` - Praha

#### New Features

- Augmented the GHS-POP resident/population raster with Overture building information to reduce noise caused by smoothing
  - Known industrial areas now "mask" the raster to prevent resident placement in industrial estates
  - Areas with no Overture coverage also "mask" the raster to prevent resident placement in entirely rural areas
- Updated worker raster to take into account worker totals from non-intersecting cells in the Overture mesh cross-ref
- Removed same-node workers entirely, resulting in a very small ~0.2-0.6% drop in total modeled demand for each city in the bundle

### 0.1.0 (2026-04-25)

#### Initial Cities

- **Czechia**
  - `BRQ` - Brno
  - `OSR` - Ostrava
  - `PLZ` - Plzeň
  - `PRG` - Praha

#### New Features

- First release of the Czechia maps.
- Sub-municipal (ZSJ-díl) resident and worker placement for all four bundles, calibrated against Census 2021 tables and the GHS-POP 2020 raster.
- Phase E special demand for airports, universities/colleges, and cultural attractions on all four bundles.
- COVID-era self-commute correction applied to all Czech bundles; aggregate self-commute share brought from ~27–34% (published census) down to ~3–8% per bundle via gravity-calibrated redistribution.
- Overture-derived buildings and OSRM routing included.

## Planned Updates

- Additional Polish cities not yet included.
- Addition of additional Central / Eastern European countries (Hungary / Slovakia / Ukraine).
- Hospital and military-base demand layers for the Estonian bundles (Terviseamet + Kaitsevägi sources curated in the pipeline but not yet rendered in v0.3.0).

## Special Demand Details

Per-country category breakdown of the modeled demand-point categories beyond residence and workplace commute. Each row is geocoded against the relevant national authoritative source and sized from operator or government-published visitor / passenger / enrollment / bed-count figures.

### Czech Republic

- **Airports**
  - Demand based on annualized passenger statistics from the Czech Civil Aviation Authority, split by international & national travelers.
- **Institutions of Learning**
  - Students in post-secondary (univerzity / vysoké školy) sized from official prezenční (in-person) enrollment datasheets published by MŠMT (Ministry of Education).
- **Cultural Attractions**
  - Attendance figures sourced from the krajské (regional) tourism statistics published on tourdata.cz, supplemented with operator annual reports and per-site visitor counters for sites missing from the krajské aggregates.
  - Zoos, botanical gardens, aquariums.
  - Art & history museums, castles, chateaux.
  - Major parks, hot-spring spa towns (Karlovy Vary, Mariánské Lázně, Františkovy Lázně, Poděbrady), and natural landmarks.
  - Major religious sites (cathedrals, monasteries, pilgrimage churches).
  - UNESCO World Heritage sites (Pražský hrad, historic centers of Český Krumlov, Kutná Hora, and Olomouc, Lednice-Valtice cultural landscape, etc.).
  - Theme parks and aquaparks (Aquapalace Praha, Aqualand Moravia, etc.).
- **Libraries**
  - Per-branch visitor counts from library operator annual reports, disaggregated per pobočka.
- **Cultural Centers, Theatres & Concert Halls**
  - Per-season attendance from operator annual reports at the principal kulturní domy, divadla, and koncertní sály venues.
- **Multi-Purpose Arenas**
  - Non-sport event volume at major arenas (O2 arena Praha, Ostravar Aréna, Home Credit Arena Liberec, etc.) modeled separately from the sport-tenant rows so concert / family-event demand is captured alongside the league spectator demand.
- **Convention & Exhibition Centers**
  - Annual visitor totals from operator annual reports for the principal Czech expo and congress venues (Výstaviště Praha, BVV Brno, etc.).
- **Sports Venues**
  - Annual spectator counts at the principal Czech league venues across football (Fortuna Liga), ice hockey (Tipsport Extraliga), and basketball, plus public sport complexes (swimming, ice rinks) at municipal sport centers.
- **Hospitals**
  - Daily commute demand at inpatient and outpatient facilities, sized from ÚZIS per-facility bed counts and occupancy.
- **Military bases**
  - Active-duty personnel at named AČR (Czech Army) installations.

### Poland

- **Airports**
  - Demand based on annualized passenger statistics from the Civil Aviation Authority (Urząd Lotnictwa Cywilnego), split by international (ruch międzynarodowy) and national (ruch krajowy) travelers.
- **Passenger Ferry Terminals**
  - Per-terminal annual passenger throughput at the Świnoujście ferry terminal, the Gdynia and Gdańsk-Westerplatte cruise + ferry terminals, and the seasonal Hel passenger pier — sized from Port Monitor passenger statistics and port-operator annual reports.
- **Institutions of Learning**
  - Students in higher education sized from the POL-on RADON registry (Polish Ministry of Education) joined with GUS public per-institution enrollment statistics, and geocoded via the Polish national address geocoder GUGiK CAPAP. A composite stacjonarni (in-person) × in-person-attendance haircut is applied to estimate active on-site demand because GUS publishes only total enrollment, not the form-of-study split.
- **Cultural Attractions**
  - Attendance figures sourced from the POT (Polska Organizacja Turystyczna) annual "Frekwencja w atrakcjach turystycznych" report, supplemented with operator annual reports and per-site visitor counters for sites missing from the POT aggregates.
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
- **Hospitals**
  - Daily commute demand at inpatient and outpatient facilities. Per-facility bed counts are allocated from voivodship totals by BDOT10k building floor area, anchored to the RPWDL national medical-entity registry.
- **Military bases**
  - Active-duty personnel at Land Forces, Air Force, Navy, Special Forces, and Territorial Defence installations.

### Estonia

- **Airports**
  - Demand based on terminal-level annual passenger statistics from Tallinna Lennujaam (Tallinn Airport).
- **Passenger Ferry Terminals**
  - Per-terminal annual passenger throughput at the Old City Harbour Helsinki-line, Stockholm-line, cruise, and Paljassaare berths, the Paldiski North / South terminals, and the regional ports at Rohuküla, Sillamäe, and Saaremaa — sized from Tallinna Sadam (Port of Tallinn) and Statistikaamet maritime statistics.
- **Institutions of Learning**
  - Students at universities (ülikoolid), technical colleges (rakenduskõrgkoolid), and junior colleges (kõrgkoolid), sized from Eesti Hariduse Infosüsteem (the Ministry of Education registry) per-institution enrollment.
- **Cultural Attractions**
  - Attendance figures sourced from Statistikaamet national tourism statistics, supplemented with operator annual reports and per-site visitor counters for sites missing from the national aggregates.
  - Zoos, botanical gardens.
  - Art and general museums, castles, manor houses.
  - Major parks, hot-spring spa resorts, and natural landmarks.
  - National parks (Lahemaa, Soomaa, Karula, Matsalu, Vilsandi).
  - Major religious sites (Lutheran, Orthodox, and Catholic cathedrals, monasteries, and pilgrimage churches).
  - UNESCO World Heritage sites — Tallinn Vanalinn (Old Town) with **per-pavilion demand disaggregation**: the published city-level total is split between the ticketed pavilions (Toomkirik, Niguliste, Kiek in de Kök, Linnamüür) and the free walking-tour area rather than concentrated at the Old Town centroid.
  - Theme parks and aquaparks.
- **Libraries**
  - Per-branch visitor counts from library operator annual reports, including the Estonian National Library and the principal municipal branches.
- **Cultural Centers, Theatres & Concert Halls**
  - Per-season attendance from operator annual reports at the principal concert halls, theatres, philharmonics, and cultural districts (kultuurikeskused).
- **Multi-Purpose Arenas**
  - Non-sport event volume at the principal Estonian arenas modeled separately from the sport-tenant rows so concert / family-event demand is captured alongside the league spectator demand.
- **Convention & Exhibition Centers**
  - Annual visitor totals from operator annual reports for the principal Estonian expo and conference venues.
- **Sports Venues**
  - Annual spectator counts at the principal Estonian league venues across football, basketball, ice hockey, and volleyball, plus public sport complexes (swimming, ice rinks) at municipal sport centers.
- **Hospitals** _(to be populated in a future release)_
  - Daily commute demand at inpatient and outpatient facilities, sized from Terviseamet (Estonian Health Board) per-facility bed counts.
- **Military bases** _(to be populated in a future release)_
  - Active-duty personnel at Kaitsevägi (Estonian Defence Forces) garrisons.

## License

All maps are released under the [GNU General Public License v3.0](https://github.com/ahkimn/subwaybuilder-eu-maps/blob/main/LICENSE).

## Credits

All maps authored by [Yukina-](https://subwaybuildermodded.com/credits/)
