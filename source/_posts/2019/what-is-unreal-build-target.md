---
title: What is UnrealEngine build target
date: 2019-08-06 23:49:20
tags: [UnrealEngine]
---

- this post covers
    - what is the build target in unreal engine
    - why we need the several build targets
- environment
    - Windows / 10
    - Unreal Engine / 4.19.2
    - Visual Studio IDE / 2017 Community

---

### overview
{% asset_img 01.jpg %}
- create new project with `Basic Code` in cpp tab

{% asset_img 02.jpg %}
- then you can see the directory like this

{% asset_img 03.jpg %}
- open the `[ProjectName].sln` and check out the csharp files whose name are ending up with `Target.cs`
- now you may wonder...
    - what the `Type = TargetType.Game` means
    - what the heck is `UnrealBuildTool`
    - what is difference between `[ProjectName]Target` and `[ProjectName]EditorTarget`
- do not hurry, first of all, we gonna learn about the build targets

### solution-configuration
{% asset_img 04.jpg %}
- most of you may have developed the game with no manipulation of `Solution Configurations`

{% asset_img 05.jpg %}
- when you folds it out, there are several options and each option is explained in [this document](https://docs.unrealengine.com/en-US/Programming/Development/BuildConfigurations/index.html) detailed
    - `DebugGame` and `Development` : build output is stand-alone binary file, which has `exe` extension
    - `DebugGame Editor` and `Development Editor` : build output is dynamic link library, which has `dll` extension
    - `Shipping` : build output is stand-alone binary file, which has `exe` extension
- the difference between
    - `DebugGame` and `Development` : the level and depth of debugging features
    - `DebugGame`, `Development` and `Shipping` : output of `Shipping` is more optimized for the reason of absence of command prompt and screen debug, etc. but, having no assets for the level presentation, both cannot be executed normally
    
{% asset_img 06.jpg %}

### build-target
- yep, you have seen the `Solution Configurations` and what they do
- and there is the way to change the behavior of each configuration by editing the `Target.cs` files

{% asset_img 07.jpg %}
- let us change the text `Editor` into `Game` in `[ProjectName]Editor.Target.cs`

{% asset_img 08.jpg %}
- remove the `Binaries` folder for checking the build output

{% asset_img 09.jpg %}
- build our project with `Development Editor`

{% asset_img 10.jpg %}
- look at the build output, what happened ? exe file has been generated, not the dll

- okay, now I would think it is time to tell you how the build works
    - when you select `DebugGame` or `Development`, unreal engine does the build based on `[ProjectName].Target.cs`
    - also, when you select `DebugGame Editor` or `Development Editor`, unreal engine will build based on `[ProjectName]Editor.Target.cs`
    - and the `Type` in `Target.cs` file is a key value of build configuration, which is read by UnrealBuildTool

{% asset_img 11.jpg %}
- as a result, there is no significant difference except for the file names
    - first 5 files are built with `Development` and `Test419.Target.cs` with `Type = TargetType.Game`
    - later 5 files are built with `Development Editor` and `Test419Editor.Target.cs` with `Type = TargetType.Game`

### why-it-needed
- this structure can help you customize build configurations
    - if you need to build for several enviroments, take care of the target files
    - there are many options not only `Type` value, find at [this document](https://docs.unrealengine.com/en-US/Programming/BuildTools/UnrealBuildTool/TargetFiles/index.html)
- typically, the target file is used for building an unreal server
    - at that time, we will use `[ProjectName]Server.Target.cs` with `Type = TargetType.Server`
    - because, in the production level, we need a server executing unreal dedicated server

### further
- I will write about unreal server build and combination with aws game lift
- in the following contents, this post should be useful for you
- there is [a good QnA](https://answers.unrealengine.com/questions/194712/differences-between-build-configurations.html) for these subjects