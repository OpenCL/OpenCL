---
ID: 26
post_title: GEGL – OpenCL in Gimp
author: OpenCL.org
post_date: 2017-04-06 20:08:20
post_excerpt: ""
layout: page
permalink: >
  http://opencl.org/projects/gegl-opencl-in-gimp/
published: true
hefo_before:
  - "1"
hefo_after:
  - "1"
---
<span class="st">The Generic Graphics Library</span> (GEGL) is best known as the backend for image processing software<em> Gimp</em>.

GEGL is a graph based image processing framework that allows users to chain together image processing operations represented by nodes into a graph. It provides operations for loading and storing images, adjusting colors, filtering in different ways, transforming and compositing images.

GEGL-OpenCL is an educational initiative that aims to get more developers to study and use OpenCL in their projects.
<ul>
 	<li>See the current state: <a href="http://gegl.opencl.org">http://gegl.opencl.org</a></li>
 	<li>Join the effort: <a href="https://github.com/OpenCL/GEGL-OpenCL">https://github.com/OpenCL/GEGL-OpenCL</a></li>
</ul>
<h1>Follow the examples</h1>
The below examples are meant to how you how to do a port. Here are the commits, so you see what changed:
<ul>
 	<li><a href="https://github.com/OpenCL/GEGL-OpenCL/pull/72/commits/91379e95356222db31f536aaceb97a65aef4dfc0">noise-hsv</a></li>
 	<li><a href="https://github.com/OpenCL/GEGL-OpenCL/pull/72/commits/f407e25046a017b635354da14bb5f010dd4c7d11">gaussian-blur-selective</a></li>
 	<li><a href="https://github.com/OpenCL/GEGL-OpenCL/pull/72/commits/6546c6fd6e3adad3bdd0ff1edb37e673e6b5bc06">motion-blur-linear</a></li>
 	<li><a href="https://github.com/OpenCL/GEGL-OpenCL/pull/72/commits/855cc073091762cabd67ba513ca7cc1b3b13dfe1">diffraction-patterns</a></li>
</ul>
Together with the Readme on Github, this should help you get started.
<h1>Contributors</h1>
<ul>
 	<li>Adel Johar (StreamHPC): setup of framework</li>
 	<li>Anton Gorenko (StreamHPC): noise-hsv, gaussian-blur-selective, motion-blur-linear, diffraction-patterns</li>
</ul>