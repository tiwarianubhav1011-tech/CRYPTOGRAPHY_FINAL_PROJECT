# Differential Privacy for Privacy-Preserving Data Analysis

A cryptography course project on publishing useful statistics about people without revealing any one person's data. It covers the Laplace mechanism for differential privacy (DP), a cryptographically secure random number generator (CSPRNG) extension, and a machine learning experiment trained on privacy-protected data.

**Team:** Aayush (25BCS001), Anubhav (25BDA013)
**Course:** Foundation of Cryptography, IIIT Dharwad
**Submitted to:** Dr. Swagatika Sahoo

> Academic prototype. This is a classroom demonstration, not a production privacy system.

## Contents

- `CRYPTOGRAPHY_DIFFERENTIAL_PRIVACY_WITH_ML.ipynb`: the full notebook (code, outputs, and explanations)
- `Differential_Privacy_Updated.pptx`: final presentation
- `Differential_Privacy_Updated.pdf`: final presentation as PDF
- `README.md`: this file

## Problem

Exact statistics can leak information about individuals. In the notebook, a count that drops from 278 to 277 after one student leaves reveals that student's hidden pass/fail result, even though no name is published. Removing names is not enough either, because quasi-identifiers such as age, gender and postcode can be joined with outside data to re-identify people.

**Research question:** How can we release useful aggregate statistics while giving a mathematical bound on what the output reveals about any single person?

## Approach

1. **Neighbouring datasets:** two datasets are neighbours if they differ by one person (add/remove-one).
2. **Sensitivity:** for a count query, the global sensitivity Δf = 1. This was checked by brute force over 300 random datasets and then justified by proof.
3. **Laplace mechanism:** release `f(D) + Z`, where `Z ~ Laplace(0, b)` and `b = Δf / ε`. This gives pure ε-DP (δ = 0).
4. **Sampling:** Laplace noise is generated as the difference of two exponential variables with mean `b`, and validated against theory (mean ≈ 0, E|Z| = b, Var = 2b²).
5. **Utility measurement:** 1000 trials per ε, comparing observed error with the theoretical value `b = 1/ε`.
6. **Composition:** repeated queries leak more, so a privacy budget must be tracked.
7. **CSPRNG extension:** replace the NumPy PRNG with Python's `secrets.SystemRandom()` (OS entropy), and show how a predictable seed breaks privacy.
8. **ML extension:** train a Linear Regression model on noise-perturbed training data and measure the effect on test error.

## Key results

**Private count (true count 278, 1000 trials):**

| ε | Noise scale b | Observed MAE | Theory (b) |
|---|---|---|---|
| 0.1 | 10 | 10.53 | 10 |
| 0.5 | 2 | 2.01 | 2 |
| 1.0 | 1 | 1.03 | 1 |

Observed error matches the theory, so the mechanism is implemented with the correct scale. Halving ε roughly doubles the error.

**Composition:** 500 repeated queries at ε = 0.5 (total budget spent: 250) reduce the averaged error to 0.147. This is why a budget manager is needed.

**Seed disclosure attack:** with NumPy seed 2026, an attacker who knows the seed regenerates the noise and recovers the exact count (278.0). With `secrets.SystemRandom()`, the noise cannot be reproduced this way.

**CSPRNG validation (20,000 samples, b = 2):** PRNG and CSPRNG both match the Laplace statistics (theory: mean 0, variance 8, mean absolute value 2). CSPRNG changes the security of the noise, not its distribution or ε.

**ML experiment (predict exam score from study hours and attendance):**

| Training data | Test MAE | R² |
|---|---|---|
| Original (baseline) | 11.71 | 0.008 |
| Noisy, ε = 10 | 12.22 | −0.081 |
| Noisy, ε = 1 | 17.16 | −1.038 |
| Noisy, ε = 0.1 | 17.74 | −1.153 |

Stronger privacy makes predictions worse. With these settings, models trained on noisy data predict worse than the mean, so the approach is not useful for this task without a larger dataset or a different method.

## Repository structure

```
.
├── CRYPTOGRAPHY_DIFFERENTIAL_PRIVACY_WITH_ML.ipynb
├── Differential_Privacy_Updated.pptx
├── Differential_Privacy_Updated.pdf
└── README.md
```

## How to run

Requirements: Python 3.9+

```bash
pip install numpy pandas matplotlib scikit-learn jupyter
jupyter notebook CRYPTOGRAPHY_DIFFERENTIAL_PRIVACY_WITH_ML.ipynb
```

Run all cells in order. The random seed (2026) is fixed for the demo sections, so those results reproduce. The CSPRNG sections are intentionally not seeded, so their outputs change on each run.

## Limitations

- **Not production-ready.** There is no access control, encryption, secure logging or auditing.
- **Educational sampler.** The notebook samples Laplace noise with floating-point operations. Production systems need a vetted DP library with documented finite-precision guarantees.
- **Synthetic data only.** All experiments use 1000 fictional students. The privacy proof does not depend on the data, but the ML results describe this dataset only.
- **Basic composition only.** Tighter accounting (advanced composition, Rényi DP) would allow more queries for the same budget.
- **Population patterns are not hidden.** DP protects individuals, not statistical trends.

## References

1. C. Dwork and A. Roth, *The Algorithmic Foundations of Differential Privacy*, Foundations and Trends in Theoretical Computer Science, 9(3–4), 2014.
2. C. Dwork, F. McSherry, K. Nissim and A. Smith, *Calibrating Noise to Sensitivity in Private Data Analysis*, TCC 2006.
3. L. Sweeney, *Simple Demographics Often Identify People Uniquely*, Carnegie Mellon University, Data Privacy Working Paper 3, 2000.
4. J. Abowd, *The U.S. Census Bureau Adopts Differential Privacy*, KDD 2018.