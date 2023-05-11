# Strictly Standardised Mean Difference (SSMD)

SSMD is a measure of effects size and can be used for both quality control like
Z-prime, as well as hit-selection. It was initially prosed by [Zhang](https://www.sciencedirect.com/science/article/pii/S0888754307000079?via%3Dihub).


$$
\beta = \frac{\mu_1 - \mu_2}{\sqrt{\sigma_1^2 + \sigma_2^2 - 2\sigma_{12}}}
$$


```python
def ssmd(x: np.ndarray, y: np.ndarray) -> float:
    mu_x = np.mean(x)
    mu_y = np.mean(y)
    sigma_x = np.var(x)
    sigma_y = np.var(y)
    cov_xy = np.cov(x, y)[0][1]
    return (mu_x - mu_y) / np.sqrt(sigma_x + sigma_y - 2 * cov_xy)
```

The $\beta$ value returned from SSMD can be compared against a table to
get an idea of the effect size.


| $\beta$                  | description      |
|--------------------------|------------------|
| 0                        | no effect        |
| $0 < \beta \le 0.25$     | extremely weak   |
| $0.25 < \beta \le 0.5$   | very week        |
| $0.5 < \beta \le 0.75$   | weak             |
| $0.75 < \beta \le 1$     | fairly weak      |
| $1 < \beta \le 1.28$     | fairly moderate  |
| $1.28 < \beta \le 1.654$ | moderate         |
| $1.654 < \beta \le 2$    | fairly strong    |
| $2 < \beta \le 3$        | strong           |
| $3 < \beta \le 5$        | very strong      |
| $\beta > 5$              | extremely strong |
