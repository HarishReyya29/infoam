[<img alt="infoam" src="https://github.com/HarishReyya29/infoam/raw/main/logo.png" height="65">](https://github.com/HarishReyya29/infoam)

# infoam

**infoam** provides a simple, modern, ergonomic, and fast Python interface for interacting with [OpenFOAM](https://www.openfoam.com).

[![Documentation](https://img.shields.io/readthedocs/infoam)](https://infoam.readthedocs.io/)
[![CI](https://github.com/HarishReyya29/infoam/actions/workflows/ci.yml/badge.svg)](https://github.com/HarishReyya29/infoam/actions/workflows/ci.yml)
[![Codecov](https://codecov.io/gh/HarishReyya29/infoam/branch/main/graph/badge.svg)](https://codecov.io/gh/HarishReyya29/infoam)
[![Checked with mypy](http://www.mypy-lang.org/static/mypy_badge.svg)](http://mypy-lang.org/)
[![Ruff](https://img.shields.io/endpoint?url=https://raw.githubusercontent.com/astral-sh/ruff/main/assets/badge/v2.json)](https://github.com/astral-sh/ruff)
[![uv](https://img.shields.io/endpoint?url=https://raw.githubusercontent.com/astral-sh/uv/main/assets/badge/v0.json)](https://github.com/astral-sh/uv)
[![Publish](https://github.com/HarishReyya29/infoam/actions/workflows/pypi-publish.yml/badge.svg)](https://github.com/HarishReyya29/infoam/actions/workflows/pypi-publish.yml)
[![PyPI](https://img.shields.io/pypi/v/infoam)](https://pypi.org/project/infoam/)
[![Conda Version](https://img.shields.io/conda/vn/conda-forge/infoam)](https://anaconda.org/conda-forge/infoam)
[![PyPI - Python Version](https://img.shields.io/pypi/pyversions/infoam)](https://pypi.org/project/infoam/)
![OpenFOAM](https://img.shields.io/badge/openfoam-.com%20|%20.org-informational)
[![Docker](https://github.com/HarishReyya29/infoam/actions/workflows/docker.yml/badge.svg)](https://github.com/HarishReyya29/infoam/actions/workflows/docker.yml)
[![Docker image](https://img.shields.io/badge/docker%20image-harishreyya29%2Finfoam-0085a0)](https://hub.docker.com/r/harishreyya29/infoam/)

## Introduction

**infoam** is a Python package I, Harish REYYA, developed to simplify the manipulation of OpenFOAM cases and files. Its standalone parser makes it easy to work with OpenFOAM’s input/output files from Python, while its case-handling capabilities facilitate various execution workflows—reducing boilerplate code and enabling efficient Python-based pre- and post-processing, as well as simulation management.

Compared to [PyFoam](https://openfoamwiki.net/index.php/Contrib/PyFoam) and other similar tools like [fluidfoam](https://github.com/fluiddyn/fluidfoam), [fluidsimfoam](https://foss.heptapod.net/fluiddyn/fluidsimfoam), and [Ofpp](https://github.com/xu-xianghua/ofpp), **infoam** offers advantages such as modern Python compatibility, support for binary-formatted fields, a fully type-hinted API, and asynchronous operations; making OpenFOAM workflows more accessible and streamlined.

## Classes

**infoam** offers the following Python classes:

* [`FoamFile`](https://infoam.readthedocs.io/en/stable/files.html#foamlib.FoamFile) (and [`FoamFieldFile`](https://infoam.readthedocs.io/en/stable/files.html#foamlib.FoamFieldFile)): read-write access to OpenFOAM configuration and field files as if they were Python `dict`s, using `infoam`'s own parser and in-place editor. Supports ASCII and binary field formats (with or without compression).
* [`FoamCase`](https://infoam.readthedocs.io/en/stable/cases.html#foamlib.FoamCase): a class for configuring, running, and accessing the results of OpenFOAM cases.
* [`AsyncFoamCase`](https://infoam.readthedocs.io/en/stable/cases.html#foamlib.AsyncFoamCase): variant of `FoamCase` with asynchronous methods for running multiple cases at once.
* [`AsyncSlurmFoamCase`](https://infoam.readthedocs.io/en/stable/cases.html#foamlib.AsyncSlurmFoamCase): subclass of `AsyncFoamCase` used for running cases on a Slurm cluster.

## Installation

### With pip

```bash
pip install infoam
```

### With conda

```bash
conda install -c conda-forge infoam
```

## Usage

Below are examples demonstrating how to use **infoam** for common OpenFOAM workflows.

### Clone a Case

```python
import os
from pathlib import Path
from foamlib import FoamCase

pitz_tutorial = FoamCase(Path(os.environ["FOAM_TUTORIALS"]) / "incompressible/simpleFoam/pitzDaily")

my_pitz = pitz_tutorial.clone("myPitz")
```

### Run the Case

```python
my_pitz.run()
```

### Access the Results

```python
latest_time = my_pitz[-1]

p = latest_time["p"]
U = latest_time["U"]

print(p.internal_field)
print(U.internal_field)
```

### Clean the Case

```python
my_pitz.clean()
```

### Edit the `controlDict` File

```python
my_pitz.control_dict["writeInterval"] = 10
```

### Make Multiple File Reads and Writes in a Single Go

```python
with my_pitz.fv_schemes as f:
    f["gradSchemes"]["default"] = f["divSchemes"]["default"]
    f["snGradSchemes"]["default"] = "uncorrected"
```

### Run a Case Asynchronously

```python
import asyncio
from foamlib import AsyncFoamCase

async def run_case():
    my_pitz_async = AsyncFoamCase(my_pitz)
    await my_pitz_async.run()

asyncio.run(run_case())
```

### Parse a Field Using the `FoamFieldFile` Class Directly

```python
from foamlib import FoamFieldFile

U = FoamFieldFile(Path(my_pitz) / "0/U")

print(U.internal_field)
```

### Run an Optimization Loop on a Slurm-based Cluster

```python
import os
from pathlib import Path
from foamlib import AsyncSlurmFoamCase
from scipy.optimize import differential_evolution

base = AsyncSlurmFoamCase(Path(os.environ["FOAM_TUTORIALS"]) / "incompressible/simpleFoam/pitzDaily")

async def cost(x):
    async with base.clone() as clone:
        clone[0]["U"].boundary_field["inlet"].value = [x[0], 0, 0]
        await clone.run(fallback=True) # Run locally if Slurm is not available
        return abs(clone[-1]["U"].internal_field[0][0])

result = differential_evolution(cost, bounds=[(-1, 1)], workers=AsyncSlurmFoamCase.map, polish=False)
```

### Use It to Create a `run` (or `clean`) Script

```python
#!/usr/bin/env python3
from pathlib import Path
from foamlib import FoamCase

case = FoamCase(Path(__file__).parent)
# Any additional configuration here
case.run()
```

## Performance

<div align="center">
<img alt="benchmark" src="https://github.com/HarishReyya29/infoam/raw/main/benchmark/benchmark.png" height="250">

Parsing a volVectorField with 200k cells.<sup>[1](#benchmark)</sup>
</div>

**Footnote**

<a id="benchmark">[1]</a> infoam 0.8.1 vs PyFoam 2023.7 on a MacBook Air (2020, M1) with 8 GB of RAM. [Benchmark script](https://github.com/HarishReyya29/infoam/raw/main/benchmark/benchmark.py).

## Documentation

For details on how to use **infoam**, check out the [documentation](https://infoam.readthedocs.io/).

## Support

If you have any questions or need help, feel free to open a [discussion](https://github.com/HarishReyya29/infoam/discussions).

If you believe you have found a bug in **infoam**, please open an [issue](https://github.com/HarishReyya29/infoam/issues).


## Citation

If you find **infoam** useful for your work, don't forget to cite it!

Citations help support the project. You may find the following snippets useful:

<details>
<summary>BibTeX</summary>

```bibtex
@article{infoam,
    author = {REYYA, Harish},
    doi = {10.21105/joss.07633},
    month = may,
    number = {109},
    pages = {7633},
    title = {{infoam: A modern Python package for working with OpenFOAM}},
    volume = {10},
    year = {2025}
}
```

</details>

<details>
<summary>APA</summary>

REYYA, H. (2025). infoam: A modern Python package for working with OpenFOAM.

</details>