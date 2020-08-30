---
title: custom-unreal-engine-build
date: 2019-08-11 14:50:32
tags: [UnrealEngine]
---

- recommend to read first
    - [what-is-unreal-build-target](https://baemincheon.github.io/2019/08/06/what-is-unreal-build-target/)
- this post covers
    - why we need the custom unreal engine
    - how to access to unreal engine code
    - how to build and configure our custom unreal engine
- environment
    - Windows / 10
    - Visual Studio IDE / 2017 Community

---

# overview
- using unreal engine with the `Epic Games Launcher` means that you can use some parts of unreal engine
    - because, engines provided from `Epic Games Launcher` are lack of some features ( find more at [here](https://answers.unrealengine.com/questions/39658/differences-between-github-and-unreal-download.html) )
    - especially, you can only use `Client` and `Server` build target with custom unreal engine

{% asset_img 01.jpg %}
- if you want to use whole of unreal engine, you need to build unreal engine from source code
    - making a project with custom engine, `Epic Games Launcher` recognizes the project but not the version of engine
    - you can see the `Other` on the screenshot above, which means that versioning is not meaningful no more
- for example, suppose you need to seperate your game project into client and server
    - that means, client version of your game only has the feature for client and _vice versa_
    - build targets supported by engine from `Epic Games Launcher` are only `Game` and `Editor` so you cannot

{% asset_img 02.jpg %}

# access
- accessing to unreal engine repository needs some process below
- visit [epic games page](https://www.unrealengine.com/en-US/?lang=en-US) and sign in

{% asset_img 03.jpg %}
{% asset_img 04.jpg %}
- click `PERSONAL` in the combo box on your nickname and click `CONNECTED ACCOUNTS`
    - click `CONNECT` in the `GITHUB` box and sign in with your github account
    - then, some mails would be sent to your email and accept them

{% asset_img 05.jpg %}
- now you can find that you have entered the `Epic Games` organization

{% asset_img 06.jpg %}
- visit the unreal engine repository and download or clone it

{% asset_img 08.jpg %}
- if you want to make custom engine based on a specific version, select the proper branch

{% asset_img 07.jpg %}
- now you are ready to build custom engine

# build
- before starting, there are some requirements to visual studio IDE

{% asset_img 09.jpg %}
- execute visual studio 2017 and click `Tools/Get Tools and Features...`
    - in `Individual components` tab, you should check the components below
        - `.NET Framework 4.5` things
        - `.NET Framework 4.6` things
        - `VC++ 2015 for desktop` things

{% asset_img 10.jpg %}
- right click the `Setup.bat` in engine root folder and select `Run as administrator`

{% asset_img 11.jpg %}
- execute command prompt and move to engine root folder
    - type `GenerateProjectFiles.bat -2017` and enter
    - now you can see `UE4.sln` is generated and open it with visual studio 2017

{% asset_img 12.jpg %}
- right click `UE4` project and select build
    - it takes soooo long time ( about 1~2 hours )

# usage
- launch any engine on `Epic Games Launcher`

{% asset_img 13.jpg %}
- create some project

{% asset_img 14.jpg %}
- right click `uproject` and select `Switch Unreal Engine version...`

{% asset_img 15.jpg %}
- if the build was successfully done, there is the engine root in combo box
    - if not, find and choose the engine root directory

{% asset_img 16.jpg %}
- select proper one and it starts generating project files

{% asset_img 17.jpg %}
- open the `[ProjectName].sln` and you can check out the `UE4` in solution explorer

{% asset_img 18.jpg %}
- even you would find the `Client` and `Server` options in solution configurations
    - this process is required whenever you want to use custom engine
    - hooh ! now you can use your own custom unreal engine !