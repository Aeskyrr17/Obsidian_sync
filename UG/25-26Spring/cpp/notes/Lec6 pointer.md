- 注意：int* ptr1, ptr2: ptr2不是指针。
    - 正确写法，int *ptr1, *ptr2;
- * has lower priority than .
        - 所以要用（）。
            - eg. Student* stu1 = xxxxx;
            - string name = `*(stu1).get_name()`; (stu1).等价于- >

1. dynamic memory allocation
    1. 对于运行的时候才能知道数组长度的情况。（编译时不知道要多大，程序运行时再去申请内存（在Heap上申请）
    2. **new: return address**
		```cpp
		int* p = new int;
		//创建了一个指针p。
		//用new在heap申请了一个int大小的内存
		//由于new返回的是地址，把这个地址赋值给p
		//p在stack上，int 在heap上
		```

		```cpp
		Student* g(){
		Student* bob = new Student("Bob", 12345);
		return bob;
		}
		```
        
	- 这里的bob在函数return之后还会保留，直到手动delete这块内存
        
2. **delete**
        
	```cpp
	int* a = new int;
	delete a;
	int* b_arr = new int[N];
	delete[] b_arr;
	/*new delete; new[] delete[]要对应，不然引发undefined error*/
	```
        
	- 这里delete/delete[]只是释放了heap上的内存，但是指针并没有被删除，更安全的做法：
        
	```cpp
	delete p;
	p = nullptr;
	```

3. pointer & array
    1. `a[i] = *(a+i)`

4. 注意reference
	```cpp
	int i = 42;
	int* p;
	int*& r = p;
	r = &i;
	
	*r = 0;//这里会把i变成0！！！！！！reference是强绑定的
	```

5. const
	1. cannot be changed after initialization
		1. ==const修饰的变量必须要是initialized的==
		eg. 
		```cpp
		const int bufsize = 512;//合法
		const i = get_size(); //合法，可以在run的时候才确定
		const int k; //不合法，没有initialization
		```

	2. reference to const
		```cpp
		const int ci = 1024;
		const int& ref1 = ci; //ok,const 对应
		ref1 = 42;//error!!!
		int &ref2 = ci;//error!!!
		```
		==非const不能通过const变量来赋值==
		**例外**
		```cpp
		int ci = 1024;
		const int& ref = ci; //这个是ok的
		ci = 45;  //ok
		ref = 45; //error!!!不能通过这个refrence来修改ci
		```
	
	3. ==const int& 可以把非const给const ref赋值，但是不能通过这个const ref来修改==
		`const double* cptr` = 一个指向const double 的指针
			**可以改变他指向哪里，但是不能通过这个cptr去改他指向的值**
		就是说
		`cptr = ...` 可以
		`*cptr = ...` 不可以
	4. ==`int* const cptr`指向对象不能变，指向的值可以变==
		`*cptr = ...` ok
		`cptr = ...` no
	5. `const int* const cptr`
### function pointer

存的是函数的地址，有时候要把“怎么做”传进另外一个函数的参数里面
1. 声明
	==bool (* pf)(const string &, const string &)==
	 一定要加括号，（* pf）是一个接受两个para返回bool的指针
2. 将函数赋值给函数指针
3. 
	```CPP
	bool (* pf)(const string &, const string &)； //定义一个指针
	pf = length_compare;
	pf = &length_compare; 
	都可以
	```
4. 通过函数指针调用函数
	bool b1 = pf(xxxx)；
	bool b2 = (* pf)(xxx)； 都可以

	```cpp
	//定义一个helper函数
	bool compare_by_id(const xxx& a, const xxx& b)
	{xxx};
	
	void bubble_sort(xxx arr[], int size, (*pf)(const xxx& a, const xxx& b))
	{   xxx
		pf(arr[i],arr[i+1])
	    xxx}
	
	//调用：
	bubble_sort(arr,size,compare_by_id);
	```