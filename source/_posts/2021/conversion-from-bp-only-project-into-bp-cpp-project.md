---
title: Conversion from BP-only project into BP+CPP project
date: 2021-11-22 23:18:38
tags: [UnrealEngine, Blueprint, CPP]
---

| Environment | |
| --- | --- |
| UnrealEngine | `branch: 5.0` |
| Visual Studio 2022 | `version: 17.0.1` |
| Windows 11 Pro | `build: 22000.318` |

# _`Overview`_

{% asset_img 01.png %}

UnrealEngine provides you two options to build your project and you can choose one of them. The options are `BLUEPRINT` and `C++` as you can see at the screenshot above. Selecting left one means that, "I gonna develop my project using only blueprint". Otherwise, selecting right one means, "I want to use both blueprint and cpp on my project".

By the way, what is different between them ? How can we convert BP only project into BP+CPP project ? Let us go over.

# _`Comparison`_

{% asset_img 02.png %}

After creation, you can see the directory if selected the BP-Only. In this project `BPOnly`, you only can execute UnrealEngine editor and write blueprints. Even if you make source code files and place them into appropriate position, your project does not compile the source code. Let us find out "why not working" by the difference between BP only project and BP+CPP project.

{% asset_img 03.png %}

After creation with `C++` selection. The `BPCPP` project supports both blueprint and cpp like its name. You can see the difference on number of files, `BPOnly` is 6 while `BPCPP` is 10. Files that exist only in `BPCPP` are here.

Name of file/folder | Description
--- | ---
`.vs` | Containing VisualStudio related files. Mostly, cached data for optimization.
`Binaries` | Containing output files of this project. Currently, this project's UnrealEditor library exists.
`Source` | Containing some simple source code files. Plus, BuildRule and TargetRule exist in this folder.
`<ProjectName>.sln` | Just like uproject file, it defines required version of VisualStudio, dependency of the project, and so on.

The only `Source` folder is not generated one. The `Binaries` folder is generated when you build the project with a certain target such as `WindowsClient`, `WindowsServer`, and `Editor`. The files related to VisualStudio are generated when you attempt to make project files. Also, UnrealEngine refers the `Source` folder while generating project files.

{% asset_img 04.png %}

So, is that all ? No, actually there is one more thing different. Check the uproject file and you can find some difference. The contents of uproject file looks like similar, but `BPCPP`'s one has a `Modules` property. The name of module is the same with project name, `BPCPP`.

{% asset_img 05.png %}

In summary, there are some differences between BP only project and BP+CPP project. (Except for generated files)

- Existence of `Source` folder
- Property `Modules` in uproject file

Where these differences come from ?

# _`Template`_
We have learned about templates used in UnrealEngine at [the post](https://baemincheon.github.io/2021/04/07/difference-between-build-cs-and-target-cs/). What found was that making new project from a template is equal to copying the template project and replacing placeholders. Right, then it would be similar to that. Find the template project for BP only project and BP+CPP project.

```cpp
TMap<FName, TArray<TSharedPtr<FTemplateItem>> > SProjectDialog::FindTemplateProjects()
{
    // Clear the list out first - or we could end up with duplicates
    TMap<FName, TArray<TSharedPtr<FTemplateItem>>> Templates;

    // Now discover and all data driven templates
    TArray<FString> TemplateRootFolders;

    // @todo rocket make template folder locations extensible.
    TemplateRootFolders.Add(FPaths::RootDir() + TEXT("Templates"));

    // Add the Enterprise templates
    TemplateRootFolders.Add(FPaths::EnterpriseDir() + TEXT("Templates"));

    // Allow plugins to define templates
    TArray<TSharedRef<IPlugin>> Plugins = IPluginManager::Get().GetEnabledPlugins();
    for (const TSharedRef<IPlugin>& Plugin : Plugins)
    {
        FString PluginDirectory = Plugin->GetBaseDir();
        if (!PluginDirectory.IsEmpty())
        {
            const FString PluginTemplatesDirectory = FPaths::Combine(*PluginDirectory, TEXT("Templates"));

            if (IFileManager::Get().DirectoryExists(*PluginTemplatesDirectory))
            {
                TemplateRootFolders.Add(PluginTemplatesDirectory);
            }
        }
    }
...
```

As you see, UnrealEngine finds template files from the path; `Root/Templates/`.

{% asset_img 06.png %}

There are many folders for each template, and now we found. The `TP_Blank` and `TP_BlankBP. The templates contain a uproject file, which is used for making new uproject file while creating new project using template.

{% asset_img 07.png %}

The `BPOnly.uproject` was created based on `TP_BlankBP.uproject`. You can check that at the `FProjectDescriptor::Write()` function.

{% asset_img 09.png %}
</br>
{% asset_img 10.png %}
</br>

```cpp
void FModuleDescriptor::WriteArray(TJsonWriter<>& Writer, const TCHAR* ArrayName, const TArray<FModuleDescriptor>& Modules)
{
    if (Modules.Num() > 0)
    {
        Writer.WriteArrayStart(ArrayName);
        for(const FModuleDescriptor& Module : Modules)
        {
            Module.Write(Writer);
        }
        Writer.WriteArrayEnd();
    }
}
```

Why the `Modules` property not copied ? Look at the `FModuleDescriptor::WriteArray()`. UnrealEngine does not write that property when it is empty.

```cpp
bool GameProjectUtils::SetEngineAssociationForForeignProject(const FString& ProjectFileName, FText& OutFailReason)
{
    if(FUProjectDictionary(FPaths::RootDir()).IsForeignProject(ProjectFileName))
    {
        if(!FDesktopPlatformModule::Get()->SetEngineIdentifierForProject(ProjectFileName, FDesktopPlatformModule::Get()->GetCurrentEngineIdentifier()))
        {
            OutFailReason = LOCTEXT("FailedToSetEngineIdentifier", "Couldn't set engine identifier for project");
            return false;
        }
    }
    return true;
}
```
</br>
{% asset_img 11.png %}

Why the `EngineAssociation` property not filled ? That property is filled later at the `FDesktopPlatformBase::SetEngineIdentifierForProject()` function.

{% asset_img 08.png %}

Of course, the `BPCPP.uproject` was created based on `TP_Blank.uproject`. In this case, whole contents of file copied. And, the `EngineAssociation` would be overwritten.

```cpp
// Retarget any files that were chosen to have parts of their names replaced here
FString DestBaseFilename = FPaths::GetBaseFilename(SrcFileSubpath);
const FString FileExtension = FPaths::GetExtension(SrcFileSubpath);
for ( const FTemplateReplacement& Replacement : TemplateDefs->FilenameReplacements )
{
    if ( Replacement.Extensions.Contains( FileExtension ) )
    {
        // This file matched a filename replacement extension, apply it now
        FString LastDestBaseFilename = DestBaseFilename;
        DestBaseFilename = DestBaseFilename.Replace(*Replacement.From, *Replacement.To, Replacement.bCaseSensitive ? ESearchCase::CaseSensitive : ESearchCase::IgnoreCase);

        if (LastDestBaseFilename != DestBaseFilename)
        {
            UE_LOG(LogGameProjectGeneration, Verbose, TEXT("'%s': Renaming to '%s/%s' as it matched file rename ('%s'->'%s')"), *SrcFilename, *DestFileSubpathWithoutFilename, *DestBaseFilename, *Replacement.From, *Replacement.To);
        }
    }
}
...
// Open all files with the specified extensions and replace text
for ( const FString& FileToFix : FilesThatNeedContentsReplaced )
{
    InnerSlowTask.EnterProgressFrame();

    bool bSuccessfullyProcessed = false;

    FString FileContents;
    if ( FFileHelper::LoadFileToString(FileContents, *FileToFix) )
    {
        for ( const FTemplateReplacement& Replacement : TemplateDefs->ReplacementsInFiles )
        {
            if ( Replacement.Extensions.Contains( FPaths::GetExtension(FileToFix) ) )
            {
                FileContents = FileContents.Replace(*Replacement.From, *Replacement.To, Replacement.bCaseSensitive ? ESearchCase::CaseSensitive : ESearchCase::IgnoreCase);
            }
        }

        if ( FFileHelper::SaveStringToFile(FileContents, *FileToFix) )
        {
            bSuccessfullyProcessed = true;
        }
    }

    if ( !bSuccessfullyProcessed )
    {
        FFormatNamedArguments Args;
        Args.Add( TEXT("FileToFix"), FText::FromString( FileToFix ) );
        OutFailReason = FText::Format( LOCTEXT("FailedToFixUpFile", "Failed to process file \"{FileToFix}\"."), Args );
        return TOptional<FGuid>();
    }
}
```

The name of folders and content of files are replaced by the codes above. In this post, from `TP_Blank` into `BPCPP`. (Or, from `TP_BlankBP` into `BPOnly`)

{% asset_img 12.png %}

# _`Module`_
We have confirmed that the difference between `BPOnly` and `BPCPP` is about a module system, which are `Modules` property in uproject and `Source` folder containing code files. Thus, it would be possible converting blueprint only project into blueprint with cpp project by making some changes. In other words, we should make a new module.

Though [a good wiki page for this](https://unrealcommunity.wiki/creating-cpp-module-oshdsg2t) exists, I will show you an example based on `TP_BlankBP` template.

{% asset_img 02.png %}

#1. Prepare a project created with `TP_BlankBP`. In this post, I use the `BPOnly` project.

{% asset_img 13.png %}

#2. Make a folder `Source` at project directory, and make a folder `<ModuleName>` in the `Source` directory.

Name the module as you want, but it is recommended to set by project name. (Because this module is the first module of project) Just to show that any name is okay, I set the module name as `Robb`, which is different with project name.

{% asset_img 14.png %}
</br>
{% asset_img 15.png %}

#3. Copy some files from the template `TP_Blank`. Replace their names and contents.

I had copied all of files in `Source` folder of `TP_Blank` template. For using the template files in this project, I replaced filenames and contents. (In this case, I need to replace the text `TP_Blank` into `Robb`)

{% asset_img 16.png %}

#4. Generate VisualStudio project files and open VisualStudio project file.

{% asset_img 17.png %}
</br>
{% asset_img 18.png %}

#5. Build the project and open UnrealEngine editor. Profit !

# _`Wrap-Up`_
It is not common case that creating a project with blueprint only option, but we are able to convert blueprint only project into blueprint with cpp project. We have checked what happens while creating our project using template, what is different between `TP_Blank` and `TP_BlankBP`, and how to add cpp module at blueprint only project. As we seen earlier in this post, the conversion we did is the same work of what UnrealEngine does.

When we make an initial module, the name of module does not have to be the same with project name. But, it is recommended to set by project name with convention and several reasons. For example, I had made a module `Robb` at the project `BPOnly`. I tried to package the project and got the result like below. Some of files have the name as `Robb`, but others have the name as `BPOnly`. Kind of disharmony on naming could be problem when accessing files with name.

{% asset_img 19.png %}
</br>
{% asset_img 20.png %}