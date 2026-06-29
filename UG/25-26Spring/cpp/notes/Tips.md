1. 有关比较/排序
	1. string比较：字典序
	2. a.compare(b) 
		1. >0, a>b
		2. =0
		3. <0
	3. ```cpp
	   void operator>(const Studnet& rhs) const {
		   return score < rhs.member;
		   }
	   ```
	4. sort: 排序时临时指定规则： sort(...,..., cmp)
	5. ```cpp
	   sort(vec.begin(), vec.end(), cmp);
	   
	   bool cmp(int a, inat b){
		   return a>b;
	   }
	   ```
		- 当`cmp(a,b)==ture`: a在b前面
			- 当定义了a>b: 大的在前面
	6. 容器自带的排序规则
		1. set/map 默认是 `set<int, less<int>> set1`: 1,2,3,4,5
			1. `set<int, greater<int>> set2`: 5,4,3,2,1
		2. priority_queue
			1. ![[UG/25-26Spring/cpp/notes/asserts/Drawing 2026-05-16 13.38.28.excalidraw]]

2. 
```CPP
find(k)         // 找 == k
lower_bound(k)  // 找 >= k
upper_bound(k)  // 找 > k
equal_range(k)  // { >= k 的第一个, > k 的第一个 }
```

3. class中private变量的意义 ![[Pasted image 20260517124802.png]]