# Fitting concentration response curves

Fitting dose or concentration response curves is a common but nuanced task. To do this in python
we can define the curve fitting function in numpy, and use [`scipy.optimize.curve_fit()`](https://docs.scipy.org/doc/scipy/reference/generated/scipy.optimize.curve_fit.html) to fit a curve to our data.


## 3 parameter curve

### Concentration

```python
import numpy as np

def hill_3_param(
    x: np.ndarray, top: float, botton: float, ec50: float
) -> np.ndarray:
    return bottom + x * (top - bottom) / (ec50 + x)
```

### log concentrations

```python
def hill_3_param(
    x: np.ndarray, top: float, botton: float, log_ec50: float
) -> np.ndarray:
    return bottom + (top - bottom)  / (1 + 10**(log_ec50 - x))
```


## 4 parameter curve

### Concentration

```python
import numpy as np

def hill_4_param(
    x: np.ndarray, top: float, bottom: float, ec50: float, hillslope: float
) -> np.ndarray:
    numerator = bottom + (x**hillslope) * (top - bottom)
    denominator = ((x**hillslope) + (ec50**hillslope))
    return numerator / denominator
```

### log concentrations

```python
def hill_4_param(
    x: np.ndarray, top: float, bottom: float, log_ec50: float, hillslope: float
) -> np.ndarray:
    return bottom + (top - bottom) / (1 + 10**((log_ec50 - x) * hillslope))
```


## Fitting the model

At it's simplest, assuming `x` is an array of concentrations and `y` is an array of
corresponding responses, we can run:
```python
popt, pcov = scipy.optimize.curve_fit(hill_3_param, x, y)
```

This returns `popt`, which are the fitted [`top`, `bottom`, `ec50`, `hillslope` (if 4 parameter)]
values for your model. These can be used directly such as the $EC_{50}$, or used
with interpolated data to plot a curve.

`pcov` is the covariance of your parameter estimates which can be used to calculate
the standard deviation of your $EC_{50}$ for instance.


### Adjusting the curve fitting

You might find the curve fitting hasn't worked particularly well with the default
settings, in this case it often helps to adjust some parameters in `scipy.optimize.curve_fit()`.

#### `p0`

`p0` serves as the initial starting point for your model parameters. The closer
these are expected values, the easier the job the curve fitting algorithm will have.
If you don't specify starting points for `p0`, they will start as 1 for all values.


```python
p0 = [100, 0, 1e-3]  # starting point for [top, bottom, ec50] params
popt, pcov = scipy.optimize.curve_fit(hill_3_param, x, y, p0=p0)
```

#### `bounds`

You often have some idea if the parameters the curve fitting produces are
at all possible given your assay. For example a negative hill-slope, or a `bottom`
values less than 0. We can specify upper and lower bounds in the curve fitting
with the `bounds` argument. These should be fairly relaxed, but within
the realms of possibility.

```python
p0 = [100, 0, 1e-3]
bounds = (
  (0, 0, 0),     # lower bounds for [top, bottom, ec50]
  (300, 300, 1)  # upper bounds for [top, bottom, ec50]
)
popt, pcov = scipy.optimize.curve_fit(hill_3_param, x, y, p0=p0, bounds=bounds)
```

If you want to specify bounds for some parameters but leave others boundless,
use `-np.inf` and `np.inf`.


E.g here we set upper and lower bounds for the `top` and `bottom` parameters,
but allow `ec50` to be fit to any value.
```python
bounds = (
  (0, 0, -np.inf),     
  (300, 300, np.inf) 
)
popt, pcov = scipy.optimize.curve_fit(hill_3_param, x, y, bounds=bounds)
```

!!! note

    If you set `bounds`, you should be aware of the default values of
    `p0` being initialised as 1. If your bounds do not cover 1 then your model will
    fail to fit. So it's probably wise to set values for `p0` if you're also
    setting `bounds`.


## `pcov` and parameter uncertainty

The `pcov` object returned on `scipy.optimize.curve_fit()` is an n by n matrix
of estimated parameter covariances, where n is the number of parameters in your
model. We can use the diagonal of this matrix to get the estimated standard
deviation of our parameters.

```python
popt, pcov = scipy.optimize.curve_fit(hill_3_param, x, y)

p_err = np.sqrt(np.diag(pcov))
```

Since our `hill_3_param` model has 3 parameters, `p_err` will now the variance
of those 3 parameters in the same order as their arguments in the original
function (top, bottom, ec50).

```python
print(f"EC50 = {popt[2]} ({p_err[2]} std. dev.)")
```
