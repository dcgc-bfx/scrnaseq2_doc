---
title: Installation
pin: true
categories: [Getting started]
tags: [installation]
---

To make our analyses as portable and reproducible as possible, we work with [singularity containers](https://sylabs.io/guides/3.0/user-guide/index.html). 

Our `scrnaseq2` singularity container includes all required software. The container is generated once, and all team members can use it on their system. 

The container version is written into the workflow output. This way, we can later go back to the same container and rely on software versions to reproduce the analysis. 

As containers are big in space, we decided to share our [DcGC singularity recipes](https://github.com/dcgc-bfx/singularity-single-cell). You can download the recipe and build the container on your own.
