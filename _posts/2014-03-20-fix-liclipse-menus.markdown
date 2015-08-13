---
layout:     post
title:      "How to fix LiClipse menus in Ubuntu"
date:       2014-03-20 10:00
tags:       LiClipse, Python
slug:       how-to-fix-liclipse-menus-in-ubuntu
subtitle:    "For some people the LiClipse menus won’t work out of the box in Ubuntu. Since I spent quite a bit of time tracking this problem down I might as well make a post about it."
---

LiClipse is a lightweight Eclipse for (mainly) Python development. For some people the LiClipse menus won’t work out of the box in Ubuntu. Since I spent quite a bit of time tracking this problem down I might as well make a post about it.

Find your liclipse.desktop file by opening terminal and typing:

```
sudo find / -name liclipse.desktop
```

The output of the command on my setup was:

```
/home/kire/.local/share/applications/liclipse.desktop
```

Edit the liclipse.desktop file by typing

```
sudo -H gedit /home/kire/.local/share/applications/liclipse.desktop
```

Then replace the line starting with "`Exec=`"
with a following line

```
Exec=env UBUNTU_MENUPROXY= /opt/liclipse/LiClipse
```

NB! The “`Exec=`” line might be very long so don’t be intimidated by deleting this. 

Save the file.