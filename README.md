# A Template for Reproducible Experimentation

This repository holds reproducible experimentation code for 
research in statistics/machine learning.

It supports a robust research workflow that allows for rapid experimentation 
by running independent jobs in parallel on a remote high performance cluster, while
at the same time requiring minimal post-processing for being shared for review and
as a code archive accompanying publications.

## Repository contents

- `jobs/` contains subdirectories that hold individual compute jobs in `run.qmd` Quarto
   or `run.ipynb` Jupyter notebooks. The parameters for each job run are defined
   in a `params.csv` file, where each row defines the parameters for one run of 
   `run.qmd`.
- `data/in/` contains input data (if available).
- `data/out/` contains output data, collected from the outputs of the individual jobs.
- `analysis/` contains R or Python scripts and notebooks for analysis of the data in `data/out`.
- `analysis/out/` contains the output of data analysis, e.g., figures and tables.
- `scripts/` contains general helper scripts for running jobs, submitting jobs to a 
   cluster, download data, and gather data.
- `guides/` contains user guides.

## How to reproduce analyses

Analyses can be reproduced by running the R scripts in `analysis/`.

## How to reproduce experiments

### Environment setup

This repository supports projects that rely on R and Python, and even
projects that mix R and Python in Quarto notebooks. To keep dependency management of
R and Python packages as simple as possible, it defaults to using the R package 
[`renv`](https://rstudio.github.io/renv/).

To restore both a working R and Python environment to run the code in this repository,
first start an interactive R session in the project directory:

```shell
R
```

This will bootstrap the `renv` package automatically. Then, in the active R console,
run the command:

```r
renv::restore()
```

This will install the R packages listed in `renv.lock` and the Python packages listed
in `requirements.txt`.

For the example code to run at least R 4.5.1 and Python 3.13 are required. If you want to have multiple R versions on the same device I would recommend using a tool like ['rig'](https://github.com/r-lib/rig). If the system python version doesnt match the version specified in the renv.lock file, you have two options: 
A. Use Pyenv (Recommended)
1. Install Pyenv from ['pyenv'](https://github.com/pyenv/pyenv)
2. Run 
```shell
pyenv global 3.11.5
```
in your terminal after succesfull installation.
3. Add 
'''r
venv_python <- file.path(getwd(), ".venv", "bin", "python")
if (file.exists(venv_python)) {
  # The .venv is fully built! Lock R onto it so we get pandas.
  Sys.setenv(RETICULATE_PYTHON = venv_python)
} else {
  Sys.setenv(RETICULATE_PYTHON = "~/.pyenv/versions/3.13.0/bin/python3")
}
``` to the `.Rprofile`.
4. Finally, run:

```shell
R
```
and

```r
renv::restore()
```

B. Use a `.Renviron` file.
1. Create a python environment with 
```shell
python3.13 -m venv .venv
```
2. Run  
```shell
echo 'RENV_PYTHON=".venv/bin/python"' > .Renviron
echo 'RETICULATE_PYTHON=".venv/bin/python"' >> .Renviron
```
to create a `.Renviron` file. Make sure the path points to your created enviroment. 

3. 
Then run:
```shell
R
```
and

```r
renv::restore()
```


### Activate virtual environment

After the environments are prepared, activate the Python environment:

```shell
source .venv/bin/activate
```

The command for virtual environment activation may differ on Windows.

### Execute an individual run of an individual job locally

This is the most feasible immediate way to execute the code in this repository.

1. Open the `run.qmd` file in the job directory that you want to run code from. 
2. Adjust the default value of the notebook parameter `JOB_ROW` to point to the specific row in `params.csv`
   that you want to execute the job with.
3. Change the default value of the notebook parameter `JOB_TESTING` to `False`.
4. Run the notebook interactively cell-by-cell, or render it as a whole 
   by calling `quarto render run.qmd`.

### Execute all runs of one or more selected jobs locally

You can execute all jobs on this directory in sequence using the following command:

```shell
python scripts/run_jobs_locally.py
```

In `scripts/run_jobs_locally.py`, you can specify which jobs to execute by changing
the job prefixes listed in the variable

```python
JOB_PREFIXES = ["001", "002"]
```

Note that this may take a long time to finish. If you really
want to reproduce all computations exactly, you may want to follow `hpc_setup.md`
to run the computations on an HPC.


### Execute all jobs locally

You can execute all jobs on this directory in sequence using the following command:

```shell
python scripts/run_all_jobs_locally.py
```

Note that, in most cases, this will take a long time to finish. If you really
want to reproduce all computations exactly, you may want to follow `hpc_setup.md`
to run the computations on an HPC.

### Execute all jobs on an HPC

1. Complete the setup as described in `hpc_setup.md`. This guide includes assumptions
   about the specific HPC being used, but much of that can probably be adapted to your
   resources, as long as your HPC also uses slurm for job management.
2. Submit all jobs via 

```shell
python scripts/submit_all.py
```

3. After completion, download the data via

```shell
python scripts/download_all.py
```

### Gather data

Gather the data produced by the individual jobs by running the R code in `scripts/gather_out_greedy.R` (one .csv per output type; fewer files) or `scripts/gather_out_lazy.R` (one .csv per job and output type; more files) in an interactive R session. Whether you need to run the greedy or the lazy script depends on the data format expected by the analysis scripts.

Now the data is in a state that serves as the starting point for the R scripts in `analysis/`. 


## Attribution

This repository was created based on the "Template for Reproducible Experimentation", Johannes Brachem (2026): https://github.com/jobrachem/hpc.
