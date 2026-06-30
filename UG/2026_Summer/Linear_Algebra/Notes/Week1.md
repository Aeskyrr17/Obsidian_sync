---
type: note
course: Linear Algebra
status: stable
domain: UG
---

# 总框架

Let
$$
	A \in \mathbb{R}^{m\times n} \qquad  x \in \mathbb{R}^n \qquad b \in \mathbb{R}^n
$$
注意区别
1. pivot column: 讨论哪一个变量是基本变量
2. pivot row / every row has a pivot : 讨论**输出空间**能否被完全覆盖

### 1. 先判断consistent，再考虑free variable

1. Consistent
	1. 一个系统至少有一个解
		1. 一个解
		2. 无穷多个解
2. inconsistent
	1. 没有任何解

### 2. 其次系统与free variable
对于
$$
	\mathbf{A}x = 0
$$
1. 没有free variable -> 只有 **trivial solution**
2. 有free cariable -> 有 **Nontrivial solution**

### 3. column & pivot ： 控制解是否唯一

如果**每一列都有pivot** $\implies_{}$ 没有free variable $\implies$ 只有trivial solution $\implies {a_{1}, a_{2}}. ..  a_{n}$ is **linearly independent**
$$
\boxed{
\begin{aligned}
&\text{每一列都有 pivot}\\
\Longleftrightarrow{}&
\text{无 free variable}\\
\Longleftrightarrow{}&
Ax=0\text{ 只有 trivial solution}\\
\Longleftrightarrow{}&
\text{columns of }A\text{ linearly independent}\\
\Longleftrightarrow{}&
T(x)=Ax\text{ one-to-one}
\end{aligned}}$$
### 4. row & pivot: 控制解是否存在

#### Span $\mathbb R^m$ 到底是什么意思
$$
\operatorname{Span}\{a_1,\dots,a_n\}=\mathbb R^m
$$
意思是：
**任意给一个$b\in\mathbb R^m$，都可以找到一组系数 $x_1,\dots,x_n$​，使得**

$$x_1a_1+\cdots+x_na_n=b.$$

$=>$ $Ax=b$ 有解。

因此：
$$
\text{columns of }A\text{ span }\mathbb R^m
$$
等价于：
$$
Ax=b\text{ 对每个 }b\in\mathbb R^m\text{ 都 consistent}
$$
再等价于：
$$
A\text{ 的每一行都有 pivot}
$$