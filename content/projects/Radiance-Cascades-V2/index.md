---
title: "V2 - GI using Radiance Cascades"
date: 2025-11-29
draft: false
description: "My second attempt to implement a basic version of Alexander Sannikov's Radiance Cascades Global Illumination technique."
tags: ["blog"]
series_order: 1
showViews: false
showLikes: false
---

## Introduction

**Link to old github code**: https://github.com/Izendi/RC_OpenGL 

Radiance Cascades is a global illumination (GI) technique introduced by Alexander Sannikov from grinding gear games. It has most notably been used in the relatively new video game *Path of Exile 2* to impressive effect.


## The Problem last time

My last attempt had issues with interval length and/or probe density being off, but I'd need to investigate more to nail it down. 
More critically though, I shot myself in the foot with my implementation by making each compute shader dispatch a probe, and each workgroup thread within that dispatch a ray. 

On my current device, a compute shader dispatch can't have more than 1024 threads; as such, I can't have probes with more rays than 1024 without re-designing my entire algorithm. 

This makes it hard to test if there are too few rays per probe at higher cascade levels.

In addition, I was implenting everything from scratch in C++ using the OpenGL API. While great for learning, it meant I spent a lot of time fixing mistakes with my custom rendering implementation rather than fine tuning the specifics of the Radiance Cascades algorithm.

## Attempt 2 - What will be different

For this attempt, I am implementing 2D Radiance Cascades in Godot game engine.
Once I have a working implementation, I will re-implement my solution in my own rendering system using either Vulkan or OpenGL. 

## Step 1 - Setting up the environment

I need to set up godot such that I can have a number of shaders that write to framebuffers and display those framebuffers for analysis quickly at runtime. To do this I will need to set up a simple Godot gui interface that lets me quickly swap between shaders.


**[to be continued...]**

## References
1. <a id="ref1"> Alexander Sannikov, "ExileCon 2023 - Rendering Path of Exile 2," YouTube, Jul. 29, 2023. [Online]. Available: https://www.youtube.com/watch?v=TrHHTQqmAaM&t=2037s. [Accessed: Jan. 14, 2025].</a>
2. <a id="ref2"> C. M. J. Osborne and A. Sannikov, "Radiance Cascades: A Novel High-Resolution Formal Solution for Multidimensional Non-LTE Radiative Transfer," arXiv, Aug. 2024. [Online]. Available: https://arxiv.org/pdf/2408.14425v1. [Accessed: Jan. 14, 2025].</a>
3. <a id="ref3"> SimonDev. "Exploring a New Approach to Realistic Lighting: Radiance Cascades" [Online Video]. Available: [URL](https://www.youtube.com/watch?v=3so7xdZHKxw). Accessed: January 14, 2025.</a>
4. <a id="ref4"> SimonDev. "Featured Courses" [Web Page]. Available: [URL](https://simondev.io/courses). Accessed: February 15, 2025</a>


## Images

![prototype](sample.png)

