source("renv/activate.R")

# the following is a fix for having the R package arrow available
# this is a dependency of liesel_gam (through ryp)
# but I could not install it via CRAN on the server
# however, it could be installed via conda in the r-4.5 environment
# so what this code does is, it makes the basic library of this conda
# environment available after initializing renv.
# if I were not using renv here, this would probably not be necessary.
# this should probably be moved to the template.sh.j2
hpc_lib <- "/mnt/vast-standard/home/brachem1/u18549/micromamba/envs/r-4.5/lib/R/library"
if (dir.exists(hpc_lib)) {
  .libPaths(c(hpc_lib, .libPaths()))
}

venv_python <- file.path(getwd(), ".venv", "bin", "python")
if (file.exists(venv_python)) {
  # The .venv is fully built! Lock R onto it so we get pandas.
  Sys.setenv(RETICULATE_PYTHON = venv_python)
} else {
  Sys.setenv(RETICULATE_PYTHON = "~/.pyenv/versions/3.13.0/bin/python3")
}

