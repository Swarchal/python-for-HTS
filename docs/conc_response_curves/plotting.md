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

```python
y_hat_upper = hill_3_param(
    x_interpolated, top+p_err[0], bottom+p_err[1], ec50+p_err[2]
)
y_hat_lower = hill_3_param(
    x_interpolated, top-p_err[0], bottom-p_err[1], ec50-p_err[2]
)

plt.figure(figsize=[10, 6])
plt.plot(x_interpolated, y_hat, label="fitted curve", color="black")
plt.scatter(
    x, y, s=50,label="raw data", color="black", facecolor="white", zorder=999
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
plt.hlines(y=top/2, xmin=min(x), xmax=ec50, linestyle="dotted", color="gray")
plt.text(x=1e-9, y=21, s=f"$EC_{{50}}$ = {ec50:.2E}")
plt.xlabel("Concentration")
plt.ylabel("Response")
plt.xscale("log")
plt.legend()
```

![conc response with undertainty bands](../img/conc_response_3_param_sd_bands.png)
