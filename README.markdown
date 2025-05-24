# infoam

**infoam** is a Python library designed to streamline interaction with [OpenFOAM](https://www.openfoam.com), offering a user-friendly, contemporary, and efficient interface.

## Introduction

**infoam** is a Python package I, Harish REYYA, crafted to enhance the management of OpenFOAM cases and files. Equipped with an independent parser, it simplifies handling OpenFOAM’s file formats in Python, while its case management tools support efficient workflows, minimizing repetitive code and enabling seamless pre-processing, post-processing, and simulation execution.

In comparison to tools like [PyFoam](https://openfoamwiki.net/index.php/Contrib/PyFoam), [fluidfoam](https://github.com/fluiddyn/fluidfoam), [fluidsimfoam](https://foss.heptapod.net/fluiddyn/fluidsimfoam), and [Ofpp](https://github.com/xu-xianghua/ofpp), **infoam** stands out with its support for modern Python versions, handling of binary field formats, a type-hinted API, and asynchronous processing, making OpenFOAM tasks more approachable and optimized.

## Classes

**infoam** provides the following Python classes:

* [`FoamFile`](https://infoam-docs.readthedocs.io/en/stable/files.html#foamlib.FoamFile) (and [`FoamFieldFile`](https://infoam-docs.readthedocs.io/en/stable/files.html#foamlib.FoamFieldFile)): Enable reading and writing OpenFOAM configuration and field files as Python dictionaries, using infoam’s custom parser and editor. Supports both ASCII and binary formats, with or without compression.
* [`FoamCase`](https://infoam-docs.readthedocs.io/en/stable/cases.html#foamlib.FoamCase): Facilitates configuration, execution, and result retrieval for OpenFOAM cases.
* [`AsyncFoamCase`](https://infoam-docs.readthedocs.io/en/stable/cases.html#foamlib.AsyncFoamCase): A version of `FoamCase` with asynchronous methods for concurrent case execution.
* [`AsyncSlurmFoamCase`](https://infoam-docs.readthedocs.io/en/stable/cases.html#foamlib.AsyncSlurmFoamCase): Extends `AsyncFoamCase` for running cases on Slurm-based HPC clusters.

## Installation

### Using pip

```bash
pip install infoam
```

### Using conda

```bash
conda install -c conda-forge infoam
```

## Usage

The following examples illustrate how to leverage **infoam** for typical OpenFOAM tasks.

### Duplicating a Case

```python
import os
from pathlib import Path
from foamlib import FoamCase

pitz_tutorial = FoamCase(Path(os.environ["FOAM_TUTORIALS"]) / "incompressible/simpleFoam/pitzDaily")

my_pitz = pitz_tutorial.clone("myPitz")
```

Create a copy of an existing OpenFOAM case for further modification.

### Executing a Case

```python
my_pitz.run()
```

Run the duplicated case to perform the simulation.

### Retrieving Results

```python
latest_time = my_pitz[-1]

p = latest_time["p"]
U = latest_time["U"]

print(p.internal_field)
print(U.internal_field)
```

Access and display the pressure and velocity fields from the latest simulation timestep.

### Cleaning Up a Case

```python
my_pitz.clean()
```

Remove temporary files generated during the simulation to reset the case.

### Modifying the `controlDict` File

```python
my_pitz.control_dict["writeInterval"] = 10
```

Adjust the write interval in the case’s control settings.

### Performing Multiple File Operations Efficiently

```python
with my_pitz.fv_schemes as f:
    f["gradSchemes"]["default"] = f["divSchemes"]["default"]
    f["snGradSchemes"]["default"] = "uncorrected"
```

Update multiple settings in the finite volume schemes file in a single operation.

### Running a Case Asynchronously

```python
import asyncio
from foamlib import AsyncFoamCase

async def run_case():
    my_pitz_async = AsyncFoamCase(my_pitz)
    await my_pitz_async.run()

asyncio.run(run_case())
```

Execute a case asynchronously to enable concurrent processing.

### Directly Parsing a Field

```python
from foamlib import FoamFieldFile

U = FoamFieldFile(Path(my_pitz) / "0/U")

print(U.internal_field)
```

Load and inspect the initial velocity field from a file.

### Optimizing on a Slurm Cluster

```python
import os
from pathlib import Path
from foamlib import AsyncSlurmFoamCase
from scipy.optimize import differential_evolution

base = AsyncSlurmFoamCase(Path(os.environ["FOAM_TUTORIALS"]) / "incompressible/simpleFoam/pitzDaily")

async def cost(x):
    async with base.clone() as clone:
        clone[0]["U"].boundary_field["inlet"].value = [x[0], 0, 0]
        await clone.run(fallback=True) # Run locally if Slurm is unavailable
        return abs(clone[-1]["U"].internal_field[0][0])

result = differential_evolution(cost, bounds=[(-1, 1)], workers=AsyncSlurmFoamCase.map, polish=False)
```

Perform an optimization loop by running cases on a Slurm cluster or locally.

### Creating a `run` or `clean` Script

```python
#!/usr/bin/env python3
from pathlib import Path
from foamlib import FoamCase

case = FoamCase(Path(__file__).parent)
# Add custom configurations here
case.run()
```

Develop a script to automate case execution or cleanup.

## Performance

<div align="center">

Parsing a volVectorField with 200k cells.<sup>[1](#benchmark)</sup>
</div>

**Footnote**

<a id="benchmark">[1]</a> infoam 0.8.1 outperforms PyFoam 2023.7 on a MacBook Air (2020, M1) with 8 GB of RAM. [Benchmark script](https://github.com/HarishReyya29/infoam/raw/main/benchmark/benchmark.py).

## Documentation

Explore detailed guidance on using **infoam** in the [documentation](https://infoam-docs.readthedocs.io/).

## Support

For questions or assistance, start a [discussion](https://github.com/HarishReyya29/infoam/discussions).

To report a potential bug in **infoam**, please create an [issue](https://github.com/HarishReyya29/infoam/issues).

## Contributing

Contributions to **infoam** are welcome! Review the [contributing guidelines](https://github.com/HarishReyya29/infoam/raw/main/CONTRIBUTING.md) for details.

## Citation

If **infoam** proves valuable for your work, please consider citing it to support the project.

The following citation formats are provided for convenience:

<details>
<summary>BibTeX</summary>

```bibtex
@article{infoam,
    author = {REYYA, Harish},
    doi = {10.21105/joss.07633},
    month = may,
    number = {109},
    pages = {7633},
    title = {{infoam: A Python Package for OpenFOAM Interaction}},
    volume = {10},
    year = {2025}
}
```

</details>

<details>
<summary>APA</summary>

REYYA, H. (2025). infoam: A Python Package for OpenFOAM Interaction. 10(109), 7633. https://doi.org/10.21105/joss.07633

</details>
