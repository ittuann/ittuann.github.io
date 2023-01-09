---
layout: article
title: STM32CubeIDE的一点使用技巧
date: 2021-12-24
key: P2021-12-24-1
tags: ["STM32","RoboMaster"]
show_author_profile: true
comment: true
sharing: true
aside:
  toc: true
---

探索 STM32CubeIDE 的功能。记录了些基础功能还有方法和技巧等等。

<!--more-->

来到RM之后平台稳定的换到了STM32。很多IDE都支持STM32的开发，比较常用的是MDK的Keil，也有用IAR来开发的。后来在ST官网发现了STM32CubeIDE，这个IDE相当于是TrueSTUDIO的上位替代。毕竟是官方出品也整合了CubeMX的功能，尝试着使用了一段时间感觉还蛮不错的，就决定一直用下去了。

ST的STM32CubeIDE是基于Eclipse开发的。英飞凌的AURIX Studio和沁恒的MounRiver Studio也都是Eclipse修改来的，界面风格比较相似直接上手也不会有很割裂的感觉。Eclipse方案的优势相对而言是开源免费，并且跨平台方便。劣势是JAVA类软件的通病，资源需求比较大。

像Embedded Studio这样的新型IDE还没来得及去尝试。网页版的Keil Studio颜值上来了但还没有离线版如果作为长期使用的IDE现阶段多少有些不放心。另外微软官方也下场制作了Visual Studio 2022和VSCode的方案。相信之后选择会越来越多样的。



22.03.13 好家伙MDK推出了个人免费使用版本，只要不是用于商用随便用。登录领license。 <https://www.keil.arm.com/mdk-community/>

另外发现MDK和Keil Studio现在也支持相互导入了。



# 快捷键

在`Window > Preferences > General > Keys`可以修改快捷键设置。`Help > Show Active Keybindings...`

可以查看当前可用快捷键。

只列出了常用的几个。另外最好按习惯换一下两个搜索功能的默认快捷键。

| 快捷键       | 说明                               |
| ------------ | ---------------------------------- |
| Ctrl+Z       | 撤销                               |
| Ctrl+Y       | 重做(撤回)                         |
| Ctrl+Shift+Z | 重做(撤回)                         |
| Alt+/        | 代码补全                           |
| Ctrl+F       | 当前文件内搜索（Find and Replace） |
| Ctrl+H       | 项目内搜索（Open Search Dialog）   |
| F3           | 跳转到声明处                       |
| F11          | 启动调试                           |



# 更好的SWV调试支持

SW调试又称SWD，还有一种不太常见但是功能更丰富的SWV调试。功能上来说SWV是SWD的扩展，最基本的操作和SWD是一样的，只需要额外的SWO引脚。

再CubeMX中`SYS > Debug `选择`Trace Asynchronous Sw`即可启用。另外需要注意下SWO引脚是否会与已配置的引脚冲突。

CubeIDE对SWV仿真的支持比Keil要好些。


SWV支持的扩展功能有很多。向控制台输出信息；变量追踪和绘制；函数耗时占比统计；异常与中断追踪；SWV追踪日志；RTOS实时状态查看

这篇文章还蛮不错的 博客配图不方便我就不再写了（逃 <https://blog.csdn.net/Naisu_kun/article/details/97393547>

另外说一下，正版的ST-LINK V2是不支持给开发板供电的。T_VCC是官方ST-LINK特有的引脚，需要将MCU电源连接至该引脚，而没有供电功能。反而是长得像U盘一样的盗版下载器可以直接供电。



# 两种编译配置：Release和Debug

CubeIDE在编译是会有两个可选项，Release和Debug，这两个其实就是是默认的两种不同的编译配置。这个功能在理解了官方的用意之后会发现其实是非常好用的。

比如我想在测试的时候使用-O0优化，而在正式发布程序的时候希望以-O3优化来运行。这样可以在调试时有较好的体验，而正式发布时也有些许的速度提速。

或者是我要在调试的时候多收集信息和数据，而在正式发布时想要精简体积。这样只需要给这两个配置不同的全局宏定义。比如配置宏定义Test_Mode在Debug配置下为0在Release下为1，并使用预编译`#if Test_Mode`的方式即可完成这个场景。



# 全局宏定义&添加引用新文件

- 全局宏定义：右键项目选择 `Prorerties` ，在弹出窗口的 `C/C++ General > Paths and Symbols > Symbols` 中即可以添加全局宏定义

- 添加引用自己的文件：右键项目选择 `Prorerties` ，弹出窗口 `C/C++ General > Paths and Symbols` 中的 `Includes` 、 `Source Location` 分页分别为.h和.c文件的对应路径。直接添加即可。

  工程名称可以使用`${ProjName}`代替

- 注意有两种编译配置，依情况可能都需要修改

  

# 启用assert_param函数

assert_param函数用来检查用户输入参数是否正确。HAL库中的函数大多内部都有assert_param函数。默认时assert_param没有启用。

开启的方式可以通过修改CubeMX设置（不推荐）：勾选`Project Manager > Code Generator > HAL Settings > Enable Full Assert`

更好的方式是只在Debug编译配置时启用。assert_param功能对性能和固件体积影响比较大，所以在正式发布版的程序中建议关闭该功能。

方法就是在项目Debug编译配置中的全局宏定义，添加 `USE_FULL_ASSERT= 1U`



# 字体相关

- 显示简体中文：右键项目选择`Prorerties` ，然后在弹出的窗口选择`Resource`，修改`Text file encoding`为`Other：GBK`。GBK不在下拉列表的选项中的话直接打字上去也是可以的。
- 更换字体：`Window > Preferences > General > Apperance > Colors and Fonts`选择`Bsaic > Text Fonts`再点击右侧Edit...即可更换。我在使用的字体是更纱黑体Sarasa Mono HC 12
- 修复中文字体大小异常：再上面更换字体的窗口，将脚注选择为“中欧字符”即可



# 自动补全

CubeIDE本身不带有自动补全功能，需要 Alt+/  的快捷键才能显示代码补全框。但毕竟基于Eclipse也是有插件能解决的。自动补全也是我体感CubeIDE优于Keil的地方。具体的方法直接在搜索引擎搜索”STM32CubeIDE 自动补全“就能找到解决教程。注意不同的IDE版本可能有些许差别。



# 杂项

不太好归类但是也想要说一下。

- 编译生成 bin / hex 文件

  默认情况CubeIDE编译生成的是elf格式的固件。在IDE左侧项目资源管理器（Project Explorer）中选择项目，然后右键项目选择`Prorerties` ，在弹出窗口的 `C/C++ Build > Settings > Tool Settings > MCU Postbuild outputs` 中即可选中生成bin和hex文件。

  推荐在上方`Configuration`中选择Release。即只在选择Release编译配置时才生成hex，在选择Debug编译配置时不生成，以此提高编译速度

- CubeIDE不允许同一个工作空间中出现同名项目

- 编译优化等级设置

  右键项目选择 `Prorerties` ，在弹出窗口的 `C/C++ Build > Settings > Tool Settings > MCU xxx Compiler > Optimization`下即可修改优化等级

- 修改ROM起始地址

  修改`STM32F407IGHX_FLASH.ld`中`FLASH    (rx)    : ORIGIN = 0x8000000` 

- 开启串口重定向printf的浮点数支持

  默认下printf只能输出整形。右键项目选择`Prorerties` ，在弹出窗口的 `C/C++ Build > Settings > Tool Settings > MCU Settings` ，勾选`Use float with printf from newlib-nano (-u_printf float)` 

- 另外可以看下稚晖军写的配置CLion用于STM32开发 <https://zhuanlan.zhihu.com/p/145801160>



------

参考链接：

> [使用ST-LINK调试程序](https://blog.csdn.net/Naisu_kun/article/details/97393547)
