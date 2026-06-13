# Causal Ecosystem Dynamics — Sardine / Anchovy / SST
### CCM · SHREC · SINDy pipeline applied to the California Current system

This repository reconstructs and extends the causal analysis of Sugihara et al. (2012)
using three complementary frameworks: Convergent Cross Mapping (CCM), Shared Recurrence
Embedding for Causality (SHREC), and Sparse Identification of Nonlinear Dynamics (SINDy).

---

## Data

| File | Description |
|------|-------------|
| `sardine_anchovy_sst.csv` | Annual landings and SST records, 1929–2006 (N = 78). Columns: `year`, `anchovy`, `sardine`, `sio_sst`, `np_sst`. Source: Sugihara et al. (2012). |
| `shrec_1var_latent_coordinates.csv` | SHREC latent coordinate ψ₁ from the 4-variable joint manifold. Written by `02_SHREC.ipynb`, consumed by `03` and `04`. |
| `shrec_pairwise_latent_coordinates.csv` | ψ₁ for all six pairwise variable combinations. Written by `02_SHREC.ipynb`. |

---

## Notebooks — run in this order

```
sardine_anchovy_sst.csv
        │
        ├─▶ 01_CCM_Sugihara_Replication.ipynb   (standalone)
        │
        └─▶ 02_SHREC.ipynb  ──▶ shrec_1var_latent_coordinates.csv
                                 shrec_pairwise_latent_coordinates.csv
                                        │
                                        ├─▶ 03_CCM_SHREC.ipynb
                                        └─▶ 04_SINDy.ipynb
```

---

### 01 — CCM Sugihara Replication
**`01_CCM_Sugihara_Replication.ipynb`**

Replicates the key figures of Sugihara et al. (2012) *Detecting Causality in Complex
Ecosystems* using Convergent Cross Mapping on the sardine / anchovy / SST dataset.

| Section | Content |
|---------|---------|
| Background | Theory of CCM and delay-embedding |
| Helper functions | `embed`, `simplex_predict`, `ccm_skill` |
| Figure 3A | CCM cross-map skill vs library length — anchovy ↔ SST |
| Figure 3B | Convergence curves with confidence intervals |
| Figure 3C & D | Pairwise CCM between all four variables |
| Figure E | Shadow manifold visualisation |
| Full CCM analysis | CCM applied to all variable pairs with significance testing |

**Inputs:** `sardine_anchovy_sst.csv`
**Outputs:** figures only

---

### 02 — SHREC
**`02_SHREC.ipynb`**

Extracts a shared latent coordinate ψ₁ from the joint recurrence structure of all four
ecosystem variables using the SHREC manifold learning pipeline, then decomposes ψ₁
spectrally to identify dominant periodicities.

| Section | Content |
|---------|---------|
| 1 — Data loading | CSV ingestion, z-score normalisation |
| 2 — Helper functions | `create_recurrence_matrix`, `shrec_reconstruct`, global parameters (E=3, k=10) |
| 3 — 4-variable SHREC | Joint manifold for Anchovy · Sardine · SST-SIO · SST-NP; ψ₁ trajectory + phase portrait; writes `shrec_1var_latent_coordinates.csv` (skipped if already present) |
| 4 — Pairwise SHREC | All six 2-variable pairs in a 2×3 panel; writes `shrec_pairwise_latent_coordinates.csv` |
| 5 — Spectral decomposition | FFT amplitude spectrum · Welch PSD · SSA (Singular Spectrum Analysis) · STL (Seasonal-Trend-Loess at PDO and ENSO bands) · STFT spectrogram |

**Inputs:** `sardine_anchovy_sst.csv`
**Outputs:** `shrec_1var_latent_coordinates.csv`, `shrec_pairwise_latent_coordinates.csv`

---

### 03 — CCM on SHREC Latent Coordinates
**`03_CCM_SHREC.ipynb`**

Tests whether the SHREC latent coordinate ψ₁ carries genuine causal signal by running
CCM between ψ₁ and each observed variable. Provides convergence analysis and asymmetry
tests to distinguish causation from correlation.

| Section | Content |
|---------|---------|
| Helper functions | `embed`, `simplex_predict`, `ccm_skill`, `safe_L_range`, convergence fitter |
| Load data | Ecological CSV + ψ₁ from SHREC CSV; alignment to common time window (1931–2006) |
| CCM: ψ vs fish | Cross-map skill curves — ψ₁ ↔ anchovy, ψ₁ ↔ sardine |
| Convergence analysis | Exponential fit ρ(L) = α·exp(−γL) + ρ∞; ρ∞ threshold test |
| CCM: ψ vs SSTs | Cross-map skill curves — ψ₁ ↔ SST-SIO, ψ₁ ↔ SST-NP |
| Convergence analysis | Same convergence diagnostics for SST pairs |

**Inputs:** `sardine_anchovy_sst.csv`, `shrec_1var_latent_coordinates.csv`
**Outputs:** figures only

---

### 04 — SINDy
**`04_SINDy.ipynb`**

Discovers sparse governing equations for the four-variable ecosystem using SINDy. Runs
four progressively more constrained stages, each with a baseline (no ψ₁) and an
augmented version (ψ₁ included as an exogenous driver informed by SHREC).

| Stage | Method | Key idea |
|-------|--------|----------|
| 1 — Percentile SINDy | Fixed percentile threshold on OLS coefficients | Fast sparsification; compares baseline vs SHREC-augmented |
| 2 — Lambda sweep | Scan threshold grid; keep terms stable across λ values (variance filter) | Robust feature selection |
| 3a — CCM-restricted (percentile) | Per-equation predictor sets enforced by CCM causal graph | Removes spurious cross-species terms |
| 3b — CCM-restricted (variance) | Same causal graph, variance-based selection | Sparser, more stable equations |
| 4 — Non-smooth + AIC | Non-smooth basis (ReLU, |x|, triangle waves, Δclimate); AIC λ-selection capped at 3 terms | Captures threshold and piecewise dynamics |

Each stage reports one-step-ahead R² and free-running simulation R² for all four
variables, with and without ψ₁.

**Inputs:** `sardine_anchovy_sst.csv`, `shrec_1var_latent_coordinates.csv`
**Outputs:** figures, printed equation tables, R² summary tables

---

## Requirements

```
numpy
pandas
scipy
matplotlib
seaborn
statsmodels
```

Install with:
```bash
pip install numpy pandas scipy matplotlib seaborn statsmodels
```

---

## References

Sugihara, G., May, R., Ye, H., Hsieh, C.-H., Deyle, E., Fogarty, M., & Munch, S. (2012).
Detecting causality in complex ecosystems. *Science*, 338(6106), 496–500.

Brunton, S. L., Proctor, J. L., & Kutz, J. N. (2016).
Discovering governing equations from data by sparse identification of nonlinear dynamical systems.
*PNAS*, 113(15), 3932–3937.
