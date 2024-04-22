---
title: "Forward and Deferred Rendering/Shading"
date: 2024-04-22
draft: false
description: "An overview of the two main types of rendering commonly used in computer graphics"
tags: [ "blog"]
series_order: 1
showViews: false
showLikes: false
---

{{< lead >}}
An overview of the two main types of rendering commonly used in computer graphics
{{< /lead >}}

## Two types of rendering

Anyone who's even done the briefest of experiments with a graphics API will likely be familiar with forward rendering. It is (for lack of a better term) the "default" form of rendering.<br><br>
It is characterized by having each individual draw call simultaneously do all the lighting calculation relevant for that draw call's geometry.

Conversely, deferred rendering is when each draw call only draws the geometry and the fragment shader stage does little to no work. Instead the data about each fragment is stored in something called the G-Buffer.
<br>
Then once all geometry is drawn, the lighting calculations are done per pixel on the screen by sampling relevant information about each pixel from the G-Buffer.
<br><br>
This allows us to avoid doing complex lighting calculations for overdrawn geometry fragments (as is often the case in simplistic forward rendering implementations) at the expense of needing to manage a large G-Buffer which can cause significant memory bottlenecks



## Step 1
Testing reference to unity documentation on Forward Rendering [<a href="#ref1">1</a>\].


## References
1. <a id="ref1"> Unity Technologies, "Forward rendering," Unity Documentation, 2023. [Online]. Available: https://docs.unity3d.com/Manual/RenderTech-ForwardRendering.html. [Accessed: Apr. 22, 2024].</a>
