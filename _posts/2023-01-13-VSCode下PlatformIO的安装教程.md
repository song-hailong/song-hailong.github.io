---
layout: post
title: "VSCode下PlatformIO的安装教程"
date: 2023-01-13
tag: IDE
---

# 1. 安装VSCode软件

前往[VSCode官网](https://code.visualstudio.com/Download) ，建议下载 **System Installer** 版本的安装包，该版本安装在非用户目录，例如C盘根目录。如果下载速度太慢可查看 [国内下载vscode速度慢问题解决](https://zhuanlan.zhihu.com/p/112215618) 文章解决。

<img src="https://s2.loli.net/2023/01/13/7RmIr6Znl2Vq9GH.png" alt="Untitled" style="zoom:80%;" />

下载完毕后进行安装，安装时建议按如下进行勾选。

<img src="https://s2.loli.net/2023/01/13/7RYkBU1gdqpja35.png"  />

安装完毕后打开软件，安装中文插件。

<img src="https://s2.loli.net/2023/01/13/VnON7mRDFjs4JkG.png" alt="Untitled" style="zoom:33%;" />

# 2. 设置PlatformIO IDE使用非内置Python

> 此步骤可跳过，直接跳转到 *3. 安装PlatformIO插件* 。 
PlatformIO依靠Python运行，默认情况下会重新下载一个Python，如果电脑上已经安装过Python且不想再安装一次的，可进行如下步骤，否则直接前往 *3. 安装PlatformIO插件* 。

按下图打开设置的 JSON 文件。

<img src="https://s2.loli.net/2023/01/13/CxfzYeU6uApwD8q.png" alt="Untitled" style="zoom:50%;" />

<img src="https://s2.loli.net/2023/01/13/Zy6rwVx9KX3buSh.png" alt="Untitled" style="zoom: 33%;" />

打开配置文件后，输入如下两条配置信息，设置PlatformIO IDE使用非内置Python，其中 `customPATH` 为电脑已装的Python路径，需按照实际情况填写。

```json
    "platformio-ide.useBuiltinPython": false,
    "platformio-ide.customPATH": "C:\\Users\\yourname\\AppData\\Local\\Programs\\Python\\Python310\\Scripts",
```

<img src="https://s2.loli.net/2023/01/13/jRCXVtyncmO6ldZ.png" alt="Untitled" style="zoom:50%;" />

# 3. 安装PlatformIO插件

在VSCode中根据下图步骤安装PlatformIO IDE插件。

<img src="https://s2.loli.net/2023/01/13/LqPmy2szAaGpZOb.png" alt="Untitled" style="zoom: 45%;" />

插件安装完毕后，VSCode右下角会出现PlatformIO的下载进程，等待其下载完毕后即可。由于PlatformIO的服务器在国外，下载速度特别慢，如一直无法下载成功，可前往 *4. 手动添加PlatformIO数据包* 。

下载完毕后，PlatformIO只下载了公共的数据包，没有下载特定板子的数据包，因此如需使用他人的工程，需依照该工程所用芯片新建一个工程，在第一次新建工程时，PlatformIO会下载好该工程所需的文件。

假设需运行的工程所使用的芯片为ESP32-C3，使用的框架为arduino，那么在第一次运行该工程前需先按照如下步骤新建一次工程，只有第一次需要，后续就可以直接运行了。

点击VSCode左下角的桌面图标。

<img src="https://s2.loli.net/2023/01/13/Pn3HvsUIAtO4BXF.png" alt="Untitled" style="zoom:50%;" />

点击 **New Project** 。

<img src="https://s2.loli.net/2023/01/13/EkXlA2oIn95JzF8.png" alt="Untitled" style="zoom: 33%;" />

设置工程信息。

<img src="https://s2.loli.net/2023/01/13/mbDkcaNW2pIKsxu.png" alt="Untitled" style="zoom:50%;" />

点击 **Finish** 后，需要较长的一段时间，此阶段PlatformIO会下载该工程所需的文件，新建完成后，即可关闭此工程（直接关闭VSCode软件）。接着打开我们所需运行的工程即可（在工程路径下，鼠标右键后，选择通过 Code 打开）。

左下角的图标含义如图。

<img src="https://s2.loli.net/2023/01/13/FH2eDf8AuYvJnpk.png" alt="Untitled" style="zoom: 50%;" />

# 4. 手动添加PlatformIO数据包

> 如在进行 *3. 安装PlatformIO插件* 时已正常下载完成，则无需进行此步骤。
如无法正常下载，则可进行此步骤。

PlatformIO相关的文件均在路径 `C:\Users\yourname\.platformio` 下，其中 `yourname` 为电脑用户名称，因此将下载好的文件复制到此处即可。

<img src="https://s2.loli.net/2023/01/13/stW86U4xAJfCZwc.png" alt="Untitled" style="zoom:45%;" />

下面提供我的PlatformIO数据包，包含ESP32和ESP8266的arduino框架。

> 阿里云盘：[https://www.aliyundrive.com/s/pFDFnmdz8mi](https://www.aliyundrive.com/s/pFDFnmdz8mi)
CSDN下载：[PlatformIO 离线安装资源](https://download.csdn.net/download/qq_40018676/87383366)
> 

<img src="https://s2.loli.net/2023/01/13/vUER6Vq5Dzd92Sy.png" alt="Untitled" style="zoom:45%;" />

<img src="https://s2.loli.net/2023/01/13/kMHZIgQ69tYsleE.png" alt="Untitled" style="zoom: 50%;" />