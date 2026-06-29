---

---

## pid

![[SelfStudy/Embedded_and_Electronics/Legacy_Notes/学不会呀/早期之电控培训（/asserts/image 12.png]]

![[SelfStudy/Embedded_and_Electronics/Legacy_Notes/学不会呀/早期之电控培训（/asserts/image 13.png]]

![[SelfStudy/Embedded_and_Electronics/Legacy_Notes/学不会呀/早期之电控培训（/asserts/image 14.png]]

## 注意的点

1. 以后提前写好debug的东西，线程内的struct是没法被观测的。
2. Ozone中能watch的东西都只能是**全局变量，**可以定义一个全局的结构体然后把他memcpy

### 调PID之玄学 

3. 先给pos一个小p，让他能动起来
4. 开始调spd，spd的p一般都比较大
5. p多了加d，d多了加p
6. spd能跟得差不多了，就是能跟上但是不明显超调的时候，时候开始调pos
7. pos加i（物理系统没有稳态静差的，例如yaw，不加i，pitch可以加i）
8. p多了加d，d多了加p

## 代码逻辑注意的点

9. `remember to sleep (因为前面if接收到IMU信息才开始remoter控制 发送信息是1ms）`
10. `//only if IMU have send data, start gimbal controlif `
`(tx_semaphore_get(&IMUThreadSem, TX_WAIT_FOREVER) == TX_SUCCESS)`
用来保证接收到IMU信息之后才发送
11. 记得maxout和maxiout
12. `pitch_pos_ref_temp += msg_remoter.left_y*k_ref_pitch;`
`yaw_pos_pid.ref += -msg_remoter.left_x*k_ref_yaw;`
摇杆的值是累加
13. 转换弧度制和角度制（不然pid会爆炸）
14. 记得判断if remoter是offline，currentset=0/ if remoter mode = nomal 开始控制
15. 电机要register

## BeforeHW&Q

> [!note]+ ServiceBooster
> - ServiceBooster(void) = appstart
> - Semaphore_create: **RemotorThreadSem, IMUThreadSem** 创建了remotor和IMU的信息量
> - thread_vreate: RemotorThread, MotorThread 创建了Remotor和Motor的线程
> - `tx_byte_pool_create` ？

> [!note]+ ServiceIMU
> - 定义了 **IMUThread， IMUThreadSem**
> - 26 InitQuaternion(*init_q4) 四元数的计算 将其转化为四元数
> - IMUThreadFun（）：
>     - 57.58 创建了IMU的topic
>     - 87开始 读acc和gyro的数据
>     - 95开始 判断acc和gyro是不是很小corr_start yaw_static
>     - 125 姿态解算update
>     - 131`tx_semaphore_put(&IMUThreadSem);`
>     - 133-140的pitch和yaw和转换不是很明白
> > [!note]+ 贴一个g老师回答?
> > ## 🧭 一、先搞清楚 IMU 姿态的三个旋转轴（右手坐标系）
> > 
> > | 轴 | 旋转名称 | 云台对应关系 | 控制用途 |
> > | --- | --- | --- | --- |
> > | X轴 | Pitch（俯仰） | 云台上下转 | 需要控制 |
> > | Y轴 | Roll（横滚） | 相机左右倾斜 | 云台不控制 |
> > | Z轴 | Yaw（偏航） | 云台左右转 | 需要控制 |
> > 
> > 你的云台是 **双轴**：
> > 
> > 👉 只需要 **X轴的 Pitch** 和 **Z轴的 Yaw**
> > 
> > 👉 不用 Roll（Y轴）
> > 
> > ---
> > 
> > ## 🔍 区分代码里的这几个 pitch/yaw
> > 
> > | 变量名 | 含义 | 坐标系 | 用途 |
> > | --- | --- | --- | --- |
> > | `QEKF_INS.Yaw` | EKF解算后的偏航角（真实的Yaw角） | 惯导系 | 云台 yaw 位置环反馈 |
> > | `QEKF_INS.Roll` | EKF解算后的 Roll，但这里被当作 Pitch！ | 惯导系 | 云台 pitch 位置环反馈 |
> > | `QEKF_INS.Pitch` | EKF解算后的 Pitch，但这里被当作 Roll！ | 惯导系 | **云台不使用** |
> > | `imu_handler->gyro_data.z` | 角速度绕 Z轴 | 机体系 | 云台 yaw 的速度环反馈 |
> > | `imu_handler->gyro_data.x` | 角速度绕 X轴 | 机体系 | 云台 pitch 的速度环反馈 |
> > | `imu_handler->gyro_data.y` | 角速度绕 Y轴 | 机体系 | 横滚速度（云台不用） |
> > 
> > ---
> > 
> > ### ❗关键点：为什么 pitch = QEKF_INS.Roll？
> > 
> > ```c++
> > msg_ins.pitch = QEKF_INS.Roll;
> > msg_ins.roll = -QEKF_INS.Pitch;
> > 
> > 
> > ```
> > 
> > 这是因为 BMI088 的 IMU 安装方向与云台坐标系**不一致**，换了轴：
> > 
> > | IMU解算出的值 | 云台实际意义 |
> > | --- | --- |
> > | Roll | Pitch |
> > | Pitch | -Roll |
> > | Yaw | Yaw（不变） |
> > 
> > 👉 所以为了让云台控制代码用对角度，做了对应关系的转换。
> > 
> > ---
> > 
> > ## 🤖 对你自己的云台控制来说，你只需要这些：
> > 
> > | 控制轴 | 位置环反馈 | 速度环反馈 |
> > | --- | --- | --- |
> > | Yaw | `msg_ins.yaw` | `imu_handler->gyro_data.z` |
> > | Pitch | `msg_ins.pitch`（其实是 Roll） | `imu_handler->gyro_data.x` |
> > 
> > ---
> > 
> > ## ✍️ 最终简化理解版（适合写代码时记）
> > 
> > | 用途 | 使用变量 |
> > | --- | --- |
> > | Pitch角度反馈 | `msg_ins.pitch` |
> > | Pitch角速度反馈 | `msg_ins.gyro_p` |
> > | Yaw角度反馈 | `msg_ins.yaw` |
> > | Yaw角速度反馈 | `msg_ins.gyro_y` |
> > | Roll相关 | 都不用 |
> > 
> > ---
> > 
> > ## 🧠 最后一句话总结
> > 
> > > IMU 的原始旋转轴是 Roll（绕X）、Pitch（绕Y）、Yaw（绕Z），但机器人安装方向不同，所以代码里用 Roll 当 Pitch。你的云台只需要 Pitch（绕X）和 Yaw（绕Z），完全不用管 Roll。
>     - 146 publish msg

> [!note]+ SeiviceMotor
> - 86 fdb 云台不需要motor的positionfdb和speedfdb
> - 有用的是currentset
> - motor的pid参数没用（？

> [!note]+ ServiceRemotor
> - 12，13 放到指定的内存是如何实现的？
> - 15 inline 不用自己拆字节，直接用Dr16_Data获取摇杆值
> - 29 创建了remotor的topic
> - 33 如果在100ms内没收到remotor数据就当作掉线，publish
> - 72 把所有remotor的消息都publish

> [!note]+ TaskGimbal
> 
> 

- msg消息结构体的定义都在magicmsgs.hpp

问题：

- maxout应该怎么填   —   maxout给10000吧 也不知道为什么
- 前馈怎么加
- “整体控制逻辑“中，云台转动为什么是positionset加上摇杆值？为什么？不是累加值吗


