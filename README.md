# From Kepler-ish to Newton-ish: Understanding learned world models in transformers

This code repository contains or points to all materials required for creating and hosting the publication entitled, *"From Kepler-ish to Newton-ish: Understanding learned world models in transformers"*.

The publication is hosted at [this URL](https://arcadia-science.github.io/world-model-orbits/).

## Data Description

No external data is required. The trajectory data and the trained checkpoints are
both committed to this repository, so the notebook runs against the exact data and
models the publication used.

**Trajectories** (`data_cv/`, ~21 MB). 20,000 two-body Kepler orbits, split 50/50
into train and test. Each orbit is 100 samples of the $(x, y)$ coordinates of a point
mass orbiting a fixed central mass with $GM = 1$, started at perihelion and integrated
with `scipy.integrate.solve_ivp` at `rtol = atol = 1e-8`. Eccentricity, semi-major axis
and initial orientation are drawn uniformly per orbit from $[0, 0.8)$, $[0.5, 2.0)$ and
$[0, 2\pi)$; coordinates span roughly $\pm 3.5$. Generation is seeded (`seed = 1`),
which makes it repeatable on one machine, but the floating-point results differ
slightly between platforms, so the data is versioned rather than regenerated.

| file | contents |
|---|---|
| `trajectories_chunk_{000000,000001}.pt` | float32 tensors of shape (10000, 100, 2) — the coordinates themselves |
| `orbital_params_chunk_*.py` | per-orbit eccentricity, semi-major/minor axes, linear eccentricity, mean radius, Laplace–Runge–Lenz vector, initial orientation |
| `euler_error_chunk_*.pt` | per-timestep deviation of a forward-Euler integration from the reference solution |
| `metadata.pt` | generation settings and coordinate ranges |

**Checkpoints** (`checkpoints/kepler_cv_blocksize_elm/`, ~424 KB). Three bias-free
nanoGPT models — 2 layers, 1 head, width 32, two-dimensional input — identical apart
from the context length and training-noise settings tabulated in step 3. Removing the
biases is what makes the equivalent-linear-mapping analysis exact.

Trajectory generation and the training loop are adapted from Liu et al.'s
[newton-kepler](https://github.com/KindXiaoming/newton-kepler)

## Reproduce

The project is a [Calkit](https://docs.calkit.org) pipeline, defined in
[`calkit.yaml`](calkit.yaml): generate the data, train the models, and execute
`index.ipynb`. Install [Calkit](https://docs.calkit.org/installation/) and run it:

```bash
curl -LsSf install.calkit.org | sh
calkit run
```

The first run offers to install Conda if it's missing. Or run `make setup`, which installs Calkit if it isn't yet, then Conda and the environment.

This creates the Conda environment from `env.yml` and the exact versions
recorded in `.calkit/env-locks/` (Python, every package, and Quarto),
then runs whatever is out of date with its code, data, or environment, and skips
the rest. On a fresh clone everything is up to date, so it does nothing; edit a
cell of `index.ipynb` or a module in `src/analysis/` and it re-executes the
notebook. `calkit status` shows what's out of date and why, and a CI check
([`pipeline.yml`](.github/workflows/pipeline.yml)) fails any pull request that
changes code without re-running what depends on it.

| stage | does | runs when |
|---|---|---|
| `generate-data` | `python -m analysis.data_generation`, writing `data_cv/` (~2.5 min) | `data_generation.py` or the environment changes |
| `train` | `python -m analysis.training`, writing the three checkpoints | never: it's frozen (see below) |
| `notebook` | `quarto render index.ipynb --execute`, writing the outputs into `index.ipynb` (~1.5 min) | the notebook, `src/analysis/`, the data, the checkpoints, the fonts, or the environment changes |

The `train` stage is frozen, so `calkit run` never retrains the models: the
committed checkpoints are the ones the publication used, and training isn't
bit-reproducible on a GPU. It records how they were made. To retrain, run
`calkit run train --force`, preferably on a GPU (~20k steps per model):

| block_size | noise_scale | role |
|---|---|---|
| 2   | 0.1 | noisy short-context (rotation heuristic) |
| 100 | 0.1 | noisy long-context (Keplerian) |
| 2   | 0.0 | noiseless short-context (Verlet + Newtonian gravity) |

`python -m analysis.training --smoke` is a fast sanity check (200 steps). It writes
its checkpoints with a `_smoke` suffix, so it never overwrites the committed ones.

The numbers quoted in the text are computed by the notebook with Quarto inline
expressions (`` `{python} ...` ``) rather than typed in, so they can't drift from
the results. Figures use the fonts in `fonts/`, so they render the same on any
machine.

### Without Calkit

With [Conda](https://conda-forge.org/download/):

```bash
conda env create -n world-model-orbits --file env.yml  # includes Quarto
conda activate world-model-orbits
python -m analysis.data_generation                      # only to regenerate data_cv/
quarto render index.ipynb --execute
```

## Contribute

Please see [CONTRIBUTING.qmd](pages/CONTRIBUTING.qmd).
