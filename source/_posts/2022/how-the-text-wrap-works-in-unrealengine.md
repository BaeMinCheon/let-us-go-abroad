---
title: How the text wrap works in UnrealEngine
date: 2022-03-22 00:07:38
tags: [UnrealEngine, TextBlock, TextWrap]
---

| Environment | |
| --- | --- |
| UnrealEngine | `branch: 5.0` |
| Visual Studio 2022 | `version: 17.1.1` |
| Windows 11 Pro | `build: 22000.556` |

# _`Overview`_

{% asset_img 01.png %}

A TextBlock has an option `AutoWrapText` and the option makes the TextBlock can wrap its text. Thanks to the option, we can display a text without concerning about breaking lines. For general cases of text, even the option works within very short time, almost 1 tick. How does it possible ? What is the implementation of that option ? Let us find out it in this post.

{% asset_img 02.png %}
{% asset_img 03.png %}

The TextBlock upper has the option turned on. Contrary, the TextBlock lower has the option turned off.

# _`Where is the code`_

```cpp
// TextWidgetTypes.h

/** True if we're wrapping text automatically based on the computed horizontal space for this widget. */
UPROPERTY(EditAnywhere, BlueprintReadOnly, Category=Wrapping)
uint8 AutoWrapText:1;
```

The option is loacted in the class `UTextLayoutWidget`. We can see the option as the class `UTextBlock` inherites `UTextLayoutWidget`. Unfortunately, the variable is not directly used for wrapping text, but used for saving the value of option.

```cpp
void UTextBlock::SynchronizeProperties()
{
	Super::SynchronizeProperties();

	TAttribute<FText> TextBinding = GetDisplayText();
	TAttribute<FSlateColor> ColorAndOpacityBinding = PROPERTY_BINDING(FSlateColor, ColorAndOpacity);
	TAttribute<FLinearColor> ShadowColorAndOpacityBinding = PROPERTY_BINDING(FLinearColor, ShadowColorAndOpacity);

	if ( MyTextBlock.IsValid() )
	{
		MyTextBlock->SetText( TextBinding );
		MyTextBlock->SetFont( Font );
		MyTextBlock->SetStrikeBrush( &StrikeBrush );
		MyTextBlock->SetColorAndOpacity( ColorAndOpacityBinding );
		MyTextBlock->SetShadowOffset( ShadowOffset );
		MyTextBlock->SetShadowColorAndOpacity( ShadowColorAndOpacityBinding );
		MyTextBlock->SetMinDesiredWidth( MinDesiredWidth );
		MyTextBlock->SetTransformPolicy( TextTransformPolicy );
		MyTextBlock->SetOverflowPolicy(TextOverflowPolicy);

		Super::SynchronizeTextLayoutProperties( *MyTextBlock );
	}
}
```

When you turn on or turn off the option `AutoWrapText`, widget's `SynchronizeProperties()` would be called. By the code `Super::SynchronizeTextLayoutProperties(*MyTextBlock);` executed, Parent's `SynchronizeProperties(TWidgetType&)` is called.

```cpp
/** Synchronize the properties with the given widget. A template as the Slate widgets conform to the same API, but don't derive from a common base. */
template <typename TWidgetType>
void SynchronizeTextLayoutProperties(TWidgetType& InWidget)
{
    ShapedTextOptions.SynchronizeShapedTextProperties(InWidget);

    InWidget.SetJustification(Justification);
    InWidget.SetAutoWrapText(!!AutoWrapText);
    InWidget.SetWrapTextAt(WrapTextAt != 0 ? WrapTextAt : TAttribute<float>());
    InWidget.SetWrappingPolicy(WrappingPolicy);
    InWidget.SetMargin(Margin);
    InWidget.SetLineHeightPercentage(LineHeightPercentage);
}
```

In this function, `InWidget` is our TextBlock. And it would call the function `SetAutoWrapText(bool)` for updating the option.

```cpp
void UTextBlock::SetAutoWrapText(bool InAutoWrapText)
{
	AutoWrapText = InAutoWrapText;
	if(MyTextBlock.IsValid())
	{
		MyTextBlock->SetAutoWrapText(InAutoWrapText);
	}
}
```

Good. The parameter `InAutoWrapText` updates the variable `AutoWrapText` and `MyTextBlock`. The variable `MyTextBlock` is `TSharedPtr<STextBlock>`. Now, the time to jump to `STextBlock`.

```cpp
void STextBlock::SetAutoWrapText(TAttribute<bool> InAutoWrapText)
{
	AutoWrapText.Assign(*this, MoveTemp(InAutoWrapText), 0.f);
}
```

Here, in `STextBlock` the variable `AutoWrapText` holds the value of option. The function `Assign()` just saves the value its inside. The value of `AutoWrapText` is used in two positions.

```cpp
// STextBlock.cpp
FVector2D STextBlock::ComputeDesiredSize(float LayoutScaleMultiplier) const
{
	SCOPE_CYCLE_COUNTER(Stat_SlateTextBlockCDS);

	if (bSimpleTextMode)
	{
		const FVector2D LocalShadowOffset = GetShadowOffset();

		const float LocalOutlineSize = (float)(GetFont().OutlineSettings.OutlineSize);

		// Account for the outline width impacting both size of the text by multiplying by 2
		// Outline size in Y is accounted for in MaxHeight calculation in Measure()
		const FVector2D ComputedOutlineSize(LocalOutlineSize * 2.f, LocalOutlineSize);
		const FVector2D TextSize = FSlateApplication::Get().GetRenderer()->GetFontMeasureService()->Measure(BoundText.Get(), GetFont()) + ComputedOutlineSize + LocalShadowOffset;

		CachedSimpleDesiredSize = FVector2f(FMath::Max(MinDesiredWidth.Get(), TextSize.X), TextSize.Y);
		return FVector2D(CachedSimpleDesiredSize.GetValue());
	}
	else
	{
		// ComputeDesiredSize will also update the text layout cache if required
		const FVector2D TextSize = TextLayoutCache->ComputeDesiredSize(
			FSlateTextBlockLayout::FWidgetDesiredSizeArgs(
				BoundText.Get(),
				HighlightText.Get(),
				WrapTextAt.Get(),
				AutoWrapText.Get(),
				WrappingPolicy.Get(),
				GetTransformPolicyImpl(),
				Margin.Get(),
				LineHeightPercentage.Get(),
				Justification.Get()),
			LayoutScaleMultiplier, GetComputedTextStyle());

		return FVector2D(FMath::Max(MinDesiredWidth.Get(), TextSize.X), TextSize.Y);
	}
}

// Callstack
UnrealEditor-Slate.dll!STextBlock::ComputeDesiredSize(float LayoutScaleMultiplier) Line 300	C++
UnrealEditor-SlateCore.dll!SWidget::CacheDesiredSize(float InLayoutScaleMultiplier) Line 936	C++
UnrealEditor-SlateCore.dll!SWidget::Prepass_Internal(float InLayoutScaleMultiplier) Line 1714	C++
[Inline Frame] UnrealEditor-SlateCore.dll!SWidget::Prepass_ChildLoop::__l2::<lambda_a0677895c4614612fd5b4c5f4771eae9>::operator()(SWidget &) Line 1751	C++
UnrealEditor-SlateCore.dll!FChildren::ForEachWidget<<lambda_a0677895c4614612fd5b4c5f4771eae9>>(SWidget::Prepass_ChildLoop::__l2::<lambda_a0677895c4614612fd5b4c5f4771eae9> Pred) Line 67	C++
[Inline Frame] UnrealEditor-SlateCore.dll!SWidget::Prepass_ChildLoop(float) Line 1721	C++
UnrealEditor-SlateCore.dll!SWidget::Prepass_Internal(float InLayoutScaleMultiplier) Line 1708	C++
...
UnrealEditor-SlateCore.dll!SWidget::Prepass_Internal(float InLayoutScaleMultiplier) Line 1708	C++
[Inline Frame] UnrealEditor-SlateCore.dll!SWidget::Prepass_ChildLoop::__l2::<lambda_a0677895c4614612fd5b4c5f4771eae9>::operator()(SWidget &) Line 1751	C++
UnrealEditor-SlateCore.dll!FChildren::ForEachWidget<<lambda_a0677895c4614612fd5b4c5f4771eae9>>(SWidget::Prepass_ChildLoop::__l2::<lambda_a0677895c4614612fd5b4c5f4771eae9> Pred) Line 67	C++
[Inline Frame] UnrealEditor-SlateCore.dll!SWidget::Prepass_ChildLoop(float) Line 1721	C++
UnrealEditor-SlateCore.dll!SWidget::Prepass_Internal(float InLayoutScaleMultiplier) Line 1708	C++
UnrealEditor-SlateCore.dll!SWidget::SlatePrepass(float InLayoutScaleMultiplier) Line 690	C++
UnrealEditor-Slate.dll!PrepassWindowAndChildren(TSharedRef<SWindow,1> WindowToPrepass) Line 1197	C++
UnrealEditor-Slate.dll!FSlateApplication::DrawPrepass(TSharedPtr<SWindow,1> DrawOnlyThisWindow) Line 1249	C++
UnrealEditor-Slate.dll!FSlateApplication::PrivateDrawWindows(TSharedPtr<SWindow,1> DrawOnlyThisWindow) Line 1294	C++
UnrealEditor-Slate.dll!FSlateApplication::DrawWindows() Line 1060	C++
UnrealEditor-Slate.dll!FSlateApplication::TickAndDrawWidgets(float DeltaTime) Line 1625	C++
UnrealEditor-Slate.dll!FSlateApplication::Tick(ESlateTickType TickType) Line 1482	C++
UnrealEditor.exe!FEngineLoop::Tick() Line 5325	C++
[Inline Frame] UnrealEditor.exe!EngineTick() Line 62	C++
UnrealEditor.exe!GuardedMain(const wchar_t * CmdLine) Line 183	C++
UnrealEditor.exe!GuardedMainWrapper(const wchar_t * CmdLine) Line 147	C++
UnrealEditor.exe!LaunchWindowsStartup(HINSTANCE__ * hInInstance, HINSTANCE__ * hPrevInstance, char * __formal, int nCmdShow, const wchar_t * CmdLine) Line 283	C++
UnrealEditor.exe!WinMain(HINSTANCE__ * hInInstance, HINSTANCE__ * hPrevInstance, char * pCmdLine, int nCmdShow) Line 330	C++
[External Code]
```

First, an execution flow by Prepass.

```cpp
// STextBlock.cpp

int32 STextBlock::OnPaint( const FPaintArgs& Args, const FGeometry& AllottedGeometry, const FSlateRect& MyCullingRect, FSlateWindowElementList& OutDrawElements, int32 LayerId, const FWidgetStyle& InWidgetStyle, bool bParentEnabled ) const
{
	SCOPE_CYCLE_COUNTER(Stat_SlateTextBlockOnPaint);

	if (bSimpleTextMode)
	{
		// Draw the optional shadow
		const FLinearColor LocalShadowColorAndOpacity = GetShadowColorAndOpacity();
		const FVector2D LocalShadowOffset = GetShadowOffset();
		const bool ShouldDropShadow = LocalShadowColorAndOpacity.A > 0.f && LocalShadowOffset.SizeSquared() > 0.f;

		const bool bShouldBeEnabled = ShouldBeEnabled(bParentEnabled);

		const FText& LocalText = BoundText.Get();
		FSlateFontInfo LocalFont = GetFont();

		if (ShouldDropShadow)
		{
			const int32 OutlineSize = LocalFont.OutlineSettings.OutlineSize;
			if (!LocalFont.OutlineSettings.bApplyOutlineToDropShadows)
			{
				LocalFont.OutlineSettings.OutlineSize = 0;
			}

			FSlateDrawElement::MakeText(
				OutDrawElements,
				LayerId,
				AllottedGeometry.ToOffsetPaintGeometry(LocalShadowOffset),
				LocalText,
				LocalFont,
				bShouldBeEnabled ? ESlateDrawEffect::None : ESlateDrawEffect::DisabledEffect,
				InWidgetStyle.GetColorAndOpacityTint() * LocalShadowColorAndOpacity
			);

			// Restore outline size for main text
			LocalFont.OutlineSettings.OutlineSize = OutlineSize;

			// actual text should appear above the shadow
			++LayerId;
		}

		// Draw the text itself
		FSlateDrawElement::MakeText(
			OutDrawElements,
			LayerId,
			AllottedGeometry.ToPaintGeometry(),
			LocalText,
			LocalFont,
			bShouldBeEnabled ? ESlateDrawEffect::None : ESlateDrawEffect::DisabledEffect,
			InWidgetStyle.GetColorAndOpacityTint() * GetColorAndOpacity().GetColor(InWidgetStyle)
			);
	}
	else
	{
		const FVector2D LastDesiredSize = TextLayoutCache->GetDesiredSize();

		// OnPaint will also update the text layout cache if required
		LayerId = TextLayoutCache->OnPaint(Args, AllottedGeometry, MyCullingRect, OutDrawElements, LayerId, InWidgetStyle, ShouldBeEnabled(bParentEnabled));

		const FVector2D NewDesiredSize = TextLayoutCache->GetDesiredSize();

		// HACK: Due to the nature of wrapping and layout, we may have been arranged in a different box than what we were cached with.  Which
		// might update wrapping, so make sure we always set the desired size to the current size of the text layout, which may have changed
		// during paint.
		const bool bCanWrap = WrapTextAt.Get() > 0 || AutoWrapText.Get();

		if (bCanWrap && !NewDesiredSize.Equals(LastDesiredSize))
		{
			const_cast<STextBlock*>(this)->Invalidate(EInvalidateWidgetReason::Layout);
		}
	}

	return LayerId;
}

// Callstack
UnrealEditor-Slate.dll!STextBlock::OnPaint(const FPaintArgs & Args, const FGeometry & AllottedGeometry, const FSlateRect & MyCullingRect, FSlateWindowElementList & OutDrawElements, int LayerId, const FWidgetStyle & InWidgetStyle, bool bParentEnabled) Line 255	C++
UnrealEditor-SlateCore.dll!SWidget::Paint(const FPaintArgs & Args, const FGeometry & AllottedGeometry, const FSlateRect & MyCullingRect, FSlateWindowElementList & OutDrawElements, int LayerId, const FWidgetStyle & InWidgetStyle, bool bParentEnabled) Line 1543	C++
UnrealEditor-SlateCore.dll!SPanel::PaintArrangedChildren(const FPaintArgs & Args, const FArrangedChildren & ArrangedChildren, const FGeometry & AllottedGeometry, const FSlateRect & MyCullingRect, FSlateWindowElementList & OutDrawElements, int LayerId, const FWidgetStyle & InWidgetStyle, bool bParentEnabled) Line 31	C++
UnrealEditor-SlateCore.dll!SPanel::OnPaint(const FPaintArgs & Args, const FGeometry & AllottedGeometry, const FSlateRect & MyCullingRect, FSlateWindowElementList & OutDrawElements, int LayerId, const FWidgetStyle & InWidgetStyle, bool bParentEnabled) Line 12	C++
UnrealEditor-SlateCore.dll!SWidget::Paint(const FPaintArgs & Args, const FGeometry & AllottedGeometry, const FSlateRect & MyCullingRect, FSlateWindowElementList & OutDrawElements, int LayerId, const FWidgetStyle & InWidgetStyle, bool bParentEnabled) Line 1543	C++
...
UnrealEditor-SlateCore.dll!SPanel::PaintArrangedChildren(const FPaintArgs & Args, const FArrangedChildren & ArrangedChildren, const FGeometry & AllottedGeometry, const FSlateRect & MyCullingRect, FSlateWindowElementList & OutDrawElements, int LayerId, const FWidgetStyle & InWidgetStyle, bool bParentEnabled) Line 31	C++
UnrealEditor-SlateCore.dll!SPanel::OnPaint(const FPaintArgs & Args, const FGeometry & AllottedGeometry, const FSlateRect & MyCullingRect, FSlateWindowElementList & OutDrawElements, int LayerId, const FWidgetStyle & InWidgetStyle, bool bParentEnabled) Line 12	C++
UnrealEditor-SlateCore.dll!SWidget::Paint(const FPaintArgs & Args, const FGeometry & AllottedGeometry, const FSlateRect & MyCullingRect, FSlateWindowElementList & OutDrawElements, int LayerId, const FWidgetStyle & InWidgetStyle, bool bParentEnabled) Line 1543	C++
UnrealEditor-SlateCore.dll!SOverlay::OnPaint(const FPaintArgs & Args, const FGeometry & AllottedGeometry, const FSlateRect & MyCullingRect, FSlateWindowElementList & OutDrawElements, int LayerId, const FWidgetStyle & InWidgetStyle, bool bParentEnabled) Line 200	C++
UnrealEditor-SlateCore.dll!SWidget::Paint(const FPaintArgs & Args, const FGeometry & AllottedGeometry, const FSlateRect & MyCullingRect, FSlateWindowElementList & OutDrawElements, int LayerId, const FWidgetStyle & InWidgetStyle, bool bParentEnabled) Line 1543	C++
UnrealEditor-SlateCore.dll!SCompoundWidget::OnPaint(const FPaintArgs & Args, const FGeometry & AllottedGeometry, const FSlateRect & MyCullingRect, FSlateWindowElementList & OutDrawElements, int LayerId, const FWidgetStyle & InWidgetStyle, bool bParentEnabled) Line 46	C++
UnrealEditor-SlateCore.dll!SWidget::Paint(const FPaintArgs & Args, const FGeometry & AllottedGeometry, const FSlateRect & MyCullingRect, FSlateWindowElementList & OutDrawElements, int LayerId, const FWidgetStyle & InWidgetStyle, bool bParentEnabled) Line 1543	C++
UnrealEditor-SlateCore.dll!SWindow::PaintSlowPath(const FSlateInvalidationContext & Context) Line 2073	C++
UnrealEditor-SlateCore.dll!FSlateInvalidationRoot::PaintInvalidationRoot(const FSlateInvalidationContext & Context) Line 399	C++
UnrealEditor-SlateCore.dll!SWindow::PaintWindow(double CurrentTime, float DeltaTime, FSlateWindowElementList & OutDrawElements, const FWidgetStyle & InWidgetStyle, bool bParentEnabled) Line 2105	C++
UnrealEditor-Slate.dll!FSlateApplication::DrawWindowAndChildren(const TSharedRef<SWindow,1> & WindowToDraw, FDrawWindowArgs & DrawWindowArgs) Line 1106	C++
UnrealEditor-Slate.dll!FSlateApplication::PrivateDrawWindows(TSharedPtr<SWindow,1> DrawOnlyThisWindow) Line 1338	C++
UnrealEditor-Slate.dll!FSlateApplication::DrawWindows() Line 1060	C++
UnrealEditor-Slate.dll!FSlateApplication::TickAndDrawWidgets(float DeltaTime) Line 1625	C++
UnrealEditor-Slate.dll!FSlateApplication::Tick(ESlateTickType TickType) Line 1482	C++
UnrealEditor.exe!FEngineLoop::Tick() Line 5325	C++
[Inline Frame] UnrealEditor.exe!EngineTick() Line 62	C++
UnrealEditor.exe!GuardedMain(const wchar_t * CmdLine) Line 183	C++
UnrealEditor.exe!GuardedMainWrapper(const wchar_t * CmdLine) Line 147	C++
UnrealEditor.exe!LaunchWindowsStartup(HINSTANCE__ * hInInstance, HINSTANCE__ * hPrevInstance, char * __formal, int nCmdShow, const wchar_t * CmdLine) Line 283	C++
UnrealEditor.exe!WinMain(HINSTANCE__ * hInInstance, HINSTANCE__ * hPrevInstance, char * pCmdLine, int nCmdShow) Line 330	C++
[External Code]	
```

Second, an execution flow by Paint.

The flows are branched at `FSlateApplication::PrivateDrawWindows()`. In the function, `DrawPrepass()` is called at line #1292, and `DrawWindowAndChildren()` is called at line #1338. Respectively, Prepass and Paint. Engine just invalidate the widget in Paint flow, so we only need to look into Prepass flow.

# _`Calculating a length of text wrap`_

```cpp
FVector2D FSlateTextBlockLayout::ComputeDesiredSize(const FWidgetDesiredSizeArgs& InWidgetArgs, const float InScale, const FTextBlockStyle& InTextStyle)
{
	// Cache the wrapping rules so that we can recompute the wrap at width in paint.
	CachedWrapTextAt = InWidgetArgs.WrapTextAt;
	bCachedAutoWrapText = InWidgetArgs.AutoWrapText;

	const ETextTransformPolicy PreviousTransformPolicy = TextLayout->GetTransformPolicy();

	// Set the text layout information
	TextLayout->SetScale(InScale);
	TextLayout->SetWrappingWidth(CalculateWrappingWidth());
	TextLayout->SetWrappingPolicy(InWidgetArgs.WrappingPolicy);
	TextLayout->SetTransformPolicy(InWidgetArgs.TransformPolicy);
	TextLayout->SetMargin(InWidgetArgs.Margin);
	TextLayout->SetJustification(InWidgetArgs.Justification);
	TextLayout->SetLineHeightPercentage(InWidgetArgs.LineHeightPercentage);

	// Has the transform policy changed? If so we need a full refresh as that is destructive to the model text
	if (PreviousTransformPolicy != TextLayout->GetTransformPolicy())
	{
		Marshaller->MakeDirty();
	}

	// Has the style used for this text block changed?
	if (!IsStyleUpToDate(InTextStyle))
	{
		TextLayout->SetDefaultTextStyle(InTextStyle);
		Marshaller->MakeDirty(); // will regenerate the text using the new default style
	}

	{
		bool bRequiresTextUpdate = false;
		const FText& TextToSet = InWidgetArgs.Text;
		if (!TextLastUpdate.IdenticalTo(TextToSet))
		{
			// The pointer used by the bound text has changed, however the text may still be the same - check that now
			if (!TextLastUpdate.IsDisplayStringEqualTo(TextToSet))
			{
				// The source text has changed, so update the internal text
				bRequiresTextUpdate = true;
			}

			// Update this even if the text is lexically identical, as it will update the pointer compared by IdenticalTo for the next Tick
			TextLastUpdate = FTextSnapshot(TextToSet);
		}

		if (bRequiresTextUpdate || Marshaller->IsDirty())
		{
			UpdateTextLayout(TextToSet);
		}
	}

	{
		const FText& HighlightTextToSet = InWidgetArgs.HighlightText;
		if (!HighlightTextLastUpdate.IdenticalTo(HighlightTextToSet))
		{
			// The pointer used by the bound text has changed, however the text may still be the same - check that now
			if (!HighlightTextLastUpdate.IsDisplayStringEqualTo(HighlightTextToSet))
			{
				UpdateTextHighlights(HighlightTextToSet);
			}

			// Update this even if the text is lexically identical, as it will update the pointer compared by IdenticalTo for the next Tick
			HighlightTextLastUpdate = FTextSnapshot(HighlightTextToSet);
		}
	}

	// We need to update our size if the text layout has become dirty
	TextLayout->UpdateIfNeeded();

	return TextLayout->GetSize();
}
```

The function `FSlateTextBlockLayout::ComputeDesiredSize()` is called during Prepass flow. Here, `bCachedAutoWrapText` caches the value of `InWidgetArgs.AutoWrapText`. This will be used at `CalculateWrappingWidth()` later.

```cpp
float FSlateTextBlockLayout::CalculateWrappingWidth() const
{
	// Text wrapping can either be used defined (WrapTextAt), automatic (bAutoWrapText and CachedSize), 
	// or a mixture of both. Take whichever has the smallest value (>1)
	float WrappingWidth = CachedWrapTextAt;
	if (bCachedAutoWrapText && CachedSize.X >= 1.0f)
	{
		WrappingWidth = (WrappingWidth >= 1.0f) ? FMath::Min(WrappingWidth, CachedSize.X) : CachedSize.X;
	}

	return FMath::Max(0.0f, WrappingWidth);
}
```

The `CachedWrapTextAt` will be the same with the value set by option `WrapTextAt` in editor. And, the `CachedSize` depends on the size of panel where the TextBlock resides in. In the example we are using, the variables would have a value like below:

- `CachedWrapTextAt` = 0
- `CachedSize.X` = 100

Because the width of SizeBox is 100 and we set the option `WrapTextAt` as 0. The function determines the length of wrapping, but it is not for the logic about how to divide texts or how to break lines. So, look back on `FSlateTextBlockLayout::ComputeDesiredSize()`.

# _`UpdateLayout when it is dirty`_

```cpp
// SlateTextBlockLayout.cpp

...
// We need to update our size if the text layout has become dirty
TextLayout->UpdateIfNeeded();
...

// TextLayout.cpp

void FTextLayout::UpdateIfNeeded()
{
	if (CachedLayoutGeneration != GSlateLayoutGeneration)
	{
		CachedLayoutGeneration = GSlateLayoutGeneration;
		DirtyFlags |= ETextLayoutDirtyState::Layout;
		DirtyAllLineModels(ELineModelDirtyState::All);
	}

	const bool bHasChangedLayout = !!(DirtyFlags & ETextLayoutDirtyState::Layout);
	const bool bHasChangedHighlights = !!(DirtyFlags & ETextLayoutDirtyState::Highlights);

	if ( bHasChangedLayout )
	{
		// if something has changed then create a new View
		UpdateLayout();
	}

	// If the layout has changed, we always need to update the highlights
	if ( bHasChangedLayout || bHasChangedHighlights)
	{
		UpdateHighlights();
	}
}
```

In the function, there is some code to call `FTextLayout::UpdateIfNeeded()`. Oh, the `UpdateLayout()` looks like the one we wanted. The code will be executed when `bHasChangedLayout` is true, and the value is usually set by `SetWrappingWidth()`.

```cpp
void FTextLayout::SetWrappingWidth( float Value )
{
	const bool WasWrapping = WrappingWidth > 0.0f;
	const bool IsWrapping = Value > 0.0f;

	if ( WrappingWidth != Value )
	{
		WrappingWidth = Value; 
		DirtyFlags |= ETextLayoutDirtyState::Layout;

		if ( WasWrapping != IsWrapping )
		{
			// Changing from wrapping/not-wrapping will affect the wrapping information for *all lines*
			// Clear out the entire cache so it gets regenerated on the text call to FlowLayout
			DirtyAllLineModels(ELineModelDirtyState::WrappingInformation);
		}
	}
}
```

Suppose you switch the option `AutoWrapText` from false into true. Here, `DirtyFlags` will flag the `ETextLayoutDirtyState::Layout`, which is 1. Therefore, `!!(DirtyFlags & ETextLayoutDirtyState::Layout)` turns into 1. The `bHasChangedLayout` becomes 1, too.

```cpp
void FTextLayout::UpdateLayout()
{
	SCOPE_CYCLE_COUNTER(STAT_SlateTextLayout);

	ClearView();
	BeginLayout();

	FlowLayout();
	JustifyLayout();
	MarginLayout();

	EndLayout();

	DirtyFlags &= ~ETextLayoutDirtyState::Layout;
}
```

The `ClearView()` and `BeginLayout()` are not important in this post. Plus, they do not something important either.

```cpp
void FTextLayout::FlowLayout()
{
	const float WrappingDrawWidth = GetWrappingDrawWidth();

	TArray< TSharedRef< ILayoutBlock > > SoftLine;
	for (int32 LineModelIndex = 0; LineModelIndex < LineModels.Num(); LineModelIndex++)
	{
		FLineModel& LineModel = LineModels[ LineModelIndex ];
		CalculateLineTextDirection(LineModel);
		FlushLineTextShapingCache(LineModel);
		CreateLineWrappingCache(LineModel);

		FlowLineLayout(LineModelIndex, WrappingDrawWidth, SoftLine);
	}
}
```

In the `FlowLayout()`, the code that calls `CreateLineWrappingCache()` is a point since the `CreateLineWrappingCache()` creates data for wrapping text.

# _`Break lines (1/3); Separating text into slices`_

```cpp
void FTextLayout::CreateLineWrappingCache(FLineModel& LineModel)
{
	if (!(LineModel.DirtyFlags & ELineModelDirtyState::WrappingInformation))
	{
		return;
	}

	LineModel.BreakCandidates.Empty();
	LineModel.DirtyFlags &= ~ELineModelDirtyState::WrappingInformation;

	for (int32 RunIndex = 0; RunIndex < LineModel.Runs.Num(); RunIndex++)
	{
		LineModel.Runs[RunIndex].ClearCache();
	}

	const bool IsWrapping = WrappingWidth > 0.0f;
	if (!IsWrapping)
	{
		return;
	}

	// If we've not yet been provided with a custom line break iterator, then just use the default one
	if (!LineBreakIterator.IsValid())
	{
		LineBreakIterator = FBreakIterator::CreateLineBreakIterator();
	}

	LineBreakIterator->SetStringRef(&LineModel.Text.Get());

	int32 PreviousBreak = 0;
	int32 CurrentBreak = 0;
	int32 CurrentRunIndex = 0;

	while( ( CurrentBreak = LineBreakIterator->MoveToNext() ) != INDEX_NONE )
	{
		LineModel.BreakCandidates.Add( CreateBreakCandidate(/*OUT*/CurrentRunIndex, LineModel, PreviousBreak, CurrentBreak) );
		PreviousBreak = CurrentBreak;
	}

	LineBreakIterator->ClearString();
}
```

In this function, we found some variables that have a name of `LineBreak` at last. Let us check what the line break iterator does.

```cpp
TSharedRef<IBreakIterator> FBreakIterator::CreateLineBreakIterator()
{
	return MakeShareable(new FICULineBreakIterator());
}
```

The `LinBreakIterator` is a line break iterator using the implementation of [ICU(International Components for Unicode)'s break iterator](https://github.com/unicode-org/icu/blob/main/icu4c/source/common/unicode/brkiter.h). The break iterator does a job of finding a location of boundaries in text. Visit [here](https://unicode-org.github.io/icu-docs/apidoc/dev/icu4c/classicu_1_1BreakIterator.html#details) for more details. To summarize, the break iterator can find where each word ends. For example, we have a text of `Text Block Test` and the break iterator can find locations just like this `Text(HERE) Block(HERE) Test(HERE)`. So, let us see how it works.

```cpp
int32 FICULineBreakIterator::MoveToNextImpl()
{
	TSharedRef<icu::BreakIterator> LineBrkIt = GetInternalLineBreakIterator();
	FICUTextCharacterIterator& CharIt = static_cast<FICUTextCharacterIterator&>(LineBrkIt->getText());

	int32 InternalPosition = CharIt.SourceIndexToInternalIndex(CurrentPosition);

	// For Hangul using per-word wrapping, we walk forward to the last Hangul character in the word and use that as the starting point for the 
	// line-break iterator, as this will correctly handle the remaining Geumchik wrapping rules, without also causing per-syllable wrapping
	if (GetHangulTextWrappingMethod() == EHangulTextWrappingMethod::PerWord)
	{
		CharIt.setIndex32(InternalPosition);

		if (IsHangul(CharIt.current32()))
		{
			// Walk to the end of the Hangul characters
			while (CharIt.hasNext() && IsHangul(CharIt.next32()))
			{
				InternalPosition = CharIt.getIndex();
			}
		}
	}

	InternalPosition = LineBrkIt->following(InternalPosition);
	CurrentPosition = CharIt.InternalIndexToSourceIndex(InternalPosition);

	return CurrentPosition;
}
```

The `MoveToNext()` calls the `MoveToNextImpl()`. And, the `MoveToNextImpl()` change the `InternalPosition`, which is used for finding a location in text.

```cpp
// UnrealEngine/Engine/Source/ThirdParty/ICU/icu4c-64_1/include/unicode/brkiter.h

/**
 * Advance the iterator to the first boundary following the specified offset.
 * The value returned is always greater than the offset or
 * the value BreakIterator.DONE
 * @param offset the offset to begin scanning.
 * @return The first boundary after the specified offset.
 * @stable ICU 2.0
 */
virtual int32_t following(int32_t offset) = 0;
```

The `InternalPosition` is passed into `following` and it is the code of ICU library.

```
[index] 0123456789...
[array] Text Block Test

[flow]
PreviousBreak = 0, CurrentBreak = 0
MoveToNext()
PreviousBreak = 0, CurrentBreak = 5
CreateBreakCandidate()
PreviousBreak = 5, CurrentBreak = 5
MoveToNext()
PreviousBreak = 5, CurrentBreak = 11
CreateBreakCandidate()
PreviousBreak = 11, CurrentBreak = 11
...
```

In our test text, the flow looks like above.

```cpp
struct FBreakCandidate
{
	/** Range inclusive of trailing whitespace, as used to visually display and interact with the text */
	FTextRange ActualRange;
	/** Range exclusive of trailing whitespace, as used to perform wrapping on a word boundary */
	FTextRange TrimmedRange;
	/** Measured size inclusive of trailing whitespace, as used to visually display and interact with the text */
	FVector2D ActualSize;
	/** Measured width exclusive of trailing whitespace, as used to perform wrapping on a word boundary */
	float TrimmedWidth;
	/** If this break candidate has trailing whitespace, this is the width of the first character of the trailing whitespace */
	float FirstTrailingWhitespaceCharWidth;

	int16 MaxAboveBaseline;
	int16 MaxBelowBaseline;

	int8 Kerning;

#if TEXT_LAYOUT_DEBUG
	FString DebugSlice;
#endif
};
```

A `FBreakCandidate` will be inserted into `BreakCandidates` each iteration. It seems the `FBreakCandidate` knows the size of word (or a part of text). What happened in `CreateBreakCandidate()` ? How could they know the actual size of text ?

# _`Break lines (2/3); Measuring size of each slice`_

```cpp
FTextLayout::FBreakCandidate FTextLayout::CreateBreakCandidate( int32& OutRunIndex, FLineModel& Line, int32 PreviousBreak, int32 CurrentBreak )
{
	...
	// We need to consider the Runs when detecting and measuring the text lengths of Lines because
	// the font style used makes a difference.
	const int32 FirstRunIndexChecked = OutRunIndex;
	for (; OutRunIndex < Line.Runs.Num(); OutRunIndex++)
	{
		FRunModel& Run = Line.Runs[ OutRunIndex ];
		const FTextRange Range = Run.GetTextRange();

		FVector2D SliceSize;
		FVector2D SliceSizeWithoutTrailingWhitespace;
		int32 StopIndex = PreviousBreak;

		WhitespaceStopIndex = StopIndex = FMath::Min( Range.EndIndex, CurrentBreak );
		int32 BeginIndex = FMath::Max( PreviousBreak, Range.BeginIndex );

		while( WhitespaceStopIndex > BeginIndex && FText::IsWhitespace( (*Line.Text)[ WhitespaceStopIndex - 1 ] ) )
		{
			--WhitespaceStopIndex;
		}

		if ( BeginIndex == StopIndex )
		{
			// This slice is empty, no need to adjust anything
			SliceSize = SliceSizeWithoutTrailingWhitespace = FVector2D::ZeroVector;
		}
		else if ( BeginIndex == WhitespaceStopIndex )
		{
			// This slice contains only whitespace, no need to adjust SliceSizeWithoutTrailingWhitespace
			SliceSize = Run.Measure( BeginIndex, StopIndex, Scale, RunTextContext );
			SliceSizeWithoutTrailingWhitespace = FVector2D::ZeroVector;
		}
		else if ( WhitespaceStopIndex != StopIndex )
		{
			// This slice contains trailing whitespace, measure the text size, then add on the whitespace size
			SliceSize = SliceSizeWithoutTrailingWhitespace = Run.Measure( BeginIndex, WhitespaceStopIndex, Scale, RunTextContext );
			const float WhitespaceWidth = Run.Measure( WhitespaceStopIndex, StopIndex, Scale, RunTextContext ).X;
			SliceSize.X += WhitespaceWidth;

			// We also need to measure the width of the first piece of trailing whitespace
			if ( WhitespaceStopIndex + 1 == StopIndex )
			{
				// Only have one piece of whitespace
				FirstTrailingWhitespaceCharWidth = WhitespaceWidth;
			}
			else
			{
				// Deliberately use the run version of Measure as we don't want the run model to cache this measurement since it may be out of order and break the binary search
				FirstTrailingWhitespaceCharWidth = Run.GetRun()->Measure( WhitespaceStopIndex, WhitespaceStopIndex + 1, Scale, RunTextContext ).X;
			}
		}
		else
		{
			// This slice contains no whitespace, both sizes are the same and can use the same measurement
			SliceSize = SliceSizeWithoutTrailingWhitespace = Run.Measure( BeginIndex, StopIndex, Scale, RunTextContext );
		}
	...
}
```

The `CreateBreakCandidate()` function is quite big size, about 200 lines. But the core of function is to calculate a size of slice. Do you remember the variable `CurrentBreak` that indicates next word ? Here, the function make a slice according to `CurrentBreak` and trim it. Trimming happens in while statement, which decreases the `WhitespaceStopIndex` until it indicates an end of last word.

{% asset_img 04.png %}

The `WhitespaceStopIndex` would be 4 in our test text. That is because the index of first whitespace is 4 in `Text Block Test`. Eventually, we will enter the function `Measure()` as the slice is not empty. The only case that `Measure()` not called is when `BeginIndex == StopIndex` is true, in other words `CurrentBreak == 0`.

```cpp
// TextLayout.cpp
FVector2D FTextLayout::FRunModel::Measure(int32 BeginIndex, int32 EndIndex, float InScale, const FRunTextContext& InTextContext)
{
	FVector2D Size = Run->Measure(BeginIndex, EndIndex, InScale, InTextContext);

	MeasuredRanges.Add( FTextRange( BeginIndex, EndIndex ) );
	MeasuredRangeSizes.Add(Size);

	return Size;
}

// SlateTextRun.cpp
FVector2D FSlateTextRun::Measure( int32 BeginIndex, int32 EndIndex, float Scale, const FRunTextContext& TextContext ) const 
{
	const FVector2D ShadowOffsetToApply((EndIndex == Range.EndIndex) ? FMath::Abs(Style.ShadowOffset.X * Scale) : 0.0f, FMath::Abs(Style.ShadowOffset.Y * Scale));

	// Offset the measured shaped text by the outline since the outline was not factored into the size of the text
	// Need to add the outline offsetting to the beginning and the end because it surrounds both sides.
	const float ScaledOutlineSize = Style.Font.OutlineSettings.OutlineSize * Scale;
	const FVector2D OutlineSizeToApply((BeginIndex == Range.BeginIndex ? ScaledOutlineSize : 0) + (EndIndex == Range.EndIndex ? ScaledOutlineSize : 0), ScaledOutlineSize);

	if (EndIndex - BeginIndex == 0)
	{
		return FVector2D(0, GetMaxHeight(Scale)) + ShadowOffsetToApply + OutlineSizeToApply;
	}

	// Use the full text range (rather than the run range) so that text that spans runs will still be shaped correctly
	return ShapedTextCacheUtil::MeasureShapedText(TextContext.ShapedTextCache, FCachedShapedTextKey(FTextRange(0, Text->Len()), Scale, TextContext, Style.Font), FTextRange(BeginIndex, EndIndex), **Text) + ShadowOffsetToApply + OutlineSizeToApply;
}
```

We will get a FVector2D from `FSlateTextRun::Measure()`, which is the size of slice. The code `Run->Measure()` is the same with calling `ShapedTextCacheUtil::MeasureShapedText()` when you are using a TextBlock. Calculating shadow offset is not important in this post, so we need to focus on `ShapedTextCacheUtil::MeasureShapedText()`.

```cpp
// ShapedTextFwd.h
typedef TSharedRef<const FShapedGlyphSequence> FShapedGlyphSequenceRef;

// ShapedTextCache.cpp
FVector2D ShapedTextCacheUtil::MeasureShapedText(const FShapedTextCacheRef& InShapedTextCache, const FCachedShapedTextKey& InRunKey, const FTextRange& InMeasureRange, const TCHAR* InText)
{
	// Get the shaped text for the entire run and try and take a sub-measurement from it - this can help minimize the amount of text shaping that needs to be done when measuring text
	FShapedGlyphSequenceRef ShapedText = InShapedTextCache->FindOrAddShapedText(InRunKey, InText);

	TOptional<int32> MeasuredWidth = ShapedText->GetMeasuredWidth(InMeasureRange.BeginIndex, InMeasureRange.EndIndex);
	if (!MeasuredWidth.IsSet())
	{
		FCachedShapedTextKey MeasureKey = InRunKey;
		MeasureKey.TextRange = InMeasureRange;

		// Couldn't measure the sub-range, try and measure from a shape of the specified range
		ShapedText = InShapedTextCache->FindOrAddShapedText(MeasureKey, InText);
		MeasuredWidth = ShapedText->GetMeasuredWidth();
	}

	check(MeasuredWidth.IsSet());
	return FVector2D(MeasuredWidth.GetValue(), ShapedText->GetMaxTextHeight());
}
```

As you can see, the `FShapedGlyphSequenceRef` is a shared reference of `FShapedGlyphSequence`. Then, what the hell is `FShapedGlyphSequence` ? And what it does ?

```cpp
FShapedGlyphSequenceRef FShapedTextCache::FindOrAddShapedText(const FCachedShapedTextKey& InKey, const TCHAR* InText)
{
	FShapedGlyphSequencePtr ShapedText = FindShapedText(InKey);

	if (!ShapedText.IsValid())
	{
		ShapedText = AddShapedText(InKey, InText);
	}

	return ShapedText.ToSharedRef();
}

FShapedGlyphSequencePtr FShapedTextCache::FindShapedText(const FCachedShapedTextKey& InKey) const
{
	FShapedGlyphSequencePtr ShapedText = CachedShapedText.FindRef(InKey);

	if (ShapedText.IsValid() && !ShapedText->IsDirty())
	{
		return ShapedText;
	}
	
	return nullptr;
}

FShapedGlyphSequenceRef FShapedTextCache::AddShapedText(const FCachedShapedTextKey& InKey, FShapedGlyphSequenceRef InShapedText)
{
	CachedShapedText.Add(InKey, InShapedText);
	return InShapedText;
}
```

First, engine try to find if there is already existing one. If not, create new one and insert it into the cache.

```cpp
// FontCache.h
/** Information for rendering a shaped text sequence */
class SLATECORE_API FShapedGlyphSequence
{
	...
	/** Array of glyphs in this sequence. This data will be ordered so that you can iterate and draw left-to-right, which means it will be backwards for right-to-left languages */
	TArray<FShapedGlyphEntry> GlyphsToRender;
	...

/** Information for rendering one glyph in a shaped text sequence */
struct FShapedGlyphEntry
{
	...
	/** The index of this glyph from the source text. The source indices may skip characters if the sequence contains ligatures, additionally, some characters produce multiple glyphs leading to duplicate source indices */
	int32 SourceIndex = 0;
	/** The amount to advance in X before drawing the next glyph in the sequence */
	int16 XAdvance = 0;
	...

// FontCache.cpp
FShapedGlyphSequence::FShapedGlyphSequence(TArray<FShapedGlyphEntry> InGlyphsToRender, const int16 InTextBaseline, const uint16 InMaxTextHeight, const UObject* InFontMaterial, const FFontOutlineSettings& InOutlineSettings, const FSourceTextRange& InSourceTextRange)
	: GlyphsToRender(MoveTemp(InGlyphsToRender))
	, TextBaseline(InTextBaseline)
	, MaxTextHeight(InMaxTextHeight)
	, FontMaterial(InFontMaterial)
	, OutlineSettings(InOutlineSettings)
	, SequenceWidth(0)
	, GlyphFontFaces()
	, SourceIndicesToGlyphData(InSourceTextRange)
{
	const int32 NumGlyphsToRender = GlyphsToRender.Num();
	for (int32 CurrentGlyphIndex = 0; CurrentGlyphIndex < NumGlyphsToRender; ++CurrentGlyphIndex)
	{
		const FShapedGlyphEntry& CurrentGlyph = GlyphsToRender[CurrentGlyphIndex];

		// Track unique font faces
		if (CurrentGlyph.FontFaceData->FontFace.IsValid())
		{
			GlyphFontFaces.AddUnique(CurrentGlyph.FontFaceData->FontFace);
		}

		// Update the measured width
		SequenceWidth += CurrentGlyph.XAdvance;
		...
```

The `FShapedGlyphSequence` has a TArray of `FShapedGlyphEntry`. And the `FShapedGlyphEntry` has several properties such as `SourceIndex` and `XAdvance`. Looks like the `FShapedGlyphEntry` has properties responding each character in text, and the `FShapedGlyphSequence` has properties responding whole text. The properties are for how to render the text appropriately. So here, we can regard the term `Glyph` as one single character.

```cpp
// SlateTextShaper.cpp
void FSlateTextShaper::PerformKerningOnlyTextShaping(const TCHAR* InText, const int32 InTextStart, const int32 InTextLen, const FSlateFontInfo& InFontInfo, const float InFontScale, TArray<FShapedGlyphEntry>& OutGlyphsToRender) const
{
	...
	for (int32 SequenceCharIndex = 0; SequenceCharIndex < KerningOnlyTextSequenceEntry.TextLength; ++SequenceCharIndex)
	{
		const int32 CurrentCharIndex = KerningOnlyTextSequenceEntry.TextStartIndex + SequenceCharIndex;
		const TCHAR CurrentChar = InText[CurrentCharIndex];

		if (!InsertSubstituteGlyphs(InText, CurrentCharIndex, ShapedGlyphFaceData, AdvanceCache, OutGlyphsToRender, LetterSpacingScaled))
		{
			uint32 GlyphIndex = FT_Get_Char_Index(KerningOnlyTextSequenceEntry.FaceAndMemory->GetFace(), CurrentChar);

			// If the given font can't render that character (as the fallback font may be missing), try again with the fallback character
			if (CurrentChar != 0 && GlyphIndex == 0)
			{
				GlyphIndex = FT_Get_Char_Index(KerningOnlyTextSequenceEntry.FaceAndMemory->GetFace(), SlateFontRendererUtils::InvalidSubChar);
			}

			int16 XAdvance = 0;
			{
				FT_Fixed CachedAdvanceData = 0;
				if (AdvanceCache->FindOrCache(GlyphIndex, CachedAdvanceData))
				{
					XAdvance = FreeTypeUtils::Convert26Dot6ToRoundedPixel<int16>((CachedAdvanceData + (1<<9)) >> 10);
				}
			}

			const int32 CurrentGlyphEntryIndex = OutGlyphsToRender.AddDefaulted();
			FShapedGlyphEntry& ShapedGlyphEntry = OutGlyphsToRender[CurrentGlyphEntryIndex];
			ShapedGlyphEntry.FontFaceData = ShapedGlyphFaceData;
			ShapedGlyphEntry.GlyphIndex = GlyphIndex;
			ShapedGlyphEntry.SourceIndex = CurrentCharIndex;
			ShapedGlyphEntry.XAdvance = XAdvance;
	...
```

Usually, the `XAdvande` is determined at `FSlateTextShaper::PerformKerningOnlyTextShaping()`. Engine uses [the FreeType library](https://gitlab.freedesktop.org/freetype/freetype) for getting a estimated size of character when it rendered. The `GlyphIndex` is calculated based on font and character value.

```cpp
bool FFreeTypeAdvanceCache::FindOrCache(const uint32 InGlyphIndex, FT_Fixed& OutCachedAdvance)
{
	// Try and find the advance from the cache...
	{
		const FT_Fixed* FoundCachedAdvance = AdvanceMap.Find(InGlyphIndex);
		if (FoundCachedAdvance)
		{
			OutCachedAdvance = *FoundCachedAdvance;
			return true;
		}
	}

	FreeTypeUtils::ApplySizeAndScale(Face, FontSize, FontScale);

	// No cached data, go ahead and add an entry for it...
	const FT_Error Error = FT_Get_Advance(Face, InGlyphIndex, LoadFlags, &OutCachedAdvance);
	if (Error == 0)
	{
		if (!FT_IS_SCALABLE(Face) && FT_HAS_FIXED_SIZES(Face))
		{
			// Fixed size fonts don't support scaling, but we calculated the scale to use for the glyph in ApplySizeAndScale
			OutCachedAdvance = FT_MulFix(OutCachedAdvance, ((LoadFlags & FT_LOAD_VERTICAL_LAYOUT) ? Face->size->metrics.y_scale : Face->size->metrics.x_scale));
		}

		AdvanceMap.Add(InGlyphIndex, OutCachedAdvance);
		return true;
	}

	return false;
}
```

The code `AdvanceCache->FindOrCache(GlyphIndex, CachedAdvanceData)` finds at cache, but it creates new one and cache it if could not find. The `FT_Get_Advance()` returns the result with parameter `&OutCachedAdvance`. We can get a size of single character through the function because the value `GlyphIndex` includes information of font and character value.

{% asset_img 05.png %}

In our test text `Text Block Test`, the result is like below:

Index | Character | GlyphIndex | XAdvance
--- | --- | --- | ---
0 | `T` | 55 | 18
1 | `e` | 72 | 17
2 | `x` | 91 | 16
3 | `t` | 87 | 11
4 | ` ` | 3 | 8
5 | `B` | 37 | 20
6 | `l` | 79 | 9
7 | `o` | 82 | 18
8 | `c` | 70 | 17
9 | `k` | 78 | 17
10 | ` ` | 3 | 8
11 | `T` | 55 | 18
12 | `e` | 72 | 17
13 | `s` | 86 | 17
14 | `t` | 87 | 11

{% asset_img 07.png %}

You can see that the same character has the same XAdvance value. For example, The character `T` has `55` of GlyphIndex and `18` of XAdvance. Go back to the `ShapedTextCacheUtil::MeasureShapedText()`, that is why the `MeasuredWidth` has a value of `220 ≒ 222 = 18 + 17 + ... + 17 + 11`. The difference `2` occurs by the kerning.

{% asset_img 06.png %}

The final width may differ a little bit because some combination of characters need a kerning. For example, though `e` and `k` have the same XAdvance value `17`, a combination `Te` has less size than a combination `Tk`. Because in the combination `Te`, `e` can stick to `T` closer than `k` in `Tk`. In other words, a character `T` can have XAdvance of `17` in the combinations such as `Ta/Tc/Td`, and so on. Otherwise such as `Tb/Tf/Th`, it can have XAdvance of `18`.

# _`Break lines (3/3); Creating lines with wrapping`_

{% asset_img 08.png %}

Go back to the `FTextLayout::CreateLineWrappingCache()`, now we can wrap text according to size (exactly, width) of each slice. All slices are stored at the container `BreakCandidates`. In our test text `Text Block Test`, the result is like below:

BreakCandidates | ActualRange | TrimmedRange
--- | --- | ---
0 | `Text ` [0, 5) | `Text` [0, 4)
1 | `Block ` [5, 11) | `Block` [5, 10)
2 | `Test` [11, 15) | `Test` [11, 15)

※ `[0, 5)` is equal to `[0, 4]`

Do you remember there is a code calls `FTextLayout::FlowLineLayout()` in `FTextLayout::FlowLayout()` ?

```cpp
void FTextLayout::FlowLineLayout(const int32 LineModelIndex, const float WrappingDrawWidth, TArray<TSharedRef<ILayoutBlock>>& SoftLine)
{
	...
	float CurrentWidth = 0.0f;
	for (int32 BreakIndex = 0; BreakIndex < LineModel.BreakCandidates.Num(); BreakIndex++)
	{
		const FBreakCandidate& Break = LineModel.BreakCandidates[ BreakIndex ];

		const bool IsLastBreak = BreakIndex + 1 == LineModel.BreakCandidates.Num();
		const bool IsFirstBreakOnSoftLine = CurrentWidth == 0.0f;
		const int8 Kerning = ( IsFirstBreakOnSoftLine ) ? Break.Kerning : 0;
		const bool BreakDoesFit = CurrentWidth + Break.ActualSize.X + Kerning <= WrappingDrawWidth;
		const bool BreakWithoutTrailingWhitespaceDoesFit = CurrentWidth + Break.TrimmedWidth + Kerning <= WrappingDrawWidth;
		...
```

Here, we accumulate a width of each BreakCandidate on `CurrentWidth`. And wrapping text occurs whenever `CurrentWidth` almost reaches to `WrappingDrawWidth`.

```cpp
else if ( !BreakDoesFit || IsLastBreak )
{
	const bool IsFirstBreak = BreakIndex == 0;

	const FBreakCandidate& FinalBreakOnSoftLine = ( !IsFirstBreak && !IsFirstBreakOnSoftLine && !BreakWithoutTrailingWhitespaceDoesFit ) ? LineModel.BreakCandidates[ --BreakIndex ] : Break;
	
	// We want the wrapped line width to contain the first piece of trailing whitespace for a line, however we only do this if we have trailing whitespace
	// otherwise very long non-breaking words can cause the wrapped line width to expand beyond the desired wrap width
	float WrappedLineWidth = CurrentWidth;
	if ( BreakWithoutTrailingWhitespaceDoesFit )
	{
		// This break has trailing whitespace
		WrappedLineWidth += ( FinalBreakOnSoftLine.TrimmedWidth + FinalBreakOnSoftLine.FirstTrailingWhitespaceCharWidth );
	}
	else
	{
		// This break is longer than the wrapping point, so make sure and clamp the line size to the given wrapping width
		WrappedLineWidth += FinalBreakOnSoftLine.ActualSize.X;
		WrappedLineWidth = FMath::Min(WrappedLineWidth, WrappingDrawWidth);
	}

	// We want wrapped lines to ignore any trailing whitespace when justifying
	// If FinalBreakOnSoftLine isn't the current Break, then the size of FinalBreakOnSoftLine (including its trailing whitespace) will have already
	// been added to CurrentWidth, so we need to remove that again before adding the trimmed width (which is the width we should justify with)
	// We should not attempt to adjust the last break on a soft-line as that might have explicit trailing whitespace
	TOptional<float> JustifiedLineWidth;
	if ( &FinalBreakOnSoftLine != &LineModel.BreakCandidates.Last() )
	{
		JustifiedLineWidth = CurrentWidth - (&FinalBreakOnSoftLine == &Break ? 0.0f : FinalBreakOnSoftLine.ActualSize.X) + FinalBreakOnSoftLine.TrimmedWidth;
	}

	CreateLineViewBlocks( LineModelIndex, FinalBreakOnSoftLine.ActualRange.EndIndex, WrappedLineWidth, JustifiedLineWidth, /*OUT*/CurrentRunIndex, /*OUT*/CurrentRendererIndex, /*OUT*/PreviousBlockEnd, SoftLine );

	if ( CurrentRunIndex < LineModel.Runs.Num() && FinalBreakOnSoftLine.ActualRange.EndIndex == LineModel.Runs[ CurrentRunIndex ].GetTextRange().EndIndex )
	{
		++CurrentRunIndex;
	}

	PreviousBlockEnd = FinalBreakOnSoftLine.ActualRange.EndIndex;

	CurrentWidth = 0.0f;
	SoftLine.Reset();
} 
```

Usually, when wrapping text needed, the codes above would be executed. `FinalBreakOnSoftLine` indicates the BreakCandidate that needs a new line after itself. In our test text `Text Block Test`, `Text ` could be assigned.

```cpp
void FTextLayout::CreateLineViewBlocks( int32 LineModelIndex, const int32 StopIndex, const float WrappedLineWidth, const TOptional<float>& JustificationWidth, int32& OutRunIndex, int32& OutRendererIndex, int32& OutPreviousBlockEnd, TArray< TSharedRef< ILayoutBlock > >& OutSoftLine )
{
	...
	// Add the new block
	{
		FBlockDefinition BlockDefine;
		BlockDefine.ActualRange = FTextRange(BlockBeginIndex, BlockStopIndex);
		BlockDefine.Renderer = BlockRenderer;

		OutSoftLine.Add( Run.CreateBlock( BlockDefine, Scale, FLayoutBlockTextContext(RunTextContext, BlockTextDirection) ) );
		OutPreviousBlockEnd = BlockStopIndex;

		// Update the soft line bounds based on this new block (needed within this loop due to bi-directional text, as the extents of the line array are not always the start and end of the range)
		const FTextRange& BlockRange = OutSoftLine.Last()->GetTextRange();
		SoftLineRange.BeginIndex = FMath::Min(SoftLineRange.BeginIndex, BlockRange.BeginIndex);
		SoftLineRange.EndIndex   = FMath::Max(SoftLineRange.EndIndex, BlockRange.EndIndex);
	}
	...
	FTextLayout::FLineView LineView;
	LineView.Offset = CurrentOffset;
	LineView.Size = LineSize;
	LineView.TextHeight = UnscaleLineHeight;
	LineView.JustificationWidth = JustificationWidth.Get(LineView.Size.X);
	LineView.Range = SoftLineRange;
	LineView.TextBaseDirection = LineModel.TextBaseDirection;
	LineView.ModelIndex = LineModelIndex;
	LineView.Blocks.Append( OutSoftLine );

	LineViews.Add( LineView );
	...
```

The function `FTextLayout::CreateLineViewBlocks()` creates new `FTextLayout::FLineView` and adds it into initialized `LineViews`. We already cleared the `LineViews` at the function `FTextLayout::ClearView()`. In our test txt, after all process, the `LineViews` will have the value like below:

LineViews | Range
--- | ---
0 | [0, 5)
1 | [5, 11)
2 | [11, 15)

{% asset_img 09.png %}

Finally, we found that the result of wrapping text. All of prerequisites are for splitting a text. Now we understand how the text can be wrapped in UnrealEngine.

# _`Wrap-up`_

Text wrapping in UnrealEngine can be divided into 3 major steps.

1. `Separating a text into slices`
Find where each word ends using ICU library.
Separate text into slices based on the indices.

2. `Measuring size of each slice`
Estimate size of rendered character using FreeType library.
Apply several modifications such as kerning, shadow, and so on.

3. `Creating lines with wrapping`
Add width until it reaches the wrapping width.
When it reaches, create new line.