---
layout: article
title: 探索FreeRTOS的一些功能和用法
date: 2021-12-25
tags: ["STM32","RoboMaster"]
comment: true
aside:
  toc: true
---

探索FreeRTOS的功能：线程，消息队列，邮箱，信号量，互斥量，任务通知，延时，虚拟定时器

<!--more-->

使用的是由ST公司封装的CMSIS V1的API。开发环境是STM32CUBEIDE V1.7.0，STM32F4 Package 1.26.2。其实我接触FreeRTOS时间不是很长，作为学习笔记在这里记录下一些开发时候用到的功能和用法~

# 线程

操作系统与裸机的最大区别就是线程啦

- 线程的定义创建和初始化均可以在STM32CubeMX中完成。在CUBEMX-FREERTOS-Tasks and Queues来创建。选择Tasks下的Add即可完成创建。

  消息队列Queues在CubeMX里面也可以创建但是灵活度不够高最好按需求手动创建，后文会提到。

- 在Code Generation Option选择As Weak会创建弱函数，可以很方便的用自己在其他位置写的函数来直接覆盖这个弱符号函数。

- 创建后的线程会自动在`freertos.c`的`MX_FREERTOS_Init()`函数完成初始化。

  ```c
  osThreadId testHandle;					// 定义线程的ID，用于对线程的各种操作（如修改优先级，中止/开始线程等
  void TestTask(void const * argument);	// 线程对应的函数体的声明
  
  osThreadDef(test, TestTask, osPriorityBelowNormal, 0, 256);	// 线程定义，参数分别为：线程的名称，线程函数体，线程优先级，线程实例化个数，线程分配的栈空间
  testHandle = osThreadCreate(osThread(test), NULL);			// 创建线程，并赋值给对应的线程ID
  
  // 线程的具体实现
  __weak void TestTask(void const * argument)
  {
    /* USER CODE BEGIN TestTask */
    /* Infinite loop */
    for(;;)
    {
      osDelay(1);
    }
    /* USER CODE END TestTask */
  }
  ```

- CubeMX在配置FreeRTOS时默认使用与HAL库相同的SysTick滴答定时器。为了避免时钟线混乱冲突，需要在System Core-SYS-Timebase Source选择一个其他的定时器。

  STM32的TIM分为高级定时器、通用定时器和基本定时器。其中基本定时器的功能最简单，只有定时的功能，一般用作时钟基源；通用定时器在基本的定时功能的基础上多出了输出比较和输入捕获功能。输出比较可以输出周期性的方波（比如PWM波和PPM波），输入捕获可以读取输入信号的高电平和低电平的时间进而可以计算出信号的周期和占空比；高级定时器除了上述功能之外，还有还包含无互补信号输出以及带刹车(断路)功能等电机控制~~（日常用不太到的）~~高级功能

- 修改Config Parameters-MINIMAL_STACK_SIZE 为256。默认的128堆栈可能会不太够用

- 每一个线程都有四种状态：挂起、阻塞、就绪和运行状态，每一种状态的特点从它的命名就可以猜出来。任务调度器在每一次切换任务的时候都会检查有没有优先级更高的线程处于就绪（ready）状态，如果有，则暂停当前执行的线程，转而执行优先级更高的线程。另外在操作系统中线程的优先级可以是一样的，当两个线程的优先级是一样时，任务调度器会不断在这两个线程间来回切换，近似相当于两个线程同步执行

# 消息队列

线程往往不是相互独立的，需要不同的线程之间进行通信。在FreeRTOS中线程的通信可以使用信号量，互斥量，队列，邮箱，任务通知进行通信。

信号、信号量、互斥量用于进程之间的触发，但对进程间的数据交换无能为力。进程间数据交换最简单的方式是全局变量，但即使在简单的系统中，把握和灵活应用全局变量也是不小的挑战，因为全局变量会引起一系列不可预知错误。

在RTOS中，消息队列和邮箱队列是进程间数据交互最为有效、安全的方式。

消息队列和邮箱队列的工作方式基本一样，唯一的区别是消息队列中传输的是待交换数据，而邮箱队列中传输是指向待交换数据的指针。

队列是具有自己独立权限的内核对象，并不属于任何任务。所有任务都可以向同 一队列写入和读出。

## 消息队列的创建

```c
QueueHandle_t messageQueue[QUEUE_NUM];	// 声明消息队列句柄
bool_t messageQueueCreateFlag = false;	// 消息队列创建完成标志位

enum MessageQueue_e {
	// 0-(MOTOR_NUM-1)给电机
    IMUANGLE = 4,
    QUEUE_NUM
};

/**
  * @brief	消息队列创建
  */
void MessageQueueCreate(void)
{
	uint8_t i = 0;
	// 电机消息队列
	for (i = 0; i < IMUANGLE; i++) {
		messageQueue[i] =  xQueueCreate(1, sizeof(motor_measure_t *));	// 创建FIFO的长度为1 指向motor_measure_t结构的指针的队列
	}
	// IMU消息队列
	messageQueue[IMUANGLE] =  xQueueCreate(1, 3 * sizeof(float));

    // 校验是否创建失败
	for (i = 0; i < QUEUE_NUM; i++) {
		if (messageQueue[i] == NULL) {
			Error_Handler();
		}
	}
	messageQueueCreateFlag = true;
}
```

- 创建队列API函数是`xQueueCreate()`，但其实这是一个宏。真正被执行的函数是`xQueueGenericCreate()`
- 程序中演示了给存放电机数据的结构体指针，以及存放欧拉角的数组angle[3]创建消息队列
- 自己写的这个`MessageQueueCreate()`函数需添加至`MX_FREERTOS_Init`中初始化。
- 建议设置一个消息队列创建成功的标志位。这样可以避免中断在消息队列完成创建前写入数据导致程序卡死。 
- 消息队列创建失败可能是由于 heap 堆栈空间不够

## 消息发送

- `xQueueSend `

  向队列尾部发送一个队列消息。等价于`xQueueSendToBack()`该函数不能在中断服务程序里面被调用，中断中必须使用带有中断安全保护功能的函数`xQueueSendFromISR`

  所有的xQueueSend都是一个宏，实际执行函数为`xQueueGenericSend()`。后缀为`FromISR`的实际执行函数为`xQueueGenericSendFromISR()`

  ```c
  BaseType_t xQueueGenericSend ( 
      QueueHandle_t xQueue,				// 队列句柄
      const void * const pvItemToQueue,	// 指针，指向要入队的项目
      TickType_t xTicksToWait,			// 如果队列满，等待队列空闲的最大时间
      const BaseType_t xCopyPosition		// 入队位置。可以选择从队列尾入队，从队列首入队和覆盖式入队
  )
  ```

- `xQueueSendFromISR`

  `xQueueSend()`的中断保护版本，用于在中断服务程序中向队列尾部发送一个队列消息，等价于 `xQueueSendToBackFromISR()`

- `xQueueSendToFront`

  向队列队首发送一个消息。中断版本为 `xQueueSendToFrontFromISR () `

- `xQueueOverwrite`

  向队列尾部发送一个队列消息。中断版本为`xQueueSendOverwriteFromISR () `
  
- `uxQueueMessagesWaiting`

  返回队列中当前有效数据单元个数。中断版本为`uxQueueMessagesWaitingFromISR()`

- `xQueueIsQueueEmptyFromISR` & `xQueueIsQueueFullFromISR`

  查询队列是否为空/满，只能在中断中使用。返回`pdFALSE`或`pdTRUE`

- 遇到问题可以参考FreeRTOS官方API文档 https://www.freertos.org/a00018.html

## 消息接收

接收 API 和发送 API 差不多， 也是实现了几个宏， 但是实际实现的函数是`xQueueGenericReceive`和`xQueueGenericReceiveFromISR`这两个。

- `xQueueReceive`

  用于从一个队列中接收消息并把消息从队列中删除。读取后会把消息从队列中删除。同样在中断服务程序里面须使用带有中断保护功能的`xQueueReceiveFromISR() `来代替。

- `xQueueReceiveFromISR`

  `xQueueReceive ()` 的中断版本。

- `xQueuePeek`

  从队列首接收到数据后，并不从队列中删出接收到的单元，不会修改队列中的数据，也不会改变数据在队列中的存储序顺。中断版本为`xQueuePeekFromISR ()`

```c
// 某中断回调函数
void ISR（void）
{
    static BaseType_t xHigherPriorityTaskWoken = pdFALSE;	// 不请求上下文切换
    // 中断处理...
	xQueueOverwriteFromISR(messageQueue, (void *)&data, &xHigherPriorityTaskWoken);
	portYIELD_FROM_ISR(xHigherPriorityTaskWoken);			// 判断是否请求上下文切换
}
```

不是很建议在中断中接收消息队列，中断执行往往越快越好。

```c
// 任务中向消息队列复写数据
xQueueOverwrite(messageQueue, (void *)&data);
// 任务中取出消息队列数据 等待时间为1ms
xQueueReceive(messageQueue, &useData, (1 / portTICK_RATE_MS));
```

在上一篇[CAN筛选器和接收发送](https://ittuann.github.io/2021/08/28/CANFilter.html)的文章里也有代码样例。

# 延时

- `osDelay()`为CMSIS-RTOS层。内部其实使用vTaskDelay来实现。在程序执行到这条语句后，当前任务阻塞（不是挂起），任务调度器转而判断其他哪个线程得以执行，当时间到了之后线程变为就绪状态，等待任务调度器调用。

  相比于HAL_Delay，HAL会一直不停的调用获取系统时间的函数，直到指定的时间流逝然后退出，故其占用了全部CPU时间。

- `vTaskDelay()`为相对延时。任务每次延时都是从调用延时函数vTaskDelay()开始算起的，延时是相对于这一时刻开始的，所以叫做相对延时函数。如果执行任务的过程中发生中断，那么任务A执行的周期就会变长，周期也会改变，延时效果不是很精确。

- `vTaskDelayUntil()`为绝对延时。绝对延时能够提供精度更高的定时效果。

  延时的时间单位为系统节拍时钟周期。如果1节拍不是1ms或是想要规范标准化代码可以使用`5 / portTICK_RATE_MS`或`pdMS_TO_TICKS(5UL)`

```c
void TestTask(void const * argument)
{	
	portTickType xLastWakeTime;
	const portTickType xFrequency = pdMS_TO_TICKS(5UL);		// 绝对延时5ms
	xLastWakeTime = xTaskGetTickCount();					// 用当前tick时间初始化 pxPreviousWakeTime

	while(1)
	{
		// 任务绝对延时
		vTaskDelayUntil(&xLastWakeTime, xFrequency);
        // 任务内容...
    }
}
```

# 信号量

- FreeRTOS的信号量包括二进制信号量、计数信号量、互斥信号量（互斥量）和递归互斥信号量（递归互斥量）。
- 互斥量和信号量使用相同的 API 函数，都直接或间接调用通用队列创建函数xQueueGenericCreate()来实现。


- 互斥量和信号量在用法上不同。
  - 互斥量和递归互斥量可以看成特殊的信号量。
  - 信号量用于任务间同步或者任务和中断间同步；互斥量用于互锁，用于保护同时只能有一个任务访问的资源，为资源上一把锁。
  - 信号量用于同步时，一般是一个任务（或中断）给出信号，另一个任务获取信号；互斥量必须在同一个任务中获取信号，同一个任务给出信号。互斥量不能用在中断服务程序中，信号量可以。
  - 互斥量具有优先级继承，信号量没有。


## 二进制信号量

- 二进制信号量既可以用于互斥功能也可以用于同步功能。与互斥量的区别在于不包含优先级继承机制。
- 二进制信号量实际上是创建了一个队列，队列项有1个，但是队列项的大小为0
- 创建二进制信号量API为`xSemaphoreCreateBinary()`

## 计数信号量

- 计数信号量则可以被认为长度大于1的队列。不必关心存储在队列中的数据，只需关心队列是否为空。

- 创建计数信号量API为 `xSemaphoreCreateCounting()`

  两个参数分别为：最大计数值（当信号到达这个值后就不再增长了）；创建信号量时的初始值

- 获取信号量API为`xSemaphoreTake`.带中断保护版本为`xSemaphoreTakeFromISR()`

- 信号量和互斥量（除递归互斥量外）释放的API接口函数都是相同的`xSemaphoreGive()`（不带中断保护）`xSemaphoreGiveFromISR()`为带保护版本。这个宏真正调用的函数是`xQueueGenericSend()`

```c
SemaphoreHandle_t xSemaphore = NULL;		// 信号量句柄

void TestTask(void const * argument)
{
    xSemaphore = xSemaphoreCreateBinary();	// 创建二进制信号量
    if (xSemaphore == NULL) {
        Error_Handler();
    }
    
    while(1)
	{
        // 等待信号量 阻塞时间设置为最多为10ms
        // 也可以设置成等待到永远 osWaitForever
        if (xSemaphoreTake(xSemaphore, (TickType_t)(10 / portTICK_RATE_MS)) == pdTRUE ) {
             // 任务内容...
        } else {
            
        }
    }
}

// 任务中发送信号量
osSemaphoreRelease(xSemaphore);	
// 任务中释放信号量
xSemaphoreGive(xSemaphore);	
// 中断中释放信号量
static BaseType_t xHigherPriorityTaskWoken = pdFALSE;	// 不请求上下文切换
xSemaphoreGiveFromISR(xSemaphore, &xHigherPriorityTaskWoken);
portYIELD_FROM_ISR(xHigherPriorityTaskWoken);			// 判断是否请求上下文切换
```

# 互斥量

- 互斥信号量（互斥量）和递归互斥信号量（递归互斥量）

- 用于互锁的互斥量可以充当保护资源的令牌。当一个任务希望访问某个资源时，它必须先获取令牌。当任务使用完资源后，必须还回令牌，以便其它任务可以访问同一资源。

- 如果一个互斥量（令牌）正在被一个低优先级任务使用，此时一个高优先级企图获取这个互斥量，高优先级任务会因为得不到互斥量而进入阻塞状态，正在使用互斥量的低优先级任务会临时将自己的优先级提升，提升后的优先级与与进入阻塞状态的高优先级任务相同。

  这个优先级提升的过程叫做优先级继承。这个机制用于确保高优先级任务进入阻塞状态的时间尽可能短，以及将已经出现的“优先级翻转”影响降低到最小。

  不过优先级继承不能解决优先级反转，只能将这种情况的影响降低到最小。硬实时系统在一开始设计时就要避免优先级反转发生。
  
- 互斥量不可以用在中断服务程序中。因为互斥量具有优先级继承机制，只有在任务中获取或给出互斥才有意义。并且中断不能因为等待互斥量而阻塞。

- 创建互斥量API为`xSemaphoreCreateMutex()`

  递归互斥量还没有完全弄明白就先不写了（逃

  更多可以参考FreeRTOS官方API文档  https://www.freertos.org/a00113.html

# 任务通知

任务通知是在FreeRTOS版本V8.2.0中推出了全新的功能。在大多数情况下，任务通知可以替代二进制信号量、计数信号量、事件组，可以替代长度为1的队列（可以保存一个32位整数或指针值）。并且任务通知速度更快、使用的RAM更少。不过任务通知并不能完全代替信号量。比如一个任务只能阻塞到一个通知上，如想要实现多个任务阻塞到同一个事件上，只能使用信号量了。

## 发送通知

- `xTaskNotifyGive()`

  发送通知（无通知值）。实际调用函数为`xTaskGenericNotify`。中断保护版本为`vTaskNotifyGiveFromISR `

  ```c
  BaseType_t xTaskGenericNotify( 
          TaskHandle_t xTaskToNotify,				// 被通知的任务句柄
          uint32_t ulValue,						// 更新的通知值
          eNotifyAction eAction,					// 枚举类型，指明更新通知值的方法
          uint32_t *pulPreviousNotificationValue )// 回传未被更新的任务通知值。如果不需要回传未被更新的任务通知值，这里设置为NULL。
  ```

- `xTaskNotify()`

  发送通知。中断保护版本为`xTaskNotifyFromISR `

- `xTaskNotifyAndQuery()`

  发送通知并查询当前通知值。中断保护版本为`xTaskNotifyAndQueryFromISR `

## 等待通知

等待通知API函数只能用在任务中，没有带中断保护版本。

- `ulTaskNotifyTake()`

  用于实现轻量级的二进制信号量和计数信号量。和发送通知API函数xTaskNotifyGive(FromISR)配合使用

  如果第一个参数xClearCountOnExit设置为pdFALSE，则用来实现二进制信号量，函数退出时将通知值清零；如果第一个参数设置为pdTRUE，则用来实现计数信号量，函数退出时，将通知值减一

- `xTaskNotifyWait ()`

  全功能版的等待通知。

  ```c
  TaskHandle_t task_local_handler = NULL;
  
  void Task(void const * argument)
  {
      task_local_handler = xTaskGetHandle(pcTaskGetName(NULL));		// 获取当前任务的任务句柄
      
      while(1)
      {
  		while (ulTaskNotifyTake(pdTRUE, portMAX_DELAY) != pdPASS);	// 等待通知
      }
  }
  
  void xxxISR()
  {
      static BaseType_t xHigherPriorityTaskWoken = pdFALSE;
      vTaskNotifyGiveFromISR(task_local_handler, &xHigherPriorityTaskWoken);	// 发送通知
      portYIELD_FROM_ISR(xHigherPriorityTaskWoken);
  }
  ```

  

# 虚拟定时器

功能相当于基本定时器，能实现毫秒级的定时执行。

虚拟定时器的回调函数和线程不一样，它不能有死循环。

```c
osTimerId superviseTimerHandle;			// 定义虚拟定时器的ID
osTimerDef(superviseTimer, supervise);	// 定义一个虚拟定时器，指定了定时器的回调函数是supervise()
superviseTimerHandle = osTimerCreate(osTimer(superviseTimer), osTimerPeriodic, NULL);	// 创建一个虚拟定时器实例，并指定了定时器模式为osTimerPeriodic模式（连续模式，还有一种模式是只执行一次的osTimerOnce）
osTimerStart(superviseTimerHandle, (5 / portTICK_RATE_MS));	// 启动虚拟定时器，配置定时器5毫秒执行一次

/* 虚拟定时器的回调函数 */
void supervise(void const * argument)
{
	/* USER CODE BEGIN supervise */
	// 内容...
	/* USER CODE END supervise */
}
```



这篇记录一下刚接触FreeRTOS的一点学习笔记。实际开发工程中也没全部用上这些API，完整的工程可以看下刚开源的飞机云台程序。有问题的话也请帮忙指出！

------

**参考资料**

FreeRTOS官方API文档 https://www.freertos.org/a00106.html

掌握 FreeRTOS™ 实时内核 https://freertoskernel.asicfans.com/

https://blog.csdn.net/qq_27114397/category_8113756.html
