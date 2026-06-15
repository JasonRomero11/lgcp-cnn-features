# LGCP Parameter Recovery via CNN with Intensity Features

## Abstract

The likelihood of a Log-Gaussian Cox Process (LGCP) has no closed form, as its evaluation requires marginalizing over a latent Gaussian field. Recent work replaces the likelihood with a convolutional neural network (CNN) trained on simulations, which predicts parameters from $\hat{L}(r)-r$ and point count $N$. Previously, this approach was tested only on square windows, and variance $\sigma^2$ and spatial range $s$ remain difficult to identify in practice. 

This work augments the CNN with eight scalar descriptors of the smoothed intensity field: variance, index of dispersion, and range ratio computed on quadrat counts, together with variance, skewness, kurtosis, Shannon entropy, and coefficient of variation from kernel density estimation. The method is tested on a simulation study over Colombia's continental geometry and applied to the 2020 Colombian seismic catalogue. The descriptors improve $R^2$ for $s$ from 0.59 to 0.80 (+34.4%) and for $\sigma^2$ from 0.54 to 0.67 (+23.4%), without affecting the estimation of $\mu$ ($R^2 \approx 0.90$).

> **Status:** The associated manuscript has been submitted to the journal *Computers & Geosciences* for review.

---

## Project Structure

```
.
├── scripts/
│   ├── simulations_rGLCP.R          # Generate LGCP simulations with feature extraction
│   └── CNN_train_and_predict.R      # Train CNN models and evaluate on real data
├── data/
│   ├── shapeZona_sp.rds             # Colombia boundary shapefile
│   └── sismos_sf_2020.rds           # 2020 Colombian seismic catalogue
├── Results_simulation/
│   ├── TRAIN/                       # Training data (simulated LGCP)
│   └── TEST/                        # Test data (simulated LGCP)
├── figures/                         # Generated plots and LaTeX tables
└── paper.tex                        # Main manuscript (LaTeX)
```

## Features Extracted

### Quadrat-based (3 features)
- **quad_var**: Variance of counts in quadrats
- **quad_VMR**: Variance-to-Mean Ratio (Index of Dispersion)
- **quad_range_ratio**: Ratio of max/min quadrat counts

### Kernel Density (5 features)
- **kde_var**: Variance of smoothed intensity surface
- **kde_skew**: Skewness of density values
- **kde_kurt**: Excess kurtosis of density values
- **kde_entropy**: Normalized Shannon entropy of density
- **kde_cv**: Coefficient of variation of density

## Quick Start

### Requirements
- R ≥ 4.0
- Python 3.8+ (for TensorFlow/Keras integration)

### R Packages
```r
install.packages(c("spatstat", "tidyverse", "pbmcapply", "sf", "patchwork"))
# TensorFlow for R:
reticulate::install_python()
keras::install_keras()
```

### Step 1: Generate Training and Test Data
```r
source("scripts/simulations_rGLCP.R")
# Generates 15,000 training + 1,000 test simulations
# Saves .rds files with LGCP parameters and computed features
```

### Step 2: Train CNN Models and Evaluate
```r
source("scripts/CNN_train_and_predict.R")
# Trains two models:
#   - M1: Baseline CNN (L(r) + N)
#   - M2: CNN + 8 intensity features
# Generates metrics table and diagnostic plots
```

## Model Architecture

**Baseline CNN (M1)**
- Input: L(r) curve (128 values) + point count N
- 3 convolutional layers (64 filters, kernel=7) with batch norm and max pooling
- 2 dense layers (64, 32 units) + output layer
- Output: 3 parameters (μ, σ², s)

**Proposed CNN (M2)**
- Input: L(r) + N + 8 intensity features
- Shared convolutional branch
- Auxiliary feature branch (32→16 units with batch norm)
- Concatenated + 2 dense layers + output

## Results (Test Set)

| Parameter | Model | R² | RMSE | MAE |
|-----------|-------|-----|------|-----|
| μ | Baseline | 0.907 | 0.095 | 0.068 |
| μ | CNN+Features | 0.910 | 0.093 | 0.067 |
| σ² | Baseline | 0.540 | 0.381 | 0.289 |
| σ² | CNN+Features | **0.673** | 0.286 | 0.214 |
| s | Baseline | 0.589 | 41542 | 29876 |
| s | CNN+Features | **0.804** | 25894 | 17312 |

## Output Files

After running the scripts:

**Data**: `Results_simulation/TRAIN/` and `Results_simulation/TEST/`
- `Data_LGCP_train_*.rds`: Training chunks with simulations
- `Data_LGCP_test_*.rds`: Test chunks with simulations

**Figures**: `figures/`
- `scatter_baseline.pdf`: Predicted vs true parameters (M1)
- `scatter_proposed.pdf`: Predicted vs true parameters (M2)
- `loss_curves.pdf`: Training history
- `r2_comparison.pdf`: R² performance comparison
- `envelope_fitted_model.pdf`: L(r) envelope validation
- `hist_N_fitted_model.pdf`: Point count distribution under fitted model
- `metrics_table.tex`: LaTeX table for manuscript




## License

[Specify your license, e.g., MIT, GPL-3.0]

## Contact

Jason Romero (jamromeror@udistrital.edu.co)
