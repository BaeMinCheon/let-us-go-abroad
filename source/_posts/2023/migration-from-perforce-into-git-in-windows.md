---
title: Migration from Perforce into Git in Windows
date: 2023-12-04 22:10:03
tags: [Perforce, Git, Migration]
---

# _`Overview`_

Sometimes, you have to switch the version control system for some reason. In this post, I will cover how to migrate Perforce stream into Git repository. I have confirmed that the method in this post works only in Windows, but you might be able to accomplish the same result with a similar way.

# _`Prerequisites`_

First of all, you should have Git and Perforce installed. Any latest version would be okay. Plus, you should be able to use their commands through the command prompt. For instance, the commands below should be working:

```
> p4 -V
Perforce - The Fast Software Configuration Management System.
Copyright 1995-2023 Perforce Software.  All rights reserved.
This product includes software developed by the OpenSSL Project
for use in the OpenSSL Toolkit (http://www.openssl.org/)
Version of OpenSSL Libraries: OpenSSL 1.1.1u  30 May 2023
See 'p4 help [ -l ] legal' for additional license information on
these licenses and others.
Extensions/scripting support built-in.
Parallel sync threading built-in.
Rev. P4/NTX64/2023.1/2468153 (2023/07/24).
```

```
> git -v
git version 2.42.0.windows.2
```

Second, you should have Python installed. The version after 2.7 would be okay. (eg. 2.8 or 3.5) Plus, you should be able to use its commands through the command prompt. For instance, the commands below should be working:

```
> python -V
Python 3.11.6
```

The last one, you have to change your system locale settings if you had written the description of changelist with non-ascii codes. You can enable the option `Beta: Use Unicode UTF-8 for worldwide language support` from the depth of `Control Panel/All Control Panel Items/Region/Administrative/Change system locale...`.

{% asset_img 01.png %}

Unless the option enabled, the commit message from migration result can be seen as `untranslatable` if the description was written with non-ascii codes.

{% asset_img 02.png %}

With the option enabled, the commit message would be migrated properly just like the image below. So, check your descriptions in Perforce and change the system locale settings.

{% asset_img 03.png %}

# _`Migration`_

Type the command of format `python <path of git-p4> clone //<depot>/<stream>/<directory>@all` in prompt. For instance, I can type the command like this:

```
> python "C:\Program Files\Git\mingw64\libexec\git-core\git-p4" clone //HellLady/mainline/HellLady@all
```

Then, all changelists from the `//<depot>/<stream>/<directory>` will be migrated into a Git repository.

{% asset_img 04.png %}

# _`Postscript`_

That is all about the migration. 😂 So simple, but it was hard to know because the official document does not cover the usage in Windows. Anyway, I hope this would be helpful for you. Check [the official document](https://git-scm.com/docs/git-p4) for more details if you also need other commands. Good luck. 🤞