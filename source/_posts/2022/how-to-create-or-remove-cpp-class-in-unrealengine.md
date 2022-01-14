---
title: How to create or remove CPP class in UnrealEngine
date: 2022-01-11 22:08:04
tags: [UnrealEngine]
---

| Environment | |
| --- | --- |
| UnrealEngine | `branch: 5.0` |
| Visual Studio 2022 | `version: 17.0.4` |
| Windows 11 Pro | `build: 22000.376` |

# _`Overview`_

Developing your UnrealEngine project with only the blueprint is not easy because the blueprint has some limitations on functionalities than the native, CPP. For instance, in blueprint you can access the source code tagged by `BlueprintCallable`, `BlueprintType`, `BlueprintReadOnly`, or those series. But, in CPP you can access all of the source code as possible and even you can modify the source code of engine. In other words, using only blueprint is like using a part of UnrealEngine. So eventually, you would want to create CPP class for more functionalities. This post covers that topic; how to create CPP class in UnrealEngine.

Plus, not only creating something but removing something is important. I will tell you how to remove CPP class in UnrealEngine, too. Let us create a project from ThirdPerson template with the options below. I named it as `Unreal_5_0`. 

{% asset_img 01.png %}

# _`Creating CPP class; method #1`_

{% asset_img 02.png %}

Open the `Content Drawer` and click `All/C++ Classes` folder. After the steps, you can see the option `New C++ Class...` when you click the `Add` button. Click it.

{% asset_img 03.png %}

In this dialog, you can select a parent of new CPP class. `Common Classes` tab contains the most commonly used classes, so you should switch to `All Classes` tab and find an appropriate class if needed.

{% asset_img 04.png %}

I chose the class `UserWidget` as a parent of new CPP class. Click the button `Next>`.

{% asset_img 05.png %}

In this dialog, you can name the new CPP class and save it with some options. I will left the name as default, `My[ParentClassName]`. The combobox beside name is for selecting a module to include this class. Our project created from ThirdPerson template starts with only one module whose name is the same with project, in this case `Unreal_5_0`.

```cpp
// GameProjectUtils.h

/** Where is this class located within the Source folder? */
enum class EClassLocation : uint8
{
    /** The class is going to a user defined location (outside of the Public, Private, or Classes) folder for this module */
    UserDefined,

    /** The class is going to the Public folder for this module */
    Public,

    /** The class is going to the Private folder for this module */
    Private,

    /** The class is going to the Classes folder for this module */
    Classes,
};
```

The radio button `Class Type` is for selecting a location of new CPP class. The enum value is `UserDefined` in default, but it would be forced to `Public` or `Private` when you select one of the radio buttons.

```cpp
// SNewClassDialog.cpp

void SNewClassDialog::OnClassLocationChanged(GameProjectUtils::EClassLocation InLocation)
{
	const FString AbsoluteClassPath = FPaths::ConvertRelativePathToFull(NewClassPath) / ""; // Ensure trailing /

	GameProjectUtils::EClassLocation TmpClassLocation = GameProjectUtils::EClassLocation::UserDefined;
	GameProjectUtils::GetClassLocation(AbsoluteClassPath, *SelectedModuleInfo, TmpClassLocation);

	const FString RootPath = SelectedModuleInfo->ModuleSourcePath;
	const FString PublicPath = RootPath / "Public" / "";		// Ensure trailing /
	const FString PrivatePath = RootPath / "Private" / "";		// Ensure trailing /

	// Update the class path to be rooted to the Public or Private folder based on InVisibility
	switch (InLocation)
	{
	case GameProjectUtils::EClassLocation::Public:
		if (AbsoluteClassPath.StartsWith(PrivatePath))
		{
			NewClassPath = AbsoluteClassPath.Replace(*PrivatePath, *PublicPath);
		}
		else if (AbsoluteClassPath.StartsWith(RootPath))
		{
			NewClassPath = AbsoluteClassPath.Replace(*RootPath, *PublicPath);
		}
		else
		{
			NewClassPath = PublicPath;
		}
		break;

	case GameProjectUtils::EClassLocation::Private:
		if (AbsoluteClassPath.StartsWith(PublicPath))
		{
			NewClassPath = AbsoluteClassPath.Replace(*PublicPath, *PrivatePath);
		}
		else if (AbsoluteClassPath.StartsWith(RootPath))
		{
			NewClassPath = AbsoluteClassPath.Replace(*RootPath, *PrivatePath);
		}
		else
		{
			NewClassPath = PrivatePath;
		}
		break;

	default:
		break;
	}

	// Will update ClassVisibility correctly
	UpdateInputValidity();
}
```

With these codes, the radio buttons just change the location of new CPP class. The new CPP class would be included in `Public` folder when you clicked a radio button `Public`, vice versa. This setting makes some differences especially onto accessibility.

```cpp
// GameProjectUtils.cpp
// GameProjectUtils::GenerateClassHeaderFile()

if ( GetClassLocation(NewHeaderFileName, ModuleInfo, ClassPathLocation) )
{
    // If this class isn't Private, make sure and include the API macro so it can be linked within other modules
    if ( ClassPathLocation != EClassLocation::Private )
    {
        ModuleAPIMacro = ModuleInfo.ModuleName.ToUpper() + "_API "; // include a trailing space for the template formatting
    }
}
```

```cpp
// Definitions.Unreal_5_0.h

#define UNREAL_5_0_API DLLEXPORT
```

Only the class of location for `Private` cannot have the macro `[ModuleName]_API`. And the macro is defined as `DLLEXPORT`. The attribute is used to export codes in MSVC, visit [here](https://docs.microsoft.com/en-us/cpp/cpp/dllexport-dllimport?view=msvc-170) for more details.

```cpp
// MyUserWidget.h

#pragma once

#include "CoreMinimal.h"
#include "Blueprint/UserWidget.h"
#include "MyUserWidget.generated.h"

/**
 * 
 */
UCLASS()
class UNREAL_5_0_API UMyUserWidget : public UUserWidget
{
	GENERATED_BODY()
	
};
```

Of course, my new CPP class `MyUserWidget` has the macro `[ModuleName]_API` because I had not chosen any radio button. It was left as `UserDefined` and `UserDefined` is usually treated like `Public`. Then, the new CPP class would not have the macro if you clicked `Private` at the dialog.

{% asset_img 06.png %}

Click `Create Class`. Engine will create intermediate files, generate project files, and build source codes.

{% asset_img 08.png %}

After that, the new CPP class is ready for you.

{% asset_img 17.png %}

FYI, remove `[ProjectRoot]/Binaries` folder and build again if you meet a dialog like above while opening the editor.

# _`Creating CPP class; method #2`_

At the method #1, you must wait for a moment while engine does a process; creating intermediate files, generating project files, and build source codes. The process of creating new CPP class is not expensive when your project is small enough, but every project gets bigger and bigger as time goes on. When it comes to the point, you would want create multiple new CPP classes and wait for only one moment. At that time, the method #2 will be able to save you.

The method #2 for creating new CPP class is quite simple; do it yourself what engine did for you. Let me explain step by step. Suppose you want to create new CPP class inherits `UserWidget` class.

{% asset_img 09.png %}

Open your VisualStudio project. Find an location to add your new CPP class at `Solution Explorer`. I will add a class at `Unreal_5_0` folder. Select `Add/New Item....` at the option.

{% asset_img 10.png %}

Select `Header File` and name the file. I will name the file as `SomeUserWidget.h`. And click the button `Browse...` to locate the file. I will locate the file as the same location in `Solution Explorer`, `[ProjectRoot]/Source/Unreal_5_0`.

{% asset_img 11.png %}

After click the button `Add`, you can find the new file at both file explorer and `Solution Explorer` in VisualStudio IDE. Repeat previous steps for creating a cpp file.

{% asset_img 12.png %}

Then you have two files for creating new CPP class.

{% asset_img 13.png %}

But they have no contents, in other words, empty. So what ? Let us fill the contents manually. The cpp file is very simple as it has only an include statement, `#include "[HeaderName]"`. Problem is the header file. Usually, a generated header file from a class inherits `UObject` (or child of `UObject`) has a format like below:
```cpp
// Copyright notice

#pragma once

#include "CoreMinimal.h"
#include "[ParentClassHeaderFile]"
#include "[ThisClass].generated.h"

/** Comment for documentation
 *
 */
UCLASS()
class ([ModuleName]_API) U[ThisClass] : public U[ParentClass]
{
	GENERATED_BODY()
};
```

For instance, we had created a class `MyUserWidget`. The header file `MyUserWidget.h` has the contents like below:

```cpp
// Fill out your copyright notice in the Description page of Project Settings.

#pragma once

#include "CoreMinimal.h"
#include "Blueprint/UserWidget.h"
#include "MyUserWidget.generated.h"

/**
 * 
 */
UCLASS()
class UNREAL_5_0_API UMyUserWidget : public UUserWidget
{
	GENERATED_BODY()
};
```

FYI, the part `[ModuleName]_API` is optional as I explained at the method #1.

{% asset_img 14.png %}

Then, we can write down some codes for `SomeUserWidget`. They look like above. Alright, now we should generate intermediate files and project files. And then build the source codes.

{% asset_img 15.png %}

For this, close your VisualStudio IDE. Right click the uproject file and select `Generate Visual Studio project files`.

{% asset_img 16.png %}

Open your VisualStudio project after generation ends. And build the editor. The engine will generate intermediate files such as `generated.h` and `gen.cpp`. For more details about generating intermediate files, visit [this post](https://baemincheon.github.io/2021/08/06/how-unreal-macro-generated/).

{% asset_img 18.png %}

Now build ended. Let us open the editor. We can see new class `SomeUserWidget` well.

# _`Removing CPP class`_

{% asset_img 07.png %}

As you can see, you cannot select `Delete` at the option about CPP class in editor. Then, how we can remove a class when we do not need it ? It is quite simple, but you cannot do it in editor.

{% asset_img 19.png %}

Close your editor and remove files for the class you want to remove. I will remove the files for the class `SomeUserWidget`.

{% asset_img 20.png %}

And then generate project files via uproject file. Plus, you must remove `[ProjectRoot]/Binaries` folder.

{% asset_img 21.png %}

Open your VisualStudio project and build editor.

{% asset_img 22.png %}

Now you can see the class `SomeUserWidget` disappeared.

{% asset_img 23.png %}

Still the intermediate files could be remained. Remove `[ProjectRoot]/Intermediate` folder and repeat the steps.