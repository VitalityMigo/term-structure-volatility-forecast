# Interest Rate Volatility Term Structure Prediction
This project predicts the realized volatility term structure of U.S. Treasury rates across tenors (2Y, 5Y, 10Y, 30Y) at a twenty-day horizon. Using PCA to decompose the vol curve into three orthogonal factors, and Elastic Net regression to predict each factor independently, the model achieves an Information Coefficient of 0.649 on the dominant level factor and directional accuracy exceeding 70% across all three components.

<br>

<img width="1487" height="486" alt="roc" src="https://github.com/user-attachments/assets/1ac394ea-cb3f-4a6d-a506-8ceb1209c321" />

## Analysis Structure
The analysis is organized around eleven sections:
- **Data Collection:** daily retrieval of U.S. Treasury yields (DGS2, DGS5, DGS10, DGS30), Fed Funds rate, 5Y inflation breakeven, and Baa credit spread via FRED API; VIX, SKEW, and MOVE Index via yfinance.
- **Realized Volatility and PCA:** construction of four realized vol series (20-day rolling std of daily yield changes), PCA decomposition into three orthogonal factors (level 82%, slope 16%, curvature 2%), and external validation against the MOVE Index (r = 0.871).
- **Target Construction:** forward PCA score levels at t+20 as prediction targets, with zero overlap between successive windows to avoid autocorrelation bias.
- **Feature Engineering:** 41 variables across seven economic groups — yield curve structure, realized vol, market sentiment, macro indicators, curve dynamics, mean-reversion z-scores, and HAR-structured persistence lags (Corsi, 2009).
- **Data Preprocessing:** chronological 80/20 train/test split (2006–2021 train, 2022–2026 test), StandardScaler re-fitted per fold in walk-forward to prevent data leakage.
- **Elastic Net:** penalized OLS combining L1 and L2 penalties, with joint cross-validation of α and ρ using 5-fold TimeSeriesSplit. Selected ρ is reported as a diagnostic of feature collinearity.
- **Gradient Boosting:** two-pass non-linear benchmark — MDI-based feature selection followed by grid-search tuning on selected subset.
- **Model Evaluation:** RMSE vs. persistence baseline (skill score), Information Coefficient (Spearman rank correlation), directional accuracy, and ROC-AUC.
- **Feature Analysis:** non-zero Elastic Net coefficients by component with economic interpretation.
- **Walk-Forward Validation:** ten-fold expanding window validation with scaler re-fitted per fold.
- **Vol Curve Reconstruction:** inverse PCA transform mapping predicted scores back to realized vol by tenor (2Y, 5Y, 10Y, 30Y).

## Results
| Metric | PC1 (level) | PC2 (slope) | PC3 (curvature) |
|---|---|---|---|
| Elastic Net RMSE | 1.399 | 0.662 | 0.247 |
| Persistence RMSE | 1.720 | 0.788 | 0.315 |
| Skill Score | 0.187 | 0.160 | 0.217 |
| IC (Spearman) | 0.649 | 0.231 | 0.122 |
| Directional Accuracy | 72.5% | 70.9% | 68.7% |
| ROC-AUC | 0.784 | 0.782 | 0.777 |
| Walk-forward RMSE | 1.852 | 0.663 | 0.273 |

Elastic Net is selected on all three components. Gradient Boosting inverts rankings on PC2 (IC = −0.207) regardless of regularisation, confirming that the signal is predominantly linear. The single retained feature for PC1 is the MOVE Index — the dominant predictor of next month's realized vol level is the current implied vol environment.

## Documents
- [`notebook.ipynb`](notebook.ipynb): Jupyter notebook containing the full analysis, from data collection to vol curve reconstruction.
- [`rate_vol.pdf`](rate_vol.pdf): study summarising the methodology, results, and economic interpretation.
- [`data/`](data/): cached CSV files for FRED series and Yahoo Finance data, generated on first run.
