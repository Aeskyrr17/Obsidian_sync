---
status: stable
course: Linear Algebra
domain: UG
type: note
---

# Graphs, networks, incidence matrices

## Graphs and networks
![[Pasted image 20260629204326.png]]
## 1. Incidence Matrix

对于一个有 $n$ 个节点、$m$ 条边的有向图：

$$
A\in\mathbb R^{m\times n}
$$

- 每一列对应一个节点
- 每一行对应一条边
- 对于边 $i\to j$：
  - 起点 $i$ 对应 $-1$
  - 终点 $j$ 对应 $1$
  - 其余元素为 $0$
Incidence matrix 通常是 sparse matrix。
---
## 2. 节点量与边量

令 $x$ 表示节点电势，则：
$$
e=Ax
$$
**其中 $e$ 表示各条边两端的电势差**。
因此，$A$ 的作用是：
$$
\text{节点上的量}\longrightarrow\text{边上的差值}
$$
- "find the difference"
---
## 3. Nullspace：未知量的自由度
$$
Ax=0
$$
表示所有边上的电势差均为零。
若图连通，则所有节点电势相同：
$$
x=c \begin{bmatrix}
1 \\
. \\
. \\
. \\
1
\end{bmatrix}
$$
因此：
$$
N(A)=\operatorname{span}\begin{bmatrix}
1 \\
. \\
. \\
. \\
1
\end{bmatrix}
$$
$$
\dim N(A)=1
$$
**物理意义：所有节点电势同时加上同一个常数，不会改变电势差。**
选择一个节点接地，相当于固定参考电势，从而消除这个自由度。

---
## 4. Rank 与自由度 -> 对应 A 矩阵

- 这里的“自由度”指：存在一种改变x的方式，不会改变方程的输出 **= Nul(A)**
	- 在其他问题中，nullspace 可能表示：
		- 机器人机构中不会造成末端运动的关节运动；
		- 结构力学中不会产生外部效果的内部运动；
		- 测量系统中无法从传感器输出识别的状态；
		- 控制系统中不可观测的状态方向。
#### related [[分析物理]]

**对于连通图：**
$$
\operatorname{rank}(A)=n-1
$$
一般地，若：
$$
A\in\mathbb R^{m\times n}
$$
$$
\operatorname{rank}(A)=r
$$
则：
$$
\dim N(A)=n-r
$$
$n-r$ 表示未知量中未被方程确定的自由度。

---
## 5. 回路与方程冗余 -> 对应 $A^T$ 矩阵

- $A^T \mathbf{y} = \mathbf{0}$ 实际上是在约定了一个**方程** =0，也就是约束， 也是**线性相关** = $Nul(A^T)$
- 每当有一个独立的loop，就是多了一个约束
eg.
闭合回路上的电势差之和为零：
$$
(x_2-x_1)+(x_3-x_2)+(x_1-x_3)=0
$$
因此，图中的回路对应矩阵行之间的**线性相关。**
$$
\dim N(A^T)=m-r
$$
$m-r$ 表示方程之间独立的冗余关系数量，也对应独立回路数量。
对于连通图：
$$
\#\text{loops}=m-(n-1)
$$
即：
$$
n-m+\#\text{loops}=1
$$

---
## 6. Kirchhoff Current Law
令 $y$ 表示各条边上的**电流**，则：
$$
A^Ty=0
$$
表示每个节点的总流入电流等于总流出电流。
因此，$N(A^T)$ 可以理解为满足节点电流守恒的回路电流空间。

---
## 7. 方程、未知量与冗余

对于：
$$
Ax=b
$$
其中：
$$
A\in\mathbb R^{m\times n}
$$
设：
$$
\operatorname{rank}(A)=r
$$
则：
- $m$：写出的方程数量
- $n$：未知量数量
- $r$：独立方程数量
- $n-r$：未知量自由度
- $m-r$：方程冗余关系数量
核心结论
$$
\text{方程数量不等于独立信息数量，真正要看 rank}
$$

---
## 8. 可解性
$$
Ax=b
$$
有解当且仅当：
$$
b\in\operatorname{Col}(A)
$$
等价地，对于所有：
$$
y\in N(A^T)
$$
必须满足：
$$
y^Tb=0
$$
即 $b$ 必须满足所有由方程冗余关系产生的兼容条件。

- $b\notin\operatorname{Col}(A)$：无解
- $b\in\operatorname{Col}(A)$ 且 $n-r=0$：唯一解
- $b\in\operatorname{Col}(A)$ 且 $n-r>0$：无穷多解

---

## 9. 电完整关系

节点电势到电势差：
$$
e=Ax
$$
Ohm's Law：
$$
y=Ce
$$
Kirchhoff Current Law：
$$
A^Ty=f
$$
合并得到：
$$
A^TCAx=f
$$
其中：
- $x$：节点电势
- $e$：边上的电势差
- $y$：边上的电流
- $C$：电导矩阵
- $f$：外部注入节点的电流


# Related

[[Column Space]]