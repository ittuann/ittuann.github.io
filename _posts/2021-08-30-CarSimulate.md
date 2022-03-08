---
layout: article
title: 智能车电机系统辨识和PID仿真自动调参
date: 2021-08-29
tags: ["智能车","PID"]
comment: true
aside:
  toc: true
---

使用 Matlab 的 System Identification 系统辨识工具箱，辨识出传递函数；用 Simulink 仿真控制系统；PID Tuner 自动整定 PID 参数

<!--more-->

步骤：系统辨识传递函数 -> Simulink仿真 -> PID Tuner自动调参

# 系统辨识传递函数

## 采集数据

这里选择根据输入和输出数据辨识传递函数。具体到智能车电机的话，就是在电机开环的情况下，给定电机恒定的占空比，记录电机速度（编码器返回值）从0到接近稳定的曲线。可以用无线串口配合匿名上位机完成原始数据的采集。

## 导入数据

### 导入数据到 Matlab

我使用的环境是 Matlab R2020b。在顶部选择导入数据，打开匿名上位机保存出来的CSV文件，选中速度的那列数据并在上方将输出类型更改为数值矩阵，最后点击右侧将电机转速数据导入工作区。

还需要输入数据，也就是给电机的pwm定值。所以只需要创建一个与输出数据元素个数相同的列向量即可。 `pwm=linespace(3000,3000,1200)` ` 注意需要在最后加上 ' 将行向量转为列向量。

### 导入数据到系统辨识工具箱

之后在 App 中选择 System Identification

在左侧import data内选择Time domain data导入时间域数据。导入的数据可以包括时域的也可以包括频域的。

Workspace Variable里的input为输入数据，填写matlab工作区内的输入数据变量名即pwm；Output即需要填入输出转速变量名

Data Information里的Starting time为起始时间设为0即可；Sample time为采样时间单位是s，根据自己单片机程序里设置的编码器的采样时间输入。注意编码器的采样频率应该和串口接收数据（也就是matlab工作区里的电机转速数据）的频率相同，不过如果丢包率很低也可以不用太在意。

勾选time plot会显示数据，可以检查数据是否正确

## 系统辨识

接下来将左侧的mydata曲线拖动到Working Data中。要对数据进行操作时，都要把数据拖到working data中才能生效。

正常需要对电机转速的数据进行预处理。但因为我在单片机串口发送的数据就已经是一阶低通后的数据，所以可以不用再处理这部分。

在Estimate选择Transfer Function Models，然后设置合适的零点极点。电机模型一般零点为1极点为2。

I/O Delay是延时时间不用管默认0即可。也可以取消勾选Time delay中的Fixed，这样就可以自动辨识时间延迟。

Estimate等待一小段时间的计算后即会计算出结果。拟合率一般80%为经验可用的界限。此时右侧的模型窗口出现了辨识的tf1，把tf1拖拽到To Workspace即可导出到工作区，系统辨识完成。

# Simulink 仿真

按图连接节点，名称都已标出

![Simulink1](https://raw.githubusercontent.com/ittuann/ittuann.github.io/main/_posts/_img/2021-08-29-CarPID1.png)

双击Transfer Fcn填入刚刚辨识完的传递函数。具体数据双击工作区的tf1直接复制tf1.Denominator和tf1.Numerator即可

# PID Tuner 自动调参

双击PID Tuner再点击右下脚Tune即可开始

调节上方两个按钮，一个是响应速度（单位秒），另一个是选择超调量。

视线为当前参数仿真效果，虚线为调整前的参数，方便对比

Show Parameters为当前参数。注意PID Tuner计算的积分项I需要乘上系统的采样时间（单位秒）才是常规位置式PID的参数





> 参考资料：
>
> PID Controller https://ww2.mathworks.cn/help/simulink/slref/pidcontroller.html
>
> System Identification 官方帮助文档 https://ww2.mathworks.cn/help/ident/index.html
>
> System Identification Toolbox https://ww2.mathworks.cn/products/sysid.html
