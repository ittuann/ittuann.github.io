---
layout: article
title: 智能车摄像头基本循迹控制
date: 2021-08-28
key: P2021-08-28-5
tags: 智能车
comment: true
sharing: true
aside:
  toc: true
---

基于摄像头的三种基本巡线控制

<!--more-->

## 摄像头的三种基本巡线：

摄像头程序处理后会给出一条中线，使用计算中线与标准值误差，赋予转向环PID。

### 三行巡线：

类似于线性CCD的控制方案，是最早期使用的方法。不过使用的有效信息太少，中线一旦跳变整个车就会抖动，车速到了一米一左右尤为明显。

aim_line取决于速度和有效行，速度越快aim_line预瞄距离越远，同时限幅在有效行之下。

```c
AngleErr = ((   5 * middleLine[aimLine] +
                3 * middleLine[aimLine + 1] +
                2 * middleLine[aimLine + 2]) / (10.000f)) - middleStandard;
```

### 加权计算全图像：

使用了整个图像，对于图像不同区域（行数）给与不同权重。为了防止跳变取近三次输出值加权计算误差。

对于不同元素可以赋予不同权重，如坡道weight3权重整体关注区域更偏向上方，环岛weight2权重关注区域为补线区域，三叉内低速横向循迹weight4权重关注区域更偏下方。

start_row 为补线起始行，valid_row为有效行最顶行，作用相当于限幅。

```c
uint8   weight1[60] = {                        //0为图像最顶行
         0, 0, 0, 0, 0, 0, 0, 0, 0, 0,
         1, 1, 3, 3, 5, 5,10,10,10,10,
        10,10,10,10,10,10,10,10,10,10,
        10,10,10,10,10,10,10,10,10,10,
        10,10,10,10,10, 5, 5, 5, 5, 5,
         3, 3, 3, 1, 1, 1, 0, 0, 0, 0,};    //基础    //注意斜率变化引起的跳变,要平滑

for(i = startRow; i > validRow + 1; i --) {
    weightSum += weight1[i];
    lineSum += weight1[i] * middleLine[i];
}   //从下往上扫

angleErr = (float)lineSum / weightSum - middleStandard;
midline_fff = midline_ff;
midline_ff  = midline_f;
midline_f = angleErr;
angleErr = midline_fff * 0.50f + midline_ff * 0.30f + midline_f * 0.20f;
```

在这些的基础上增加了最小二乘法对摄像头中线的再拟合，作用也是为了更为了使循迹平滑。

```   c
slope = RegressionCal(validRow + 3, startRow); 
angleErr = angleErr * 0.80f + slope * 0.20f;
```

### 用部分图像计算偏差：

```c
memset(partErr, 0, sizeof(partErr));
improperRowCount = 0;
memset(improperRow, 0, sizeof(improperRow));

if (startPart > startRow)    startPart = startRow - 2;    //起始行限幅
endPart = startPart - partUsed;
if (endPart < validRow)    endPart = validRow + 2;    //截至行限幅

for (i = startPart; i > endPart; i--) {    //从下往上扫
    partErr[startPart - i] = (float)(middleLine[i] - middleStandard) * 100.0f / ((float)imgRealWidth[i] / 2.0f);
    partAve += middleLine[i];
//        if (i == endPart - 2) {
//            Part_Err[startPart - i] = partErr[startPart - i] * 1.20f;
//        }   //放大截至行误差
}    //某种意义上相当于逆透视
    //partUsed 相当于前瞻,区域越小越灵敏 10行适合低速,高速可以取15

partAve = partAve / partUsed;            //计算范围内中线平均值
for (i = startPart; i > endPart; i--) {
    if (ABS(middleLine[i] - partAve) > 10) {
        improperRow[startPart - i] = middleLine[i];
        partUsed--;
        improperRowCount++;
    }
}   //记录偏离中线均值10以上的点 滤掉跳变点

for (i = 0; i < (partUsed / 5); i++) {
    partErr[i] = 0.000f;
}   //最底下1/5区域置零, 为了平滑

for (i = 0; i < partUsed; i++) {
    if (improperRow[i] == 0) {
        partErrSum += partErr[i];
    }
}   //累加非跳变点

angleErr  = partErrSum * errK / (float)partUsed;
angleErr_fff = angleErr_ff;
angleErr_ff = angleErr_f;
angleErr_f = angleErr;
angleErr = angleErr0 * 0.70f + angleErr1 * 0.20f + angleErr2 * 0.10f;
```

