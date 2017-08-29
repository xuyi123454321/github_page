---
title: Linux上类似Mac Spotlight的字典实现方案
date: 2017-07-18 14:54:08
tags:
- Linux
- Fun
---

## 笔者的初衷和工作环境
我在Linux环境中使用StarDict这款字典，一直以来都没有遇到什么问题，StarDict的取词功能也很方便，虽然已经很久没有维护的样子，还是能找到很多可用的字典。美中不足的是查词还是需要唤出主界面，笔者一直觊觎Mac上Spotlight提供的字典功能，希望在Linux上也能有一种界面简洁至极的查词解决方案。后来我了解到StarDict有命令行版本sdcv (StarDict Console Version ?)，于是很长时间内就采用了快捷键唤出终端模拟器、输入sdcv查词再Ctrl+D退出的解决方案（假装比调出主界面少按几个快捷键）。但是terminator（笔者常用的终端模拟器）还是比不上Spotlight好看。

当初我使用i3wm的时候接触到rofi这个工具，使用效果大概是这样的（图片来自Google）。
{% asset_img rofi_demo.png rofi demostration %}
这个东西的功能类似于dmenu，就是把输入组织成多项显示出来，具体介绍也可以去官网了解[rofi](https://davedavenport.github.io/rofi/)。总之rofi的界面基本满足我的审美要求，唤出和退出也很方便，一个快捷键和Esc足矣。本文将介绍怎么用rofi和sdcv配置一个好用+美观的查词工具。

顺便介绍一下笔者现在的工作环境，当然一张screenfetch截图足矣。
{% asset_img screenfetch.png screenfetch %}
我的rofi版本是1.3.1，sdcv版本是0.5.0-beta2。

## 核心原理
实现这个小工具的关键在于rofi的使用。读者可以运行一下下面命令体验一下rofi的功能。
```bash
echo -e "Option 1\nOption 2\nOption 2\n" | rofi -dmenu # dmenu工作模式
rofi -show run # 用作应用选单
```
阅读rofi文档可以知道rofi可以用`-modi`选项指定运行的程序，比如下面的例子。
```bash
rofi -modi ls:'ls /' -show ls # 格式 -modi <name>:<script>
```
在笔者的环境下用rofi的时候并不能愉快的引用bash环境变量，比如`rofi -modi ls:'ls $HOME' -show ls`就不能正常运行。

那么要实现查词功能我们只要采用下面命令就可以啦。
```bash
rofi -modi sdcv:sdcv -show sdcv
```
效果大概是这个样子
{% asset_img demo1.png dictionary demostration %}

如果读者成功实(bei)现(keng)就会发现如果查词的时候输入字典中不存在的单词，rofi就会卡住，甚至整个桌面环境都卡住，必须切tty把rofi杀死。这是因为sdcv在获得不存在的单词时会进入交互模式，一直等待输出，而rofi这个东西优先级还是很高的，于是就无法进行其他图形界面操作啦。解决方法很简单，只要用sdcv的`-n`选项禁用交互就可以了。命令如下。
```bash
rofi -modi sdcv:'sdcv -n' -show sdcv
```

## 个性化和快捷键配置
读者在自己的电脑上运行上面代码效果可能与上面的图相差较大，这是由rofi主题决定的。rofi的一个优点就是颜色配置的方便，可以参考官网的主题。
[https://davedavenport.github.io/rofi/p05-Themes.html](https://davedavenport.github.io/rofi/p05-Themes.html)
配置需要写入`$HOME/.Xresources`中从而在登录时启用，或者用`xrdb -merge ~/.Xresources`手动载入。还可以通过rofi命令的一些选项改变显示的字体和行数。我的命令是这样的
```
rofi -modi sdcv:'sdcv -n' -show sdcv -font 'mono 18' -lines 3
```

如何配置快捷键我这里就不解释了，但是我想说一下我在配置快捷键时遇到过没有显示的问题。通过重定向输出得到Lacale设置没能成功载入的错误信息，所以我最后在快捷键中采用的命令如下
```
/bin/bash -c "source /etc/profile; /usr/bin/rofi -modi sdcv:'sdcv -n' -show sdcv"
```

顺便提一下rofi可以支持多个应用并用Tab‘键切换，所以有兴趣的朋友可以自己试着实现更多的功能。

Happy Hacking :)
