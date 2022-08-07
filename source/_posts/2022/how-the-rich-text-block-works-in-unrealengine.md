---
title: How the RichTextBlock works in UnrealEngine (part.1)
date: 2022-07-28 20:59:20
tags: [UnrealEngine, TextBlock, RichTextBlock]
---

| Environment | |
| --- | --- |
| UnrealEngine | `branch: 5.0` |
| Visual Studio 2022 | `version: 17.2.6` |
| Windows 11 Pro | `build: 22000.795` |

# _`Overview`_

We have learned about the `TextBlock` in UnrealEngine at [the previous post](https://baemincheon.github.io/2022/03/22/how-the-text-wrap-works-in-unrealengine/). As we saw, the `TextBlock` provides the function to split a long text into multiple lines. But, it was only for the text, combinations of character.

Sometimes, we want to put something that is not a character in the middle of text. For example, you may want to put an image for key icon into the text that describes character's skill. Maybe, you want to highlight a part of the text by coloring it. Furthermore, you could want to put a "widget" in the middle of text. The widget would interact with the player's action so that they can have better experience of the user interface.

{% asset_img 01.png %}
> An option description in PUBG Xbox.

UnrealEngine has a solution for that, [the RichTextBlock widget](https://docs.unrealengine.com/4.27/en-US/InteractiveExperiences/UMG/UserGuide/UMGRichTextBlock/). You can put an image or anything else in the middle of text. Plus, it also supports auto-wrapping just like at the `TextBlock`. Now you know why its name is the "Rich"TextBlock. Then, let us check out how the `RichTextBlock` widget is implemented and how it works.

# _`An example`_

Already there is a tutorial in [the document](https://docs.unrealengine.com/4.27/en-US/InteractiveExperiences/UMG/UserGuide/UMGRichTextBlock/), but I will show you an another example including how to make your custom `RichTextBlock` decorator. Suppose you want to display the text like the screenshot below.

{% asset_img 02.png %}

The size of `SizeBox` is (512, 512). The text used in `RichTextBlock` is here:

```
Test <Emphasis> Test </> <somewidget id="Ferris_02"/> Test <somewidget id="Ferris_01"/> Test aaa aaa aaa aaa aaa aaa aaa aaa <img id="Ferris_01"/> aaa <somewidget id="Ferris_02"/> aaa
```

As we can see, the images are put in the middle of text. The `RichTextBlock` parses the input text and decorates the text with your configurations. Without some configurations, the tags such as `<Emphasis>` and `<somewidget>` would be displayed as a plain text. Yes, you should do some configurations for using the `RichTextBlock`.

{% asset_img 03.png %}

I have already set some properties, `TextStyleSet` and `DecoratorClasses`.

# _`RichTextStyle`_

The `TextStyleSet` is used for decorating a text just like a markup. You can specify a font, size, color, and so on with it. I made two data rows in the data table, and that is why some of text was displayed with green color. Check the screenshot below.

{% asset_img 04.png %}
<br/>
{% asset_img 05.png %}

The `RichTextBlock` decorates rest of text if you make a `Default` row. That is why the text not embraced with tags was displayed with white color.

{% asset_img 06.png %}

Without the `TextStyleSet`, the `RichTextBlock` cannot display the text properly. You can make a data table containing `RichTextStyleRow` with the instructions.

```
1. Right click on contents browser.
2. find the item `Data Table` at the category `Miscellaneous`, and click it.
3. The dialog `Pick Row Structure` pops up.
4. Click the drop down, and select `RichTextStyleRow`.
```

Now, you can manipulate the data table. But, you should be careful that the name of data row is the same with the name of tag in the `RichTextBlock`.

# _`RichImage`_

```cpp
/** Simple struct for rich text styles */
USTRUCT(Blueprintable, BlueprintType)
struct UMG_API FRichImageRow : public FTableRowBase
{
	GENERATED_USTRUCT_BODY()

public:

	UPROPERTY(EditAnywhere, Category = Appearance)
	FSlateBrush Brush;
};

/**
 * Allows you to setup an image decorator that can be configured
 * to map certain keys to certain images.  We recommend you subclass this
 * as a blueprint to configure the instance.
 *
 * Understands the format <img id="NameOfBrushInTable"></>
 */
UCLASS(Abstract, Blueprintable)
class UMG_API URichTextBlockImageDecorator : public URichTextBlockDecorator
```

UnrealEngine provides a decorator class, `URichTextBlockImageDecorator`. It helps you add an image widget in the middle of text.

{% asset_img 07.png %}

Without it, the `RichTextBlock` cannot create an image from the tag `img`. You can make a data table containing `RichImageRow` with the instructions.

```
1. Right click on contents browser.
2. find the item `Data Table` at the category `Miscellaneous`, and click it.
3. The dialog `Pick Row Structure` pops up.
4. Click the drop down, and select `RichImageRow`.
```

Now, you can manipulate the data table. Also, you should be careful that the name of data row is the same with the name of tag in the `RichTextBlock` as I mentioned at the `RichTextStyle`. So, remember it because this mechanism will work on other cases (Decorators using their own data table) too.

{% asset_img 08.png %}

However, you need one step more to apply the data table.

```cpp
// Engine/Source/Runtime/UMG/Public/Components/RichTextBlock.h

/**  */
UPROPERTY(EditAnywhere, Category=Appearance, meta=(RequiredAssetDataTags = "RowStructure=RichTextStyleRow"))
TObjectPtr<class UDataTable> TextStyleSet;

/**  */
UPROPERTY(EditAnywhere, Category=Appearance)
TArray<TSubclassOf<URichTextBlockDecorator>> DecoratorClasses;
```

The `TextStyleSet` needs only a data table, but the `DecoratorClasses` takes a class inherits `URichTextBlockDecorator`. That is why `URichTextBlockImageDecorator` inherits that.

{% asset_img 11.png %}
<br/>
{% asset_img 10.png %}
<br/>
{% asset_img 09.png %}

So, you should create a blueprint class inherits `URichTextBlockImageDecorator` because the class `URichTextBlockImageDecorator` has the UCLASS keyword `Abstract`. And, assign it into the `DecoratorClasses` at the `RichTextBlock` widget. The blueprint class should reference the data table for images.

# _`Custom decorator`_

I have written a custom decorator for this example, the `URichTextBlockSomeWidgetDecorator`. As you can see in the example, it displays a combination of image and text. First of all, the code for this class is here.

- [RichTextBlockSomeWidgetDecorator.h](https://gist.github.com/BaeMinCheon/109b251f1c88ae3f7e73dd711d0cc153)
- [RichTextBlockSomeWidgetDecorator.cpp](https://gist.github.com/BaeMinCheon/6521a88bce86bd4cd60b7b383cf45637)

And the followings are the major changes.

```cpp
public class TestRichTextBlock : ModuleRules
{
	public TestRichTextBlock(ReadOnlyTargetRules Target) : base(Target)
	{
		PCHUsage = PCHUsageMode.UseExplicitOrSharedPCHs;

		PublicDependencyModuleNames.AddRange(new string[] { "Core", "CoreUObject", "Engine", "InputCore", "HeadMountedDisplay", "UMG", "Slate", "SlateCore" });
	}
}
```

You must add the modules at your `Build.cs`: `UMG`, `Slate`, and `SlateCore`.

```cpp
bool FRichInlineSomeWidget::Supports(const FTextRunParseResults& RunParseResult, const FString& Text) const
{
	bool Result = false;
	const bool IsContainId = RunParseResult.MetaData.Contains(TEXT("id"));
	const bool IsNameSomeWidget = RunParseResult.Name == TEXT("somewidget");
	if (IsContainId && IsNameSomeWidget)
	{
		const FTextRange& IdRange = RunParseResult.MetaData[TEXT("id")];
		const FString TagId = Text.Mid(IdRange.BeginIndex, IdRange.EndIndex - IdRange.BeginIndex);
		const bool bWarnIfMissing = false;
		Result = Decorator->FindSomeWidgetRow(*TagId, bWarnIfMissing) != nullptr;
	}
	return Result;
}
```

I have changed the tag that my decorator supports. `img -> somewidget`

```cpp
void SRichInlineSomeWidget::Construct(const FArguments& InArgs, const FRichSomeWidgetRow* Row, const FTextBlockStyle& TextStyle, TOptional<int32> Width, TOptional<int32> Height, EStretch::Type Stretch)
{
	const FSlateBrush* InBrush = &(Row->Brush);
	check(InBrush)
	const FText InText = Row->Text;
	const TSharedRef<FSlateFontMeasure> FontMeasure = FSlateApplication::Get().GetRenderer()->GetFontMeasureService();
	const float MaxHeight = FontMeasure->GetMaxCharacterHeight(TextStyle.Font, 1.0f);
	float IconHeight = FMath::Max(MaxHeight, InBrush->ImageSize.Y);
	if (Height.IsSet())
	{
		IconHeight = Height.GetValue();
	}
	float IconWidth = IconHeight;
	if (Width.IsSet())
	{
		IconWidth = Width.GetValue();
	}
	ChildSlot
	[
		SNew(SBox)
		[
			SNew(SHorizontalBox)

			+SHorizontalBox::Slot()
			.AutoWidth()
			.VAlign(VAlign_Center)
			[
				SNew(SImage)
				.Image(InBrush)
			]

			+SHorizontalBox::Slot()
			.AutoWidth()
			.VAlign(VAlign_Center)
			[
				SNew(STextBlock)
				.Text(InText)
			]
		]
	];
}
```

Used the max value for `IconHeight` because I wanted to display the image properly. Plus, the decorator has a `TextBlock` for descripting an image. In the example, a text `Ferris_01` or `Ferris_02` is located on the right of Ferris' image.

So, you can create a custom decorator like this. Rest works are just similar with `RichImage`, creating some blueprint classes (decorator and data table) and assigning each other. Let your decorator have awesome functions :)

# _`Preview of Part #2`_

At this part, we have seen how to use the `RichTextBlock` and how to make a custom decorator.

- Only for a text, you should create a data table and assign it.
- For other content, you should create a custom decorator and assign it. But, UnrealEngine has already made a default decorator for an image, `URichTextBlockImageDecorator`.
- When creating a custom decorator, you should know them below:
	- Specify an unique tag name. There should be no confliction.
	- Design a widget layout with Slate. You can reference many examples from engine codes, just find all references of `ChildSlot`.
	- Create a SWidget version of your widget if you want to put your widget into the custom decorator. As you can see, the `SNew` accepts only the class inherits `SWidget`. In most of cases, it is okay to inherit the class `SLeafWidget`.

At next part, we would find out how does the `RichTextBlock` wrap its contents. It will be interesting because the `RichTextBlock` can have an image as a content.