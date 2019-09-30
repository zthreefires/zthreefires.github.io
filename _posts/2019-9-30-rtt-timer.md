---
layout: post
#标题配置
title:  RT-thread学习笔记（六）——软件定时器
#时间配置
date:   2019-09-30 10:00:00 +0800
#大类配置
categories: embeded
#小类配置
tag: RT-thread
---

* content
{:toc}



# 基本概念
rtt提供基于硬件定时器的软件定时器，它使用起来比硬件定时器更便捷，缺点是精度较于硬件定时器要差一些。

# 使用方法

``` c
static rt_timer_t swtmr = RT_NULL;                  //定义软件定时器控制块
static void swtmr_callback(void* parameter)         //定义软件定时器回调函数，计时器到时间后会自动调用
{
    dosomething();
}

int main(void)
{                                                   //初始化软件定时器控制块
    swtmr = rt_timer_create("swtmr_callback",       //软件定时器名称
                            swtmr_callback,         //软件定时器回调函数句柄
                            0,                      //回调函数的入口参数
                            1000,                   //软件定时器超时时间
                            RT_TIMER_FLAG_ONE_SHOT | RT_TIME_FLAG_SOFT_TIMER);    //触发模式，本例为一次触发后结束，也可周期触发。
    if (swtmr !=RT_NULL)
        rt_timer_start(swtmr);
}

```


# 注意事项
1. 软件定时器的使用不再是线程的形式。
2. 软件定时器依托于系统时钟节拍值，可以通过rt_tick_get()获取当前系统节拍时钟值。
