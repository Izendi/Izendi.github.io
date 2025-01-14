---
title: "GI using Radiance Cascades"
date: 2024-12-27
draft: false
description: "My attempt to implement a basic version of Alexander Sannikov's Radiance Cascades Global Illumination technique."
tags: ["blog"]
series_order: 8
showViews: false
showLikes: false
---

## Introduction

Radiance Cascades is a global illumination (GI) technique introduced by Alexander Sannikov from grinding gear games. It has most notably been used in the relatively new video game *Path of Exile 2* to impressive effect.

## The Problem Being Solved
Traditional methods of generating accurate global illumination, such as Ray and Path tracing are limited by the high performance cost that grows with the number of rays being cast.
At best, with basic ray tracing, increasing the number of rays cast per pixel has a linear growth cost. But when we look into more advanced techniques such as path tracing, which spawn multiple rays
per ray surface bounce, we begin to look at exponential computational cost.

Many methods exist to improve pathtracing, such as supporting it via hardware acceleration, only sampling the most important ray bounce direction, or reducing the overall number of rays to create a noisy image and
then cleaning it up using generative AI techniques (such as NVIDEA's DLSS solution).

Despite these methods making real time global illumination more accessible than ever, the fundamental limitation of runway computational cost is still present and developers are always working around this limitation.

**Radiance cascades** however offers an alternate approach for approximating global illumination. It was inspired by radiance probes, a technique that also aims to approximate global illumination more quickly.
Radiance probes work by placing probes around your scene, these probes cast out rays and gather radiance information from the scene. Then each fragment making up your scene, samples from a set of nearby probes to get an approximation of the global illumination.
Probes have been used effectivly, but the questions such as how many probes to use or how many rays each probe should cast remain and the cost is still growing at least linearly as the number of probes increase.

## How Radiance Cascades Solve This Problem
Radiance cascades attempts to solve these problems in two ways. First by explicitly structuring how probes are placed in the scene and secondly defining hard rules for how many rays are cast for each probe, which is determined by their placement in a hierarchy.

Probes are split into "cascade hierarchies/levels" as Sannikov calls them. The first  level (cascade 0) has the highest number of probes. Let's say n (let's say n = 32), but only casts
x number of rays (for simplicity let's say x = 4). Total number of rays for cascade level 0 is 32*4 = 128.
Then cascade level 1 will have n/4 number of probes (32/4 = 8), but each probe will cast 2*x ( or 2*4 = 8 in this case ) rays. 8 * 8 = 64 total rays.
Then cascade level 2 will have (n/4)/4 number of probes ( (32/4)/4 = 2 ), but each probe will cast 2*(2*x) ( or 2*(2*4) = 16 ) rays. 2 * 16 = 32 total rays.

If we look at the sequence of rays being added with each level: 128, 64, 32. We can see the total number of rays going down with each level.
For each cascade level above the current one, we can bilinearly interpolate between the 4 nearest probes to generate probe data for that lower level cascade. This means we have ray data equal to the sum of the number of rays cast on each level for each level 0 probe.
For instance a single probe on cascade level 0 will have access to it's 4 original rays + 24 from interpolation (8 from level 1 and 16 from level 2). (*MORE REFINED DETAIL NEEDED HERE*)

In the creators own words, it allows "the total cost of calculating these cascades to approach a constant." [<a href="#ref1">1</a>\]
Each level you cast less rays, but through interpolation and averaging each level 0 probe has access to data that represents casting 28 rays of varying lengths.
You can increase the number of probes at the base level or even the number of rays, the total cost will increase, but it will approach and remain a constant.

In general terms, we distribute levels of cascade probes from level 0 up to level n. Each level changes the "resolution" of it's linear and angular component. [<a href="#ref2">2</a>\]
What this basically means is that there are lots of probes laid out at level 0, potentially hundreds or thousands, uniformly laid out in a grid. The lowest layers have lots of probes but very few rays (high linear resolution), as you increase the level of the cascade the linear resolution drops (number of probes), and the angular resolution (number of rays) increases. But since the rays double with every level and the probes decrease by a quarter every level, the total number of rays in each cascade level is half the previous level.

In Sannikov's paper he shows that there is a relationship between this angular and linear resolution using the penumbra effect of a light. [<a href="#ref2">2</a>\]
A video by SimonDev on youtube sums it up nicely (link in reference), he says "There is a relationship how far you are from an object, how big the object is and how closely you need to take samples together." [<a href="#ref3">3</a>\] (*Timestamp* **8:33** *onwards*)
(*MORE DETAIL NEEDED HERE*)

## Intro to my first implementation
I first wanted to implement this using Vulcan, but due to it's high initial learning curce I switched to using OpenGL which I am more familiar with. I will attempt a version using Vulcan at a later date.

Radiance cascades can be complex, especially in 3D, as such, my fisrt attempt will be a simple 2D implementation using signed distance functions and ray marching.

I first needed to obtain an intuition for how they worked. This was done by reading Sannikov's paper, SimonDev's Youtube video and asking for help on the Radiance Cascades and Graphics Programming Discords. [<a href="#ref1">1</a>\] [<a href="#ref2">2</a>\]

## Step 1: Setting up the environment

For the basic 2D implementation, all I needed was a fullscreen quad to render shader output. 
To facilitate this, I created a new OpenGL template that would allow me to quickly set up, test and cycle through new shaders.
All of which are rendered to the full screen quad. 
I used I'm gui to modify input parameters and added a button to recompile shaders at runtime. 

(*ADD IMAGE HERE*)

## Step 2: Testing basic shader functionality

The


## Step 3: Defining the probes and how they will store radiance data

The


## References
1. <a id="ref1"> Alexander Sannikov, "ExileCon 2023 - Rendering Path of Exile 2," YouTube, Jul. 29, 2023. [Online]. Available: https://www.youtube.com/watch?v=TrHHTQqmAaM&t=2037s. [Accessed: Jan. 14, 2025].</a>
2. <a id="ref2"> C. M. J. Osborne and A. Sannikov, "Radiance Cascades: A Novel High-Resolution Formal Solution for Multidimensional Non-LTE Radiative Transfer," arXiv, Aug. 2024. [Online]. Available: https://arxiv.org/pdf/2408.14425v1. [Accessed: Jan. 14, 2025].</a>
3. <a id="ref3"> SimonDev. "Exploring a New Approach to Realistic Lighting: Radiance Cascades" [Online Video]. Available: [URL](https://www.youtube.com/watch?v=3so7xdZHKxw). Accessed: January 14, 2025.</a>



