---
share_link: https://share.note.sx/3uk97pzp#MUDmYsAHTbt/8dbVfrwjdA
share_updated: 2026-06-26T00:20:37+08:00
---
# 目录




# 约定

| 物理意义       | 数学符号                                               |
| ---------- | -------------------------------------------------- |
| 轮主动力矩      | $`\tau_{w,l},\tau_{w,r}`$                          |
| 髋主动力矩      | $`\tau_{l,l},\tau_{l,r}`$                          |
| 腿对机身水平作用力  | $`F_{l,l}^{h},F_{l,r}^{h}`$                        |
| 腿对机身垂直作用力  | $`F_{l,l}^{v},F_{l,r}^{v}`$                        |
| 轮对腿水平作用力   | $`F_{w,l}^{h},F_{w,r}^{h}`$                        |
| 轮对腿垂直作用力   | $`F_{w,l}^{v},F_{w,r}^{v}`$                        |
| 轮质量        | $`m_{w}`$                                          |
| 腿质量        | $`m_{l}`$                                          |
| 机体质量       | $`m_{b}`$                                          |
| 位移相关       | $`x,\dot{x},\ddot{x}`$                             |
| yaw        | $`\phi,\dot{\phi}, \ddot{\phi}`$                   |
| pitch      | $`\theta_{b},\dot{\theta}_{b},\ddot{\theta}_{b}`$  |
| 摆角         | $`\theta_{l},\dot{\theta}_{l},\ddot{\theta}_{l}`$  |
| 轮角         | $`\theta_{w},\dot{\theta}_{w},\ddot{\theta}_{w}`$  |
| 轮转轴到腿转轴距离  | $`l,l_{l},l_{r}`$                                  |
| 轮转轴到腿质心距离  | $`l_{w},l_{w,l},l_{w,r}`$                          |
| 腿转轴到腿质心距离  | $`l_{b},l_{b,l},l_{b,r}`$                          |
| 轮加速度       | $`a_{w}^h,a_{w}^v`$                                |
| 腿加速度       | $`a_{l}^h,a_{l}^v`$                                |
| 机体加速度      | $`a_{b}^h,a_{b}^v`$                                |
| 机体质心偏移     | $`d_{b},\theta_{b}^0`$                             |
| 腿质心偏移      | $`d_{l},\theta_{l}^0`$                             |
## 单腿建模
### 广义坐标
$$
q=\begin{bmatrix}x\\ \theta_l\\ \theta_{b}\end{bmatrix},\qquad
\dot q=\begin{bmatrix}\dot x\\ \dot\theta_l\\ \dot\theta_{b}\end{bmatrix},\qquad
\ddot q=\begin{bmatrix}\ddot x\\ \ddot\theta_l\\ \ddot\theta_{b}\end{bmatrix}
$$
### 输入

$$
u=\begin{bmatrix}\tau_w\\ \tau_l\end{bmatrix},\qquad R>0
$$
### 分析：

轮子、腿、机体质心
$$
\begin{align}
r_{w} &= \begin{bmatrix}
x \\
R
\end{bmatrix}
 \\
 \\
r_{l} &= \begin{bmatrix}
x+ l_{w}\sin \theta_{l} \\
R+l_{w}\cos \theta_{l}
\end{bmatrix}
 \\ \\
 r_{b} &= \begin{bmatrix}
 x+l\sin \theta_{l} + d_{m}\sin \theta_{b} \\
 R+l\cos \theta_{l}+d_{m}\cos \theta_{b} 
 \end{bmatrix}
\end{align}


$$

#### Kinetic energy
$$
\begin{align}
	T_{w} &= \frac{1}{2} \dot{x}^2 \left( m_{w} + \frac{I_{w}}{R^2} \right)\\ \\
	T_{l} &= \frac{1}{2} m_{l} \dot{r_{l}}^2 + \frac{1}{2} I_{l} \dot{\theta_{l}}^2 \\ \\
	T_{b} &= \frac{1}{2} m_{b} \dot{r_{b}}^2 + \frac{1}{2} I_{b} \dot{\theta_{b}} ^2
\end{align}

$$
将质心坐标带入得
$$
\begin{aligned}
T &= T_{w}+ T_{l} + T_{b} \\ \\
&=\frac{1}{2}\left(m_w+\frac{I_w}{R^2}\right)\dot{x}^{2} \\
&+\frac{1}{2}m_l\left(
\dot{x}^{2}
+l_w^{2}\dot{\theta_l}^{2}
+2\dot{x}l_w\cos\theta_l\,\dot{\theta_l}
\right)
+\frac{1}{2}I_l\dot{\theta_l}^{2} \\
&+\frac{1}{2}m_b\left(
\dot{x}^{2}
+l^{2}\dot{\theta_l}^{2}
+d_m^{2}\dot{\theta_b}^{2}
+2l\cos\theta_l\,\dot{x}\dot{\theta_l}
+2d_m\cos\theta_b\,\dot{x}\dot{\theta_b}
+2ld_m\cos(\theta_l-\theta_b)\dot{\theta_l}\dot{\theta_b}
\right)
+\frac{1}{2}I_b\dot{\theta_b}^{2}
\end{aligned}
$$

#### Potential energy
$$
	\begin{align}
	V_{w} &= m_{w}g \\ \\
	V_{l} &= m_{l} g(R + l_{w} \cos \theta_{l})\\ \\
	V_{b} &= m_{b}g(R + l\cos \theta_{l} + d_{m}\cos \theta_{b})\\ \\
	\end{align}
$$
最终求偏导，可以把带$R$ 的常数项都忽略
$$
V=m_lgl_w\cos\theta_l
+m_bg\left(l\cos\theta_l+d_m\cos\theta_b\right)
$$
#### Lagrangian

$$
\mathcal{L}=T-V
$$

##### Euler-Lagrange equation （记为EL)

$$
E(q,\dot q,\ddot q)
=\frac{d}{dt}\left(\frac{\partial \mathcal{L}}{\partial \dot q}\right)
-\frac{\partial \mathcal{L}}{\partial q}
=Q
$$

其中

$$
\frac{d}{dt}\left(\frac{\partial \mathcal{L}}{\partial \dot q}\right)
=
\frac{\partial}{\partial q}\left(\frac{\partial \mathcal{L}}{\partial \dot q}\right)\dot q
+
\frac{\partial}{\partial \dot q}\left(\frac{\partial \mathcal{L}}{\partial \dot q}\right)\ddot q
$$

##### 广义力

轮电机：
$$
\begin{align}
	\delta W_{w} &= \tau_{w} \delta \phi - \tau_{w} \delta \theta_{l} \\ \\
	&= \tau_{w} (\frac{\delta x}{R} - \delta \theta_{l})
\end{align}
$$
髋关节电机：
$$
\begin{align}
\delta W_{l} &= \tau_{l} \delta \theta_{l} - \tau_{l} \delta \theta_{b} \\ \\
&= \tau_{l} ( \delta \theta_{l} - \delta \theta_{b})
\end{align}

$$
整理得：
$$
	\begin{align}
	\delta W = \frac{\tau_{w}}{R} & \delta x + (\tau_{l} - \tau_{w})\delta \theta_{l} - \tau_{l} \delta \theta_{b}
	\end{align}
$$
由广义力定义：
$$
\delta W = Q^T \delta q
$$
可得到广义力为：

$$
\begin{align}
Q&= \, \begin{bmatrix}
\dfrac{\tau_w}{R}\\[6pt]
\tau_l-\tau_w\\
-\tau_l
\end{bmatrix} \, 
= B_{\tau} u
\end{align}

$$
其中
$$
	B_{\tau} = \begin{bmatrix}
	\frac{1}{R} & 0 \\
	-1 & 1 \\
	0 & -1
	\end{bmatrix}
$$
整理拉格朗日方程后，可以得到
$$
	\begin{align}
	M(q) \ddot{q} + C(q,\dot{q})+ G(q) = B_{\tau}u \\  \\
	\ddot{q} = M(q)^{-1}[B_{\tau}u - C(q,\dot{q}) - G(q)] \\ \\
	\end{align}
$$
$$
	\ddot{q} = f(q,\dot{q},u)
$$
```matlab
% EL = M(q)*ddq + h(q,dq)
% 其中h(q,dq) = C(q,dq)+G(q)
M = simplify(jacobian(EL, ddq));
h = simplify(EL - M*ddq);
ddq_expr = simplify(M \ (Q - h));

```

### 将二阶方程改为一阶状态方程

定义状态变量
$$
\begin{align}
	X &= \begin{bmatrix}
	q \\
	\dot{q}
	\end{bmatrix}
\, = \begin{bmatrix}
x \\
\theta_{l} \\
\theta_{b} \\
\dot{x} \\
\dot{\theta_{l}} \\
\dot{\theta_{b}}
\end{bmatrix} \\ \\
\dot{X} &= \begin{bmatrix}
\dot{q} \\
\ddot{q}
\end{bmatrix} \\ \\
&= \begin{bmatrix}
\dot{q} \\
 M(q)^{-1}[B_{\tau}u - C(q,\dot{q}) - G(q)]
\end{bmatrix}
\end{align}

$$
定义$f(X,u)$
$$
	f(X,u) = \begin{bmatrix}
\dot{q} \\
 M(q)^{-1}[B_{\tau}u - C(q,\dot{q}) - G(q)]
\end{bmatrix}
$$

## Related

- [[RM/balancedCar/balancedCar]]
- [[RM/balancedCar/Notes/VMC]]
- [[SelfStudy/Control_and_Robotics/Lagrangian Mechanics]]
