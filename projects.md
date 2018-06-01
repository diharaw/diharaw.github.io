---
layout: default
title: Projects
---

<div class="post">
	<h1 class="pageTitle">Projects</h1>
</div>

<h3><a href="https://github.com/diharaw/Terminus-Engine">Terminus Engine</a></h3>

<!-- ![Me]({{ site.baseurl }}/assets/img/editor.jpg){:style="float: right; height: 250px; margin-top: 20px; padding-right: 0px; padding-left: 20px"} -->

A C++ cross-platform game engine with rendering backends for OpenGL and Direct3D 11 (Vulkan, Direct3D 12 and Metal in the future), Lua scripting, runtime-compiled C++, customizable rendering pipeline and more. 

![Terminus-Engine]({{ site.baseurl }}/assets/img/editor.jpg)

---
<h3><a href="https://github.com/diharaw/Rasterator">Rasterator</a></h3>

<!-- ![Me]({{ site.baseurl }}/assets/img/rasterator.jpg){:style="float: right; height: 250px; margin-top: 20px; padding-right: 0px; padding-left: 20px"} -->

A C++ software rasterizer created for the purpose of learning how the graphics pipeline functions. Features triangle rasterization, perspective correct interpolation, texture mapping, bilinear filtering and multithreading via OpenMP.

![Rasterator]({{ site.baseurl }}/assets/img/rasterator.jpg)

---
<h3><a href="https://github.com/diharaw/dwThreadPool">dwThreadPool</a></h3>

C++ 11 thread pool library with task-based multithreading. Allows you to create task continuations, groupings and dependencies and also to wait on certain tasks. Fully cross-platform and completely dependency-free: can be dropped into any existing C++ codebase. 

![dwThreadPool]({{ site.baseurl }}/assets/img/threadpool.jpg)

---
<h3><a href="https://github.com/diharaw/EmoLib">EmoLib</a></h3>

A Caffe-powered Facial and Speech emotion recognition library written in C++. The transfer learning method was used to train the Facial emotion classifier which allowed us to reach 98% accuracy. The Speech emotion classifier fared at 67% accuracy. This was done as my final year thesis.

![dwThreadPool]({{ site.baseurl }}/assets/img/emolib.jpg)

---
<h3><a href="https://github.com/diharaw/Terminus-GFX">Terminus-GFX</a></h3>

A graphics API abstraction which provides a clean and low-level API that reflects modern graphics API's such as Vulkan and Direct3D 12. Currently backends  are available for OpenGL and Direct3D 11 with OpenGL ES, WebGL, Vulkan, Direct3D 12, and Metal coming in the near furture. Utilizes Vulkan GLSL shaders which are cross compiled using SPIRV-Cross when used in non-Vulkan backends. Used within Terminus Engine.

---
<h3><a href="https://github.com/diharaw/Terminus-Math">Terminus-Math</a></h3>

A header-only C++ linear algebra library for use in computer graphics applications with implementation of vectors, matrices and other common functions.

---
<h3><a href="https://github.com/diharaw/dwCompute">dwCompute</a></h3>

Header-only CUDA and OpenCL abstraction written in C++. Most common functionality of both API's are covered with more to be added in the future. Provides a set of macros which allows you to write a single Kernel to be consumed by both API's. Backends selected via a simple preprocessor definition.

---
<h3><a href="https://github.com/diharaw/dwSerializer">dwSerializer</a></h3>

C++ serialization library with support for JSON, XML and binary file formats. Completely template based, therefore not requiring any inheritance. 