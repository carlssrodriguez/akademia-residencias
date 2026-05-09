# Senior Housing Location Algorithm — Spain

> Data Science project developed for the **Akademia Future Builders** programme by **Fundación Innovación Bankinter**.  
> Selection of 100 optimal locations across Spain for senior care facilities, using a weighted territorial attractiveness index built on public data from INE, IMSERSO, Social Security and the Ministry of Transport.

---

## Overview

Acting as the board of directors of a senior care chain, the challenge was to propose 100 candidate locations across Spain with a rigorous, data-driven justification. The business constraints: €2,100/month per resident, 85% occupancy target, and zero-tolerance for payment defaults.

The result is a full Data Science pipeline that processes 106MB of raw census data across 36,334 census sections, crosses eight public data sources, engineers 31 business variables, builds a custom F-of-M (Figure of Merit) index and selects the top 100 locations with guaranteed geographic diversification.

---

## Tech stack

![Python](https://img.shields.io/badge/Python-3.11-blue?logo=python)
![pandas](https://img.shields.io/badge/pandas-2.1-lightblue?logo=pandas)
![scikit-learn](https://img.shields.io/badge/scikit--learn-1.3-orange?logo=scikit-learn)
![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-orange?logo=jupyter)
![folium](https://img.shields.io/badge/folium-maps-green)

```
Python 3.11 · pandas · numpy · scikit-learn · folium · matplotlib · seaborn · openpyxl
```

---

## Project structure

```
akademia-residencias/
│
├── notebooks/
│   ├── M1_pipeline_datos.ipynb          # Data engineering: raw data ingestion and cleaning
│   ├── M2a_demanda_y_soledad.ipynb      # Feature engineering: demand and loneliness variables
│   ├── M2b_variables_economicas.ipynb   # Feature engineering: pension, house price, substitution rate
│   ├── M3_fofm.ipynb                    # F-of-M index: scoring and top 100 selection
│   ├── M4_clustering.ipynb              # K-Means: strategic profile segmentation
│   └── M5_visualizaciones.ipynb         # Visualisations: interactive map and charts
│
├── data/
│   ├── dataset_maestro.csv              # Master dataset: 36,334 sections × 31 variables
│   ├── localizacion_limpia.csv          # GPS coordinates per census section
│   ├── municipios_limpios.csv           # 8,124 Spanish municipalities with codes and names
│   └── renta_limpia.csv                 # Household income per municipality (INE 2024)
│
├── outputs/
│   ├── top100_ubicaciones.csv           # Final 100 locations with all metrics
│   ├── top100_con_clusters.csv          # Top 100 with strategic profile assigned
│   ├── ranking_municipios.csv           # Full ranking of 2,825 candidate municipalities
│   ├── resumen_provincias.csv           # Aggregated results by province
│   ├── resultados_akademia.xlsx         # Excel with all result tabs
│   ├── mapa_top100.html                 # Interactive map — open in browser
│   ├── grafico_perfiles.png             # Profile distribution chart
│   ├── grafico_top20.png                # Top 20 visual ranking
│   └── grafico_solvencia_mercado.png    # Solvency vs market size scatter
│
├── docs/
│   └── CasoPractico-2025.pdf            # Original case brief
│
├── .gitignore
└── README.md
```

---

## Methodology

### 1. Data engineering — M1

The INE census pyramid file is 106MB compressed with a 1.3GB internal XML. Loading it entirely into memory exhausts available RAM. The pipeline uses an incremental XML parser with `xml.etree.ElementTree`, processing and releasing each row, staying well under 500MB peak usage.

### 2. Feature engineering — M2

31 variables engineered across 36,334 census sections:

| Component | Variables |
|-----------|-----------|
| **Solvency** | Household income (INE 2024), average pension per municipality (Social Security Jan 2025), house price per m² (Ministry of Transport Q4 2025) |
| **Demand** | Population 75+, % population 75+, population 80+, future cohort projected to 2029 |
| **Loneliness** | Widowhood ratio (women 80+ / men 80+), single-person households projection 2029 |
| **Competition** | Residential coverage index (IMSERSO), hospital presence |
| **Future** | Projected cohort 2029 (+6.2% cumulative growth rate), substitution rate |

### 3. F-of-M index — M3

Custom weighted territorial attractiveness index with min-max normalisation:

```
F-of-M = 0.30 · Solvency + 0.25 · Demand + 0.25 · Loneliness + 0.15 · Competition + 0.05 · Future
```

- **Solvency** breaks down as: household income 15% + pension 10% + house price 5%
- Hard solvency filter: sections with household income below €25,200/year receive score 0 regardless of other variables
- Minimum market size: 500+ people aged 75+ per municipality
- Geographic diversification: maximum 3 municipalities per province

### 4. K-Means clustering — M4

Segmentation of the 100 locations into 4 strategic profiles using K-Means with StandardScaler. K=4 selected by silhouette score.

| Profile | Municipalities | Avg. income | Avg. 75+ pop. | F-of-M |
|---------|---------------|-------------|---------------|--------|
| Premium — maximum solvency | 4 | €66,070 | 4,997 | 0.509 |
| Major city — mass market | 4 | €32,320 | 103,064 | 0.349 |
| Mid-size city — relevant market | 37 | €30,502 | 19,065 | 0.283 |
| Ageing zone — adjusted solvency | 55 | €29,910 | 5,158 | 0.273 |

### 5. Visualisation — M5

Interactive map built with folium using GPS coordinates per census section. Charts generated with matplotlib and seaborn. Results exported to Excel for non-technical stakeholders.

---

## Results

- **36,334 census sections** evaluated across Spain
- **11,896 sections discarded** for failing the solvency filter
- **2,825 candidate municipalities** identified
- **100 final locations** across **46 provinces**

**Top 5 locations:**

| # | Municipality | Income | Avg. pension | House price | 75+ pop. | F-of-M |
|---|-------------|--------|--------------|-------------|----------|--------|
| 1 | Pozuelo de Alarcón (Madrid) | €83,812 | €1,853/mo | €5,102/m² | 7,192 | 0.553 |
| 2 | Majadahonda (Madrid) | €60,937 | €1,884/mo | €4,619/m² | 5,384 | 0.502 |
| 3 | Torrelodones (Madrid) | €61,366 | €1,824/mo | €4,455/m² | 1,446 | 0.497 |
| 4 | Sant Cugat del Vallès (Barcelona) | €58,164 | €1,712/mo | €4,856/m² | 5,967 | 0.485 |
| 5 | Barcelona | €37,371 | €1,526/mo | €4,501/m² | 186,908 | 0.438 |

---

## Getting started

```bash
# 1. Clone the repository
git clone https://github.com/carlssrodriguez/akademia-residencias.git
cd akademia-residencias

# 2. Create virtual environment
python -m venv .venv
.venv\Scripts\activate       # Windows
source .venv/bin/activate    # Mac/Linux

# 3. Install dependencies
pip install pandas numpy scikit-learn folium matplotlib seaborn openpyxl notebook ipykernel odfpy

# 4. Run notebooks in order
# M1 → M2a → M2b → M3 → M4 → M5
```

> **Note**: Raw ODS source files from INE are not included in the repository due to size. They must be downloaded separately and placed in the project root before running M1. `piramide_limpia.csv` is also excluded for the same reason — it is generated by M1.

---

## Data sources

Eight files from five public organisations. Raw files are not versioned in this repository.

### INE — National Statistics Institute

| Dataset | Description |
|---------|-------------|
| Municipal census — population pyramid | Population by sex and 5-year age groups for all 36,334 census sections. Base of all demand variables. |
| Household income 2024 | Net mean income per person and household for 6,811 municipalities. Source of the Solvency component. |
| Geographic coordinates per census section | Latitude and longitude of each census section centroid. Used for map visualisation. |
| Single-person household projection 2024-2039 | Households of one person aged 70+ by province until 2039. Reinforces the Loneliness component. |
| Elderly population projection 2024-2039 | Annual growth rate of population aged 70+ at national level. Used to project future cohort to 2029. |

### Ministry of Health

| Dataset | Description |
|---------|-------------|
| National Hospital Catalogue 2025 | 848 hospital centres with municipality and type. Source of `tiene_hospital` and `hosp_en_provincia`. |

### IMSERSO

| Dataset | Description |
|---------|-------------|
| Residential care supply by Autonomous Community | Care home places and coverage index by region. Source of the Competition component. |

### Social Security

| Dataset | Description |
|---------|-------------|
| Contributory pensions by municipality — January 2025 | Mean monthly pension and number of pensioners for 8,136 municipalities. Source of `pension_media`. |

### Ministry of Transport

| Dataset | Description |
|---------|-------------|
| Assessed housing value by municipality — Q4 2025 | Mean price per m² for the 306 municipalities with 25,000+ inhabitants, with provincial fallback for smaller ones. Source of `precio_m2_final`. |

---

*Built with Python · Public data from INE, IMSERSO, Social Security and Ministry of Transport · April 2026*
