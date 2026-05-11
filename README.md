# CoBET · dCoBET · wa-dCoBET

**Adaptive Multiscale Binary Expansion Tests for Independence**

[![Python](https://img.shields.io/badge/python-3.8%2B-blue)](https://www.python.org)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

A family of nonparametric independence tests built on dyadic binary expansion features and SNR-guided adaptive weighting. Benchmarked against HSIC and dCov across Clayton-copula simulation settings.

---

## Methods

| Method | Description |
|---|---|
| **CoBET** | Copula-based Binary Expansion Test — univariate / multivariate, plug-in variance |
| **dCoBET** | Multivariate extension with identity or spectral (J) weight matrices |
| **wa-dCoBET** | Weight-Adaptive dCoBET — 10-fold SNR blending per coordinate pair, BH-FDR heatmap |
| **HSIC** | Hilbert-Schmidt Independence Criterion baseline (`hyppo`) |
| **dCov** | Distance Covariance / dCor baseline (`hyppo`) |

---

## Installation

**Clone and install (editable mode — recommended for research)**
```bash
git clone https://github.com/yyang3388/cobet.git
cd cobet
pip install -e .
```

**Install directly from GitHub**
```bash
pip install git+https://github.com/yyang3388/cobet.git
```

**With baseline comparators (HSIC, dCov)**
```bash
pip install -e ".[baselines]"
# or
pip install "cobet[baselines] @ git+https://github.com/yyang3388/cobet.git"
```

---

## Quick Start

### Pairwise Z-heatmap (wa-dCoBET)

```python
from cobet import pairwise_heatmap_new_stat

out = pairwise_heatmap_new_stat(
    n=500, d_coords=10, theta=2, K=4,
    b=0.4, transform_key="logquad",
    q_fdr=0.05, seed_data=123,
)
# Returns dict with keys: Z, T, Var, p, sig_bh, w_id, w_J
```

### Power simulation (CoBET / dCoBET)

```python
from cobet import run_plugin_only

results = run_plugin_only(
    n=500, theta=2, K=4,
    b_config={
        "trigU":   [0.03, 0.05, 0.10],
        "logquad": [0.10, 0.20, 0.30],
    },
    d=5,
    weights_list=("identity", "J"),
    R_eval=500, alpha=0.05, seed=123,
)
```

### wa-dCoBET power grid

```python
from cobet import run_full_grid_and_export

b_config_by_n = {
    250:  {"trigU": [0.05, 0.08, 0.1], "logquad": [0.1, 0.3, 0.5]},
    500:  {"trigU": [0.03, 0.05, 0.10], "logquad": [0.1, 0.2, 0.3]},
    1000: {"trigU": [0.01, 0.03, 0.05], "logquad": [0.07, 0.15, 0.2]},
}

df_agg, df_base = run_full_grid_and_export(
    n_list=[250, 500, 1000],
    b_config_by_n=b_config_by_n,
    theta=2, K=4, d_coords=10,
    R_eval=500, out_path="wa_dcobet_results.xlsx",
)
```

### HSIC and dCov baselines

```python
from cobet import simulate_hsic, simulate_dcorr

hsic_rows = simulate_hsic(
    n=500, theta=2, D=10,
    b_config={"linear": [0.0, 0.05, 0.1, 0.2]},
    n_simulations=500,
)

dcov_rows = simulate_dcorr(
    n=500, theta=2, D=10,
    b_config={"trigU": [0.0, 0.05, 0.1]},
    n_simulations=500, reps=499,
)
```

---

## Package Structure

```
cobet/
├── cobet/
│   ├── __init__.py       ← public API
│   ├── utils.py          ← Clayton copula sampler + transform families
│   ├── cobet.py          ← CoBET & dCoBET core + simulation runner
│   ├── wa_dcobet.py      ← wa-dCoBET + pairwise heatmap + BH-FDR
│   └── baselines.py      ← HSIC & dCov wrappers
├── docs/
│   └── index.html        ← GitHub Pages website
├── examples/
│   └── demo.ipynb        ← Jupyter demo notebook
├── setup.py
├── requirements.txt
└── README.md
```

---

## Transform Families

| Key | Model |
|---|---|
| `trigU` | X = sin(Φ⁻¹(u)),  Y = cos(b·X + v) |
| `expquad` | X = exp(−Z²),  Y = exp(−b·(X−1)² + v) |
| `linear` | X = u,  Y = b·X + v |
| `logquad` | X = log1p(Z²)/(1+log1p(Z²)),  Y = cos(b·X+v)·exp(−b·(X−0.7)²) |

All use a d-dimensional Clayton copula for the latent dependence structure.

---

## Dependencies

- `numpy`, `scipy`, `pandas`, `matplotlib` — core (always required)
- `openpyxl` — Excel export
- `hyppo` — required only for `simulate_hsic` / `simulate_dcorr`

---



---

## License

MIT License — see [LICENSE](LICENSE).
