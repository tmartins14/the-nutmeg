# The Nutmeg

*Football, Data, Storytelling. Period.*

## What this is

A data-driven football publication. One piece a month — 
rigorously researched, visually interactive, written for 
anyone who loves the game. We find the numbers that change 
how you see football, then we make them impossible to 
look away from.

## Status

Pre-launch. First piece in development.

## Stack

- **Analysis:** Python (Pandas, NumPy, scikit-learn)
- **Visualisation:** D3.js
- **Writing:** Markdown
- **Publishing:** TBD (GitHub Pages / Observable / standalone site)

## Data sources

- Understat — shot-level xG, top 5 European leagues from 2014
- StatsBomb Open Data — event-level data, selected competitions
- Transfermarkt — market values, transfers, career timelines
- Public domain — historical World Cup and tournament data

## Pieces

| # | Slug | Status | Target publish |
|---|------|--------|----------------|
| 01 | world-cup-48 | In development | May 2026 |

## Repo structure

Each piece lives in `pieces/[slug]/` with the following structure:

- `data/raw/` — source data, not committed to git
- `data/processed/` — cleaned data, committed
- `analysis/` — notebooks and scripts
- `writing/` — markdown drafts
- `viz/` — D3 source and output
- `src/` — shared, reusable code across all pieces
    - `src/data/` — data loaders and cleaning functions
    - `src/viz/` — visualisation utilities and colour palette
    - `src/utils/` — general helper functions

Each piece lives in `pieces/[slug]/` and imports from `src/` 
where possible, keeping piece-level code focused on 
piece-specific logic only.

## Licenses

- Code and analysis: MIT
- Writing and compiled data: CC BY 4.0