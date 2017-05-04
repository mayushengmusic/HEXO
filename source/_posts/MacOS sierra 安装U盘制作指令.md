---
title: 'MacOS sierra 安装U盘制作指令'
date: 2016-12-20 18:19:20
tags: MacOS
---

APP Store下载MacOS sierra安装包以后,在终端执行以下代码:
```bash
sudo /Applications/Install\ macOS\ Sierra.app/Contents/Resources/createinstallmedia --volume /Volumes/<USB> --applicationpath /Applications/Install\ macOS\ Sierra.app --nointeraction &&say Done
```