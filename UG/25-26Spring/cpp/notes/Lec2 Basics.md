## Entry Point
- Code inside `main()` will be executed line by line.

## Data Types
- Number types
- Character type & String type
- Enum type
- Boolean type
- Void type
- Null pointer type

## `const`
- 避免 magic number：先用 `const xxx xxx = xxx;` 来定义
- `constexpr`：known at compile time

## 类型转换
- `static_cast<>()`
- Example:
```
int a = static_cast<int>(c);
```
![[Pasted image 20260320110639.png]]
## 运算 Tips

### Division

- 至少有一个 `float`，结果就是 `float`
- 如果都是 `int`，结果也是 `int`，会丢失精度
```
cout << (1.0 + 4) / 2 << "\n"; // 2.5  
cout << (1 + 4) / 2 << "\n";   // 2
```

### Modulus

- `-7 % 4 == -3`
### `<cmath>`
![[Pasted image 20260320111523.png|376]]
## Logical Operators
- `!`
- `||`
- `&&`
## Bitwise Operators
- `&`
- `|`
- `~`
- XOR: `^`
    - `0 ^ 0 = 0`
    - `1 ^ 1 = 0`
    - `1 ^ 0 = 1`
    - `0 ^ 1 = 1`
    - 相同为 `0`，不同为 `1`
- `<<`, `>>`
    - e.g. `a << 8`
    - `a` 向左移动 8 bit
    - `a` is multiplied by `2^8`

## 非0即真

```
std::bitset<8>(a) //print a number in binary;
```
  
