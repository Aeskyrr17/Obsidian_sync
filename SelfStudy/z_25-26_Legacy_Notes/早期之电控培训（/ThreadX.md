---

---
- 

```c++
#define TX_NAME(s) const_cast<CHAR*>(s)

```

> 把字符串 `s` 强制转换成 `CHAR*`（可写指针），即使 `s` 原本是 const 的。

> [!note]+ 具体语法
> ### ① `#define TX_NAME(s)`
> 
> 这是一个宏：
> 
> - 名字叫 `TX_NAME`
> - 传入一个参数 `s`
> 
> 使用方式：
> 
> ```c++
> TX_NAME("Semaphore")
> 
> 
> ```
> 
> ---
> 
> ### ② `const_cast<CHAR*>(s)`
> 
> 这是 C++ 的一个强制类型转换操作符。
> 
> const_cast 的作用：
> 
> > 去掉 const / volatile 修饰符，但不改变底层类型。
> 
> 即：
> 
> ```plain text
> const char*  →  char*
> const CHAR*  →  CHAR*
> 
> 
> ```
> 
> 示例：
> 
> ```c++
> const char* str = "ABC";
> char* p = const_cast<char*>(str);
> 
> 
> ```
> 
> ---
> 
> # ⭐ 为什么 ThreadX 必须用这个？
> 
> ThreadX 的 API，例如：
> 
> ```c
> tx_semaphore_create(&sem, CHAR *name, ...)
> 
> 
> ```
> 
> 要求 `name` 参数必须是可写的 `CHAR*`：
> 
> ```plain text
> CHAR* name  // 非 const
> 
> 
> ```
> 
> 但你一般传的是：
> 
> ```c++
> "Semaphore"
> 
> 
> ```
> 
> 它的类型是 `const char *`。
> 
> 所以 C++ 会报错，ThreadX 写法本质是来自 C 语言（C 没有 const 保护/限制），因此 C++ 需要 cast。
> 
> ---
> 
> # ⭐ 所以最终这个宏做了什么？
> 
> ### 输入：
> 
> ```c++
> TX_NAME("Semaphore")
> 
> 
> ```
> 
> ### 展开：
> 
> ```c++
> const_cast<CHAR*>("Semaphore")
> 
> 
> ```
> 
> ### 输出类型：
> 
> ```plain text
> CHAR*
> 
> 
> ```
> 
> 可以正常传给 ThreadX。
