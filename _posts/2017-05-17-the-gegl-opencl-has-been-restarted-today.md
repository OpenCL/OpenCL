---
ID: 233
post_title: >
  The GEGL-OpenCL has been restarted
  today!
author: Vincent Hindriksen
post_date: 2017-05-17 11:43:27
post_excerpt: ""
layout: post
permalink: >
  http://opencl.org/2017/05/17/the-gegl-opencl-has-been-restarted-today/
published: true
hefo_before:
  - "0"
hefo_after:
  - "0"
---
With StreamHPC's <a href="https://github.com/ex-rzr">Anton</a> porting 4 kernels to OpenCL we've restarted the GEGL porting project! Only 64 filters to go.<img class="alignnone wp-image-234 size-large" src="http://opencl.org/wp-content/uploads/2017/05/GEGL_Logo.svg_-1024x343.png" alt="" width="525" height="176" />

We're happy this initiative has been restarted, as it did not get off the ground last year. Biggest change is that it's now fully a community project and labeled "OpenCL.org's GEGL Port"

We'll be doing our best to celebrate the work of each individual member by naming all contributors on the <a href="http://opencl.org/projects/gegl-opencl-in-gimp/">project page</a>.
<h1>Follow the examples</h1>
The project's <a href="https://github.com/OpenCL/GEGL-OpenCL/blob/master/README.md">README.md</a> gives an overview on how to get started, but does not show how to do the OpenCL-part.

The four ported filters are noise-hsv, gaussian-blur-selective, motion-blur-circular,  and diffraction-patterns. These are meant as examples on how to do port to OpenCL. Here are the commits, so you see what changed:
<ul>
 	<li><a href="https://github.com/OpenCL/GEGL-OpenCL/pull/72/commits/91379e95356222db31f536aaceb97a65aef4dfc0">noise-hsv</a> (<a href="https://github.com/OpenCL/GEGL-OpenCL/blob/master/operations/common/noise-hsv.c">original C code</a>)</li>
 	<li><a href="https://github.com/OpenCL/GEGL-OpenCL/pull/72/commits/f407e25046a017b635354da14bb5f010dd4c7d11">gaussian-blur-selective</a> (<a href="https://github.com/OpenCL/GEGL-OpenCL/blob/master/operations/common/gaussian-blur-selective.c">original C code</a>)</li>
 	<li><a href="https://github.com/OpenCL/GEGL-OpenCL/pull/72/commits/6546c6fd6e3adad3bdd0ff1edb37e673e6b5bc06">motion-blur-circular</a> (<a href="https://github.com/OpenCL/GEGL-OpenCL/blob/master/operations/common/motion-blur-circular.c">original C code</a>)</li>
 	<li><a href="https://github.com/OpenCL/GEGL-OpenCL/pull/72/commits/855cc073091762cabd67ba513ca7cc1b3b13dfe1">diffraction-patterns</a> (<a href="https://github.com/OpenCL/GEGL-OpenCL/blob/master/operations/common/diffraction-patterns.c">original C code</a>)</li>
</ul>
<h1>Let's do your own port</h1>
Have you read the project's <a href="https://github.com/OpenCL/GEGL-OpenCL/blob/master/README.md">README.md</a> and understood from the above examples how to do such port? Let's get started then.

All to-be-ported filters are on the <a href="https://github.com/OpenCL/GEGL-OpenCL/issues">issue-list</a>. If you want to keep it easy at start, you can first choose to pick or a simple example or one that is similar to the above examples. There is always help on the <a href="https://gegl-opencl.slack.com/">Slack-chat</a>! Also when you're new to this Github-thing.

After you did a port yourself, <a href="https://github.com/OpenCL/GEGL-OpenCL/pulls">submitted it on Github</a> and got an accepted PR, then you can add your name to the list of contributors together with the filter(s) you ported. Then it's just time for GEGL-maintainers to do their own tests and merge with the main repository.