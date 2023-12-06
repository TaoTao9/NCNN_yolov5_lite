# 在vscode上使用cmake对NCNN进行debug

在上一篇博客使用[在树莓派下使用NCNN部署YOLOv5-lite-CSDN博客](https://blog.csdn.net/weixin_45725336/article/details/133946474?spm=1001.2014.3001.5501)都是直接使用命令行运行，但是具体如何使用vscode来进行debug调试，发现网上这方面没有专门讲这方面的，故写今天这个笔记。

首先回顾一下[在树莓派下使用NCNN部署YOLOv5-lite-CSDN博客](https://blog.csdn.net/weixin_45725336/article/details/133946474?spm=1001.2014.3001.5501)使用NCNN源码来进行编译，但是我们使用的是NCNN源码工程里面的example写好的CMakeLists.txt来生成，但是这样子太臃肿了，所以下面是使用NCNN源码编译出静态库，然后调用静态库来编写工程。

首先编译源码生成静态库，这里参考我之前写的这篇博客，搭建好依赖包的环境，之后

下载ncnn源码

```
https://github.com/Tencent/ncnn.git
cd ncnn
```

编译源码，在NCNN工程目录下，

```
mkdir build
cd build
cmake ../
make -j4
make install 
```

这样子打开工程目录下的build/install，这个install里面就是包括静态库和头文件的包，可以将install改名成ncnn，之后放在我们的工程里面，这里将[在树莓派下使用NCNN部署YOLOv5-lite-CSDN博客](https://blog.csdn.net/weixin_45725336/article/details/133946474?spm=1001.2014.3001.5501)里面使用到的cpp、bin、param文件添加到工程里面。

在工程下编写CMakeLists.txt文件

```
cmake_minimum_required(VERSION 3.0)#最小版本支持。
project(yolov5_list)#yolov5_list是工程的名字
set(CMAKE_CXX_STANDARD 11)
# 指定 ncnn 的安装路径
set(ncnn_DIR ncnn/lib/cmake/ncnn)
# 查找 ncnn
find_package(ncnn REQUIRED)
find_package(OpenCV REQUIRED)
# 输出 OpenCV 版本
message(STATUS "OpenCV version: ${OpenCV_VERSION}")
# 添加可执行文件,yolov5_lite是最终的执行文件的名字，yolov5_lite_e.cpp是运行的cpp文件。
add_executable(yolov5_lite yolov5_lite_e.cpp)
# 链接到 ncnn
target_link_libraries(yolov5_lite ncnn)
# 链接到 OpenCV，这里必须提前安装了opencv
target_link_libraries(yolov5_lite ${OpenCV_LIBS})
# 包含 OpenCV 头文件目录
target_include_directories(yolov5_lite PRIVATE ${OpenCV_INCLUDE_DIRS})
```

现在工程目录就变成这样子。

```
├── ncnn
├── CMakeLists.txt
├── v5lite-e.bin
├── v5lite-e.param
└── yolov5_lite_e.cpp
```

之后顺便打开vscode的debug按钮


之后在.vscode目录下的launch.json中编写运行的脚本

```
{
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "name": "yolov5_lite",//debug的名字，可以自定义
            "type": "cppdbg",
            "request": "launch",
            "program": "${workspaceFolder}/build/yolov5_lite",//生成的执行文件的路径，一般就是build路径加在CMakeList中的可执行文件的名字
            "args": ["/home/pi/tao_workspace/test_yolov5lite/banana1.jpg"],
            "stopAtEntry": false,
            "cwd": "${fileDirname}",
            "environment": [],
            "externalConsole": false,
            "MIMode": "gdb",
            "miDebuggerPath": "/usr/bin/gdb",//需要提前安装过gdb
            // "setupCommands": [
            //     {
            //         "description": "Enable pretty-printing for gdb",
            //         "text": "-enable-pretty-printing",
            //         "ignoreFailures": true
            //     },
            //     {
            //         "description": "Set Disassembly Flavor to Intel",
            //         "text": "-gdb-set disassembly-flavor intel",
            //         "ignoreFailures": true
            //     }
            // ],
            "preLaunchTask": "Build my project"
        }
    ]
}
```

在.vscode目录下的tasks.json中编写运行的脚本，没有就新建一个tasks.json，注意名字一定要是tasks.json。

```
{
    "version": "2.0.0",
    "options": {
        "cwd": "${workspaceFolder}/build/"
    },

    "tasks": [
        {
            "label": "cmake",
            "type": "shell",
            "command": "cmake",
            "args": [
                ".."
            ]
        },
        {
            "label": "make",
            "group":{
                "kind":"build",
                "isDefault":true
            },
            "command": "make",
            "args":[
            ]
        },
        {
            "label":"Build my project",//一定要跟launch.json的"preLaunchTask"的值一样
            "dependsOn":[
                "cmake",
                "make"             
            ]
        }
    ]
}
```

这样子就可以开始debug了，打断点，cmake模式选择debug，点击build构建项目，点击上面debug的按钮，按钮的名字是你在launch.json文件的"name"的值，


以上的内容具体参考我的博客：[在vscode上使用cmake对NCNN进行debug-TTao9博客](https://blog.csdn.net/weixin_45725336/article/details/134137558)

具体中间的细节我可能没办法面面俱到，具体可以参考b站这个教程：[单文件编译和调试](https://www.bilibili.com/video/BV1vv4y1o7y5?p=3&vd_source=93b6102f8bc10743dc9a66031e99eae3)
