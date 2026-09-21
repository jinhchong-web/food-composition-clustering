# Unsupervised Learning Approaches to Food Composition

Clustering a European food-composition dataset by nutrient profile using six unsupervised learning algorithms, to explore how different foods group together based on nutrient content.

## Overview

This project preprocesses a large, multi-country food-composition dataset and applies six clustering algorithms to uncover nutrient-based food groupings:

- **K-Means** (elbow method to select k)
- **Agglomerative (Hierarchical) Clustering** (ward linkage)
- **DBSCAN** (density-based)
- **Gaussian Mixture Model**
- **Mean Shift**
- **Affinity Propagation**

Results are visualized via PCA (2D projection) and compared to understand how centroid-based, density-based, and exemplar-based methods behave differently on the same data.

## Repository Structure

```
food-composition-clustering/
├── README.md
├── LICENSE
├── .gitignore
├── requirements.txt
├── Data.ipynb                      # Data cleaning, feature engineering, outlier handling
├── Training.ipynb                  # Scaling, clustering, PCA visualization, evaluation
├── Food_composition_dataset.xlsx   # Raw dataset
└── preprocessed_data.csv           # Output of Data.ipynb, input to Training.ipynb
```

## Dataset

- **Source:** [Food composition database for nutrient intake: selected vitamins and minerals in selected European countries](https://data.niaid.nih.gov/resources?id=zenodo_438313) (NIAID Data Ecosystem / Zenodo)
- **Size:** 236,994 rows, 12 columns, covering Germany, France, the UK, the Netherlands, Italy, Sweden, and Finland
- Columns include `FOOD_ID`, `COUNTRY`, hierarchical food categories (`level1`–`level3`, `efsaprodcode2_recoded`), `NUTRIENT_ID`/`NUTRIENT_TEXT`, `UNIT`, and `LEVEL` (nutrient amount)
- **License:** [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) (European Food Safety Authority). Redistribution is permitted with attribution, so `Food_composition_dataset.xlsx` is included in this repo as-is.

## Methodology

### 1. Data Preprocessing (`Data.ipynb`)
- Dropped non-informative ID columns (`code`, `efsaprodcode2`)
- Split the comma-packed `efsaprodcode2_recoded` field into `foodLevel1`–`foodLevel4`
- Standardized `NUTRIENT_TEXT` (stripped parenthetical symbols, e.g. "Calcium (Ca)" → "Calcium")
- Normalized units (µg → mg) so all nutrient values share a common scale
- Pivoted the table so each nutrient becomes its own column per food item, taking the mean where duplicate food–nutrient pairs existed
- Label-encoded categorical columns for downstream clustering
- **Outlier handling:** Z-score and IQR methods were tested but performed poorly on highly skewed nutrients (notably potassium); capping/flooring (5th–95th percentile clipping) plus log transformation were used instead, since they adjust extreme values rather than discarding rows
- Exported the cleaned, pivoted dataset to `preprocessed_data.csv`

### 2. Clustering (`Training.ipynb`)
- Standardized features with `StandardScaler`
- **K-Means:** elbow method selected k = 10
- **Agglomerative Clustering:** euclidean distance, ward linkage, n_clusters = 10
- **DBSCAN:** eps = 0.8, min_samples = 5 → 44 clusters (plus outliers labeled -1)
- **Gaussian Mixture Model:** 10 components, spherical covariance
- **Mean Shift:** bandwidth estimated automatically, bin_seeding = True
- **Affinity Propagation:** damping = 0.9 → 102 clusters
- Evaluated with silhouette score, Calinski-Harabasz score, and Davies-Bouldin score, and visualized each result via PCA (2D projection)

## Results Summary

Algorithms with a predefined cluster count (K-Means, Agglomerative, GMM) produced comparatively stable, consistent clusters. Density- and exemplar-based methods (DBSCAN, Mean Shift, Affinity Propagation) were far more sensitive to local density and produced widely different cluster counts (44 and 102 respectively), reflecting the dataset's structural complexity.

## Limitations

- High dimensionality and skew in nutrient values made outlier handling difficult; Z-score and IQR methods underperformed on this data
- Density-based algorithms' cluster counts varied significantly, suggesting the feature space may need further refinement
- Results are unsupervised and exploratory — no ground-truth labels exist to validate cluster quality against

## Future Work

- Reorganize the dataset (e.g. a nutrient-centric index) to improve clustering stability
- Explore dimensionality reduction beyond PCA (e.g. UMAP) before clustering
- Compare additional distance metrics and cluster validation techniques

## Setup

```bash
pip install -r requirements.txt
```

Run `Data.ipynb` first to produce `preprocessed_data.csv`, then run `Training.ipynb` for clustering and evaluation. Both notebooks expect to be run from the repository root.

## License

- **Code** (notebooks, scripts) in this repository is licensed under the [MIT License](LICENSE).
- **Data** (`Food_composition_dataset.xlsx`, and `preprocessed_data.csv` as a derivative of it) is from the European Food Safety Authority's Food Composition Database and is licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). Attribution is required for any reuse — see Citation below.

## Citation

If you use this work, please attribute the original dataset:
> Food composition database for nutrient intake: selected vitamins and minerals in selected European countries. European Food Safety Authority (EFSA), via NIAID Data Ecosystem. https://data.niaid.nih.gov/resources?id=zenodo_438313. Licensed under CC BY 4.0 (https://creativecommons.org/licenses/by/4.0/).

No changes were made to the raw dataset values themselves prior to the preprocessing steps described in this README (see `Data.ipynb` for the transformations applied to produce `preprocessed_data.csv`).
