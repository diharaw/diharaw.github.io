---
layout: post
title:  "Creating an Emscripten HTML5 build from a CMake-based project"
date:   2019-01-24
description: A tiny profiler class to use in OpenGL application to help you time your GPU operations.
---

In case you've been living under a rock for the past few years, Emscripten is a magical compiler that takes C/C++ source code and generates JavaScript that can run on any web browser. 
In addition to this, it also translates OpenGL calls to WebGL, which makes it perfect for game development. It's being used heavily in the games industry with big-name engines
such as Unity, Unreal and Godot using it for targetting the HTML5 platform. 

Another tool that's used frequently is CMake which is a cross-platform build system generator. It essentially generates projects for a number of supported IDE's and compilers using a small textfile called a CMakeList.
With modern games being available on multiple platforms, build system generators such as CMake are proving invaluable in order to ease the burden of multi-platform development (the fact that you don't need to maintain projects for each platform is a big win).

However, I ran into problems when I wanted to create an Emscripten build for a project of mine that used CMake. The documentation for this particular use case is lacking and it took me a while to actually get it right.
So here's a small write up from all of my findings.

### Prerequisites

Before we get into the details, first make sure you have the following installed.

* Emscripten SDK
* CMake
* Visual Studio 2017 (Community Edition or other)

There's no specific version required so just grab whatever is the latest. This process will work with Visual Studio 2015 as well, but I will not be covering that here.

Now let's add some paths to our PATH environment variable.

* Emscripten SDK root. Default: "C:\emsdk"
* Visual Studio Developer Command Prompt batch files. Default: "C:\Program Files (x86)\Microsoft Visual Studio\2017\Community\VC\Auxiliary\Build".
* CMake executalbe path. Default: "C:\Program Files\CMake\bin".

These will make writing batch files for automating the whole generation process a whole lot easier.

### CMake Toolchains

CMake has a neat little feature called Toolchains which allows you to generate cross-platform projects. These Toolchains take the form of a text file with the extension ".cmake" which specfies which compiler, linker etc to use.
For our example, we'll be using the CMake Toolchain file provided by Emscripten which is located at "[emsdk_root]/emscripten/[emsdk_version]\cmake\Modules\Platform\Emscripten.cmake". 

### Test Application

Since our focus is on the CMake side of things, we'll be using source code from GitHub as a starting point in order to hit the ground running. This Gist (https://gist.github.com/SuperV1234/5c5ad838fe5fe1bf54f9) contains source for a simple
application that uses SDL2 and OpenGL to render a triangle to the screen. 

### The CMakeList

The goal of this article isn't to teach CMake so I will skip all of that and present you the full source for the CMakeLists text file. The purpose of each line is explained within the given comments.

### Generating Makefiles 

The only way to build large Emscripten projects is via makefiles and thankfully CMake offers you generators for various flavors of makefiles. 
On Windows, makefiles cannot be compiled straight-away unlike Linux or macOS since there is no in-built "make" function. We have a few options in order to get around this,

1. Cygwin - A Unix-like environment and command-line interface for Microsoft Windows.
2. MSYS/MinGW - Similar to Cygwin.
3. NMake - The make implementation provided by Visual Studio.

Since Windows developers most likely have Visual Studio already installed, we'll only be covering the NMake option.  

Create a folder in your root source directory where you want the Makefiles to be generated and open a command prompt from within this newly created folder.

Before we actually run the CMake command to generate the Makefiles, we have to setup some environment variables first. 
This is where the path variables we setup earlier will come into play. 

* emsdk activate latest - This command will setup all the Emscripten environment variables that are required to locate the Emscripten compiler, linker and so on.
* vcvarsamd64_x86 - This script will setup the Visual Studio Developer Command Prompt. Without this, the NMake command will fail to execute.

NOTE: Make SURE you setup the Visual Studio Developer Command Prompt AFTER the Emscripten environment. Not doing so will lead to NMake not being located.

After executing these commands in the command prompt, we are ready to generate the Makefiles. Since we're using CMake to target a platform OTHER than x86, we need to specify the Emscripten Toolchain file as we discussed before. 
And so here is the finalized command to generate the makefile.

{% highlight c++ %}
cmake .. -DCMAKE_TOOLCHAIN_FILE=C:\emsdk\emscripten\1.38.12\cmake\Modules\Platform\Emscripten.cmake -G "NMake Makefiles"
{% endhighlight %}

Running this in the command prompt should result in the makefile being successfully generated. If for some reason it fails, make sure you setup your PATH variables correctly and that you ran the vcvarsamd64_x86 command AFTER the emsdk command.

### Building the Project

Now that the hard part is out of the way, let's actually build the project. This can be done by simply running "nmake" in the same folder as the makefile within the command prompt. 
Make sure vcvarsamd64_x86 was executed within this command prompt instance. Otherwise the nmake command will not be located. If all goes as planned, you should see an html file within the bin directory of the projects' root folder.
In order to run this html file successfully, you need to use a web server. If you don't want to install Apache or anything similar, Emscripten has you covered with its' built-in server which can be invoked with the following command:

{% highlight c++ %}
emrun [html_file]
{% endhighlight %}

During the first time running this, you'll have to setup which browser emrun should use. After that, running this command will result in your Emscripten application being opened up in your chosen web browser. 
The final result should look like this:

Congradulations! You've got your first Emscripten app up-and-running through CMake!

### A cross-platform example

TODO

So that wraps up this short article on Emscripten. Hope this helps anyone that wants to deploy their CMake-based project onto the web! 