---

---
[[视觉联调CRC校验有时候成功有时候warning]]

[[HardFualt Handler（）]]

[[26.1.16 收不到pos_fdb（是drone的）]]

[[电机控制setoutput相关]]

[[视觉联调tips]]

[[REFEREE：数据解包，线程要睡觉。。]]

[[TookChain]]

[[build&linking阶段]]

### 没反应？

> [!note]+ 线程创建相关
> 1. ==看有没有create线程。。。。。。。。 （能正常build，能下载，就是没反应）其实应该先debug模式看看进不进线程==
> 2. 不进某个线程，也可能是优先级更高的线程卡住了
> 3. （现象）比如说freemaster找不到variable？看看是不是整个文件的变量都找不到？

> [!note]+ onemessage相关
> 1. 记住，熟悉好流程：创建好topic，publish/创建好suber，publish
> 2. 

嵌入式

> [!note]+ 不进中断
> > [!note]+ 内存放的位置：H7的DMA都只能访问D1，D3访问不到
> 

对于非标硬件！！！eg步进电机

1. ==注意线序！！！==
2. ==注意线序！！！==
3. ==注意线序！！！==
4. ==注意线序！！！==
5. ==注意线序！！！==
6. ==注意线序！！！==

# 1. 硬件

7. ==注意线序！！！==包括串口的txrx，can的high和low。串口能收到数据也不一定说明线序就是对的（txrx接上了就可收/发，但gnd不一定接对了）
8. 接线问题
    1. ST-link 接板子（下载）
        - 板子朝左 从上到下DCGV（SDO SCK GND 3V3）
    2. 串口 （USART接USB模块）
        - UASRT从左到右：TX RX GND USB不要接反了!!!

<!-- Column 1 -->



<!-- Column 2 -->

![[SelfStudy/Embedded_and_Electronics/Legacy_Notes/Debug（不许让傻子写代码 & 小tips/asserts/e245eb9f-8955-493a-872b-8edc4d46583a.png]]

<!-- Column 3 -->

![[SelfStudy/Embedded_and_Electronics/Legacy_Notes/Debug（不许让傻子写代码 & 小tips/asserts/image.png]]


<!-- Column 4 -->


<!-- Column 5 -->


9. 电调id是否对应
10. CAN线序：不同板子的CAN high和CAN low顺序可能不一样，要对照手册/图找can线
11. 

# 2. 软件问题

## CubeMX配置

12. GPIO有没有配错
13. 时钟树
14. 

## 代码

15. bsp等其他的init都要在Cubemx的init后面
16. 看是否override了（PID调参作业 没配FDCAN2口，但FDCAN2有函数override了，所以没法跑）
17. CAN数据帧 


