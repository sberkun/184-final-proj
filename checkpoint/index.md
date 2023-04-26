---
title: CS184 Project Checkpoint
nav_order: 2
mathjax: true
---

# Final Project Checkpoint


# Rendering Volumetric Scattering

Meiqi Sun, Akhil Vemuri, Connor Dang, Samuel Berkun


# Accomplishments and Results

## Absorption And Scattering
Absorption: 
1. We get a random sample of $$t$$ from the exponential distribution with rate 0.1. The rate is a proxy for the density of the fog, i.e. with a higher rate, we are expected to hit a particle at a shorter $$t$$, and therefore, indicating the fog is denser. However, with a lower rate, we hit the particle at a larger $$t$$, which means the fog is sparser. We use this to compute the intersection point between the ray and fog particle. If the point of intersection is behind the scene ($$t$$ value of ray-fog intersection > $$ t$$ value of ray-object intersection), then we disregard the particle, and just perform regular ray tracing. However, if the compute $$t$$ value is less than the first ray-object intersection point, that means our ray first bounced on the fog particle.
2. When we determine the location of the particle, we perform fog-particle intersection and apply a scattering effect using the phasor scattering function. Specifically, we used the analytical phasor function $$p(\theta) = \frac{1}{4 \pi} \frac{1 - g ^ 2}{[1 + g ^ 2 - 2g\cos\theta] ^ \frac{3}{2}}$$ as the pdf for $$\theta$$, and applied importance sampling on the "oval". We vary the value of $$g$$ to adjust the level of shearing we have for our pdf function. The higher the value of g, the more "oval-ly" the directions of scattering, and the more scattering we have along the opposite direction of the ray. The value of $$g$$ we used for our rendering for best effects is 0.6.



# Problems to be solved:
1. Adding spotlight source to the scene using blender and export the dae file so that the volumetric effects are more obvious (currently, we are unable to export any dae file that successfully include the effect of a spot light source)
    * Currently, we are using an area light source to evenly distribute light across the scene. This creates a uniform lighting pattern that doesn't necessarily emphasize any particular direction. In contrast, directional lighting (spot lighting) emits light in a specific direction, which can help create distinct shadows and highlight depth within the scene.
    * The sample scene in the photo below helps to illustrate the effect we are looking for:

        ![alt text](dragon_goal.png "Directional Lighting")

2. Apply absorption to the scene, such that a good balance exists between scattering and absorbing. We haven't added the effect yet, because we want to derive the math so it is coherent. 
    * Naive approach: Conduct two consecutive coin tosses, and use that information to determine: if the first coin turns head  - don't absorb any light; if the first coin is tail + second coin is head - absorb a certain portion of the light; if both coins are tails - absorb all the light. Consequence: the image seemed too dark, and weird black spots exist in the scene. Should take exponential into consideration, consider a more smooth absorption. 

3. Consider a heterogeneous fog rendering (possibly). 



# Video and Presentation

[TODO: link to video]()
[Presentation](https://docs.google.com/presentation/d/e/2PACX-1vR3yRybiOHc-NpsB6VhsimPqGPOrPzHQoE5ZCKoerq5uKw18zFb7ZVRjgXRkEJnYtf6GFyZWRwIJ2Ur/pub?start=false&loop=false&delayms=3000)
