#needs-review 

可以。你现在这套协议已经够完整了，正好借它把常见通信协议概念串起来。先记住一个总框架：

```text
USART 负责“怎么传字节”
协议负责“这些字节是什么意思”
业务层负责“收到命令后实际做什么”
```

对应你的工程：

```text
USART10 + DMA
    ↓
HostComm：拆帧、CRC、Type、Seq
    ↓
DartHostService：校验并处理请求
    ↓
DartLibrary / SysCtrl：真正修改参数和 runtime
```

---

# 1. 物理层：USART 是什么

USART/UART 只负责一件事：

> 按顺序传输字节。

例如上位机想发：

```text
AA 55 01 10 25 00 12 34
```

USART 不知道：

- `AA 55` 是帧头；
    
- `10` 是 GET_DART_TABLE；
    
- `25` 是 Seq；
    
- 最后两个字节是 CRC。
    

它只负责把这串字节送过去。

常见 UART 参数：

```text
460800 baud
8 data bits
No parity
1 stop bit
```

简称：

```text
460800, 8N1
```

---

# 2. 为什么需要“帧”

串口收到的是连续字节流：

```text
AA 55 01 10 ... AA 55 01 13 ...
```

它没有天然的“消息边界”。

所以协议必须自己定义：

```text
一条消息从哪里开始
一条消息有多长
一条消息在哪里结束
```

这就是帧：

```text
SOF1 SOF2 Ver Type Seq Len Payload CRC16
```

例如：

```text
AA 55 01 14 37 00 XX XX
```

表示一条完整的 GET_PRE_TENSION 请求。

## 各字段含义

|字段|含义|
|---|---|
|`SOF`|帧头，帮助找到消息起点|
|`Ver`|协议版本|
|`Type`|这是什么消息|
|`Seq`|这是第几号请求|
|`Len`|Payload 有多少字节|
|`Payload`|实际业务数据|
|`CRC`|检查数据是否损坏|

---

# 3. Type：消息类型

`Type` 回答的是：

> 这条消息要干什么？

例如：

```text
0x01 HEARTBEAT_REQ
0x10 GET_DART_TABLE
0x11 SET_DART_PARAMS_BATCH
0x13 SET_SEQUENCE
0x20 DART_PARAM_STATE
0x70 ACK
0x7F ERROR_RSP
```

同一种 Type 可以发送很多次，所以还需要 `Seq` 区分每一次。

```text
Type = SET_SEQUENCE
Seq = 20

Type = SET_SEQUENCE
Seq = 21
```

它们是两次不同的修改。

---

# 4. Request 和 Response

大多数双向协议都采用：

```text
请求 Request
→ 处理
→ 响应 Response
```

例如读取 sequence：

```text
PC → H7：GET_SEQUENCE，Seq=10
H7 → PC：SEQUENCE_STATE，Seq=10
```

修改 sequence：

```text
PC → H7：SET_SEQUENCE，Seq=11
H7 → PC：ACK，Seq=11
H7 → PC：SEQUENCE_STATE，Seq=11
```

这里：

- `ACK` 表示修改成功；
    
- `SEQUENCE_STATE` 返回实际生效后的值。
    

---

# 5. ACK 是什么

`ACK` 是 acknowledgement，意思是：

> 我确认已经成功处理了你的请求。

比如上位机发送：

```text
SET_PRE_TENSION
Seq = 32
pre_tension = 32.0
```

下位机真正完成：

```cpp
dart.config.pre_tension_kg = 32.0f;
```

之后回复：

```text
ACK
ref_seq = 32
result = APPLIED
```

上位机才可以显示：

```text
写入成功
```

## ACK 不只是“我收到了”

这里有两个不同层次：

### 接收确认

```text
我收到了这条消息
```

### 执行确认

```text
我已经校验并真正应用了这条消息
```

你的协议更适合使用第二种。

也就是说：

```text
HostComm 收到帧
```

不能立刻发 `APPLIED`。

必须等：

```text
DartHostService 校验
→ 修改 DartConfig
→ 修改成功
```

之后再发 ACK。

否则可能出现：

```text
HostComm：收到，先回成功
SysCtrl：参数非法，实际没修改
```

上位机就被骗了。

---

# 6. 什么时候需要 ACK

## 应该 ACK 的操作

凡是会改变设备状态的请求，通常都应该 ACK：

```text
SET_DART_PARAMS_BATCH
SET_SEQUENCE
SET_PRE_TENSION
RESET_TEST_ROUND
```

因为上位机必须知道：

```text
到底有没有真正执行成功
```

## 读取请求不一定需要单独 ACK

例如：

```text
GET_SEQUENCE
→ SEQUENCE_STATE
```

收到 `SEQUENCE_STATE` 本身就已经说明请求成功，因此理论上不用额外 ACK。

但对于 `GET_DART_TABLE`：

```text
DART_PARAM_STATE × 16
```

单独加一个最后的 ACK 很有用，因为它表示：

```text
16 行已经全部发完
```

否则上位机不知道：

```text
我只收到 15 行，是还没发完，还是丢了一行？
```

## 周期状态通常不 ACK

例如每 50 ms 发送：

```text
DART_RUNTIME_STATE
FAST_TELEMETRY
```

不要每帧 ACK。

否则会变成：

```text
状态包
ACK
状态包
ACK
状态包
ACK
```

浪费一半带宽，还可能形成 ACK 风暴。

周期遥测偶尔丢一帧通常没关系，下一帧马上覆盖。

---

# 7. ACK、ERROR 和 NACK

成功时：

```text
ACK
```

失败时：

```text
ERROR_RSP
```

有些协议称失败确认为：

```text
NACK / NAK
```

即 negative acknowledgement。

你的协议采用 `ERROR_RSP` 更好，因为它不仅说失败，还能说为什么失败：

```text
PARAM_OUT_OF_RANGE
STATE_FORBIDDEN
CRC_ERROR
LENGTH_ERROR
DUPLICATE_DART_ID
```

例如：

```text
SET_PRE_TENSION = 9999
→ ERROR_RSP
→ error_code = PARAM_OUT_OF_RANGE
```

或者：

```text
Launcher 正在 FIRING
发送 RESET_TEST_ROUND
→ ERROR_RSP
→ STATE_FORBIDDEN
```

---

# 8. Seq 是什么

`Seq` 是 sequence number，通信序号。

它像订单编号：

```text
SET_SEQUENCE，Seq=40
SET_PRE_TENSION，Seq=41
```

回复：

```text
ACK，ref_seq=41
```

上位机就知道：

```text
预张紧修改成功
sequence 修改还没回复
```

`Seq` 用于：

- 把响应与请求对应起来；
    
- 判断超时的是哪条请求；
    
- 区分连续发送的同类型命令；
    
- 后续做重复请求检测；
    
- 周期数据中判断是否丢帧。
    

它和飞镖的：

```cpp
config.sequence[4]
```

完全无关。

---

# 9. Timeout：超时

上位机发送请求后，不能永远等下去。

例如：

```text
发送 SET_SEQUENCE，Seq=52
等待最多 300 ms
```

如果 300 ms 内没有收到：

```text
ACK Seq=52
或 ERROR Seq=52
```

就认为超时。

超时不一定代表下位机没执行，可能是：

- 请求没到；
    
- 请求到了，回复没到；
    
- CRC 错误；
    
- 下位机队列满；
    
- 串口断开；
    
- 下位机卡住；
    
- ACK 丢失。
    

所以超时只能说明：

> 上位机无法确认这次操作的结果。

此时最好显示：

```text
写入结果未知，请回读确认
```

而不是直接说：

```text
写入失败
```

---

# 10. Retry：重发

超时后可以重发。

例如：

```text
第一次：
SET_PRE_TENSION Seq=60
→ 超时

第二次：
再次发送 SET_PRE_TENSION Seq=60
```

重发时通常保持同一个 Seq，表示：

```text
这是同一次请求的重试
```

但这会引出一个问题：

> 下位机第一次其实已经执行，只是 ACK 丢了，那么重发会不会执行第二次？

---

# 11. 幂等性

如果重复执行一次，结果仍然一样，这种操作称为幂等。

例如：

```text
把 pre_tension 设置为 32
```

执行一次：

```text
pre_tension = 32
```

执行两次：

```text
pre_tension = 32
```

结果一样，所以它是幂等的。

你当前这些操作基本都幂等：

```text
SET_DART_PARAMS_BATCH
SET_SEQUENCE
SET_PRE_TENSION
GET_...
```

但下面这些不是天然幂等：

```text
current_shot_number++
FIRE
toggle
move one step
```

例如 FIRE 重发两次，可能真的发射两次。

因此未来如果开放发射控制，必须增加：

```text
Seq 去重
```

下位机记住最近处理过的 Seq：

```text
收到重复 Seq
→ 不再次执行动作
→ 只重新发上次 ACK
```

---

# 12. Heartbeat 是什么

心跳是定期发送的小消息，用来判断：

> 对方还活着吗？链路还通吗？

例如上位机每 300 ms 发送：

```text
HEARTBEAT_REQ
```

下位机回复：

```text
HEARTBEAT_RSP
```

如果连续几次没收到回复：

```text
超过 1 秒
→ 标记设备离线
```

## 心跳有什么用

### 1. 判断物理连接

USB-UART 可能还显示为 COM 口，但 H7 已经：

- 掉电；
    
- 死机；
    
- HardFault；
    
- 串口异常；
    
- 线程没运行。
    

心跳能发现：

```text
端口还开着，但设备已经不工作
```

### 2. 判断通信线程是否工作

能收到心跳响应说明至少：

```text
USART RX
帧解析
CRC
HostComm
hostreq
DartHostService
hosttx
USART TX
```

这条链路基本通了。

### 3. 获取设备信息

你的心跳响应还可以携带：

```text
protocol_version
uptime
config_revision
device_state
```

上位机能知道：

```text
固件是否重启
协议版本是否匹配
配置是否变化
设备是否处于 error
```

### 4. 判断设备是否重启

例如之前：

```text
uptime = 120000 ms
```

突然变成：

```text
uptime = 500 ms
```

说明 MCU 刚刚 Reset。

此时上位机应该意识到：

```text
RAM 参数可能已经恢复默认
```

然后重新读取配置表。

---

# 13. 心跳不是安全许可

收到心跳只说明：

```text
设备在线、通信大体正常
```

不代表：

- Launcher 安全；
    
- 允许发射；
    
- 电机正常；
    
- 传感器正常；
    
- 门已经打开。
    

所以不要把：

```text
heartbeat online
```

等价于：

```text
system ready to fire
```

心跳是通信健康检查，不是业务安全检查。

---

# 14. CRC 是什么

CRC 用于检测传输过程中数据是否损坏。

例如原本发送：

```text
pre_tension = 32.0
```

某一位受干扰变成：

```text
pre_tension = 96.0
```

接收端重新计算 CRC，发现和帧尾 CRC 不一致：

```text
CRC ERROR
```

于是丢弃整帧，不执行修改。

CRC 能检测大多数随机传输错误，但它不是：

- 加密；
    
- 身份认证；
    
- 防黑客；
    
- 防止逻辑错误。
    

它只是检查：

> 收到的字节是否和发送的字节一致。

---

# 15. Len 是什么

`Len` 表示 Payload 长度。

例如：

```text
GET_SEQUENCE
Len = 0
```

```text
SET_SEQUENCE
Len = 4
```

```text
SET_PRE_TENSION
Len = 4
```

接收端通过 Len 知道：

```text
还需要再收多少字节，之后才是 CRC
```

还可以检查消息是否合法：

```text
SET_SEQUENCE 规定必须 4 字节
但收到 Len=3
→ LENGTH_ERROR
```

---

# 16. Version 是什么

协议会持续修改。

今天：

```text
SET_SEQUENCE Payload = 4 bytes
```

明年可能变成：

```text
sequence 数量可变
```

如果上位机和下位机使用不同协议，却都假装能通信，很危险。

所以帧里加入：

```text
Ver = 0x01
```

发现版本不兼容时：

```text
VERSION_UNSUPPORTED
```

而不是硬着头皮解析错误格式。

---

# 17. Request、Response 和 Telemetry

你的协议中其实有三类消息。

## Request

上位机主动要求设备做事：

```text
GET_DART_TABLE
SET_SEQUENCE
RESET_TEST_ROUND
```

## Response

设备对请求作出回复：

```text
ACK
ERROR_RSP
DART_PARAM_STATE
SEQUENCE_STATE
```

## Telemetry

设备主动周期上报：

```text
DART_RUNTIME_STATE
DOOR_STATE
LAUNCHER_STATE
FAST_TELEMETRY
```

Telemetry 不一定对应某个请求，通常不需要 ACK。

---

# 18. FIFO / Queue 是什么

假设 HostComm 很快收到三条请求：

```text
A
B
C
```

普通 topic 可能只保留最新值：

```text
只剩 C
```

FIFO 则保存顺序：

```text
A → B → C
```

SysCtrl 依次处理：

```text
先 A
再 B
再 C
```

所以：

```text
hostreq 用 FIFO
```

是为了防止请求被覆盖。

同理：

```text
GET_DART_TABLE
→ 16 行参数 + ACK
```

`hosttx` 也需要 FIFO，否则可能只剩最后一条。

---

# 19. Flow control / Backpressure

如果生产消息的速度比发送速度快：

```text
SysCtrl 每秒产生 100 条
USART 每秒只能发 50 条
```

队列会越来越满。

这叫生产者比消费者快。

处理方式包括：

- 限制发送频率；
    
- 增大 FIFO；
    
- 对低优先级遥测丢帧；
    
- ACK/ERROR 优先；
    
- 在队列满时返回 `QUEUE_FULL`；
    
- 不允许无限堆积。
    

对于你的系统，优先级应该是：

```text
ACK / ERROR
> 配置回读
> Runtime 状态
> 快速遥测
```

宁可少画一帧曲线，也不能丢掉关键 ACK。

---

# 20. Parser 状态机是什么

因为 UART 是连续字节流，解析器通常按状态工作：

```text
WaitSof1
→ WaitSof2
→ WaitHeader
→ WaitPayload
→ WaitCrcLo
→ WaitCrcHi
```

例如：

```text
当前 WaitSof1
收到 AA
→ WaitSof2

收到 55
→ WaitHeader
```

如果 CRC 正确：

```text
HandleFrame()
```

如果错误：

```text
丢弃
→ 重新 WaitSof1
```

这种逐字节状态机适合处理：

- 半包；
    
- 粘包；
    
- 噪声；
    
- 帧错位；
    
- 一次 DMA 收到多帧。
    

---

# 21. 半包和粘包

## 半包

一帧分几次收到：

```text
第一次 DMA：AA 55 01 10
第二次 DMA：20 00 CRC...
```

解析器不能要求一次 DMA 刚好收到一整帧。

它要保留状态，等下一批字节继续。

## 粘包

一次 DMA 收到多帧：

```text
帧 A + 帧 B + 帧 C
```

解析器处理完 A 后，必须继续处理 B、C。

DMA 回调的边界和协议帧边界没有关系。

---

# 22. 主动状态的 Seq

周期遥测不是某个请求的回复，因此可以让下位机维护：

```text
device_tx_seq = 0, 1, 2...
```

上位机如果收到：

```text
100
101
103
```

就知道：

```text
102 可能丢了
```

不过周期状态丢一帧通常不重传，因为下一帧马上会来。

---

# 23. 你这套协议推荐的交互规则

## 写请求

```text
PC 分配 Seq
→ 发送 SET
→ 等 ACK 或 ERROR
→ 超时则提示并回读
```

## 读单个状态

```text
PC 发送 GET，Seq=N
→ H7 返回对应 STATE，Seq=N
```

可以不额外 ACK。

## 读完整飞镖表

```text
PC 发送 GET_DART_TABLE，Seq=N
→ H7 返回 16 个 DART_PARAM_STATE，Seq=N
→ H7 返回 ACK，Seq=N
```

上位机确认：

```text
16 个不同 dart_id 均已收到
并且 ACK 已收到
```

才算完整成功。

## 周期遥测

```text
H7 主动发送
不等待 ACK
旧数据可被新数据覆盖
```

## 心跳

```text
PC 每 300 ms 发送
连续约 3 次未回复
→ 标记 H7 offline
```

---

# 最后用一句话串起来

一套可靠的串口协议，本质上是在连续字节流之上增加：

```text
帧头确定起点
Len 确定长度
Type 表示用途
Seq 对应请求
CRC 检测损坏
ACK 确认成功
ERROR 说明失败原因
Timeout 检测无响应
Retry 处理偶发丢包
Heartbeat 检测在线状态
FIFO 保证消息不被覆盖
状态机处理半包与粘包
```

你现在这套 HostComm，已经把这些工业通信里最常见的骨架基本覆盖到了。