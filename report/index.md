---
title: CS184 Project Report
nav_order: 3
mathjax: true
---

# Fog

By Meiqi Sun, Akhil Vemuri, Connor Dang, and Samuel Berkun 

## Abstract
Through this project, we built a volumetric ray tracer that renders the effect of fog onto the scene. The basic raytracer had the assumption that the rays would travel in straight light until it hits the surface of an object. However, when mediums are present, we need to break the assumption: when hitting a fog particle, the ray will also be affected (absorption, scattering, etc). This project supports both homogeneous and bounded-volume heterogeneous fog rendering by applying the techniques of ray-marching, anisotropic scattering, fog shadowing, and mesh-bounded scattering.

## Technical Approach

### Starting Point

We started from the raytracer in CS184's Project 3-1. We then integrated parts of Project 3-2 into it, such as the mirror and glass materials, to get a combined project 3 as our starting point.

The existing raytracer had capabilities to render area and point lights, and several types of materials. It could also render dae files created with Blender, although it did not have the capability to deal with several features present in typical Blender projects (i.e. non-triangular meshes, image textures, spotlights, principled BSDFs). 

### Dae File Generation and new Features

Since we wanted to showcase the abilities of our volumetric renderer, we spent a significant amount of time learning Blender, creating dae files, and implementing new features. 

We first added spotlights to the raytracer implementation, and ran into a significant number of technical challenges while doing so (see [challenges](#challenges)).

When it came time to add volume-bounded fog, we also added a custom BSDF type, a parser for it in `collada.cpp`, and a BSDF implementation in `bsdf.cpp`.


### Initial Implementation

There are a few different ways one could approach rendering volumetric effects like smoke or fog. The most naive way would be to simply scatter billions of tiny sphere meshes throughout the scene, accurately simulating the billions of particles that make up fog or smoke. However, this is a bit inefficient, especially since most computers don't even have the memory required to store all of those sphere meshes.

Instead of capturing billions of ray-sphere intersections, we use a statistical approach operating under the assumptions that the particles are very small, uniformly distributed in space, and that the distance between particles is much larger than the particles itself. By doing this, we can effectively approximate the light-particle interaction for a given media with some notion of an average interaction. Initially, we tried 2 approaches to accomplish this goal.

In our first approach, or the ray marching implementation, we begin by computing an intersection using the BVH. Next, we iteratively traverse the ray starting from the camera in $$t=0.1$$ increments. With $$p=0.015$$, we encounter a fog particle which we use as the basis of our intersection for the rest of the raytracing algorithm. If no particle is encountered, this means the ray intersects the scene. By varying the value $$p$$, we can control the density of the fog.

This implementation was slow and the high quality of the results depended on using small values of t to simulate a continuous distribution the particles. Additionally, we found that this implementation did not fit the structure of our existing path tracer well - would require lots of invasive changes to the overall raytracing pipeline to implement and make efficient. 

In our second approach, we used an analytical method that models the continuous distribution of fog particles via a exponential distribution. This allowed us to compute the potential fog intersection point without having to iterate along the ray path by simply sampling the inverse CDF of the exponential distribution ($$-\log(\epsilon)/ \lambda$$)- giving us better performance and numerical precision for our methods.

We modelled fog density as the exponential parameter $$\lambda$$ and found that values between $$0.1$$ to $$0.3$$ worked well. As in the first approach, we begin by casting a way into the scene to find $$t_i$$ (the time of intersection with the scene). Next, we sample from our exponential distribution to determine a value $$t_f$$ to use as the location of the intersected fog particle along our ray path. If $$t_f > t_i$$, then we compute the radiance from the fog particle and return it. Otherwise, our ray has intersected the scene before the fog particle in which we compute the radiance from the intersection point in the scene.

### Isotropic vs Anisotropic scattering

Several papers we read mentioned that realistic scattering was typically not isotropic, and gave the Henyey-Greenstein function as an easy-to-compute anisotropic distribution that could give more realistic results (see references 1, 2). The Henyey-Greenstein function is given by

 $$\frac{1}{4 \pi} \frac{1 - g ^ 2}{[1 + g ^ 2 - 2g\cos\theta]^\frac{3}{2}}$$ 
 
This provides a configurable phase function when modeling ray scattering after encountering a fog particle. 

We choose this function because it is very flexible and allows us to model different types of scattering. As we vary $$g$$,  we adjust the level of shearing we have for our pdf function. When $$g = 0$$, the function simplifies to $$\frac{1}{4 \pi}$$, which entails an uniform scattering across all directions. The higher the absolute value of $$g$$ is, the more "oval-ly" it spreads along the directions of scattering. And for positive $$g$$, we have more scattering along the opposite direction of the ray; while for negative $$g$$, we would have more scattering along the original direction of the ray. 

Here is a comparison between isotropic scattering, and anisotropic scattering using the Henyey-Greenstein function. The following scene uses three point lights of different colors, to show how the colors interact with the fog:


No fog | Isotropic scattering ($$g = 0$$)
-------- | --------
![](./rea/no-fog.png) | ![](./rea/isotropic.png)

Back-biased scattering ($$g > 0$$) | Forward-biased scattering ($$g < 0$$)
-------- | --------
![](./rea/g0.6.png) | ![](./rea/g-0.6.png)


Both isotropic scattering and forward-biased scattering provide realistic-looking results. Forward-biased scattering tends to create more "glare" from lights pointed toward the camera, which may be desirable depending on the scene. In the above scene, it aids in creating a natural-looking haze around the point lights.

## Improving the performance of Anisotropic Scattering

Our initial implementation of anisotropic scattering used the Harvey-Weinstein function to calculate the reflectance of the fog, but sampled the reflectance direction using a uniform spherical distribution. This created noise at high anisotropy levels (such as $$g = 0.9$$). This solution was to importance sample the Harvey-Weinstien function. Reference 3 describes an analytical solution, as well as several methods that can be used if an analytical solution was not available. 

TODO: mention paper 2 for importance sampling function


TODO: importance sampling H-W function
We applied the inverse CDF approach. We referenced online sources (see reference 3), and proved that the proposed inverse CDF do correspond to the Henyey-Greenstein function as the pdf.

Since it is hard to directly compute the integral of Henyey-Greenstein function, we instead sample from the uniform distribution and use  <br> $$\mu = \frac{1}{2g} (1 + g^2 - (\frac{1 - g^2}{1 - g + 2g\epsilon_1})^2)$$ <br> to get the value of $$\cos(\theta)$$. We can then work out the value of the pdf & $$\sin \theta$$ (trigonometry).
 
We also get another independent sample from the uniform distribution $$\epsilon _2$$ and use <br> $$\phi = 2 \pi \epsilon _2$$ <br> to get the value of $$\phi$$ for transforming to "world" coordinates.

### Fog Shadowing

Note that in the rendering pipeline, direct lighting functions assume that a surface may be directly lit unless it is eclipsed by an object. This assumption still holds mostly true - however, a surface may now be eclipsed by either an object _or fog_.

To simulate an object being eclipsed by fog, our initial approach was to do a statistical test for fog when tracing a direct lighting ray, similar to the below psuedocode:
```cpp
double fog_dist = random_fog_location(ray);
if (!bvh->intersect(ray, isect) && fog_dist > dist_to_light) {
    ...
    L_out += lighting;
}
```
However, this created renders that were slightly grainier than the original pathtracer. To make it converge faster, we analytically solved for the probability that a given ray would be eclipsed by fog. Specifically, fog with density $$\lambda$$ has an exponential distribution, with CDF $$1 - e^{-\lambda t}$$, so if the light is some distance $$t$$ away, the CDF gives the probability that it will hit fog. The updated implementation has the following psuedocode:
```cpp
if (!bvh->intersect(ray, isect)) {
    ...
    L_out += prob_no_fog * lighting;
}
```

Here is what a typical render will look like without and with fog shadowing:

No fog shadowing | Fog shadowing
-------- | --------
![](./cbbunny/no-shadowing.png) | ![](./cbbunny/shadowing.png)

The difference is mainly a decrease in overall brightness; it's most apparent in surfaces further from the light, such as the floor.

### Mesh-bounded Scattering

With our fog implementation complete, we decided to go a step further and attempt mesh-bounded scattering. For mesh-bounded scattering, the fog is enclosed in a mesh and light rays intersecting with the mesh are no longer bounced off of the surface of the volume and instead may pass completely through, intersect with another object inside, or interact with a fog particle.

To implement this, we took inspiration from the glass/mirror materials previous implemented in the initial codebase. With these materials, the BRDF functions simply return a mirrored 

TODO: A 1-2 page summary of your technical approach, techniques used, algorithms implemented, etc. (use references to papers or other resources for further detail). Highlight how your approach varied from the references used (did you implement a subset, or did you change or enhance anything), the unique decisions you made and why.
TODO: A description of problems encountered and how you tackled them.


## Challenges
- Problem: Dae File Spotlight Rendering
    - Since we were working on volumetric rendering, we wanted to add multiple spotlights to the scene so that we can better visualize our results. However, throughout the process, we met multiple challenges. 
        1. First of all, the most recent version of Blender had a different format for the exported dae file, and our initial codebase is unable to render the image. Thus, we tried to install and use Blender 2.7 instead (we also later tried to update `collada.cpp` to enhance the parsing capability). 
        2. Rendering on local machines took a very long time, and would frequently give completely black images (possibly due to the specific OS requirements oforf some modules used in dae file parsing). We initially thought it was a dae file issue, and spent very long time trying to compare and debug the dae file. But it turns out that if we run on the hive machine, the images soon rendered properly.
        3. We tried to move camera positions and angles when creating the dae files, and soon realized that we should instead stick to the default. Otherwise, our rendered image would be targeted at a very wrong direction.
        4. When we added spotlights to the scene, it was initially not recognized by our code. We again thought it was because we had the wrong spotlight settings from Blender (actually most of the light source types work, but only this spotlight isn't showing up properly). We later realized that it was because we had an empty spotlight implementation in `light.cpp`, and that after completing the function and returning the correct radiance, we were able to observe spotlights in our results.




- Problem: Noisy Rendering
    - With our original implementation, the images looked quite noisy: there were many black,  unlit pixels in the scene, and even we sample rates past a certain amount, the results look exactly the same (exact same number of black pixels) even after we increase the sample rate. 
    - We resolved the issue by comparing the sampling rate plot and the rendered image to notice that the black pixels correspond exactly to the parts with little samples (blue on the rate graph). We therefore inferred that the algorithm is tricked into thinking that the pixel value has converged when it just got some black pixels due to chance. We realized that this is because when we get closer to the light source, there's less illuminated pixels relative to the unlit pixels which makes it increasing likely to sample unlit pixels and more likely to get a lot of dark pixels in a row. Hence, the algorithm then "believes" that the value has converged so won't keep sampling the pixel anymore. Figuring out the cause of the issue, we increased the batch size of adaptive sampling, and the scene looked much better. 


- Lessons We Learnt
1. Check the compatibility of the codebase w/ different types of lights before rendering
    It might be a parser issue/implementation issue/light power issue, etc. We should think thoroughly about the possible causes of the issues while debugging. 
2. Pay attention to adaptive sampling
3. Understand the papers and extract (simplify to) the essential parts to build the model for our implementation

## Results

### Fog-like haze

We found that an anisotropy of 0.6 (forward biased) led to a misty-looking haze over the entire scene:
[TODO: renders]

As the scattering density is increased, this haze tends to take on the color of the lights in the scene, "washing out" the other colors in the scene. In particular, note that the colors on the red and blue walls tend to disappear:

[TODO: renders]

This haze tends to create a "glare" as well, especially with more forward-biased anisotropy. This is especially apparent in the above scenes

### Light rays

Light rays are a cool effect, caused by the direct lighting of dust, fog, or mist particles in the path of a light beam. This is especially apparent when rendering scenes with volumetric effects.

[TODO: renders]

As each ray hits an object, it either bounces off or is absorbed, depending on the material. The resulting color and intensity of the ray are determined by the object's surface properties, such as its reflectivity, transparency, and texture. And when the ray intersects a volumetric element like fog, it scatters in different directions, creating the appearance of light rays emanating from the source.


## References

1. Light Transport in Participating Media: https://cs.dartmouth.edu/~wjarosz/publications/dissertation/chapter4.pdf
2. Rendering Participating Media with Bidirectional Path Tracing: http://luthuli.cs.uiuc.edu/~daf/courses/rendering/papers/lafortune96rendering.pdf 
3. On Sampling Of Scattering Phase Functions: https://arxiv.org/pdf/1812.00799.pdf

## Contributions from each team member

 - Everyone:
    - general approach and bug fixing
 - Meiqi Sun:
    - math
    - dae file generation
    - spotlight implementation
 - Akhil Vemuri
    - dae file generation
    - spotlight dae file generation
 - Connor Dang
    - ray marching implementation
    - mesh-bounded volume implementation
 - Samuel Berkun
    - spotlight implementation
    - scattering implementation
    - anisotropic scattering
