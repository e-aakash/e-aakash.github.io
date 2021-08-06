---
layout: post
title:  "AssImp: Build for Static Linking"
date:   2020-10-02 16:00:00 +0530
---

<link rel="stylesheet" type="text/css" href="/assets/github-buttons.css" />

<script src="https://ajax.googleapis.com/ajax/libs/jquery/1.12.0/jquery.min.js"></script>
<script type="text/javascript" src="/assets/github-buttons.js"></script>

_Updated on 6th August, 2021_

## What is AssImp & Why Static Linking

[AssImp](https://www.assimp.org/) is Open Asset Import Library for OpenGL, which is used for export or import of 3D models into applications being developed.

I needed AssImp as part of a project in one of our courses, Computer Graphics and Product Modelling. I needed a completely setup Visual Studio solution for synchronized workspace among the group members. So I decided to use static links of all dependencies, and add them in version control, so that problems are not of type mismatched dependency. 

But I couldn't find any guide online, just a stackoverflow answer with some steps. So these are the steps I used to create a .lib library of AssImp from source. 

**Note**: Executable file for assimp and tests exe file would not be generated because of non existent dll files

## Prerequisites

[Git](https://git-scm.com/), [CMake](https://cmake.org/) and [Visual Studio](https://visualstudio.microsoft.com/) (with c++ build tools) is required for following the build process.

## The Process

1. Clone the source code of assimp from github: [https://github.com/assimp/assimp](https://github.com/assimp/assimp) and checkout desired release tag 

2. Create a build directory in the assimp directory, open the CMake-gui and point correct source code and build location, and correct Visual Studio version in it. Clicking configure will show options. Make sure to select `ASSIMP_BUILD_ZLIB` ~~and `BUILD_SHARED_LIBS`~~. Unselect `BUILD_SHARED_LIBS` option in cmake options to make static library 

3. ~~A Visual Studio solution will be created in the build folder. In the properties of `assimp` and `zlib`, change "Configuration Type" to "Static Library", and in Advanced tab, change Target File Extension to ".lib". In `UpdateAssimpLibsDebugSymbolsAndDLLS`, change all ".dll" to ".lib" in Properties -> Build Events -> Post-build Events~~ 

    This is automatically done when you unselect shared libs option (I had wrong understanding of that flag, which was cleared by [official build docs](https://github.com/assimp/assimp/blob/master/Build.md#cmake-build-options))

4. Build the `ALL_BUILD` in required arch in **Debug** mode. _Note_: assimp-cmd and unit may fail to build

5. The generated .lib file will be in \<BuildDir>/code/Debug. Note that this will also require zlib and irrXML static libraries to work properly. They are present in \<BuildDir>/contrib/

6. Copy include files from assimp directory, and additionally copy `config.h` from \<BuildDir>/include/assimp for header files of assimp.

You now have the include and library files of assimp, which is all that is requred for properly linking the dependency

## Release Build

When you want to create release build of your project, you may want to link to release build of assimp (optimized and free of debug symbols, hence smaller sized). Following steps are different:

4. Build `ALL_BUILD` in required arch in **Release** mode. 

5. The generated .lib file will be in \<BuildDir>/code/Release. Note that this will also require zlib and irrXML static libraries to work properly. They are present in \<BuildDir>/contrib/

    Update Release configuration in your project to link to these lib files

    **Imp**: The release built files does not end with "d", so make sure you set correct library names in linker options in Release configuration of your project

6. Copy include files from assimp directory, and additionally copy `config.h` from \<BuildDir>/include/assimp for header files of assimp

Now you can create release build for your project with release builds of assimp static library. 
