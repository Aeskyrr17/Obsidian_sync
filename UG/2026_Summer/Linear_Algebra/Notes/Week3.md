---
status: developing
course: Linear Algebra
type: note
domain: UG
---
# determinants

## 计算方法

# properties of determinants

- column 与 row 对det的规则一样

- 不改变det：
	1. elementary row exchange(一行加上另一行的倍数)
- det = -det
	- 交换两行
		- 每进行一次交换就乘一个负号
- det = k det
	- 把某一行乘一个倍数
		-  把n行乘k就是 $det = k^n det$

$$
\begin{align}
	\det(A^T) &= \det(A) \\ \\
	\det(AB) &= \det(A)\det(B) \\ \\
	\det(A^{-1}) &= \frac{1}{\det(A)}
\end{align}

$$

## 用Elimination 把matrix化成upper triangular然后计算det
- 最好只用row replacement & row exchange

1. if invertile，对角线都有pivot， det $\neq$ 0
2. 三角矩阵的det是：对角线元素相乘

# determinant 的应用

## 1. Cramer's Rule解方程

- 对于方程组$A\mathbf{x} = \mathbf{b}$ (invertible)
$$
	x_{i} =  \frac{detA_{i}(\mathbf{b})}{\det A}
$$
$\det A_{i}(\mathbf{b})$ : 把第n列换成$\mathbf{b}$

## 2. Adjugate(伴随矩阵)求inverse

- cofactor $C_{ij} = (-1)^{i+j}\det A_{ij}$
$$
	\begin{align}
	C = \begin{bmatrix}
	C_{11} & C_{12}  & C_{13}  & \dots \\
	C_{21} & C_{22}  & \dots & \dots \\
	\dots & \dots & \dots & \dots
	\end{bmatrix}
	\end{align}
$$
$$
	adj A = C^T
$$
$$
	A^{-1} = \frac{1}{\det A} adj A
$$
重要恒等式：
$$
	\mathbf{A} adj \mathbf{A} = \det \mathbf{A} \mathbf{I}
$$
- 矩阵乘上自己的adjugate，会得到det倍的单位矩阵