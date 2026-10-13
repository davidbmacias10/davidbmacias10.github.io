---
layout: page
title: Campus Market Share Segmentation
description: Segmenting charter campuses by demographic market share and enrollment-target fit using H3 geospatial markets.
img: assets/img/market_2.jpeg
importance: 1
category: work
related_publications: false
---

<!--
  FILL-INS: every [bracketed] value comes from your own output files
  (enrollment_gap_summary_2526.csv and enrollment_target_feasibility.xlsx).
  Image files market_1.jpg to market_4.jpg still need to be created from those outputs.
  Keep campus-level results aggregated: the enrollment targets are internal data.
-->

# Project Summary

## Introduction

This project segments every IDEA Public Schools campus in Texas along two questions leadership kept asking separately:

1. **Market position:** Does the campus serve its fair share of the students who live around it, compared with the traditional public (ISD) schools nearby?
2. **Target fit:** Is the campus's 2027 enrollment target sized to what that local market can actually support?

Crossing the two answers puts each campus in one of nine segments. Each segment comes with a recommended action on its target (increase, hold or reduce) and a strength of evidence. The goal was to turn "is this target realistic?" from a debate into a defensible, data-backed call.

---

# Phase 1: Data Preparation & Defining the Market

I combined three sources, each answering a different question about the local student population:

| Data | What it tells us | Role in the analysis |
|------|------------------|----------------------|
| **IDEA and competitor enrollment** (TEA, 2024–25 and 2025–26) | What the student population looks like at every campus, by special population, ethnicity and grade | The baseline. If neither we nor our competitors have the kids, the gap isn't about us |
| **HHS food stamp (SNAP) data** | Children receiving SNAP by county, updated monthly | The fastest-moving signal of what a county is facing, though counties are large and the population isn't evenly spread |
| **Traffic cameras** | Daily movement into and out of cities and neighborhoods | Confirms the two population measures above and catches surges as families move |

**Comparisons are regional.** ISD competitors in El Paso can't be compared with those in Austin, so every market and target was built within its own region.

**Granularity:** Market share uses whole-campus enrollment (elementary, middle and high), because Kindergarten alone is too thin a slice to benchmark a campus on. Kindergarten targets were handled separately, by comparing our Kinder enrollment with competitors' and with HHS data showing that the under-5 population exists.

Key preparation steps:
- Campus numbers arrived with leading apostrophes in some files, so I stripped them and cast every ID to an integer before joining
- 2025–26 files were the source of truth for which IDEA campuses exist; IDEA campuses without coordinates were listed for review instead of silently dropped
- Each campus was assigned an **H3 hexagon (resolution 7)** from its latitude and longitude

```python
H3_RESOLUTION = 7
K2_RING_SIZE = 2

def clean_campus_id(series):
    """Strip leading apostrophes/whitespace and convert to int."""
    return (series.astype(str).str.lstrip("'").str.strip()
            .pipe(pd.to_numeric, errors="coerce").astype("Int64"))

schools["campus_id"] = clean_campus_id(schools["USER_School_Number"])
schools["hex_id"] = schools.apply(
    lambda r: h3.latlng_to_cell(r["Y"], r["X"], H3_RESOLUTION), axis=1)
```

### What counts as a campus's "market"

District boundaries don't describe where families actually choose between schools, so I defined each campus's market geographically: **its own hexagon plus every hexagon within two rings of it**. Every ISD campus inside that area counts as local competition.

```python
k2_hexes = h3.grid_disk(idea_hex, K2_RING_SIZE)
local_isd = isd_df[isd_df["hex_id"].isin(k2_hexes)]
```

![IDEA Campuses and Their Local Market Areas](/assets/img/market_1.jpeg)

**Coverage:** [N] IDEA campuses, compared against [N] ISD campuses inside their market areas.

---

# Phase 2: Measuring Market Position

For each demographic, I compared the share of IDEA's enrollment in that group with the average share at the surrounding ISD schools:

**Market ratio = IDEA campus % of a demographic ÷ average % at ISD campuses in its market**

A ratio of 1.0 means the campus serves exactly its fair share. The ratio works as a benchmark: if a neighboring school serves 10 special education students out of 100, an IDEA campus in the same area should serve at least that share, and ideally more.

### Key demographics

| Demographic | Why it matters |
|-------------|----------------|
| **Economically disadvantaged** | The strongest indicator. It tracks closely with HHS data, enrollment data and competitor data. When this population shifts in a county, city, region or at our competitors, we can expect a shift as well |
| **Emergent bilingual** | A core part of the mission. If a competitor serves a higher share, so should we |
| **Special education** | Same benchmark logic as emergent bilingual |

I classified each ratio into five tiers:

| Market Ratio | Market Position |
|--------------|-----------------|
| > 1.15 | Well Above |
| 1.10 – 1.15 | Above |
| 0.95 – 1.10 | At |
| 0.85 – 0.95 | Below |
| < 0.85 | Well Below |

### Which demographic decides

A campus can be above market on one group and below on another, so a single "market share" number needs a rule for which demographic matters. I tiered demographics by **how much of the area's enrollment they make up**, then cascaded:

1. Rank the area's demographics by share of total area enrollment
2. Check the campus's ratio on the largest group
3. If the campus is already above market there, move to the next largest
4. Report the first demographic where the campus is **not** above market

This keeps the analysis focused on the population that defines each neighborhood, and it surfaces the biggest group a campus is under-serving.

---

# Phase 3: Kids to Parity & Market Saturation

### How many students would close the gap?

For campuses below market, I calculated **kids to parity**: the additional students of the primary demographic needed to reach a ratio of 1.0. Adding students raises both the numerator and the denominator of the campus's percentage, so a simple subtraction undercounts. Solving for the right number:

```python
# Solve (idea_demo_count + extra) / (idea_enrollment + extra) = isd_avg_pct
raw_parity = (isd_avg_pct * idea_enrollment - idea_demo_count) / (1 - isd_avg_pct)
kids_to_parity = max(0, round(raw_parity))
```

The same formula gives the students needed to reach Above (1.10) and Well Above (1.15).

### Are those students actually there?

A gap only matters if the kids exist to recruit. I flagged a market as **saturated** when closing the gap would require:
- More than **20%** of all students of that demographic in the area, or
- More than **10%** of that demographic from **multiple** ISD campuses at once

The first version flagged saturation if any single campus would lose 10%. I changed it because families choose schools individually, and one small neighboring campus shouldn't mark an otherwise large, healthy market as full.

![Kids to Parity for Below-Market Campuses](/assets/img/market_2.jpeg)

---

# Phase 4: Measuring Target Fit

Next I compared each campus's 2027 enrollment target with what its market supports:
- Summed the 2027 target across grades, excluding pre-K
- Matched target-file school names to TEA campuses with exact matching first, then fuzzy matching (`rapidfuzz` token sort); every fuzzy match was printed for review, and matches under 75% confidence were flagged
- Defined **fair market capacity** = current enrollment + kids to parity

This grounds each target in the kids who actually live nearby. If the students exist, a campus can enroll some of them. If an area has a small population, it's hard to argue a campus is under-targeted.

```python
ALIGN_BAND = 0.10

def feasibility_flag(target_27, fair_market_capacity):
    ratio = target_27 / fair_market_capacity
    if ratio > 1 + ALIGN_BAND:
        return "OVER TARGET"    # target exceeds what the market supports
    if ratio < 1 - ALIGN_BAND:
        return "BELOW TARGET"   # kids exist that the target doesn't plan for
    return "AT TARGET"          # within ±10% of fair market capacity

report["fair_market_capacity"] = report["idea_total_enrollment"] + report["primary_kids_to_parity"]
report["feasibility"] = report.apply(
    lambda r: feasibility_flag(r["sum_target_27"], r["fair_market_capacity"]), axis=1)
```

---

# Phase 5: The Nine Segments

### Reading each position

| Position | Below | At | Above |
|----------|-------|----|-------|
| **Market** | Neighboring schools serve a higher share of the key demographics (special education, emergent bilingual, economically disadvantaged) than we do | We serve the same share as our neighboring competition | We serve a meaningfully higher share of the key demographics, a sign these families are choosing IDEA for its academics |
| **Target** | There are kids in the area, and we should be enrolling more of them | We are following the enrollment trends of our neighboring competition | We have taken more than our fair share, to the point that our enrollment rises as neighboring schools' enrollment falls |

Crossing market position (Below / At / Above) with target fit (Below / At / Over) gives nine segments. For this matrix, Well Below rolls into Below and Well Above into Above. Market position decides **whether a campus can be pushed** for more enrollment, and target fit decides **which way its target should move**.

![Campuses by Market Position and Target Fit](/assets/img/market_3.png)

| Market | Target | What it means | Target Action | Strength of Case |
|--------|--------|---------------|---------------|------------------|
| Below | Below | Under its fair share, and the target doesn't ask it to close the gap | Increase | Strong |
| Below | At | Under its fair share; the target is sized to close the gap | Hold | n/a |
| Below | Over | Under its fair share, but the target asks for more than parity would deliver | Reduce to fair market capacity | Moderate |
| At | Below | Holds its fair share, but the target plans for a decline | Increase to about current enrollment | Moderate |
| At | At | Steady state: enrollment, share and target agree | Hold | n/a |
| At | Over | Holds its fair share; target asks for 10%+ growth with no market gap to fund it | Reduce | Moderate |
| Above | Below | Outperforms its market, but the target plans a drop | Increase | Weak |
| Above | At | Outperforms its market; the target asks it to hold | Hold | n/a |
| Above | Over | Already past fair share, and the target asks for 10%+ more | Reduce | Strong |

### Results

| Segment | Campuses | Recommended Action |
|---------|----------|--------------------|
| Below market, Below target | [N] | Increase target |
| Below market, At target | [N] | Hold |
| Below market, Over target | [N] | Reduce to fair market capacity |
| At market, Below target | [N] | Increase target |
| At market, At target | [N] | Hold |
| At market, Over target | [N] | Reduce |
| Above market, Below target | [N] | Increase (weak case) |
| Above market, At target | [N] | Hold |
| Above market, Over target | [N] | Reduce (strong case) |

The output went to leadership as one workbook with three tabs: **Growth Potential** (below target), **Sustain** (at target) and **Saturated** (over target). Each row showed the campus, its primary market status, kids to parity, saturation flag and target gap.

![Target Gap by Segment](/assets/img/market_4.png)

---

# Final Thoughts

Key lessons from the project:
- **Define the market the way families experience it.** Hexagon rings around each campus captured local competition better than district lines.
- **Segmentation needs a decision rule, a metric is not enough to go off of.** The demographic cascade turned dozens of ratios per campus into one defensible number.
- **Opportunity is not attainability.** This analysis shows whether the students exist and whether a target is sized to them. It does not show whether a campus can recruit them. Keeping those two claims separate is what made the results credible with leadership.
- **Pairing each segment with an action and a strength of evidence** turned the analysis from a report into a tool for target-setting conversations.

---

# Future Work

- **Grade-level attainability:** compare each grade's target with its historical new-student intake, track record against past targets, and applications and waitlists
- **Demand signals:** bring in lottery and waitlist data so "Room to Grow" reflects interest, not just population
- **Leading indicators:** extend the Kindergarten work comparing enrollment growth with county SNAP under-5 trends to other entry grades
- **Significance testing:** test whether above-market ratios are statistically significant rather than relying on a fixed 1.10 threshold
- **Seat capacity:** add building and staffing limits so recommended targets are physically achievable
