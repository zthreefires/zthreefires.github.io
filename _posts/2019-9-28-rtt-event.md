---
layout: post
#标题配置
title:  RT-thread学习笔记（五）—— 事件
#时间配置
date:   2019-09-28 21:00:00 +0800
#大类配置
categories: embeded
#小类配置
tag: RT-thread
---

* content
{:toc}



# 基本概念
事件的作用是使某线程在特定事件发生后再继续执行。具体实现方法就是位操作的并运算，rtt将事件用二进制的形式存储在事件寄存器中。比如有八个事件，当它们都没有发生的时候即为00000000。当第一个事件发生时，就变成了00000010。而线程中如何检查事件是否发生了呢？假设一个线程需要第二个事件和第四个事件发生之后才会继续执行，那么该线程的事件寄存器中就存储了00001010。当其需要检查事件是否发生的时候，就让该线程事件寄存器中的值和整个程序的事件寄存器中的值进行并运算，当运算完仍为00001010，就应当继续执行。而如果其中的1变成了0，则说明有事件还未发生。



# 使用方法
``` c
static rt_event_t test_event = RT_NULL;         //定义事件句柄
#define KEY1_EVENT    (0x01 << 0)               //定义事件 key1为00000000（具体位数由硬件平台决定）
#define KEY2_EVENT    (0x01 << 1)               //定义事件 key2为00000010

static rt_thread_t receive_thread = RT_NULL;    //定义监测事件线程
static void receive_thread_entry(void* parameter)
{
   rt_uint32_t recved;  
   while (1)
   {                                            //开始监测事件是否发生，若未发生则线程挂起 
     rt_event_recv(test_event,                  //事件句柄
                   KEY1_EVENT | KEY2_EVENT,     //要监测是否发生的事件变量 该例中为 00000011
                   RT_EVENT_FLAG_AND | RT_EVENT_FLAG_CLEAR, //设置监测模式为两个事件都发生才继续执行，执行后清空寄存器标识位
                   RT_WAITING_FOREVER,          //若事件未发生，永远等待
                   &recved);                    //监测结果的接收变量
     if (recved == (KEY1_EVENT | KEY2_EVENT))
     {
       rt_kprintf( "key1 & key2 \n");
       LED1_TOGGLE;
     }
     else
     {
       rt_kprintf("event error! \n");
     }
     
   }
}
static rt_thread_t send_thread = RT_NULL;       //定义发送事件线程
static void send_thread_entry(void* parameter)
{
  while (1)                                     //当KEY1和KEY2被按下时，相对应标识位置一
  {
    if (Key_Scan(KEY1_GPIO_PORT, KEY1_PIN) == KEY_ON)
    {
      rt_kprintf("key1 tap! \n");
      rt_event_send(test_event, KEY1_EVENT);
    }
    else if (Key_Scan(KEY2_GPIO_PORT, KEY2_PIN) == KEY_ON)
    {
      rt_kprintf("key2 tap! \n");
      rt_event_send(test_event, KEY2_EVENT);
    }

    rt_thread_delay(20);
  }

}



int main(void)
{	
  test_event = rt_event_create("test_mux",
                             RT_IPC_FLAG_PRIO);

  if (test_event != RT_NULL)
  {
    rt_kprintf("event create successes! \n\n");
  }
                
	receive_thread =                          
    rt_thread_create( "receive",              
                      receive_thread_entry,   
                      RT_NULL,             
                      512,                
                      3,                   
                      20);                
                   
   if (receive_thread != RT_NULL)
        rt_thread_startup(receive_thread);
    else
        return -1;


  send_thread = 
    rt_thread_create("send",
                      send_thread_entry,
                      RT_NULL,
                      512,
                      2,
                      20);
  
  if (send_thread!=RT_NULL)
        rt_thread_startup(send_thread);
  else 
        return -1;
}

```

main函数中应当只进行事件等对象和线程的初始化。
