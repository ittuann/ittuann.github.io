---
layout: article
title: 能量机关其他部分的配置
date: 2022-05-14
key: P2022-05-14
tags: ["STM32", "RoboMaster"]
show_author_profile: true
comment: true
sharing: true
aside:
  toc: true
---

ADC+DMA采集。RNG硬件随机数发生器。ADC的Vrefint Channel内部参照电压。FreeRTOS的Delay_us函数。这是能量机关其他部分的配置。

<!--more-->

# ADC+DMA采集

## 转换时间

ADC的工作流程为采样 - 比较 - 转换。采样是指对某一时刻的模拟电压进行采集，比较是指将采样的电压在比较电路中进行比较，转换是指将比较电路中结果转换成数字量

- ADC时钟

​ ADC输入时钟ADC_CLK，由PCLK2经过分频产生，最大值是36MHz。

- 采样周期

​ ADC需要若干个ADC_CLK周期完成对输入的电压进行采样。周期就是1/ADC_CLK。ADC最小需要3个周期，因为大幅击打不会有很高的频率，所以设置为480 Cycles也可以。

- 总转换时间

​ ADC的总转换时间 = （设置的采样周期数 + 12个周期）x ADC_CLK。加12个周期是因为转换精度设置为12Bits

## ADC Parameter Settings

- DMA Settings开启ADC。

  - 配置下DMA模式为`Circular`，既循环更新数据。默认的Normal模式触发后只执行 一次。
  - 设置方向Direction为从外设到内存。
  - 配置自增地址为Memory方式，因为我程序里定义uint16_t 的数组来存储多路ADC数据，占两个字节所以选择`half word`

- Scan Conversion Mode 设置为 `ENABLE`。

  多通道AD转换配置为使用扫描

- Continuous Conversion Mode 设置为 `ENABLE`。

  使能自动连续转换。DISABLE为单次转换，转换一次后停止需要代码手动控制才重新启动转换。

- Discontinuous Conversion Mode 默认为 `DISABLE`。

  关闭非连续转换模式，默认为DISABLE即可

- DMA Continuous Requests 设置为 `ENABLE`。

  每次转换完成后产生EOC的同时，也产生DMA请求

  指定DMA以连续模式执行。（必须开启DMA才有效，且设置为ENABLE时必须配置DMA为circular模式）

- End of Conversion Selection

  每个通道转换结束后发送 EOC 标志／所有通道转换结束后发送EOC标志

- Number Of Conversion 设置为 `6` 。

  配置几路通道采集及采集顺序。大幅一共5个扇叶加上一个 Vrefint Channel。下面的 RANK1-6 即可配置采集顺序和采样周期。

## Vrefint Channel 内部参照电压

VREFINT是STM32的内部参照电压。一般来说STM32的ADC采用Vcc作为Vref，但为了防止Vcc存在波动较大导致Vref不稳定，进而导致采样值的比较结果不准确，STM32可以通过内部已有的参照电压VREFINT来进行校准，接着以VREFINT为参照来比较ADC的采样值，从而获得比较高的精度。VREFINT的电压为1.2V。Vrefint Channel在STM32内部完成没有对应引脚，只能出现在主ADC1中使用。

```c
volatile fp32 voltage_vrefint_proportion = 8.0586080586080586080586080586081e-4f; // 初始化

adc_calibration = adc_original * (1.20f / voltage_vrefint_proportion);
```

## 程序

```c
uint16_t ADC_ConvertedValue[6] = {0};
HAL_ADC_Start_DMA(&hadc1, (uint32_t *)&ADC_ConvertedValue, 6);
```

即可开启DMA传输。

ADC的DMA中断函数为`DMA2_Stream0_IRQHandler`。可以直接在DMA中断函数内加入判断是否大幅被击打的程序部分。

最终实现的方法可以用很多种，比如用外部触发信号像是定时器之类，这里就是展示了其中一种的配置思路和方法。

## 不使用DMA的ADC单次采集

```c
/**
  * @brief			获取ADC采样值
  * @retval			none
  * @example		adcx_get_chx_value(&hadc1, ADC_CHANNEL_1);
  */
uint16_t adcx_get_chx_value(ADC_HandleTypeDef *ADCx, uint32_t ch)
{
	static ADC_ChannelConfTypeDef sConfig = {0};
	sConfig.Channel = ch;							// ADC转换通道
	sConfig.Rank = 1;								// ADC序列排序 即转换顺序
	sConfig.SamplingTime = ADC_SAMPLETIME_15CYCLES;	// ADC采样时间

	HAL_ADC_ConfigChannel(ADCx, &sConfig);			// 设置ADC通道的各个属性值
//	if (HAL_ADC_ConfigChannel(ADCx, &sConfig) != HAL_OK) {
//		Error_Handler();							// Debug时使用
//	}

	HAL_ADC_Start(ADCx);							// 开启ADC采样

	HAL_ADC_PollForConversion(ADCx, 1);				// 等待ADC转换结束

	if (HAL_IS_BIT_SET(HAL_ADC_GetState(ADCx), HAL_ADC_STATE_REG_EOC)) {	// 判断转换完成标志位是否设置
		return (uint16_t)(HAL_ADC_GetValue(ADCx));	// 获取ADC值
	} else {
		return 1;									// 发现数值为1代表错误
	}
}
```

# RNG硬件随机数发生器

STM32F4自带了RBG硬件随机数发生器，RNG处理器是一个以连续模拟噪声为基础的随机数发生器，在主机读数时提供一个 32 位的随机数。

在CubeMX的Security使能RNG之后，还需要配置下时钟树的RNG专用时钟 (PLL48CLK) 。

生成的 `MX_RNG_Init();` 已经完成了所需的初始化。初始化的过程很简单，首先使能挂载到AHB2总线的随机数发生器时钟，然后使能随机数发生器即可完成初始化。

```c
uint32_t HAL_RNG_GetRandomNumber(RNG_HandleTypeDef *hrng);
```

该函数用于读取随机数值。

每次读取之前，须先判断RNG_SR这个RNG 状态寄存器的DRDY最低位，如果该位为1则则说明RNG_DR这个数据寄存器的数据是有效的，可以读取得到随机数值，读取后该位自动清零；如果不为 1则需要等待。该函数已包含判断DRDY位并返回读取到的随机数值。

```c
#include "stm32f4xx_hal_rng.h"
/**
  * @brief          生成0-5随机数 用于确定大幅的亮灯顺序
  */
void RandGet(void)
{
	uint8_t i = 0, j = 0, tmp = 0;
    for(i = 0; i < 5; i ++) {
        while (1) {
        	tmp = HAL_RNG_GetRandomNumber(&hrng) % 5;
            for(j = 0; j < i; j ++) {
                if(tmp == Rand[j]) {
					break;
				}
			}
            if(j == i) {
                Rand[i] = tmp;
                break;
            }
        }
    }
}

// 如果是F1的板子没有RNG，可以换成伪随机数，用当前时间作为种子生成随机数
#include <stdlib.h>
// 时间作为种子
srand((unsigned int)HAL_GetTick());
tmp = rand() % 5;
HAL_Delay(1);
```

# Delay_us

用FreeRTOS时，TimeBaseSource为SysTick以外的其他定时器，对应的us延时也需要改动。

```c
/**
  * @brief			us延时
  * @param[in]		us
  */
#define TimebaseSource_is_SysTick 1
#if	(!TimebaseSource_is_SysTick)
	extern TIM_HandleTypeDef htim9;		// 当使用FreeRTOS, TimebaseSource为其他定时器时
	#define Timebase_htim htim9
	#define Delay_GetCurrentValueReg()	__HAL_TIM_GetCounter(&Timebase_htim)
	#define Delay_GetReloadReg()		__HAL_TIM_GetAutoreload(&Timebase_htim)
#else
	#define Delay_GetCurrentValueReg()	(SysTick->VAL)
	#define Delay_GetReloadReg()		(SysTick->LOAD)
#endif

static uint32_t fac_us = 0;
static uint32_t fac_ms = 0;

void Delay_us_init(void)
{
	#if	(!TimebaseSource_is_SysTick)
		fac_ms = 1000000;				// 作为时基的计数器时钟频率在HAL_InitTick()中被设为了1MHz
		fac_us = fac_ms / 1000;
	#else
		fac_ms = SystemCoreClock / 1000;
		fac_us = fac_ms / 1000;
	#endif
}

void Delay_us(uint16_t us)
{
	/*** 定时器功能实现 ***/
	uint32_t ticks = us * fac_us;
	uint32_t reload = Delay_GetReloadReg();
    uint32_t told = Delay_GetCurrentValueReg();
    uint32_t tnow = 0;
    uint32_t tcnt = 0;
    while (1) {
        tnow = Delay_GetCurrentValueReg();
        if (tnow != told) {
            if (tnow < told) {
                tcnt += told - tnow;
            } else {
                tcnt += reload - tnow + told;
            }
            told = tnow;
            if (tcnt >= ticks) {
                break;
            }
        }
    }

    /*** NOP空语句实现 ***/
//	uint32_t delay = us * fac_us / 4;
//	do {
//		__NOP();
//	}
//	while (delay --);
}
```
