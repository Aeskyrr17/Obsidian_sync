# Graphs, networks, incidence matrices

## Graphs and networks
![[Pasted image 20260629204326.png]]

## Incidence matrices
# Graphs, Networks & Incidence Matrices

## 1. Incidence Matrix

对于一个有 (n) 个节点、(m) 条边的有向图，incidence matrix

$$
A\in\mathbb R^{m\times n}  
$$

- 每一列对应一个节点
    
- 每一行对应一条边
    
- 边 $(i\to j)$ 对应的行：
    
    - 起点为 (-1)
        
    - 终点为 (1)
        
    - 其他为 (0)
        

矩阵通常是 **sparse matrix**。

---

## 2. 节点量与边量

令

$$
x=  
\begin{bmatrix}  
x_1&\cdots&x_n  
\end{bmatrix}^T  
$$

表示节点电势，则

$$[  $$
e=Ax  
]

表示各条边两端的电势差。

因此，(A) 将：

[  
\text{节点上的量}\longrightarrow\text{边上的差值}  
]

---

## 3. Nullspace：未知量的自由度

[  
Ax=0  
]

表示所有边上的电势差均为零。

若图连通，则所有节点电势相等：

[  
x=c  
\begin{bmatrix}  
1\  
\vdots\  
1  
\end{bmatrix}  
]

因此：

[  
N(A)=\operatorname{span}  
\left{  
\begin{bmatrix}  
1\  
\vdots\  
1  
\end{bmatrix}  
\right}  
]

[  
\dim N(A)=1  
]

物理意义：所有节点电势同时加上同一个常数，不会改变电势差。选一个节点接地，可以消除该自由度。

---

## 4. Rank 与自由度

对于连通图：

[  
\operatorname{rank}(A)=n-1  
]

一般地，若

[  
A\in\mathbb R^{m\times n},\qquad \operatorname{rank}(A)=r  
]

则：

[  
\dim N(A)=n-r  
]

表示未知量中未被方程确定的自由度。

---

## 5. 回路与方程冗余

闭合回路上的电势差之和为零，例如：

[  
(x_2-x_1)+(x_3-x_2)+(x_1-x_3)=0  
]

因此，回路对应矩阵行之间的线性相关。

[  
\dim N(A^T)=m-r  
]

表示方程之间独立的冗余关系数量，也对应图中的独立回路数量。

对于连通图：

[  
#\text{loops}=m-(n-1)  
]

即：

[  
n-m+#\text{loops}=1  
]

---

## 6. Kirchhoff Current Law

令

[  
y\in\mathbb R^m  
]

表示各条边上的电流，则：

[  
A^Ty=0  
]

表示每个节点的总流入电流等于总流出电流。

因此：

[  
N(A^T)  
]

可以理解为满足节点电流守恒的回路电流空间。

---

## 7. 方程、未知量与冗余

对于

[  
Ax=b,\qquad A\in\mathbb R^{m\times n}  
]

|数量|含义|
|---|---|
|(m)|写出的方程数量|
|(n)|未知量数量|
|(r)|独立方程数量|
|(n-r)|未知量自由度|
|(m-r)|方程冗余关系数量|

核心：

[  
\boxed{\text{方程数量不等于独立信息数量，真正要看 rank}}  
]

---

## 8. (Ax=b) 的可解性

[  
Ax=b  
]

有解当且仅当：

[  
b\in\operatorname{Col}(A)  
]

等价地，对所有

[  
y\in N(A^T)  
]

必须满足：

[  
y^Tb=0  
]

即右侧 (b) 必须满足所有由冗余方程产生的兼容条件。

- (b\notin\operatorname{Col}(A))：无解
    
- (b\in\operatorname{Col}(A))，且 (n-r=0)：唯一解
    
- (b\in\operatorname{Col}(A))，且 (n-r>0)：无穷多解
    

---

## 9. 电路完整关系

[  
e=Ax  
]

[  
y=Ce  
]

[  
A^Ty=f  
]

合并得到：

[  
A^TCAx=f  
]

其中：

- (x)：节点电势
    
- (e)：边上的电势差
    
- (y)：边上的电流
    
- (C)：电导矩阵
    
- (f)：外部注入节点的电流