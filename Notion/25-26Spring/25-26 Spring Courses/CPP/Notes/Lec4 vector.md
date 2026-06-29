---
notion-id: 315fc701-9d52-8029-bcd4-ea12736fafa6
---
1. 创建一个vector
```c++
std::vector<int> vec;
std::vector<int> vec(3); //{0,0,0},会被初始化
std::vector<int> vec(3,10);  //{10,10,10}
std::vector<int> vec{1,2,3,4};
std::vector<int> vec2(vec); std::vector<int> vec2 = vec;

```
2. 常用成员函数（ppt有一些比较复杂的逻辑，可以回去复习）
```c++
std::vector<int> v;
v.push_back(1); //在后面加一个1，{1}
v.push_back(2); //{1,2}
v.empty(); 1//false
v.back(); //2
v.pop_back(); //{1}，删掉后面一个
v.at(0) //1,会判断越界，更安全
v[0];//1
v[2]; //编译能过，但实际上有越界error!!!
v.size(); //1
```
![](https://prod-files-secure.s3.us-west-2.amazonaws.com/c3dfc701-9d52-81ac-87ce-0003a14d6707/ef2114da-3474-4658-a434-87deeb586e1c/image.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=ASIAZI2LB4667UBDTDV7%2F20260329%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20260329T044218Z&X-Amz-Expires=3600&X-Amz-Security-Token=IQoJb3JpZ2luX2VjEDwaCXVzLXdlc3QtMiJHMEUCIQC8REpy5EbrprOypCDlbitRFj6sPR2qpLlw3mthEHI1awIgOV08xM4oVdEYhuxefIMetlMpG%2FK8qfoqgBq2EI6Fp%2FYq%2FwMIBRAAGgw2Mzc0MjMxODM4MDUiDCPGqROjAaP%2Fgvc24yrcA4F22A6NDGnkZnfGSGsZTLfSdRwxBSnD8hDTlGYVRJh%2BWg5yeuxeKiHGghwNkq%2B4zaN3EhK%2FFZoAFOgwZGaYw6YAHS%2FBRm7PEOFXcEhByobJGhqDrBCTNXqcm%2Fh9JYcZN7RsS8OosRtffeIC2nQZn%2FQtOZ54mk%2Fv4irm7GCbAr6iJfKC2ZDvglaG%2BGMS%2FdBsm%2FOBRS6JGUGsy0lWiN%2Ftz%2FEFwb2Xin7ICKd1cwiR8DdKKqLIJ0UblaF3JUowiW245nfw9vnuOScn6alAIOHoFHp%2BEcwKLmLXDSMvj3lZKzmIZwovCQwwrh8euPAlS08QRZWxXqt7lWkU63JQ4SxL5kX1kAe76jp%2F5qS6wUbAsN2FLorUelBz6z6vMS18Q7%2FvA8QHmxgj%2F8ZqWV3a3kTKW9eZxPg111UA%2BsD%2FYcSMYrGCQ%2BfPanewA7z9QjAXxk20UG%2FHf44uCpE70a5p77LH5aMwuJZ%2FnJSEEVL4PVwN7Uv9oVh9GeLH%2B%2F0pwkTX2pL8mfaMrvZJtdY%2FeGbRfvtfPympNSCouF%2FYAVx08AvgUnYKYi%2Bu4mALcPrgTNyH0WF0cH5dcF5zUqUWCwMS%2BO3xAJqyuhc0mliUl%2By2WU%2B6LxoDjz2ygl08IkQK6AUxMO%2B%2Bos4GOqUBYe0XbbQROUlIACyY0Ll8gqJ1Hxlvx5QPKOeMyCRktv%2FDIW5mxqOEbqfBdLst7hm5gPl0k7qLd6%2Foerq%2Fz23mOU2i4A0Ij9m%2Bs305wRBFUwg%2FwN%2FkYbE489aLOpXkSwcuFxucqeOt4ibXQkrp8qK%2FXso5sBjNyTeM0doDehPyEJ5b2an5Z4X5MeEm1mSeDTSTbbXoGKoxmO%2BLmh9ebnr77mCxw1a7&X-Amz-Signature=c19f5d9e837b5ee020d4613476c8989943e21f1f7f9c607befc9ba73cdbd252c&X-Amz-SignedHeaders=host&x-amz-checksum-mode=ENABLED&x-id=GetObject)