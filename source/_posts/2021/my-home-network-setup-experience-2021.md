---
title: My Home Network Setup Experience (2021)
date: 2021-10-11 20:03:07
tags: [Network, Iptime]
---

# _Prologue_
About 4 months ago, I moved to new house, which is rented for 2 years. The building had been built in 2018, so I expected a quite simple and modern facilities including home networks infra. But, on the day moving to new house, a previous tenant said to me that "Only one LAN port works while others not". At that time, I took this as a misunderstanding of the previous tenant. Because it is common that general people cannot handle or solve an network issue easily. Well...as you can see I write this post, he was right.

The previous tenant has mostly used the internet via wireless network, a.k.a. Wi-fi. It seemed that he does not know computer things. Even he connected to other rooms with exposed LAN cables. (I did not take the picture, but it was similar to a picture below.)

{% asset_img 01.jpg %}

# _Floor Plan_
{% asset_img 02.png %}

I made a floor plan for new house. The green markers mean LAN ports. The LAN port with blue check mark was the only one working properly. Other LAN ports were not. The orange marker means a terminal box. When I first opened the terminal box, it looked like a picture below.

{% asset_img 03.png %}

Very weird. The red cable might be the inbound. But other cables are connected in disorder. In this situation, I cannot guess which cable is destinated to certain LAN port. So I followed steps below for examination.

- ​Check whether the red cable is inbound. The result was yes.
- Connect a inbound cable to each cable. Check where each cable is connected to.
- Repeat the second step until all of unknown ports found.

After some moments, I could organize a mapping for LAN ports. Let us see the picture below.

{% asset_img 04.png %}

# _My Goal_
Now, preparation done. It was time to do my plan. My plan was...

- Reuse my gears as possible. At that time, I had a wireless router and some switch hubs.
- Enable 2 ports. The port #3 and port #4.
- Activate wireless network at appropriate position.

For this purpose, I planned to put a switch hub into the terminal box. Then, the wireless router should be near port #3. Because the Wi-fi signal would get weak when the router is in terminal box. However, on trying this plan, I found a very critical problem.

# _First Try_
There is no power socket in the terminal box. In other words, there is no way to place a switch hub in the terminal box. In general, switch hub consumes power even it is small amount. What a panic !

I searched for bypassing the issue. Fortunately, there is one way fits in my case. [The PoE, Power over Ethernet](https://en.wikipedia.org/wiki/Power_over_Ethernet). The PoE is usually used at certain devices such as CCTV, Network Router, and VoIP Phone. These devices can be installed restricted environments.

- There may be no power socket or power source due to small space.
- There may be only LAN cable due to intra structure.

Yes. That is a perfect feature for this situation.

- I could use only LAN cables in the terminal box.
- I did not want to lay the power socket as the house is rented.

So, I chose to find injector and splitter. They are needed to implement PoE infra. The injector injects signal and power into LAN cable. The splitter splits it into signal and power at destination. Therefore, the floor plan can be redrawn as below.

{% asset_img 05.png %}

Though it looks like some mess, anyway it worked. First, injector provides power to splitter via Ethernet. Therefore, splitter can supply power to switch hub. Second, inbound signal gets distributed by switch hub in terminal box. Finally, home network is constructed by the router on port #3.

- I reused my gears. The wireless router and switch hubs.
- Now I can access internet via port #3 and port #4.
- I activated wireless network at the middle of house, living room.

Great. I was satisfied with the result...for a while.

# _Second Try_
It was totally fine that connecting multiple devices with the router. Of course, because the router is extremely close. But, the problem happened when I had added several devices on port #4 side. When using one device on port #4, the device could recognize the network well. In contrast, when using two devices on port #4, one of them could not recognize the network. It can be drawn as below.

{% asset_img 06.png %}

By the way, the devices were too far from the router. There were two switch hub between the router and devices, and it could prone some network conflicts. I had decided to change my home network configuration, with keeping my goals mentioned before.

{% asset_img 07.png %}

A solution for this problems is simple. Placing a router in the terminal box. Then, my home network would be like picture above. But, one thing left behind, a wireless network. I cannot expect a wireless network functions well if the wireless router is in terminal box because the terminal box must be closed.

{% asset_img 08.png %}

So, I had to buy new router for putting it in the terminal box. Placing new router in the terminal box and leave old router in the same position would be okay. In this case, the old router must be used like switch hub, not a router. My home network looks like picture above. And I will be able to connect the devices on port #4.

# _Third Try_
Hmm...everything works well. Nothing malfunctions. But Wi-fi SSID issue was annoying me. New router and old router had been activated on wireless network, and they got each one of SSID. Yeah, right. There are TWO SSID separately, even though in the same network. I wanted them to merge into one.

Searched again. Maybe the [EasyMesh](https://www.wi-fi.org/knowledge-center/faq/what-is-wi-fi-easymesh) a solution for me. The EasyMesh is one of Wi-fi technology that enables to merge multiple access points. Even EasyMesh does not care about type of frequency of access point. In other words, all of access points with 2.4GHz or 5.0GHz frequency will be merged into single access point. That was what I looked for !

{% asset_img 09.png %}

My routers were made by [EFM networks](https://www.wi-fi.org/about/member-companies/efm-networks), the company famous of ipTIME trademark. EFM networks provides an utility program controls EasyMesh configuration like picture above. (Almost every company seems that develops and implements the EasyMesh specification, so find out other companies too.)

Setting up EasyMesh was completely easy. The structure of my home network was fit for the conditions. Now I can access Wi-fi via only single SSID. Furthermore, Wi-fi range is larger than before by merging two SSID. What a convenient :)

# _Epilogue_
With the series of effort, I could setup my home network completely without great expense. It was lucky that there was no need to call workers. Maybe I would tell a next tenant of these story and give advice. I do not want anyone to suffer from the problems. ;)

Oh, I almost forgot. You should check the router before buying it if you want to setup EasyMesh. EasyMesh requires two types of router; MeshController and MeshAgent. Check the item you gonna buy whether it is kind of MeshController or MeshAgent.