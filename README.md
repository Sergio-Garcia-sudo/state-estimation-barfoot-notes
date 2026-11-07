# State Estimation for Robotics: study notes and numerical experiments

Self-study repository following Timothy Barfoot's *State Estimation for Robotics* (2nd ed.). The goal is to build the probabilistic foundations needed to read SLAM and visual-inertial odometry papers. Every formula is checked numerically: if the theory and a simulation disagree, there is a gap in my understanding.

## Contents

| Week | Topic | Book sections | Folder |
|------|-------|---------------|--------|
| 1 | Gaussian PDFs: sampling, linear and nonlinear transformations, product of Gaussians, Mahalanobis distance, Cramér-Rao bound | 2.2.6 to 2.2.14 | [`week-01-gaussians`](week-01-gaussians) |

Each week has a notebook with the experiments and an `answers.md` with written answers to the end-of-section questions.

## Week 1 at a glance

| Experiment | What is checked | Result |
|---|---|---|
| Sampling N(μ, Σ) | z = μ + V x with Σ = V Vᵀ (Cholesky) | sample mean and covariance match to < 0.004 |
| Linear map y = G x | μ_y = G μ_x, Σ_yy = G Σ_xx Gᵀ (G is 2×3) | match to < 0.02 |
| y = exp(x) | exact density vs. linearization | linearization is good for σ = 0.1, poor for σ = 1 (true mean ≈ 1.65 vs. linear 1.00) |
| Fusing two Gaussians | information form: Σ⁻¹ = Σ₁⁻¹ + Σ₂⁻¹ | matches a brute-force grid product; fused variance 0.444 vs. 0.5 for the best sensor |
| Mahalanobis distance | d² ~ χ²(N), mean N, variance 2N | rejected fraction at the 95 % gate: 5.02 %; inside the 1σ ellipse: 39.3 % (not 68 %) |
| Mis-tuned filters | scaling Σ by 0.5 and 2 | mean d² = 4.0 and 1.0 (2N and N/2); rejected 22.4 % and 0.25 % |
| Cramér-Rao bound | cov of the average of K samples = Σ/K | log-log slope −1.006; the median sits about 1.5 times above the bound (theory: π/2 ≈ 1.57 for large K) |

## Running the notebooks

```bash
pip install -r requirements.txt
jupyter notebook week-01-gaussians/week01_gaussians.ipynb
```

The notebook also runs in Google Colab without changes. All random numbers use a fixed seed (`np.random.default_rng(0)`), so the outputs are reproducible.

## Notes in my own words (Week 1)

1. **Where G Σ Gᵀ comes from and where it appears in a Kalman filter.** It comes from passing a Gaussian through a linear transformation: cov[y] = E[(Gx − Gμ)(Gx − Gμ)ᵀ] = G Σ Gᵀ. In a Kalman filter it appears when propagating the covariance of the state or of the measurements through a linear model.
2. **Why linearization fails with large uncertainty.** The linearization is a Taylor series centered at the expected value and is only valid close to it. With large uncertainty the values move away from the expected value and the approximation breaks down.
3. **Why fusing in information form is just adding.** In the normalized product of Gaussians the fused covariance is the inverse of the sum of the inverse covariances. Since information is the inverse covariance, the total information matrix is the sum of the individual ones.
4. **How to use d² to detect a badly tuned filter.** For a well-tuned filter, d² over many samples follows a chi-squared distribution. Comparing the fraction of samples above the theoretical 95 % threshold with 5 % reveals the problem: more rejections than expected means overconfident, fewer means underconfident.
5. **What it means that the average reaches the Cramér-Rao bound.** The average is unbiased and has the smallest variance any unbiased estimator of the mean can have, so it is an efficient estimator.

## Reference

T. D. Barfoot, *State Estimation for Robotics*, 2nd edition, Cambridge University Press, 2024. Chapter 2: Primer on Probability Theory.
