---

---
## Q & Tasks：

1. FDCAN和classic CAN
2. 
3. 
- 整体逻辑
    - 主控板计算双环 PID输出电流/电压的设定值→发送到CAN总线
    - CAN与电调通信
    - 电调将接收到的目标值转换为给电机的信号
    - 电机转动后反馈speed/position
    - via CAN回传参数
    - 继续给PID运算
- 
