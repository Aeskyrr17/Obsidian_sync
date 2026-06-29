---

---
- **工具链配置 Dec14 + build阶段debug**
> [!note]+ 先明确各个工具名字：
> - GCC-compiler 转换为.o文件
> - GNU-二进制工具集 包含.ld, .as, .ar等 生成.elf
> - GDB-调试器Debug用 
> - arm-none-eabi：就是一个被配置为针对 **ARM 裸机目标**、使用 **EABI 标准** 来生成代码的 **GNU 编译器**。

    - 目前的解决方法：直接在PATH里把arm放在了mingw64前面
        - 还没解决mingw64和arm交叉编译链冲突问题
    1. **fatal error: tx_api**==**.h**==**: No such file or directory**
        - include directory
    2. **undefined reference to 'function_name’**
        - 编译能过，但是链接阶段找不到对应.o文件
        1. 检查编译：检查是不是所有源文件(.c .cpp .S)都被==target_sources==包含
        2. 库（`libxxx.a`）有没有被`target_link_libraries` 链接到最终的可执行文件
