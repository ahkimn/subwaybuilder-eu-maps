# Map-Pack README Style Guide

Canonical formatting standard for the three public map-pack READMEs:
`subwaybuilder-eu-maps`, `subwaybuilder-jp-maps`, `subwaybuilder-tw-maps`.

**This file is the single source of truth.** It is maintained here (in
`subwaybuilder-jp-data`) and synced verbatim into each map repo's root as
`README_STYLE.md` by `scripts/release/sync_readme_style.py`. Edit it here, then
run the sync — never edit a repo-local copy directly.

Each map README is rendered into that repo's release `index.html` landing page by
`publish_map_pack.py` (the `README` block), so README formatting is player-facing.

---

## 1. Canonical section order

Every README uses this section order. Single-country repos (JP, TW) omit the
per-country `####` subdivisions and use flat prose / a flat list instead.

1. `# subwaybuilder-<cc>-maps` — the H1 title.
2. `## Summary` — 1–2 short paragraphs: what each map covers and where the
   metropolitan-area boundary comes from.
3. `## Features` — high-level capability bullets. Special demand gets **one
   summary bullet here that links down** to `## Special Demand Details` — the
   full per-category breakdown does **not** live under Features.
4. `## High-Level Methodology` — how residents/workers/commutes are modeled.
   Multi-country: a shared intro paragraph + one `#### <Country>` subsection each.
   Single-country: flat prose.
5. `## Primary Data Sources` — bold + linked source bullets (see §3). Multi-country:
   an optional `#### Shared` block + one `#### <Country>` subsection each.
   Single-country: one flat list.
6. `## Issues/Questions` — one paragraph on how to report problems / suggest data.
7. `## Known Issues` — current caveats. Multi-country: `### <Country>` subsections
   + `### Cross-country` + `### Resolved (historical)`. Single-country: a flat list
   with a trailing resolved/struck-through group.
8. `## Changelog` — reverse-chronological `### <version> (<YYYY-MM-DD>)` entries.
9. `## Planned Updates` — near-term roadmap bullets.
10. `## Special Demand Details` — the per-category breakdown moved out of Features.
    Multi-country: `### <Country>` subsections. Single-country: flat.
11. `## License`
12. `## Credits`

---

## 2. The bolded-bullet conventions (the core of this guide)

Substantive bullets lead with a **bold summary phrase**, then prose. Trivial
list bullets (city codes, plain source names) stay unbolded.

- **Changelog items** (New Features / Other Features / Bugfixes):

  ```markdown
  - **Bold summary phrase.** One or two sentences on the mechanism and the reason.
    - Optional nested sub-bullet for a caveat or a second-order detail.
  ```

  The bold phrase is a self-contained, sentence-case summary ending with a period
  *inside* the bold. It should read as a headline; the following prose is the detail.

- **Primary Data Sources** (§3):

  ```markdown
  - **Source Name** (parenthetical scope / what it provides) — [Link Label](https://deep/url) · [Second Link](https://…)
  ```

- **Special Demand Details** (§6):

  ```markdown
  - **Category**
    - Sub-bullet describing how the demand is sized and what sub-types are covered.
  ```

- **Resolved issues** — strike the original text, then a bold parenthetical:

  ```markdown
  - ~~Original issue text.~~ **(Resolved in X.Y — one line on the fix.)**
  ```

Punctuation: use a spaced em-dash ` — ` (U+2014), not `--` or a hyphen, for the
source→link separator and for parenthetical asides. Separate multiple links with
` · ` (spaced middle dot).

---

## 3. Primary Data Sources

- Format: `- **Source Name** (parenthetical scope) — [Link Label](url)`.
- **URLs as deep/specific as possible** — link the exact dataset, table, or product
  page (e.g. a `data.gov.*/dataset/<id>` or a statistical-office series URL), not the
  organization homepage, whenever a stable deep link exists.
- The parenthetical states what the source *provides* in this pipeline (population,
  boundaries, building footprints, commute matrix, enrollment, bed counts, …),
  named in public terms.
- Multi-country repos group under `#### <Country>` (+ a `#### Shared` block for
  cross-country sources like OSM, Overture, OSRM, bathymetry). Single-country repos
  use one flat list.

---

## 4. Voice & content rules

- **No internal / pipeline terminology.** Never surface implementation names in a
  public README — e.g. pipeline phase names, internal file or column names, or
  internal artifact names. Describe the *concept* in public terms ("special-demand
  points", "the sub-municipal residence grain", "the commute-calibration pass"),
  not the code that produces it.
- **Changelogs describe the how and the why, not an exhaustive what.** State the
  mechanism and the motivation. Avoid precise inventory counts ("added 47
  institutions"), specific internal table IDs, and other numbers that will drift or
  read as internal — the durable *what* (which sources, which categories) belongs in
  Primary Data Sources / Special Demand Details, not the changelog. A representative
  figure that conveys scale or impact is fine when it is the point of the change
  (e.g. a calibrated error dropping from +2.6 km to −0.03 km); an exhaustive tally is
  not.
- **Changelog dates** are the release/tag dates; when in doubt, read them from the
  repo's git tags (`git tag --sort=-creatordate` / `git log <tag>`). Every version
  heading carries a `(YYYY-MM-DD)`.
- **Country / place names** in headings and city lists use native script with a
  parenthetical romanization/English where helpful: `` - `CODE` - Native (Romaji) ``.
- **Emphasis** — bold for the summary phrases and source names above; italics
  (`_…_`) for foreign-language proper nouns on first use where it aids the reader.
  Don't bold whole sentences of prose.

---

## 5. Changelog subsection vocabulary

Under each `### <version> (<date>)`, use only these `####` subsections, in this
order, omitting any that don't apply:

- `#### Initial Cities` — first-ever release of a country's maps.
- `#### New Cities` — cities added this version.
- `#### Updated Cities` — existing cities re-exported this version.
- `#### New Features` — player-facing capabilities added.
- `#### Other Features` — smaller / infrastructural changes.
- `#### Bugfixes` — corrections.
- `#### Planned Updates` — only when previewing an upcoming version inline.

City lists are unbolded: `` - `CODE` - Native (Romaji) ``. Feature/bugfix items use
the bold-lead-in convention (§2). Images use `![alt](img/name.png)` on their own line.

---

## 6. Special Demand Details

A dedicated `## Special Demand Details` section (not nested under Features). One
`- **Category**` bullet per covered category, each with sub-bullets on how the
demand is sized and which sub-types it covers. A country lists only the categories
it actually covers.

**Category names** are standardized across all three repos, drawn from the internal
special-demand schema's public English labels (`source_data/special_demand_types.json`
`label.en`). Use these canonical public names / rollups:

- **Airports** · **Ports & Passenger Ferry Terminals**
- **Institutions of Learning** (schools + universities & colleges)
- **Hospitals**
- **Cultural Attractions** — a grouped bullet whose sub-bullets cover museums,
  heritage & World Heritage sites, parks & gardens, natural features, zoos &
  aquariums, amusement/theme parks, religious institutions, and bathhouses/onsen &
  resorts, as applicable per country
- **Libraries** · **Cultural Centers, Theatres & Concert Halls**
- **Sports Venues** · **Multi-Purpose Arenas**
- **Convention & Exhibition Centers** · **Shopping Centers**
- **Military Bases**

When a category is scheduled but not yet shipped, mark the heading
`- **Category** _(to be populated in a future release)_`.

---

## 7. Single-country vs multi-country

- **Multi-country (EU):** per-country `####`/`###` subsections in Methodology, Data
  Sources, Known Issues, and Special Demand Details; a `#### Shared` data-source block.
- **Single-country (JP, TW):** flat prose / flat lists — no per-country subdivision —
  but the same section order, the same bold-bullet conventions, and the same
  Special-Demand-as-its-own-section split.

---

## 8. Maintaining this guide

1. Edit **this file** in `subwaybuilder-jp-data`.
2. Run `python scripts/release/sync_readme_style.py` to copy it verbatim into each
   map repo's root `README_STYLE.md`.
3. Commit the synced copies with the change.

`CLAUDE.md` points here for any README edit across the three map repos.
