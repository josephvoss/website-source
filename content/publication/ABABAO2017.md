---

title: |
  Student Cluster Competition 2016 Reproducibility Challenge: Genomic
  partitioning with ParConnect
type: publication
layout: single
date: 2017
description: |
  Ababao, R., Garcia, J. A., Voss, J., Proctor, W. C., & Evans, R. T.
  (2017). Student Cluster Competition 2016 Reproducibility Challenge: Genomic
  partitioning with ParConnect. Parallel Computing. https://doi.org/10.1016/j.parco.2017.07.002
doi: https://doi.org/10.1016/j.parco.2017.07.002
bibtex: /bib/ababao2017.bib
 
---

As part of a reproducibility initiative from the Student Cluster Competition
2016, specific results and trends presented in “A parallel connectivity
algorithm for de Bruijn graphs in metagenomic applications” were similarly
produced and verified. The general lack of reproducibility within the scientific
community is a known issue, but few have the time, resources, or incentives to
fully address it. Motivation for reproducibility resides in the need to
independently validate previous research claims and test the difficulty or ease
with which these claims may be reasserted. This fundamental tenant becomes ever
more important, particularly due the prohibitive simulation cost and data
complexity currently associated with metagenomics. The algorithms in the
aforementioned article provide a scalable, distributed memory solution to the
problem of assembling and labeling connected components in graphs associated
with metagenomic samples. We aim to verify four of the components demonstrated
by this article; namely, the deterministic countability of connected components
in the data sets used, the computation to communication ratio of different
work-balanced parallel algorithm implementations, the results obtained from said
algorithm implementations, and the scaling behavior of the algorithms as the
number of MPI processes are increased.
