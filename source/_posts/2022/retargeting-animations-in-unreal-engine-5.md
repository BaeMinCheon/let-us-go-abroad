---
title: Retargeting animations in UnrealEngine 5
date: 2022-10-04 22:15:18
tags: [UnrealEngine, Animation, Retargeting]
---

| Environment | |
| --- | --- |
| UnrealEngine | `version: 5.0.3` |
| Windows 11 Pro | `build: 22621.521` |

# `Overview`

It is common that an animation asset is binding at a certain [skeleton asset](https://docs.unrealengine.com/4.27/en-US/AnimatingObjects/SkeletalMeshAnimation/Skeleton/). So you might have experience that you could not utilize the skeleton A's animation at the skeleton B's animation. Because an animation data is made of trace of bones. Therefore, an animation asset would not be compatible when you attempt to apply it to the different skeleton asset.

Think about two skeletons, A and B. The skeleton A has a bone for head, but the skeleton B does not. An animation asset for the skeleton A would not be compatible with the skeleton B, because the skeleton B does not have a bone for head. Though, you might want to apply it, at least parts of animation without head. Fortunately, UnrealEngine provides you to reuse the animation assets by retargeting bones, even if the number of bones or position of bones are different. That is the "Retargeting animations".

# `Preparation`

I will explain you while showing an example. First, install the project [Lyra](https://www.unrealengine.com/marketplace/en-US/product/lyra). You can purchase the project in the UnrealEngine marketplace and install it into your local system. After that, purchase [Animation Starter Pack](https://www.unrealengine.com/marketplace/en-US/product/animation-starter-pack), too. Both of them are free.

{% asset_img 01.png %}

Add `Animation Starter Pack` into the project `Lyra` you installed. Now, open it.

{% asset_img 02.png %}

You can see the folder `AnimStarterPack` at the below of `Content`, and there are several animation assets fit for `SK_Mannequin`.

{% asset_img 03.png %}

Open an animation asset, and you can see the list of animations available in the current skeleton.

{% asset_img 04.png %}

Switch to the tab for skeleton, and you can see the skeleton asset with the hierarchy of bones. Okay, we have checked the asset `Animation Starter Pack`. Jump to the next, the project `Lyra`.

{% asset_img 05.png %}

In the `Lyra`, there are one skeleton asset, but two skeleton meshes; `SKM_Manny` and `SKM_Quinn`. Each of them for male and female appearance.

{% asset_img 06.png %}

The name of skeleton asset is the same with the asset `Animation Starter Pack` with `SK_Mannequin`. From now on, I will name the skeleton for `Lyra` as UE5 skeleton, and the skeleton for `Animation Starter Pack` as UE4 skeleton.

{% asset_img 07.png %}

Also, you can find animation assets for UE5 skeleton at `Content/Characters/Heroes/Mannequin/Animations/Actions`. All we have to do is, retargeting animations from ue4 skeleton into ue5 skeleton, and retargeting animations from ue5 skeleton into ue4 skeleton.

{% asset_img 08.png %}

Create a folder `RetargetedAnimations` at `Content/AnimStarterPack`. We will save the retargeted animations and so on here. First, you should create IK Rigs for each skeleton mesh.

{% asset_img 09.png %}
</br>
{% asset_img 10.png %}
</br>
{% asset_img 11.png %}

Name them as `IKRigUE4` and `IKRigUE5`.

{% asset_img 12.png %}
</br>
{% asset_img 13.png %}

They might look like this. Huh, it is time to setup the IK Rig.

# `Setup IK Rig`

{% asset_img 14.png %}

The IK Rig is used to define many properties, especially for retargeting animations. First of all, you should choose the root of retargeting. It is recommended to choose a `pelvis` in most cases. (Especially, when it is a human form.) Right click the `pelvis` and select the `Set Retarget Root`. Then, the text `(Retarget Root)` is displayed by the `pelvis`. After that, UnrealEngine will retarget the animations from the root, `pelvis`. Do it on both of IK Rig assets.

{% asset_img 15.png %}
</br>
{% asset_img 16.png %}

Get back to the content browser, create a IK Retargeter. You should choose a source IK Rig to create a IK Retargeter. Choose the IKRigUE4, and name this as `UE4_TO_UE5`. Open it up.

{% asset_img 17.png %}

The source IK Rig is the `IKRigUE4`. Therefore, you should assign `IKRigUE5` at the `Target IKRig Asset`.

{% asset_img 18.png %}

Now you can see both of them in the viewport. Try to play an animation from the asset browser.

{% youtube -Xmq0A8bWtA %}

Then, the target one will not be animating properly. Just like the video. As you have set the pelvis as a retarget root, it looks like only the pelvis is synchronized, while others are not. The problem is, the hierarchy of bones is different between two skeletons.

{% asset_img 19.png %}

For instance, the UE4 skeleton has 3 bones for spine; `spine_01`, `spine_02`, and `spine_03`.

{% asset_img 20.png %}

The UE5 skeleton has 5 bones for spine; `spine_01`, `spine_02`, `spine_03`, `spine_04`, and `spine_05`. UnrealEngine cannot retarget animations because the number and position of bones are different between two skeletons. So, you should specify how to match the bones, and you can do it with a chain.

# `Setup Chain`

{% asset_img 21.png %}

The IK Rig asset has a panel `IK Retargeting` beside a panel `Asset Browser`. You can specify some chains here, and it chains a part of bones as a group. UnrealEngine matches the group of same name when you attempt to retarget animations. Let me show you an example. Add new chain, and name it as `leg_left`. We are going to group bones for the left leg.

{% asset_img 22.png %}

Check the bones. After the pelvis, left leg starts at `thigh_l` and ends at `ball_l`. So, set the `Start Bone` and `End Bone` of chain `leg_left`. You have set the chain for left leg in UE4 skeleton. Next, you should set the chain in UE5 skeleton, too.

{% asset_img 23.png %}

Create a chain `leg_left`. Check the bones for left leg. Set the `Start Bone` and `End Bone`. Then, we are good to go.

{% asset_img 24.png %}

Back to the retargeter asset, click the panel `Chain Mapping`. And click the button `Auto-Map Chains`. The `Auto-Map Chains` will match the chains of similar name. You can also match the chains of different name, but you should do it manually in that case.

{% youtube EK2101WQNC0 %}

Try to play some animations. You can notice there is a change. Yes, as you can see in the video, the left leg is synchronized. All you have to do is, create chains and match them.

{% asset_img 25.png %}

I recommend you to create the chains; `leg_right`, `spine`, `arm_left`, `arm_right`, `head`. Here are the chain settings I used for UE4 skeleton.

Chain Name | Start Bone | End Bone
--- | --- | ---
leg_left | thigh_l | ball_l
leg_right | thigh_r | ball_r
spine | spine_01 | spine_03
arm_left | clavicle_l | hand_l
arm_right | clavicle_l | hand_r
head | neck_01 | head

{% asset_img 26.png %}

Unfortunately, UnrealEngine does not support to copy the chain settings yet. So, you should write the same settings in UE5 skeleton.

{% youtube 6UqOYwRcv_4 %}

Back to the retargeter asset, again. Click the button `Auto-Map Chains`, and try to play some animations. It looks like the video. Does it look like perfect ? No, focus on their hands. You shoud care about fingers, too. (Even for toes if the animation covers them 🤣)

{% asset_img 27.png %}

I will show you an example for index finger, rest of fingers are your work.

{% youtube L-m4X-hfExY %}

After the work for all fingers, it should look like this video.

# `Edit Pose`

{% asset_img 28.png %}

Sometimes, you might want to retarget animations but two skeleton are different each other. Suppose you have a skeleton of pose A, and a skeleton of pose T.

{% youtube nQS_ecHXdKw %}

In this situation, the retargeted animations look weird even you set the chains well. Just like the video. Oh...it is like the necromorph in the Dead Space...😱 It happens due to the pose, the two skeletons are different on the pose. You should edit one's pose so that they have the same pose.

{% asset_img 29.png %}

Here, I will edit the skeleton of pose T. Let me edit the pose as A. First, click the button `Edit Pose`.

{% asset_img 30.png %}

We have returned to the base pose. Now you can select bones of the target IK Rig.

{% asset_img 31.png %}

I recommend you to change the property `Target Actor Offset` if you need. You can check the rotation more precisely when it is set by 0.

{% asset_img 32.png %}
</br>
{% asset_img 33.png %}

I have editted the pose by this settings;

- Rotating the upper arm by -60 degrees on the axis Y.
- Rotating the lower arm by +40 degrees on the axis Z.

Do this settings on two arms.

{% asset_img 34.png %}

Now it seems okay. Then, set a proper value to `Target Actor Offset`. Click the button `Edit Pose` to leave the edit mode.

{% youtube poPoFTkDmgk %}

You would see the result just like the video. Quite better than before. But, there is onething you should remember about the feature `Edit Pose`. It is that, you cannot rotate bones not in any chain.

{% asset_img 35.png %}

Suppose an IK Rig asset does not have a chain for right leg. As you can see in the screenshot, there is only a chain for left leg. Go to the retargeter asset.

{% asset_img 36.png %}

You cannot see the section for right leg, even you have entered the edit mode. So, it is crucial that creating necessary chains before you attempt to edit pose in the retargeter asset.

# `Export`

{% asset_img 37.png %}

Get back to the UE4 & UE5 skeletons. You could play animations for UE4 skeleton via the `Asset Browser` in the retargeter asset `UE4_TO_UE5`. Plus, you can export selected animations to create animation assets for UE5 skeleton, which is the target skeleton.

{% asset_img 38.png %}
</br>
{% asset_img 39.png %}

Export animations at `Content/AnimStarterPack/RetargetedAnimations`.

{% asset_img 40.png %}

When you open it up, you can see the asset is using the skeletal mesh for UE5 skeleton. Great. It is simple that retargeting animations in opposite direction; UE5 -> UE4.

{% asset_img 41.png %}

- Create a retargeter asset based on `IKRigUE5`.
- Set the `Target IKRig Asset` as `IKRigUE4`.
- Click the button `Auto-Map Chains`.
- Select animations you want to export.
- Export them, profit !

{% asset_img 42.png %}

You can see it is using the skeletal mesh for UE4 skeleton. 😎