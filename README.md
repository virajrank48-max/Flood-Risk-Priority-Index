# Flood Risk Priority Index for Indian Catchments

**Where should flood management and preparedness be prioritised? A hazard-and-exposure ranking of Indian river catchments, with a Monte Carlo test of how stable the ranking is.**

Flood *counts* are a poor guide to priority: a catchment can record many small floods in a sparsely populated area, while another records fewer floods but contains dense cities, roads and economic activity. This project builds a catchment-level **Flood Risk Priority Index** from the [INDOFLOODS](https://doi.org/10.5281/zenodo.14584654) database by combining a **hazard score** with an **exposure/vulnerability score**, then checks the ranking with sensitivity analysis and a simulation of alternative flood histories.

> The index is a **screening and prioritisation tool**. It is not a prediction of economic loss and not an official risk rating.

---

## Key results

- **Data:** 4,548 flood events at 155 river gauges, July 1965 to September 2020 (2,919 "Flood", 1,629 "Severe Flood").
- **Scored catchments:** 46 of 155. Only catchments with at least 20 events and 15 monitored years are ranked, so percentile scores are not built on a handful of floods.
- **Top 10 by risk-priority score:** T. bekkupe (319), Terwad (643), Yelli (679), Kuzhithurai (852), Biligundulu (394), T.k.halli (361), Pauni (571), Kallooppara (427), Samdoli (681), Gokak falls (397).
- **Not just a flood count:** the index correlates only weakly with event count (Spearman rho = 0.27, p = 0.07) and flood frequency (rho = 0.23, p = 0.12). Biligundulu has the most floods and the highest frequency (8.6 per year) but ranks 5th, because most of its floods are minor.
- **Quadrants:** 13 catchments sit in the high-hazard, high-exposure quadrant (the first priority list), 12 low/low, 11 low-hazard/high-exposure, 10 high-hazard/low-exposure.
- **Typology (k-means, k = 4):** two groups share the highest mean risk (about 0.61) for different reasons, one flood-prone, the other asset-dense. Typology suggests the *type* of action; the ranking suggests the *order*.
- **Stable under reweighting:** top-10 overlap between hazard-focused, balanced and exposure-focused weights is 8 to 9 of 10. With 1,000 random indicator-weight draws, gauges 643, 319, 852, 361 and 679 stay in the top 10 in more than 95% of draws.
- **Monte Carlo (5,000 simulated 10-year futures):** index rank and median simulated rank agree (Spearman rho = 0.79).
  - **Robust priorities (5):** 319, 679, 361, 571, 427 (in the index top 10 and in the simulated top 10 in at least 80% of futures).
  - **Index top 10 but less stable (5):** 643, 852, 394, 397, 681.
  - **Additional candidate (1):** Thumpamon (429), outside the index top 10 but in the simulated top 10 in 82% of futures.
  - Across 5, 10 and 20-year horizons, 8 of the index top 10 reach the simulated top 10 in at least half of futures.
- **Geography:** high-priority catchments cluster in peninsular India (Cauvery, Krishna, Godavari, and west-flowing rivers of Kerala and Tamil Nadu). This partly reflects data coverage: most of the Ganga-Brahmaputra plains are unranked, so the maps say nothing about flood risk there.
- **Context check:** state-level agreement with the district flood severity index (DFSI) is weak and not significant (rho = 0.39, p = 0.26, about 10 states). The index is **not independently validated**.

## Method

| Step | What it does |
|---|---|
| 1. Load and clean | Standardise names, parse dates, link tables on `GaugeID`, drop duplicate events; durations below 0 or above 60 days and zero 3-day/5-day rainfall totals set to missing |
| 2. Exploratory analysis | Flood types, durations, seasonality, annual counts per active gauge, rainfall versus flood behaviour (between catchments and within gauges) |
| 3. Hazard indicators | Flood frequency per monitored year, severe-flood share, 95th-percentile duration. Rainfall and height above danger level are computed but not scored (see Limitations) |
| 4. Outlier masking | 103 events at 17 gauges whose peak level is more than 50 m from the danger level (mostly datum mismatches, plus clear recording errors) set to missing |
| 5. Exposure and vulnerability | Population density, urban share, road density, night lights, GDP per person (PPP), low HDI (reversed), log population |
| 6. Index | Each indicator becomes a percentile score; hazard and exposure are the means of their components; **R = sqrt(Hazard x Exposure)** (geometric mean) |
| 7. Quadrants and typology | Median split of hazard and exposure; k-means on the two scores |
| 8. Alternative rankings | Comparison with raw event count, frequency and hazard-only rank |
| 9. Sensitivity | Three weighting scenarios and 1,000 random indicator-weight draws |
| 10. Monte Carlo | Yearly flood rate ~ Gamma (from observed events and years monitored), flood count ~ Poisson, severe share ~ Beta, expected impact = simulated severe floods x exposure score; catchments re-ranked in every simulated future |
| 11. DFSI comparison | State-level contextual comparison |
| 12. Maps | Static maps of India and an interactive HTML map |

The geometric mean stops high exposure with almost no hazard, or high hazard with almost no exposure, from dominating the ranking. Priority categories use the risk percentile (Low up to 40%, Moderate to 80%, High to 90%, Very high above 90%).

## Repository structure

```
.
├── indian_flood_risk_priority_index.ipynb
├── data/                      # INDOFLOODS files (see "Data"; not included)
├── outputs/                   # CSVs, static maps, interactive map
├── presentation/
│   └── Flood_Risk_Priority_Index.pptx
├── requirements.txt
└── README.md
```

## How to run

```bash
git clone https://github.com/<your-username>/<repo-name>.git
cd <repo-name>
pip install -r requirements.txt
# download the data files into ./data (see below), then:
jupyter notebook indian_flood_risk_priority_index.ipynb
```

`requirements.txt`:

```
pandas
numpy
scipy
scikit-learn
matplotlib
seaborn
geopandas
folium
mapclassify
branca
jupyter
```

Run all cells. Random seeds are fixed (42), so the clustering and simulations reproduce. The first run of the map section downloads a national outline of India from the DataMeet public repository, so it needs an internet connection.

## Data

Download from the INDOFLOODS Zenodo record and place in `data/`:

- `floodevents_indofloods.csv`
- `precipitation_variables_indofloods.csv`
- `catchment_characteristics_indofloods.csv`
- `metadata_indofloods.csv`
- `catchments_shapefiles_indofloods.zip` (needed for the maps)
- Optional: `DFSI.csv` (district flood severity index, used only for the context comparison)

**Sources and citation** (the INDOFLOODS portal asks users to cite both):

- Kuntla, S. K., & Saharia, M. (2025). INDOFLOODS: A comprehensive database for flood events in India enhanced with catchment attributes. *Bulletin of the American Meteorological Society*, 106(2), E333-E343. https://doi.org/10.1175/BAMS-D-24-0008.1
- Kuntla, S. K., & Saharia, M. (2025). INDOFLOODS [Data set]. Zenodo. https://doi.org/10.5281/zenodo.14584654
- DFSI: Saharia, M., Jain, S. K., Prakash, V., Malik, H., Sreejith, O. P., & Joshi, D. (2025). A district-level flood severity index for flood management in India. *Natural Hazards*. https://doi.org/10.1007/s11069-025-07493-9 (file distributed with the India Flood Inventory-Impacts record on Zenodo).

The data are openly available for academic and research use. The raw files are not redistributed in this repository.

## Limitations

- **Relative, not absolute.** Scores are percentiles within the 46 scored catchments; they rank catchments against each other and say nothing about absolute risk.
- **Coverage.** Only 46 of 155 catchments are ranked. Peninsular India dominates the ranking partly because long, event-rich records exist there.
- **Narrow hazard score.** It uses three indicators. 3-day and 5-day rainfall are almost the same variable (correlation 0.98) and height above danger level had datum problems, so both are reported but not scored.
- **Time mismatch.** Exposure variables are about 2015; floods span 1965-2020.
- **Nested catchments and correlated indicators.** Upstream and downstream catchments overlap, and some exposure indicators move together, so effective independence is lower than the count suggests. Catchment averages also hide local hotspots.
- **Simulation scope.** It tests how stable the ranking is under random variation in flood occurrence and severity, assuming rates from the observed record. It does not forecast actual floods. Expected impact (severe floods x exposure score) is a ranking heuristic, not a loss estimate.
- **Weak validation.** The DFSI comparison is state-level and not significant; district and catchment boundaries differ.
- **Peak level is an elevation, not depth.** It is never compared across gauges; height above each gauge's own danger level is used instead.

## Possible extensions

Score rainfall and danger-level exceedance after resolving datum issues; weight indicators with expert or stakeholder input; add flood impact data (district-level damages) for validation; extend coverage to more gauges and to the Ganga-Brahmaputra plains.

## Team

Jessica Dhabhai, Viraj Rank

## License

[FILL IN: e.g. MIT for the code.] Data remain subject to the INDOFLOODS and DFSI terms above.