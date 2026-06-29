---

---

DJI电机的“spd_mode/Relax_mode(不设置就是默认relax_mode)，指的是pid计算相关mode

setoutput（）就是电机自己的计算pid。

所以，没有设置mode的电机，不需要setoutput（），会导致电机不动

只有sendconrtoldata（)是发送控制指令给电机