---
title: 解决windows资源管理器提示桌面不可用的问题
url: 99.html
id: 99
categories:
  - 计算机杂论
date: 2018-04-04 21:13:29
tags:
---

很多人用压缩软件或者文件管理工具的时候都遇到过引用了一个不可用的位置如下图的问题 首先是打开注册表编辑器,windows键+R键输入regedit回车 找到下面两个条目： \[code lang=bash\] HKEY\_CURRENT\_USER\\\Software\\\Microsoft\\\Windows\\\CurrentVersion\\\ExplorerShell\\\ Folders \[/code\] \[code lang=bash\] HKEY\_CURRENT\_USER\\\Software\\\Microsoft\\\Windows\\\CurrentVersion\\\ExplorerUser\\\ Shell Folders \[/code\] 相应的在注册表右侧中都有一个“Desktop”的项，将上面两个键值对比一下，你会发现，两个键值并不相同，一般是“%USERPROFILE%Desktop”或“%USERPROFILE%桌面”和“c:Users你的用户名”。 建议把%USERPROFILE%Desktop修改为你的桌面的实际位置（例如c:\\Users\\你的用户名\\Dssktop）