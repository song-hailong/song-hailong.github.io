---
layout: post
title: "CLion 安装 platformIO 教程"
date: 2023-05-05
tag: IDE
---

> 如已在Visual Studio Code中正常使用PlatformIO，则只需进行步骤3和步骤4。    
> 本教程在windowns环境下编写，macOS / Linux 下安装流程相同。

# 1. 安装Python

> 可参考：[python详细安装教程](https://zhuanlan.zhihu.com/p/104502997)
> 

前往[Python官网的下载页面](https://www.python.org/downloads/)，点击下载即可下载最新版本的安装包。

<img src="https://s2.loli.net/2023/05/05/QCoMVp2BSXIR36m.png" alt="Untitled" style="zoom:67%;" />

一定要勾选 **Add Python.exe to PATH**，然后点“Install Now”即可完成安装。

<img src="https://s2.loli.net/2023/05/05/ZMxFIsuTQ7kJPCh.png" alt="Untitled" style="zoom:50%;" />

# 2. 安装PlatformIO Core

要安装或升级 PlatformIO Core，先下载（右键→将链接另存为）[get-platformio.py](https://raw.githubusercontent.com/platformio/platformio-core-installer/master/get-platformio.py) 脚本，打开cmd，进入到脚本所在路径下，然后运行以下命令：

```python
# 进入到脚本所在路径下,根据实际填写
# cd D:\Users\xxx\Downloads

# run it
python get-platformio.py
```

如需在命令行中使用platformio命令，需配置环境变量：将`C:\Users\你的用户名\.platformio\penv\Scripts;`加到 Path 环境变量里。

<img src="https://s2.loli.net/2023/05/05/Ogd8tJKVa1C2IWZ.png" alt="Untitled" style="zoom:50%;" />

> 参考：https://docs.platformio.org/en/latest/core/installation/methods/installer-script.html

# 3. 安装CLion

有edu邮箱的可去Jetbrains官网申请[教育许可证](https://www.jetbrains.com/zh-cn/community/education/#students/)，或者使用[学信网](https://www.chsi.com.cn/)学籍验证报告申请。在[申请页面](https://www.jetbrains.com/shop/eform/students)选择对应的申请方式即可。完成后，选择CLion进行下载安装，安装完成后登录拥有教育许可证的JetBrains账号即可使用。

<img src="https://s2.loli.net/2023/05/05/5hY93rWxL82ntHl.png" alt="Untitled" style="zoom:50%;" />

没有教育许可证的可百度找破解版安装，版本号需大于2020.1。

# 4. 安装PlatformIO插件

先打开 CLion 的插件管理器, 然后搜索`PlatformIO`, 点 install 安装即可。

<img src="https://s2.loli.net/2023/05/05/Gj5KRyidaMJbmoC.png" alt="Untitled" style="zoom: 50%;" />

# 5. 使用

点击新建项目，选择PlatformIO，选择需要的开发板及框架，设置项目路径及名称，最后点击创建即可。

<img src="https://s2.loli.net/2023/05/05/xCbZ1ty2FVJr9jp.png" alt="Untitled" style="zoom:50%;" />

首次创建该开发板的工程时，需等待较长的一段时间，此阶段PlatformIO会下载该工程所需的配置文件。如创建失败可手动添加PlatformIO配置文件，见步骤7。

编译和下载按钮在右上角，按下图切换到上传模式即可烧入，如需调试则切换到调试模式。

<img src="https://s2.loli.net/2023/05/05/y3BrsSOuMDQjlFx.png" alt="1683289953496.png" style="zoom:50%;" />

<img src="https://s2.loli.net/2023/05/05/q3njGlhK4b8uTXP.png" alt="Untitled" style="zoom:45%;" />

如无PlatformIO配置，下拉后选择 编辑配置，在点击+号添加对应配置即可。

<img src="https://s2.loli.net/2023/05/05/LlSVvMFOxzp9nsJ.png" alt="Untitled" style="zoom:50%;" />

<img src="https://s2.loli.net/2023/05/05/m3czTfwgvLsI4Ce.png" alt="Untitled" style="zoom: 45%;" />

程序上传后，如在步骤2中添加了platformio环境变量，可在CLion中使用串口监视器。

打开 终端（Terminal），输入`platformio device monitor` ，就可以打开串口监视器，查看串口输出。再次烧入时需将此终端关闭，或者按 Ctrl+C 键退出串口监视器。

<img src="https://s2.loli.net/2023/05/05/W4rIsZVXmK8hqOF.png" alt="Untitled" style="zoom: 35%;" />

串口监视器的比特率默认是 9600，如想更换，比如 115200，除把代码中的`Serial.begin(9600)`换成`Serial.begin(115200)`外，还需在 platformio.ini 中加一句`monitor_speed = 115200`，否则会出现乱码。

# 6. 使用第三方库

## 6.1 PlatformIO提供的库

PlatformIO拥有丰富的第三方库。

使用 DHT22 传感器进行举例。

先打开 [PlatformIO 的 Libraries registry](https://registry.platformio.org/search)，勾选**Library**后搜索 DHT sensor library：

<img src="https://s2.loli.net/2023/05/05/jCvBOdK7m8lGkpS.png" alt="Untitled" style="zoom:50%;" />

找到DHT sensor library后，点击打开：

<img src="https://s2.loli.net/2023/05/05/TLjyEUWR1DV8JwF.png" alt="Untitled" style="zoom: 50%;" />

在库的详情页面，最需要关注的是 Readme 和 Installation，其中 Readme 介绍如何使用这个库, 而 Installation 介绍如何安装。

<img src="https://s2.loli.net/2023/05/05/ZaVyxho4GPHj5Rs.png" alt="Untitled" style="zoom: 33%;" />

首先安装，打开 Installation，根据提示将需要的库依赖写在 platform.ini 中即可：

<img src="https://s2.loli.net/2023/05/05/xLOjItCihvPwuNV.png" alt="Untitled" style="zoom:50%;" />

然后点锤子形状的编译按钮，会发现编译过程中有下载依赖的行为。

<img src="https://s2.loli.net/2023/05/05/mAoFUNPjzCMl4w1.png" alt="Untitled" style="zoom:50%;" />

不出意外的报错了。

<img src="https://s2.loli.net/2023/05/05/FOmVfAXZKcNU6uY.png" alt="Untitled" style="zoom:50%;" />

提示找不到头文件。

库被 PlatformIO 下载到了`./.pio/libdeps/开发平台名/`中，可以找到这些库存放头文件的路径 (一般直接就是第三方库目录，或者是其下的 src 目录)。

打开`CMakeLists.txt`，然后使用`include_directories`语句，将这些存放头文件的路径包含进去。

文件夹路径可通过 右键文件夹→复制路径/引用.…→来自内容根的路径 获取。

<img src="https://s2.loli.net/2023/05/05/nAHRVsSqKf7BEvJ.png" alt="Untitled" style="zoom:50%;" />

<img src="https://s2.loli.net/2023/05/05/HCFb7vkjtuWYgz9.png" alt="Untitled" style="zoom: 67%;" />

修改之后，重新右键，`Reload CMake Project`，或者点右上角的`Reload Changes`提示，重新载入CMake 工程。

<img src="https://s2.loli.net/2023/05/05/Vrv52yFPNmcBM3A.png" alt="Untitled" style="zoom: 67%;" />

可以看到，依然报错，且报错的内容与前面相同。╰（‵□′）╯

<img src="https://s2.loli.net/2023/05/05/prKNv8jkfC1u7tm.png" alt="Untitled" style="zoom:50%;" />

点开报错的内容，跳转到报错的地方。

<img src="https://s2.loli.net/2023/05/05/8tQoHIhsbwmJ6Tz.png" alt="Untitled" style="zoom:50%;" />

根据提示修改报错内容。

<img src="https://s2.loli.net/2023/05/05/yJ9NMUiRLuk3CtS.png" alt="Untitled" style="zoom:50%;" />

编译成功

<img src="https://s2.loli.net/2023/05/05/uUJFhwtnqHyWjlv.png" alt="Untitled" style="zoom:50%;" />

根据例程编写程序后发现，虽然可编译通过，但是编译器还是会报红，原因可能是库文件夹名称存在空格。

<img src="https://s2.loli.net/2023/05/05/9dVqp8ubz4DUY5P.png" alt="Untitled" style="zoom:50%;" />

修改库文件名称。右键文件夹→重构→重命名，作用域选所有位置，修改完成后点击重构。

<img src="https://s2.loli.net/2023/05/05/mr3Uq1ZiWw8cYFO.png" alt="Untitled" style="zoom:50%;" />

记得修改`CMakeLists.txt`中，包含库文件的路径。修改后点击 加载CMake更改。

<img src="https://s2.loli.net/2023/05/05/FX3ju4wplmazBWy.png" alt="Untitled" style="zoom:50%;" />

以及刚才修改的头文件。

<img src="https://s2.loli.net/2023/05/05/pXH9NqI6kMTDRKV.png" alt="Untitled" style="zoom:50%;" />

然后就可以使用了。

<img src="https://s2.loli.net/2023/05/05/KgpwTso9uPGQDl1.png" alt="Untitled" style="zoom: 45%;" />

> 参考：[用clion自带的platformIO和开发esp32!!!](https://blog.csdn.net/keysking/article/details/105925962?ops_request_misc=&request_id=&biz_id=102&utm_term=clion%20platformio&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduweb~default-4-105925962.142^v86^wechat,239^v2^insert_chatgpt&spm=1018.2226.3001.4187) 

## 6.2 其他库

只需将库文件下载下来后，复制到`lib`文件夹中，然后在`CMakeLists.txt` 文件中包含路径即可。

> Arduino IDE下载的库文件均在路径 `C:\Users\yourname\Documents\Arduino\libraries` 下

<img src="https://s2.loli.net/2023/05/05/MLqrutBJ2TRpc97.png" alt="Untitled" style="zoom:50%;" />

# 7. 手动添加PlatformIO数据包

> 如工程已创建成功，则不需进行此步骤。
> 

PlatformIO相关的文件均在路径 `C:\Users\yourname\.platformio` 下，其中 `yourname` 为电脑用户名称，因此将下载好的文件复制到此处即可。

<img src="https://s2.loli.net/2023/01/13/stW86U4xAJfCZwc.png" alt="Untitled" style="zoom:45%;" />

下面提供我的PlatformIO数据包，包含ESP32和ESP8266的arduino框架。

> 阿里云盘：[https://www.aliyundrive.com/s/pFDFnmdz8mi](https://www.aliyundrive.com/s/pFDFnmdz8mi)
CSDN下载：[PlatformIO 离线安装资源](https://download.csdn.net/download/qq_40018676/87383366)

<img src="https://s2.loli.net/2023/01/13/vUER6Vq5Dzd92Sy.png" alt="Untitled" style="zoom:45%;" />

<img src="https://s2.loli.net/2023/01/13/kMHZIgQ69tYsleE.png" alt="Untitled" style="zoom: 50%;" />