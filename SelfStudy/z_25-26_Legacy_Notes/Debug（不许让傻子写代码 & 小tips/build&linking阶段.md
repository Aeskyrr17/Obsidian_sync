---

---
> [!note]+ TaskGimbal 线程
> - 线程 if收到imu数据，要sleep
> - 不要在for循环里创建topic或者创建suber
> 

- **H7 ld链接错误 Dec14**
    - 项目编译失败，链接阶段报告大量 `undefined reference` 错误，指向 ThreadX RTOS 的核心函数，如 `_tx_thread_sleep` 和 `_txe_semaphore_put` 等。
    - 查看onemessage+外层cmakelist，抄的文档没发现错误
    - 是cmake中stm32生成的cmakelists不完整。就是没在cubemx上配好threadx
