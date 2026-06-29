**Odometry** 指的是：利用机器人自身的连续传感器数据，估计机器人从上一时刻到当前时刻的**相对运动**。

它通常关心的是：
$$ x_{k-1} \rightarrow x_k  $$
或者更准确地说，是估计相邻时刻之间的运动增量：
$$
\Delta x_k  
$$
在机器人系统中，真实状态 (x_k^{true}) 通常无法直接获得，机器人只能通过传感器和算法得到估计值：
$$
\hat{x}_k  
$$
因此 odometry 输出的不是绝对真值，而是机器人“根据自身传感器认为自己移动了多少”。

常见 odometry 类型：

|类型|信息来源|基本思想|
|---|---|---|
|Wheel / Encoder Odometry|电机编码器、轮速|轮子转了多少，所以机器人应该移动了多少|
|IMU Odometry|陀螺仪、加速度计|根据角速度和加速度积分，估计姿态、速度和位置变化|
|Visual Odometry, VO|相机图像|通过连续图像之间的变化，反推出相机/机器人运动|
|LiDAR Odometry|激光点云|通过连续点云匹配，估计机器人相对位姿变化|
|VIO|Camera + IMU|视觉提供环境约束，IMU 提供高频运动预测|
|LIO|LiDAR + IMU|LiDAR 提供几何约束，IMU 提供高频运动预测|

- Odometry 的特点是：
1. **短期连续性好**：可以高频估计机器人刚刚动了多少。
2. **长期会漂移**：误差会随时间累计。
3. **通常需要修正**：SLAM、GPS、地图匹配、回环检测、landmark 等长期约束可以修正 odometry 漂移

- Odometry 和 SLAM 的关系：
- **Odometry**：估计短时间内“我刚刚动了多少”。
- **SLAM**：同时估计“我在哪里”和“地图是什么样”，并通过回环检测、图优化等方法修正长期漂移。

## Related

- [[SelfStudy/Control_and_Robotics/Kalman Filter]]
- [[SelfStudy/Control_and_Robotics/Robot_Control/拓展卡尔曼]]
- [[SelfStudy/Control_and_Robotics/Control and Robotics]]
