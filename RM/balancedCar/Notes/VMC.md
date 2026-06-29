---
share_link: https://share.note.sx/dyxkyeha#NVy5tzoYsDGj6+WNiK7puQ
share_updated: 2026-06-27T20:51:41+08:00
---

# 五连杆VMC
- 定义：

| 符号                                                       | 意义            |
| -------------------------------------------------------- | ------------- |
| $l_{1} ,l_{2}, l_{3,}l_{4}$                              | 五连杆各杆长       |
| $\phi_{1}, \phi_{2}, \phi_{3},\phi_{4}$                 | 各关节角          |
| $\dot\phi_{1}, \dot\phi_{2}, \dot\phi_{3}, \dot\phi_{4}$ | 各关节角速度        |
| $x_{c},y_{c}$                                            | 足端坐标          |
| $\tau_{1}, \tau_{4}$                                     | 主动关节力矩        |
| $F_{L},\tau_{p}$                                         | 虚拟腿长方向力和虚拟力矩 |

### 广义坐标
$$
x = \begin{bmatrix}
L \\
\phi
\end{bmatrix}
$$

### 广义力
$$
F_{virtual} = \begin{bmatrix}
F_{L} \\
\tau_{p}
\end{bmatrix}
$$

### 关节空间
$$
\phi = \begin{bmatrix}
\phi_{1} \\
\phi_{4}
\end{bmatrix}
\qquad
\dot{\phi} = \begin{bmatrix} 
\dot{\phi}_{1}\\
\dot{\phi}_{4}
\end{bmatrix}
\qquad
\tau = \begin{bmatrix}
\tau_{1}\\
\tau_{4}
\end{bmatrix}
$$

### 几何约束
$$
\begin{gathered}
x_{c} = -d + l_{1}\cos \phi_{1} + l_{2}\cos \phi_{2} \\ \\
y_{c} = l_{1}\sin \phi_{1} + l_{2}\sin \phi_{2} \\ \\
x = \begin{bmatrix}
x_{c} \\
y_{c}
\end{bmatrix}
\end{gathered}
$$

#### Constraint equations
$$
\begin{aligned}
C_{1} &= l_{1}(\cos \phi_{4} - \cos \phi_{1}) + l_{2}(\cos \phi_{3} - \cos \phi_{2})+2d \\ \\
C_{2} &= l_{1}(\sin \phi_{4} -\sin \phi_{1}) + l_{2}(\sin \phi_{3} - \sin \phi_{2})\\
\end{aligned}
$$

$$
C = \begin{bmatrix}
C_{1}\\
C_{2}
\end{bmatrix}
$$

### 主动关节与被动关节
保留主动关节：
$$
p = \begin{bmatrix}
\phi_{1}\\
\phi_{4}
\end{bmatrix}
$$

消去被动关节：
$$
q = \begin{bmatrix}
\phi_{2}\\
\phi_{3}
\end{bmatrix}
$$

由约束方程：
$$
C(p,q)=0
$$

对约束方程求微分：
$$
dC = C_{p}dp + C_{q}dq = 0
$$

其中：
$$
C_{p}=\frac{\partial C}{\partial p}
=\begin{bmatrix}
l_{1}\sin\phi_{1} & -l_{1}\sin\phi_{4}\\
-l_{1}\cos\phi_{1} & l_{1}\cos\phi_{4}
\end{bmatrix}
$$

$$
C_{q}=\frac{\partial C}{\partial q}
=\begin{bmatrix}
l_{2}\sin\phi_{2} & -l_{2}\sin\phi_{3}\\
-l_{2}\cos\phi_{2} & l_{2}\cos\phi_{3}
\end{bmatrix}
$$

所以被动关节速度可以用主动关节速度表示：
$$
\frac{dq}{dp} = -C_{q}^{-1}C_{p}
$$

即：
$$
dq = -C_{q}^{-1}C_{p}dp
$$

### 足端雅可比
足端位置对主动关节与被动关节分别求偏导：
$$
x_{p}=\frac{\partial x}{\partial p}
=\begin{bmatrix}
-l_{1}\sin\phi_{1} & 0\\
l_{1}\cos\phi_{1} & 0
\end{bmatrix}
$$

$$
x_{q}=\frac{\partial x}{\partial q}
=\begin{bmatrix}
-l_{2}\sin\phi_{2} & 0\\
l_{2}\cos\phi_{2} & 0
\end{bmatrix}
$$

足端速度为：
$$
dx = x_{p}dp + x_{q}dq
$$

代入被动关节速度：
$$
\begin{aligned}
dx &= x_{p}dp + x_{q}\left(-C_{q}^{-1}C_{p}dp\right)\\
&= \left(x_{p}-x_{q}C_{q}^{-1}C_{p}\right)dp
\end{aligned}
$$

因此五连杆等效雅可比为：
$$
J_{xy} = x_{p}+x_{q}\frac{dq}{dp}
= x_{p}-x_{q}C_{q}^{-1}C_{p}
$$

满足：
$$
\dot{x}=J_{xy}\dot{\phi}
$$

### 延腿长和切向力转换
腿长：
$$
L = \sqrt{x_{c}^{2}+y_{c}^{2}}
$$

虚拟腿与水平方向夹角为：
$$
\alpha
$$

从足端坐标系到腿坐标系的旋转矩阵：
$$
R = \begin{bmatrix}
\cos\alpha & \sin\alpha\\
-\sin\alpha & \cos\alpha
\end{bmatrix}
$$

虚拟力中第二项为力矩，需要先除以腿长转成切向力：
$$
M = \begin{bmatrix}
1 & 0\\
0 & L
\end{bmatrix}
$$

$$
\begin{bmatrix}
F_{L}\\
F_{T}
\end{bmatrix}
= M^{-1}F_{virtual}
=\begin{bmatrix}
F_{L}\\
\frac{\tau_{p}}{L}
\end{bmatrix}
$$

再转换到足端 $x,y$ 坐标系：
$$
F_{xy}=R^{-1}M^{-1}F_{virtual}
$$

### 虚拟力到关节力矩
由虚功原理：
$$
\tau^{T}d\phi = F_{xy}^{T}dx
$$

代入：
$$
dx=J_{xy}d\phi
$$

得到：
$$
\tau = J_{xy}^{T}F_{xy}
$$

所以：
$$
\boxed{
\tau = J_{xy}^{T}R^{-1}M^{-1}F_{virtual}
}
$$

也就是代码中的：
```matlab
tau_expr = J_xy.' * (R \ (M \ F_virtual));
```

# 串联腿VMC
matlab结果
```matlab
[- (l1*sin(phi1/2 - phi4/2))/2 - (l1^2*cos(phi1/2 - phi4/2)*sin(phi1/2 - phi4/2))/(2*(l2^2 - l1^2*sin(phi1/2 - phi4/2)^2)^(1/2)), (l1*sin(phi1/2 - phi4/2))/2 + (l1^2*cos(phi1/2 - phi4/2)*sin(phi1/2 - phi4/2))/(2*(l2^2 - l1^2*sin(phi1/2 - phi4/2)^2)^(1/2))]
[                                                                                                                            1/2,                                                                                                                           1/2]
```

## Related

- [[RM/balancedCar/balancedCar]]
- [[RM/balancedCar/Notes/WBR Modeling/singleleg_Lagrangian]]
- [[RM/balancedCar/25-26_legacynotes/VMC逻辑]]
