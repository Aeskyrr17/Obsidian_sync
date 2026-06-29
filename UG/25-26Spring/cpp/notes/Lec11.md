### insert返回值
- 对于map，insert的返回值是
```cpp
auto return_val = pair< map<A,B>::iterator, bool>
```
1. return_val是一个pair
	1. 第一个是“map的iterator”，第二个是“insert是否成功”
```
return_val.first->second : map的B
(!return_val.second) : 插入不成功（或者说key在map里已经存在）
```
- 对于multimap的insert，返回值就是一个it（因为不需要bool）

### map的下标操作和解引用
对于一般container，`*it` and `vec[1]` 返回的都是元素本身
- 对于map
	- `map["key"]` 返回 mapped_type: 就是pair的second，key对应的值
	- `*it` 返回value_type: 一整个pair
- map[A] 如果不存在，会自动插入