# Wells

## Creating well labels from row and column positions

If we have row and column positions, we can convert these into well-labels.

```python
from string import ascii_uppercase


def row_col_to_well(row: int, col: int) -> str:
    return f"{ascii_uppercase[row-1]}{col:02}" 

```

```python
row_col_to_well(1, 1)
> "A01"
```


## Row and column positions from well labels


```python
def well_to_row_col(well: str) -> tuple[int, int]:
    row = ord(well[0].upper()) - 64
    col = int(well[1:])
    return row, col
```

```python
well_to_row_col("A01")
> (1, 1)
```

Instead of just returning a tuple, we can use a `NamedTuple`, so we can access
the row and column values by name, rather than position.

```python
from typing import NamedTuple


class Well(NamedTuple):
    label: str
    row: int
    column: int


def make_well(well: str) -> Well:
    row, column = well_to_row_col(well)
    return Well(well, row, column)
```

```python
make_well("A01")
>  Well(label='A01', row=1, column=1)
```

## Zero padding well labels

Wells are typically zero-padded, such as "A01" rather than "A1", but we sometimes
have to convert well labels between the two.

This uses python string formatting, which can handily format integers as strings
with zero padding for us.

```python
def pad_well(well: str) -> str:
    row = well[0]
    col = int(well[1:])
    return f"{row}{col:02}"
```

```python
pad_well("A1")
> "A01"
```

So the magic is done in `{col:02}`. This means the `col` number, and everything
after the colon is the fomatting specifications, in this case we want 2 digits,
zero-padded if necessary `:02`.

Unpadding wells is almost identical, except we don't specify any special formatting
in `{col}`.

```python
def unpad_well(well: str) -> str:
  row = well[0]
  col = int(well[1:])
  return f"{row}{col}"
```

```python
unpad_well("A01")
> "A1"
```

## Generating a well labels

If we wanted to generate all the well labels in a 384-well plate:
```python
from string import ascii_uppercase

wells = [f"{r}{c:02}" for r in ascii_uppercase[:16] for c in range(1, 25)]
```

It's worth noting that python's `range(1, 25)` function is not inclusive of the
final number in the sequence, that's why we went to 25 and not 24.


## 1536 well plates

TODO
