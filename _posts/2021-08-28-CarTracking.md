---
layout: article
title: 智能车摄像头循迹控制
date: 2021-08-28
tags: 智能车
aside:
  toc: true
---

## 摄像头巡线：

摄像头程序处理后会给出一条中线，使用计算中线与标准值误差，赋予转向环PID。

### 三行巡线：

类似于线性CCD的控制方案，是最早期使用的方法。不过使用的有效信息太少，中线一旦跳变整个车就会抖动，车速到了一米一左右尤为明显。

aim_line取决于速度和有效行，速度越快aim_line预瞄距离越远，同时限幅在有效行之下。

```c
AngleErr = ((	5 * middle_line[aim_line] +
				3 * middle_line[aim_line + 1] +
				2 * middle_line[aim_line + 2]) / (10.000f)) - middle_standard;
```

### 加权计算全图像：

使用了整个图像，对于图像不同区域（行数）给与不同权重。为了防止跳变取近三次输出值加权计算误差。

对于不同元素可以赋予不同权重，如坡道weight3权重整体关注区域更偏向上方，环岛weight2权重关注区域为补线区域，三叉内低速横向循迹weight4权重关注区域更偏下方。

start_row 为补线起始行，valid_row为有效行最顶行，作用相当于限幅。

```c
uint8   weight1[60] = {						//0为图像最顶行
         0, 0, 0, 0, 0, 0, 0, 0, 0, 0,
         1, 1, 3, 3, 5, 5,10,10,10,10,
        10,10,10,10,10,10,10,10,10,10,
        10,10,10,10,10,10,10,10,10,10,
        10,10,10,10,10, 5, 5, 5, 5, 5,
         3, 3, 3, 1, 1, 1, 0, 0, 0, 0,};    //基础    //注意斜率变化引起的跳变,要平滑

for(i = start_row; i > valid_row + 1; i --) {
    weight_sum += weight1[i];
    line_sum += weight1[i] * middle_line[i];
}   //从下往上扫

midlinenow  = (float)line_sum / weight_sum - middle_standard;
midline_fff = midline_ff;
midline_ff  = midline_f;
midline_f   = midlinenow;
AngleErr = midline_fff * 0.50f + midline_ff * 0.30f + midline_f * 0.20f;
```

在这些的基础上增加了最小二乘法对摄像头中线的再拟合，作用也是为了更为了使循迹平滑。

```   c
Slope = Regression_cal(valid_row + 3, start_row); 
AngleErr = AngleErr * 0.80f + Slope * 0.20f;
```

### 用部分图像计算偏差：

```c
memset(Part_Err, 0, sizeof(Part_Err));
Improper_Row_Count = 0;
memset(Improper_Row, 0, sizeof(Improper_Row));

if (Start_Part > start_row)	Start_Part = start_row - 2;	//起始行限幅
End_Part = Start_Part - PartUsed;
if (End_Part < valid_row)	End_Part = valid_row + 2;	//截至行限幅

for (i = Start_Part; i > End_Part; i--) {	//从下往上扫
    Part_Err[Start_Part - i] = (float)(middle_line[i] - middle_standard) * 100.0f / ((float)img_real_width[i] / 2.0f);
    Part_Ave += middle_line[i];
//        if (i == End_Part - 2) {
//            Part_Err[Start_Part - i] = Part_Err[Start_Part - i] * 1.20f;
//        }   //放大截至行误差
}	//相当于逆透视
	//PartUsed 相当于前瞻, 10行适合低速, 高速可以取15, 区域越小越灵敏

Part_Ave = Part_Ave / Part_Used;			//计算范围内中线平均值
for (i = Start_Part; i > End_Part; i--) {
    if (ABS(middle_line[i] - Part_Ave) > 10) {
        Improper_Row[Start_Part - i] = middle_line[i];
        Part_Used--;
        Improper_Row_Count++;
    }
}   //记录偏离中线均值10以上的点 滤掉跳变点

for (i = 0; i < (Part_Used / 5); i++) {
    Part_Err[i] = 0.000f;
}   //最底下1/5区域置零, 为了平滑

for (i = 0; i < Part_Used; i++) {
    if (Improper_Row[i] == 0) {
        Part_Err_Sum += Part_Err[i];
    }
}   //累加非跳变点

AngleErr  = Part_Err_Sum * Err_K / (float)Part_Used;
AngleErr2 = AngleErr1;
AngleErr1 = AngleErr0;
AngleErr0 = AngleErr;
AngleErr = AngleErr0 * 0.70f + AngleErr1 * 0.20f + AngleErr2 * 0.10f;
```

# 弯道控制：

弯道的速度可以由公式![[公式]](https://www.zhihu.com/equation?tex=m%5Cfrac%7BV%5E%7B2%7D+%7D%7BR%7D+%3Dma%3DF)=![[公式]](https://www.zhihu.com/equation?tex=%5Cmu+mg)（F是摩擦力），这样V=![[公式]](https://www.zhihu.com/equation?tex=%5Csqrt%7B%5Cmu+gR%3Cbr%2F%3E%7D+),其中![[公式]](https://www.zhihu.com/equation?tex=%5Cmu+)是摩擦系数，由地面和轮胎决定，R是转弯半径。由于地面和轮胎在过弯时是给定的，这样在比赛中我们为了保证V大，只能保证更大的转弯半径。R越大，速度V就越大。所以稳定沿着电磁线循迹并不是最优解，最好是采用外内外切弯。即入弯时贴弯道的内弯，出弯时贴外弯。这种情况下赛车通过整个弯道过程中行车线半径是固定的，即定曲率行车线。

<img src="https://raw.githubusercontent.com/ittuann/ittuann.github.io/main/_posts/_img/CarTracking1.png" alt="img" style="zoom: 75%;" />

实践中发现通过调整纯跟踪算法的预瞄距离，可以有效提高路径规划的最优性。对于全向车也是一样，弯道的控制方案最好为，入弯减速避免打滑，出弯加速节约时间。

