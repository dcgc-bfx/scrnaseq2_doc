---
title: Setting up the environment
pin: true
categories: [Getting started]
tags: [installation]
---

# Required software

To make our analyses as portable and reproducible as possible, we work with [singularity containers](https://sylabs.io/guides/3.0/user-guide/index.html). 

Our `scrnaseq2` singularity container includes all required software. The container is generated once, and all team members can use it on their system. 

The container version is written into the workflow output. This way, we can later go back to the same container and rely on software versions to reproduce the analysis. 

As containers are big in space, we decided to share our [DcGC singularity recipes](https://github.com/dcgc-bfx/singularity-single-cell). You can download the recipe and build the container on your own.

# Download the scrnaseq2 code

## Option 1: Fork the `scrnaseq2` repository

## Option 2: Clone the `scrnaseq2` repository

```
# Clone git
git clone git@github.com:dcgc-bfx/scrnaseq2.git
```

# Run `scrnaseq2`

## Option 1: Run with `RStudio`

Start RStudio and associate a new project with this repository at *File* -\> *New* *Project* -\> *Existing* *Directory* and browse to the path of the git repository.

## Option 2: Run from command line