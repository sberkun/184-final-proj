---
title: CS184 Project Proposal
nav_order: 1
---


# Final Project Proposal

## Rendering Volumetric Scattering

Meiqi Sun, Akhil Vemuri, Connor Dang, Samuel Berkun  

## Summary

We would like to render different volumetric effects, such as water, dust, fog, and smoke. Smoke will be particularly interesting to implement because it has varying density, which will scatter light with different intensities. 

## Problem Description 
### Trying to Solve
Our current surface renderer is limited by the assumption that light only scatters when it hits a surface. And thus, we are unable to render images under different mediums. Hence, we hope to enable light scattering before it hits any surface, i.e. medium can also scatter the lights.

### Why is it important
This is important in practice because we want to render photos that are more realistic looking. For example, under different weather conditions, the scene we observe is different even if we trace the same amount of lights to the exact same object. On foggy days, the scene might be blurry, while on sunny days, the scene may look much brighter. Hence, to model different medium properties and effects, we need to enable light scattering via medium.

### Why is it challenging
In the real world, light scattering occurs because of intersections between light and microscopic particles that exist in the space between objects. In theory, the desired effect can be achieved by filling the space between objects with tiny particles and using the class-implemented ray tracer. However, this is computationally expensive (and pretty much infeasible) because of the number of potential intersections that would need to be tested for a given ray. The challenge here lies in achieving an identical effect without needing to compute an extreme number of these microscopic ray-particle intersections.


## Goals and Deliverables
Many real-time rendering applications like games or interactive simulations seek to incorporate atmospheric effects such as mist, fog, and haze. As such, we hope to present an alternative physically-based approach that captures these effects while maintaining realtime performance.

Our baseline goal is to create multiple renders using volumetric scattering across various mediums, including homogenous / heterogenous and refractive volumes. Using our surface rendering implementation in project 3, we should be able to extend our project with relative ease as we have an already established benchmark to work from.

### Benchmarking our System
 - Rendered Effects: We want to be able to see light rays in the scene. We should be able to observe differences in light absorption/reflection between renders that use different mediums (smoke vs. fog). The resulting renders containing smoke, fog, etc. should be relatively realistic.
 - Rendering Time: The simplest metric to benchmark the runtime performance of the system.


### What we plan to deliver
- Renders with homogeneous volumes like dust/fog: Vary the intensity of the medium and compare against different intensities
- Renders with liquids (which have both a refractive surface, and a volumetric effect internally)


### What we hope to deliver
In addition to the goals we outlined above, we may explore the following:

 - Heterogeneous Materials: Try rendering smoke effect, varying the density according to distance to "centers"
 - Optimization: We would like the pathtracer to perform on-par (if not better) than the baseline pathtracer in project 3-2. Ideally, volumetric effects should not significantly impact render times.
 - Animation: We would like to extend the path tracer to render animations, specifically smoke simulations.
 - Bidirectional Path Tracing: This may reduce noise in scenes involving many bounces. For example, if there is both fog and reflections, bidirectional path tracing may be very valuable. However, this goal is extremely ambitious, as implementing bidirectional path tracing will require rewriting much of the path tracing logic.


## Schedule

Week 0 (4/3 - 4/9):
 - Read the literature and derive the formulas/develop the background needed
 - Build the basic codebase framework (Project 3-1)

Week 1 (4/10 - 4/16):
 - Implement volumetric scattering for homogenous volumes
 - Render a dusty/foggy room with simple setup/scene (similar to the example renders)

Week 2 (4/17-4/23):
 - Implement volumetric scattering for refractive volumes (i.e. water, other liquids)
 - Implement volumetric scattering for heterogenous volumes (i.e. simple smoke). Render some simple smoke.

Week 3 (4/24-4/30):
 - Experiment and potentially model different mediums
 - Experiment with animations / simulations
 - Optimization: Speed up the rendering process
 - Start writing final project report

Week 4 (5/1-5/4):
 - Finish writing final project report
 - Record project videos/Demos
 - Relax if we finish early (we won't but still)


## Resources

Papers / Online Resources:
 - [http://luthuli.cs.uiuc.edu/~daf/courses/rendering/papers/lafortune96rendering.pdf](http://luthuli.cs.uiuc.edu/~daf/courses/rendering/papers/lafortune96rendering.pdf)
 - [https://cseweb.ucsd.edu/~ravir/papers/singlescat/scattering.pdf](https://cseweb.ucsd.edu/~ravir/papers/singlescat/scattering.pdf)
 - [http://www.cs.cmu.edu/~ILIM/projects/LT/multiple_scattering/ptping_media.html](https://cseweb.ucsd.edu/~ravir/papers/singlescat/scattering.pdf)
 - [https://www.cs.rpi.edu/~cutler/classes/advancedgraphics/S07/final_projects/fischc/fog_simulation.html](https://www.cs.rpi.edu/~cutler/classes/advancedgraphics/S07/final_projects/fischc/fog_simulation.html)
 - [https://cseweb.ucsd.edu/classes/sp17/cse168-a/CSE168_14_Volumetric.pdf](https://cseweb.ucsd.edu/classes/sp17/cse168-a/CSE168_14_Volumetric.pdf)


Computing resources
- Four (4) laptops (one from each team member).
- We may use Blender to create smoke simulations, which we will then try to render using our renderer.

Existing code:
 - Project 3 pathtracer code

