---
title: How to run `scrnaseq2`
pin: true
categories: [Getting started]
tags: [execute]
---

## Setup

``` bash
# run within git directory and create rstudio-server/run and rstudio-server/lib
# change port
singularity run -B /projects,/group -B $(pwd)/rstudio-server/run:/var/run/rstudio-server -B $(pwd)/rstudio-server/lib:/var/lib/rstudio-server --writable-tmpfs  --app rserver /group/dcgc/sequencing_processed/support/pipeline/singularity/singularity-single-cell_1.5
.4.sif 8789
```
