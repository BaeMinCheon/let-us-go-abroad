---
title: unreal-input-system-via-gamepad
date: 2020-10-25 18:11:09
tags: [UnrealEngine, Input]
---

- this post covers
    - key mapping of Xbox One controller in UnrealEngine
    - how UnrealEngine handles input from user
    - input process for gamepad (with Xbox One controller)
- environment
    - Unreal Engine 4 `ver. 4.25`
    - Visual Studio 2019 `ver. 16.7.6`
    - Windows 10 `ver. 2004`
- reference
    1. [https://support.xbox.com/en-US/help/hardware-network/controller/xbox-one-wireless-controller][reference #1]

[reference #1]: https://support.xbox.com/en-US/help/hardware-network/controller/xbox-one-wireless-controller

---

# _Terms_

Term | Synonym | Meaning
- | - | -
Non-Axis | Key, Button | input type which has only two states: `Pressed` or `Released`
Axis | Stick | input type which has numberless states
Key |  | (sometimes) it is used for indicating any input type. be careful with context for distinction of `Non-Axis`
Deadzone | Threshold | range of input values, which blocks values in range. only bigger value cannot be blocked

# _Gamepad Non-Axis/Axis Mapping in UnrealEngine_

```cpp
UnrealEngine/Engine/Source/Runtime/Core/Private/GenericPlatform/GenericApplication.cpp

...
const FGamepadKeyNames::Type FGamepadKeyNames::LeftAnalogX("Gamepad_LeftX");
const FGamepadKeyNames::Type FGamepadKeyNames::LeftAnalogY("Gamepad_LeftY");
const FGamepadKeyNames::Type FGamepadKeyNames::RightAnalogX("Gamepad_RightX");
const FGamepadKeyNames::Type FGamepadKeyNames::RightAnalogY("Gamepad_RightY");
const FGamepadKeyNames::Type FGamepadKeyNames::LeftTriggerAnalog("Gamepad_LeftTriggerAxis");
const FGamepadKeyNames::Type FGamepadKeyNames::RightTriggerAnalog("Gamepad_RightTriggerAxis");
...

```

we can find some pre-defined non-axis/axis keys as `FName` in `GenericApplication.cpp`. they are for mapping from various input messages to generic input messages. "various input messages" means, there are many types of gamepad in the world. the button/stick layout differs in Xbox One controller, Playstation 4 controller and so on. the gamepads below are Xbox One, Playstation 4, Stadia and Switch in order.

{% asset_img 01.jpg %}
{% asset_img 02.jpg %}
{% asset_img 03.png %}
{% asset_img 04.jpg %}

look at the Xbox One one and Playstation 4 one. they have many differences such as position of stick and exsitance of touch pad. even in comparison for Xbox One one and Switch one, the number of buttons differs. in this situation, it is not easy for every individual developer to support every type of gamepad, so the need of generic mapping for gamepad input arises. let us find out the generic mapping with Xbox One controller examples.

{% asset_img 06.png %}
{% asset_img 07.png %}

the tables are from the [reference #1]. you can find more details for each button/stick at the URL. though there are so many items in table, some of them are not counted as user input in common situation. so, we can gotta consider the items below:

Index | Item Name | Unreal Mapping | Input Type
- | - | - | -
1 | Left Stick | (Move Horizontally) Gamepad_LeftX | Key, Axis
 |  | (Move Vertically) Gamepad_LeftY | Key, Axis
 |  | (Move Left Side More Than Deadzone) Gamepad_LeftStick_Left | Key
 |  | (Move Up Side More Than Deadzone) Gamepad_LeftStick_Up | Key
 |  | (Move Right Side More Than Deadzone) Gamepad_LeftStick_Up | Key
 |  | (Move Down Side More Than Deadzone) Gamepad_LeftStick_Down | Key
 |  | (Click) Gamepad_LeftThumbstick | Key
2 | Left Bumper | Gamepad_LeftShoulder | Key
3 | View Button | Gamepad_Special_Left | Key
6 | Menu Button | Gamepad_Special_Right | Key
7 | Right Bumper | Gamepad_RightShoulder | Key
8 | Directional Pad | (Left) Gamepad_DPad_Left | Key
 |  | (Up) Gamepad_DPad_Up | Key
 |  | (Right) Gamepad_DPad_Right | Key
 |  | (Down) Gamepad_DPad_Down | Key
10 | Right Stick | (Move Horizontally) Gamepad_RightX | Key, Axis
 |  | (Move Vertically) Gamepad_RightY | Key, Axis
 |  | (Move Left Side More Than Deadzone) Gamepad_RightStick_Left | Key
 |  | (Move Up Side More Than Deadzone) Gamepad_RightStick_Up | Key
 |  | (Move Right Side More Than Deadzone) Gamepad_RightStick_Up | Key
 |  | (Move Down Side More Than Deadzone) Gamepad_RightStick_Down | Key
 |  | (Click) Gamepad_RightThumbstick | Key
11 | Right Trigger | Gamepad_RightTriggerAxis | Key, Axis
 |  | (Press More Than Deadzone) Gamepad_RightTrigger | Key
14 | Left Trigger | Gamepad_LeftTriggerAxis | Key, Axis
 |  | (Press More Than Deadzone) Gamepad_LeftTrigger | Key
X | X Button | Gamepad_FaceButton_Left | Key
Y | Y Button | Gamepad_FaceButton_Up | Key
A | A Button | Gamepad_FaceButton_Bottom | Key
B | B Button | Gamepad_FaceButton_Right | Key

some of them are handled as not only Key but Axis, too.

{% asset_img 08.png %}
</br>
{% asset_img 09.png %}

- the `Gamepad_LeftY` is the one of cases

# _Gamepad Non-Axis Input Handling Process_

{% asset_img 10.png %}

focus the function `XInputInterface::SendControllerEvents()`. there is the logic to filter hardware input state.

```cpp
UnrealEngine/Engine/Source/Runtime/ApplicationCore/Private/Windows/XInputInterface.cpp

...

void XInputInterface::SendControllerEvents()
{
    ...

    CurrentStates[X360ToXboxControllerMapping[0]] = !!(XInputState.Gamepad.wButtons & XINPUT_GAMEPAD_A);
    CurrentStates[X360ToXboxControllerMapping[1]] = !!(XInputState.Gamepad.wButtons & XINPUT_GAMEPAD_B);
    CurrentStates[X360ToXboxControllerMapping[2]] = !!(XInputState.Gamepad.wButtons & XINPUT_GAMEPAD_X);
    CurrentStates[X360ToXboxControllerMapping[3]] = !!(XInputState.Gamepad.wButtons & XINPUT_GAMEPAD_Y);
    CurrentStates[X360ToXboxControllerMapping[4]] = !!(XInputState.Gamepad.wButtons & XINPUT_GAMEPAD_LEFT_SHOULDER);
    CurrentStates[X360ToXboxControllerMapping[5]] = !!(XInputState.Gamepad.wButtons & XINPUT_GAMEPAD_RIGHT_SHOULDER);
    CurrentStates[X360ToXboxControllerMapping[6]] = !!(XInputState.Gamepad.wButtons & XINPUT_GAMEPAD_BACK);
    CurrentStates[X360ToXboxControllerMapping[7]] = !!(XInputState.Gamepad.wButtons & XINPUT_GAMEPAD_START);
    CurrentStates[X360ToXboxControllerMapping[8]] = !!(XInputState.Gamepad.wButtons & XINPUT_GAMEPAD_LEFT_THUMB);
    CurrentStates[X360ToXboxControllerMapping[9]] = !!(XInputState.Gamepad.wButtons & XINPUT_GAMEPAD_RIGHT_THUMB);
    CurrentStates[X360ToXboxControllerMapping[10]] = !!(XInputState.Gamepad.bLeftTrigger > XINPUT_GAMEPAD_TRIGGER_THRESHOLD);
    CurrentStates[X360ToXboxControllerMapping[11]] = !!(XInputState.Gamepad.bRightTrigger > XINPUT_GAMEPAD_TRIGGER_THRESHOLD);
    CurrentStates[X360ToXboxControllerMapping[12]] = !!(XInputState.Gamepad.wButtons & XINPUT_GAMEPAD_DPAD_UP);
    CurrentStates[X360ToXboxControllerMapping[13]] = !!(XInputState.Gamepad.wButtons & XINPUT_GAMEPAD_DPAD_DOWN);
    CurrentStates[X360ToXboxControllerMapping[14]] = !!(XInputState.Gamepad.wButtons & XINPUT_GAMEPAD_DPAD_LEFT);
    CurrentStates[X360ToXboxControllerMapping[15]] = !!(XInputState.Gamepad.wButtons & XINPUT_GAMEPAD_DPAD_RIGHT);
    CurrentStates[X360ToXboxControllerMapping[16]] = !!(XInputState.Gamepad.sThumbLY > XINPUT_GAMEPAD_LEFT_THUMB_DEADZONE);
    CurrentStates[X360ToXboxControllerMapping[17]] = !!(XInputState.Gamepad.sThumbLY < -XINPUT_GAMEPAD_LEFT_THUMB_DEADZONE);
    CurrentStates[X360ToXboxControllerMapping[18]] = !!(XInputState.Gamepad.sThumbLX < -XINPUT_GAMEPAD_LEFT_THUMB_DEADZONE);
    CurrentStates[X360ToXboxControllerMapping[19]] = !!(XInputState.Gamepad.sThumbLX > XINPUT_GAMEPAD_LEFT_THUMB_DEADZONE);
    CurrentStates[X360ToXboxControllerMapping[20]] = !!(XInputState.Gamepad.sThumbRY > XINPUT_GAMEPAD_RIGHT_THUMB_DEADZONE);
    CurrentStates[X360ToXboxControllerMapping[21]] = !!(XInputState.Gamepad.sThumbRY < -XINPUT_GAMEPAD_RIGHT_THUMB_DEADZONE);
    CurrentStates[X360ToXboxControllerMapping[22]] = !!(XInputState.Gamepad.sThumbRX < -XINPUT_GAMEPAD_RIGHT_THUMB_DEADZONE);
    CurrentStates[X360ToXboxControllerMapping[23]] = !!(XInputState.Gamepad.sThumbRX > XINPUT_GAMEPAD_RIGHT_THUMB_DEADZONE);

    ...
}

...

```

the next screenshot shows the value of `CurrentStates` when you press A button in Xbox One gamepad.

{% asset_img 11.png %}

we can see the key, deadzone or threshold macro in the code. the macros are defined at `XInput.h`

```cpp
C:/Program Files (x86)/Windows Kits/10/Include/10.0.18362.0/um/XInput.h

...

//
// Constants for gamepad buttons
//
#define XINPUT_GAMEPAD_DPAD_UP          0x0001
#define XINPUT_GAMEPAD_DPAD_DOWN        0x0002
#define XINPUT_GAMEPAD_DPAD_LEFT        0x0004
#define XINPUT_GAMEPAD_DPAD_RIGHT       0x0008
#define XINPUT_GAMEPAD_START            0x0010
#define XINPUT_GAMEPAD_BACK             0x0020
#define XINPUT_GAMEPAD_LEFT_THUMB       0x0040
#define XINPUT_GAMEPAD_RIGHT_THUMB      0x0080
#define XINPUT_GAMEPAD_LEFT_SHOULDER    0x0100
#define XINPUT_GAMEPAD_RIGHT_SHOULDER   0x0200
#define XINPUT_GAMEPAD_A                0x1000
#define XINPUT_GAMEPAD_B                0x2000
#define XINPUT_GAMEPAD_X                0x4000
#define XINPUT_GAMEPAD_Y                0x8000

//
// Gamepad thresholds
//
#define XINPUT_GAMEPAD_LEFT_THUMB_DEADZONE  7849
#define XINPUT_GAMEPAD_RIGHT_THUMB_DEADZONE 8689
#define XINPUT_GAMEPAD_TRIGGER_THRESHOLD    30

...
```

there is an exception case for input smaller than deadzone. let us take an example with `Gamepad_LeftX` message. check out screenshots below.

{% asset_img 12.png %}

when you input a tiny change on left stick, `InputAxis()` is called. and the key will be accumulated in `EventAccumulator`.

```cpp
UnrealEngine/Engine/Source/Runtime/Engine/Private/UserInterface/PlayerInput.cpp

...

void UPlayerInput::ProcessInputStack(...)
{
    ...

    Exchange(KeyState->EventCounts[EventIndex], KeyState->EventAccumulator[EventIndex]);

    ...
}

...
```

after that, the key in `EventAccumulator` is moved to `EventCounts`.

{% asset_img 13.png %}

if you keep operating the stick, the key is regarded as down.

{% asset_img 14.png %}

in this situation, the key is regarded as released if all keys are flushed.

{% asset_img 15.png %}

# _Gamepad Axis Input Handling Process_

the callstack of Axis one is similar to the Non-Axis one. there is a difference of execution at `XInputInterface::SendControllerEvents()`

```cpp
UnrealEngine/Engine/Source/Runtime/ApplicationCore/Private/Windows/XInputInterface.cpp

...

void XInputInterface::SendControllerEvents()
{
    ...

    // Send new analog data if it's different or outside the platform deadzone.
    auto OnControllerAnalog = [this, &ControllerState](const FName& GamePadKey, const auto NewAxisValue, const float NewAxisValueNormalized, auto& OldAxisValue, const auto DeadZone) {
        if (OldAxisValue != NewAxisValue || FMath::Abs((int32)NewAxisValue) > DeadZone)
        {
            MessageHandler->OnControllerAnalog(GamePadKey, ControllerState.ControllerId, NewAxisValueNormalized);
        }
        OldAxisValue = NewAxisValue;
    };

    const auto& Gamepad = XInputState.Gamepad;

    OnControllerAnalog(FGamepadKeyNames::LeftAnalogX, Gamepad.sThumbLX, ShortToNormalizedFloat(Gamepad.sThumbLX), ControllerState.LeftXAnalog, XINPUT_GAMEPAD_LEFT_THUMB_DEADZONE);
    OnControllerAnalog(FGamepadKeyNames::LeftAnalogY, Gamepad.sThumbLY, ShortToNormalizedFloat(Gamepad.sThumbLY), ControllerState.LeftYAnalog, XINPUT_GAMEPAD_LEFT_THUMB_DEADZONE);

    OnControllerAnalog(FGamepadKeyNames::RightAnalogX, Gamepad.sThumbRX, ShortToNormalizedFloat(Gamepad.sThumbRX), ControllerState.RightXAnalog, XINPUT_GAMEPAD_RIGHT_THUMB_DEADZONE);
    OnControllerAnalog(FGamepadKeyNames::RightAnalogY, Gamepad.sThumbRY, ShortToNormalizedFloat(Gamepad.sThumbRY), ControllerState.RightYAnalog, XINPUT_GAMEPAD_RIGHT_THUMB_DEADZONE);

    OnControllerAnalog(FGamepadKeyNames::LeftTriggerAnalog, Gamepad.bLeftTrigger, Gamepad.bLeftTrigger / 255.f, ControllerState.LeftTriggerAnalog, XINPUT_GAMEPAD_TRIGGER_THRESHOLD);
    OnControllerAnalog(FGamepadKeyNames::RightTriggerAnalog, Gamepad.bRightTrigger, Gamepad.bRightTrigger / 255.f, ControllerState.RightTriggerAnalog, XINPUT_GAMEPAD_TRIGGER_THRESHOLD);

    ...
}

...

```

in this case, `OnControllerAnalog()` is called even a tiny change of input value exists. because the code compares with `OldAxisValue != NewAxisValue`. the function will not be called only when there is no change on input value.