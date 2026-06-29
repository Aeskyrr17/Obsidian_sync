---

---
1. 为什么不用eventcallback-因为cam传的东西定长？那为什么还要定MAIX_SIZE？
2. main.cpp中创建实例会报错
- `MaixComm::``*Instance*``()->Init();` 可以build 
- 原来的MaixComm *maixcomm = MaixComm::Instance();不能build
    - 因为是static？—静态 没学懂不会
3. 只用include main.h就可以了 why 
4. 不能在maix.hpp里面写crc的声明
