---
title: "华为 A199 救砖札记"
toc: true
tags:
  - "Android"
  - "huawei"
  - "A199"
categories:
  - 玩机技巧
top:
---

## 1. 前言

这货之前一直被我丢在柜子里。晚上想要注册多一个微信号，需要多一台设备来登陆。毕竟双开有被封号的风险。

之前已经刷入了第三方的 REC，所以本以为挺简单。想着先刷回官方版本，然后自己精简的。

于是安装官方的指引：

1. 官方的包解压出来，把`dload`放到了外置 SD 卡
2. 设置-->关闭快速启动
3. 设置-->关于手机-->系统更新-->本地升级
4. 重启

**然后就崩了**...

第三方的 REC 无法识别，然后一直卡在 TWRP 的界面，重启或者试图进入 REC 都会一直卡在 TWEP 启动界面。

然后在官方论坛找了一下，没什么有用的帖子...刷机氛围真是不行...

直到看到某个官方教程帖子里说，华为的 fastboot 界面其实就是开机界面（麦芒A199 的就是那个麦芽）。使用**音量键减 + 开机键**进入。

能进入 fastboot 说明还是有救的。



## 2. 环境准备

### 2.1 安装驱动

我使用的是 Win10，需要安装华为的驱动之后才能识别设备，不然 fastboot 是无法查找到设备的。

[关键词搜索：麦芒 A199](http://consumer.huawei.com/cn/support/search/index.htm?keywords=%E9%BA%A6%E8%8A%92A199)

华为官方的搜索界面都找不到这机子了...滑到列表最下面，看到`HUAWEI Stick UTPS-V200R003B015D11SP01C983( for win10)`这个选项：

![HUAWEI Stick UTPS-V200R003B015D11SP01C983( for win10)](https://img.ioioi.top/wiki/58ca9d2bd48d2.png)

然后下载下来。如果官方撤了这个文件，可以到我分享的度盘下载：

```c
链接: http://pan.baidu.com/s/1dFadWkT 密码: bdyd
```

下载之后解压再解压，里面是一个`iso`镜像文件，可以选择装载到你的虚拟驱动器，或直接解压出来都没问题。

打开`AutoRun.exe`安装，就行了。

照道理现在应该已经装好了。我就是这么装的，并且之前我的系统也没有装过华为的其他驱动，所以我判断这样应该就可以。

### 2.2 刷入 REC

1. 获取 ADB 套件

这一步需要用到`adb`套件。

点击下面的链接从 Google 官方下载：

https://dl.google.com/android/repository/platform-tools-latest-windows.zip

如果因为某些**不可描述**的原因导致你无法下载，可以通过我分享的度盘链接下载：

```c
链接: http://pan.baidu.com/s/1miQCxnU 密码: kyum
```

*建议从官方下载，保证取得最新版的套件*。

下载之后解压出来，得到 `adb`套件。

1. 获取官方的 REC

麦芒的官方 REC 我在论坛没找到。所以需要从官方的 ROM 中提取出来。

下面是提取的步骤，如果你不想自己提取，可以直接用我提取出来：

```c
链接: http://pan.baidu.com/s/1eRO689O 密码: q4f9
```

[官方 ROM 下载](http://www.emui.com/plugin.php?id=hwdownload&mod=detail&mid=1)

如果官方撤掉了，从我的盘下载：

```c
链接: http://pan.baidu.com/s/1c21kqDq 密码: pi7u
```

提取 REC 需要用到一个提取软件，直接点下面下载：

```c
链接: http://pan.baidu.com/s/1boDhujH 密码: 5kun
```

下载完解压出来，双击运行`HuaweiUpdateExtractor.exe`。

选择`Update File`右边的按钮选择官方 ROM

![HuaweiUpdateExtractor](https://img.ioioi.top/wiki/58ca9fa275ed9.png)





选择 ROM 之后，右键单击`RECOVERY`解压出来：

![解压 REC](https://img.ioioi.top/wiki/58caa084186d0.png)



然后把解压出来的`RECOVERY`放到第一步中 ADB 套件的文件夹中。

1. 刷入 REC

进入 ADB 套件文件夹，可以看到诸如`adb`, `fastboot`等工具。

按住键盘的`Shift`键，同时鼠标右键单击该文件夹的空白处，选择“在此处打开命令行窗口”：

![open ps](https://img.ioioi.top/wiki/58caa1a512317.png)

我这里是`Powershell`窗口，大部分时候显示为`cmd`窗口。

按住手机**音量键减 + 开机键**进入`fastboot`模式，显示为开机的那个麦芽 logo。

进入命令行界面，输入以下命令检查设备是否链接：

```shell
fastboot.exe devices
```

如果有输入提示（一般是含有一串数字+英文字母），则意味着设备已经被识别了。如果没什么输出、提示，则意味着没有被识别。那么你要自行寻找其他驱动。

接着输入以下命令，进行刷入 REC

```shell
fastboot.exe flash recovery RECOVERY.img
```

输入回车之后应该就开始刷入了。命令行窗口会有提示：

![Flash REC](https://img.ioioi.top/wiki/58caa385361f6.png)

由于我之前是升级过程卡住了，刷入 REC 之后自动开始了流程，这点还是很赞的：

![show time](https://img.ioioi.top/wiki/58caa3eaaed61.jpg)

接着就可以进行愉快地玩耍啦~

------

*Best Wish.*

*by rosu*

*2017-3-16*

