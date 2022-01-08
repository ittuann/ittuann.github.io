---
layout: article
title: CAN通信配置过滤器和三个发送邮箱
date: 2021-08-28
tags: STM32, RoboMaster
comment: true
aside:
  toc: true
---

RM比赛用的电机基本都使用CAN通信，但是一条CAN线上只用一个发送邮箱在挂在设备多的情况可能会导致发送不完，但其实完全可以把三个发送邮箱都用上。

```c
/**
  * @brief          CAN筛选器
  */
void Can_Filter_Init(void)
{
	/***	CAN1	***/
    CAN_FilterTypeDef sFilterConfig;
	sFilterConfig.FilterActivation = ENABLE;			// 激活过滤器
	sFilterConfig.FilterBank = 0;						// 设置主CAN过滤器组编号
	sFilterConfig.FilterMode = CAN_FILTERMODE_IDMASK;	// 设为列表模式
	sFilterConfig.FilterScale = CAN_FILTERSCALE_16BIT;	// 配置为16位宽
	sFilterConfig.FilterIdHigh = 0x0000;				// CAN_FxR1寄存器高16位
	sFilterConfig.FilterIdLow = 0x0000;
	sFilterConfig.FilterMaskIdHigh = 0x0000;			// CAN_FxR2寄存器高16位
	sFilterConfig.FilterMaskIdLow = 0x0000;
	sFilterConfig.FilterFIFOAssignment = CAN_RX_FIFO0;	// 接收到的报文放入到FIFO0中
	
	if (HAL_CAN_ConfigFilter(&hcan1, &sFilterConfig) != HAL_OK) {						// 配置CAN1接收筛选过滤器
		Error_Handler();
	}
	if (HAL_CAN_Start(&hcan1) != HAL_OK) {												// 开启CAN1
		Error_Handler();
	}
	if (HAL_CAN_ActivateNotification(&hcan1, CAN_IT_RX_FIFO0_MSG_PENDING) != HAL_OK) {	// 开启CAN1的FIFO0接收中断
		Error_Handler();
	}
}
```

```c
#include <stdbool.h>

/**
  * @brief          发送C620电调控制电流
  * @param[in]		motorID对应的电机控制电流, 范围 [-16384, 16384]，对应电调输出的转矩电流范围 [-20A, 20A]
  * @param[in]      etcID: 控制报文标识符(电调ID)为1-4还是5-8
  */
void CAN_Cmd_C620(CAN_HandleTypeDef *hcan, int16_t motor1, int16_t motor2, int16_t motor3, int16_t motor4, bool_t etcID)
{
	CAN_TxHeaderTypeDef TxHeader;						// CAN通信协议头
	uint8_t TxData[8] = {0};							// 发送电机指令缓存
	uint32_t TxMailboxX = CAN_TX_MAILBOX0;				// CAN发送邮箱

	if (etcID == false) {
		TxHeader.StdId = CAN_3508_ALL_ID;				// 标准格式标识符ID
	} else {
		TxHeader.StdId = CAN_3508_ETC_ID;
	}
	TxHeader.ExtId = 0;
	TxHeader.IDE = CAN_ID_STD;							// 标准帧
	TxHeader.RTR = CAN_RTR_DATA;						// 传送帧类型为数据帧
	TxHeader.DLC = 0x08;								// 数据长度码
	TxData[0] = (uint8_t)(motor1 >> 8);
	TxData[1] = (uint8_t)motor1;
	TxData[2] = (uint8_t)(motor2 >> 8);
	TxData[3] = (uint8_t)motor2;
	TxData[4] = (uint8_t)(motor3 >> 8);
	TxData[5] = (uint8_t)motor3;
	TxData[6] = (uint8_t)(motor4 >> 8);
	TxData[7] = (uint8_t)motor4;

	//找到空的发送邮箱 把数据发送出去
	while (HAL_CAN_GetTxMailboxesFreeLevel(hcan) == 0);	// 如果三个发送邮箱都阻塞了就等待直到其中某个邮箱空闲
	if ((hcan->Instance->TSR & CAN_TSR_TME0) != RESET) {
		// 检查发送邮箱0状态 如果邮箱0空闲就将待发送数据放入FIFO0
		TxMailboxX = CAN_TX_MAILBOX0;
	} else if ((hcan->Instance->TSR & CAN_TSR_TME1) != RESET) {
		TxMailboxX = CAN_TX_MAILBOX1;
	} else if ((hcan->Instance->TSR & CAN_TSR_TME2) != RESET) {
		TxMailboxX = CAN_TX_MAILBOX2;
	}
	// 将数据通过CAN总线发送
	#if DEBUGMODE
		if (HAL_CAN_AddTxMessage(hcan, &TxHeader, TxData, (uint32_t *)TxMailboxX) != HAL_OK) {
			Error_Handler();
		}
	#else
		HAL_CAN_AddTxMessage(hcan, &TxHeader, TxData, (uint32_t *)TxMailboxX);
	#endif
}
```

可以加上判断状态是否为HAL_OK，如果CAN信息发送失败则进入Error_Handler()死循环。测试时可以打断点判断问题所在，但正式版本时不想给运行着的程序增加可能卡死循环的风险所以会关掉判断。while循环等待空邮箱也可以关掉。

```c
/**
  * @brief			HAL库CAN FIFO0接受邮箱中断（Rx0）回调函数
  * @param			hcan : CAN句柄指针
  */
void HAL_CAN_RxFifo0MsgPendingCallback(CAN_HandleTypeDef *hcan)
{
	CAN_RxHeaderTypeDef RxHeader;						// CAN通信协议头
	uint8_t rx_data[8] = {0};							// 暂存CAN接收数据
	motor_measure_t motorDataTmp;						// 电机数据
	uint8_t i = 0;
	
	if (hcan == &hcan1)
	{
		if (HAL_CAN_GetRxMessage(hcan, CAN_RX_FIFO0, &RxHeader, rx_data) == HAL_OK)	// 接收CAN总线上发送来的数据
		{
			// 对应电机向总线上发送的反馈的标识符和程序电机序号
			switch (RxHeader.StdId) {
				case CAN_PITCH_MOTOR_ID : i = MotorID_GimbalPitch; break;
				case CAN_YAW_MOTOR_ID : i = MotorID_GimbalYaw; break;
				#if DEBUGMODE
					default : i = RxHeader.StdId - CAN_3508_M1_ID; break;
				#endif
			}
			// 电机返回数据协议解析
			get_motor_measure(&motorDataTmp, rx_data);
			// 向消息队列中填充数据
			if (messageQueueCreateFlag) {
				xQueueOverwriteFromISR(messageQueue[i], (void *)&motorDataTmp, pdFALSE);
			}
		}
	}
}
```

完整的工程可以看我开源的飞机云台程序~
