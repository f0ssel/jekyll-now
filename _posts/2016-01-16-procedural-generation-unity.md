---
layout: post
title: Procedural Generation in Unity
---

I wanted to move over some images I had from my old blog onto this one so they don't get lost. These are from early 2016 where I was learning and playing around with procedural generation, specifically with terrains. I loved the low poly look so I stuck with the non-shared vertex faces. Most of these were just fun projects to run around in and learn new things. 

Starting out with generating vertexes and applying meshes

![]({{ site.baseurl }}/images/2016-01-16/1.png)

Some images showing off the generated noise. I got the library to generate this noise from [here](https://github.com/ricardojmendez/LibNoise.Unity)

![]({{ site.baseurl }}/images/2016-01-16/2.png)

![]({{ site.baseurl }}/images/2016-01-16/3.png)

![]({{ site.baseurl }}/images/2016-01-16/4.png)

To make it easy and fast to update these meshes on the fly I chunked them into 20x20 grids

![]({{ site.baseurl }}/images/2016-01-16/5.png)

Later on I wanted to generate terrain for a standalone island ala [Rust](https://rust.facepunch.com/)

![]({{ site.baseurl }}/images/2016-01-16/islandgen01.png)

I then came up with an idea for procedurally coloring the mesh based on height and really liked the look of the result. I would pick a pixel off a texture image based on altitude and location and set the UV of the triangle to that. Here's the image I used. 

![]({{ site.baseurl }}/images/2016-01-16/terrainblur.png)

And here's the result:

<center><img src="{{ site.baseurl }}/images/2016-01-16/islandgen03.png"></center>

![]({{ site.baseurl }}/images/2016-01-16/islandgen04.png)

![]({{ site.baseurl }}/images/2016-01-16/islandgen05.png)

![]({{ site.baseurl }}/images/2016-01-16/islandgen06.png)

![]({{ site.baseurl }}/images/2016-01-16/islandgen07.png)
