# 

## 1. 创建矩阵

```matlab
A = [1 2 3; 4 5 6];      % 2×3 矩阵
v = [1 2 3];             % 行向量
v = [1; 2; 3];           % 列向量
```

空格或逗号表示**同一行的不同元素**，分号表示**换行**。

```matlab
A = [1, 2, 3;
     4, 5, 6];
```

---

## 2. 常用特殊矩阵

```matlab
zeros(3,4)        % 3×4 零矩阵
ones(3,4)         % 3×4 全 1 矩阵
eye(3)            % 3×3 单位矩阵
eye(3,4)          % 3×4 “单位矩阵”
rand(3,4)         % [0,1] 均匀随机矩阵
randn(3,4)        % 标准正态随机矩阵

diag([1 2 3])     % 对角矩阵
```

```matlab
size(A)           % 返回矩阵尺寸
size(A,1)         % 行数
size(A,2)         % 列数
length(A)         % 最大维度，不建议用于判断矩阵行列数
numel(A)          % 元素总数
```

---

## 3. 等差序列与采样

```matlab
1:5               % [1 2 3 4 5]
1:2:9             % [1 3 5 7 9]
5:-1:1            % [5 4 3 2 1]

linspace(0,10,6)  % 在 0 到 10 间生成 6 个等距点
```

一般形式：

```matlab
start:step:end
```

---

## 4. 矩阵索引

设：

```matlab
A = [1 2 3;
     4 5 6;
     7 8 9];
```

### 单个元素

```matlab
A(2,3)            % 第 2 行、第 3 列，结果为 6
```

### 整行、整列

```matlab
A(2,:)            % 第 2 行
A(:,3)            % 第 3 列
```

其中 `:` 表示该维度上的全部元素。

### 选择多个行列

```matlab
A([1 3],:)        % 第 1、3 行
A(:,[1 3])        % 第 1、3 列
A(1:2,2:3)        % 第 1~2 行、第 2~3 列
```

### 最后一个元素

```matlab
A(end,end)        % 右下角元素
A(end,:)          % 最后一行
A(:,end)          % 最后一列
A(1:end-1,:)      % 除最后一行外的所有行
```

### 线性索引

```matlab
A(1)              % 第 1 个元素
A(4)              % 第 4 个元素
A(:)              % 将矩阵按列展开为列向量
```

MATLAB 按照**列优先**顺序储存矩阵。

---

## 5. 修改、添加与删除元素

```matlab
A(2,3) = 10;      % 修改元素
A(:,2) = 0;       % 将第 2 列全部设为 0
```

添加行或列：

```matlab
A(end+1,:) = [10 11 12];   % 添加一行
A(:,end+1) = [13;14;15];   % 添加一列
```

删除行或列：

```matlab
A(2,:) = [];       % 删除第 2 行
A(:,3) = [];       % 删除第 3 列
```

---

## 6. 矩阵拼接

### 横向拼接

```matlab
C = [A B];
C = horzcat(A,B);
```

要求 `A` 和 `B` 的行数相同。

### 纵向拼接

```matlab
C = [A; B];
C = vertcat(A,B);
```

要求 `A` 和 `B` 的列数相同。

### 分块矩阵

```matlab
M = [A B;
     C D];
```

---

# 7. 矩阵运算与逐元素运算

这是 MATLAB 最重要的区别之一。

## 7.1 加法与减法

```matlab
C = A + B;
C = A - B;
```

要求 `A` 和 `B` 尺寸相同，或满足隐式扩展条件。

---

## 7.2 矩阵乘法

```matlab
C = A * B;
```

若：

```text
A: m×n
B: n×p
```

则：

```text
C: m×p
```

---

## 7.3 逐元素乘法

```matlab
C = A .* B;
```

每个位置对应相乘：

```text
C(i,j) = A(i,j) × B(i,j)
```

通常要求尺寸相同。

---

## 7.4 矩阵除法

### 左除：求线性方程

```matlab
x = A \ b;
```

表示求解：

```text
Ax = b
```

这是 MATLAB 中求解线性方程组的推荐方式。

不要优先写：

```matlab
x = inv(A) * b;
```

---

### 右除

```matlab
X = A / B;
```

表示求解：

```text
XB = A
```

可理解为：

```matlab
A / B ≈ A * inv(B)
```

但 MATLAB 会使用数值上更稳定的算法。

---

## 7.5 逐元素除法

```matlab
C = A ./ B;       % A 的元素分别除以 B 的对应元素
C = A .\ B;       % B 的元素分别除以 A 的对应元素
```

```text
A ./ B = A(i,j) / B(i,j)
A .\ B = B(i,j) / A(i,j)
```

---

## 7.6 幂运算

### 矩阵幂

```matlab
B = A^2;
```

等价于：

```matlab
B = A * A;
```

要求 `A` 是方阵。

### 逐元素幂

```matlab
B = A.^2;
```

表示：

```text
B(i,j) = A(i,j)^2
```

---

## 7.7 运算符对照表

|矩阵运算|逐元素运算|含义|
|---|---|---|
|`A * B`|`A .* B`|矩阵乘法 / 对应元素乘法|
|`A / B`|`A ./ B`|矩阵右除 / 对应元素右除|
|`A \ B`|`A .\ B`|矩阵左除 / 对应元素左除|
|`A ^ n`|`A .^ n`|矩阵幂 / 对应元素幂|

记忆方法：

> 点号 `.` 表示“每个元素各算各的”。

加减法不需要点：

```matlab
A + B
A - B
```

因为矩阵加减本身就是对应元素运算。

---

# 8. 转置与共轭转置

```matlab
A.'               % 普通转置
A'                % 共轭转置
transpose(A)       % 普通转置
ctranspose(A)      % 共轭转置
```

对于实数矩阵：

```matlab
A.' == A'
```

对于复数矩阵，两者不同：

```matlab
A = [1+2i, 3+4i];

A.'                % 只交换行列
A'                 % 交换行列并取复共轭
```

---

# 9. 矩阵常用性质

```matlab
det(A)             % 行列式
rank(A)            % 秩
trace(A)           % 迹：对角元素之和
inv(A)             % 逆矩阵
pinv(A)            % Moore–Penrose 伪逆
cond(A)            % 条件数
rcond(A)           % 倒数条件数估计
```

建议：

```matlab
x = A \ b;
```

而不是：

```matlab
x = inv(A) * b;
```

只有在确实需要逆矩阵本身时才使用 `inv(A)`。

---

## 10. 对角线与三角矩阵

```matlab
diag(A)            % 提取 A 的主对角线
diag(A,1)          % 提取主对角线上方第 1 条对角线
diag(A,-1)         % 提取主对角线下方第 1 条对角线

diag(v)            % 将向量 v 放到主对角线上
diag(v,1)          % 将 v 放到上方第 1 条对角线上
```

```matlab
triu(A)            % 上三角部分
triu(A,1)          % 严格上三角部分
tril(A)            % 下三角部分
tril(A,-1)         % 严格下三角部分
```

---

# 11. 行列式、秩、逆与线性方程

```matlab
d = det(A);
r = rank(A);
A_inv = inv(A);
```

求解：

```matlab
x = A \ b;
```

多个右端项：

```matlab
X = A \ B;
```

此时相当于同时求解：

```text
AX = B
```

---

## 12. 行最简形式

```matlab
R = rref(A);       % Reduced Row Echelon Form
```

增广矩阵：

```matlab
Aug = [A b];
R = rref(Aug);
```

注意：`rref` 适合学习、符号分析和小规模问题，但在数值计算中通常不使用它直接求解方程。

---

# 13. 特征值与特征向量

```matlab
[V,D] = eig(A);
```

满足：

```matlab
A * V = V * D;
```

其中：

- `D`：对角矩阵，对角线是特征值。
    
- `V`：每一列是对应的特征向量。
    

只求特征值：

```matlab
lambda = eig(A);
```

---

# 14. 常用矩阵分解

## LU 分解

```matlab
[L,U] = lu(A);
```

一般情况下：

```matlab
[L,U,P] = lu(A);
```

满足：

```matlab
P * A = L * U;
```

---

## QR 分解

```matlab
[Q,R] = qr(A);
```

满足：

```matlab
A = Q * R;
```

---

## 奇异值分解 SVD

```matlab
[U,S,V] = svd(A);
```

满足：

```matlab
A = U * S * V';
```

---

## Cholesky 分解

```matlab
R = chol(A);
```

对于对称正定矩阵：

```matlab
A = R' * R;
```

---

# 15. 重塑矩阵

## reshape

```matlab
B = reshape(A,m,n);
```

将 `A` 重塑为 `m×n` 矩阵，元素总数必须保持不变。

```matlab
A = [1 2 3;
     4 5 6];

B = reshape(A,3,2);
```

注意 MATLAB 按列读取和填充元素。

---

## 拉直为向量

```matlab
v = A(:);          % 转成列向量
v = A(:).';        % 转成行向量
```

---

## 交换维度

```matlab
B = permute(A,[2 1 3]);
```

二维矩阵交换行列通常直接用：

```matlab
B = A.';
```

---

## squeeze

```matlab
B = squeeze(A);
```

删除长度为 1 的维度。

---

# 16. 翻转与旋转

```matlab
fliplr(A)          % 左右翻转
flipud(A)          % 上下翻转
flip(A,1)          % 沿第 1 维翻转
flip(A,2)          % 沿第 2 维翻转

rot90(A)           % 逆时针旋转 90°
rot90(A,2)         % 旋转 180°
```

---

# 17. 排序

```matlab
sort(A)            % 每列升序排序
sort(A,1)          % 每列排序
sort(A,2)          % 每行排序

sort(A,'descend')  % 降序
```

同时获得排序后的索引：

```matlab
[B,index] = sort(A);
```

---

# 18. 最大值、最小值与求和

```matlab
sum(A)             % 每列求和
sum(A,1)           % 每列求和
sum(A,2)           % 每行求和
sum(A,'all')       % 所有元素求和
```

```matlab
max(A)             % 每列最大值
max(A,[],1)        % 每列最大值
max(A,[],2)        % 每行最大值
max(A,[],'all')    % 所有元素最大值
```

```matlab
min(A)
mean(A)            % 平均值
median(A)          % 中位数
prod(A)            % 乘积
std(A)             % 标准差
var(A)             % 方差
```

返回最大值及其位置：

```matlab
[value,index] = max(v);
```

二维矩阵全局最大值位置：

```matlab
[value,index] = max(A,[],'all');
[row,col] = ind2sub(size(A),index);
```

---

# 19. 向量运算

```matlab
dot(a,b)           % 点积
cross(a,b)         % 叉积，仅适用于三维向量
norm(a)            % 2-范数
norm(a,1)          % 1-范数
norm(a,inf)        % 无穷范数
```

归一化：

```matlab
a_unit = a / norm(a);
```

矩阵范数：

```matlab
norm(A)            % 默认矩阵 2-范数
norm(A,'fro')      % Frobenius 范数
```

---

# 20. 逻辑运算与筛选

## 比较运算

```matlab
A == B             % 等于
A ~= B             % 不等于
A > B
A < B
A >= B
A <= B
```

结果是逻辑矩阵：

```matlab
A > 0
```

---

## 逻辑运算

```matlab
A & B              % 逻辑与，逐元素
A | B              % 逻辑或，逐元素
~A                  % 逻辑非

a && b             % 短路与，只用于标量逻辑值
a || b             % 短路或，只用于标量逻辑值
```

---

## 逻辑索引

```matlab
A(A > 0)           % 提取所有大于 0 的元素
A(A < 0) = 0;      % 将所有负数改为 0
```

多个条件：

```matlab
A(A > 0 & A < 10)
```

注意每个条件最好加括号：

```matlab
(A > 0) & (A < 10)
```

---

## 查找元素位置

```matlab
index = find(A > 0);
```

二维坐标：

```matlab
[row,col] = find(A > 0);
```

---

# 21. 判断函数

```matlab
isempty(A)         % 是否为空
isscalar(A)        % 是否为标量
isvector(A)        % 是否为向量
ismatrix(A)        % 是否为矩阵
isrow(A)           % 是否为行向量
iscolumn(A)        % 是否为列向量
issquare(A)        % 是否为方阵，较新版本 MATLAB
```

数值判断：

```matlab
isnan(A)           % 是否为 NaN
isinf(A)           % 是否为 Inf
isfinite(A)        % 是否为有限值
isreal(A)          % 是否为实数
```

整体判断：

```matlab
all(A > 0,'all')   % 是否所有元素都大于 0
any(A > 0,'all')   % 是否至少有一个元素大于 0
```

---

# 22. 复数矩阵

```matlab
z = 3 + 4i;

real(z)            % 实部
imag(z)            % 虚部
abs(z)             % 模
angle(z)           % 相位，单位 rad
conj(z)            % 复共轭
```

复数矩阵转置：

```matlab
A.'                % 普通转置
A'                 % 共轭转置
```

---

# 23. 隐式扩展

设：

```matlab
A = [1 2 3;
     4 5 6];

v = [10 20 30];
```

则：

```matlab
A + v
```

MATLAB 会将 `v` 自动扩展到每一行：

```text
[11 22 33
 14 25 36]
```

列向量同理：

```matlab
u = [10;20];
A + u
```

结果：

```text
[11 12 13
 24 25 26]
```

旧版 MATLAB 常使用：

```matlab
bsxfun(@plus,A,v)
```

---

# 24. Kronecker 积与重复矩阵

```matlab
kron(A,B)          % Kronecker 积
repmat(A,m,n)      % 将 A 重复 m 行、n 列
```

例如：

```matlab
repmat([1 2],3,2)
```

---

# 25. 向量化计算

不推荐：

```matlab
for i = 1:length(x)
    y(i) = x(i)^2;
end
```

推荐：

```matlab
y = x.^2;
```

不推荐：

```matlab
for i = 1:size(A,1)
    for j = 1:size(A,2)
        C(i,j) = A(i,j) * B(i,j);
    end
end
```

推荐：

```matlab
C = A .* B;
```

---

# 26. 常见矩阵计算示例

## 计算 (y=Ax)

```matlab
y = A * x;
```

---

## 计算二次型 (x^TAx)

对于实数向量：

```matlab
q = x' * A * x;
```

明确使用普通转置：

```matlab
q = x.' * A * x;
```

---

## 求解 (Ax=b)

```matlab
x = A \ b;
```

---

## 最小二乘解

当 `A` 不是方阵时：

```matlab
x = A \ b;
```

如果方程超定，MATLAB 通常返回最小二乘解：

```text
minimize ||Ax-b||
```

---

## 计算 (A^{-1}BA^{-T})

```matlab
C = A \ B / A.';
```

通常比显式求逆更好。

---

## 计算 (J^TF)

```matlab
tau = J.' * F;
```

如果变量是实数，也常写：

```matlab
tau = J' * F;
```

---

# 27. 符号矩阵

需要 Symbolic Math Toolbox。

```matlab
syms x y
A = [x y;
     y x];
```

符号矩阵运算：

```matlab
det(A)
inv(A)
rank(A)
simplify(A)
expand(A)
factor(A)
```

符号求解线性方程：

```matlab
syms x y
eq1 = 2*x + y == 5;
eq2 = x - y == 1;

sol = solve([eq1,eq2],[x,y]);
```

符号矩阵求导：

```matlab
diff(A,x)
```

Jacobian：

```matlab
f = [x^2 + y;
     x*y];

q = [x;y];

J = jacobian(f,q);
```

代入：

```matlab
subs(A,x,2)
subs(A,[x y],[2 3])
```

符号转数值：

```matlab
A_num = double(subs(A,[x y],[2 3]));
```

---

# 28. 数值精度与相等判断

浮点数不建议直接判断：

```matlab
a == b
```

更稳妥：

```matlab
abs(a-b) < 1e-9
```

矩阵：

```matlab
norm(A-B,'fro') < 1e-9
```

或：

```matlab
max(abs(A-B),[],'all') < 1e-9
```

符号矩阵可以使用：

```matlab
isAlways(A == B)
```

---

# 29. 运算优先级

大致顺序：

1. 括号 `()`
    
2. 转置、幂：`'`、`.'`、`^`、`.^`
    
3. 正负号：`+A`、`-A`
    
4. 乘除：`*`、`/`、`\`、`.*`、`./`、`.\`
    
5. 加减：`+`、`-`
    
6. 比较：`==`、`~=`、`>`、`<`
    
7. 逻辑与或：`&`、`|`
    
8. 短路逻辑：`&&`、`||`
    

复杂表达式建议主动加括号：

```matlab
tau = J.' * (R * F);
```

---

# 30. 最常见易错点

## 易错 1：矩阵乘法和逐元素乘法

```matlab
A * B              % 矩阵乘法
A .* B             % 对应元素乘法
```

---

## 易错 2：矩阵平方和元素平方

```matlab
A^2                % A*A
A.^2               % 每个元素平方
```

---

## 易错 3：转置符号

```matlab
A'                 % 共轭转置
A.'                % 普通转置
```

处理实数时通常结果相同，处理复数时不同。

---

## 易错 4：矩阵索引从 1 开始

```matlab
A(1,1)             % 第一个元素
```

MATLAB 没有 `A(0,0)`。

---

## 易错 5：矩阵按列存储

```matlab
A(:)
reshape(A,m,n)
```

都按照列优先顺序进行。

---

## 易错 6：不要用 `inv(A)*b` 解方程

推荐：

```matlab
x = A \ b;
```

而不是：

```matlab
x = inv(A) * b;
```

---

## 易错 7：行向量和列向量不同

```matlab
v = [1 2 3];       % 1×3
v = [1;2;3];       % 3×1
```

例如：

```matlab
v * v'             % 标量
v' * v             % 3×3 矩阵
```

---

## 易错 8：`length(A)` 不等于行数

```matlab
length(A)          % max(size(A))
size(A,1)          % 行数
size(A,2)          % 列数
```

---

## 易错 9：判断矩阵是否相等

```matlab
A == B
```

返回的是逻辑矩阵，而不是单个真假值。

整体判断：

```matlab
isequal(A,B)
```

浮点近似判断：

```matlab
norm(A-B,'fro') < 1e-9
```

---

# 31. 高频速查表

```matlab
A(i,j)                 % 第 i 行第 j 列
A(i,:)                 % 第 i 行
A(:,j)                 % 第 j 列
A(:)                   % 按列展开
A(end,:)               % 最后一行
A(:,end)               % 最后一列

A * B                  % 矩阵乘法
A .* B                 % 逐元素乘法
A / B                  % 矩阵右除
A \ B                  % 矩阵左除
A ./ B                 % 逐元素除法
A ^ 2                  % 矩阵平方
A .^ 2                 % 元素平方

A'                     % 共轭转置
A.'                    % 普通转置

det(A)                 % 行列式
rank(A)                % 秩
inv(A)                 % 逆矩阵
pinv(A)                % 伪逆
trace(A)               % 迹
rref(A)                % 行最简形式

x = A \ b              % 解 Ax=b
[V,D] = eig(A)         % 特征值和特征向量
[L,U,P] = lu(A)        % LU 分解
[Q,R] = qr(A)          % QR 分解
[U,S,V] = svd(A)       % SVD

size(A)                % 尺寸
reshape(A,m,n)         % 重塑
diag(A)                % 提取对角线
diag(v)                % 创建对角矩阵
sum(A,2)               % 每行求和
sum(A,1)               % 每列求和
max(A,[],'all')        % 全局最大值

A(A > 0)               % 逻辑索引
find(A > 0)            % 查找位置
```

## 一句话记忆

> 不带点的 `* / \ ^` 通常表示线性代数中的矩阵运算；带点的 `.* ./ .\ .^` 表示每个元素分别运算。