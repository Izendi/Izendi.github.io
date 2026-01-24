---
title: "GI using Radiance Cascades"
date: 2025-12-27
draft: false
description: "My attempt to implement a basic version of Alexander Sannikov's Radiance Cascades Global Illumination technique."
tags: ["project","blog", "Play In Browser"]
series_order: 8
showViews: false
showLikes: false
---
## Try my current version 
[Radiance Cascades Demo](https://izendi.github.io/rc_demo/radiance_cascades_v1_demo.html)

**CONTROLS:** Hold the **right mouse button** to draw segments, Use the **left mouse button** to select menu options.

***MENU*:** Top left slider controls segment thickness. "Toggle length Slider" button reveals a menu where you can adjust the ray march distance "IL = Interval Length" and start distance "SD = Start Distance" by adjusting sliders

 

## Introduction

**Link to github code**: [https://github.com/Izendi/radiance_cascades_V2](https://github.com/Izendi/radiance_cascades_V2)

Radiance Cascades is a global illumination (GI) technique introduced by Alexander Sannikov from grinding gear games. It has most notably been used in the relatively new video game *Path of Exile 2* to impressive effect.

## Cascade Levels being merged

The images below shows each cascade level being merged into the final image. (With top left being Cascade 4, then going left to right, cascade 3, cascade 2, cascade 1, cascade 0 and the final image in the bottom right)

![Features scene](RC_All_Levels.png)

## Example Scenes

![Hi There Scene](RC_Writing_2.png)

![Lightshow Scene](featured_2.png)

## The Problem Being Solved
Radiance cascades aims to be a real time solution to the **spatial frequency problem** in global illumination.

Spatial frequency is how quickly a signal (lighting in this case) changes over space.

**High spatial frequency** is when lighting changes fast over a short distance.

**Low spatial frequency** is when lighting changes slowly and smoothly over large areas.

Look at the image below taken from Alexander Sannikov's radiance cascade paper. We have a light source casing light downwards being partially occluded by a horizontal segment. This creates a penumbra (an area not completly in shadow that is partially lit).
We can see that if we sample along segment A, the lighting changes very quickly as we move through space. But, if we sample along segment B, the lighting changes slowly as we move through space.

![Penumbra No Markings](Penumbra.png)

![Penumbra With Markings](Penumbra_2.png)

The main takeaway from this is that the lighting contribution from far away objects changes slowly as we move though space but the contribution from nearby objects changes rapidly as we move through space.

Additionally, let's think about this in terms of lighting probes that cast out rays.
If a probe is nearby an object, many of it's rays have a very high chance of colliding with the object. But as we move further away from the object, fewer rays will collide with the object. (See the two images below for an example).

![Nearby Probe](Nearby_Probe.png)

![Penumbra With Markings](Far_Probe.png)

We can use these **two rules**:

- **RULE 1**: lighting contribution from far away objects changes slowly as we move though space but the contribution from nearby objects changes rapidly as we move through space.

- **RULE 2**: Probes nearby object have a high ray collision chance but as they move further away that collision chance drops.

To form the basis for radiance cascades.

## The Radiance Cascades Implementation

Using **RULE 1** and **RULE 2** we can define a specific probe layout to take advantage of these rules.

This probe layout is the foundation of radiance cascades.

As we move away from object, we need more rays to capture their lighting contribution but we can worry less about the spacing of those probes, that is to say, a probe far away from a light probe can be shifted around with minimal changes to the total light gather by the probe, but a probe nearby the light with rapidly change the light being gathered with small movements through space.

This is where radiance cascades uses the idea of probe hierarchies, also called cascade hierarchies.

Cascade 0, is made up of many probes very close together, but each probe only casts a small number of rays (maybe 4 rays, in the top right, top left, bottom left and bottom right directions).
The rays can't travel very far from their probe and they are intended to gather nearby lighting data.
Above cascade 0 we have cascade level 1. This cascade level has fewer probes than cascade level 0, with each probe being space further apart from each other, but each probe casts more rays. (maybe 16 rays, 4 in each top right, top left, and bottom left and bottom right directions).

This pattern continues until the highest cascade which has the fewest probes (with large distances between the probes), but each probe casts a large number of rays very far into the scene and the rays start position is offset greatly from the probes center.

The diagram below explains the idea:
- BLUE = Cascade 0
- ORANGE = Cascade 1
- GREEN = Cascade 2
- RED = Cascade 3
- PURPLE = Cascade 4

![RC Paper EX](RC_Paper_Ex.png)

We can see in the above diagram (taken from the original Radiance Cascades paper [<a href="#ref2">2</a>\]), that as we go up a cascade level the number of probes decreases but the number of rays that each probe emits increases (in this example by a factor of 2 per level).

Additionally, each ray starts where the previous cascade ray ended. So if cascade 0 has a start distance offset of 0 and a max ray march distance of 2, cascade 1's rays would start 2 pixels away from the probe. If cascade 1 then marched 10 pixels, cascade 2's rays would start 12 pixels offset from the probes center.
This is done so that probes in higher cascade levels, only capture lighting data from far away (as intended) and do not capture nearby data (which is the job of the lower cascade levels).

You can mess around with these values in the demo by selecting the "*Toggle length sliders*" button and messing with the ray interval lengths and start offset lengths.

All of this ties back in to the idea of higher cascade levels being responsible for **high spatial frequency** sampling and low cascade levels being responsible for low **spatial frequency sampling**. 

Remember high spatial frequency is when light changes fast over a short distance (this is what low level probes handle as there are a large number of them and they only cast a small number of short distance rats).
Whereas low spatial frequency, which is when light changes slowly over large areas, is handled by high level probes as they cast lots of long distance rays at a large offset from the probes center, to capture far away object lighting contributions. When far away from an object, moving though space only changes the lighting contribution slowly, so these probes can be spaced far apart.

Overall, this allows the number of rays in each cascade level to remain the same (grow linearly), which each cascade level handles a different spatial frequency.

## Current implementation details

This implementation aimed to fix the problems encountered in my old attempt: [OLD Radiance Cascades Implementation](../../samples/radiance-cascades-old/).

The old version had too few rays per cascade level, as such, I aimed to implement a large number of rays for this attempt and compare the difference.

In this version, each cascade level 1048576 rays (just over a million). Having each cascade level have the same number of rays meant I could store each in a 1024 by 1024 resolution texture.

The probe to ray ratio is as follows:

- Cascade 4 has: (1024 probes and each probe casts 1024 rays)

- Cascade 3 has: (4096 probes and each probe casts 256 rays)

- Cascade 2 has: (16384 probes and each probe casts 64 rays)

- Cascade 1 has: (65536 probes and each probe casts 16 rays)

- Cascade 0 has: (262144 probes and each probe casts 4 rays)

Each ray is ray marched through the scene until it hits an SDF or reaches it's maximum length limit.

If a ray hits an SDF it stored the color data of that SDF, if it hits nothing, it finds the 4 nearest probes to the current emitting probe in cascade layer N+1 (unless we are the highest cascade level already), then it finds the 4 rays with the most similar angle from each of the 4 probes, averages the color from each then bilinearly interpolates between the 4 color values and stores that in Cascade N's texture data for that ray.

So, starting at cascade 4 we merge downwards until we reach cascade 0, then we have a 512 by 512 resolution output buffer which samples the cascade 0 texture (cascade 0 now contains all combined data for all cascade levels), and uses the built in texture sampling bilinear interpolation to generate a smooth final output (which is what you see in all the example images).

the image below shows the texture output for each cascade level from left to right, with the top left being cascade 4 and the bottom right being the final output sampling from cascade 0.

![Features scene](RC_All_Levels.png)

You can mess around with this yourself in the demo by switching the display buffer with the top left selector drop down menu.

## References
1. <a id="ref1"> Alexander Sannikov, "ExileCon 2023 - Rendering Path of Exile 2," YouTube, Jul. 29, 2023. [Online]. Available: https://www.youtube.com/watch?v=TrHHTQqmAaM&t=2037s. [Accessed: Jan. 14, 2025].</a>
2. <a id="ref2"> C. M. J. Osborne and A. Sannikov, "Radiance Cascades: A Novel High-Resolution Formal Solution for Multidimensional Non-LTE Radiative Transfer," arXiv, Aug. 2024. [Online]. Available: https://arxiv.org/pdf/2408.14425v1. [Accessed: Jan. 14, 2025].</a>
3. <a id="ref3"> SimonDev. "Exploring a New Approach to Realistic Lighting: Radiance Cascades" [Online Video]. Available: [URL](https://www.youtube.com/watch?v=3so7xdZHKxw). Accessed: January 14, 2025.</a>
4. <a id="ref4"> SimonDev. "Featured Courses" [Web Page]. Available: [URL](https://simondev.io/courses). Accessed: February 15, 2025</a>

