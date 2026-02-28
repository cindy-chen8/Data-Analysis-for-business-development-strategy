# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Kpay case study: analyzing a raw dataset of 200,000+ merchants scraped from Google Maps in a major Australian city to develop actionable business development strategy for Kpay's BD Team.

## Environment Setup

- Python 3.11 via conda (`environment.yml`) or local venv (`kpay_env/`)
- Key dependencies: pandas, numpy, matplotlib, seaborn, scikit-learn, openpyxl, jupyter
- Setup: `conda env create -f environment.yml` then `conda activate kpay_case_study`
- Or use existing venv: `source kpay_env/bin/activate`

**Known issue:** The `kpay_env` venv has a broken numpy installation (RecursionError on import). Use the conda environment instead.

## Data

- Raw: `data/Case Study - Case Study.csv` (~30MB, 200k+ rows)
- Cleaned: `data/kpay_cleaned.csv` (~43MB)
- Key columns: `business_name`, `lead_key`, `state`, `suburb`, `sector_level_1`, `sector_level_2`, `sector_level_3`

## Notebooks

- `Part_1_data_clearning_enrichment.ipynb` — Data cleaning and enrichment pipeline (loads CSV with `dtype=str` to preserve phone formatting)
- `analysis.ipynb` — EDA: geographic distribution, sector analysis, data quality checks (gitignored, scratch/exploratory)

## Conventions

- Load raw CSV with `dtype=str` to preserve phone number formatting
- Use `seaborn` whitegrid style with `figsize=(12, 6)` default for plots
