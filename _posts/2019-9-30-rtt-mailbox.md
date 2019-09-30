---
layout: post
#标题配置
title:  RT-thread学习笔记（六）——邮箱
#时间配置
date:   2019-09-30 09:00:00 +0800
#大类配置
categories: embeded
#小类配置
tag: RT-thread
---

* content
{:toc}



# 基本概念
邮箱是线程间通信的方式之一，应当算是特殊的消息队列，但二者在应用中的区别还要再查资料弄明白。

# 使用方法

``` c
static rt_mailbox_t test_mb = RT_NULL;                          //初始化邮箱变量
char test_str[] = "This is a test!\n";                          //定义测试字符串

static rt_thread_t receive_thread = RT_NULL;                    //初始化接收数据线程
static void receive_thread_entry(void* parameter)               //定义接收数据线程入口函数
{
    rt_err_t uwRet = RT_EOK;
    char *r_str;
    while(1)
    {
        uwRet = rt_mb_recv(test_mb,                             //接收邮箱数据api
                            (rt_uint32_t*)&r_str,               //用于接收数据的变量地址
                            RT_WAITTING_FOREVER);               //如果邮箱中没有数据则挂起等待
        if (uwRet == RT_EOK)
        {
            rt_kprintf("recived:%s\n", r_str);
        }
        else
        {
            rt_kprintf("error:0x%x\n", uwRet);
        }
    }
}

static rt_thread_t send_thread = RT_NULL;                       //初始化发送数据线程
static void send_thread_entry(void* parameter)                  //定义发送数据线程入口函数
{           
    rt_err_t uwRet = RT_EOK;
    while(1)
    {
        if (Key_Scan(KEY1_GPIO_PORT, KEY1_PIN) == KEY_ON)
        {
            rt_kprintf("KEY1 ON!\n");
            uwRet = rt_mb_send(test_mb, (rt_uint32_t)&test_str);//发送与偶像数据api
            if (uwRet == RT_EOK)
            {
                rt_kprintf("send ok!\n");
            }
            else
            {
                rt_kprintf("send failed!\n");
            }
        }
        rt_thread_delay(20);
    }
}

int main()
{
    test_mb = rt_mb_create("test_mb", 10, RT_IPC_FLAG_FIFO);
    if (test_mb != RT_NULL)
    {
        rt_kprintf("mail box create success!\n");
    }

    receive_thread = rt_thread_create("receive",
                                      receive_thread_entry,
                                      RT_NULL,
                                      512,
                                      3,
                                      20);
  if (receive_thread != RT_NULL)
          rt_thread_startup(receive_thread);
  else
          return -1;
  send_thread = rt_thread_create("send",
                                  send_thread_entry,
                                  RT_NULL,
                                  512,
                                  3,
                                  20);
  if (send_thread != RT_NULL)
          rt_thread_startup(send_thread);
  else
          return -1;
}
```


