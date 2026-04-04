# Trade Data Reconciliation Pipeline — Fragmentation Replication v1

A Python pipeline replicating the **CEPII/BACI methodology** (Gaulier & Zignago, 2010) for reconciling bilateral trade data from UN COMTRADE mirror flows.

> **Status:** Version 1 — initial implementation. This is an active work-in-progress, I will try to update as I go
> I am working on this as part of my work under the Ecnomics Program and Scholl Chair in International Business at CSIS

---

## Background

When countries report international trade to the United Nations, the **same shipment is recorded twice**: once by the exporting country (at FOB, Free on Board) and once by the importing country (at CIF: Cost, Insurance, Freight). These mirror reports rarely agree due to differences in valuation, timing, classification, and reporting quality.

The **BACI dataset**, produced by CEPII (Centre d'Études Prospectives et d'Informations Internationales), addresses this by applying a gravity-model-based methodology to:

1. **Estimate CIF/FOB ratios** using bilateral distance, contiguity, landlocked status, and product-level unit values
2. **Assess reporter reliability** through variance decomposition of mirror flow discrepancies
3. **Reconcile trade values** using reliability-weighted averages of export and import reports

This project replicates that methodology for the **CSIS fragmentation research program**, applying it to HS4-level trade data from the UN COMTRADE Tariff Data Module (TDM).

## What This Pipeline Does

The pipeline processes raw UN COMTRADE bilateral trade data and produces reconciled trade values through two complementary approaches:

### Prong 1 — Tiered CIF Adjustment (Interim)

A pragmatic baseline method that assigns fixed CIF rates based on geographic tiers (contiguous, same-continent, cross-continent). **This is not from the BACI paper** — it serves as a quick benchmark while the gravity model is validated.

### Prong 2 — Gravity Regression (BACI Replication)

The core methodology, implementing three key equations from Gaulier & Zignago (2010):

- **Eq (1) — CIF rate estimation:** `ln(CIF_rate) = α + β·ln(dist) + χ·ln(dist)² + δ·contig + φ·landlocked_i + γ·landlocked_j + η·ln(UV) + ε`
- **Eq (3) — Reconciliation weights:** `w = g(σ²_j) / [g(σ²_i) + g(σ²_j)]` where `g(σ²) = exp(σ²)(exp(σ²) - 1)`
- **Eq (4-5) — Reliability decomposition:** Variance components estimated from log-ratios of mirror flows

## Pipeline Structure

The notebook contains 25 cells (8 markdown + 17 code) organized as:

| Section | Description |
|---------|-------------|
| 1. Setup | Install dependencies, mount Google Drive, configure paths |
| 2. Extract & Load | Unzip TDM data, auto-detect encoding (UTF-16/UTF-8), load into DuckDB |
| 3. Validation | 11 automated quality checks on the raw data |
| 4. GeoDist & Metadata | Load CEPII bilateral distance data, landlocked sets, continent mapping |
| 5. Mirror Pairs | FULL OUTER JOIN of export and import reports; compute mirror ratios |
| 6. Prong 1 | Tiered CIF adjustment and simple-average reconciliation |
| 7. Prong 2 | Gravity regression, CIF prediction, reliability weights, weighted reconciliation |
| 8. Comparison | Side-by-side Prong 1 vs Prong 2 analysis; save outputs |

## Tech Stack

- **Google Colab** — runtime environment
- **DuckDB 0.10.3** — fast in-notebook analytical SQL engine
- **pandas / numpy** — data manipulation
- **statsmodels** — OLS gravity regression with robust standard errors
- **pycountry** — ISO country code mapping
- **CEPII GeoDist** — bilateral distance and geographic variables

## Data Requirements

| File | Source | Description | Size |
|------|--------|-------------|------|
| `_TDM_HS4_2015_2024.zip` | UN COMTRADE TDM | Bilateral trade data at HS4 level (compressed) | 1.53 GB |
| `TDM_report.csv` | Pipeline output from above zip | Extracted trade report used by the notebook | 9.17 GB |
| `dist_cepii.csv` | [CEPII GeoDist](http://www.cepii.fr/CEPII/en/bdd_modele/bdd_modele_item.asp?id=6) | Bilateral distances, contiguity, colonial ties | ~1 MB |
| `geo_cepii.csv` | CEPII GeoDist | Country-level geographic metadata | ~50 KB |

### Data Access

The TDM trade data files (`_TDM_HS4_2015_2024.zip` and `TDM_report.csv`) are **not included in this repository** due to their size (1.5 GB and 9.2 GB respectively, well above GitHub's 100 MB file limit). They are stored in the shared Google Drive folder:

```
Google Drive > My Drive > Colab Notebooks > CSIS
```

To run the notebook, ensure all data files are placed in: `/MyDrive/Colab Notebooks/CSIS/`

The CEPII GeoDist files are downloaded automatically by the notebook at runtime from the CEPII website.

## Outputs

The pipeline produces the following CSV files in the `outputs/` subfolder:

- `prong1_tiered_reconciled_2023.csv` — Prong 1 reconciled values
- `prong2_gravity_reconciled_2023.csv` — Prong 2 reconciled values
- `prong_comparison_2023.csv` — Flow-level comparison of both prongs
- `gravity_coefficients_2023.csv` — Estimated gravity regression coefficients

## Reference

Gaulier, G. and Zignago, S. (2010), "BACI: International Trade Database at the Product-Level. The 1994-2007 Version", *CEPII Working Paper*, No. 2010-23.

## Version History

| Version | Date | Description |
|---------|------|-------------|
| v1.0 | April 2026 | Initial implementation — single year (2023), Prong 1 + Prong 2 |

## Planned Improvements (v2+)

- [ ] Extend to multi-year panel (2015–2024) with time fixed effects
- [ ] Add tariff and FTA variables to gravity specification
- [ ] Implement HS6-level disaggregation
- [ ] Automate COLUMN_MAP detection from TDM headers
- [ ] Add visualization dashboard for reconciliation diagnostics
- [ ] Benchmark coefficients against published BACI results

## Author

**Nina Pham** — CSIS

## License

This project is for internal research purposes at CSIS.
