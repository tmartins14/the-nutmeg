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
4. **Format incentive structures** — Does the addition of a Round of 32 and the eight best third-place qualification rule change competitive incentives in a measurable way? Specifically: does the possibility of advancing as a third-place team reduce the urgency of the final group match for teams sitting third, increasing the likelihood of low-stakes football?
5. **Expansion signatures** — Did previous expansions (1982: 16→24 teams; 1998: 24→32 teams) leave a visible signature in the competitiveness data? If so, what kind, and how quickly did the tournament adjust?
6. **New entrant profiles** — Which confederations absorb the new berths, and what is the historical Elo distribution within those confederations at World Cup entry?

---

## Data sources

All raw data lives in `pieces/01-world-cup-48/data/raw/` and is not modified after acquisition. All transformations happen in processing scripts.

### 1. World Cup match data (stage + results, 1930–2022)

**File:** `world_cup_matches.csv`  
**Source:** [Fjelstul World Cup Database](https://github.com/jfjelstul/worldcup), Joshua C. Fjelstul, Ph.D. — CC BY-SA 4.0  
**Coverage:** All men's World Cup matches, 1930–2022 (22 tournaments, 964 matches after filtering women's tournaments)  
**Key fields:** Tournament ID, match date, stage name, group name, group stage flag, knockout stage flag, home/away team name, home/away score, extra time flag, penalty shootout flag  
**Used for:** Identifying group stage matches, stage lookup for merging with results data

---

### 2. International match results (full history)

**File:** `results.csv`  
**Source:** [Kaggle — International Football Results from 1872 to 2026](https://www.kaggle.com/datasets/martj42/international-football-results-from-1872-to-2017), maintained by Mart Jürisoo  
**Coverage:** All international fixtures, 1872–2026 (49,287 matches)  
**Key fields:** Date, home team, away team, home score, away score, tournament, city, country, neutral ground flag  
**Used for:** Calculating national team Elo ratings from first principles across the full historical record. Primary source of match results.

---

### 3. International shootout results

**File:** `shootouts.csv`  
**Source:** [Kaggle — International Football Results from 1872 to 2026](https://www.kaggle.com/datasets/martj42/international-football-results-from-1872-to-2017)  
**Coverage:** 675 penalty shootout outcomes across international matches  
**Key fields:** Date, home team, away team, winner  
**Used for:** Identifying shootout matches in Elo calculation. Shootout results are treated as draws — the 90 minutes of football is the signal; penalty outcomes are not.

---

### 4. Former country names

**File:** `former_names.csv`  
**Source:** [Kaggle — International Football Results from 1872 to 2026](https://www.kaggle.com/datasets/martj42/international-football-results-from-1872-to-2017)  
**Coverage:** Name mappings for nations that have changed names or dissolved  
**Key fields:** Current name, former name, years active  
**Used for:** Reference during team name normalisation

---

### 5. International goalscorers

**File:** `goalscorers.csv`  
**Source:** [Kaggle — International Football Results from 1872 to 2026](https://www.kaggle.com/datasets/martj42/international-football-results-from-1872-to-2017)  
**Coverage:** Goal-level records for international matches  
**Key fields:** Date, home team, away team, scorer, minute, own goal flag, penalty flag  
**Used for:** Supplementary; not part of primary analysis

---

### 6. 2026 qualifier Elo profiles (modelled — not yet built)

**Source:** Derived from `results.csv` Elo calculations + FIFA confirmed confederation allocations  
**Method:** The 16 new berths created by expansion are distributed by confederation. For each confederation, the Elo distribution of nations at the margin of historic qualification thresholds is used to estimate the likely competitive profile of new entrants. This is a projection, not a confirmed team list.

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

### Elo ratings

Elo ratings are calculated from scratch using `results.csv` — the full international match history from 1872 to 2026. This gives us each team's Elo at any point in time, including at the start of each World Cup group stage.

We follow the eloratings.net methodology:
- All teams initialised at 1500
- K-factor varies by match type: 60 (World Cup matches), 50 (continental championships), 40 (World Cup qualifiers and other major qualifiers), 30 (other competitive matches), 20 (friendlies)
- K adjusted upward for goal margin: ×1.5 for 2-goal wins, ×1.75 for 3-goal wins, scaling further beyond that
- Shootout results recorded as draws for Elo purposes — `shootouts.csv` identifies which matches went to penalties; the shootout winner is irrelevant to Elo
- Home advantage: +100 Elo points added to home team's effective rating when match is not on neutral ground

**Validation:** Germany's Elo at 2014 WC entry: 2102. Brazil's Elo at 1970 WC entry: 1881. Top-rated team ever at WC entry: Brazil 2022 at 2229. All values consistent with expectations.

**Note on cross-era comparison:** Absolute Elo values are not directly comparable across eras — the rating pool inflates over time as more teams accumulate history. All cross-era analysis uses Elo *gaps between teams within a tournament* rather than absolute values.

**Known exclusions** — three teams appear in WC match data but have no Elo rating due to gaps in the results record:
- East Germany (1974) — dissolved, no results data
- Yugoslavia (1998) — banned from international football 1992–1994; results data ends 1992
- Serbia and Montenegro (2006) — not present as a combined entity in results.csv

---

### Competitive Intensity Score (CIS)

The primary metric for the centrepiece heat map. Defined per match as:

```
CIS = 1 − (|actual_goal_diff| / max_normaliser) × elo_gap_weight
```

Where:
- `actual_goal_diff` = absolute goal difference at full time
- `max_normaliser` = 7 (95th percentile of historical group stage goal differentials)
- `elo_gap_weight` = pre-match Elo difference between the two sides, normalised to [0.5, 1.5]

A CIS of 1.0 = maximum competitive intensity (close match, evenly rated sides). A CIS near 0 = dominant result between unevenly rated sides.

**Note:** The `elo_gap_weight` scaling range of [0.5, 1.5] is a design choice subject to sensitivity testing. The analysis will be run at multiple weighting schemes before the range is locked.

---

### Upset definition

A match is classified as an **upset** if:
- The lower-Elo side wins, **and**
- The pre-match Elo gap is ≥ 100 points

An Elo gap of 100 points corresponds to roughly a 64% win probability for the higher-rated side. Sensitivity checks will also be run at 75 and 125 point thresholds.

**Upset rate** per tournament:

```
upset_rate = upsets / total_group_stage_matches_with_elo_gap_≥100
```

---

### Format change: what actually changed in 2026

FIFA considered and rejected a 16 groups of 3 format in favour of 12 groups of 4. The group stage structure is therefore largely unchanged from previous tournaments — each team plays 3 matches, top 2 from each group advance automatically.

The meaningful structural changes are:

- **48 teams instead of 32** — 12 groups instead of 8
- **Round of 32 is new** — the 8 best third-placed teams also advance, creating a knockout round that has never existed before
- **104 total matches** — up from 64
- **39-day tournament** — up from 32 days

The third-place qualification rule is the format element most worth interrogating for competitive incentives. A team sitting third after two matches faces an ambiguous decision: they don't know the qualifying threshold until all groups finish, which means they can't know whether pressing for a win is necessary or whether a draw will do. This is a real incentive distortion worth testing against historical data on how teams perform in final group matches when already eliminated vs. when chasing qualification.

---

### Expansion comparison framework

The 1982 (16→24) and 1998 (24→32) expansions are used as historical analogues. For each expansion event:

1. Mean CIS for the three editions before expansion
2. Mean CIS in the first post-expansion edition
3. Mean CIS across the two editions following (to capture reversion)

This produces a before/during/after competitive profile benchmarked against 2026 projections.

---

## Repo structure

```
pieces/01-world-cup-48/
├── README.md                   ← You are here
├── Makefile
├── data/
│   ├── raw/                    ← Source data, unmodified, not committed to git
│   │   ├── world_cup_matches.csv
│   │   ├── results.csv
│   │   ├── shootouts.csv
│   │   ├── goalscorers.csv
│   │   └── former_names.csv
│   └── processed/              ← Cleaned and transformed, committed to git
│       ├── wc_matches_clean.csv
│       ├── results_clean.csv
│       ├── elo_ratings.csv
│       ├── elo_at_wc_entry.csv
│       └── name_audit.txt
├── scripts/
│   ├── normalize_team_names.py
│   └── calculate_elo.py
├── analysis/
├── viz/
│   └── src/
└── writing/
    ├── draft.md
    └── notes.md

src/                            ← Shared utilities across all pieces
├── data/
│   ├── loaders.py
│   └── cleaners.py
├── viz/
└── utils/
```

---

## Status

| Task | Status |
|------|--------|
| Data acquisition | ✅ Complete |
| Team name normalisation | ✅ Complete |
| Elo calculation | ✅ Complete |
| CIS calculation | ⬜ Not started |
| Upset rate analysis | ⬜ Not started |
| 2026 entrant modelling | ⬜ Not started |
| Centrepiece heat map | ⬜ Not started |
| Upset rate chart | ⬜ Not started |
| New entrant scatter | ⬜ Not started |
| Format explainer | ⬜ Not started |
| First draft | ⬜ Not started |
| Final copy | ⬜ Not started |

---

## A note on uncertainty

This piece makes projections about 2026. Those projections are modelled from current Elo ratings and historical qualification patterns — they are not confirmed team lists. Where projections appear in visuals, uncertainty is shown explicitly. We do not present modelled findings as fact.

---

## Editorial stance

We started with two competing hypotheses — that expansion dilutes quality, and that new entrants may be more competitive than critics claim. We have not predetermined which is true. Previous expansions (1982, 1998) did not straightforwardly damage competitive quality by every metric. If 2026 points in a similarly complicated direction, we report that.

---

*Last updated: April 2026 — data acquisition, team name normalisation, and Elo calculation complete. CIS calculation next.*