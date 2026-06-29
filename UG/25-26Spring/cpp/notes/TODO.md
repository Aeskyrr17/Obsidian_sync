1. **STL 容器与 member functions**
    - `vector / deque / list / forward_list`
    - `stack / queue / priority_queue`
    - `set / multiset / map / multimap`
    - `unordered_set / unordered_map`
    - `insert / erase / find / count / at / [] / equal_range`
    - `pair<iterator, bool>`
2. **iterator + algorithm**
    - `begin/end`
    - `*it`, `it->first`, `it->second`
    - `find`, `count`, `sort`
    - 你之前还特意问过：**是否要会自己实现 `find` / `count`**。我当时的判断是：  
        **你至少要会写 simplified template version。**
3. **templates**
    - function template
    - class template
    - 为什么 `unordered_set<Person, PersonHash>` 里面会有 `PersonHash`
    - `my_find / my_count` 这种模板 + iterator 混合题
4. **operator overloading**
    - 你之前问过 `friend`、PPT 里的运算符重载怎么写；
    - 我们当时定下的是：  
        **考试优先掌握 PPT 风格：member operator overload；friend 不是重点。**
5. **inheritance / polymorphism 的细节**
    - example 考了纯虚、slicing、`dynamic_cast`
    - 但之前 PPT 里我们还聊过：
        - constructor/destructor order
        - virtual destructor
        - `override`
        - function hiding / deleting inherited functions
        - `final`
        - pure virtual destructor 为什么还要实现
6. **exceptions**
    - 你专门问过 `throw error` 和 `handling exception`
    - example paper **完全没考**
    - 但既然在 PPT，**今晚必须过一遍最基本写法**