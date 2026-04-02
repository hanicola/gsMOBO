# mobo-medium-conditions

Code and data for the project **“Multiobjective design of growth media with genome-scale metabolic models and Bayesian optimization”**, implementing a **multi-objective Bayesian optimization (gsMOBO)** workflow for designing and optimizing growth media.

The repository contains:

* The **full optimization pipeline** (simulation and gsMOBO loop),
* **Helper notebooks** for adapting the method to your own model,
* **Processed data** used to generate the figures in the associated manuscript,
* **Raw data** corresponding to the processed data
---

## Repository structure

Top-level layout:

```text
mobo-medium-conditions/
├── analysis/
├── data/
├── optimization/
├── .DS_Store
├── .gitignore
└── README.md 
```

### `analysis/`

> Jupyter notebooks and helper code for analyzing results and generating figures.

Contains:

* **Figure-generation notebooks**: Notebooks that read pre-processed `.csv` data (from `data/`) and generate the figures appearing in the main text and supplementary material.
* **Post-processing notebooks / scripts**: Code that:

  * Reads the **complete simulation data** (produced by the optimization workflow),
  * Extracts the relevant subsets,
  * Writes **compact `.csv` files** into `data/` for plotting.

### `data/`

> Processed data used to build figures.

Contains:

* **CSV files (`*.csv`)**:

  * Pre-processed subsets of the full simulation results (e.g., selected conditions, Pareto fronts, summary metrics),
  * Data structured in a way that is convenient for the plotting notebooks in `analysis/`.

### `optimization/`

> Full code needed to run the Bayesian optimization and simulations. 

* **Main pipeline notebook** (referred to in the original README as `Main_MOBO.ipynb`):

  * Loads your metabolic/mechanistic model (via [CobraPy](https://cobrapy.readthedocs.io/) or compatible tools),
  * Defines medium components, bounds, and costs,
  * Configures and calls the gsMOBO routine (e.g. `media_BayesOpt`),
  * Runs simulations and stores outputs.

* **Auxiliary notebooks/scripts**:

  * Utility functions for:

    * Building design spaces,
    * Evaluating objectives (growth, production, cost),
    * Handling constraints and batch evaluations,
  * Helper routines for saving simulation outputs in a structured format consumed by `analysis/`.
  * Code to conduct a sensitivity analysis of the Pareto-optimal data

## Getting started

Create a virtual environment (via `venv` or `conda`) and install the dependencies used in `analysis/requirements.txt`


## Adapting the method to your own project

The repository is designed so that **you mostly only need to edit `Main_MOBO.ipynb`** to apply the approach to a new system. 

### Step 1 — Load your model

In `optimization/Main_MOBO.ipynb`:

* Load or construct your **metabolic / process model** using CobraPy or a compatible library.
* If necessary, adapt your model so that:

  * Reaction and metabolite naming follow CobraPy conventions,
  * Key fluxes (biomass, product) are clearly defined.

### Step 2 — Define the design space (medium components)

Still in `Main_MOBO.ipynb`:

* Specify the **medium components** you want to optimize (e.g., carbon sources, nitrogen sources, trace elements).
* For each component, define:

  * **Bounds** (minimum/maximum concentrations or fluxes),
  * **Cost** (per unit), if medium cost is one of your objectives.

If a cost is not provided, default values are used (as per the original project description). ([GitHub][1])

### Step 3 — Define constraints, if applicable

In case your optimisation is subject to constraints following e.g., from the medium formulation, encode them in a way compatible with the requirements of the algorithm.

### Step 4 — Configure objectives

Choose the combination of objectives you care about, e.g.:

* **Growth** (maximize biomass flux/growth rate),
* **Production** (maximize flux of a particular reaction representing your product),
* **Cost** (minimize total medium cost).

You can include any subset or combination, as long as they are compatible with the model and optimization routine.

### Step 5 — Run the Bayesian optimization (`media_BayesOpt`)

From `Main_MOBO.ipynb`:

* Call the main routine (referred to in the original README as `media_BayesOpt`) with:

  * The model,
  * Design space definition (medium components, bounds, costs),
  * Objective configuration and weights (if needed),
  * BO hyperparameters (number of iterations, batch size, kernel settings, etc., depending on implementation).

The function will:

1. Propose medium compositions to test,
2. Evaluate objectives through simulations (or experiments, if connected),
3. Update the surrogate model and acquisition function,
4. Iterate until the stopping criterion is met.

### Step 6 — Export and analyze results

* Save the optimization traces and final Pareto fronts.
* Use or extend the `analysis/` notebooks to:

  * Inspect trade-offs between objectives,
  * Visualize medium compositions and model behavior,
  * Export `.csv` datasets to `data/` and regenerate figures in `figs/`.
