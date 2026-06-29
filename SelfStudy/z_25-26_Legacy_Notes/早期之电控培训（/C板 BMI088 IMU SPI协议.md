---

---
## SPI

全双工通信协议

![[SelfStudy/Embedded_and_Electronics/Legacy_Notes/学不会呀/早期之电控培训（/asserts/image 11.png]]

(这张图是发送数据）

SCK：时钟

SS：用高/低电平判断是否接收数据

- 收集数据：时钟下沿时接收
- 发送数据：时钟上沿时发送
- 对于SPI，并没有规定只能发送多少位，取决于芯片，大部分是8位

- 对于BMI088里面使用的一帧是8位1字节

## BMI088

- 7加速度计+陀螺仪计，均有三个方向
- 加速度/陀螺仪速度值，分别有3个数据，每个数据16位，分为高八位&低八位，一共存在12格八位寄存器
    - 0x12到0x17 加速度
    - 0x02到0x07 陀螺仪速度值
- 温度值总长度11位，

### tip 

MSB & LSB: most significant byte/bit & least significant byte/bit

加速度计的ID寄存器地址为0x00，标准ID值为0x1E，陀螺仪的ID寄存器地址为0x00，
标准ID值为0x0F

```c++
(int16_t)((buf[1]) << 8) | buf[0];
```

拼接数据

```c++
 accel[0] = bmi088_raw_temp * BMI088_ACCEL_SEN;
```

* BMI088_ACCEL_SEN 是一个系数，把接收到的传感器的raw数字转换为实际的加速度/陀螺仪转速/温度

