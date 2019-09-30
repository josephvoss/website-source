---

title: Fluid Flow Simulation
date: 2016-05-01T00:00:00-06:00
draft: false
layout: posts
tags: 
  - high-performance-computing
  - academic
  - software
description: |
 April to May 2016:
 Navier-Stokes solver in C++ using MPI on TACC systems

---

<img class="img-responsive center_img" style="max-width: 75%" src="/img/solvedFlow2.png">
<div align="center">Solved velocity and pressure for flow through a 2
dimensional channel</div>
<br>



For the final project in my Parallel Programming class we were tasked with solving 
some problem using MPI on the HPC systems at Texas Advanced Computing Center (TACC). 
I was taking Fluid Mechanics at the same time, and chose to replicate one of the labs 
we had completed about the development of laminar flow. With the MPI C++ bindings, I 
wrote a basic simulation which depicted the flow of fluid through a two dimensional 
channel in the terms of pressure and velocity of the fluid elements in a mesh. 
This data was then exported to python to generate a matplotlib animation which shows 
how successive iterations changed the result of the simulation.

[Link to report](/pdf/FluidFlowWriteup.pdf).

[Link to github repo](https://github.com/josephvoss/FluidFlow).
