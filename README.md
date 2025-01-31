# Fast Density-Based Clustering Validation (DBCV)

Fast Density-Based Clustering Validation (DBCV) implementation for Python, with suppport for parallel and dynamically-adjustable high precision computation.

![Moon with noise example.](./assets/example_moons_with_noise.png)

---

## Table of contents
1. [Install](#install)
2. [Usage](#usage)
    1. [Basic usage](#basic-usage)
    2. [Example with noise "cluster"](#example-with-noise-cluster)
    3. [Multiprocessing](#multiprocessing)
    4. [High precision computation](#high-precision-computation)
3. [Reference](#reference)


---

## Install
```bash
python -m pip install "git+https://github.com/mikolaj-kow/DBCV"
```

---

## Usage

### Basic usage
```python
import dbcv
import sklearn.datasets

X, y = sklearn.datasets.make_moons(n_samples=300, noise=0.05, random_state=1782)

score = dbcv.dbcv(X, y)
print(score)
# 0.978017974682571
```

### Example with noise "cluster"
```python
import sklearn.datasets
import numpy as np
import dbcv

X, y = sklearn.datasets.make_moons(n_samples=500, noise=0.10, random_state=1782)

noise_id = -1  # NOTE: dbcv.dbcv(..., noise_id=-1) by default.

rng = np.random.RandomState(1082)
X_noise = rng.uniform(*np.quantile(X, (0, 1), axis=0), size=(100, 2))
y_noise = 100 * [noise_id]

X, y = np.vstack((X, X_noise)), np.hstack((y, y_noise))

score = dbcv.dbcv(X, y, noise_id=noise_id)
print(score)
# 0.8072520501068048
```

### Multiprocessing
You can use the `dbcv.dbcv(..., n_processes=n)` argument to specify the number of parallel processes during computations. The default value of `n_processes` is `"auto"`, which is equivalent to 1 for datasets with 200 or fewer instances, and 4 for datasets with more than 200 instances.

```python
import dbcv
import sklearn.datasets

X, y = sklearn.datasets.make_moons(n_samples=300, noise=0.05, random_state=1782)

score = dbcv.dbcv(X, y, n_processes=2)
print(score)
# 0.978017974682571
```

### High precision computation

If you are facing underflow issues and need more precision bits, you can dynamically adjust how many are available by enabling `dbcv.dbcv(..., enable_dynamic_precision=True)`. The number of precision bits available is controlled by `dbcv.dbcv(..., enable_dynamic_precision=True, bits_of_precision=n)`.

```python
import dbcv
import sklearn.datasets

X, y = sklearn.datasets.make_moons(n_samples=300, noise=0.05, random_state=1782)

score = dbcv.dbcv(X, y, enable_dynamic_precision=True, bits_of_precision=512)
print(score)
# 0.978017974682571
```

Note that enabling this option makes the DBCV calculation much slower than the plain numpy/scipy version. However, this option may be necessary for computing DBCV in high dimensions, since density calculations easily underflow under that circumstance. The table below displays runtimes from computing DBCV on an dataset of shape (10,000, 784) twenty times in a row for variable amount of precision bits and for dynamic precision disabled.

Bits           | Runtime mean ± std (slowdown w.r.t. 'Off') |
:--            | :--                             |
Off (64 bits)  | 8.7808 ± 0.0475                 |
64             | 42.8156 ± 0.2801 (**+3.88x**)   |
128            | 45.3012 ± 0.0959 (**+4.16x**)   |
256            | 49.8338 ± 0.2369 (**+4.68x**)   |
512            | 58.2917 ± 0.1223 (**+5.64x**)   |
1024           | 80.1517 ± 0.2222 (**+8.13x**)   |

---

## Reference
Moulavi, Davoud & Andretta Jaskowiak, Pablo & Campello, Ricardo & Zimek, Arthur & Sander, Joerg. (2014). Density-Based Clustering Validation. 10.1137/1.9781611973440.96.
