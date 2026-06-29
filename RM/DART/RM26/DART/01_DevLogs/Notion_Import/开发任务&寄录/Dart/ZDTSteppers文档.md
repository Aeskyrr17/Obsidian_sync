---
notion-id: 302fc701-9d52-8071-af44-e6f2e563bef9
---
tips:

1. 发送和接收都采用can的扩展帧ID
    1. 修改cubemx的extend filter element Nbr， 给大于0的数，不然无法进入can回调
    2. init的时候修改过滤配置
2. ID为动态变化，详情看文档
