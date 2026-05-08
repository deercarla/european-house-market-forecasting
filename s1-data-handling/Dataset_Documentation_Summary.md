---
title: "Dataset Documentation"
subtitle: "European Housing Price Forecasting | Machine Learning & Deep Learning, CBS Spring 2026"

output:
  pdf_document:
    latex_engine: xelatex
    highlight: tango
    toc: false
    number_sections: false

geometry:
  - a4paper
  - top=20mm
  - left=15mm
  - right=15mm
  - bottom=20mm

header-includes:
  - \usepackage{titlesec}
  - \usepackage{titling}
  - \usepackage{xcolor}
---

# Dataset Documentation — Summary

**European Housing Price Forecasting \| Machine Learning & Deep Learning, CBS Spring 2026**

------------------------------------------------------------------------

## Project Overview

Country-quarter panel covering up to EU-27 members, spanning approximately 2005–2025 (\~1,700 rows). Goal: forecast next-quarter residential house price growth (QoQ %) and compare models trained on data-driven vs. literature-based country clusters.

------------------------------------------------------------------------

## Datasets at a Glance

| \# | Dataset | Source | Raw Freq. | Role |
|---------------|---------------|---------------|---------------|---------------|
| 1 | House Price Index (HPI) | Eurostat `prc_hpi_q` | Quarterly | Target variable + autoregressive feature |
| 2 | HICP Inflation | Eurostat `prc_hicp_midx` | Monthly | Demand-side cost pressure |
| 3 | GDP Index | Eurostat `namq_10_gdp` | Quarterly | Economic activity |
| 4 | Unemployment Rate | Eurostat `une_rt_q` | Quarterly | Labour market conditions |
| 5 | Building Permits Index | Eurostat `sts_cobp_q` | Quarterly | Housing supply proxy |
| 6 | Old-age Dependency Ratio | Eurostat `demo_pjanind` | Annual | Structural demographics (clustering) |
| 7 | Population Density | Eurostat `demo_r_d3dens` | Annual | Urbanisation / land scarcity (clustering) |
| 8 | MIR Mortgage Rates | ECB API | Monthly | Cost of borrowing — eurozone only |
| 9 | Long-term Interest Rates | Eurostat `irt_lt_mcby_q` | Quarterly | Sovereign yields — all EU-27 |
| 10 | Manifesto Project (MARPOR) | MARPOR `mpd_data.csv` (2025a) | Per election | Parliament ideology score |

------------------------------------------------------------------------

## Variable Notes by Group

### 1. Housing Market — Target & Momentum

**House Price Index (HPI)** — Nominal index (2015 = 100). Used in two forms: QoQ % change as a momentum feature (`hpi_qoq_pct`), and shifted one quarter forward as the prediction target (`target_hpi_next_q`). QoQ growth is preferred over the index level for stationarity. The index clearly captures the 2008–09 crash, the post-2015 recovery, the COVID-era boom, and the 2022–23 correction driven by ECB rate hikes.

### 2. Macroeconomic Demand

**Harmonized Consumer Price Index (HICP)** — Quarterly mean of monthly observations. Affects housing through multiple channels simultaneously: real income erosion, construction cost inflation, and ECB tightening signals. The direction of its net effect on prices is empirically ambiguous — high inflation can suppress demand via affordability, or support prices via the property-as-hedge channel, depending on which dominates.

**Gross Domestic Product (GDP)** — Chain-linked volume index (2005 = 100), seasonally and calendar adjusted. QoQ growth measures short-term economic momentum. The 2008–09 and 2020 crashes are visible as large negative observations, allowing the model to learn how housing markets respond to economic shocks.

**Unemployment** — ILO definition, seasonally adjusted, ages 15–74. Both the level and QoQ change are included: a country with 8% unemployment but falling is structurally different from one at 8% and rising. Unemployment lags GDP (firms are slow to hire and fire), so it adds information about household-level financial stress beyond what GDP alone captures.

### 3. Supply-Side Constraints

**Building Permits** — Index (2021 = 100), residential dwellings only. A leading supply indicator: permits precede actual completions by 6–18 months, so current permit volumes partly predict supply conditions two to four quarters ahead. The lag structure in the model is designed to exploit this.

### 4. Financing Costs

**MIR Mortgage Rates** — Actual annualised agreed rate on new euro-denominated mortgage loans, not the ECB policy rate (transmission is imperfect and varies by country and competitive conditions). Included at level and two lags, as rate changes typically take 2–4 quarters to fully affect transaction volumes and prices. The 2022–23 episode — rates rising from below 1.5% to above 4% — is a key training signal. **Structurally absent (NaN) for 7 non-eurozone countries**: DK, SE, PL, CZ, HU, BG, RO.

**Long-term Government Bond Yields** — 10-year government bond yields (EMU convergence criterion). Captures sovereign risk, capital market conditions, and the benchmark for fixed-rate mortgage pricing and property valuations. Crucially provides an interest rate signal for all EU-27, covering the non-eurozone countries missing from MIR. The two variables are complementary rather than redundant.

### 5. Structural, Demographic, and Political Features

**Old-age Dependency Ratio** — People aged 65+ per 100 working-age people (20–64). Annual, carried forward. Used primarily as a clustering feature to capture long-run structural differences in housing demand trajectories and fiscal pressure — Eastern European markets (ageing, often shrinking populations) behave very differently from demographically younger ones.

**Population Density** — Inhabitants per km². Annual, carried forward. Proxies for land scarcity and urbanisation intensity: dense, supply-constrained markets (e.g. Netherlands) have structurally different price dynamics from sparse ones (e.g. Finland). Primarily a clustering feature.

**Parliament RILE (MARPOR)** — Seat-weighted average of party right-left scores across the full legislature, carried forward between elections. Proxies for the general direction of housing policy (rent regulation, zoning, owner-occupation incentives). Included at level and two lags, since policy effects typically materialise with a delay. **Key limitation:** the score covers all parliamentary parties, not the governing coalition specifically, as government membership data is not included in the standard MARPOR release. The full-parliament score is a reasonable but acknowledged proxy.

------------------------------------------------------------------------

## Panel Construction

**Frequency Harmonisation:** - Monthly series (HICP, MIR) → quarterly arithmetic mean of the three months within each quarter - Annual series (demographics) → value repeated across all four quarters; appropriate given that these variables genuinely change slowly - Election data (MARPOR) → last-observation-carried-forward until the next election

**Missing Data:**

| Variable | Gap | Treatment |
|--------------------------|------------------|-----------------------------|
| `mortgage_rate_pct` | Structurally absent for 7 non-eurozone countries throughout | Retained as NaN — informative alongside the `eurozone_member_timevarying` flag |
| `lt_interest_rate` | \~44 values, primarily Malta and Cyprus in early quarters | Forward-filled |
| `parliament_rile` | NaN before first MARPOR-recorded election per country | Good coverage from 2005 onwards |
| Demographics | Sparse early-year gaps in some countries | Forward-filled |

**Final Panel:** \~1,700 rows after dropping observations with missing HPI growth rates or the target. 26 countries × 65–80 quarters. The two-dimensional variation — cross-sectional (across countries) and temporal (within countries over time) — is the core structure ML panel models are designed to exploit.
