---
share_link: https://share.note.sx/zafobawm#Tv/GFNQQsbjjKFvjGhKFSWEg6gpl1ebTTCHa36gKFcU
share_updated: 2026-06-15T01:32:13+08:00
---
请确保下列软件的安装路径**不能有中文或空格**。建议统一安装到类似 `D:\Tools\xxx` 的目录。

> 面向对象：第一次配置 STM32 嵌入式开发环境的新同学。  
> 目标：能够生成工程、编译工程、下载/调试程序，并能使用基础调试工具观察变量。

# 0. 基础知识

## 0.1 工具链是做什么的？

STM32 开发通常需要下面几类工具：

- **工程配置工具**：配置芯片型号、时钟、GPIO、串口、定时器等外设，并生成初始化代码。
- **编译构建工具**：把 C/C++ 代码编译、链接成单片机可以运行的 `.elf` / `.hex` / `.bin` 文件。
- **下载与调试工具**：把程序烧录进单片机，并支持断点、单步、查看变量。
- **版本管理工具**：管理代码历史，便于多人协作。

## 0.2 PATH 是什么？为什么要配置？

`PATH` 是系统环境变量。把工具所在目录加入 `PATH` 后，就可以在终端中直接使用命令，例如：

```powershell
git --version
cmake --version
arm-none-eabi-gcc --version
openocd --version
```

如果命令提示“找不到”，通常说明：

1. 软件没有安装成功；
2. 没有加入 `PATH`；
3. 加入了错误的目录；
4. 修改 `PATH` 后没有重启终端或 IDE。

配置完环境变量后，建议重启 PowerShell、VS Code / CLion，再进行验证。

# 1. 开发工具

## 1.1 STM32CubeMX

用途：图形化配置 STM32 芯片外设，并生成工程代码。

- 官网：[STM32CubeMX](https://www.st.com/en/development-tools/stm32cubemx.html#get-software)
- 版本建议：最新版，或至少 6.15+
- 注意：下载 **STM32CubeMX**，不是 STM32CubeMX 2。

## 1.2 Git

用途：代码版本管理和团队协作。

- 官网：[Git Downloads](https://git-scm.com/downloads)
安装后验证：
```powershell
git --version
```
建议完成基础配置（在powershell中运行）：
```powershell
git config --global user.name "你的名字"
git config --global user.email "你的邮箱"
```

## 1.3 编辑器 / IDE
### VS Code
- 官网：[VS Code Download](https://code.visualstudio.com/download)

需要安装插件：
- CMake Tools
- Cortex-Debug
- C/C++
可选：
- clangd
- Doxygen
- ......

# 2. 编译与构建
## 2.1 CMake
用途：管理工程构建流程，生成编译命令。
- 官网：[CMake Download](https://cmake.org/download/)
- 建议下载 3.31 固定版本

安装时建议勾选：
- `Add CMake to the system PATH for all users`  
  或  
- `Add CMake to the system PATH for current user`

安装后验证：
```powershell
cmake --version
```
如果没有自动加入 `PATH`，手动添加 CMake 的 `bin` 目录，例如：
```text
D:\Tools\CMake\bin
```

## 2.2 Ninja

用途：作为 CMake 的后端构建工具，负责实际执行编译任务。相比传统 Makefile，Ninja 更轻量，构建速度更快。

- 安装包：待定

安装后需要把 `ninja.exe` 所在目录加入 `PATH`，例如：

```text
D:\Tools\Ninja
```

安装后验证：

```powershell
ninja --version
where ninja
```

如果已经安装 STM32CubeCLT 或 CLion，可能已经自带 Ninja。此时需要用 `where ninja` 确认实际调用的是哪个版本。

## 2.3 GNU Arm Embedded Toolchain

用途：编译 ARM Cortex-M 单片机程序。
- 官网：[Arm GNU Toolchain](https://developer.arm.com/downloads/-/arm-gnu-toolchain-downloads)

Windows 下通常下载 `arm-gnu-toolchain-...-mingw-w64-i686-arm-none-eabi` 相关安装包或压缩包。
需要加入 `PATH` 的目录是工具链的 `bin` 目录，例如：
```text
D:\Tools\ArmGNU\bin
```
安装后验证：
```powershell
arm-none-eabi-gcc --version
arm-none-eabi-g++ --version
arm-none-eabi-gdb --version
```

# 3. 下载与调试
## 3.1 OpenOCD
用途：通过 ST-Link、DAPLink、J-Link 等调试器下载和调试程序。
- 

安装后需要把 OpenOCD 的 `bin` 目录加入 `PATH`，例如：
```text
D:\Tools\OpenOCD\bin
```
验证：
```powershell
openocd --version
```

## 3.2 Ozone
官网：https://www.segger.com/products/development-tools/ozone-j-link-debugger/
云盘：[Ozone_Windows_V324_x86.exe](https://zanpw3z2hb6.feishu.cn/file/B04Jb6E69oP9ocxhPEXc2fsMnle)
**注意：应该先安装ozone，再安装JLink**
![566](https://zanpw3z2hb6.feishu.cn/space/api/box/stream/download/asynccode/?code=ODk1ZTg3ZjQyMjI1NjRjMTY2ZjFmZDg4ZjIyY2IxNTRfOVN0VUNBcTZxRXFadXlhQ3RLOVlFcXVnRW55OExOdnhfVG9rZW46S0RKVWJZVUVwb0tEQUt4ak5nM2NxTzRtbjJiXzE3ODE0NTc3NTM6MTc4MTQ2MTM1M19WNA&add_watermark=true&scene_type=CCM)
- 记住勾选Install a new instance
    
## 3.3 JLink驱动（安装版本7.22b 32bit）
官网：[J-Link Downloads](https://www.segger.com/downloads/jlink/)
云盘：[JLink_Windows_V722b.exe](https://zanpw3z2hb6.feishu.cn/file/BMcSbzowdoSisuxGWymc7K0vnyc)

![531](https://zanpw3z2hb6.feishu.cn/space/api/box/stream/download/asynccode/?code=MmZlYWNlMDVhMTExZTJmY2IwN2U2M2Y3N2JhOTYxMDdfSENMQzFPcm02azFVZWR0ejZxR0t6ZU1GczVOWWN1UURfVG9rZW46TUk5UGJ0eG1xb2JHUnN4emw4RGNmUzM3blRlXzE3ODE0NTc3NTM6MTc4MTQ2MTM1M19WNA&add_watermark=true&scene_type=CCM)
- 注意不要勾选update dll in other application，否则jlink会把ozone里面老的驱动和启动项替代掉。choose destination和ozone一样，选择install a new instance。如果安装了老的相同版本的jlink，请先卸载（版本相同不用管，直接新装一个）。

## 3.4 FreeMASTER
一款可以实时查看曲线和进行动态调参的软件，非常好用
在网站中找到 FreeMASTER_Serial_Communication_Driver_V2.0 和 FreeMASTER 3.2 安装，需要注册NXP账号，然后直接安装即可
- 官网：
- https://www.nxp.com.cn/design/design-center/software/development-software/freemaster-run-time-debugging-tool:FREEMASTER

# 4. 推荐检查流程

安装完成后，打开新的 PowerShell，依次执行：

```powershell
git --version
cmake --version
arm-none-eabi-gcc --version
arm-none-eabi-g++ --version
arm-none-eabi-gdb --version
openocd --version
ninja --version
where git
where cmake
where ninja
where arm-none-eabi-gcc
where openocd
```

如果都能正常输出版本号，说明基础工具链基本配置完成。

# 5. 常见问题

## 5.1 命令行找不到工具

优先检查：

1. 软件是否安装成功；
2. 是否把正确的 `bin` 目录加入 `PATH`；
3. 是否重启终端 / IDE；
4. `where 工具名` 输出的路径是否符合预期。

## 5.2 工程路径导致编译失败

避免使用：

- 中文路径；
- 带空格路径；
- 过深路径；
- OneDrive、微信、QQ 等同步目录。

推荐工程路径示例：

```text
D:\RM\project_name
D:\Workspace\project_name
```

## 5.3 同一个工具安装了多个版本

使用下面命令检查实际调用路径：

```powershell
where cmake
where ninja
where arm-none-eabi-gcc
where openocd
```

如果输出多个路径，系统会优先使用排在最上面的版本。必要时调整 `PATH` 顺序。

# 6. 说明

完成工具链配置只是为了避免开发过程中的环境问题。实际开发时，建议优先使用战队提供的模板工程，不建议每次从零开始搭建工程。
