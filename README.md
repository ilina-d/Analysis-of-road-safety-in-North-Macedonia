# Analysis of Road Safety in North Macedonia
 
A detailed analysis of the factors contributing to road safety on Macedonian highways, based on data provided by the Faculty of Civil Engineering. The goal is to determine the safety rating of a given road section using physical, environmental, and traffic features, aligned with the IRAP star rating system.
 
[Dataset](#dataset) • [Target Variable](#target-variable) • [Analysis Steps](#analysis-steps) • [Clustering Experiments](#clustering-experiments) • [Usage](#usage)
 
---
 
## Dataset
 
The dataset covers **161 sections of Macedonian highways**, sourced from `avtopati_tabela.xlsx`. Each section has:
 
- **24 static or averaged features** — physical road characteristics (section length, altitude, speed limit, curve geometry, pavement quality, signage age, etc.) and environmental averages over a 10-year period (rainfall, snowfall, temperature).
- **10 years of annual traffic data (2014–2023)** — average annual daily traffic (`PGDS`) and a weighted accident index (`Wi`) per year.
### Feature Descriptions
 
| Feature | Description |
|---|---|
| `Last_Con` | Year of last pavement construction |
| `Len_Section` | Section length [m] |
| `Avg_Rain` | Average annual rainfall [mm/m²] |
| `Max_Snow` | Average maximum annual snowfall [mm/m²] |
| `Avg_Temp` / `Max_Temp` / `Min_Temp` | Average / max / min annual temperatures [°C] |
| `Avg_Altitude` | Average altitude [m] |
| `Avg_Speed_Limit` | Average speed limit [km/h] |
| `Curve_Char` | Curvature characteristic (sum of deflection angles / length) |
| `Avg_Rad` | Average horizontal radius of curvature |
| `Lat_Force` | Lateral force on vehicles in curves [N/kg] |
| `Avg_Incline` | Average longitudinal slope [%] |
| `Friction_Coef` | Friction coefficient |
| `Wheel_Ruts` | Wheel rut depth [mm] |
| `IRI` | International Roughness Index [m/km] |
| `PCI` | Pavement Condition Index [%] |
| `Counterslope_Coef` | Counter-slope coefficient |
| `Visibility_H` / `Visibility_V` | Horizontal / vertical visibility coefficients |
| `Sign_Age` | Age of vertical road signs [y] |
| `Sign_Renewal` | Annual renewal rate of horizontal road markings [times/y] |
| `Node_Intersection_Density` | Density of road nodes, intersections, and toll stations |
| `Bridge_Param` | Density of bridges and viaducts |
| `PGDS_20XX` | Average annual daily traffic for a given year |
| `Wi_20XX` | Weighted accident index for a given year: `Wi = (Nl×1 + Nt×10 + Nz×85) / section_length` |
 
---
 
## Target Variable
 
The safety of each section is quantified in two ways:
 
- **`Target_Wi`** — the mean `Wi` across all years the section existed (null years excluded), clipped to a maximum of 30.
- **`Target_Stars`** — an ordinal IRAP-aligned star rating (1–5) derived from `Target_Wi`:

| Stars | Wi Range |
|---|---|
| ⭐⭐⭐⭐⭐ (5) | 0 – 2 |
| ⭐⭐⭐⭐ (4) | 2 – 5 |
| ⭐⭐⭐ (3) | 5 – 10 |
| ⭐⭐ (2) | 10 – 20 |
| ⭐ (1) | 20 – 30 |
 
> A note on the data: years where a section did not yet exist were originally marked as `Wi = 0` in the source Excel file. These were corrected to `null` before computing the average, as averaging in zero values would artificially lower the safety score.
 
---
 
## Analysis Steps
 
### 1. Data Loading & Preprocessing
- Dropping identifier columns (`RB`, `R_NUMBER`, `S_NUMBER`, `S_NAME`) and converting columns to correct types
- Computing `Target_Wi` and `Target_Stars` using the corrected average (null-aware)
- Comparing the corrected vs. original (zero-inclusive) averages — class differences observed in only 7 rows
### 2. Dataset Enrichment (Explored & Discarded)
An expanded dataset was constructed by melting the 10 annual `PGDS`/`Wi` columns into individual rows (161 sections × 10 years = 1,610 rows), adding `Year` and a derived `Years_Since_Last_Con` feature. This was ultimately **discarded** because:
- PGDS and `Years_Since_Last_Con` showed no meaningful correlation with `Wi`
- Using yearly `Wi` as the target produced class distributions incompatible with the IRAP thresholds (which are calibrated to averaged Wi)
- 69% of yearly `Wi` values differed by at least 1 star from the section's average target, with 24% differing by 2 or more stars
### 3. Exploratory Data Analysis (EDA)
- **Distribution & outlier plots** — boxplots and histograms for all features; outliers in road geometry features (curvature, slope, visibility) preserved as informative extremes
- **Scatter matrix** — colored by `Target_Stars`; near-perfect linear correlations identified among `Avg_Rain`, `Avg_Snow`, `Avg_Temp`, `Min_Temp`, `Max_Temp`, and `Avg_Altitude`
- **Correlation heatmaps** — Pearson, Spearman, and Kendall coefficients; the 6 climate/altitude features confirmed redundant
- **Mutual Information scores** — computed for both `Target_Wi` (regression) and `Target_Stars` (classification); `Lat_Force`, `Visibility_H`, and `IRI` scored zero MI in both cases
### 4. Feature Engineering & Scaling
- Removed 5 redundant climate/altitude columns (`Avg_Rain`, `Avg_Temp`, `Max_Temp`, `Min_Temp`, `Avg_Altitude`) due to near-perfect multicollinearity
- Applied **Min-Max scaling** to preserve the effect of extreme values in geometric features
- Retained `Lat_Force`, `Visibility_H`, and `IRI` for now, flagged for ablation testing
---
 
## Clustering Experiments
 
Five unsupervised clustering algorithms were tested to evaluate whether the input features naturally form 5 groups matching the IRAP star categories — both on the full scaled feature space and after PCA reduction to 12 components (90% variance retained):
 
| Algorithm | Full Space | PCA-Reduced |
|---|---|---|
| K-Means | ✗ | ✗ |
| Agglomerative Clustering | ✗ | ✗ |
| Gaussian Mixture Model | ✗ | ✗ |
| Spectral Clustering | ✗ | ✗ |
| HDBSCAN | — | — |
 
None of the algorithms produced clusters that meaningfully align with the existing star rating categories. This suggests either that the decision boundary between safety classes is complex and non-convex, or that the features alone are insufficient to cleanly separate the categories without supervision.
 
> *Next planned step: supervised regression and decision tree models with cross-validation, addressing the class imbalance in `Target_Stars`.*
 
---
 
## Usage
 
### Requirements
 
```bash
pip install pandas numpy matplotlib seaborn scipy scikit-learn missingno plotly openpyxl jupyter
```
 
### Running the Notebook
 
```bash
# Clone the repository
git clone https://github.com/ilina-d/Analysis-of-road-safety-in-North-Macedonia.git
cd Analysis-of-road-safety-in-North-Macedonia
 
# Launch Jupyter
jupyter notebook
```
 
Then open `analiza_avtopati.ipynb`. The notebook requires `avtopati_tabela.xlsx` to be present in the same directory (not included in the repo).
