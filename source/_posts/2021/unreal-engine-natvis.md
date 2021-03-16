---
title: unreal-engine-natvis
date: 2021-03-01 22:10:07
tags: [UnrealEngine, Natvis]
---

- this post covers
    - what is Natvis
    - how to write custom Natvis visualization
    - customization in UnrealEngine
- environment
    - Unreal Engine 4 `ver. 4.25`
    - Visual Studio 2019 `ver. 16.8.1`
- reference
    1. [https://docs.microsoft.com/en-us/visualstudio/debugger/create-custom-views-of-native-objects?view=vs-2019][reference #1]
    2. [https://docs.microsoft.com/en-us/previous-versions/visualstudio/visual-studio-2015/debugger/format-specifiers-in-cpp?view=vs-2015&redirectedfrom=MSDN][reference #2]
	3. [https://www.ibm.com/support/knowledgecenter/SSEQTP_liberty/com.ibm.websphere.wlp.doc/ae/rwlp_xml_escape.html][reference #3]

[reference #1]: https://docs.microsoft.com/en-us/visualstudio/debugger/create-custom-views-of-native-objects?view=vs-2019
[reference #2]: https://docs.microsoft.com/en-us/previous-versions/visualstudio/visual-studio-2015/debugger/format-specifiers-in-cpp?view=vs-2015&redirectedfrom=MSDN
[reference #3]: https://www.ibm.com/support/knowledgecenter/SSEQTP_liberty/com.ibm.websphere.wlp.doc/ae/rwlp_xml_escape.html

# NATive type VISualization
According to [reference #1], the name of Natvis framework means visualization of native types. It can help your debugging with more plentiful visibility, and sometimes support mutiple environments that have a different size on the same data type. Suppose you want to make a string class storing its text as UTF-32 or something custom format, it should not be displayed well because it is not kind of ASCII. Though, do you want to see the data (in this case, the text) at `Watch` or `Local` viewport ? Then you should implement your own xml for custom Natvis visualization.

*SCREENSHOT WITHOUT CUSTOM NATVIS*

{% asset_img 01.png %}

*SCREENSHOT WITH CUSTOM NATVIS*

{% asset_img 02.png %}

# Basic syntax and usage
The custom visualizer has XML syntax and it would be comfortable than a sole programming language. Just create a file of any name with `.natvis` extension and locate at the directory `Documents/Visual Studio 2019/Visualizers`. The Visual Studio IDE will find all natvis files at there and parse them. At first after you create the file, you need the tag `AutoVisualizer`.

```xml
<AutoVisualizer>
</AutoVisualizer>
```

But you may meet the error like below with our current visualizer. (You should turn on an option for showing errors related to Natvis. Manipulate the option at `Tools/Options/Debugging/Output Window/General OutputSettings/Natvisdiagnostic messages`. I recommend you to set the level as `Error`.)

```
Natvis: ...\Documents\Visual Studio 2019\Visualizers\Example.natvis(1,2): Fatal error:
Expected element with namespace 'http://schemas.microsoft.com/vstudio/debugger/natvis/2010'.
```

So you should specify the schemas. Like this.

```xml
<AutoVisualizer xmlns="http://schemas.microsoft.com/vstudio/debugger/natvis/2010">
</AutoVisualizer>
```

And, the `AutoVisualizer` tag can have a child tag such as `Type`. The `Type` tag must have an attribute `Name`. `Name` can be set as the name of type. For example, you should type `SomeClass` at the attribute when you created a type `SomeClass`.

```cpp
class SomeClass
{
public:
    int ID = 0;
};

int main()
{
    SomeClass Some01;
    Some01.ID = 100;
    
    return 0;
}
```
```xml
<AutoVisualizer xmlns="http://schemas.microsoft.com/vstudio/debugger/natvis/2010">
	<Type Name="SomeClass">
	</Type>
</AutoVisualizer>
```
{% asset_img 03.png %}

The `Type` tag can have a child tag such `DisplayString/Expand`. The `DisplayString` tag can be used for displaying a string at the debugging window like below.

```xml
<AutoVisualizer xmlns="http://schemas.microsoft.com/vstudio/debugger/natvis/2010">
	<Type Name="SomeClass">
		<DisplayString>
			This is my class
		</DisplayString>
	</Type>
</AutoVisualizer>
```
{% asset_img 04.png %}

You can get the value of member variable. Brace the member variable as `{}`.

```xml
<AutoVisualizer xmlns="http://schemas.microsoft.com/vstudio/debugger/natvis/2010">
	<Type Name = "SomeClass">
		<DisplayString>
			My ID is {ID}
		</DisplayString>
	</Type>
</AutoVisualizer>
```
{% asset_img 05.png %}

With the `Expand` tag, you can customize the expanded view. The `Item` tags consist of the list. If you customize the expanded view as `Expand` tag, automatically `Raw View` item created, which was the original expanded view. You can decorate each line of list in expanded view. The specifier `sb` and `x` are respectively meaning "Display the string without quotation marks" and "Display the integer with hexa-decimal format". For more details, visit [reference #2].

```xml
<AutoVisualizer xmlns="http://schemas.microsoft.com/vstudio/debugger/natvis/2010">
	<Type Name = "SomeClass">
		<DisplayString>
			My ID is {ID}
		</DisplayString>
		<Expand>
			<Item Name="Description">
				"Natvis is awesome", sb
			</Item>
			<Item Name="ID">
				ID, x
			</Item>
		</Expand>
	</Type>
</AutoVisualizer>
```
{% asset_img 06.png %}

# Example in UnrealEngine
Let us find an example in UnrealEngine one. You can find `UE4.natvis` if you installed UnrealEngine at your local system. Mostly, the `UE4.natvis` located in `Engine/Extras/VisualStudioDebugging/UE4.natvis`.

```xml
<?xml version="1.0" encoding="utf-8"?>
<AutoVisualizer xmlns="http://schemas.microsoft.com/vstudio/debugger/natvis/2010">
    ...
    <!-- FString visualizer -->
    <Type Name="FString">
        <DisplayString Condition="Data.ArrayNum == 0">Empty</DisplayString>
        <DisplayString Condition="Data.ArrayNum &lt; 0">Invalid</DisplayString>
        <DisplayString Condition="Data.ArrayMax &lt; Data.ArrayNum">Invalid</DisplayString>
        <DisplayString Condition="Data.ArrayMax &gt;= Data.ArrayNum">{Data.AllocatorInstance.Data,su}</DisplayString>
        <StringView Condition="Data.ArrayMax &gt;= Data.ArrayNum">Data.AllocatorInstance.Data,su</StringView>
    </Type>
    ...
</AutoVisualizer>
```

First of all, the `FString` welcomes us. Have a look for why the `FString` visualizer has been made like this. ( `&lt;` and `&gt;` things are escaped characters in xml. For more details, visit [reference #3]. )

{% asset_img 07.png %}

Put a breakpoint at where the `FString` is initialized. Before initialization, we can see `Invalid` at the debugging window.

{% asset_img 08.png %}

Expand the items. We can see the `ArrayNum` has a negative value. The condition `Data.ArrayNum < 0` is satisfied and `Invalid` would be shown.

{% asset_img 09.png %}

After initialization, we can see the string very well. In this case, the condition `Data.ArrayMax >= Data.ArrayNum` is satisfied and `L"ABC"` would be shown. Why does the string look like `L"..."` ? Because of the format specifier `su`. Check the [reference #2] again.

```cpp
Engine/Source/Runtime/Core/Public/Containers/UnrealString.h

class CORE_API FString
{
private:
	friend struct TContainerTraits<FString>;

	/** Array holding the character data */
	typedef TArray<TCHAR> DataType;
	DataType Data;
	...
};
```

The `FString` stores its string with `TArray<TCHAR>`. So we could see the `ArrayNum` or `ArrayMax` things at the `FString` visualizer.

```cpp
Engine/Source/Runtime/Core/Public/Containers/Array.h

template<typename InElementType, typename InAllocator>
class TArray
{
	template <typename OtherInElementType, typename OtherAllocator>
	friend class TArray;

public:
	typedef typename InAllocator::SizeType SizeType;
	typedef InElementType ElementType;
	typedef InAllocator   Allocator;

	...

protected:

	template<typename ElementType, typename Allocator>
	friend class TIndirectArray;

	ElementAllocatorType AllocatorInstance;
	SizeType             ArrayNum;
	SizeType             ArrayMax;
```
```cpp
Engine/Source/Runtime/Core/Public/Containers/ContainersFwd.h

template<int IndexSize> class TSizedDefaultAllocator;
using FDefaultAllocator = TSizedDefaultAllocator<32>;
using FDefaultAllocator64 = TSizedDefaultAllocator<64>;
class FDefaultSetAllocator;

class FString;

template<typename T, typename Allocator = FDefaultAllocator> class TArray;
...
```
```cpp
Engine/Source/Runtime/Core/Public/Containers/ContainerAllocationPolicies.h

...
template <int IndexSize>
struct TBitsToSizeType
{
	static_assert(IndexSize, "Unsupported allocator index size.");
};

template <> struct TBitsToSizeType<8>  { using Type = int8; };
template <> struct TBitsToSizeType<16> { using Type = int16; };
template <> struct TBitsToSizeType<32> { using Type = int32; };
template <> struct TBitsToSizeType<64> { using Type = int64; };

/** The indirect allocation policy always allocates the elements indirectly. */
template <int IndexSize>
class TSizedHeapAllocator
{
public:
	using SizeType = typename TBitsToSizeType<IndexSize>::Type;
	...
};

...
template <int IndexSize> class TSizedDefaultAllocator : public TSizedHeapAllocator<IndexSize> { public: typedef TSizedHeapAllocator<IndexSize> Typedef; };
...
```

And you can find the type of `ArrayNum` and `ArrayMax` is `int32` with this flow.

Sometimes, the `UE4.natvis` gives us a hint for understanding the complicated engine code. Even someday you may need to customize `UE4.natvis` for special case while supporting various platforms. It would be also good to learn Natvis if you mostly use Visual Studio IDE. Read premade ones and write your ones. :)