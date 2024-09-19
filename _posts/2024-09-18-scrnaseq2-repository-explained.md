---
title: scrnaseq2 repository explained
authors: [ktrns, andpet0101]
categories: [Getting started]
tags: [code, modules]
---

## The concept of modules

An analysis can be split into individual parts (pre-processing, normalization, etc.). 

We have structured these parts into separate modules whithin the `scrnaseq2` git repository. The core concept is:

- Modules read some input, do an analysis, and store the output for the next modules.
- For the same analysis part, there can be different modules depending on the kinds of data, e.g. normalization for RNA-seq data and normalization for ATAC-seq data.
- Modules can be chained into an analysis.
- Modules can be run with parameters.

This has the following advantages:

- Different types of analyses are possible.
- Modules that were already run do not need to be run again which saves significant time.
- In the final HTML report, each module is a chapter and shown as a separate page. This makes the report more structured.
- Each module is run as a separate `R` session which makes it cleaner and more memory efficient.
- Each module has certain requirements to the input data that must be fulfilled. This allows for independent and parallel development.

## Directory structure

This is a first overview of the important directories and files of the `scrnaseq2` git repository:

- `.gitignore`: Ignore file for git. Set up such that at first all files are ignored and then exceptions are made for the relevant files. This helps with all the temporary files created by `R` and `quarto`.
- `R`: Contains files with user-defined `R` functions. Note that they still contain a number of old and not used functions.
  - `collect_module_results.R`: `R` script that is run at the end of the analysis to collect all results that should be published to the user
  - `functions_analysis.R`: Functions for the analysis of data
  - `functions_degs.R`: Functions for DEG analysis. Outdated and not used
  - `functions_io.R`: Functions for reading and writing data
  - `functions_plotting.R`: Functions to generate common plots
  - `functions_util.R`: Utility functions for general purposes
  - `general_configuration.R`: Contains general settings that are always sourced first
- `README.md`: GitHub repository starting page
- `_quarto-scrnaseq.yml`: Quarto configuration file for the analysis of scRNA-seq data. It contains the modules needed to run for this analysis and its required parameters. Note that we could have similar yet different configuration files for different analyses. 
- `_quarto.yml`: Quarto configurations that are always used (knitr options, formats). Always loaded first and then overwritten by additional configurations.
- `css`: CSS style sheets for HTML.
- `datasets`: Location for datasets to analyze. There are already a number of test datasets (that need to be downloaded first). In addition, there are Excel configuration files for loading the datasets.
- `docs`: Further documentation to be shown on the GitHub page
- `index.qmd`: Starting page for each report. It should contain relevant information about the biological experiment, the reearch questions and TODOs. It also contains brief information about `scrnaseq2` and how to cite the repository.
- `misc`: Contains various files that cannot be put in another directory.
  - `nature.csl`: Citation style format (currently Nature)
  - `references.bib`: Literature information in BibTex format for citing papers
  - `template.docx`: Template for publishing a analysis as Word document
- `modules`: An analysis consists of different parts (e.g. pre-processing or normalization) that are called modules. Please see below for more information.
- `references.qmd`: Quarto document for printing the bibliography of all references used in `scrnaseq2`
- `scripts`: Contains `R` and `quarto` scripts which belong to `scrnaseq2` and use parts it code. However, they are stand-alone/independent, and not part of the main analysis. They are typically run from the command line.
- `scrnaseq.Rproj`: RStudio project settings. User-specific. Ignored by git.

## How modules are chained into an analysis

The first (trivial) possibility is to run each module separately and interactively in RStudio in the desired order.

The intended way however is to define them in an analysis profile (`yml` configuration file). Profiles allow to run `quarto` with different settings. For example, the analysis profile `scrnaseq` is defined in the file `_quarto-scrnaseq.yml`. This file contains the section `chapters`:

```bash
  chapters:
  - "index.qmd"
  - "modules/1_read_data/read_data.qmd"
  - "modules/2_qc_filtering/rna/qc_filtering.qmd"
  - "modules/3_normalization/rna/normalization.qmd"
  - "modules/4_dimensionality_reduction/rna/dimensionality_reduction.qmd"
  - "modules/5_clustering_umap/clustering_umap.qmd"
  - "modules/6_clusterqc/clusterqc.qmd"
  - "modules/7_cluster_annotation/cluster_annotation.qmd"
  - "modules/8_cluster_composition/cluster_composition.qmd"
  - "modules/9_export/export.qmd"
  - "references.qmd"
```

It defines all modules and the order in which they should be run and then included in the report. The modules `index.qmd` and `references.qmd` are always included.

**Important**: During the development it helps to comment out non-relevant parts as then they will not be rendered.

In addition, the `yml` configuration file also contains the analysis parameters:

``` bash
params:
  general:
    # Verbose
    verbose: true
    
    # Project ID
    project_id: "pbmc"
    
    # Species
    species: "homo_sapiens"
    
    # Ensembl version
    ensembl: 98
    
  modules:
    read_data:
      paramB: "E"
```

All parameters in the section `general` will be applied to all modules but can be overwritten by module-specific settings in the section `modules`. The module names are used to assign parameters to modules. Furthermore, all parameters defined here overwrite the parameters in the YAML header of the module documents.

## How an analysis is run

During development, I would suggest to first run each module manually in `RStudio`.

Important is to always be in the project directory (since paths are relative to this directory). Use `Session` -> `Set Working Directory` -> `To Project Directory`. 

To render the document, either click the `Render` button directly, or `Build` -> `Render Book`. In both cases, it will run and render all included modules (chapters) in the order defined in the configuration file. At the end, the report will be in separate subdirectory `scnraseq_book`, and will be opened by RStudio. The word `scrnaseq` here refers to the analysis profile `_quarto_scrnaseq.yml`.

## How module results are reused

There are two levels how module results are reused:

- At the end of the `R` code of a module, the `Seurat` object is stored in the module directory. The next module loads it and in turn at the end saves a copy. This means that an analysis can be run step by step (and not in a full run anymore). It works both interactively (run code line-by-line in `RStudio`) and non-interactively (render via `quarto`).

- When a module is rendered by `quarto`, it will run the `R` code. The result is a markdown file (with text and tables) as well as the plots as PNGs. Quarto then converts these files into the final report. Now quarto can decide whether to run and render a module or reuse the existing content depending on the `freeze` setting. If in a module `freeze` is set to `yes`, then this module will not be run and rendered but its existing content (stored in the directory `_freeze`) will be included. If `freeze` is set to `auto`, then it will only be run and rendered if the module has changed. 

**Important**: Even changing some text will cause this. However, other modules that depend on it are not updated automatically so it might lead to inconsistencies. Best is to run a module, and once you are happy with it, set `freeze` to `yes`. 

**Important:** `freeze`does not apply when a) you run the code interactively chunk by chunk in RStudio or b) explicitly render this document in `RStudio`.

## Core structure of a module

The code for a module is saved in a sub-directory of the `modules` directory. Data-type specific variants of the same module are organized in sub-directories, e.g. `2_preprocessing/rna/` contains the code for pre-processing of RNA data. The actual `quarto` document is named like the module, e.g. `preprocessing.qmd` for the preprocessing module.

The `quarto` document should contain the following core chunks which can be simply copied.

#### Part 1: YAML header

```bash
---
# Module-specific parameters (that are not already in the profile yaml)
params:
  # Name of the module used in configurations
  module: "normalization_rna"
    
  # Relative path to the module directory (which contains the qmd file)
  module_dir: "modules/3_normalization/rna"

  ...
  
# Module execution
execute:
  # Should this module be frozen and never re-rendered?
  # - auto: if the code has not changed (default)
  # - true/false: freeze/re-render
  # Does not apply in interactive mode or when explicitly rendering this document via in rstudio
  freeze: auto
---
```

The YAML header contains **defaults** for module-specific parameters. They can (and should be) overwritten in an external analysis-specific configuration file. The only two parameters that need to be defined are **module** (sets the name for the module) and **module_dir** (sets the directory of the module). These identify the module (how to know what module this is) and are required for all module-specific operations.

The `freeze`setting in the YAML header tells quarto whether to render or freeze the document. If a document is frozen, quarto will always reuse the text and the pictures and will not run the `R` code.

#### Part 2: Setup chunk

The setup chunk is special. When you are in a notebook mode, the chunk named setup will be run automatically once, before any other code is run. Therefore, avoid to do computations here and simply used it for general settings and libraries. Furthermore it is always labelled `setup`.

```r
#| label: setup
#| message: false
#| warning: false
#| eval: false
# Remove the eval setting when using the code

# If running code interactively in rstudio, set profile here
# When rendering with quarto, profile is already set and should not be overwritten
if (nchar(Sys.getenv("QUARTO_PROFILE")) == 0) {Sys.setenv("QUARTO_PROFILE" = "default")}

# Source general configurations (always)
source("R/general_configuration.R")

# Source required R functions
source("R/functions_util.R")
source("R/functions_io.R")
source("R/functions_plotting.R")
source("R/functions_analysis.R")

# Load libraries
library(knitr)
library(magrittr)
library(gt)
library(Seurat)
library(ggplot2)
library(future)

# Get module name and directory (needed to access files within the module directory)
module_name = params$module_name
module_dir = params$module_dir

# Parallelisation plan for all functions that support future
plan(multisession, workers=4, gc=TRUE)
```

- `QUARTO_PROFILE`: When running the code non-interactively (render), there is this environment variable containing the analysis profile name. With this, we know which of the `yml` configuration files to load. For example, for scRNA-seq data, this variable would contain the value `scrnaseq` and therefore we load `_quarto-scrnaseq.yml`. However when running the code interactively in `RStudio` (step-by-step), this is not set and we need to set it manually. 
- Then we source all our functions. In addition, we source general configurations that apply to all `R` code. Have a look at this file - there are also a few fancy colour palettes...
- Then we load packages that need to explicitly loaded. In general, we try to include th package name in the function call though.
- Next we get module name and directory.
- Finally, we configure how parallelization should be done. This applies to all functions that use the `future` framework (which most packages incl `Seurat` do). Four cores is a conservative setting that can be changed depending on function and analysis. 

#### Part 3: Preparation chunk

This chunk loads the input for the module. Furthermore, the input should be checked here. For example, for a module that finds markers, it should be checked whether the input `Seurat` object has clusters.

```r
#| label: normalization_rna_preparation
#| eval: false
# Remove the eval setting when using the code

###############
# Directories #
###############

# Module directory 'results' contains all output that should be provided together with the final report of the project
dir.create(file.path(module_dir, "results"), showWarnings=FALSE)
files = list.files(path=file.path(module_dir, "results"), full.names=TRUE)
if (length(files) > 0) unlink(files, recursive=TRUE)

# Module directory 'sc' contains the final Seurat object
dir.create(file.path(module_dir, "sc"), showWarnings=FALSE)
files = list.files(path=file.path(module_dir, "sc"), full.names=TRUE)
if (length(files) > 0) unlink(files, recursive=TRUE)

#################
# Seurat object #
#################

# Read in seurat object
if (!is.null(param("prev_module_dir"))) {
  prev_module_dir = param("prev_module_dir")
} else {
  prev_module_dir = PreviousModuleDir(module_dir)
}
prev_sc_obj = file.path(prev_module_dir, "sc", "sc.rds")
if(!file.exists(prev_sc_obj)) {
  stop(FormatString("Could not find a sc.rds file in {{prev_module_dir}}. Was the respective module already run?"))
}
sc = SeuratObject::LoadSeuratRds(prev_sc_obj)

# Move on-disk layers to faster temp location if requested
on_disk_counts = param("on_disk_counts")
on_disk_use_tmp = param("on_disk_use_tmp")

if (on_disk_counts & on_disk_use_tmp) {
  on_disk_path = tempdir()
  with_progress({
    sc = UpdateMatrixDirs(sc, dir=on_disk_path)
  }, enable=verbose)
} else {
  on_disk_path = file.path(module_dir, "sc")
}

# Set default assay
default_assay = param("default_assay")
Seurat::DefaultAssay(sc) = default_assay
```

- The first part creates (or cleans) two important sub-directories of the module directory:
  - `results`: Contains all files that should be published to the user (tables). Once a report is generated, an `R` script will copy and organize all files from these directories.
  - `sc`: In this directory, the final `Seurat` object and associated files are saved for the next module.
- Next, it will be determined which of the previous modules should be used as input. This can be set explicitly (`prev_module_dir`) or is determined by a function that reads the yaml configuration file of the current analysis profile.
- Then the `Seurat` object is read into memory. If on-disk counts are used, then the `Seurat` object will contain the paths to the respective directories (which are in the same directory as the `Seurat` object).
- If on-disk counts are used, performance is better if the counts directories are on a local SSD storage than on a standard disk-based server. They will be copied to a temp directory (e.g. `/tmp/Rxsbfhs` ) and the `Seurat` object will be updated. **Problem**: Currently the temp directory is deleted when the `R` session exists or maybe suspends?
- Finally, the default assay (data type) will be set. For all `Seurat` functions, it is then not necessary anymore to explicitly specify the assay.

#### Part 4: Analysis code

Do analysis

#### Part 5: Finish

```r
#| label: normalization_rna_save_seurat
#| eval: false
# Remove the eval setting when using the code

# Save Seurat object and layer data
with_progress({
  SaveSeuratRds_Custom(sc,
              outdir=file.path(module_dir, "sc"))
}, enable=TRUE)

```

- This chunk saves the updated `Seurat` object (and if used its on-disk counts matrices):

```r
#| label: normalization_rna_finish
#| eval: false
# Remove the eval setting when using the code


# Stop multisession workers
plan(sequential)
```

- If parallel workers are used, this will stop them properly.