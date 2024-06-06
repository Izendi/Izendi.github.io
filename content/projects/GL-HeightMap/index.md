---
title: "OpenGl HeightMap with Phong Shading"
date: 2023-11-25
draft: false
description: "Overview of Development process"
tags: ["project", "blog"]
series_order: 1
showViews: false
showLikes: false
---

{{< katex >}}

{{< lead >}}
A basic heightfield implementation in OpenGL
{{< /lead >}}

## Introduction
A Height Field is simple and fun way to create some really nice looking terrain in a virtual enviornment. It has some limitations (such as the inability to create caves and overhangs), but otherwise, is a fantastic introductory terrain generation system that requires only basic graphics programming knowledge to implement.

The basic Idea is to generate the data for a plane mesh made up of a grid of nodes (evenly spaced apart), for each node, we sample from a height field texture to determine it's Y value.
The height field texture itself is just a top down grey scale image of a landscape in which darker shades of grey indicate lower elevation and lighter shades of grey indicate higher elevation.

A really easy way to understand it is with this image below:

![hf example img](HF_in_engine.png)

The top mesh is simply rendering the height field texture to a flat quad, the mesh below it is also rendering the same texture to a flat plane, but the vertices making up the plane mesh have their ***y*** values adjusted (up or down) depending on the shade of grey on the textures corresponding texel.
If your still a bit unsure, don't worry, I'll go into more depth in upcoming explanation

## Rendering a Height Field
The rendering process is quite intuitive once you understand the basic idea, but there are some minor difficulties that can cause a slight headache if not handled properly

## 1. Creating a Height Field Mesh

To create a height field we first want to define a plane mesh with nodes and edges laid out in an evenly spaced x and z space grid.

The image below gives an example:

{{< imgOutput id="image1" message="HF_mesh.png" >}}

This image represents a top down view of the mesh, no matter what height field data we end up using, this ***x*** and ***z*** components of the nodes in this mesh will remain the same. The height field will only effect the ***y*** component of each nodes position data.
<br>

{{< imgOutput id="image2" message="labeled_mesh_diagram.png" >}}

In OpenGl, we want to create a VBO of vertex data to represent a mesh like the one shown above. 
A simple way to do this is to calculate the number of nodes, based on the number of texels in the height field texture, then create an array of that size to hold to the data. 
We can set the distance/height between each node (the x and z offset) to any small value we like, but for each node we need to sample the color from the height field texture and use that to determine the ***y*** value.

But first we need to obtain an image to use as a height field, and then find out how to sample from it.

## 2. How to obtain a Height Field Texture

There are multiple ways to obtain a height field texture. The simplest would just be to go to google images and search for "height field" and grab one of the images.
<br><br>
Alternatively you can visit [this site](https://tangrams.github.io/heightmapper/) which provides satellite images of the world in height field format. Go to any location you like (e.g. Mt Fuji in Japan, the Andes in South America) and you can generate a height field of that location.
<br><br>
You can also create your own by hand using most image editing tools (such as photoshop, GIMP, Krita, etc.) or even specialized software like Terragen.
<br><br>
One of the most interesting options would be to procedurally generate our own height field texture using pseudo random noise such as Perlin or Simplex noise. If you are just starting out though, I recommend first using a pre-existing texture from one of the above options, as the setup for generating Perlin noise in your own code can be fairly tedious depending on the software tools you are using.
If you are interested however, I have my own project page where I go into detail about [generating a procedural world using Perlin noise](../procedural-world-pn).
<br><br>
If you really want to use noise for your height map, and don't care care about dynamically adjusting the values during program runtime, tools like Photoshop and GIMP have their own noise generation functions. You could use one of these tools to generate your height field texture file, export it as an image (I chose PNG for simplicity), and use it for our height field generation code.

For this example however, we will stick with a pre-made height field. I used this one that I found online for this example (feel free to also use it, but note that **it does not belong to me**, so if this is a professional application, I advise generating your own):

{{< imgOutput id="image3" message="heightmap.png" >}}


## 3. Calculating mesh Surface normals

We're close to actually creating our vertex data for our height map, but before we do so, I need to discuss an important requirement if we want to implement Phong shading (or almost any kind of traditional shading technique).
<br><br>
In order to implement Phong shading, our mesh needs to have surface normals. (these are just unit vectors that are perpendicular to the surface at any particular point on the mesh). 
We will need to to calculate the light intensity at each point on the mesh, so unless we want flat shading, we need to calculate them for our height field mesh.
<br><br>
I'm bringing this up now, as there are multiple ways of calculating the surface normals, and multiple points in our code were we can do so. Deciding on the best approach now will save us the headache of attempting to retroactively add it later.
<br><br>
At the risk of oversimplifying things, there are two points we would ideally chose to calculate the surface normals?
<br>
**Option 1** - Calculate them on the CPU before our render loop begins. and then store the surface normal data alongside the vertex position and texture coordinate data. Then we simply sample the surface normal data in our shader code from the vertex data inputs.
<br>
**Option 2** - Calculate the them every frame in the shader code.
<br><br>
Both options have their pros and cons, **option 1** front loads the calculation at startup, but then is highly efficient during runtime, as our GPU only needs to sample the pre calculated normals each frame. The downside being that our height field mesh geometry must remain static and can't change (unless we reconfigure the vertex buffer data between frames which is computationally expensive and inefficient).
**option 2** is more computationally expensive than option 1, but means that our geometry can be modified between frames and the normals will accurately represent the new geometry layout.
<br><br>
For this basic example I see no need to have our height field mesh vertex data change between frames. As such option 1 is a more preferable solution (in terms of efficiency).


{{< mermaid >}}
graph LR;
A[Set Width and Height]-->B[Iterrate over HF];
B-->C[Generate VBO]
{{< /mermaid >}}

## Final Result

![hm img](Featured.png)

[ In Development ]

## Tests
Testing reference to unity documentation on Forward Rendering [<a href="#ref1">1</a>\].
Testing reference to youtube video [<a href="#ref2">2</a>\].


## References
1. <a id="ref1"> Unity Technologies, "Forward rendering," Unity Documentation, 2023. [Online]. Available: https://docs.unity3d.com/Manual/RenderTech-ForwardRendering.html. [Accessed: Apr. 22, 2024].</a>
2. <a id="ref2"> Cambridge Computer Science Talks, 2023 "Forward and Deferred Rendering," Online video clip, YouTube, Available: <https://www.youtube.com/watch?v=n5OiqJP2f7w\>. [Accessed on: Apr. 26, 2024].</a>


