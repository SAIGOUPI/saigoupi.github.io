---
layout: post
title: 'CE（Cheat Engine）本地编译教程'
subtitle: '使用CE源码本地编译（绿色健康）'
date: 2025-08-07
categories: 游戏开发
tags: 
- Cheat Engine 
- Game
---
## 为什么要本地编译CE？

Cheat Engine（CE）是一个内存操控工具，可以通过修改扫描内存，并修改指定内存的数值的方式来修改游戏内数值，达到作弊的效果。大部分人都是用其来进行单机游戏的数值修改，我最近在玩怪物猎人荒野，到后期实在是不想刷了，于是也开始研究这些作弊工具。

CE官方其实提供了安装文件，不用自己编译。可是看网上说这个安装程序，会捆绑安装一些其他的流氓软件（可以在安装时通过勾选不安装），本来我对CE这类“开挂软件”就不太了解，所以还是有一些防备心理的。CE官方安装程序捆绑一些软件，也是为了生计着想，毕竟人家是免费开源软件。

另外，CE的Github上也有本地编译的步骤说明，稍微研究一下，发现整个过程也很简单。有源码，有教程，本地编一下其实也很快，而且还更加放心。

## 前期准备

CE的Github官方库下载最新的release源码

[cheat-engine官方Git仓库](https://github.com/cheat-engine/cheat-engine/releases/tag/7.5)

根据官方的README要求，下载 `Lazarus` IDE

[Lazarus官方下载](https://sourceforge.net/projects/lazarus/files/Lazarus%20Windows%2064%20bits/Lazarus%202.2.2/)

> info "Lazarus是什么？" 
> Lazarus 并不是一个编译器，而是一个 Free Pascal 编译器 的集成开发环境(IDE)。它基于Free Pascal 编译器，用于开发跨平台的应用程序，特别是使用 Object Pascal 语言的图形用户界面(GUI) 应用程序。简单来说，Lazarus 提供了一个可视化界面，让开发者可以像使用Delphi 一样方便地开发应用程序，而背后实际执行编译工作的，是Free Pascal 编译器。

按照如下要求分别下载两个exe安装文件
> The default installer is:
> 
> `lazarus-2.2.2-fpc-3.2.2-win64.exe`
> 
> You should download this file, if you want to work on any Windows 64 bit version.
> The installers include FPC 3.2.2.2 and they include the Lazarus help files.
> 
> Add-On for building and debugging 32bit Windows applications:
> 
> `lazarus-2.2.2-fpc-3.2.2-cross-i386-win32-win64.exe`
> 
> This file can be installed as add-on to the 64 bit Lazarus IDE (on Systems with Windows 64 bit only), if you wish to develop for 32bit Windows too.

首先运行第一个exe，安装完毕后，再运行第二个exe安装额外功能。

## CE软件编译

打开Lazarus，任务栏中打开`Project-> Open Project`，在下载的源码中找到`Cheat Engine`文件夹下`cheatengine.lpi`的文件，双击就可以打开CE的工程了。


![Lazarus打开项目](/assets/img/posts/2025-08-07-CE-localbuild/image.png)

任务栏左上角在编译的地方更换目标版本，win11就用`Release 64-Bit`即可。

![更换64位版本](/assets/img/posts/2025-08-07-CE-localbuild/image-3.png)

最后任务栏点击`Run-> Build`就会开始编译了，显示绿色Success就是编译好了，然后到`Cheat Engine\bin`路径下，找到`cheatengine-x86_64.exe`，双击运行就会启动CE软件了。

![编译成功](/assets/img/posts/2025-08-07-CE-localbuild/image-4.png)

## CE教程小游戏编译

CE软件已经可以直接用了，使用教程在网上一搜一大堆。

另外为了方便用户入门，CE自己还专门做了一个教程小游戏，可以让用户边学边玩，教程也是一个单独的工程，还是同样的打开工程，找到路径`Cheat Engine/Tutorial`下的`tutorial.lpi`工程，双击打开

![打开教程工程](/assets/img/posts/2025-08-07-CE-localbuild/image-5.png)

和上面同样的操作，选择版本后点击编译，这时可能会有如下报错:

```
newvirtualstringtree.pas(8,32) Fatal: Cannot find laz.VirtualTrees used by newvirtualstringtree. Check if package laz.virtualtreeview_package is in the dependencies.
```

很简单，就是项目缺少依赖，按照以下步骤添加依赖包`laz.virtualtreeview_package`即可
> Go to "Project" in the top left and then click "Project Inspector".
Click on the Add button with the plus.
Select the page New Requirement.
Write "laz.virtualtreeview_package" into the "Package-Name" field.
Click Ok.

![添加依赖](/assets/img/posts/2025-08-07-CE-localbuild/image-6.png)

![添加依赖](/assets/img/posts/2025-08-07-CE-localbuild/image-7.png)

![添加依赖](/assets/img/posts/2025-08-07-CE-localbuild/image-8.png)

然后再重新编译，成功之后在CE软件的同路径下，找到`tutorial-x86_64.exe`，双击就可以打开教程小游戏了

![CE教程小游戏](/assets/img/posts/2025-08-07-CE-localbuild/image-9.png)