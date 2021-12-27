---
title: Retrospection 2021
date: 2021-12-18 21:54:09
tags: [Retrospection]
---

# _`Two years from beginning a blog`_
What a monumental, it has been 2 years ago when I posted the first article on this blog; [The article about UnrealEngine build target.](https://baemincheon.github.io/2019/08/06/what-is-unreal-build-target/) I have usually written a post each month, mostly about UnrealEngine. The topic of post is chosen by my interest. For instance, things that I want to know, what I want to check, or just a record for memo.

An example fits in first rule is [the post about UnrealEngine macro generation](https://baemincheon.github.io/2021/08/06/how-unreal-macro-generated/). At that time, I was curious how UnrealEngine uses a macro for implementing its framework. Another one for second rule is [the post about growth of `std::vector` in cpp](https://baemincheon.github.io/2021/05/16/cpp-std-vector-growth/). At that time, I already knew there is difference between GCC and MSVC on growth size of `std::vector`. But, I had also wanted to check where the difference comes from. Last one for third rule is [the post about how to setup Perforce server](https://baemincheon.github.io/2021/09/10/perforce-server-setup-with-digital-ocean/). Starting a small game project, I had to setup Perforce server for my team. However, finding an easy and good manual for this was so hard that I decided to post the process of setup Perforce server. For myself and anyone.

Plus, almost every resource about UnrealEngine in Youtube or Google is targeting for the blueprint user, not the cpp user. So it was so hard to use a feature in cpp source code with the resources. Even they usually do not explain how the code works, just explain about how to use it. In this aspect, I am trying to explain the source code of engine for making it transparent. Furthermore, I am trying to put an example on every explanation. Because I have been tired of resources explained by only some words; Beginners cannot understand them, even follow the process without a precise description. Writing as simple as possible, demonstrating as many as possible, these policies would be maintained as long as possible.

I collect data via Google Analytics plugged in this blog. Top 10 pages most visited (until now) are below:

1. `/2020/02/09/unreal-widget-coordinate-system/`
2. `/2021/04/07/difference-between-build-cs-and-target-cs/`
3. `/2020/10/25/unreal-input-system-via-gamepad/`
4. `/2020/03/14/unreal-unique-pointer/`
5. `/2019/08/06/what-is-unreal-build-target/`
6. `/2019/08/11/custom-unreal-engine-build/`
7. `/2019/08/16/differences-of-unreal-build-targets/`
8. `/2020/01/28/unreal-blueprint-practical-use/`
9. `/2021/03/01/unreal-engine-natvis/`
10. `/2020/01/17/unreal-fname-anatomy/`

It seems that a post about widget, input, or build is more popular than others. For helping UnrealEngine newbies, I would better consider writing a post about those topics in next year. Looking back my experiences on studying UnrealEngine, I was also struggling for those topics.

# _`Career with UnrealEngine`_
On working at PUBG Studio for 2 years, I have learned many things. (Even though some of them are not my part) Managing infra structure for large scale project, working remote in efficient way, developing and testing on console platforms, and so on. But most of all, various experiences of developing with UnrealEngine. Thanks to talented coworkers' help, I could have done my tasks.

Reading and understanding the engine code is essential to development of industry level, but my skill was not sufficient to do that. So, in first year, I was busy to learn about topics mentioned in conversations. It was enough to take up my time. Repeated working at office and studying at home. After that, in second year, I became to understand most of what coworkers said. I started to read more engine code deeper and deeper. It is the time when I could say that "I know UnrealEngine...a bit ?". Now, entering third year, I have plenty of skills enough to advice coworkers. Especially, many of coworkers ask me as I have experiences of console platforms. (But I still think I should learn more about console platforms)

I have done main tasks such as optimizing contents, fixing bugs and crashes, and making development environment better. Most of all, I want to say about only the first thing. Optimizing contents was the most challenging task. You may know the common sense in program optimization;

> When you increase a speed, an available memory would be decreased. On the other hand, when you increase an available memory, a speed would be decreased.

Due to the structure of memory in computer, loading data from auxiliary storage is quite expensive operation. (In comparison to main storage) Anyone who wants the program to run faster would sacrifice the memory of program for loading data as much as possible. Particularly, among the programs, the size of data in game is bigger than others. That is why games try to load data before it is needed. Because any gamer does not want to see the freezed screen whenever picking up an item.

Right...in this condition, optimizing contents should not be easy one. What more worse is, new contents come in the game every update. In other words, an available memory could be decreased as the total size of data increased. So, it is hard to maintain a speed of program without sacrificing an available memory. Even more and more users are using the SSD thesedays, we have to support the old-gen consoles such as XboxOne and Playstation4, which are the devices uses HDD in default. 

Additionally, loading data asynchronously is not always a best solution. Because, eventually it also consumes the resource of computer, and results a hitch or freeze. Yeap, so we struggled to manage an available memory of our game almost every update. Mainly, we have rarely chosen to skip some data loaded before entering ingame, instead we have chosen to optimize assets or codes to reduce its size so that the total size of data get reduced. Profiling our game and finding assets or codes taking up significant times, and analyzing what the assets or codes are wrong, and optimizing them...with magical spell. (I mean, the way how to optimize contents varies every time so it is like MAGIC, which is not easy to explain)

What I want to do in future would be developing games with newest features of UnrealEngine. But it would be hard without engine migration as we are using 4.16 version of UnrealEngine. Additionally, it is also hard to use the most fascinating features; Nanite and Lumen, as we are supporting old-gen consoles. Maybe I keep going my tasks as before, but at least, gonna to learn the newest features of UnrealEngine. Maybe, sometime I can use them. Or, I would post with contents of them. ;)

# _`Move to new house`_
In the middle of 2021, I had moved to new house as mentioned at [this post](https://baemincheon.github.io/2021/10/11/my-home-network-setup-experience-2021/). New house is bigger than old house, and I had to buy some furnitures. (Basic furnitures such as bed, purifier, and so on are provided in old house) Therefore, I had searched several items including closet, purifier, bed, desk, and so on. Especially, I had a difficulty for buying devices related to home network. Because I never had a need to buy these tools or devices. Even when I was in Army as Signal Corps, those things were supplies. (I did not choose what to buy)

Now I have a living room. Trying to put a sofa, table, or something fits in living room, I decided to buy UHD TV. No, not only UHD resolution...also 120Hz framerate should be great. That is a maximum range of next-gen console's signal output. Yeap, I bought a next-gen console, too. Xbox Series X (XSX). Finally it looks like below:

{% asset_img 01.jpg %}

The TV is the product of SAMSUNG, [50QN90A](https://www.samsung.com/in/tvs/qled-tv/qn90a-50-inch-neo-qled-4k-smart-tv-qa50qn90aaklxl/). It supports resolution and framerate upto UHD/120Hz, and has 50 inches display. I am satisfied with this product in overall aspects, what a nice TV. Only one thing disappointed is, the OS of TV is TIZEN, which is SAMSUNG's proprietary OS. This OS does not have plenty of applications yet. For example, if you want to watch Twitch ? There is no twitch application in TIZEN, so you have to watch Twitch via web browser. Very inconvenient. Or you should check [this](https://github.com/fgl27/smarttv-twitch#installation-guide-tizen-studio-tested-on-linux-ubuntu-16-and-up-and-windows-10-64-bits).

After living at new house for about 6 months, I think it was a good choice. More space gets me more comfortable, and makes it possible to stay home more than before. Still we are living with the various, so I wanted to stay home without some reason. Besides, I am also working from home, so more space is suitable for me. :)

# _`Thought about next-gen console`_
I cannot help saying this, I bought the next-gen console to play Battlefield 2042. Yeap...sadly, the game has totally been ruined. Huh...I might write some post for that sometime. As a fan of Battlefield series, it is so sad that people compare Cyberpunk 2077 and Battlefield 2042. Even worse, it is not easy to find any point that Battlefield 2077 is better than Cyberpunk 2077. Crap. Anyway, that is why I decided to buy a next-gen console.

Though a start of experience was not good, the console itself was amazing. Recently, UnrealEngine has released [The Matrix Awakens: An Unreal Engine 5 Experience](https://www.unrealengine.com/en-US/wakeup?sessionInvalidated=true), which is kind of tech demo that shows what UnrealEngine 5 can do. The game runs at UHD/30Hz with DynamicResolution on XSX, but it looks like REAL footage. At least, it seems this tech would become popular on movie industry. Already many of short films, part of films, or commercials use UnrealEngine, but currently they are using UnrealEngine for capturing a video. In other words, they are not realtime rendering. In the view, the tech demo was so impressed. "Wow, is it possible in realtime rendering ?"

Absolutely, this is not possible only with improvement of processor performance. In comparison of playing video, playing game needs more various resources not only processor resource. Playing video only needs loading next frame and next frame...until the video ends. (Displaying is none of business here) But, playing game needs loading assets should be displayed, executing codes may update the world or actors, integrating everything into a projection...in SINGLE ONE frametime.

CPU takes charge of executing codes, and GPU takes charge of integrating into a projection. Then, who takes charge of loading assets ? In general, CPU and memory works for it. CPU tries to read data from memory and blocks itself until the read done, but it takes some time if memory bandwidth is low. Low bandwidth means that you are try to load data from auxiliary storage, and it must not be always faster than loading data from main storage (ex: RAM). Even the problem gets worse when the size of data bigger and bigger. Thus, people have tried several techniques; A compression skill is sometimes applied to minimize the size of data. Just loading a compressed data and decompress it into the original data. Of course, this skill needs an extra resource from CPU, which could trigger a hitch while playing a game. Next, a [asset duplication](https://docs.unrealengine.com/4.26/en-US/BlueprintAPI/EditorScripting/Asset/DuplicateAsset/) skill is common one in games supporting for device using HDD. The skill is for reducing the seek time of HDD, because the seek time of HDD is too slow to load data in realtime. This skill also has a drawback that the size of game package gonna be VERY large. Horrible.

Thankfully, both Microsoft and Sony seem to have recognized these kind of issues. Specially, Microsoft. They made an exclusive processor for decompressing data to minimize any side effect of data compression skill. Like Sony did, Microsoft customized their NVMe SSD. Furthermore, Microsoft gave game developers more flexibity atm manpulating storage by providing DirectStorage API. Finally, Sampler Feedback Streaming (SFS) could be built on these basement skills. SFS will optimize game's memory usage and file I/O operations. Then, Microsoft names all the skills as an architecture [Xbox Velocity Architecture](https://news.xbox.com/en-us/2020/07/14/a-closer-look-at-xbox-velocity-architecture/). Previously, I did not believe their words because it looks like hype. But I do know that is an amazing advance in game development. Turning back to UnrealEngine, EpicGames did great job so far with the new features.

# _`Plans on next year`_
Except for writing a blog post each month, I have considered some plans to do.

> Plan #1. Making a game.

Recently, I started to develop a game. About 3 months ago, my ex-coworker had suggested an Action RPG game project, and I accepted that. I do not know when the game is released, but some milestones should be completed in next year. In 2022, there may be a playable build. Developing this game, I should develop many features or design critical systems. With these kind of experiences, I retrospect when I was a student. It is the same that struggling on every issue, but the result would be better than when I was a student.

> Plan #2. Starting to develop a game engine.

Working as a game programmer for 2 years, I have thought that "How about making my own game engine ?". It would be fun that developing my own game engine, proud if many games are made of it, satisfied that I could do this. This kind of thought would be similar many of programmers have a dream that making their own programming language, operating system, database system, or etc. The game engine I will develop would be lightweight, but Rust gonna be used. I want to use Rust as many as possible in developing it. Because managing the side effect of using CPP would not be easy in a small project.

> Plan #3. Publishing console application or game.

Actually, I have already subscribed the Xbox developer plan. It was cheaper than I expected, $19 USD only one time. (Maybe I would work for Playstation sometime...but I want to focus on Xbox now) I want to utilize this Xbox developer license. How or what to make ? Let us think about. Case first, Playing games in Xbox, I will note them if I experience some inconveniences. It would be great project if there would be something to make users convenient. Second case, making a tech demo just like The Matrix Awakens would be fun. It could be a kind of inspiration to some developers. Well...I may not make any application or game if I am busy for other works. It is on the back burner. XD