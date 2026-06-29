---

---
### 1. 中断模式下 

1. HAL_UART_Transmit_IT()

![[SelfStudy/Embedded_and_Electronics/Legacy_Notes/学不会呀/早期之电控培训（/asserts/image 2.png]]

2. HAL_UART_Receive_IT()
- 不会堵塞程序运行，上次没接收完就到了下一个循环。
- 放在循环前，只执行一次
- 要等数据接收完成
- 采用回调函数callback（HAL帮忙检测是否完成（固定的字节数）），在调用callback
- receivedata全局变量
- 逻辑直接写在callback函数中

![[SelfStudy/Embedded_and_Electronics/Legacy_Notes/学不会呀/早期之电控培训（/asserts/image 3.png]]

### 2. DMA发送不定长消息

- CubeMX相关设定（UART设定DMA的TX和RX）

![[SelfStudy/Embedded_and_Electronics/Legacy_Notes/学不会呀/早期之电控培训（/asserts/image 4.png]]

