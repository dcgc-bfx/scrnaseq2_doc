---
title: Setting up the environment
author: ktrns
pin: true
categories: [Getting started]
tags: [installation, software, rstudio]
---

## Step 1: Setting up the required software

### Option 1: Install required software

To use our workflow, you will need to install `Rstudio` and all required `R` packages. 

### Option 2: Use our software Singularity container (recommended for Linux users):

To make our analyses as portable and reproducible as possible, we work with [singularity containers](https://sylabs.io/guides/3.0/user-guide/index.html). 

Our `scrnaseq2` singularity container includes all required software. The container is generated once, and all team members can use it on their system. 

The container version is written into the workflow output. This way, we can later go back to the same container and rely on software versions to reproduce the analysis. 

As containers are big in space, we decided to share our [DcGC singularity recipes](https://gitlab.hrz.tu-chemnitz.de/dcgc-bfx/singularity/singularity-single-cell). You can download the recipe and build the container on your own.

## Step 2: Setting up the scrnaseq2 code

### Option 1: Clone the scrnaseq2 repository

The easiest way to get started is to clone our `scrnaseq2` GitHub repository to your local storage. 

### Option 2: Fork the scrnaseq2 repository

If you intend to contribute to the workflow, please first create your own fork of the `scrnaseq2` GitHub repository. You can then clone your own fork, work with it, and once your code is finalized and working, you can create a pull request.

## Step 3: Open the scrnaseq2 code in RStudio

### Option 1: RStudio (if you installed all software yourself):

Start `RStudio`.

### Option 2: RStudio Server (if you use our software container, Linux users):

Start `RStudio Server`:

```bash
# Define your locations
DIR_REPO=...
FILE_CONTAINER=...

# Navigate to your local scrnaseq2 repository
cd $DIR_REPO

# Create these two directories that are required for RStudio Server
mkdir -p $DIR_REPO/rstudio-server/run $DIR_REPO/rstudio-server/lib

# Start RStudio Server
singularity run -B $DIR_REPO/rstudio-server/run:/var/run/rstudio-server -B $DIR_REPO/rstudio-server/lib:/var/lib/rstudio-server --writable-tmpfs --env RSTUDIO_USER=$(whoami) --env RSTUDIO_PASSWORD=notsafe --env RSTUDIO_PORT=1000 --contain --app rserver $FILE_CONTAINER
```

## Step 4: Run the scrnaseq2 workflow

### Set up a new project in RStudio
 
You should start your analysis by setting up an RStudio project. You can do this within Rstudio at `File` -> `New Project` -> `Existing Directory`, and browse to the path of the git repository.

The RStudio project helps with organizing the files but does not change the git repository itself. Moreover, it allows for user-specific settings (see `Tools` -> `Project Options`). All RStudio project settings are ignored by git.

### Download test data

To get started with the `scrnaseq2` workflow, it is best if you try it once with a test dataset we provide.

To do so, open the file `datasets/10x_pbmc_5k_protein/download.R` in `RStudio`. Then change the working directory with `Session` -> `Set Working Directory` -> `To Source File Directory`. Run the code and files should appear in this directory. Repeat for `datasets/10x_pbmc_1k_healthyDonor_v3Chemistry/download.R`. Once done, change working directory back to project directory.

### First run

Try to run the workflow at `Build` -> `Render Book`. 

Did it work? :partying_face: