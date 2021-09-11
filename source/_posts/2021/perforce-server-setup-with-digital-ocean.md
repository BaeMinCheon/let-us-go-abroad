
---
title: How to setup Perforce server with Ubuntu
date: 2021-09-10 23:50:39
tags: [Perforce, DigitalOcean, Ubuntu]
---

| Environment | |
| --- | --- |
| Helix Core (P4D) | `version: P4D/LINUX26X86_64/2021.1/2156517` |
| Ubuntu | `version: 20.04 LTS` |
| Helix Visual Client (P4V) | `version: P4V/NTX64/2021.3/2170446` |
| Windows 10 | `build: 19043.1165` |

| References | |
| --- | --- |
| Helix Core Server Administrator Guide | https://www.perforce.com/manuals/p4sag/Content/P4SAG/chapter.install.html |
| .p4ignore for UnrealEngine | https://github.com/mattmarcin/ue4-perforce/blob/master/.p4ignore |
| P4 Typemap for UnrealEngine | https://docs.unrealengine.com/4.27/en-US/ProductionPipelines/SourceControl/Perforce/ |

# _Prepare Ubuntu Instance_
As the Perforce is a kind of CVCS*, it is recommended to use a dedicated server. The dedicated server should run all day and night, and should have a fixed IP address. Thus, you would better choose a cloud server if not have a machine for the purposes. In this post, I chose [DigitalOcean](https://www.digitalocean.com/) for a cloud server provider. The DigitalOcean provides instances with cheaper cost than others such as AWS. You know, you may not need a high quality instance for your small size project.
※ CVCS : Centralized Version Control System. Find more information at [here](https://en.wikipedia.org/wiki/Distributed_version_control#Distributed_vs._centralized).

Sign-up and Sign-in the DigitalOcean. Click the button `Create` and choose `Droplets`.
{% asset_img 01.png %}
At the page `Create Droplets`, you would be asked to choose options for a instance. In this post, we gonna choose options like below:

- `Choose an image`
    - Ubuntu 20.04 (LTS) x64
- `Choose a plan`
    - SHARED CPU
        - Basic
    - CPU options
        - Regular Intel with SSD
        - $5/month (= $0.007/hour)
            - 1 core CPU, 1 GB RAM, 25 GB SSD, 1000 GB transfer
- `Add block storage`
    - $5/month (= $0.007/hour)
        - 50 GB SSD
- `Choose a datacenter region`
    - (Select a datacenter that is closest to your location)
    - (In my case, it is Singapore)
- `Select additional options`
    - Monitoring
- `Authentication`
    - Password
        - (Choose a password for entering the instance)

Any options I did not mention are left as default selection. Let us create our Droplet by clicking the button `Create Droplet`.
{% asset_img 02.png %}
Now you can see the new Droplet. Connect the instance via SSH. You can do it with Powershell or WSL in Windows 10. The `X.X.X.X` must be replaced with the IP address of instance.
```
> ssh root@X.X.X.X
```
But, when you attempt to connect the instance via SSH, you are asked to enter a password.
```
root@X.X.X.X's password:
```
Enter the password that you typed at the `Authentication` text block. If the right password entered, you can see the logs like below:
```
Welcome to Ubuntu 20.04.2 LTS (GNU/Linux 5.4.0-73-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Sat Sep 11 16:12:19 UTC 2021

  System load:  0.0               Users logged in:       0
  Usage of /:   9.3% of 24.06GB   IPv4 address for eth0: X.X.X.X
  Memory usage: 24%               IPv4 address for eth0: X.X.X.X
  Swap usage:   0%                IPv4 address for eth1: X.X.X.X
  Processes:    113

66 updates can be applied immediately.
1 of these updates is a standard security update.
To see these additional updates run: apt list --upgradable


*** System restart required ***
Last login: Sat Sep 11 11:35:09 2021 from X.X.X.X
root@ubuntu-test:~#
```
At the your instance page, you can check the volume setting. Click the `Config Instructions`.
{% asset_img 03.png %}
{% asset_img 04.png %}
If you selected the option `Automatically Format & Mount` at the section `Add block storage`, the volume is already attached even you did nothing. In other words, the process `Mount the volume` is already done. Let us check whether the volume is well mounted. The `volume_X` must be replaced with yours.
```
> cd /mnt/volume_X
> ls
lost+found
```

You successfully setup an Ubuntu instance. Good to go !

# _Install & Setup P4D_
We need to setup public key for accessing Perforce packages. For this, you need to download the public key at https://package.perforce.com/perforce.pubkey. The download can be done by command below:
```
> curl https://package.perforce.com/perforce.pubkey > perforce.pubkey
```
This command let you save the public key as a file, whose name is `perforce.pubkey`.
```
> curl https://package.perforce.com/perforce.pubkey > perforce.pubkey
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  1707  100  1707    0     0   2024      0 --:--:-- --:--:-- --:--:--  2022
> ls
perforce.pubkey  snap
```
You can check the contents of file with `cat`.
```
> cat perforce.pubkey
-----BEGIN PGP PUBLIC KEY BLOCK-----
Version: GnuPG v2.0.14 (GNU/Linux)

mQINBFIOq14BEADUC4gm+gjS/E/y2xXouALvMuK2xO/8nbcXJCUAD6Bi1xxmyDaR
LXHDJ5lIzZV8/Jctck2bBIWE1WE8Qfpfz/eAU5lJoQTovt0OkOnyyAyFBSk9yXtN
fscQGdTXkl9LVVfsaTHVT3WGZF+iMCIZOVjYGjqRh3ozZp3LWOQl3cwgOZKQCi9Q
y/YRn6XZIOiQQEfvLzrBL1oyD1BoOq8Y2CrwTfhyz93qIRu089mAr7lo2e6UM/KV
JRjk6rPFcKIE0aOP2UwgY/6LMeK65MAKib76EFbygXBprz9K5zwq70A7MGSPjRPw
A7kdzw53flZyNscI2c093jW/PkeDw4++01QFky/FFqJncjIHoOid42NvQXD/+E5e
JKhqYReS2eHpv5qgsSc2Febd5Ccd0B4+2ryY3MBXqaj759NH6uWAowHjAv90y4Cb
c2FugNBAJ6XQBQaXcWsfPnWpBFYL36LxBCcu+ddiycTWS9SWFT3h0FIgTbTQNNQr
Fgjg5vYAw+TWU8wf1I3sak+wbU25h7ErKN1oSJ/EbPwUFOc6zjaDUlnIgmCnQgEj
NdrlWGGfgCfYTHZTnGW6fGlpByDnYO0wn/okPJxRStnkbqb8QGKRa1uTObSM0/4J
aqsaReo2E45x2TAIY6rNuiLet/r1hZzpLs3dffvoddscb/LshW1eiNU36QARAQAB
tERQZXJmb3JjZSBTb2Z0d2FyZSAoUGFja2FnZSBTaWduaW5nKSA8c3VwcG9ydCtw
YWNrYWdpbmdAcGVyZm9yY2UuY29tPokCPgQTAQIAKAUCUg6rXgIbAwUJEswDAAYL
CQgHAwIGFQgCCQoLBBYCAwECHgECF4AACgkQcSPLdg/xiGlcKxAAvkzIPVzhc2am
GaAAUse4mQ6InNqCLiGEbNxPWnd13myGesbESfyBxex5Gb2t47gVllA7P9hOcvsv
J6g56WPD1yb+5Wrdchdn6SkSEfg1MOAMTskFPPJJ/3ZgfHKn/kv3tOJPcQsidRFl
uqNMHSroHpOExYaTgB7IhYBjnYHLwUgH1ikCFgkRzdaDW4Qfx6IRB1vpSjzxCjzP
Cc78cf4VDmBdSfwsO6/ON19ZcxtLjHvQK5sz91qsEJdJZjyq6YCHYfP+Zx8/M55S
ixZ6/QsLRAsUYGuBjuWMpMgXjB06TXVbSg97bZt2tHBMZJ2OEMgn09eysyS9uwlY
HMtpu23jDTn6sKlRE6PYbZQirt2Ydq56wOQqJzrW1BbadX56DY8FLYBh8H9kcBaT
MGT1fiLfaEn1C8dG3D9aHdaPXZu+zGcMrPY+GMcObAAk3ICRXR20NknSByOxNEyz
nOqCsmr1nRdrpnf4+52G53xYKroWVRYeDdBukC8ik6weFjK4qy7C4ujOe1AmoBis
g6+R0huka1TYr9r94um+idvHniLnaZvnxPKEUMLnGesx9LYio4slqQj+nN6fIelv
onltXR49hpuAiOKtYUISmsk+rI+ep60DfDSQGbrV8HkW6KjuHjmE7EJKEEnIk5N/
JJfimlBk+rNgbcz0fpT6IDdS6PEoGwk=
=fj1X
-----END PGP PUBLIC KEY BLOCK-----
```
Let us register the public key.
```
> gpg --with-fingerprint perforce.pubkey
gpg: directory '/root/.gnupg' created
gpg: keybox '/root/.gnupg/pubring.kbx' created
gpg: WARNING: no command supplied.  Trying to guess what you mean ...
pub   rsa4096 2013-08-16 [SC] [expires: 2023-08-14]
uid           Perforce Software (Package Signing) <support+packaging@perforce.com>
```
Add the Perforce packaging key to your APT keyring.
```
> wget -qO - https://package.perforce.com/perforce.pubkey | sudo apt-key add -
OK
```
Execute the command for adding Perforce to your APT configuration.
```
> sudo add-apt-repository 'deb http://package.perforce.com/apt/ubuntu focal release'
Hit:1 https://repos.insights.digitalocean.com/apt/do-agent main InRelease
Hit:2 https://repos-droplet.digitalocean.com/apt/droplet-agent main InRelease
Get:3 http://mirrors.digitalocean.com/ubuntu focal InRelease [265 kB]
Hit:4 http://mirrors.digitalocean.com/ubuntu focal-updates InRelease
Hit:5 http://mirrors.digitalocean.com/ubuntu focal-backports InRelease
Get:6 http://package.perforce.com/apt/ubuntu focal InRelease [3650 B]
Hit:7 http://security.ubuntu.com/ubuntu focal-security InRelease
Get:8 http://package.perforce.com/apt/ubuntu focal/release amd64 Packages [8168 B]
Fetched 277 kB in 1s (298 kB/s)
Reading package lists... Done
```
Run update.
```
> apt-get update
Hit:1 https://repos.insights.digitalocean.com/apt/do-agent main InRelease
Get:2 http://mirrors.digitalocean.com/ubuntu focal InRelease [265 kB]
Hit:3 https://repos-droplet.digitalocean.com/apt/droplet-agent main InRelease
Hit:4 http://mirrors.digitalocean.com/ubuntu focal-updates InRelease
Hit:5 http://mirrors.digitalocean.com/ubuntu focal-backports InRelease
Hit:6 http://package.perforce.com/apt/ubuntu focal InRelease
Hit:7 http://security.ubuntu.com/ubuntu focal-security InRelease
Fetched 265 kB in 1s (321 kB/s)
Reading package lists... Done
```
Install the package.
```
> sudo apt-get install helix-p4d
Reading package lists... Done
Building dependency tree
Reading state information... Done
...
Started 0 services.
No services configured.
Processing triggers for man-db (2.9.1-1) ...
Processing triggers for systemd (245.4-4ubuntu3.6) ...
```
Now you have one last step, launching the Perforce service ! Execute the batch file for it.
```
> sudo /opt/perforce/sbin/configure-helix-p4d.sh

Summary of arguments passed:

Service-name        [(not specified)]
P4PORT              [(not specified)]
P4ROOT              [(not specified)]
Super-user          [(not specified)]
Super-user passwd   [(not specified)]
Unicode mode        [(not specified)]
Case-sensitive      [(not specified)]

For a list of other options, type Ctrl-C to exit, and then run:
$ sudo /opt/perforce/sbin/configure-helix-p4d.sh --help


You have entered interactive configuration for p4d. This script
will ask a series of questions, and use your answers to configure p4d
for first time use. Options passed in from the command line or
automatically discovered in the environment are presented as defaults.
You may press enter to accept them, or enter an alternative.

Please provide the following details about your desired Perforce environment:


Perforce Service name [master]:
```
You will be asked to enter some configurations such as name of service, directory, case sensitiviy, and so on. Setup them appropriately.
```
Perforce Service name [master]: Test
Service Test not found. Creating...
Perforce Server root (P4ROOT) [/opt/perforce/servers/Test]:
```
You should select the proper directory. It would be better to select the attached volume if the size of your project would be more than 25GB.
```
Perforce Service name [master]: Test
Service Test not found. Creating...
Perforce Server root (P4ROOT) [/opt/perforce/servers/Test]:
Create directory? (y/n) [y]: y
Perforce Server unicode-mode (y/n) [n]: y
Perforce Server case-sensitive (y/n) [y]:
Perforce Server address (P4PORT) [ssl:1666]:
Perforce super-user login [super]:
Perforce super-user password:
Re-enter password.
Perforce super-user password:

Configuring p4d service 'Test' with the information you specified...

Perforce db files in '/opt/perforce/servers/Test/root' will be created if missing...
...
::
::  - For help with creating Perforce Helix user accounts, populating
::    the depot with files, and making other customizations for your
::    site, see the Helix Versioning Engine Administrator Guide:
::
::    https://www.perforce.com/perforce/doc.current/manuals/p4sag/index.html
::
::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::
```
Now you can access the Perforce service via P4V at a client. Before that, take care of typemap. The typemap is an abbreviation of `Type Mapping`. You can define how Perforce handles certain type of files by it. If your project uses UnrealEngine, for the types related to UnrealEngine, Epic Games recommends to setup like below:
```
TypeMap:
                binary+w //depot/....exe
                binary+w //depot/....dll
                binary+w //depot/....lib
                binary+w //depot/....app
                binary+w //depot/....dylib
                binary+w //depot/....stub
                binary+w //depot/....ipa
                binary //depot/....bmp
                text //depot/....ini
                text //depot/....config
                text //depot/....cpp
                text //depot/....h
                text //depot/....c
                text //depot/....cs
                text //depot/....m
                text //depot/....mm
                text //depot/....py
                binary+l //depot/....uasset
                binary+l //depot/....umap
                binary+l //depot/....upk
                binary+l //depot/....udk
                binary+l //depot/....ubulk
```
You can edit the typemap of instance by executing command `p4 typemap`. The command would open typemap file with vi editor.
{% asset_img 05.png %}
Add the Epic Games's mappings to your mapping file*. Now all preparation of server side completed.
※ FYI, the `//` string does not mean "It is a kind of comment." in P4 typemap system. You should copy the all of text.

# _Install & Setup P4V_
Download the P4V installer at https://www.perforce.com/downloads/helix-visual-client-p4v and install with default options. When the installation compeleted, execute the P4V. You will see the display.
{% asset_img 06.png %}
Enter `ssl:X.X.X.X:1666` at the section `Server`. The `X.X.X.X` must be replaced with the IP address of instance. Enter `super` at the section `User`. The user `super` is an administrator account we have set. Now click the button `OK`. Check `Trust this fingerprint` and click the button `Connect` if you encounter the dialog like below:
{% asset_img 07.png %}
Enter the password you set while launching the Perforce service at instance.
{% asset_img 08.png %}
You can see the display when successfully entered.
{% asset_img 09.png %}
The admin tool can be accessed at `Tools/Administration`.
{% asset_img 10.png %}
In the tool, you can add or delete user directly.
{% asset_img 11.png %}
Let us prepare some Depot and Stream. Click the `Depots`. Right-click any depot and select `New Depot...`.
{% asset_img 12.png %}
Type the name of new Depot.
{% asset_img 13.png %}
Select `stream` at the section `Deopt type` and click `OK`.
{% asset_img 14.png %}
{% asset_img 15.png %}
Close the admin tool and return to the P4V*. Now you can find the new Depot at Depot view.
※ Actually, the admin tool was the program P4Admin, which is different with P4V. Just, Perforce supports to launch the P4Admin from P4V.
{% asset_img 16.png %}
Restart the P4V for applying changes from P4Admin. After restart, find the `File/New/Stream...` and click it.
{% asset_img 17.png %}
Let us make a Stream, name of `mainline`. The Stream will be placed in the new Deopt. Click the button `OK`.
{% asset_img 18.png %}
Click `New Workspace...` at workspace view.
{% asset_img 19.png %}
Name the new workspace and click the button `Browse` in line of `Stream`. You can find the Stream `mainline` at the dialog. Select it.
{% asset_img 20.png %}
{% asset_img 21.png %}
Finally, we have prepared a workspace in totally empty new Perforce service ! But, you should config `p4ignore` before getting into the work. Open any terminal and execute the command below:
```
> p4 set P4IGNORE=.p4ignore
```
This command will let your Perforce refer the file whose name is `.p4ignore`. It is kind of configuration lets you can use Perforce like git, which provides `.gitignore`. To apply this changes, restart P4V. And, create `.p4ignore` at your workspace directory. Let us test whether `.p4ignore` works well. Fill the contents of `.p4ignore` like below:
```
*.sln
```
Select `Mark for Add...` for `.p4ignore` and submit it.
{% asset_img 22.png %}
Next, create a empty file whose name is `Test.sln`. Try to add this !
{% asset_img 23.png %}
You try to check out the file, but the dialog would be popped-up. Great, your p4ignore works well.
{% asset_img 24.png %}
If your project uses UnrealEngine, you should search for good one. I recommend you to use [the p4ignore mentioned at references](https://github.com/mattmarcin/ue4-perforce/blob/master/.p4ignore). Okay, then...all of preparation done. You are good to go :) !