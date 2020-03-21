---
title: unreal-cpp-ctor-and-dtor
date: 2020-03-01 21:15:46
tags: [Unreal]
---

- this post covers
    - what do cpp ctor and dtor do in unreal engine
    - what is the `CDO` in unreal engine
    - what are the ctor and dtor of `UCLASS`
    - difference between cpp ctor/dtor and `UCLASS` ctor/dtor
- environment
    - Unreal Engine (4.24)
    - Visual Studio 2017
    - Windows 10
- reference
    1. [https://docs.unrealengine.com/en-US/Programming/UnrealArchitecture/Objects/index.html][reference #1]
    2. [https://docs.unrealengine.com/en-US/Programming/UnrealArchitecture/Reference/Classes/index.html][reference #2]
    3. [https://www.unrealengine.com/en-US/blog/unreal-property-system-reflection][reference #3]

[reference #1]: https://docs.unrealengine.com/en-US/Programming/UnrealArchitecture/Objects/index.html
[reference #2]: https://docs.unrealengine.com/en-US/Programming/UnrealArchitecture/Reference/Classes/index.html
[reference #3]: https://www.unrealengine.com/en-US/blog/unreal-property-system-reflection

---

# *overview*

```cpp
UCLASS(config=Game)
class AProjectSandboxCharacter : public ACharacter
{
    ...

public:
    AProjectSandboxCharacter();
    virtual ~AProjectSandboxCharacter();

    ...
}
```
</br>

{% asset_img 01.jpg %}

somebody may have the experience your breakpoint stops in cpp constructor even the object is not placed in the world. you might say,
`"Why this happens ? I do not get it why the constructor is called even while opening editor !"`

{% asset_img 02.jpg %}

vice versa, destructor is called when editor is closed. in your intuition, the cpp constructor or destructor seems to be called when the actor is created or deleted. but it does not as you saw. then, let us find what do cpp ctor and dtor do in unreal engine. we would get a clue to truth through it.

---

# *class default object*

unreal engine provides their own macro system, which has several features below:
- garbage collection
- reference update
- reflection
- serialization
- etc.

to support these features, unreal engine constructed massive macro magics and hacks. so, cpp in unreal engine acts unlike cpp as we know. this ctor/dtor issue is one of them(different behaviors). from [reference #2], we can find the reference of `Class Default Object, CDO`.

{% asset_img 03.jpg %}

cpp constructor makes the `Class Default Object` and it is copied whenever you create instance of the `UObject`. what `UObject` means in this post, is the object created by `NewObject` API. from [reference #1], we can find the reference of `NewObject` API.

{% asset_img 04.jpg %}

in summary...
- if your class(or somewhat) should be utilized with APIs of unreal engine, this must inherit `UObject` and follow some conventions
- if you do that, your class gonna have `CDO`, which is used for cloning object instance
- in this condition, cpp ctor/dtor only works for `CDO`

※ for more information of `CDO`, visit [reference #1] and [reference #2]
※ for more information of `Unreal Reflection`, visit [reference #3]

---

# *real ctor/dtor for unreal*

that is why we could see that the breakpoint stops at cpp ctor/dtor before unreal editor opens. `CDO` is needed to display the asset editor of the class. try some tests for yourself.

{% asset_img 05.jpg %}
</br>
{% asset_img 06.jpg %}

the pictures above says, changes in cpp ctor will be shown in asset editor (if the asset inherits the class). not only the simple `float` variables, but it affects the various component or material things.

so, let us suppose real ctor/dtor for unreal should do its work when the instance of class is created in "game", such as spawning bullets when player shoots the gun. there are several APIs for this purpose, but every child of `UObject` does not have common API.

here is a table for major classes. almost of gameplay framework classes inherit them.

child | function
- | -
`UUserWidget` | `UUserWidget::NativeConstruct`
`AActor` | `AActor::OnConstruction`
`UActorComponent` | `UActorComponent::InitializeComponent`

---

# *summary*

- due to supporting several features, cpp in unreal engine acts unlike cpp as we know.
- cpp ctor/dtor is for `CDO`, not for the cloned instances in "game". there are seperate APIs for them.
    - and it differs upon a class. there is no common unreal ctor/dtor.
- if you plan to make custom class inherits `UObject`, you should consider how to make unreal ctor/dtor for it.
    - or just let the class inherits a class already implemented unreal ctor/dtor, such as `UUserWidget`