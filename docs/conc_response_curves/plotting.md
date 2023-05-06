# Plotting concentration response curves

Using example data from [here](https://gist.githubusercontent.com/Swarchal/f853a013cb5c055343b3c0efbf0b79ba/raw/8768cc544d80c0314ab710151b3d16a5801bb346/conc_response.csv), we'll fit and plot a 3 parameter concentration response curve.


## Generating interpolated points

Given our parameters from our fitted model, we can now generate an estimated response
value for any given concentration.


```python
import numpy as np
import pandas as pd
from scipy.optimize import curve_fit


def hill_3_param(
    x: np.ndarray, top: float, botton: float, ec50: float
) -> np.ndarray:
    return bottom + x * (top - bottom) / (ec50 + x)


# read in example concentration response data
df = pd.read_csv("conc_response.csv").dropna()
df_a = df[df["drug"] == "A"]

conc = df_a["conc"].to_numpy()
response = df_a["response"].to_numpy()

popt, pcov = curve_fit(hill_3_param, x=con, y=response)
top, bottom, ec50 = popt
```

We now have our parameters of our 3 parameter model (top, bottom, ec50):
```
[4.05044597e+01 3.22471271e-01 1.10047765e-07]
```


Now we generate 100 interpolated values between our minimum and maximum concentration
values to plot a smoother curve.

```
x_interpolated = np.logspace(min(np.log10(conc)), max(np.log(conc)), 100)
y_interpolated = hill_3_param(x_interpolated, *popt)
```


## Plotting with matplotlib

Now we can plot both the raw data points and the fitted curve in matplotlib.

```python
plt.plot(x_interpolated, y_interpolated, label="fitted curve", color="black")
plt.scatter(
    conc, response, label="raw data", color="black", facecolor="white", zorder=99
)
plt.vlines(x=ec50, ymin=0, ymax=top/2, linestyle="dotted", color="gray")
plt.hlines(y=top/2, xmin=min(conc), xmax=ec50, linestyle="dotted", color="gray")
plt.text(x=1e-9, y=23, s=f"$EC_{{50}} = {ec50:.2E}$")
plt.xlabel("Concentration")
plt.ylabel("Response")
plt.xscale("log")
plt.legend()
```

![conc response curve](../img/conc_response_3_param_single.png)


## Plotting standard deviations with `pcov`

As mentioned in the curve fitting section we can get the standard deviations of
our curve parameters from the `pcov` matrix.

```python
p_err = np.sqrt(np.diag(pcov))
```

We can then plot this uncertainty on the curve by calculating 2 additional curves,
one with our parameters + standard deviation, and another - standard deviation.
Then we can use `plt.fill_between()` to fill the area between the upper and lower
bounded curves.

```python
y_hat_upper = hill_3_param(
    x_interpolated, top+p_err[0], bottom+p_err[1], ec50+p_err[2]
)
y_hat_lower = hill_3_param(
    x_interpolated, top-p_err[0], bottom-p_err[1], ec50-p_err[2]
)

plt.plot(x_interpolated, y_interpolated, label="fitted curve", color="black")
plt.scatter(
    conc, response, label="raw data", color="black", facecolor="white", zorder=99
)
plt.fill_between(
    x_interpolated,
    y1=y_hat_upper,
    y2=y_hat_lower,
    color="gray",
    alpha=0.3,
    label="$\pm$ 1SD"
)
plt.vlines(x=ec50, ymin=0, ymax=top/2, linestyle="dotted", color="gray")
plt.hlines(y=top/2, xmin=min(conc), xmax=ec50, linestyle="dotted", color="gray")
plt.text(x=1e-9, y=23, s=f"$EC_{{50}} = {ec50:.2E}$")
plt.xlabel("Concentration")
plt.ylabel("Response")
plt.xscale("log")
plt.legend()
```

![conc response with uncertainty bands](../img/conc_response_3_param_sd_bands.png)

## Plotting multiple curves

If we're plotting multiple curves it's worth writing some functions so we
don't have to repeat lots of curves for each drug.

Since our data is in the form of a dataframe, we can split by the `drug` column
and loop through the different drugs, calculating the curve fitting parameters and plot
each one in turn

```python
from typing import NamedTuple

class Params(NamedTuple):
    top: float
    bottom: float
    ec50: float


class ModelFit(NamedTuple):
    x_raw: np.ndarray
    y_raw: np.ndarray
    x_interp: np.ndarray
    y_fit: np.ndarray
    params: Params


def fit_3_param_curve(df: pd.DataFrame) -> ModelFit:
    x = df["conc"].to_numpy()
    y = df["response"].to_numpy()
    popt, pcov = curve_fit(hill3param, x, y)
    x_interp = np.logspace(min(np.log10(x)), max(np.log10(x)), 100)
    y_fit = hill3param(x_interp, *popt)
    return ModelFit(x, y, x_interp, y_fit, Params(*popt))


def plot_3_param_curve(model_fit: ModelFit):
    params = model_fit.params
    plt.plot(model_fit.x_interp, model_fit.y_fit, label=name)
    plt.scatter(model_fit.x_raw, model_fit.y_raw, s=50, zorder=999)
    plt.vlines(
        x=params.ec50, ymin=0, ymax=params.top/2, linestyle="dotted", color="gray"
    )
    plt.hlines(
        y=params.top/2, xmin=min(x), xmax=params.ec50, linestyle="dotted", color="gray"
    )
    

plt.figure(figsize=[10, 6])
for name, group in df.groupby("drug"):
    model_fit = fit_3_param_curve(group)
    plot_3_param_curve(model_fit)
plt.xlabel("Concentration")
plt.ylabel("Response")
plt.xscale("log")
plt.legend(title="Drug", loc="upper left")
```

![multiple conc response curves](../img/conc_response_3_param_multiple.png)
