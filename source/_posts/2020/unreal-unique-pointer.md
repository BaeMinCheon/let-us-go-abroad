---
title: UnrealEngine unique pointer
date: 2020-03-14 21:47:31
tags: [UnrealEngine, SmartPointer, UniquePointer]
---

- this post covers
    - what is unique pointer in unreal engine
    - how it is used
    - restriction and caution on usage
- environment
    - Unreal Engine (4.24)
    - Visual Studio 2017
    - Windows 10
- reference
    1. [https://docs.unrealengine.com/en-US/Programming/UnrealArchitecture/SmartPointerLibrary/index.html][reference #1]

[reference #1]: https://docs.unrealengine.com/en-US/Programming/UnrealArchitecture/SmartPointerLibrary/index.html

---

# *overview*

as unreal engine handles objects inherits `UObject`, we can use unreal engine easily. there are several benefits when the object inherits `UObject`.

- garbage collection
- reference update
- reflection
- serialization
- etc.

then you might have one question,
"how we can handle the objects does not inherit `UObject` ? should we use raw pointer for the objects ?"
well...there are 2 ways for this.

1. using `std::unique_ptr` of cpp std library in `memory.h`
2. using `TUniquePtr` of unreal API in `UniquePtr.h`

you can use `std::unique_ptr` in unreal project, but unreal engine implements their own smart pointer library. and it is common that using `TUniquePtr` in unreal project unless you do not need cpp std library.

as purpose and functionality are the same, `TUniquePtr` is similar to `std::unique_ptr`. `TUniquePtr` also provides the unique ownership and other features. let us check out what it is and how it is used.

---

# *built-in example*

you can find some example in unreal engine code.

```cpp
UnrealEngine/Engine/Source/Runtime/SandboxFile/Public/IPlatformFileSandboxWrapper.h

class SANDBOXFILE_API FSandboxPlatformFile : public IPlatformFile
{
    ....
};
```

class `FSandboxPlatformFile` is not a class inherits `UObject` and it is possible to be indicated with `TUniquePtr`.
( conventionally, prefix `U` is attached when the class inherits `UObject` )

```cpp
UnrealEngine/Engine/Source/Editor/UnrealEd/Classes/CookOnTheSide/CookOnTheFlyServer.h

UCLASS()
class UNREALED_API UCookOnTheFlyServer : public UObject, public FTickableEditorObject, public FExec
{
    ....

    TUniquePtr<class FSandboxPlatformFile> SandboxFile;

    ....
};
```

class `UCookOnTheFlyServer` is a class inhertis `UObject` and it contains `TUniquePtr` with `FSandboxPlatformFile` as member variable.

```cpp
UnrealEngine/Engine/Source/Editor/UnrealEd/Private/CookOnTheFlyServer.cpp

void UCookOnTheFlyServer::PopulateCookedPackagesFromDisk(const TArray<ITargetPlatform*>& Platforms)
{
    ....
    
    FString EngineSandboxPath = SandboxFile->ConvertToSandboxPath(*FPaths::EngineDir()) + TEXT("/");

    ....
}
```

accessing the object through `TUniquePtr` is the same on `std::unique_ptr`. using `->` operator, you can access the object as the normal pointer.

```cpp
UnrealEngine/Engine/Source/Editor/UnrealEd/Private/CookOnTheFlyServer.cpp

FString UCookOnTheFlyServer::ConvertToFullSandboxPath( const FString &FileName, bool bForWrite ) const
{
    check(SandboxFile);

    ....
}
```

validation on `TUniquePtr` is the same on raw pointer. if the value is zero, the `TUniquePtr` is pointing `nullptr`. you can use `check` for ensuring whether the `TUniquePtr` is valid.

```cpp
UnrealEngine/Engine/Source/Editor/UnrealEd/Private/Commandlets/AssetRegistryGenerator.cpp

bool FAssetRegistryGenerator::SaveManifests(FSandboxPlatformFile* InSandboxFile, int64 InExtraFlavorChunkSize)
{
    ....
}
```

`FAssetRegistryGenerator::SaveManifests` gets `FSandboxPlatformFile*` as one of parameters. the type of parameter is not the `TUniquePtr`, so we should convert `TUniquePtr<T>` into `T*`.

```cpp
UnrealEngine/Engine/Source/Editor/UnrealEd/Private/CookOnTheFlyServer.cpp

void UCookOnTheFlyServer::CookByTheBookFinished()
{
    ....

    Generator.SaveManifests(SandboxFile.Get());

    ....
}
```

use the `TUniquePtr::Get()` if you need the raw pointer of `TUniquePtr<T>`.

```cpp
UnrealEngine/Engine/Source/Editor/UnrealEd/Private/CookOnTheFlyServer.cpp

void UCookOnTheFlyServer::CancelCookByTheBook()
{
    ....

    SandboxFile = nullptr;

    ....
}
```

assigning `nullptr` into `TUniquePtr`, you can let `TUniquePtr` release the object and have its value as `nullptr`.

```cpp
UnrealEngine/Engine/Source/Runtime/SandboxFile/Public/IPlatformFileSandboxWrapper.h

class SANDBOXFILE_API FSandboxPlatformFile : public IPlatformFile
{
    ....
    
    FSandboxPlatformFile(bool bInEntireEngineWillUseThisSandbox = false);

    ....
}
```

constructor of `FSandboxPlatformFile` takes one parameter as boolean.

```cpp
UnrealEngine/Engine/Source/Editor/UnrealEd/Private/CookOnTheFlyServer.cpp

void UCookOnTheFlyServer::CreateSandboxFile()
{
    ....

    SandboxFile = MakeUnique<FSandboxPlatformFile>(false);

    ....
}
```

`MakeUnique` returns `TUniquePtr` object and calls the constructor of the template class. in this case, `MakeUnique<FSandboxPlatformFile>` takes one boolean value.

---

# *restriction and caution*

suppose you have a class like below

```cpp
class UserInfo
{
public:
    UserInfo(int32 NewUserID, char* NewUserName)
        : UserID(NewUserID), UserName(NewUserName), bInitialized(false)
    {
        bInitialized = true;
    }
    virtual ~UserInfo()
    {
        if (UserName != nullptr)
        {
            delete[] UserName;
        }

        bInitialized = false;
    }

private:
    int32 UserID;
    char* UserName;
    bool bInitialized;
};
```

you should only use `TUniquePtr` at the object, which exists only one thing. unless, you would get an exception for delete on `nullptr`.

```cpp
void SomeUActor::BeginPlay()
{
    ....

    CreateUserInformation();
}

void SomeUActor::EndPlay(const EEndPlayReason::Type EndPlayReason)
{
    ....

    ReleaseUserInformation();
}

void SomeUActor::CreateUserInformation()
{
    char* String = new char[100];
    strcpy(String, "baemincheon");
    UserInformation = MakeUnique<UserInfo>(0, String);

    TUniquePtr<UserInfo> AnotherPtr(UserInformation.Get());
}

void SomeUActor::ReleaseUserInformation()
{
    UserInformation = nullptr;
}
```

in this code, `CreateUserInformation` will be called in `BeginPlay` and `ReleaseUserInformation` will be called in `EndPlay`. `UserInformation` is a memeber variable with `TUniquePtr<UserInfo>` type. another `TUniquePtr<UserInfo>` exists in `CreateUserInformation`, which gets `UserInfo*`.

{% asset_img 02.jpg %}

what happens when `CreateUserInformation` is ended ? `AnotherPtr` would disappear and its destructor would be called. when the destructor of `TUniquePtr` is called, it releases memory that `TUniquePtr` has pointed. as a result, variable `UserInformation` would be a dangling pointer.

{% asset_img 03.jpg %}

`UserInformation` has abnormal values.

{% asset_img 01.jpg %}

because already the memory is released, an exception would be thrown when we execute `ReleaseUserInformation`. it is why you have to use `TUniquePtr` at the object only existing one thing and care about moving the ownership. moving the ownership with raw pointer is dangerous as we have seen.

```cpp
void SomeUActor::CreateUserInformation()
{
    char* String = new char[100];
    strcpy(String, "baemincheon");
    UserInformation = MakeUnique<UserInfo>(0, String);

    TUniquePtr<UserInfo> AnotherPtr(MoveTemp(UserInformation));
}
```

let us move the ownership with `MoveTemp` API. this code makes `AnotherPtr` point the memory and `UserInformation` set the `nullptr`.

{% asset_img 04.jpg %}

`UserInformation` has `nullptr`. so we can avoid the exception.

---

# *summary*

`TUniquePtr` is similar to `std::unique_ptr` and its usage and restriction, too.

- initialization: you can initialize `TUniquePtr` with 2 ways

type | method
- | -
using raw pointer | `TUniquePtr<T> PointerA(T*);`
using other `TUniquePtr`| `TUniquePtr<T> PointerA(MoveTemp(PointerB));`

- transfering ownership: for preventing side effect, you should use `MoveTemp`

```cpp
TUniquePtr<T> PointerA = new T(...);
TUniquePtr<T> PointerB;
PointerB = MoveTemp(PointerA);
```

- release: there are various ways to release the `TUniquePtr`

```cpp
// way #1 : release implicitly
PointerA = nullptr;

// way #2 : release explicitly
PointerB.Release();

{   // way #3 : using scope
    TUniquePtr<T> PointerC = new T(...);
    ....
}   // the destructor of PointerC is called when this block ends
```