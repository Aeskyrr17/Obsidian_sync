
# 广义坐标
$$
	q = \begin{bmatrix}
	x \\
	\phi \\
	\theta_{l,l} \\
	\theta_{l,r} \\
	\theta_{b} 
	\end{bmatrix} \quad \dot{q} = \begin{bmatrix}
	\dot{x} \\
	\dot{\phi} \\
	\dot{\theta_{l,l}} \\
	\dot{\theta_{l,r}} \\
	\dot{\theta_{b}}
	\end{bmatrix} \quad \ddot{q} = \begin{bmatrix}
	
	\end{bmatrix}
$$
# 输入
$$
	u = \begin{bmatrix}
	\tau_{w,l} \\
	\tau_{w,r} \\
	\tau_{l,l} \\
	\tau_{l,r} \\
	\end{bmatrix}
$$
# 分析

轮子、腿、机体质心
$$
\begin{align}
r_{w,l} &= \begin{bmatrix}
x \\
R
\end{bmatrix}\quad & r_{w,r} &= \begin{bmatrix}
x \\
R
\end{bmatrix}
 \\
 \\
r_{l,l} &= \begin{bmatrix}
x+ l_{w}\sin \theta_{l} \\
R+l_{w}\cos \theta_{l}
\end{bmatrix} \quad & r_{l,r} &= \begin{bmatrix}
x + l_{w,r}\sin \theta_{l,r} \\
R + l_{w,r}\cos \theta_{l,r}
\end{bmatrix}
 \\ \\
 r_{b} &= \begin{bmatrix}
 
 \end{bmatrix}
\end{align}


$$

## Related

- [[RM/balancedCar/balancedCar]]
- [[RM/balancedCar/Notes/WBR Modeling/singleleg_Lagrangian]]
- [[SelfStudy/Control_and_Robotics/Lagrangian Mechanics]]
