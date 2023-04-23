# Z-score

The Z-score can be used for both standardisation of values as well as hit-selection.

At it's most basic, it's subtracting the mean of a population ($\mu$) and dividing by
the standard deviation of the same population ($\sigma$), resulting in values
centered at 0, and Z being the number of standard deviations away from the
population centre.

$$
Z = \frac{x - \mu_x}{\sigma_x}
$$


```python
def z_score(x: np.ndarray) -> np.ndarray:
    return (x - np.mean(x)) / np.std(x)
```


## robust Z-score

The robust z-score is similar to the typical z-score, but is less sensitive to
outliers. It uses the median and median absolute deviation instead of mean
and standard deviation.

```python
import numpy as np
from scipy.stats import median_abs_deviation as mad

def robust_z_score(x: np.ndarray) -> np.ndarray:
    return (x - np.median(x)) / mad(x)
```


## Z-score using control values

A common metric in HTS is to use the z-score, but with the mean and standard deviation
of the neutral control wells instead of the population mean and standard deviation.

```python
def z_score_to_controls(x: np.ndarray, controls: np.ndarray) -> np.ndarray:
    return (x - np.mean(controls)) / np.std(controls)
```

### From a dataframe

If we have a dataframe with columns of readout and annotations we can write a
function that will subset the data for us to calculate the Z-score against
the control values.

| Cell_Count | Metadata_Compound_Name |
|------------|------------------------|
| 3987       | ABC00001               |
| 4985       | ABC00002               |
| 4590       | ABC00003               |
| 3098       | ABC00004               |
| 4909       | DMSO                   |

```python
import pandas as pd

def z_score_to_controls_df(
    df: pd.DataFrame,
    feature_cols: list[str],
    control_column_name: string,
    control_name: string,
) -> pd.DataFrame:
    df_copy = df.copy()
    control_df = df[df[control_column] == control_name]
    mu = control_df[feature_cols].mean()
    sigma = control_df[feature_cols].std()
    df_copy[feture_cols] = (df[feature_cols] - mu) / sigma
    return df_copy


df_zscored = z_score_to_controls_df(
    df = df,
    feature_cols = ["Cell_Count"],
    control_column_name = "Metadata_Compound_Name",
    control_name = "DMSO"
)
```
