# 48 Teams, One World Cup: Did Football Just Break Its Own Showpiece?

**A data journalism project by The Nutmeg**

---

## What this project is

A long-form visual story interrogating what FIFA's expansion of the World Cup — from 32 to 48 teams — actually does to the tournament's competitive quality. Not opinion. Not punditry. Evidence.

The piece holds two tensions simultaneously: the genuine case for inclusion and global football development, and the measurable risk that more teams means more mismatches and a diluted group stage product. We follow the data wherever it goes.

Published methodology, data sources, and processing scripts are documented here in full.

---

## Research questions

This project is structured around six core questions the data must answer before any findings are reported:

1. **Scoreline trend** — Is average goal differential in World Cup group stage matches trending up or down over time, and is the trend statistically meaningful or noise?
2. **Upset definition** — How do we operationalise an "upset"? Scoreline alone is insufficient. We define upsets using Elo-expected outcomes: a win by the lower-ranked side, weighted by the pre-match Elo gap.
3. **Elo distribution shift** — What does the distribution of Elo rating gaps look like in the historical 32-team field, and how does it change when we model the addition of the next 16 qualifiers by confederation allocation?
4. **Format incentive structures** — Does the groups-of-3 format change competitive incentives in a measurable way? Specifically: does the elimination of one group match per team increase the probability of tactical draws and dead rubbers?
5. **Expansion signatures** — Did previous expansions (1982: 16→24 teams; 1998: 24→32 teams) leave a visible signature in the competitiveness data? If so, what kind, and how quickly did the tournament adjust?
6. **New entrant profiles** — Which confederations absorb the new berths, and what is the historical Elo distribution within those confederations at World Cup entry?

---

## Data sources

### 1. World Cup match results (1930–2022)

**Primary source:** [Kaggle — FIFA World Cup Dataset](https://www.kaggle.com/datasets/abecklas/fifa-world-cup) (scraped from official FIFA records and Wikipedia match data)

**Fields used:**
- Year, stage, match ID
- Home team, away team
- Full-time score (home goals, away goals)
- Host nation flag

**Scope:** All matches, all editions. Analysis focuses on group stage matches only, but full dataset is retained for knockout comparison work.

**Known limitations:**
- Pre-1966 group stage formats differ structurally from the modern format; 1930–1962 data is retained for historical context but excluded from primary trend analysis
- Score data for some early editions sourced from Wikipedia and subject to minor transcription error

---

### 2. National team Elo ratings (historical)

**Source:** [eloratings.net](https://www.eloratings.net) — maintains a continuous historical Elo series for international football from 1872 to present

**Fields used:**
- Team name
- Elo rating at tournament entry (defined as the rating on the date of each team's first group stage match)
- Peak Elo by tournament edition

**Why Elo over FIFA Rankings:**
The FIFA World Ranking uses a proprietary points system that has been revised multiple times and is known to reward volume of competitive fixtures over quality. Elo ratings are zero-sum, historically continuous, and widely used in football analytics literature. For cross-era comparison, Elo is the only viable option.

**Known limitations:**
- Elo ratings are sensitive to result recency; teams in low-fixture windows (e.g. emerging nations) may carry more uncertainty
- The eloratings.net methodology applies a base rating of 1500 for new entrants, which may understate the quality of nations with strong domestic leagues but limited international history

---

### 3. 2026 qualifier Elo profiles (modelled)

**Source:** eloratings.net current ratings + confederation allocation modelling

**Method:** FIFA's confirmed confederation berth allocations for 2026 are used to identify the marginal qualifying slots per confederation (i.e. the slots that did not exist under the 32-team format). For each confederation, we take the current Elo distribution of nations ranked just outside the historic qualification threshold and use these as the likely profile of the 16 new entrants.

This is a projection, not a confirmed squad list. It is presented as a distributional estimate, not a prediction of specific nations.

**Confederation berth allocations (32→48):**

| Confederation | 2022 berths | 2026 berths | New slots |
|---------------|-------------|-------------|-----------|
| UEFA (Europe) | 13 | 16 | +3 |
| CONMEBOL (S. America) | 4.5 | 6 | +1.5 |
| CAF (Africa) | 5 | 9 | +4 |
| AFC (Asia) | 4.5 | 8 | +3.5 |
| CONCACAF (N/C America) | 3.5 | 6 | +2.5 |
| OFC (Oceania) | 0.5 | 1 | +0.5 |
| Inter-confederation playoffs | 1.5 | 2 | — |

*Source: FIFA.com confirmed allocation, 2023*

---

## Methodology

### Competitive intensity score

The primary metric used in the centrepiece heat map visualisation is a **Competitive Intensity Score (CIS)** defined per match as follows:

```
CIS = 1 − (|actual_goal_diff| / max_normaliser) × elo_gap_weight
```

Where:
- `actual_goal_diff` = absolute goal difference at full time
- `max_normaliser` = 7 (the 95th percentile of historical group stage goal differentials; scores beyond this are treated as maximum blowouts)
- `elo_gap_weight` = a scaling factor derived from the pre-match Elo difference between the two sides, normalised to [0.5, 1.5]

A CIS of 1.0 represents maximum competitive intensity (close match between evenly rated sides). A CIS approaching 0 represents a dominant result between unevenly rated sides.

**Rationale:** Raw scoreline alone conflates two different things — quality of opposition and margin of victory. A 3–0 result between sides separated by 400 Elo points tells us something different from a 3–0 between sides rated within 50 points of each other. The CIS attempts to weight for this distinction.

---

### Upset definition

A match is classified as an **upset** if:
- The lower-Elo side wins, **and**
- The pre-match Elo gap is ≥ 100 points (i.e. the favourite was meaningfully favoured)

An Elo gap of 100 points corresponds roughly to a win probability of ~64% for the higher-rated side. This is a conservative threshold — we are not counting every underdog win, only those where the result genuinely defied expectation.

**Upset rate** is calculated per tournament edition as:

```
upset_rate = upsets / total_group_stage_matches_with_elo_gap_≥100
```

This denominator adjusts for the varying number of matches per edition and for the proportion of matches featuring a meaningful Elo gap.

---

### Expansion comparison framework

The 1982 and 1998 expansions are used as historical analogues. For each, we calculate:

1. Mean CIS for the three editions before expansion
2. Mean CIS in the first edition post-expansion
3. Mean CIS across the following two editions (to capture reversion effects)

This produces a before/during/after competitive profile for each expansion event, against which 2026 projections are benchmarked.

---

## Repo structure

```
/
├── README.md               ← You are here
├── data/
│   ├── raw/                ← Source data, unmodified
│   │   ├── wc_matches.csv
│   │   ├── elo_ratings.csv
│   │   └── confederation_allocations.csv
│   ├── processed/          ← Cleaned and transformed data
│   │   ├── group_stage_matches.csv
│   │   ├── matches_with_elo.csv
│   │   └── cis_by_match.csv
│   └── modelled/           ← 2026 projections
│       └── new_entrant_profiles.csv
├── scripts/
│   ├── 01_clean_matches.py
│   ├── 02_merge_elo.py
│   ├── 03_calculate_cis.py
│   ├── 04_upset_rate.py
│   └── 05_model_2026_entrants.py
├── visuals/
│   ├── heatmap/            ← Centrepiece scrollytelling visual
│   ├── upset_rate_chart/
│   ├── new_entrant_scatter/
│   └── format_explainer/
└── notes/
    ├── editorial_structure.md
    ├── data_qa_log.md
    └── findings_draft.md
```

---

## A note on uncertainty

This piece makes projections about 2026. Those projections are modelled from current Elo ratings and historical qualification patterns — they are not confirmed team lists. The further the projection from confirmed data, the wider the uncertainty band.

Where projections are used in visuals, uncertainty is shown explicitly. We do not present modelled findings as fact.

---

## Editorial stance on the data

We started this project with two competing hypotheses — that expansion dilutes quality, and that the new entrants may be more competitive than critics claim. We have not predetermined which is true.

Previous expansion events (1982, 1998) did not straightforwardly damage competitive quality by every metric. If the 2026 data points in a similarly complicated direction, we report that. The piece follows the evidence.

---

## Contact

**The Nutmeg** — editorial team  
For data queries or methodology questions, open an issue in this repo.

---

*Last updated: work in progress — methodology subject to revision as data acquisition proceeds.*