---

title: |
  Automated System Health and Performance Benchmarking Platform: High
  Performance Computing Test Harness with Jenkins
type: publication
layout: single
date: 2017
description: |
  Voss, J., Garcia, J. A., Proctor, W. C., &amp; Evans, R. T. (2017). Automated
  System Health and Performance Benchmarking Platform: High Performance
  Computing Test Harness with Jenkins. In Proceedings of the HPC Systems
  Professionals Workshop (pp. 1:1–1:8). New York, NY, USA: ACM.
  https://doi.org/10.1145/3155105.3155106
doi: https://doi.org/10.1145/3155105.3155106
bibtex: /bib/flat17.bib
 
---

Datacenters have a growing need to monitor and maintain complicated computing
machines and verify their systems are functioning at a high level. In order to
achieve this goal, it is critical that administrators are able to readily verify
basic user operations and survey system performance, quickly discerning when a
configuration is sub-optimal. We have created a system health and performance
monitoring tool that enables tracking both health and historical performance.
The tool presents this data visually and enables the identification of realized
and potential problems intuitively. This tool leverages Slurm, a workload
manager common in, yet critical to High Performance Computing workflows. We
construct the tool around Jenkins, a popular and well-supported testing
automation framework which has been used in recent system health and regression
testing, as well as PyTest, an assertion-driven Python unit test framework,
after evaluating several potential automation tools and testing frameworks. This
project develops a test harness for the Texas Advanced Computing Center to run
multiple extendable suites of benchmarking and system health applications
demonstrated on the Stampede2 and Lonestar5 HPC systems. The applications chosen
to run within the test harness include existing in-house benchmarks, such as the
Performance Assessment Workbench, and community benchmarks, e.g. STREAM, in
addition to newly created system health monitoring scripts.
