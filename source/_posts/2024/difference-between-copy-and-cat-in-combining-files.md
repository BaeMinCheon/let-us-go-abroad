---
layout: 2024
title: Difference between copy and cat in combining files
date: 2024-05-13 22:46:37
tags: [Windows, Linux, Copy, Cat]
---

# _`Overview`_

There are some commands for combining files such as `copy` and `cat`. Respectively, `copy` is a command for Windows prompt and `cat` is a command for Unix prompt. You can check the specifications at official documentations below:

- https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/copy
- https://www.man7.org/linux/man-pages/man1/cat.1.html

# _`Comparison #1`_

Suppose we have two text files, `a.txt` and `b.txt`. Each of them has simple contents.

{% asset_img 01.png %}

{% asset_img 02.png %}

We gotta combine them into one text file, `c.txt`.

In Windows, it would be like this:

```
C:\Users\qmffk\Downloads>copy a.txt + b.txt c.txt
a.txt
b.txt
        1 file(s) copied.

C:\Users\qmffk\Downloads>type c.txt
hello, a !hello, b !
C:\Users\qmffk\Downloads>
```

In Linux, it would be like this:

```
thanang@ROSS-DESKTOP:/mnt/d/test$ cat a.txt b.txt > c.txt
thanang@ROSS-DESKTOP:/mnt/d/test$ cat c.txt
hello, a !hello, b !thanang@ROSS-DESKTOP:/mnt/d/test$
```

Both look like the same, but there is a difference between two `c.txt`.

{% asset_img 03.png %}

Using [WinMerge](https://winmerge.org/downloads/?lang=en), we can see the additional character `1A` (in hex) at Windows' `c.txt`. In short, the character `1A` (`26` in decimal) is appended for indicating an EOF (= End Of File). This is why this is mentioned in the documentation. (For more information about the character `1A / 032 / SUB / Substitute`, visit [here](https://www.ascii-code.com/))

```
...You can copy an ASCII text file that uses an end-of-file character (CTRL+Z) to indicate the end of the file...

...To copy a file called memo.doc to letter.doc in the current drive and ensure that an end-of-file character (CTRL+Z) is at the end of the copied file...
```

So, how do we prevent from appending the EOF character ? This is also mentioned in the documentation.

```
The effect of /b depends on its position in the command–line string: - If /b follows source, the copy command copies the entire file, including any end-of-file character (CTRL+Z). - If /b follows destination, the copy command doesn't add an end-of-file character (CTRL+Z).
```

# _`Comparison #2`_

Do combine `a.txt` and `b.txt` into `c.txt` again.

In Windows, it would be like this:

```
C:\Users\qmffk\Downloads>copy /b a.txt + b.txt c.txt
a.txt
b.txt
        1 file(s) copied.

C:\Users\qmffk\Downloads>type c.txt
hello, a !hello, b !
C:\Users\qmffk\Downloads>
```

In Linux, it would be the same with before.

{% asset_img 04.png %}

Using WinMerge, we can see they are the same.

So, you should plus the flag `/b` in `copy` commandline for experiencing the same result as `cat` in Unix.