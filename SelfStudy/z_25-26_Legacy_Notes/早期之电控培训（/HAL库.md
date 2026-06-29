---

---
## 句柄 HandleStruct

> 句柄（Handle Struct） = 外设的运行信息 + 配置信息 + 外设寄存器地址 + DMA/中断状态

```c++
typedef struct
{
USART_TypeDef *Instance;   // 外设寄存器的基地址，比如 USART1
UART_InitTypeDef Init;     // 波特率、字长、停止位的配置
DMA_HandleTypeDef hdmatx;  // 发送用的 DMA
DMA_HandleTypeDef hdmarx;  // 接收用的 DMA
...
} UART_HandleTypeDef;
```

handle struct主要是用于储存某一个东西的多个参数

e.g. UART ：寄存器地址（Instance）init dma状态 中断状态 …

所以huart（handle uart）就是这个结构体的实例变量-真正的句柄

> 
> | 概念 | 解释 | 举例 |
> | --- | --- | --- |
> | **HandleTypeDef（句柄类型定义）** | 一种结构体类型 | `UART_HandleTypeDef` |
> | **句柄（变量）** | 用这个结构体类型创建的变量，用来管理外设 | `huart1` |
