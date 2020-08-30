---
title: unreal-linux-server-build
date: 2019-08-19 00:46:45
tags: [UnrealEngine, Linux, WSL]
---

- recommend to read first
    - [what-is-unreal-build-target](https://baemincheon.github.io/2019/08/06/what-is-unreal-build-target/)
    - [custom-unreal-engine-build](https://baemincheon.github.io/2019/08/11/custom-unreal-engine-build/)
    - [differences-of-unreal-build-targets](https://baemincheon.github.io/2019/08/16/differences-of-unreal-build-targets/)
- this post covers
    - how to setup cross compile environment
    - how to build linux server
    - how to use client and server in cross platform
- environment
    - Windows / 10
    - Visual Studio IDE / 2017 Community
    - Unreal Engine / 4.21 Built From Source Code

---

# overview
- if you want to seperate the game into client and server
    - mostly, the clients would be executed on windows
    - as the server is up to you, you can select more efficient option
        - in aws, linux server instead of windows to save cost
- so it is needed to build linux server, but you can use cross compilation
    - this makes you can build linux server on windows
- in this post, I suppose you have already an unreal engine built from source code and test project
    - especially, I used 4.21 version

# setup
- download the proper toolchain from [this document](https://docs.unrealengine.com/en-US/Platforms/Linux/GettingStarted/index.html)
    - in my case, `clang-6.0.1-based` toolchain is needed
- execute the toolchain installer
    - you do not have to do extra works when your engine version is equal to or over 4.14
    - if not, reference the document mentioned for the details

{% asset_img 01.jpg %}
- in the test project, edit the `DefaultEngine.ini` of `[ProjectRoot]/Config/DefaultEngine.ini`
    - add the code below
    - the code will add configurations for linux version build

```
[/Script/LinuxTargetPlatform.LinuxTargetSettings]
TargetArchitecture=X86_64UnknownLinuxGnu
```

{% asset_img 02.jpg %}
- right click `uproject` and select `Generate Visual Studio project files`
    - open `sln` and build the test project with `Development Editor & Win64`

# build
{% asset_img 03.jpg %}
- double click `uproject` and you would see the test project on unreal editor

{% asset_img 04.jpg %}
- select `Development` in `File/Package Project/Build Configuration`

{% asset_img 05.jpg %}
- select `Linux` in `File/Package Project`
    - choose an arbitrary directory for saving the package
    - in my case, I created `Packages` folder in project directory and use it

{% asset_img 06.jpg %}
- after packaging, you can see the directory like this

{% asset_img 07.jpg %}
- open `sln` and build the test project with `Development Server & Linux`

{% asset_img 08.jpg %}
- you can see the `[ProjectName]Server` build, which will be executed on linux

{% asset_img 09.jpg %}
- copy the `[ProjectName]Server` into package binary folder
    - now you can execute `[ProjectName]Server` on linux

# usage
- in this post, I will show you an example with wsl
    - I recommend to make shared folder for sharing files
    - if you do not know about it, read [this post](https://baemincheon.github.io/2019/08/19/shared-folder-in-wsl/)

{% asset_img 10.jpg %}
- I used a shared folder called `LinuxShare` for sharing the package files

{% asset_img 11.jpg %}
- there are build files in package binary folder
    - `[ProjectName]` is build file from `Game` build target, which cannot be executed for absense of graphics api
    - `[ProjectName]Server` is build file from `Server` build target, which is copied by you

{% asset_img 12.jpg %}
{% asset_img 13.jpg %}
- execute server build with option `127.0.0.1 -log` on wsl(linux)

{% asset_img 14.jpg %}
- execute client build twice with option `127.0.0.1 -log -windowed resx=720 resy=480` on windows
    - you can see the same result that we saw in previous post