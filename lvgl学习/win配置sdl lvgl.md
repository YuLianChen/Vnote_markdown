# win配置sdl lvgl
## 前言

目前网上`windows`仿真`LVGL`的资料都是比较久远的,不太适合现有的开发,因此重新整理了一下资料.

目标:

使用`Vscode`进行`LVGL`开发和仿真.

关键词: `LVGL`, `vscode`,`llvm`,`cmake`,`windows`

## 环境配置

编译器目前使用的是`llvm-MinGW-msvcrt`:[Releases · mstorsjo/llvm-mingw (github.com)](https://github.com/mstorsjo/llvm-mingw/releases)

`LVGL`官方提供了`vscode`的`demo`:[lvgl/lv_port_pc_vscode (github.com)](https://github.com/lvgl/lv_port_pc_vscode),不过官方只在linux和mac做了适配,在windows下还需要自己修改下参数

`LVGL`在`vscode`的仿真是基于`SDL`的,因此也需要下载下[Releases · libsdl-org/SDL (github.com)](https://github.com/libsdl-org/SDL/releases) .注意下载版本为:[SDL2-devel-2.30.6-mingw.zip](https://github.com/libsdl-org/SDL/releases/download/release-2.30.6/SDL2-devel-2.30.6-mingw.zip)

`Vsocde`就常用的插件,这里调试用的`codeLLDB`,自己下载下.

编译器下载了加入环境变量,这里就不细说了

`SDL`下载后放到一个常用的位置即可,不需要加入环境变量,加入后也没啥用,只需要记住解压到的位置就行

通过以下指令拉取最新代码

```
git clone --recursive https://github.com/lvgl/lv_port_pc_vscode
```

## 代码编译

使用code打开`simulator.code-workspace`工作区,使用`cmaketools`正常配置,选择上述下载的编译器即可.

[![](https://img2023.cnblogs.com/blog/2392961/202408/2392961-20240808114449942-486697652.png)](https://img2023.cnblogs.com/blog/2392961/202408/2392961-20240808114449942-486697652.png)

这时候编译会报找不到`SDL`的错误.

打开`CmakeLists.txt`,添加`SDL`的查找路径.

```
list(APPEND CMAKE_PREFIX_PATH "D:/Program Files/SDL_MINGW/")
set(SDL2_NO_MWINDOWS 1) # 设置为0的话不显示控制台,也就看不到打印信息
find_package(SDL2 REQUIRED SDL2)
```


还需要修改`main.c`,不然会报`error: undefined symbol: SDL_main`

```
#include "lvgl/demos/lv_demos.h"
#include LV_SDL_INCLUDE_PATH //包含SDL的头文件
#添加头文件不起作用，只能修改main函数为SDL_main
```

或者修改`main`函数为`SDL_main`函数

这时候就能正常编译了,但是还没法运行,因为缺少`SDL.dll`运行时,需要手动复制到可执行文件路径下,也可以在`CmakeLists.txt`中添加后处理执行

```
add_custom_target (run COMMAND ${EXECUTABLE_OUTPUT_PATH}/main DEPENDS main)
# 添加下面这行语句
add_custom_command(
        TARGET main  POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E copy
        "${SDL2_DIR}/../x86_64-w64-mingw32/bin/SDL2.dll" ${EXECUTABLE_OUTPUT_PATH}
        )
```

然后就可以愉快的运行了

[![](https://img2023.cnblogs.com/blog/2392961/202408/2392961-20240808114451257-1717304764.png)](https://img2023.cnblogs.com/blog/2392961/202408/2392961-20240808114451257-1717304764.png)

## 调试

打开`simulator.code-workspace`

找到`LLVM`,修改`cppdbg`为`lldb`

        ```
        {
            "name": "Debug LVGL demo with LLVM",
            "type": "lldb",
            "request": "launch",
            "program": "${workspaceFolder}/bin/main",
            "args": [],
            "cwd": "${workspaceFolder}",
            "preLaunchTask": "Build",
            "stopAtEntry": false,
            "MIMode": "lldb"
        },
```

然后就可以调试了

[![](https://img2023.cnblogs.com/blog/2392961/202408/2392961-20240808114452614-1230261976.png)](https://img2023.cnblogs.com/blog/2392961/202408/2392961-20240808114452614-1230261976.png)

* [前言](#前言)
*     [环境配置](#环境配置)
*     [代码编译](#代码编译)
*     [调试](#调试)

__EOF__

[![](https://pic.cnblogs.com/avatar/2392961/20210511195457.png)](https://pic.cnblogs.com/avatar/2392961/20210511195457.png)

* **本文作者：** [USTHzhanglu](https://www.cnblogs.com/USTHzhanglu)
* **本文链接：** [https://www.cnblogs.com/USTHzhanglu/p/18348535](https://www.cnblogs.com/USTHzhanglu/p/18348535)
* **关于博主：** 评论和私信会在第一时间回复。或者[直接私信](https://msg.cnblogs.com/msg/send/USTHzhanglu)我。
* **版权声明：** 本博客所有文章除特别声明外，均采用 [BY-NC-SA](https://creativecommons.org/licenses/by-nc-nd/4.0/ "BY-NC-SA") 许可协议。转载请注明出处！
* **声援博主：** 如果您觉得文章对您有帮助，可以点击文章右下角**【推荐】**一下。