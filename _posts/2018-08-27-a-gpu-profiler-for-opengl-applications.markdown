---
layout: post
title:  "A GPU Profiler for OpenGL Applications"
date:   2018-08-27
description: A tiny profiler class to use in OpenGL application to help you time your GPU operations.
---

![Nimble-Profiler]({{ site.baseurl }}/assets/img/profiler.jpg)

An important part of any Graphics Programming job is optimization. And to do that, you'd need know just how fast your shiny new shadow mapping algorithm, post-process shader etc is running. And the recommended measure of performance is ideally in Miliseconds rather than Frames Per Second (Read [this article](https://www.mvps.org/directx/articles/fps_versus_frame_time.htm) for a great explanation on why). 

Simply timing your OpenGL calls on the CPU side is the completely wrong way to go about this as the CPU and GPU operate on completely different timelines (the GPU is *slightly* behind the CPU). And depending on the load, one could take longer than the other. So in order to measure how long OpenGL commands take to complete on the GPU, we have to utilize Queries. 

This article is an explanation of the tiny GPU Profiler class implemented within my open-source rendering engine, [Nimble](https://github.com/diharaw/Nimble).

## GPU Queries

A query in OpenGL (or any other graphics API) is an object used to asynchronously query a certain piece of information on the GPU-side back to the CPU-side. Typically the query operates within a range of commands specified by the user through the API calls `glBeginQuery` and `glEndQuery`. 

{% highlight c++ %}
// Generate query objects
GLuint query;

glGenQueries(1, &query)

// Begin the query before issuing the commands from which you would like to query from
glBeginQuery(query, GL_TIME_ELAPSED);

// Commands go here...

// End the query afterwards
glEndQuery(GL_TIME_ELAPSED);

{% endhighlight %}

There are many types of queries available in OpenGL, but in this article we'll only focus on GL_TIME_ELAPSED query. If you wish to learn about the other queries, please visit the [Khronos Wiki Page](https://www.khronos.org/opengl/wiki/Query_Object). Here's the description of the GL_TIME_ELAPSED query straight from the wiki:

> GL_TIME_ELAPSED​: Records the time that it takes for the GPU to execute all of the scoped commands. The timer starts when all commands before the scope have completed, and the timer ends when the last scoped command has completed. Requires OpenGL 3.3 or ARB_timer_query

As you can see, this command will return the GPU time taken by the commands between glBeginQuery and glEndQuery. Since the execution of the query is asynchronous, we are unable to read the results back instantly. A spinlock on the CPU-side is also a horrible choice, as it forces a synchronization point between the CPU and GPU which eventually starves them both of work at some point. To counter this, we have to read the results back *a few frame later*. 

For this to work without causing any stalls, we must buffer the Query objects. For example, if we have 3 query objects, we can cycle through each one and read back the results when we eventually come back to re-submit, essentially behaving like a ring-buffer. 

## Nimble's GPU Profiler

This is the same concept used in the GPU Profiler implementation from Nimble. The Profiler API looks as follows.

{% highlight c++ %}

GPUProfiler::begin("Name of your GPU operation");

// OpenGL calls go here...

GPUProfiler::end("Name of your GPU operation");

// Somewhere else in the code

float time = GPUProfiler::result("Name of your GPU operation");

{% endhighlight %}

The Profiler class keeps a map of all Profiling Scopes (which is essentially a buffer of Queries). When a non-existent Query is started, a Profiling Scope for that query name is created and added into the map. Before actually calling glBeginQuery, it tries to retrieve any results it might have from earlier query submissions. For each frame the Profiler cycles through each query object in the buffer until it wraps around. These buffered queries prevents us from hitting any stalls. 

## Implementation

The entire implementation can be found in the following links. OpenGL calls are not directly used here as I have my own C++ OpenGL wrapper class within my sample framework ([ogl.h](https://github.com/diharaw/dwSampleFramework/blob/master/include/ogl.h) and [ogl.cpp](https://github.com/diharaw/dwSampleFramework/blob/master/src/ogl.cpp)).

* [gpu_profiler.h](https://github.com/diharaw/Nimble/blob/master/src/gpu_profiler.h)
* [gpu_profiler.cpp](https://github.com/diharaw/Nimble/blob/master/src/gpu_profiler.cpp)

Hope this will help out anyone looking for a quick way to profile their OpenGL code.