### template
```cpp
template <typename T>
T max(T x, T y)
{
	return (x < y) ? y : x;
}

int main()
{
	std::cout << max<int>(1,2) << '\n';
	
	return 0;
}
```
- compiler调用`max<int>`，识别到definition for `max<int>(int,int)`还未存在，will implicitly use `max<T>` to create one

#### standard template library
##### 三个部分
- container
- iterator
	- `number.begin()`
	- `number.end()` 指向的不是末尾元素，是后面一个
- algorithm

- 什么时候要判断`if (it != numbers.end())`
	1. 使用了 return iterator的func
	```cpp
	auto it = std::find(numbers.begin(), numbers.end(), target);
	
	if (it != numbers.end()){  <-- 如果找到了
		std::cout << "Found" << *it << std::endl;	
	} else{
		std::cout << "Not found" << std::endl;
	}
	```
	1. 要 `*it`
		要先判断是不是numbers.end()，防止错误