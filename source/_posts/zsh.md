---
title: bash？zsh！
date: 2015-10-09 14:56:21
tags: [Ubuntu]
thumbnail: /images/zsh/thumbnail.png
---
 日常用Ubuntu，发现自带的bash不太好用，比如命令字体无高亮，无法自动补全，于是就有了下面这一幕

<!--more-->

![bash](/images/zsh/bash.png)

偶然了解到一个叫zsh的Shell，试用了一下，确实好用，现安利一下。

![zsh](/images/zsh/zsh.png)

### 安装

```shell
sudo apt-get install zsh
```

### 配置

网上有各种配置教程，也有各种配置文件。但我在这有更简单的方法

1 . 先[下载](https://codeload.github.com/trumi/zshrc/zip/master)我的配置文件，打开压缩包
2 . 将.zshrc文件复制到home目录下
3 .  将zsh作为系统默认的shell
```shell
# 为root用户修改默认shell为zsh
chsh -s /bin/zsh root
# 为当前用户修改默认shell为zsh
chsh -s /bin/zsh
# 恢复命令
chsh -s /bin/bash
```
4 . 打开Shell，进入新的世界吧
5 . 如果觉得配色不太喜欢，可自行更改配置文件，注释较全

注：我的这份配置文件也是来源于网上，但在此基础上做了一点修改，修复了几个bug。
