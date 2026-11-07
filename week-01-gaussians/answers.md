# Week 1: written answers

Answers to the questions at the end of each section of `week01_gaussians.ipynb`. Sample values come from running that notebook (seed 0, M = 100 000 samples) and can vary slightly with the random stream.

## Questions 1: sampling (section 2.2.13)

**Change the correlation from 0.8 to −0.8 and to 0. What happens to the cloud and why?**

With a correlation of −0.8 the cloud is flipped: it now leans the other way (negative slope). A negative correlation means that when one variable decreases the other tends to increase. With a correlation of 0 the cloud becomes a circle: since there is no relationship between the variables (and both variances are 1), the points spread equally in every direction.

**What fraction of the samples should fall inside the 1-sigma ellipse in 2D?**

39.3 %. The 1-sigma ellipse is the set of points with Mahalanobis distance d² < 1, and d² follows a χ²(2) distribution, whose CDF at 1 is 1 − e^(−1/2) ≈ 0.393. It is not the 68 % of the 1D case (see section 5).

## Questions 2: linear transformation (section 2.2.6)

**Change G to an invertible 3×3 matrix. Compute det(G) and check that the volume of the ellipse changes by |det G|.**

For the invertible matrix used in the notebook, det(G) = 2 and √(det Σ_yy / det Σ_xx) = 2 as well, so both quantities agree: the volume of the ellipsoid is multiplied by |det G| = 2.

**Why can't you recover x from y when G is 2×3?**

y has only two components, so when trying to go backwards one of the components of x could take any value. G projects from 3 dimensions to 2 and collapses a whole direction, its null space, here spanned by (−1, 1, 2). All the x that differ along that direction give the same y, so y does not determine x.

## Questions 3: nonlinear transformation (section 2.2.7)

**With which sigma is the linearization acceptable and with which one is it not? How does this relate to the curvature of exp(x)?**

The linearization is acceptable for sigma = 0.1. For 0.5 and 1.0 it differs noticeably from the exact density. The linearization is a first-order Taylor expansion around the expected value of x, so as sigma grows and the samples move away from that value, the approximation gets worse. Since only the first derivative is used, the curvature of exp(x) is ignored: the linearization drops terms that add spread, which is why the real spread is larger. For example, with sigma = 0.5 the real standard deviation is about 0.60 (linearized: 0.5), and with sigma = 1 the gap is much larger: about 2.16 (linearized: 1.0).

**Compare the sample mean of y with g(μ_x) = 1. Why do they not match when sigma is large?**

For sigma = 1 the sample mean of y is about 1.65, not 1 (the exact value is e^(1/2) ≈ 1.6487). The linearization only keeps g(μ_x) = 1 and drops the second-order term, which contributes ≈ ½ g″ σ² = σ²/2. Since exp is convex, the real mean lies above g(μ_x), and the difference grows with σ².

## Questions 4: product of Gaussians (section 2.2.8)

**Why is the fused ellipse small along both axes, if each sensor is only good along one?**

Informations (1/variance) add axis by axis, so the fused variance is smaller than either sensor's variance along every axis. Along x: 0.25 + 2 = 2.25; along y: 2 + 0.25 = 2.25. The fused variance is 1/2.25 = 0.444 on both axes, smaller than the best single sensor (0.5): each sensor covers the axis where the other one is bad. The ratio of the ellipse areas is √(0.1975 / 2) = √0.0988 ≈ 0.314, so the fused ellipse covers only 31.4 % of the area of the S1 ellipse (it is 68.6 % smaller).

**Compare det(S) with det(S1) and det(S2). Relate it to the entropy (section 2.2.10).**

det(S1) = det(S2) = 2, while det(S) ≈ 0.1975. The entropy of a Gaussian is H = ½ ln((2πe)ᴺ det Σ), so it depends on ln det Σ: a smaller determinant means a smaller entropy. The change with respect to a single sensor is ΔH = ½ ln(0.1975 / 2) ≈ −1.16 nats.

## Questions 5: Mahalanobis distance and chi-squared (section 2.2.9)

**Overconfident filter (0.5·Σ instead of Σ). What happens to the mean of d2 and to the rejected fraction?**

d2 is multiplied by 2, so the theoretical mean is 2N = 4 and the sample mean was 4. The rejected fraction was 22 % (it should be 5 %): the filter believes it has half the uncertainty it really has, so it throws away good measurements.

**Underconfident filter (2·Σ). What conclusion do you draw for tuning a filter?**

d2 is divided by 2, so the theoretical mean is N/2 = 1 and the sample mean was 1. The rejected fraction was 0.25 %: the filter believes it has twice the uncertainty it really has, so it accepts almost everything, outliers included. To tune a filter, look for a mean of d2 close to N and, with a 95 % gate, about 5 % of rejected measurements. If the mean is above N and more than 5 % is rejected, the covariance is too small (overconfident) and must be increased; if the mean is below N and less than 5 % is rejected, the covariance is too large (underconfident) and must be reduced.

## Questions 6: Cramér-Rao lower bound (section 2.2.14)

**What is the slope of the line on a log-log scale and why?**

The slope is −1 (measured: −1.006). Since tr(cov(μ̂)) = tr(Σ)/K, taking logarithms gives log(tr) = log(tr Σ) − log K, a straight line with slope −1: multiplying K by 10 divides the variance by 10. The empirical points fall on the Cramér-Rao line.

**Replace the average by the median. Is it still on the bound? What does that say about efficient estimators?**

With the median the points stay above the bound, on a line almost parallel to the one of the average. Its variance also decreases as 1/K, but it is about π/2 ≈ 1.57 times larger than the bound (the measured ratio grows from 1 at K = 1 to about 1.5 for K ≥ 30). The Cramér-Rao bound is a limit for any unbiased estimator, and only an efficient estimator (such as the average for Gaussian data) reaches it. The median is valid and unbiased but not efficient: it needs about 57 % more data to match the precision of the average.

## Closing: Week 1 in my own words

1. **Where G Σ Gᵀ comes from and where it appears in a Kalman filter.** G Σ Gᵀ comes from passing a Gaussian through a linear transformation. Computing cov[y] gives E[(Gx − Gμ)(Gx − Gμ)ᵀ] = G Σ Gᵀ. In a Kalman filter it appears when obtaining the covariance after passing the state or the measurements through a linear transformation (for example, the prediction and measurement models).
2. **Why linearization fails with large uncertainty.** The linearization is a Taylor series centered at the expected value, and it is only valid close to that value. As the uncertainty grows, the values move away from the expected value and the approximation breaks down.
3. **Why fusing in information form is just adding.** In the normalized product of Gaussians, the fused covariance is the inverse of the sum of the inverses of the covariances. Since the information is the inverse of the covariance, fusing several Gaussians means that the total information matrix is the sum of the information matrices of each one.
4. **How to use d² to detect a badly tuned filter.** If many samples are taken and d² is computed for each one, the values must follow a chi-squared distribution. Computing the theoretical 95 % acceptance threshold and comparing it with the percentage of samples actually rejected shows whether the filter is badly tuned: if more measurements are rejected than theory says, the filter is overconfident; if fewer, it is underconfident.
5. **What it means that the average reaches the Cramér-Rao bound.** The average is an unbiased estimator and has the smallest possible variance: no unbiased estimator can estimate the expected value with less variance than the Cramér-Rao bound allows, and the average reaches exactly that value. That is what makes it an efficient estimator.
