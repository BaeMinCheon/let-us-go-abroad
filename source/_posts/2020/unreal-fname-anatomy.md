---
title: UnrealEngine FName anatomy
date: 2020-01-17 00:09:22
tags: [UnrealEngine]
---

- this post covers
    - what is `FName` in Unreal Engine
    - how `FName` works
- environment
    - _Custom_ Unreal Engine (based on 4.24)
    - Visual Studio 2017
    - Windows 10
- reference
    - https://docs.unrealengine.com/en-US/Programming/UnrealArchitecture/StringHandling/FName/index.html
    - https://docs.unrealengine.com/en-US/API/Runtime/Core/UObject/FScriptName/index.html
    - https://dq8iqaixvew1d.cloudfront.net/en-US/Programming/Assets/Registry/index.html?lang=zh-CN

---

### what `FName` is
- one of the 3 main string types in Unreal Engine, which is used for indicating individual object in game
- as its purpose, `FName` do not have features for string manipulation such as trim, reverse...etc.
- it would be better to consider `FName` is a literal index rather than a digital index

### case-insensitive
- you may have seen the expression, "FNames are case-insensitive", which means you cannot make more than one `FName` with the same alphabet arrangement
- for example, if you already have an object its name is "Sheri", you cannot make an object its name is "sheri". let us see below:

{% asset_img 01.jpg %}

- why this happens ? find it out in the code

{% asset_img 02.jpg %}

- we can find the phrase at the `ContentBrowserUtils::IsValidObjectPathForCrate()`
    - the function validates whether new object can be created in certain directory
- as the comment says, the phrase is shown when there is already an object with the same path
    - in other words, you can create an object with the same name in different path

{% asset_img 03.jpg %}

- yes, not only object, folder is also affected with the rule

### `FName` as a hash value
- now, you should have a question:
    - _"how does Unreal Engine save the object path ?"_
- we can assume that may be not literal method, rather hash method
    - because literal method would tell us that upper case differs from lower case

{% asset_img 04.jpg %}

- digging one step, we can find something strange
- as we saw, the warning pharse is shown when there is already an object with the same path
    - we can see returning some object ( that is not `nullptr` ) in picture above
- `ObjectPath` does not have the identical indices
    - especially, `ComparisonIndex` and `DisplayIndex`
    - strangefully, `DisplayIndex` indicates the intended string, other does not

※ more information about the indices in #2 reference

{% asset_img 05.jpg %}
</br>
{% asset_img 06.jpg %}

- digging one step more, `ComparisonIndex` is set by one of `FName` that is created earlier
    - it may be confusing, let us summary as table

String | ComparisonIndex
- | -
`Sheri` (AssetName) | 884751
`/Game/ThirdPersonCPP/Blueprints/Sheri.Sheri` (ObjectPath) | 884754
`sheri` (AssetName) | 886628
`/Game/ThirdPersonCPP/Blueprints/sheri.sheri` (ObjectPath) | 886631

※ the value of index is not consistent. it can have a different value each time
※ more information about the names and pathes in #3 reference

### lower-case string compare
- digging one step more !!
    - why did later one set by earlier one ?
- let us look at `StaticFindObjectFastInternalThreadSafe()`
    - the function finds and returns an certain object(or package) depending on parameters

{% asset_img 07.jpg %}

- at line #588, there is an `==` operation between two `FName`s
    - if they are the same, the expression must become true
- then, we got the one last destination `FName::operator==()`
    - actually, we should go to `StrnicmpImpl()` due to the callstack below
    - `FName::operator==() → FNameHelper::EqualString() → StringAndNumberEqualsString() → FPlatformString::Strnicmp() → StrnicmpImpl()`

{% asset_img 08.jpg %}

- in this function, each character of each string is compared in lower case
- now we understand why `Sheri` and `sheri` are identical in using `FName`

### wrap-up
- `FName` is not identified with case
    - we could see it with `Sheri` and `sheri` example
    - every `FName` is compared in lower case
- `FName` is used with index format
    - for effectiveness, real string is cached and is accessed with index
    - there are two indices, `ComparisonIndex` and `DisplayIndex`
- `FName` duplication test is relying on path name
    - you can have two objects such as `/Game/Sheri/Umbrella` and `/Game/Donita/Umbrella`
    - this means that the same `AssetName` can exist multiple times in one project