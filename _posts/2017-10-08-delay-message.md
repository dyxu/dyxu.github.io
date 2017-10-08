---
layout: post
title: 延迟消息设计与实现（转载）
author: reprint
keywords: ["Golang", "数据结构"]
description:
categories:
  - 技术
tags: ["Golang", "数据结构"]

---

{% include JB/setup %}

**本文转载自[博客](https://mp.weixin.qq.com/s/eDMV25YqCPYjxQG-dvqSqQ)和[微信公众号](http://www.cnblogs.com/jkko123/p/7239420.html)**

## 1 缘起

很多时候，业务有“在一段时间之后，完成一个工作任务”的需求。
 
例如：滴滴打车订单完成后，如果用户一直不评价，48小时后会将自动评价为5星。

一般来说怎么实现这类“48小时后自动评价为5星”需求呢？
 
常见方案：启动一个cron定时任务，每小时跑一次，将完成时间超过48小时的订单取出，置为5星，并把评价状态置为已评价。
假设订单表的结构为：`t_order(oid, finish_time, stars, status, …)`，更具体的，定时任务每隔一个小时会这么做一次：
`select oid from t_order where finish_time > 48hours and status=0;`
`update t_order set stars=5 and status=1 where oid in[…];`
如果数据量很大，需要分页查询，分页update，这将会是一个for循环。
 
方案的不足：

1. 轮询效率比较低
2. 每次扫库，已经被执行过记录，仍然会被扫描（只是不会出现在结果集中），有重复计算的嫌疑
3. 时效性不够好，如果每小时轮询一次，最差的情况下，时间误差会达到1小时
4. 如果通过增加cron轮询频率来减少（3）中的时间误差，（1）中轮询低效和（2）中重复计算的问题会进一步凸显

如何利用“延时消息”，对于每个任务只触发一次，保证效率的同时保证实时性，是今天要讨论的问题。
 
## 2 高效延时消息设计与实现

高效延时消息，包含两个重要的数据结构：

1. 环形队列，例如可以创建一个包含3600个slot的环形队列（本质是个数组）
2. 任务集合，环上每一个slot是一个Set<Task>
 
同时，启动一个timer，这个timer每隔1s，在上述环形队列中移动一格，有一个curIndex指针来标识正在检测的slot。
 
Task结构中有两个很重要的属性：
1. CycleNum：当curIndex第几圈扫描到这个Slot时，执行任务
2. TaskFunction：需要执行的任务指针

{% include image.html
            img="images/2017/10/delay_message.png"
            title="延迟消息原理"
            caption="延迟消息原理"
            url="http://dyxu.github.io" %}

假设当前curIndex指向第一格，当有延时消息到达之后，例如希望3610秒之后，触发一个延时消息任务，只需：

1. 计算这个Task应该放在哪一个slot，现在指向1，3610秒之后，应该是第11格，所以这个Task应该放在第11个slot的Set<Task>中
2. 计算这个Task的CycleNum，由于环形队列是3600格（每秒移动一格，正好1小时），这个任务是3610秒后执行，所以应该绕3610/3600=1圈之后再执行，于是CycleNum=1
 
curIndex不停的移动，每秒移动到一个新slot，这个slot中对应的Set<Task>，每个Task看CycleNum是不是0：

1. 如果不是0，说明还需要多移动几圈，将CycleNum减1
2. 如果是0，说明马上要执行这个Task了，取出TaskFunciton执行（可以用单独的线程来执行Task），并把这个Task从Set<Task>中删除
 
使用了“延时消息”方案之后，“订单48小时后关闭评价”的需求，只需将在订单关闭时，触发一个48小时之后的延时消息即可：

1. 无需再轮询全部订单，效率高
2. 一个订单，任务只执行一次
3. 时效性好，精确到秒（控制timer移动频率可以控制精度）

## 3 实现

```go
package main;
 
import (
    "time"
    "errors"
    "fmt"
)
 
//延迟消息
type DelayMessage struct {
    //当前下标
    curIndex int;
    //环形槽
    slots [3600]map[string]*Task;
    //关闭
    closed chan bool;
    //任务关闭
    taskClose chan bool;
    //时间关闭
    timeClose chan bool;
    //启动时间
    startTime time.Time;
}
 
//执行的任务函数
type TaskFunc func(args ...interface{});
 
//任务
type Task struct {
    //循环次数
    cycleNum int;
    //执行的函数
    exec   TaskFunc;
    params []interface{};
}
 
//创建一个延迟消息
func NewDelayMessage() *DelayMessage {
    dm := &DelayMessage{
        curIndex:  0,
        closed:    make(chan bool),
        taskClose: make(chan bool),
        timeClose: make(chan bool),
        startTime: time.Now(),
    };
    for i := 0; i < 3600; i++ {
        dm.slots[i] = make(map[string]*Task);
    }
    return dm;
}
 
//启动延迟消息
func (dm *DelayMessage) Start() {
    go dm.taskLoop();
    go dm.timeLoop();
    select {
    case <-dm.closed:
        {
            dm.taskClose <- true;
            dm.timeClose <- true;
            break;
        }
    };
}
 
//关闭延迟消息
func (dm *DelayMessage) Close() {
    dm.closed <- true;
}
 
//处理每1秒的任务
func (dm *DelayMessage) taskLoop() {
    defer func() {
        fmt.Println("taskLoop exit");
    }();
    for {
        select {
        case <-dm.taskClose:
            {
                return;
            }
        default:
            {
                //取出当前的槽的任务
                tasks := dm.slots[dm.curIndex];
                if len(tasks) > 0 {
                    //遍历任务，判断任务循环次数等于0，则运行任务
                    //否则任务循环次数减1
                    for k, v := range tasks {
                        if v.cycleNum == 0 {
                            go v.exec(v.params...);
                            //删除运行过的任务
                            delete(tasks, k);
                        } else {
                            v.cycleNum--;
                        }
                    }
                }
            }
        }
    }
}
 
//处理每1秒移动下标
func (dm *DelayMessage) timeLoop() {
    defer func() {
        fmt.Println("timeLoop exit");
    }();
    tick := time.NewTicker(time.Second);
    for {
        select {
        case <-dm.timeClose:
            {
                return;
            }
        case <-tick.C:
            {
                fmt.Println(time.Now().Format("2006-01-02 15:04:05"));
                //判断当前下标，如果等于3599则重置为0，否则加1
                if dm.curIndex == 3599 {
                    dm.curIndex = 0;
                } else {
                    dm.curIndex++;
                }
            }
        }
    }
}
 
//添加任务
func (dm *DelayMessage) AddTask(t time.Time, key string, exec TaskFunc, params []interface{}) error {
    if dm.startTime.After(t) {
        return errors.New("时间错误");
    }
    //当前时间与指定时间相差秒数
    subSecond := t.Unix() - dm.startTime.Unix();
    //计算循环次数
    cycleNum := int(subSecond / 3600);
    //计算任务所在的slots的下标
    ix := subSecond % 3600;
    //把任务加入tasks中
    tasks := dm.slots[ix];
    if _, ok := tasks[key]; ok {
        return errors.New("该slots中已存在key为" + key + "的任务");
    }
    tasks[key] = &Task{
        cycleNum: cycleNum,
        exec:     exec,
        params:   params,
    };
    return nil;
}
 
func main() {
    //创建延迟消息
    dm := NewDelayMessage();
    //添加任务
    dm.AddTask(time.Now().Add(time.Second*10), "test1", func(args ...interface{}) {
        fmt.Println(args...);
    }, []interface{}{1, 2, 3});
    dm.AddTask(time.Now().Add(time.Second*10), "test2", func(args ...interface{}) {
        fmt.Println(args...);
    }, []interface{}{4, 5, 6});
    dm.AddTask(time.Now().Add(time.Second*20), "test3", func(args ...interface{}) {
        fmt.Println(args...);
    }, []interface{}{"hello", "world", "test"});
    dm.AddTask(time.Now().Add(time.Second*30), "test4", func(args ...interface{}) {
        sum := 0;
        for arg := range args {
            sum += arg;
        }
        fmt.Println("sum : ", sum);
    }, []interface{}{1, 2, 3});
 
    //40秒后关闭
    time.AfterFunc(time.Second*40, func() {
        dm.Close();
    });
    dm.Start();
}
```