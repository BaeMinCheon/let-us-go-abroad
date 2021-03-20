---
title: Shared folder in WSL
date: 2019-08-19 00:57:29
tags: [WSL]
---

- this post covers
    - how to share a folder in wsl
- environment
    - Windows 10 / Home
    - WSL / Ubuntu 18.04 LTS
    - Visual Studio Code

---

### pre-task
- install WSL in your Windows 10
    - open the `Microsoft Store`
    - search for `Ubuntu`
    - click `Ubuntu 18.04 LTS`
    - get and install it
- prepare the environment
    - initialize your wsl

### make-link
{% asset_img 01.png %}
- make a folder in some directory on Windows
    - ex) `D:\Code\LinuxShare`

{% asset_img 02.png %}
- make a link in some directory on WSL
    - format) `ln -s /mnt/[partition]/[folder...] [link-name]`
    - ex) `ln -s /mnt/d/Code/LinuxShare share-with-windows`

### make-file
{% asset_img 03.png %}
- make a text file on Windows side
    - ex) `D:\Code\LinuxShare\test.txt`

{% asset_img 04.png %}
- check out the text file on WSL side
    - ex) `~/share-with-windows/test.txt`

{% asset_img 05.png %}
{% asset_img 06.png %}
- edit the text file and save it on WSL side
    - ex) `hello WSL !` → `hello Windows !`

{% asset_img 07.png %}
- check out the text file on Windows side

### typing-and-compiling
{% asset_img 08.png %}
{% asset_img 09.png %}
- make a c file and edit it on Windows side
    - ex) `D:\Code\LinuxShare\test.c`

{% asset_img 10.png %}
- compile the c file and execute it
    - ex) `~/share-with-windows/test.c`
- you can now type texts on Windows and compile that on WSL

{% asset_img 11.png %}
- also output file, `a.out` will be seen on Windows side