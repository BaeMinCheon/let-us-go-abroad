---
title: Differences between Build.cs and Target.cs in UnrealEngine
date: 2021-04-07 22:52:46
tags: [UnrealEngine, Build.cs, Target.cs]
---

- this post covers
    - what does `Build.cs` do
    - what does `Target.cs` do
    - differences between `Build.cs` and `Target.cs`
- environment
    - Unreal Engine 4 `ver. 4.25`
    - Visual Studio 2019 `ver. 16.9.1`
- reference
    1. [https://docs.unrealengine.com/en-US/ProductionPipelines/BuildTools/UnrealBuildTool/ModuleFiles/index.html][reference #1]
    2. [https://docs.unrealengine.com/en-US/ProductionPipelines/BuildTools/UnrealBuildTool/TargetFiles/index.html][reference #2]

[reference #1]: https://docs.unrealengine.com/en-US/ProductionPipelines/BuildTools/UnrealBuildTool/ModuleFiles/index.html
[reference #2]: https://docs.unrealengine.com/en-US/ProductionPipelines/BuildTools/UnrealBuildTool/TargetFiles/index.html

### *`UnrealBuildTool.ModuleRules`*
UnrealEngine provides its own module system, which is absolutely different with [CPP 20 Module](https://en.cppreference.com/w/cpp/language/modules). The class `UnrealBuildTool.ModuleRules` is for the module system and it is written by `[ModuleName].Build.cs`. You can decide what to include for creating output files (= DLL). For example, `ShaderCompileWorker` project has the module rules below:

```csharp
/Engine/Source/Programs/ShaderCompileWorker/ShaderCompileWorker.Build.cs

public class ShaderCompileWorker : ModuleRules
{
	public ShaderCompileWorker(ReadOnlyTargetRules Target) : base(Target)
	{
		PrivateDependencyModuleNames.AddRange(
			new string[] {
				"Core",
				"Projects",
				"RenderCore",
				"SandboxFile",
				"TargetPlatform",
				"ApplicationCore",
				"TraceLog",
				"ShaderCompilerCommon"
			});

		if (Target.Platform == UnrealTargetPlatform.Linux)
		{
			PrivateDependencyModuleNames.AddRange(
			new string[] {
				"NetworkFile",
				"PakFile",
				"StreamingFile",
				});
		}

		PrivateIncludePathModuleNames.AddRange(
			new string[] {
				"Launch",
				"TargetPlatform",
			});

		PrivateIncludePaths.Add("Runtime/Launch/Private");      // For LaunchEngineLoop.cpp include

		// Include D3D compiler binaries
		string EngineDir = Path.GetFullPath(Target.RelativeEnginePath);

		if (Target.Platform == UnrealTargetPlatform.Win32)
		{
			RuntimeDependencies.Add(EngineDir + "Binaries/ThirdParty/Windows/DirectX/x86/d3dcompiler_47.dll");
		}
		else if (Target.Platform == UnrealTargetPlatform.Win64)
		{
			RuntimeDependencies.Add(EngineDir + "Binaries/ThirdParty/Windows/DirectX/x64/d3dcompiler_47.dll");
		}
	}
}
```

{% asset_img 07.png %}
</br>
{% asset_img 08.png %}

There are several libraries such as `Core`, `Projects`, `RenderCore` and so on. We can also find them in `Binaries` folder like below:
(FYI, the `ShaderCompileWorker.exe` is created with `ShaderCompileWorker.Target.cs` not the `ShaderCompileWorker.Build.cs`.)

{% asset_img 04.png %}

In other words, output files are created with the name containing its module when you add corresponding libraries to module rules (ex: `[TargetName]-[ModuleName]-[ConfigurationName].dll`). Additionally, UnrealEngine re-uses them as possible. Suppose your project need some libraries already included on engine side. In this situation, UnrealEngine does not create output files for the duplicated libraries included in your project. Instead of that, UnrealEngine leaves some meta file describes what the project included.

```csharp
// Copyright Epic Games, Inc. All Rights Reserved.

using UnrealBuildTool;

public class ThirdPerson_4_25 : ModuleRules
{
	public ThirdPerson_4_25(ReadOnlyTargetRules Target) : base(Target)
	{
		PCHUsage = PCHUsageMode.UseExplicitOrSharedPCHs;

		PublicDependencyModuleNames.AddRange(new string[] { "Core", "CoreUObject", "Engine", "InputCore", "HeadMountedDisplay" });
	}
}
```

{% asset_img 05.png %}

This is a generated module rules based on third person template. This module rules contains `CoreUObject` library but we cannot find it in `Binaries` folder. Let us see the `ThirdPerson_4_25Editor.target` file.

```
/Project/Binaries/Win64/ThirdPerson_4_25Editor.target

...
"BuildProducts": [
	...
	{
		"Path": "$(EngineDir)/Binaries/Win64/UE4Editor-CoreUObject.dll",
		"Type": "DynamicLibrary"
	},
	{
		"Path": "$(EngineDir)/Binaries/Win64/UE4Editor-CoreUObject.pdb",
		"Type": "SymbolFile"
	},
...
```

```cpp
/Engine/Source/Programs/UnrealBuildTool/System/TargetReceipt.cs

...
/// <summary>
/// Write the receipt to disk.
/// </summary>
/// <param name="Location">Output filename</param>
/// <param name="EngineDir">Engine directory for expanded paths</param>
public void Write(FileReference Location, DirectoryReference EngineDir)
{
	...
	Writer.WriteArrayStart("BuildProducts");
	foreach (BuildProduct BuildProduct in BuildProducts)
	{
		Writer.WriteObjectStart();
		Writer.WriteValue("Path", InsertPathVariables(BuildProduct.Path, EngineDir, ProjectDir));
		Writer.WriteValue("Type", BuildProduct.Type.ToString());
		Writer.WriteObjectEnd();
	}
	Writer.WriteArrayEnd();
	...
```

For more details about the module system, visit [reference #1].

### *`UnrealBuildTool.TargetRules`*
UnrealEngine provides its own target system, which makes you can create an executable. There are various target configurations such as `Editor`, `Client` and `Server`. The class `UnrealBuildTool.TargetRules` is for the target system and it is written by `[TargetName].Target.cs`. You can decide which modules to include for a certain target. For example, `UE4` project has the target rules for editor below:

```csharp
// Copyright Epic Games, Inc. All Rights Reserved.

using UnrealBuildTool;
using System.Collections.Generic;

public class UE4EditorTarget : TargetRules
{
	public UE4EditorTarget( TargetInfo Target ) : base(Target)
	{
		Type = TargetType.Editor;
		BuildEnvironment = TargetBuildEnvironment.Shared;
		bBuildAllModules = true;
		ExtraModuleNames.Add("UE4Game");
	}
}
```

</br>

```csharp
ExtraModuleNames.Add("UE4Game");
```

The target rules specifies `UE4Game` module included, which is located in `Engine/Source/Runtime/UE4Game`. So, output files for editor are consist of modules in `UE4Game`.

```csharp
// Copyright Epic Games, Inc. All Rights Reserved.

using UnrealBuildTool;

public class UE4Game : ModuleRules
{
	public UE4Game(ReadOnlyTargetRules Target) : base(Target)
	{
		PrivateDependencyModuleNames.Add("Core");

		if (Target.Platform == UnrealTargetPlatform.IOS || Target.Platform == UnrealTargetPlatform.TVOS)
		{
			PrivateDependencyModuleNames.AddRange(new string[] { "Core", "CoreUObject", "Engine" });
			if (Target.Platform == UnrealTargetPlatform.IOS)
			{
				DynamicallyLoadedModuleNames.Add("IOSAdvertising");
			}
		}
		else if (Target.Platform == UnrealTargetPlatform.Android)
		{
			PrivateDependencyModuleNames.AddRange(new string[] { "Core", "CoreUObject", "Engine" });
			DynamicallyLoadedModuleNames.Add("AndroidAdvertising");
		}
	}
}
```

</br>

{% asset_img 09.png %}

Yes, they are. :)

```csharp
Type = TargetType.Editor;
```

`UnrealBuildTool.TargetRules` has a field `Type`.

```csharp
/Engine/Source/Programs/UnrealBuildTool/Configuration/TargetRules.cs

...csharp
/// <summary>
/// TargetRules is a data structure that contains the rules for defining a target (application/executable)
/// </summary>
public abstract partial class TargetRules
{
	...
	/// <summary>
	/// The type of target.
	/// </summary>
	public global::UnrealBuildTool.TargetType Type = global::UnrealBuildTool.TargetType.Game;
	...
```

The field is used for branching target-specific features such as build configuration. For example, UnrealEngine manages target configurations as `enum`.

```csharp
/Engine/Source/Programs/UnrealBuildTool/Configuration/UEBuildTarget.cs

...
/// <summary>
/// The type of configuration a target can be built for
/// </summary>
public enum UnrealTargetConfiguration
{
	/// <summary>
	/// Unknown
	/// </summary>
	Unknown,

	/// <summary>
	/// Debug configuration
	/// </summary>
	Debug,

	/// <summary>
	/// DebugGame configuration; equivalent to development, but with optimization disabled for game modules
	/// </summary>
	DebugGame,

	/// <summary>
	/// Development configuration
	/// </summary>
	Development,

	/// <summary>
	/// Shipping configuration
	/// </summary>
	Shipping,

	/// <summary>
	/// Test configuration
	/// </summary>
	Test,
}
...
```

Unlike others, the editor target support only 3 types of target configuration. `Debug`, `DebugGame` and `Development`.

```csharp
/Engine/Source/Programs/UnrealBuildTool/Configuration/TargetRules.cs

...
/// <summary>
/// Gets a list of configurations that this target supports
/// </summary>
/// <returns>Array of configurations that the target supports</returns>
internal UnrealTargetConfiguration[] GetSupportedConfigurations()
{
	// Otherwise take the SupportedConfigurationsAttribute from the first type in the inheritance chain that supports it
	for (Type CurrentType = GetType(); CurrentType != null; CurrentType = CurrentType.BaseType)
	{
		object[] Attributes = CurrentType.GetCustomAttributes(typeof(SupportedConfigurationsAttribute), false);
		if (Attributes.Length > 0)
		{
			return Attributes.OfType<SupportedConfigurationsAttribute>().SelectMany(x => x.Configurations).Distinct().ToArray();
		}
	}

	// Otherwise, get the default for the target type
	if (Type == TargetType.Editor)
	{
		return new[] { UnrealTargetConfiguration.Debug, UnrealTargetConfiguration.DebugGame, UnrealTargetConfiguration.Development };
	}
	else
	{
		return ((UnrealTargetConfiguration[])Enum.GetValues(typeof(UnrealTargetConfiguration))).Where(x => x != UnrealTargetConfiguration.Unknown).ToArray();
	}
}
...
```

</br>

{% asset_img 10.png %}

For more details, visit [reference #2].

### *`[ModuleName].Build.cs`*
Each module has its own `Build.cs` file. For example, a `[ProjectName].Build.cs` will be generated when you create new project with cpp enabled. Because UnrealEngine makes a default module that has the same name with project. (Exactly, `Build.cs` and `Target.cs` files are copied from template in general cases.)

```cpp
/Engine/Source/Editor/GameProjectGeneration/Private/GameProjectUtils.cpp

bool GameProjectUtils::CreateProject(const FProjectInformation& InProjectInfo, FText& OutFailReason, FText& OutFailLog, TArray<FString>* OutCreatedFiles)
{
	if ( !IsValidProjectFileForCreation(InProjectInfo.ProjectFilename, OutFailReason) )
	{
		return false;
	}

	FScopedSlowTask SlowTask(0, LOCTEXT( "CreatingProjectStatus", "Creating project..." ));
	SlowTask.MakeDialog();

	TOptional<FGuid> ProjectID;
	FString TemplateName;
	if ( InProjectInfo.TemplateFile.IsEmpty() )
	{
		ProjectID = GenerateProjectFromScratch(InProjectInfo, OutFailReason, OutFailLog);
		TemplateName = InProjectInfo.bShouldGenerateCode ? TEXT("Basic Code") : TEXT("Blank");
	}
	else
	{
		ProjectID = CreateProjectFromTemplate(InProjectInfo, OutFailReason, OutFailLog, OutCreatedFiles);
		TemplateName = FPaths::GetBaseFilename(InProjectInfo.TemplateFile);
	}
...
```

```cpp
/Engine/Source/Editor/GameProjectGeneration/Private/GameProjectUtils.cpp

TOptional<FGuid> GameProjectUtils::CreateProjectFromTemplate(const FProjectInformation& InProjectInfo, FText& OutFailReason, FText& OutFailLog, TArray<FString>* OutCreatedFiles)
{
	...

	// Discover and copy all files in the src folder to the destination, excluding a few files and folders
	TArray<FString> FilesToCopy;
	TArray<FString> FilesThatNeedContentsReplaced;
	TMap<FString, FString> ClassRenames;
	IFileManager::Get().FindFilesRecursive(FilesToCopy, *SrcFolder, TEXT("*"), /*Files=*/true, /*Directories=*/false);

	...

	// Perform the copy
	const FString DestFilename = DestFolder / DestFileSubpathWithoutFilename + DestBaseFilename + TEXT(".") + FileExtension;
	if ( IFileManager::Get().Copy(*DestFilename, *SrcFilename) == COPY_OK )
	{
		CreatedFiles.Add(DestFilename);

		if ( ReplacementsInFilesExtensions.Contains(FileExtension) )
		{
			FilesThatNeedContentsReplaced.Add(DestFilename);
		}

		// Allow project template to extract class renames from this file copy
		if (FPaths::GetBaseFilename(SrcFilename) != FPaths::GetBaseFilename(DestFilename)
			&& TemplateDefs->IsClassRename(DestFilename, SrcFilename, FileExtension))
		{
			// Looks like a UObject file!
			ClassRenames.Add(FPaths::GetBaseFilename(SrcFilename), FPaths::GetBaseFilename(DestFilename));
		}
	}
	...
```

<br/>
{% asset_img 02.png %}
<br/>
{% asset_img 01.png %}

*WHEN YOU CREATE A PROJECT ITS NAME OF `ThridPerson_4_25` FROM THIRD PERSON TEMPLATE*

Saying that again, `[ModuleName].Build.cs` defines the dependencies for building its module. So, every module must have its own `[ModuleName].Build.cs` file and every module has its own `[ModuleName].Build.cs` will generate a DLL when you build the project.

Module generation can be done by manipulating some CSharp scripts (`Build.cs` and `Target.cs` files). You can find how at https://www.ue4community.wiki/creating-cpp-module-oshdsg2t.

### *`[TargetName].Target.cs`*
While every module must have a `Build.cs` file, but every module do not have to have a `Target.cs` file. Some modules have only `Build.cs` file. It means the modules should be used for library not a standalone. The `AIModule` is a good example. The module has only `Build.cs` as it is written for providing a support to make AI.

{% asset_img 11.png %}
</br>
{% asset_img 12.png %}
</br>
```csharp
/Engine/Source/Editor/UnrealEd/UnrealEd.Build.cs

...
PublicIncludePathModuleNames.AddRange(
	new string[] {
		"AssetRegistry",
		"AssetTagsEditor",
		"CollectionManager",
		"BlueprintGraph",
		"AddContentDialog",
		"MeshUtilities",
		"AssetTools",
		"KismetCompiler",
		"NavigationSystem",
		"GameplayTasks",
		"AIModule",
		"Engine",
		"SourceControl",
	}
);
...
```

```csharp
/Engine/Source/Runtime/Engine/Engine.Build.cs

...
if (Target.bBuildEditor == true)
{
	PublicDependencyModuleNames.AddRange(
		new string[] {
			"UnrealEd",
			"Kismet"
		}
	);	// @todo api: Only public because of WITH_EDITOR and UNREALED_API
...
```

```csharp
/Engine/Source/Developer/TargetPlatform/TargetPlatform.Build.cs

...
PrivateIncludePathModuleNames.Add("Engine");
...
```

```csharp
/Engine/Source/Runtime/Core/Core.Build.cs

...
PrivateIncludePathModuleNames.AddRange(
	new string[] {
		"TargetPlatform",
		"DerivedDataCache",
		"InputDevice",
		"Analytics",
		"RHI"
	}
);
...
```

```csharp
/Engine/Source/Runtime/UE4Game/UE4Game.Build.cs

...
PrivateDependencyModuleNames.Add("Core");
...
```

```csharp
/Engine/Source/UE4Editor.Target.cs

// Copyright Epic Games, Inc. All Rights Reserved.

using UnrealBuildTool;
using System.Collections.Generic;

public class UE4EditorTarget : TargetRules
{
	public UE4EditorTarget( TargetInfo Target ) : base(Target)
	{
		Type = TargetType.Editor;
		BuildEnvironment = TargetBuildEnvironment.Shared;
		bBuildAllModules = true;
		ExtraModuleNames.Add("UE4Game");
	}
}

```

Sometimes, some modules should not be included on certain target configurations. For instance, only editor features should not be included in client or server target configuration. In the need, we can branch for ease like below:

```csharp
Engine/Source/Developer/SlateReflector/SlateReflector.Build.cs

...
// Editor builds include SessionServices to populate the remote target drop-down for remote widget snapshots
if (Target.Type == TargetType.Editor)
{
	PublicDefinitions.Add("SLATE_REFLECTOR_HAS_SESSION_SERVICES=1");

	PrivateDependencyModuleNames.AddRange(
		new string[] {
			"PropertyEditor",
		}
	);

	PrivateIncludePathModuleNames.AddRange(
		new string[] {
			"SessionServices",
		}
	);

	DynamicallyLoadedModuleNames.AddRange(
		new string[] {
			"SessionServices",
		}
	);
}
else
{
	PublicDefinitions.Add("SLATE_REFLECTOR_HAS_SESSION_SERVICES=0");
}
...
```

We can use `Widget Relfector` only at editor target configuration. As we see, non-editor target will not contain the reflector feature. One more step, we can force to block generating projects by throwing an exception like this.

```csharp
/Engine/Source/Editor/UnrealEd/UnrealEd.Build.cs

...
public class UnrealEd : ModuleRules
{
	public UnrealEd(ReadOnlyTargetRules Target) : base(Target)
	{
		if(Target.Type != TargetType.Editor)
		{
			throw new BuildException("Unable to instantiate UnrealEd module for non-editor targets.");
		}
...
```

### *Wrap-Up*
`Build.cs` and `Target.cs` are different with each other on why they are used.
- `Build.cs`
	- defines how the module should be created.
	- type of module output file is usually `.dll`.
	- every module must have its own `[ModuleName].Build.cs`.
	- there can be duplicated output files if the module is included in multiple `Target.cs` files or if build configurations are different on each other.
		- ex: `Target1-ModuleA.dll`, `Target2-ModuleA.dll`, `Target2-ModuleA-Win64-Debug.dll` and etc.
- `Target.cs`
	- defines how the executable should be created.
	- type of target output file is usually `.exe`.
	- user can write `[TargetName].Target.cs` and modules can be included selectively.
	- there can be duplicated output files if build configurations are different on each other.
		- ex: `UE4Editor.exe`, `UE4Editor-Win64-Debug.exe` and etc.

Logically, a target is above one than a module.

{% asset_img 13.png %}