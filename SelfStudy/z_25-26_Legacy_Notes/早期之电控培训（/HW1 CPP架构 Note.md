---

---
# 文件架构

```javascript
你的MX生成文件夹名字
├─.idea
├─cmake-build-debug
├─Core
...
//省略一堆MX自动创建的文件夹，下面是你创建的文件夹
...
└─User
    └─BSP
        ├─bsp_usart.hpp
        └─bsp_usart.cpp
    └─crc
        ├─crc.hpp
        └─crc.cpp
    └─Maix
        ├─Maix.hpp
        └─Maix.cpp
```

# BSP

> [!note]+ ## BSP.hpp
> ```c++
> #ifndef BSP_USART_HPP
> #define BSP_USART_HPP
> #ifdef __cplusplus
> extern "C" {
> #endif
> #ifdef __cplusplus
> }
> #endif // #ifdef_cplusplus
> void USART_Init();
> #endif // #ifdef BSP_USART_HPP
> ```
> 
> - 封装所有的HAL有关函数 

> [!note]+ ## BSP.cpp
> - 把TransmitDMA， ReceiveToIdle等卸载init里面
> - 然后再定义eventcallback，会自动进入回调函数
> 
> ```c++
> #include "bsp_usart.hpp"
> #include "main.h"
> #include "Maix.hpp" //用MAIX中的类
> #include <string.h> //memcpy用，拷贝数据
> 
> extern UART_HandleTypeDef huart1;
> extern DMA_HandleTypeDef hdma_usart1_rx;
> extern DMA_HandleTypeDef hdma_usart1_tx;
> 
> //使用宏定义定义 MAIX_SIZE ——从我们要发送的数据得知，应该是12
> #define MAIX_SIZE 12
> 
> uint8_t UART_RxBuffer[MAIX_SIZE] = {0}; 
> //定义接收DMA信息的数组
> 
> /*
>  *我们应该在这里重新定义一个UART_RxBuffer，（个人理解）作为硬件层传输数据的桥梁
>  *可以想 因为这里是UART_xxxx说明是从UART传来的
>  *最后要把这个UART_RxBuffer的数据拷贝到Maix中我们想要接受的数据组
>  */
> 
> void USART_Init()
> {
>     //关闭DMA半传输中断 @TODO:为什么？-----因为我们只需要传输完成中断，半传输是STM32默认的
>     __HAL_DMA_DISABLE_IT(&hdma_usart1_rx, DMA_IT_HT);
>     __HAL_DMA_ENABLE_IT(&hdma_usart1_rx, DMA_IT_TC);
>     __HAL_DMA_DISABLE_IT(&hdma_usart1_tx, DMA_IT_HT);
>     __HAL_DMA_ENABLE_IT(&hdma_usart1_tx, DMA_IT_TC);
> 
>     //开启DMA信息接收
>     HAL_UARTEx_ReceiveToIdle_DMA(&huart1, UART_RxBuffer, sizeof(UART_RxBuffer));
> 
> }
> /**
>  * @brief 串口接受回调函数
>  */
> void HAL_UARTEx_RxEventCallback(UART_HandleTypeDef *huart, uint16_t Size)
> {
>     HAL_UART_Transmit_DMA(&huart1, UART_RxBuffer, sizeof(UART_RxBuffer));
>     //把数据发回来，测试收没收到
>     
>     if (huart->Instance == USART1)
>     {
>         //把接收DMA信息的数组的内容（我们定义为前面的UART_RxBuffer）copy到RxBuffer这个数组里
>         //RxBuffer指的就是我们receive的buffer
>         memcpy(MaixComm::Instance()->MaixCommRx.buffer, UART_RxBuffer, sizeof(UART_RxBuffer));
>         //一次传输完成后重新开启DMA接收
>         HAL_UARTEx_ReceiveToIdle_DMA(&huart1, UART_RxBuffer, sizeof(UART_RxBuffer));
> 
> /*
>  *如果来不及处理，DMA会开始下一轮接受数据，会覆盖UART_RxBuffer,所以要拷贝到另外一个缓冲区去
>  */
>     }
> }
> 
> 
> ```
> 
> ```c++
> //记住在bsp层调用MAIX.hpp中定义的类的实例的方法
> MaixComm::Instance()->MaixCommRx.buffer
> ```

# Maix

> [!note]+ ## maix.hpp
> - 定义class
> ```c++
> #ifndef MAIX_HPP
> #define MAIX_HPP
> 
> #include <cstdint>
> 
> #ifdef __cplusplus
> extern "C" {
> #endif
>     struct maix_comm_data_t
>     {
>         uint8_t header;   // 数据包头
>         uint8_t Vx;       // X轴速度
>         uint8_t Vy;       // Y轴速度
>         uint8_t Vw;       // 角速度
>         uint8_t M1, M2, M3, M4, M5, M6;  // 6个舵机控制值
>         uint16_t crc;     // CRC校验值
>     } __attribute__((packed));
> 
>     union maix_comm_rx_t
>     {
>         maix_comm_data_t data;
>         uint8_t buffer[sizeof(maix_comm_data_t)];
>     };
> 
>     //maix_comm_rx_t联合体使得我们可以使用buffer数组直接接收原始数据，
>     //也可以使用data访问解析后的结构体数据,
>     
>     class MaixComm
>     {
>     public:
> 	    //在MaixComm类里里面，MaixCommRx就是union结构体的名字
>         maix_comm_rx_t MaixCommRx;
>       //MaixCommRx是一个maix_comm_rx_t类型的对象
>         void Init();
>         void Update();
> 
>         static MaixComm *Instance()
>         {
>             static MaixComm instance;
>             return &instance;
>         }
>         //创建了一个instance实例 此实例是指针
>         
>     private:
>         uint8_t PrevReceivedData[sizeof(maix_comm_rx_t)];
> 				//定义了一个previous ReceidedData 上次接受的数据缓冲区
> 
>     };
> #ifdef __cplusplus
>     }
> #endif
> 
> 
> #endif
> ```

> [!note]+ ## Maix.cpp
> ```c++
> 
> #include "Maix.hpp"
> #include "crc.hpp"
> #include "bsp_usart.hpp"
> #include<string.h>
> 
> #define MAIX_COMM_SIZE sizeof(maix_comm_data_t)
> 
> uint32_t count = 0;
> void MaixComm::Init()
> {
>     // uint8_t MaixCommRx[MAIX_COMM_SIZE]= {0};
>     memset(&MaixCommRx.buffer, 0, sizeof(MaixCommRx));
>     // uint8_t PrevReceivedData[MAIX_COMM_SIZE];
>     // memset(PrevReceivedData, 0, sizeof(PrevReceivedData));
> }
> //初始化MaixCommRx数组？）
> void MaixComm::Update()
> {
>     count++;
>     //计时器要在一开始++，在最后清零
>     //Update这个函数是在while（1）循环里面的
> 	//访问结构体里面的对象 MaixCommer.Buffer
>     if (Verify_CRC16_Check_Sum(MaixCommRx.buffer, sizeof(maix_comm_data_t)))
>     {
>         memcpy(&PrevReceivedData, &MaixCommRx.buffer, sizeof(maix_comm_data_t));
>         //把buffer里面的地址拷贝到previous Received Data的地址（mempy的规定）
>     }
>     else
>     {
>         memcpy(&MaixCommRx.buffer,&PrevReceivedData, sizeof(maix_comm_data_t));
>     }
>     if (count>=100)
>     {
>     memset(&PrevReceivedData, 0, sizeof(maix_comm_data_t));
>     }
> 
>     count = 0;
> }
> 
> 
> ```
