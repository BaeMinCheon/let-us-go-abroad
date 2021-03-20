---
title: Differences in UnrealEngine build targets
date: 2019-08-16 00:13:26
tags: [UnrealEngine]
---

- recommend to read first
    - [what-is-unreal-build-target](https://baemincheon.github.io/2019/08/06/what-is-unreal-build-target/)
    - [custom-unreal-engine-build](https://baemincheon.github.io/2019/08/11/custom-unreal-engine-build/)
- this post covers
    - what does mean each build target
    - how to use each build target
    - differences of some build targets
- environment
    - Windows / 10
    - Visual Studio IDE / 2017 Community

---

# overview
- I suppose you already have the unreal engine built from source of 4.21 version
    - if you do not, read the `custom-unreal-engine-build` post

{% asset_img 02.jpg %}
- once you build the `UE4` project with option `Development Editor & Win64`
    - you can find `[EngineRoot]/Engine/Binaries/Win64/UE4Editor.exe` and execute it

{% asset_img 01.jpg %}
- in this window, you can create project using your custom engine
    - create a `Third Person` template project for testing several build targets in this post

# build-targets
{% asset_img 12.jpg %}
- in unreal engine, "build (file)" means the executable file or library file made from source code
    - ex1) `UE4Editor-[ProjectName].dll` of `Editor` build target
    - ex2) `[ProjectName]Client.exe` of `Client` build target
    - ex3) `[ProjectName].exe` of `Game` build target
- there are four build targets frequently used
    - Client
    - Server
    - Game
    - Editor


- `Client`
    - this target is used for building only client
    - it has several features especially displaying screen
    - so if the system does not have any graphics API, it cannot be executed


- `Server`
    - this target is used for building only server
    - it does not have several features especially displaying screen
    - so regardless of graphics API, it can be executed


- `Game`
    - this target is used for building whole game
    - it has all features from `Client` and `Server` targets
    - so a build from this target can be used as client, and server too
        - it is not recommended for the commercial game project because user could get the server feature


- `Editor`
    - this target is used for executing the game on unreal editor
    - only the build from this target can be opened in unreal editor
        - also, unreal editor only can open this build, which means packaging impossible
    - totally, it has all features from `Game`


# packaging
- because build files do not contain unreal assets, it cannot be executed alone
    - we have checked out what happens when only using a build file in previous post
    - so, executing the game from any build file, you need to package unreal assets used in the project file
    - and packaging can be done in unreal editor, not the source code editor such as visual studio IDE
- there are some prerequisites for pacakaging the test project

{% asset_img 08.jpg %}
- open the `sln` for the test project and build it with `Development Editor & Win64`
    - and press `F5` to start debugging

{% asset_img 09.jpg %}
- you would see the `Third Person` default map with unreal editor

{% asset_img 10.jpg %}
- open the `Project Settings` window and find `Maps & Modes` tab
    - set every map with the default map

{% asset_img 11.jpg %}
- find `Packaging` tab
    - add the default map to the `List of maps to include in a packaged build`

{% asset_img 06.jpg %}
{% asset_img 13.jpg %}
- write target config files for `Client` and `Server` targets

{% asset_img 07.jpg %}
- right click `uproject` and select `Generate project files`


- re-open the `sln` and build the test project with each solution configuration
    - `Development & Win64`
    - `Development Client & Win64`
    - `Development Server & Win64`

{% asset_img 14.jpg %}
- now you can see the three build files


- open the test project with unreal editor
    - way1) double click `uproject`
    - way2) execute `[EngineRoot]/Engine/Binaries/Win64/UE4Editor.exe` and select the project

{% asset_img 15.jpg %}
- select `Development` in `File/Package Project/Build Configuration`

{% asset_img 16.jpg %}
{% asset_img 17.jpg %}
- select `Windows (64-bit)` in `File/Package Project/Windows`
    - choose an arbitrary directory for saving the package
    - in my case, I created `Packages` folder in project directory and use it

{% asset_img 18.jpg %}
- after packaging, you can see the directory like this

{% asset_img 19.jpg %}
- as we packaged the project with `Development` the executable files are the same

{% asset_img 20.jpg %}
- copy the build files into the `Packages/WindowsNoEditor/[ProjectName]/Binaries/Win64`
    - `[ProjectName].exe`
    - `[ProjectName]Client.exe`
    - `[ProjectName]Server.exe`

{% asset_img 21.jpg %}
- run the command prompt and move to the `Packages/WindowsNoEditor/[ProjectName]/Binaries/Win64`
    - execute each build file with some options for knowing what happens inside

{% asset_img 22.jpg %}
{% asset_img 23.jpg %}
- execute `Game` build file with options `-log -windowed resx=720 resy=480`
    - `-log` option makes the game print logs
    - `-windowed` option prevents the game from running as full-screen
- you can terminate the game
    - way1) click `X` on the right of game window
    - way2) press a [grave accent](https://en.wikipedia.org/wiki/Grave_accent) and type `exit`

{% asset_img 24.jpg %}
- execute `Client` build file with options `-log -windowed resx=720 resy=480`
    - you can see the same result of `Game` build

{% asset_img 25.jpg %}
- execute `Server` build file with options `-log`
    - you can see there is no game screen with `Server` build

# game-client-server
- I mentioned `Game` as "it has all features from `Client` and `Server` targets"


- execute `Game` build 3 times with options below
    - 1) `-server 127.0.0.1 -log -windowed resx=720 resy=480`
    - 2) `127.0.0.1 -log -windowed resx=720 resy=480`
    - 3) `127.0.0.1 -log -windowed resx=720 resy=480`

{% asset_img 26.jpg %}
- the first game acts as (client + server) and other games act as client
    - this means `Game target = Client target + Server target`


- execute `Server` build with options below
    - `127.0.0.1 -log`
- execute `Client` twice with options below
    - 1) `127.0.0.1 -log -windowed resx=720 resy=480`
    - 2) `127.0.0.1 -log -windowed resx=720 resy=480`

{% asset_img 27.jpg %}
- the first game acts as server and other games act as client
    - you can see server do not have a screen