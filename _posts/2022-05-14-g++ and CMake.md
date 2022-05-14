---
layout: post
title: 	VS Code 基于 g++ and CMake 编译 C/C++
date: 2022-05-14
tags: C/C++
---

# g++ and CMake

> 本文档为视频笔记整理
>
> 对应B站视频：<https://www.bilibili.com/video/BV13K411M78v?p=1>

# 1 开发环境搭建

## 1.1 安装mingw-w64编译器（GCC for Windows 64 & 32 bits）、Cmake工具（选装）

*   下载完毕后直接解压复制到文件夹，例复制到D:\ProgramFiles。

*   添加环境变量：

    > D:\ProgramFiles\cmake\bin
    > D:\ProgramFiles\mingw64\bin

*   验证安装是否成功，在终端里分别输入**gcc** 、**g++** 和 **cmake** 进行验证。

![](https://s2.loli.net/2022/05/14/EtSHKW5zoxuQXrm.png)

## 1.2 VSCode插件安装

*   C/C++

*   cmake（选装）

*   cmake tools（选装）

***

# 2 基于 g++ 命令编译

## 2.1 编译单文件

新建一个文件夹后，右键用VS Code打开，在VS Code里面新建一个.cpp文件，例 main.cpp。写好代码之后，打开当前路径的终端，输入以下命令后回车进行编译。

```c++
g++ .\main.cpp
```

> 输入 .\main.cpp 时，可输入main后按【Tap】键自动补全。

编译完成后会生成一个.exe文件，此时文件名为系统自动命名，例 a.exe，接着输入以下命令运行该文件。

```c++
.\a.exe
```

> 以上编译命令生成的.exe文件未带调试信息，不可进行调试。

### 生成带调试信息的的可执行文件

```c++
g++ -g .\main.cpp -o my_single_swap
```

> \-g 生成带调试信息的可执行文件
> \-o 指定生成可执行文件的名字

执行该命令后会生成一个 my\_single\_swap.exe 的可执行文件，并且该文件可以进行调试。

### 调试

打开【运行和调试】，点击【创建launch.json】，选择环境【C++（GDB/LLDB）】，选择配置【g++.exe】，此时json文件就创建好了。

![](https://s2.loli.net/2022/05/14/R8YuTpJvoKrf7qN.png)

![](https://s2.loli.net/2022/05/14/6xmT9MhUBCYildP.png)

![](https://s2.loli.net/2022/05/14/Vr6qUC2HLKS8b4o.png)

打断点进行调试，打断点方式为：在每行程序序号前点击鼠标左键，或者按F9键。

点击【运行】→【启动调试】（快捷键F5），开始调试。

## 2.2 编译多文件

```c++
g++ -g .\main.cpp .\swap.cpp -o my_multi_swap
```

执行该命令后会生成一个 my\_multi\_swap.exe 的可执行文件。

### 调试

创建json文件步骤与编译单文件一样，创建好json文件之后，打开launch.json，将"program"里的路径修改为"\${fileDirname}/my\_multi\_swap.exe"，也就是指向生成的可执行文件my\_multi\_swap.exe，由于已经手动生成了可执行文件，把"preLaunchTask"行注释掉。接着就可以进行打断点调试了。

***

# 3 基于cmake进行编译

在main.cpp同级目录下，新建一个【CMakeLists.txt】文件，在文件夹里面输入以下代码：

```makefile
project(MYSWAP)

add_executable(my_cmake_swap main.cpp swap.cpp)
```

> project 为项目名称；
> add\_executable 为编译信息，第一个参数为 生成的.exe文件名，后面的参数为待编译的文件。

保存后打开【查看】→【命令面板】（快捷键：Ctrl+Shift+P），输入【cmake】后选择【CMake: Configure】，选择工具包【GCC 8.1.0】，然后VSCode左下角会显示：CMake: \[Debug]: Ready，并且目录下会有一个【build】文件夹。

打开终端，进入到build文件夹，输入以下命令：

```makefile
cd .\build\
cmake ..
mingw32-make.exe   #替换成cmake —build .  也可

```

![](https://s2.loli.net/2022/05/14/LBU7xy3AdgiFKjN.png)

可执行文件my\_cmake\_swap.exe就生成好了。

以上为软件生成build文件，若需手动生成，在终端输入以下命令：

```makefile
# 新建build文件夹
mkdir build
# 进入到build目录
cd build
# 如果电脑上已安装了VS，可能会调用微软MSVC编译器，
# 使用(cmake -G "MinGW Makefiles" ..) 代替(cmake ..)
# 仅第一次使用cmake时使用（cmake -G "MinGW Makefiles" ..），后面可使用（cmake ..）

# cmake编译上级目录
cmake .. 
# 执行windows下的make指令
mingw32-make.exe

```

### 调试

打开launch.json，将"program"里的路径修改为"\${workspaceFolder}\\\build\\\my\_cmake\_swap.exe"，也就是指向生成的可执行文件my\_cmake\_swap.exe，把"preLaunchTask"行注释掉。接着就可以进行打断点调试了。

***

# 4 配置json，一键生成并调试

有两个json文件，分别如下：

*   launch.json – for debug

    作用：配置调试信息，用来调试编译好的文件：

    1.  program：可执行文件的路径；

    2.  preLaunchTask：执行调试前所执行的task

*   tasks.json – for build before debug

    作用：包含调试前的操作指令，用来做调试前的编译工作

    1.  可以避免每次修改代码后，手动编译；即tasks.json其实是和手动编译 的作用等价的。

    2.  tasks.json包含了某个task的编译命令: 编译代码，并生成可执行文件。

    3.  label 应与launch.json中的preLaunchTask名字一致

前提：按照2.1里的调试步骤新建好launch.json文件。

新建【tasks.json】文件，在新建launch.json时会建好，若无，把launch.json里的"preLaunchTask"行注释取消，按下F5，在错误弹窗中选择【Configure Task】，再选择【C/C++: g++.exe build active file】；或者打开【查看】→【命令面板】（快捷键：Ctrl+Shift+P），输入【tasks】后选择【Tasks: Configure Task】，再选择【C/C++: g++.exe build active file】。

## 4.1 通过g++命令一键编译

将tasks.json里的内容修改成以下：

```json
{
  "version": "2.0.0",
  "tasks": [
    {
      "type": "cppbuild",
      "label": "C/C++: g++.exe 生成活动文件",
      "command": "D:\\ProgramFiles\\mingw64\\bin\\g++.exe",
      "args": [
        "-fdiagnostics-color=always",
        "-g",
        "main.cpp",
        "swap.cpp",
        "-o",
        "${fileDirname}\\out.exe"
      ],
      "options": {
        "cwd": "${fileDirname}"
      },
      "problemMatcher": [
        "$gcc"
      ],
      "group": "build",
      "detail": "编译器: D:\\ProgramFiles\\mingw64\\bin\\g++.exe"
    }
  ]
}
```

将launch.json里的内容修改成以下：

```json
"version": "0.2.0",
    "configurations": [
        {
            "name": "g++.exe - 生成和调试活动文件",   //launch的名字
            "type": "cppdbg",                      //调试类型
            "request": "launch",
            "program": "${fileDirname}/out.exe",   //可执行文件的路径
            "args": [],                            //调试main()括号里的参数
            "stopAtEntry": false,
            "cwd": "${fileDirname}",               //必须是大文件夹目录
            "environment": [],
            "externalConsole": false,
            "MIMode": "gdb",
            "miDebuggerPath": "D:\\ProgramFiles\\mingw64\\bin\\gdb.exe", //调试器路径
            "setupCommands": [
                {
                    "description": "为 gdb 启用整齐打印",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                }
            ],
            "preLaunchTask": "C/C++: g++.exe 生成活动文件"
        }
    ]
}
```

主要修改内容为tasks.json里面的"args"，"command"和"args"相当于命令：

```makefile
g++ -g .\main.cpp .\swap.cpp -o my_multi_swap
```

launch.json文件里"program"的路径要与tasks.json里"args"的输出路径对应。

launch.json文件里"preLaunchTask"要与tasks.json里的"label"一样。

修改完成后就可以按F5一键生成和调试了。

## 4.2 通过cmake一键编译

前提：已经按照3里的步骤创建好CMakeLists.txt文件。

将tasks.json里的内容修改成以下：(此处tasks.json的作用相当于命令cmake ..和mingw32-make.exe)

```json
{   
    "version": "2.0.0",
    "options": {
        "cwd": "${workspaceFolder}/build"
    },
    "tasks": [
        {
            "type": "shell",
            "label": "cmake",
            "command": "cmake",
            "args": [
                ".."
            ],
        },
        {
            "label": "make",
            "group": {
                "kind": "build",
                "isDefault": true
            },
            "command": "mingw32-make",
            "args": [

            ],
        },
        {
            "label": "Build",
            "dependsOn":[
                "cmake",
                "make"
            ]
        }
    ],
}

```

将4.1中launch.json文件里"program"的路径修改成指向CMakeLists.txt文件里的输出文件。

![](https://s2.loli.net/2022/05/14/lAqbedYRJ6f9h1L.png)

修改完成后就可以按F5一键生成和调试了。
