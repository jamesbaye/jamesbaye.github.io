# Beware of Pandas index misalignment

A few weeks ago I encountered a behaviour in Pandas that is discussed surprisingly
little. I wanted to set an `.iloc` slice of a `pd.Series` with values coming
from another `pd.Series`.  In doing so I learnt that one must be very careful
about index alignment, as it can lead to unexpected behaviour.

Say I have two series, `old` and `new`,

```python
import pandas as pd
```

```python
old = pd.Series(["A", "B", "C"])
new = pd.Series(["new B", "new C"])
```

I want to replace the values `["B", "C"]` in `old`, with values `["new B", "new C"]`
in `new`. One might naively do:

```python
old.iloc[1:3] = new
```

However this returns

```python
old
```
>    0        A
>    1    new C
>    2      NaN
>    dtype: object

What happened? This is because pandas aligns series based on their indices.
In the assignment `old.iloc[1:3] = new`, the indices on the left and on the right
hand sides do not match:

```python
old.iloc[1:3]
```
    1    B
    2    C
    dtype: object

```python
new
```
    0    new B
    1    new C
    dtype: object

Since pandas aligns series based on their indices, `B` is replaced with `new C`
(both have an index of 1), and `C` is replaced with `NaN` since there is no value
in `new` that has an index of `2`.

We can get the intended behaviour by explicitely setting the right indices in `new`

```python
new = pd.Series(["new B", "new C"], index=[1, 2])
old.iloc[1:3] = new
old
```
    0        A
    1    new B
    2    new C
    dtype: object

Alternatively, and more practically, we can assign `new.values`, which is an
array rather than the original indexed series. Then, no index alignment can occur:

```python
old = pd.Series(["A", "B", "C"])
new = pd.Series(["new B", "new C"])
old.iloc[1:3] = new.values
old
```
    0        A
    1    new B
    2    new C
    dtype: object

In general, it is best to avoid `.iloc` slice assignments, especially if slice
assignments can be performed with built-ins such as `groupby` instead.
But if `.iloc` you must, hopefully now you won't go loco like I did, with
pandas misalignment!
