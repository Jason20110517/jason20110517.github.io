---
layout: post
title: "Debian系Linux破解密码"
date:   2024-2-9
tags: [Linux]
comments: true
author: jason20110517
---

### Debian系Linux破解密码

1.开机长按`Shift`，进入grub菜单；

2.通过键盘$\uparrow\downarrow$​键选择`Advanced options for Ubuntu`，按下`Enter；`

3.通过键盘$\uparrow\downarrow$键随便选到一个你认识的系统，按`e`；

4.在打开的编辑菜单中通过键盘$\uparrow\downarrow$键选到`linux`开头的那一行，再通过键盘$\leftarrow\rightarrow$键选到字符串`ro`的字符`r`上，通过`Del`键把后面的字符到`locate=en_US`全部删除，再填入`rw init=/bin/bash`，按`F10`保存；

5.在打开的终端中输入`mount -o remount,rw /`，按`Enter`执行，再执行`ls /home`可以输出你的用户名，然后执行`passwd <你的用户名>`然后输入两遍新密码，注意输入密码时不会显示，同理可以修改`root`密码；

6.修改完成后，执行`sync`和`exec /sbin/init`即可进入系统；
