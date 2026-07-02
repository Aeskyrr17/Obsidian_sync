---
domain: SelfStudy
status: developing
type: note
---
# 1. Git Submodule

用于再一个Git仓库中引用另一个独立的Git仓库

pnx_template 
	├── pnx_bsp 
	├── pnx_devices 
	├── pnx_libraries 
	├── pnx_modules 
	└── pnx_os

主仓库会保存
1. sub的仓库地址
2. 主仓库当前指定的submodule的commit

常记录在`.gitmodules`文件中，例如
```
[submodule "pnx_bsp"] 
	path = pnx_bsp 
	url = git@github.com:HKUSTGZ-ROBOMASTER-PNX/pnx_bsp.git
```

## 2. 首次拉取 Submodule

如果主仓库已经克隆下来，但子模块还是空的：
```bash
git submodule update --init --recursive
```
- `--init`：初始化尚未初始化的子模块
- `--recursive`：继续处理嵌套的子模块

这个命令会把子模块拉到 **主仓库当前记录的指定 commit** (不是sub的origin的最新)
## 3. 克隆时直接拉取 Submodule

推荐使用：

```bash
git clone --recurse-submodules 仓库地址
```

这样会同时克隆主仓库和所有子模块。

如果已经普通克隆：

```bash
git clone 仓库地址
```

再执行：

```bash
git submodule update --init --recursive
```

---

## 4. 更新主仓库后同步 Submodule

常用流程：

```bash
git pull
git submodule sync --recursive
git submodule update --init --recursive
```

其中：

```bash
git submodule sync --recursive
```

用于把 `.gitmodules` 中的子模块地址同步到本地配置。

当子模块 URL 被修改后，这一步尤其重要。

---

## 5. 更新到子模块远程仓库最新版本

如果想让所有子模块更新到其远程分支的最新 commit：

```bash
git submodule update --remote --recursive
```

区别：

```bash
git submodule update --init --recursive
```

表示：

> 更新到主仓库指定的版本

而：

```bash
git submodule update --remote --recursive
```

表示：

> 更新到子模块远程分支的最新版本

不要把这两个命令混为一谈。

---

## 6. 更新 Submodule 后为什么要提交

主仓库记录的是子模块的 commit 指针。

执行：

```bash
git submodule update --remote --recursive
```

之后，子模块 commit 可能发生变化。

检查：

```bash
git status
```

可能看到：

```text
modified: pnx_bsp
```

这不一定表示子模块内部文件被修改，而可能表示：

> 子模块当前 commit 与主仓库记录的 commit 不同

提交新的子模块指针：

```bash
git add pnx_bsp
git commit -m "Update pnx_bsp submodule"
```

多个子模块可以一起添加：

```bash
git add pnx_bsp pnx_devices pnx_libraries pnx_modules pnx_os
git commit -m "Update submodules"
```

---

## 7. `path/to/submodule` 是占位符

命令示例：

```bash
git add path/to/submodule
```

不能原样输入。

应替换为真实子模块路径，例如：

```bash
git add pnx_bsp
```

如果直接输入：

```bash
git add path/to/submodule
```

会出现：

```text
pathspec did not match any files
```

---

## 8. 查看 Submodule 状态

```bash
git submodule status
```

常见结果：

```text
 abcdef123 pnx_bsp
```

表示子模块已初始化，并处于主仓库记录的 commit。

如果前面有 `-`：

```text
-abcdef123 pnx_bsp
```

表示子模块尚未初始化。

如果前面有 `+`：

```text
+abcdef123 pnx_bsp
```

表示当前子模块 commit 与主仓库记录的不一致。

---

## 9. 主仓库正常但 Submodule 拉取失败

主仓库和子模块可能使用不同协议。

例如主仓库使用 HTTPS：

```text
https://github.com/HKUSTGZ-ROBOMASTER-PNX/pnx_template.git
```

而子模块使用 SSH：

```text
git@github.com:HKUSTGZ-ROBOMASTER-PNX/pnx_bsp.git
```

因此可能出现：

```text
主仓库 git pull 正常
子模块 clone 失败
```

检查主仓库地址：

```bash
git remote -v
```

查看子模块地址：

```powershell
Get-Content .gitmodules
```

这种情况下，应检查 SSH 配置，而不是继续重复执行 submodule 命令。

---

## 10. 单独测试子模块仓库

在拉取全部子模块前，可以单独测试某个仓库：

```powershell
git ls-remote git@github.com:HKUSTGZ-ROBOMASTER-PNX/pnx_bsp.git
```

如果输出一系列 commit hash，说明：

- 仓库存在
    
- SSH 身份认证成功
    
- 当前账号具有读取权限
    

如果 SSH 本身失败，则需要先处理 SSH 配置。

---

## 11. 拉取失败后的重试

SSH 或网络问题解决后，通常直接重新执行即可：

```bash
git submodule sync --recursive
git submodule update --init --recursive
```

Git 会继续拉取之前失败的子模块。

如果出现：

```text
destination path already exists and is not an empty directory
```

说明之前失败时可能留下了残余目录，需要再检查或清理对应子模块目录。

不要在没有必要时直接删除所有子模块。

---

## 12. 常用命令速查

### 首次初始化

```bash
git submodule update --init --recursive
```

### 克隆主仓库和子模块

```bash
git clone --recurse-submodules 仓库地址
```

### 更新主仓库后同步

```bash
git pull
git submodule sync --recursive
git submodule update --init --recursive
```

### 更新子模块到远程最新版本

```bash
git submodule update --remote --recursive
```

### 查看子模块状态

```bash
git submodule status
```

### 测试某个子模块仓库

```bash
git ls-remote 子模块仓库地址
```

---

## 13. 推荐日常工作流

一般更新项目：

```bash
git pull
git submodule sync --recursive
git submodule update --init --recursive
```

只有明确要升级子模块版本时，才执行：

```bash
git submodule update --remote --recursive
```

然后提交新的子模块 commit 指针：

```bash
git add 子模块路径
git commit -m "Update submodules"
```