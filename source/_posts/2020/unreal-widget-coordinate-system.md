---
title: unreal-widget-coordinate-system
date: 2020-02-09 19:46:17
tags: Unreal, Widget
---

- this post covers
    - introduction to widget coordinate system
    - what is `FGeometry` is
    - how to get transform of widget
    - the meaning of each part of transform
    - tips for manipulating widget transform
- environment
    - Unreal Engine (4.24)
    - Visual Studio 2017
    - Windows 10
- reference
    1. [https://docs.unrealengine.com/en-US/API/Runtime/SlateCore/Layout/FGeometry/index.html][reference #1]
    2. [https://wiki.unrealengine.com/index.php?title=UE4_Transform_Calculus_-_Part_1][reference #2]
    3. [https://wiki.unrealengine.com/index.php?title=UE4_Transform_Calculus_-_Part_2][reference #3]

[reference #1]: https://docs.unrealengine.com/en-US/API/Runtime/SlateCore/Layout/FGeometry/index.html
[reference #2]: https://wiki.unrealengine.com/index.php?title=UE4_Transform_Calculus_-_Part_1
[reference #3]: https://wiki.unrealengine.com/index.php?title=UE4_Transform_Calculus_-_Part_2

---

# _widget coordinate system_

{% asset_img 02.jpg %}

suppose you have a game window. it might look like the picture and focus onto the popup widget

{% asset_img 03.jpg %}

in this condition, we can call the coordinate system as `Window Space`

- `Window Space`
    - a space has an origin with left-top of window
    - we can use this coordinate system for getting certain position in window

{% asset_img 04.jpg %}

even we can imagine the coordinate system `Monitor Space`

- `Monitor Space`
    - a space has an origin with left-top of monitor
    - we can use this coordinate system for getting certain position in monitor

※ position values are approximate
※ these terms are not official. they are just used for explaining this post

# _`FGeometry` introduction_

```cpp
UnrealEngine/Engine/Source/Runtime/SlateCore/Public/Layout/Geometry.h

/**
 * Represents the position, size, and absolute position of a Widget in Slate.
 * The absolute location of a geometry is usually screen space or
 * window space depending on where the geometry originated.
 * Geometries are usually paired with a SWidget pointer in order
 * to provide information about a specific widget (see FArrangedWidget).
 * A Geometry's parent is generally thought to be the Geometry of the
 * the corresponding parent widget.
 */
USTRUCT(BlueprintType)
struct SLATECORE_API FGeometry
```

- in unreal engine, `FGeometry` can help getting those transforms mentioned above
- the `FGeometry`
    - contains useful data for widget especially the transform
    - is updated whenever widget is painted or ticked
- every variance to transform is accumulated in `FGeometry`
    - in short, each tick, every widget has latest information for calculating transform

```cpp
UnrealEngine/Engine/Source/Runtime/SlateCore/Private/Widgets/SWidget.h

//UE_DEPRECATED(4.23, "GetCachedGeometry has been deprecated, use GetTickSpaceGeometry instead")
const FGeometry& GetCachedGeometry() const;

/**
 * Gets the last geometry used to Tick the widget.  This data may not exist yet if this call happens prior to
 * the widget having been ticked/painted, or it may be out of date, or a frame behind.
 *
 * We recommend not to use this data unless there's no other way to solve your problem.  Normally in Slate we
 * try and handle these issues by making a dependent widget part of the hierarchy, as to avoid frame behind
 * or what are referred to as hysteresis problems, both caused by depending on geometry from the previous frame
 * being used to advise how to layout a dependent object the current frame.
 */
const FGeometry& GetTickSpaceGeometry() const;

/**
 * Gets the last geometry used to Tick the widget.  This data may not exist yet if this call happens prior to
 * the widget having been ticked/painted, or it may be out of date, or a frame behind.
 */
const FGeometry& GetPaintSpaceGeometry() const;
```

```cpp
UnrealEngine/Engine/Source/Runtime/SlateCore/Private/Widgets/SWidget.cpp

const FGeometry& SWidget::GetCachedGeometry() const
{
    return GetTickSpaceGeometry();
}

const FGeometry& SWidget::GetTickSpaceGeometry() const
{
    return PersistentState.DesktopGeometry;
}

const FGeometry& SWidget::GetPaintSpaceGeometry() const
{
    return PersistentState.AllottedGeometry;
}
```

- we can get `FGeometry` object via `SWidget::Get*Geometry()` series
    - FYI, `SWidget::GetCachedGeometry()` has been deprecated from 4.23
- for general purpose, you can use any getter
    - `DesktopGeometry` and `AllottedGeometry` are not that different
    - but unreal engine recommend you to use `AllottedGeometry` one

※ for more information, visit [reference #1] + [reference #2] + [reference #3]

# _`FGeometry` APIs_
- `GetLocalSize()`

```cpp
UnrealEngine/Engine/Source/Runtime/SlateCore/Public/Layout/Geometry.h

/** @return the size of the geometry in local space. */
FORCEINLINE const FVector2D& GetLocalSize() const { return Size; }
```
returns a size in local space. this size is usually determined with value set in editor

- `GetAbsoluteSize()`

```cpp
UnrealEngine/Engine/Source/Runtime/SlateCore/Public/Layout/Geometry.h

/**
 * Get the absolute size of the geometry in render space.
 */
FORCEINLINE FVector2D GetAbsoluteSize() const
{
    return AccumulatedRenderTransform.TransformVector(GetLocalSize());
}
```
returns a size in outer space. this size is based on your screen, which is real size

- `LocalToViewport()`

```cpp
UnrealEngine/Engine/Source/Runtime/UMG/Public/Blueprint/SlateBlueprintLibrary.h

/**
 * Translates local coordinate of the geometry provided into local viewport coordinates.
 *
 * @param PixelPosition The position in the game's viewport, usable for line traces and 
 * other uses where you need a coordinate in the space of viewport resolution units.
 * @param ViewportPosition The position in the space of other widgets in the viewport.  Like if you wanted
 * to add another widget to the viewport at the same position in viewport space as this location, this is
 * what you would use.
 */
UFUNCTION(BlueprintPure, Category="User Interface|Geometry", meta=( WorldContext="WorldContextObject" ))
static void LocalToViewport(UObject* WorldContextObject, const FGeometry& Geometry, FVector2D LocalCoordinate, FVector2D& PixelPosition, FVector2D& ViewportPosition);
```
returns positions in outer space and local space respectively with parameters, which are `PixelPosition` and `ViewportPosition`

- `LocalToAbsolute()`

```cpp
UnrealEngine/Engine/Source/Runtime/UMG/Public/Blueprint/SlateBlueprintLibrary.h

/**
 * Translates local coordinates into absolute coordinates
 *
 * Absolute coordinates could be either desktop or window space depending on what space the root of the widget hierarchy is in.
 *
 * @return  Absolute coordinates
 */
UFUNCTION(BlueprintPure, Category="User Interface|Geometry")
static FVector2D LocalToAbsolute(const FGeometry& Geometry, FVector2D LocalCoordinate);
```
returns a position in `Monitor Space`

# _how to use_
those APIs can be used for calculating transform. let us find out how to use them

{% asset_img 05.jpg %}
</br>
{% asset_img 09.jpg %}
</br>
{% asset_img 10.jpg %}
</br>
{% asset_img 11.jpg %}
</br>
{% asset_img 12.jpg %}

Child | Position | Size | Parent
- | - | - | -
Red One | (100, 50) | (60, 30) | Canvas Panel #1
Green One | (0, 0) | (60, 30) | Canvas Panel #2
Blue One | NaN | (60, 30) | Horizontal Box
White One | NaN | (60, 30) | Horizontal Box

we gotta use this widget for example. the widget has children with multiple depth. each editor property of child is above

{% asset_img 06.jpg %}
</br>
{% asset_img 07.jpg %}
</br>
{% asset_img 08.jpg %}

※ transform property can be divided into _local_ and _absolute_ due to `DPI Scale`
※ in this example(and also default value), we got `DPI Scale = 2/3` which means that every property gets multiplied with 0.666666...

- `Window Space`
    - widget
        - size
{% asset_img 13.jpg %}
            - local = `Widget->GetCachedGeometry()->GetLocalSize()`
{% asset_img 14.jpg %}
            - absolute = `Widget->GetCachedGeometry()->GetAbsoluteSize()`
                *(60, 30) * 0.666666... ≒ (39.99996..., 19.99998...)*
        - position
{% asset_img 15.jpg %}
            - local = `Widget->GetCachedGeometry()->LocalToViewport()->ViewportPosition`
{% asset_img 16.jpg %}
            - absolute = `Widget->GetCachedGeometry()->LocalToViewport()->PixelPosition`
                *(100, 50) * 0.666666... ≒ (66.66666..., 33.33333...)*
- verification (with Adobe Photoshop)
{% asset_img 20.jpg %}
    - widget
        - size
{% asset_img 17.jpg %}
</br>
{% asset_img 18.jpg %}
        - position
{% asset_img 19.jpg %}

- `Monitor Space`
    - window
        - size
            - local
{% asset_img 22.jpg %}
            approximately 1920x1080
            - absolute
{% asset_img 23.jpg %}
                *(1920, 1080) * 0.666666... = (1280, 720)*
        - position
            - local
{% asset_img 25.jpg %}
</br>
{% asset_img 26.jpg %}
            cannot calculate. also not meaningful, unless the `DPI Scale` is the same with system's one
            - absolute
{% asset_img 24.jpg %}
    - widget
        - size
        the same with before
        - position
            - local
            cannot calculate. also not meaningful, unless the `DPI Scale` is the same with system's one
            - absolute
{% asset_img 21.jpg %}
- verification (with Adobe Photoshop)
{% asset_img 27.jpg %}
    - window
        - size
{% asset_img 28.jpg %}
</br>
{% asset_img 29.jpg %}
        - position
{% asset_img 30.jpg %}
    - widget
        - size
        the same with before
        - position
{% asset_img 31.jpg %}

`Window Space` summary

Property | Widget
- | -
Size.Local | (60, 30)
Size.Absolute | (40, 20)
Position.Local | (100, 50)
Position.Absolute | (66, 33)

`Monitor Space` summary

Property | Window | Widget
- | - | -
Size.Local | (1920, 1080) | (60, 30)
Size.Absolute | (1280, 720) | (40, 20)
Position.Local | NaN | NaN
Position.Absoulte | (640, 350) | (640 + 66, 350 + 33) = (706, 383)

# _widget translation_

suppose you have to translate a widget into certain position. how will you ?
first, we need to check whether the slot of widget has `SetPosition` API. if the slot does not, we cannot do widget translation
but we can do it, as the red image was in `Canvas Panel`

```cpp
UnrealEngine/Engine/Source/Runtime/UMG/Public/Components/CanvasPanelSlot.h

/** Sets the position of the slot */
UFUNCTION(BlueprintCallable, Category="Layout|Canvas Slot")
void SetPosition(FVector2D InPosition);
```
</br>
```cpp
UnrealEngine/Engine/Source/Runtime/UMG/Public/Components/CanvasPanelSlot.cpp

void UCanvasPanelSlot::SetPosition(FVector2D InPosition)
{
    LayoutData.Offsets.Left = InPosition.X;
    LayoutData.Offsets.Top = InPosition.Y;

    if ( Slot )
    {
        Slot->Offset(LayoutData.Offsets);
    }
}
```

the parameter `InPosition` should be local position. it is easy to think, this is the same work with changing value in editor
second, we need to get local position of the widget. let us use the example red image again
in previous example, red image had a local position of (100, 50) in `Window Space`

{% asset_img 09.jpg %}
</br>
{% asset_img 15.jpg %}
</br>
{% asset_img 32.jpg %}

let us move the widget with offset (100, 50) again
third, we need to set local position with new one. in this time, `SetPosition(200, 100)` will be needed
and we gotta check a local position at next tick. the local position would be (200, 100) and absolute position also be twice

{% asset_img 33.jpg %}
</br>
{% asset_img 34.jpg %}
</br>
{% asset_img 35.jpg %}

- verification (with Adobe Photoshop)
{% asset_img 36.jpg %}
    - size
    the same with before
    - position
        - absolute position in `Window Space`
{% asset_img 37.jpg %}
        - absolute position in `Monitor Space`
{% asset_img 38.jpg %}

wrap-up !
- getting local or absolute position is usually possible regardless of layout of widget such as widget hierarchy
- we can only set local position through the API of slot such as `Canvas Panel Slot`
- you need to calculate new position (based on several conditions, especially `DPI Scale`) to translate widget into desired position