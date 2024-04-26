---
title: "Forward and Deferred Rendering/Shading"
date: 2024-04-22
draft: false
description: "An overview of the two main types of rendering commonly used in computer graphics"
tags: ["blog"]
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

Conversely, deferred rendering splits the rendering process into two rendering passes:
- A geometry Pass (Which renders all geometry without doing any lighting calculations)
- Lighting pass, (which iterates over each pixel in the final image to perform lighting calculations)
  
<br><br>
Hence the name "deferred shading," we defer all lighting calculations to a later stage.
<br><br>
Deferred shading works thanks to something called the **G-Buffer**. In the geometry pass, each fragment shader invocation, instead of doing lighting calculations, sends all data about each fragment (that will later be required to calculate the lighting) to the **G-Buffer**.
<br>
The **G-Buffer** is actually a collection of buffers, such as one for each fragments depth value, surface normals, color values, etc. 



## Tests
Testing reference to unity documentation on Forward Rendering [<a href="#ref1">1</a>\].
Testing reference to youtube video [<a href="#ref2">2</a>\].


## References
1. <a id="ref1"> Unity Technologies, "Forward rendering," Unity Documentation, 2023. [Online]. Available: https://docs.unity3d.com/Manual/RenderTech-ForwardRendering.html. [Accessed: Apr. 22, 2024].</a>
2. <a id="ref2"> Cambridge Computer Science Talks, 2023 "Forward and Deferred Rendering," Online video clip, YouTube, Available: <https://www.youtube.com/watch?v=n5OiqJP2f7w\>. [Accessed on: Apr. 26, 2024].</a>

