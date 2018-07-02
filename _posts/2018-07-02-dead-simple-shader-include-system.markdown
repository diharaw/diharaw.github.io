---
layout: post
title:  "Dead Simple Shader Include System"
date:   2018-07-02
description: A quick tutorial on how to write a super simple shader include system for OpenGL projects that will save you countless hours of rewriting and copy/pasting.
---

In a graphics project of considerable size, your shader codebase will start getting pretty big and a lot of common elements such as structure declarations, helper functions and constants will eventually be duplicated because of this. How we would ideally want to write our shaders is like this:

{% highlight c++ %}
#include <../../common/uniforms.glsl>

out vec3 FragColor;

in vec2 PS_IN_TexCoord;

uniform sampler2D s_ColorMap;
uniform sampler2D s_VelocityMap;

void main()
...
{% endhighlight %}

Those of you who are new to OpenGL might have at one point or another wanted to do C/C++ style header includes like the one shown above inside your GLSL code, only to find out that OpenGL does not actually support that. But fear not, you can always cobble together your own shader include system which is exactly what I will be showing you in this post.

## Overview

Functionality wise our system will be dead simple (hence the title). It will simply paste the code from the given shader source into the currently processed shader source, starting from the line where the current #include directive used to be. For example, if the files "foo.glsl" and "bar.glsl" were as follows,

{% highlight c++ %}
// Contents of foo.glsl
struct Foo
{
    vec4 color;
};

// Contents of bar.glsl
#include <foo.glsl>

void main()
{
    Foo foo;
}
{% endhighlight %}

Then the result of reading "bar.glsl" would generate the following shader source.

{% highlight c++ %}
struct Foo
{
    vec4 bar;
};

void main()
{
    Foo foo;
}
{% endhighlight %}

## Implementation

I started to write a paragraph description of the function, but decided to scrap it since commented source code is much easier to understand. So here you go!

{% highlight c++ %}
bool read_shader(std::string path, std::string& out)
{
    std::string og_source;

    // Try to read source file. If failed, return false.
    if (!read_text(path, og_source))
        return false;

    std::istringstream stream(og_source);
    std::string line;

    // Iterate source line-by-line
    while (std::getline(stream, line))
    {
        // If a "#include" is encountered...
        if (line.find("#include") != std::string::npos)
        {
            // ...find the positions of the angled brackets.
            size_t start = line.find_first_of("<") + 1;
            size_t end = line.find_last_of(">");

            // Substring between those positions to get the included path.
            std::string include_path = line.substr(start, end - start);

            // Find the position of the last slash from the input path (the string passed into this function).
            std::string path_to_shader = "";
            size_t slash_pos = path.find_last_of("/");

            // If a slash is encountered, substring from the beginning to the slash position to get
            // the path without the file name.
            if (slash_pos != std::string::npos)
                path_to_shader = path.substr(0, slash_pos + 1);

            std::string include_source;

            // Recursively call the read_shader function by appendeding the input file's path to the include path.
            if (!read_shader(path_to_shader + include_path, include_source))
            {
                // If reading fails, print an error and return false.
                std::cout << "Included file <" <<  include_path <<  "> cannot be opened!" << std::endl;
                return false;
            }
            else
            {
                // If reading succeeds, append the read source into the current output.
                out += include_source + "\n\n";
            }
        }
        else // Not a line containing a "#include", so simply append it to the output.
            out += line + "\n";
    }

    return true;
}
{% endhighlight %}

Note: read_text is a function that simply reads a text file into an std::string.

## Limitations

All you sharp readers may have already spotted a few flaws while going through the code. First off, if the root source path is absolute, then all included source paths within it must be relative. Otherwise the appending operation will result in an invalid path. 
Another problem is include ordering: our preprocessor is dumb, unlike the ones in C/C++ compilers. All it does is simply copy the contents of the included files into the root shader source in the order they were declared in. This will cause issues if, for example, you include a file containing your structure declarations *after* a file that actually uses them. It will surely result in the GLSL compiler throwing errors at you.

However, even with these problems this system will save you an *immense* amount of time if you have a lot of common functions between your shaders, it sure did help me (This code was ripped straight out of a simple OpenGL rendering engine called "Nimble", which I will write about soon). I may update this post later on once I make some improvements to it, but in the mean time, feel free to give it your own spin. Anyways hope this helped someone out. Until the next post, happy programming!