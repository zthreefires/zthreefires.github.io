---
layout: post
#标题配置
title:  STM32学习笔记——GPIO使用
#时间配置
date:   2019-07-11 10:00:00 +0800
#大类配置
categories: embeded
#小类配置
tag: STM32
---

* content
{:toc}



使用STM32的外设之前都需要使用对应的初始化结构体和函数对该GPIO引脚进行初始化。另外，STM32为了降低功耗，给每一个外设都配置了独立的时钟，所以在使用外设之前还需要查看原理图和时钟树来初始化对应的时钟。

# 总体上的使用思路是：
1. 创建初始化结构体  
2. 初始化对应的GPIO总线时钟  
3. 配置初始化结构体  
4. 使用初始化函数进行初始化  
5. GPIO口配置完毕  

- 比如配置GPIOH12号引脚为推挽输出模式，初始上拉电平，高速传输信号用以下代码：

``` c
void GPIO_Config(void)
{
	//创建初始化结构体,GPIO_InitTypeDef是一个包含多个枚举类型元素的结
	//构体
	GPIO_InitTypeDef GPIO_InitStruct;
	
	//初始化GPIOH时钟
	RCC_AHB1PeriphClockCmd(RCC_AHB1Periph_GPIOH, ENABLE);
	
	//配置初始化结构体成员，枚举类型元素中的具体选项应到stm32f4xx_gpio.h
	//中查找
	GPIO_InitStruct.GPIO_Pin = GPIO_PIN_12;
	GPIO_InitStruct.GPIO_Mode = GPIO_Mode_OUT;
	GPIO_InitStruct.GPIO_OType = GPIO_OType_PP;
	GPIO_InitStruct.GPIO_PuPd = GPIO_PuPd_UP;
	GPIO_InitStruct.GPIO_Speed = GPIO_Fast_Speed;
	
	//使用初始化函数用初始化结构体进行初始化
	GPIO_Init(GPIOH, &GPIO_InitStruct);
	
	//使对应引脚启动后输出高电平
	GPIO_SetBits(GPIOH, GPIO_PIN_12);
	
	//使对应引脚启动后输出低电平
	GPIO_ResetBits(GPIOH, GPIO_PIN_12);
}
```

- 再比如初始化GPIOA的0引脚为输入模式，不使用上下拉电阻，对于输入GPIO引脚来说不需要配置GPIO_OType和GPIO_Speed两个选项，可通过下述函数进行初始化：

``` c
void GPIO_Config(void)
{
	GPIO_InitTypeDef GPIO_InitStructure;
	
	RCC_AHB1PerphClockCmd(RCC_AHB1Periph_GPIOA, ENABLE);
	
	GPIO_InitStructure.GPIO_Pin = GPIO_Pin_0;
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IN;
	GPIO_InitStructure.GPIO_PuPd = GPIO_PuPd_NOPULL;
	
	GPIO_Init(GPIOA, &GPIO_InitStructure);
}
```


