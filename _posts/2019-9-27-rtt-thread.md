---
layout: post
#标题配置
title:  RT-thread学习笔记——线程管理
#时间配置
date:   2019-09-27 10:00:00 +0800
#大类配置
categories: embeded
#小类配置
tag: RT-thread
---

* content
{:toc}




线程分为动态线程和静态线程，线程在使用过程中有三个要素，分别是线程控制块、线程栈和线程的入口函数。
使用动态线程不需要显式定义线程栈，只需要通过rt_thread_create()接口创建线程控制块和定义入口函数。
需要注意的是，动态线程创建之后需要监测是否正常创建。
##下面是动态线程的创建过程：

``` c

static rt_thread_t thread1 = RT_NULL;
static void thread1_entry(void* parameter)
{	
    dosomething();
}

int main(void)
{	         
	thread1 =                          
    rt_thread_create( "thread1",          //线程名    
                      thread1_entry,      //线程入口函数名
                      RT_NULL,            //传入线程的参数
                      512,                //线程栈的大小
                      3,                  //线程的优先级
                      20);                //线程的时间片大小
                   
   if (thread1 != RT_NULL)
        rt_thread_startup(thread1);       //启动线程，线程进入就绪状态
    else
        return -1;
}

```

##接下来是静态线程的创建过程，与动态线程不同，静态线程需要主动创建线程栈。需要注意的是在定义线程栈之前要设置多少个字节对齐。
##静态线程和动态线程初始化的线程句柄数据类型也不同。

``` c
static struct rt_thread thread1;
ALIGN(RT_ALIGN_SIZE)
static rt_uint8_t rt_thread1_stack[1024];
static void thread1_entry(void* parameter)
{	
    dosomething();
}

int main(void)
{
    rt_thread_init( &thread1,                       //线程变量
                    "thread1",                      //线程名 
                    thread1_entry,                  //线程入口函数
                    RT_NULL,                        //传入的参数
                    &rt_thread1_stack[0],           //线程栈首地址
                    sizeof(rt_thread1_stack),       //线程栈大小
                    3,                              //线程优先级
                    20);                            //线程时间片
    rt_thread_startup(&thread1);
}
```

