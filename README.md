# East Java Food Commodity Price Wrangling & Disparity Analysis (2024)

[![Python](https://img.shields.io/badge/Python-3.10%2B-blue.svg?logo=python&logoColor=white)](https://www.python.org/)
[![Pandas](https://img.shields.io/badge/Pandas-2.0%2B-150458.svg?logo=pandas&logoColor=white)](https://pandas.pydata.org/)
[![Status](https://img.shields.io/badge/Status-Completed-success.svg)]()
[![Institution](https://img.shields.io/badge/UNESA-B.Sc.%20Data%20Science-blueviolet.svg)](https://datascience.fmipa.unesa.ac.id)
[![License: CC BY 4.0](https://img.shields.io/badge/License-CC_BY_4.0-lightgrey.svg)](https://creativecommons.org/licenses/by/4.0/)

An end-to-end **Data Wrangling & Analytics** pipeline integrating multi-source public datasets to monitor traditional market food prices across 9 regencies and cities in East Java throughout 2024. The project harmonizes daily food commodity prices with meteorological observations and official administrative/demographic data to deliver clean, analysis-ready daily and weekly datasets for regional food security monitoring.

This repository represents the official deliverable for the **Midterm Project (UTS Gasal 2026/2027)** in the **Data Wrangling** course, Undergraduate Data Science Program, Faculty of Mathematics and Natural Sciences, **Universitas Negeri Surabaya (UNESA)**.

---

## 1. Analytical Focus: Package D

The analysis investigates price hierarchy, quality premia, and temporal stability across two vital food commodity groups:
1. **Rice Quality Tiers**: Lower Quality I (*Beras Kualitas Bawah I*), Medium Quality I (*Beras Kualitas Medium I*), and Super Quality I (*Beras Kualitas Super I*).
2. **Cooking Oil Variants**: Bulk Unbranded (*Minyak Goreng Curah*) vs. Branded Packaged 1 (*Minyak Goreng Kemasan Bermerk 1*).

### Core Research Questions:
> 1. *What is the price spread between different quality tiers of rice and between bulk vs. branded packaged cooking oil across regencies and cities in East Java?*
> 2. *Do these price margins remain stable, or do they fluctuate dynamically between January and December 2024?*

---

## 2. Multi-Source Raw Data

All raw inputs are derived from authoritative, publicly accessible government and meteorological repositories without manual pre-cleaning:

| No | Source Entity | Raw File | Data Format & Characteristics |
|:--:|:---|:---|:---|
| 1 | **PIHPS Nasional**<br>*(Bank Indonesia)* | `pihps_pasar_tradisional_jatim_2024.xlsx` | 9-sheet Excel workbook (one sheet per region), wide-format matrix with 262 business-day columns, strings with comma separators (`"12,900"`), missing markers (`"-"`). |
| 2 | **NOAA GSOD**<br>*(NCEI, United States)* | `noaa_gsod_jatim_2024.csv` | Daily surface weather summaries from 4 stations in East Java, imperial units (°F, inches), sentinel missing codes (`999.9`, `99.99`), station duplicates in Surabaya. |
| 3 | **Kepmendagri No 300.2.2-2138 (2025)**<br>*(Ministry of Home Affairs)* | `kode_wilayah_jatim.csv`<br>`penduduk_jatim.csv`<br>`luas_jatim.csv` | Official hierarchy of 9,199 administrative codes (provinces down to villages), population headcounts, and geographic land area per regency/city. |

---

## 3. Repository Architecture

```text
jatim-food-price-wrangling/
│
├── UTS_DataWrangling_Paket_D.ipynb   # Master Jupyter Notebook (complete execution & outputs)
│
├── UTS_DW_data/                      # Original unedited raw data
│   ├── 1_harga_pasar_tradisional/   # Raw PIHPS daily market price workbook
│   ├── 2_cuaca_stasiun/             # Raw NOAA GSOD weather summaries
│   ├── 3_wilayah/                   # Raw Kepmendagri administrative CSV files
│   ├── PETUNJUK.md                  # Academic prompt & guidelines
│   └── SUMBER_DATA.md               # Raw data provenance documentation
│
├── data_processed/                   # Clean, integrated analytical datasets
│   ├── harga_pangan_jatim_2024.csv           # Daily integrated observation level (46,112 rows × 16 cols)
│   └── harga_pangan_jatim_mingguan_2024.csv  # Weekly ISO-calendar aggregated dataset (9,328 rows × 19 cols)
│
├── kamus_data.md                     # Comprehensive technical data dictionary
├── laporan.pdf                       # Official academic final project report (PDF)
├── .gitignore                        # Git ignore specifications
└── README.md                         # Project documentation (this file)
```

---

## 4. Data Wrangling Methodology (6-Stage Pipeline)

```
Discovery ──► Structuring ──► Cleaning ──► Enriching ──► Validating ──► Publishing
```

1. **Discovery**:
   - Inspected file encodings, schema dimensions, missing patterns, sentinel values (`999.9`, `99.99`, `"-"`), and regional naming discrepancies.
2. **Structuring**:
   - *Unpivoted* (`pd.melt`) the 262 wide-format daily date columns into long-format tidy rows.
   - Segregated aggregate category rows (`level == 1`) from specific sub-commodity variants (`level == 2`).
   - Parsed hierarchical administrative codes via regex to count subdistricts (`35.xx.xx`), urban wards/kelurahan (`35.xx.xx.1xxx`), and rural villages/desa (`35.xx.xx.2xxx`).
3. **Cleaning**:
   - Stripped comma thousand-separators and leading spaces from price text; casted to `float64`.
   - Converted imperial weather units to metric: Temperature to Celsius ($^\circ\text{C} = (\text{F}-32) \times 5/9$) and precipitation to millimeters ($\text{mm} = \text{inches} \times 25.4$).
   - Neutralized NOAA sentinel missing markers (`999.9` and `99.99`) to `np.nan`.
   - Imputed 101 intermittent missing daily prices using **chronological linear interpolation within `(region, commodity)` groups** followed by forward/backward fill at sequence edges.
4. **Enriching & Integration**:
   - Derived population density ($\text{people/km}^2 = \text{population} / \text{land area}$).
   - Averaged observations from the two overlapping Surabaya weather stations (Perak & Juanda) to prevent row multiplicity upon join.
   - Synchronized timeline to the standard ISO-8601 calendar week (`tahun_iso`, `minggu_ke`).
   - Executed **`LEFT JOIN`** using official Kepmendagri regional codes (`35.xx`) as the authoritative primary key.
5. **Validating**:
   - Asserted relational integrity: 100% of price records retained (0 rows dropped).
   - Conducted dual-method outlier detection via **Interquartile Range (IQR)** and **Modified Z-Score (MAD)**. Extreme values were flagged but preserved as genuine economic reflections of pre-harvest and Ramadan market spikes.
   - Audited four dimensions of data quality: *Completeness*, *Validity*, *Consistency*, and *Uniqueness*.
6. **Publishing**:
   - Exported clean analytical datasets to `data_processed/` in UTF-8 CSV format.
   - Authored the structured data dictionary (`kamus_data.md`).

---

## 5. Key Empirical Findings (Package D)

### A. Rice Quality Price Hierarchy & Margins
* The price structure strictly adheres to an expected positive tier hierarchy across all 9 regions: $\text{Lower I} < \text{Medium I} < \text{Super I}$.
* **Super I vs. Medium I Margin**: Widest in **Kota Madiun (+IDR 2,789.89/kg)** and **Kab. Banyuwangi (+IDR 2,185.02/kg)**; narrowest in **Kota Probolinggo (+IDR 469.47/kg)** and **Kota Malang (+IDR 608.59/kg)**.
* **Medium I vs. Lower I Margin** *(across the 6 reporting regions)*: Widest in **Kab. Jember (+IDR 1,637.50/kg)** and **Kab. Sumenep (+IDR 1,486.45/kg)**; narrowest in **Kota Probolinggo (+IDR 383.97/kg)** and **Kota Surabaya (+IDR 477.77/kg)**.
* *Data Limitation Note:* Lower Quality I Rice was structurally unmonitored by PIHPS in Kab. Banyuwangi, Kota Kediri, and Kota Madiun.

### B. Cooking Oil Price Hierarchy (Branded Packaged 1 vs. Bulk Curah)
* Branded Packaged 1 Cooking Oil commands a consistent packaging and brand premium over unbranded bulk oil across all territories.
* Premium margins are widest in rural/regency regions: **Kab. Banyuwangi (+IDR 2,964.22/liter)**, **Kab. Jember (+IDR 2,520.33/liter)**, and **Kab. Sumenep (+IDR 2,375.76/liter)**.
* Margins are tightest in urban distribution hubs: **Kota Madiun (+IDR 736.07/liter)** and **Kota Kediri (+IDR 895.42/liter)**.

### C. Temporal Stability and Market Dynamics Throughout 2024
* **Rice Market Dynamics**: Highly seasonal in agricultural production hubs (e.g., Kab. Sumenep monthly std = IDR 752.43; Kota Madiun std = IDR 714.16). Quality spreads contracted sharply during February–March 2024 (*paceklik* / pre-harvest season and the onset of Ramadan) due to steeper price surges in medium rice, widening back out following the main harvest. Conversely, metropolitan hubs (Kota Surabaya and Kota Malang) exhibited tightly regulated, stable rice prices year-round ($CV < 2.2\%$, spread std = IDR 169.62).
* **Cooking Oil Compression**: Experienced a **systematic and dramatic margin compression from January to December 2024** across 8 of 9 regions (e.g., in Surabaya, the packaged-to-bulk spread contracted from +IDR 4,210 to -IDR 490). This compression was driven by a relentless rise in bulk oil prices throughout 2024 ($CV = 6.13\% - 8.28\%$), whereas branded packaged prices remained anchored near government retail ceiling regulations (HET) ($CV = 0.63\% - 4.43\%$).

---

## 6. Tech Stack & Environment

Built with **Python 3.10+** utilizing core data science libraries:
* **`pandas`**: High-performance data structures, DataFrame transformations, unpivoting (`melt`), pivoting, and groupby aggregations.
* **`numpy`**: Vectorized numerical computations, sentinel handling, and log transformations.
* **`matplotlib`** & **`seaborn`**: Statistical visualizations, multi-panel line plots, and distribution KDE plots.
* **`openpyxl`**: Parsing multi-sheet Excel workbooks.
* **`pathlib`**: Cross-platform file path resolution.

### Quick Setup:
```bash
pip install pandas numpy matplotlib seaborn openpyxl jupyter
```

---

## 7. How to Reproduce

1. **Clone the repository**:
   ```bash
   git clone https://github.com/rava-andika/jatim-food-price-wrangling.git
   cd jatim-food-price-wrangling
   ```
2. **Launch Jupyter Notebook**:
   ```bash
   jupyter notebook UTS_DataWrangling_Paket_D.ipynb
   ```
3. **Execute the Pipeline**:
   Select `Kernel` $\rightarrow$ `Restart & Run All`. All stages from raw file ingestion to final analytical dataset export will run sequentially in approximately 10–15 seconds.
4. **Inspect Generated Datasets**:
   Processed files are automatically updated in [`data_processed/`](data_processed/).

---

## 8. Provenance, Licensing & Author

### Data Provenance
* **PIHPS Nasional (Bank Indonesia)**: Public traditional market daily price bulletins.
* **NOAA NCEI (Global Summary of the Day)**: Public domain U.S. government environmental data.
* **Kepmendagri No 300.2.2-2138 (2025)**: Public domain official statutory legal gazette of the Republic of Indonesia.

### Author & Academic Integrity
* **Author**: Rava Dwi Andika
* **Student ID (NIM)**: 25031554039
* **Program**: Undergraduate in Data Science (B.Sc.), Universitas Negeri Surabaya (UNESA)
* **Academic Year**: Gasal 2026/2027

*Code and documentation are released under the [Creative Commons Attribution 4.0 International (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/) license for academic and educational portfolio purposes.*
