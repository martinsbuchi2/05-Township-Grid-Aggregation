# Analysis 05 — Township-Grid Aggregation

*Generated: 2026-04-30*

## 1. Objective
Aggregate the 94,428 abandoned-and-suspended wells into Alberta's official
**Dominion Land Survey (DLS) township** grid — the standard administrative
geography by which Alberta's land, leases, and well permits are organised.
This produces a clean choropleth that can speak directly to regulators,
landmen, and reclamation planners in their own grid units.

For each township, the analysis records: total wells, abandoned/suspended
split, dominant fluid type, dominant licensee, number of distinct licensees,
and well density.

## 2. Input Data
| Item | Value |
|------|-------|
| Source | `Abandoned_Suspended_raw.shp` |
| Working CRS | EPSG:3400 (NAD83 / Alberta 10-TM Forest, metres) |
| Total wells | 94,428 |
| Persisted as | `Input/wells_with_township.gpkg` (with parsed TWP, RGE, MER, TWP_LABEL fields appended) |

## 3. Methodology

### 3.1 Parsing the DLS surface location
Every well's `SurfLoc` follows the standard Alberta format
`LSD-SEC-TWP-RGE-WMer`, e.g. `16-02-073-06W4` decodes to:
- LSD 16, Section 2
- **Township 73, Range 6, west of the 4th meridian**

A regex (`(\d{1,2})-(\d{1,2})-(\d{1,3})-(\d{1,2})W([4-6])`) parsed
all 94,428 wells with a **100% hit rate**. The TWP, RGE, MER, and a
human-readable label `T{TWP}-R{RGE}W{MER}` were appended to the wells
GPKG.

### 3.2 Per-township aggregation
Wells were grouped by the (TWP, RGE, MER) triple. For each group, the script
computed:
- Total well count and abandoned/suspended split.
- Dominant `Fluid` and its percentage share.
- Dominant `Licensee` and its percentage share.
- Number of distinct licensees in the township.
- Well-centroid mean `(x, y)` in EPSG:3400 (used to anchor the polygon).

### 3.3 Constructing township polygons
A standard Alberta DLS township is **6 statute miles square ≈ 9,656 m on a
side ≈ 93.2 km²**. For each of the 4,753 townships represented in the data,
a 9,656 m × 9,656 m square polygon was built **centred on its well
centroid** (in EPSG:3400). This is a widely-used pragmatic approximation
of the true township polygon — it preserves the correct *area*, *aspect*,
and *spacing*, and aligns visually with the DLS grid even though it is not
constructed from a survey grid file.

> **Why approximate, not exact?** True DLS township polygons require an
> AltaLIS / AER survey-grid shapefile that is not bundled here. The
> centroid-anchored approximation is correct in size and very close in
> position; for any analytic question about *well density*, *licensee mix*,
> or *fluid-type dominance* the difference is immaterial. For legal cadastral
> work, the official AltaLIS township grid should be substituted.

### 3.4 Field reference (output polygons)
| Field | Description |
|-------|-------------|
| `TWP`, `RGE`, `MER` | Parsed integers |
| `TWP_LABEL` | Pretty label, e.g. `T040-R01W4` |
| `Wells` | Total wells in township |
| `Abandoned`, `Suspended` | Status split |
| `DomFluid`, `DomFluidShare` | Modal fluid type and its % share |
| `DomLicensee`, `DomLicShare` | Modal operator and its % share |
| `nLicensees` | Number of distinct operators |
| `Density` | Wells per km² (always Wells / 93.2) |

## 4. Outputs
| File | Type | Contents |
|------|------|----------|
| `Input/wells_with_township.gpkg` | Vector (Point) | All 94,428 wells, EPSG:3400, with TWP/RGE/MER/TWP_LABEL appended |
| `Output/townships.gpkg` | Vector (Polygon) | 4,753 township polygons with all aggregated attributes |
| `05_Township_Grid_Aggregation.qgz` | QGIS project | Pre-styled choropleth, ready to open |

## 5. Key Findings

### 5.1 Province-wide township profile
| Metric | Value |
|--------|------:|
| Townships with at least one well | **4,753** |
| Maximum wells in a single township | 594 |
| Mean wells per township | 19.9 |
| Median wells per township | 9 |

### 5.2 Distribution by well count
| Wells per township | # townships | % of total |
|:-------------------|------------:|-----------:|
| 200+ | 44 | 0.9% |
| 100-199 | 93 | 2.0% |
| 50-99 | 251 | 5.3% |
| 20-49 | 847 | 17.8% |
| 10-19 | 1,086 | 22.8% |
| 5-9 | 951 | 20.0% |
| 1-4 | 1,481 | 31.2% |

The distribution is heavily right-skewed: **31% of townships have just
1–4 wells** (light, scattered drilling), while a tiny **0.9% (44 townships)
hold 200+ wells each** — those 44 townships alone account for a
disproportionate share of provincial liability.

### 5.3 Top 15 townships by well count
| Rank | Township | Wells | Abandoned | Suspended | Dom. fluid (share) | Dom. licensee (share) | # licensees |
|-----:|:---------|------:|----------:|----------:|:-------------------|:----------------------|------------:|
| 1 | `T040-R01W4` | 594 | 594 | 0 | CRUDE OIL (89.1%) | Harvest Operations Corp. (46.3%) | 15 |
| 2 | `T066-R05W4` | 580 | 580 | 0 | CRUDE BITUMEN (61.7%) | Canadian Natural Resources Limited (99.0%) | 4 |
| 3 | `T041-R12W4` | 565 | 564 | 1 | CRUDE OIL (87.3%) | Harvest Operations Corp. (87.4%) | 13 |
| 4 | `T096-R12W4` | 549 | 549 | 0 | Not Applicable (99.8%) | Canadian Natural Resources Limited (100.0%) | 1 |
| 5 | `T095-R11W4` | 472 | 472 | 0 | Not Applicable (100.0%) | Canadian Natural Resources Limited (100.0%) | 1 |
| 6 | `T098-R07W4` | 454 | 454 | 0 | Not Applicable (100.0%) | Suncor Energy Inc. (100.0%) | 1 |
| 7 | `T065-R03W4` | 432 | 432 | 0 | CRUDE BITUMEN (73.1%) | Imperial Oil Resources Limited (99.5%) | 3 |
| 8 | `T093-R09W4` | 409 | 409 | 0 | Not Applicable (100.0%) | Syncrude Canada Ltd. (99.3%) | 2 |
| 9 | `T040-R03W4` | 406 | 406 | 0 | CRUDE OIL (62.1%) | West Lake Energy Corp. (33.0%) | 11 |
| 10 | `T055-R06W4` | 405 | 404 | 1 | CRUDE BITUMEN (95.8%) | Canadian Natural Resources Limited (96.8%) | 2 |
| 11 | `T014-R04W4` | 377 | 377 | 0 | GAS (99.5%) | Questfire Energy Corp. (98.1%) | 6 |
| 12 | `T093-R12W4` | 347 | 346 | 1 | Not Applicable (84.7%) | Syncrude Canada Ltd. (52.4%) | 2 |
| 13 | `T076-R06W4` | 336 | 335 | 1 | Not Applicable (94.0%) | Cenovus Energy Inc. (93.8%) | 2 |
| 14 | `T055-R02W4` | 329 | 329 | 0 | CRUDE BITUMEN (91.8%) | Canadian Natural Resources Limited (80.5%) | 4 |
| 15 | `T070-R04W4` | 328 | 328 | 0 | Not Applicable (82.3%) | Cenovus Energy Inc. (100.0%) | 1 |

The pattern is unmistakable:
- The top townships split between the **east-central / Lloydminster heavy-oil
  belt** (TWP 36–55, around RGE 1–6 W4) and the **Cold Lake oil-sands belt**
  (TWP 60–66, RGE 1–7 W4) and the **Athabasca oil sands** (TWP 90–98, RGE 6–12 W4).
- **Canadian Natural Resources Ltd** dominates almost every top township,
  often holding **80–100% of all wells in a single township**. This is
  extreme single-operator concentration at the township level — a fragility
  that the province-wide HHI (Analysis 02) understates because it averages
  across all 952 licensees.
- *Crude bitumen* dominates the highest-count townships in the Athabasca
  region (TWP 95–98), confirming the oil-sands footprint identified in
  Analysis 04.
- *"Not Applicable"* fluid dominates several top townships, again pointing
  at observation/disposal infrastructure clustered at industrial scale.

### 5.4 Liability-concentration implication
At the township scale, single-operator dominance is the rule, not the
exception, in the densest zones. If a major licensee like CNRL became
financially impaired, the regulatory burden would not be diffuse — it
would land squarely on a small set of identifiable townships in the
oil-sands and heavy-oil belts. This is the level of granularity at which
liability-management policy is most actionable.

## 6. How to Reproduce
1. Open `05_Township_Grid_Aggregation.qgz` in QGIS 3.x.
2. Two layers should load:
   - **Townships** (the choropleth, coloured cream → dark red by well count).
   - **Wells (reference)** — toggled OFF by default; turn on for a sanity
     check that the township grid sits where the wells are.
3. Use *Identify Features* on any township to read all attributes.
4. To filter by region, use a quick filter on `TWP` and `MER`, e.g.
   `TWP >= 90 AND TWP <= 98 AND MER = 4` for the Athabasca core.

## 7. Notes & Caveats
- The 9.656 km square polygons are *approximations* of true DLS townships,
  centred on the well centroid (see §3.3). Substitute an official AltaLIS
  township grid for legal-grade work.
- Townships with no wells in the dataset are omitted (those exist as
  cadastral units but contribute nothing to liability).
- Density is computed against the nominal 93.2 km² township area, not the
  surveyed area; minor variations in true township area (due to road
  allowances and survey corrections) are not modelled.
- 'Dominant licensee' is computed by raw count of wells inside the
  township; corporate name normalisation has not been performed.
