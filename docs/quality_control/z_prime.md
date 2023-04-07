# Z prime

## Standard Z`
[Z prime](https://en.wikipedia.org/wiki/Z-factor) is used as a quality control
metric to measure the separation between positive and negative controls.

$$
{\displaystyle {\text{Z`}}=1-{3(\sigma _{p}+\sigma _{n}) \over |\mu _{p}-\mu _{n}|}}
$$

Where $\mu_p$ and $\mu_n$ are the mean of the positive and negative controls, and
$\sigma_p$ and $\sigma_n$ are the standard deviations of the positive and negative
controls. A value greater than 0.5 is typically considered good.


To calculate the Z` between a list of positive control values and negative control values:
```python
import numpy as np

def z_prime(pos: list, neg: list) -> float:
    sigma_p, sigma_n = np.std(pos), np.std(neg)
    mu_p, mu_n = np.mean(pos), np.mean(neg)
    return 1 - (3 * (sigma_p + sigma_n) / np.abs(mu_p - mu_n))
```

## Robust Z`

The robust Z prime is the same calculation as before, but using the median
instead of mean, and median absolute deviation instead of standard deviation.

```python
import numpy as np
from scipy.stats import median_abs_deviation as mad

def robust_z_prime(pos: list, neg: list) -> float:
    sigma_p, sigma_n = mad(pos), mad(neg)
    mu_p, mu_n = np.median(pos), np.median(neg)
    return 1 - (3 * (sigma_p + sigma_n) / np.abs(mu_p - mu_n))
```
