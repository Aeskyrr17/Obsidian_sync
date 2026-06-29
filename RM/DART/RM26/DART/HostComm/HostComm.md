1. 一句话说明这个上位机是什么

`HostComm` 是一个 Electron + Node.js + SerialPort 写成的飞镖参数调试上位机：它通过 USB-UART 连到 STM32H7 的 USART10，只做心跳、参数读取/写入、sequence、pre_tension、测试轮次 reset 和日志，不做发射控制。

2. 四个工作区之间的关系

|部分|作用|本次阅读结论|
|---|---|---|
|`dart_hostcomm_api_v0_2.md`|协议设计文档|说明了帧格式、Type、Payload、ACK/ERROR、UI 需求|
|`RM26_H7`|STM32H7 下位机实际实现|wire 协议最终以这里为准；USART10 实际 `115200 8N1`|
|`2026RM_Dart_Cli`|参考 Electron 上位机|只参考 main/preload/renderer/SerialPort/parser/log 结构；业务协议已重写|
|`HostComm`|当前上位机工程|实现了新的飞镖 HostComm 协议和 UI|

3. 整体架构图

发送方向：

```
用户点击 UI
→ HostComm/src/renderer/renderer.js
→ window.hostComm.xxx()
→ HostComm/preload.js contextBridge
→ Electron IPC invoke
→ HostComm/main.js ipcMain.handle(...)
→ RequestManager.send(...)
→ payload-builders.js
→ frame-builder.js
→ SerialService.write(...)
→ serialport
→ USB-UART
→ STM32H7 USART10
→ RM26_H7/User/Task/Src/HostComm.cpp
→ om topic: hostreq
→ DartHostService::Poll(...)
→ DartHostService::ProcessRequest(...)
→ DartLibrary config/runtime
```

响应方向：

```
DartHostService 生成 ACK / STATE
→ om topic: hosttx
→ HostComm.cpp ProcessTxQueue / StartTxFrame
→ USART10 DMA TX
→ USB-UART
→ serialport data
→ SerialService
→ FrameParser.push(...)
→ payload-decoders.js
→ RequestManager.handleFrame(...)
→ main.js 转发 host:frame / host:device / host:frame-log
→ preload.js
→ renderer.js
→ 更新表格 / 状态 / 日志
```

每一层存在的原因：

|层|为什么需要|
|---|---|
|renderer|只管 DOM、按钮、表格、日志，不碰串口|
|preload|安全桥，限制网页只能调用白名单 API|
|main process|Electron 主进程，能访问 Node、文件、串口|
|serial-service|把 SerialPort 和字节流 parser 封起来|
|request-manager|管 Seq、pending、ACK、ERROR、timeout、事务完成|
|protocol|独立处理字节协议，不混 DOM|
|H7 HostComm|只收发帧、校验 CRC、发布请求|
|DartHostService|真正应用参数、生成 ACK/ERROR/STATE|

4. Electron 三层结构

Main process，即主进程，是 Electron 应用的“后台”。当前是 [main.js](D:/01_Workspace/RM/dart/dart_new/HostComm/main.js)。它负责：

- `createWindow()` 创建 `BrowserWindow`，加载 `index.html`
- 设置安全参数：`contextIsolation: true`、`nodeIntegration: false`
- 创建 `SerialService` 和 `RequestManager`
- 用 `ipcMain.handle(...)` 注册 renderer 可调用的请求
- 用 `mainWindow.webContents.send(...)` 把串口事件推回 renderer
- 在 `window-all-closed` 时停止 heartbeat、清 pending、断串口

关键对象：

```
const serial = new SerialService();
const requests = new RequestManager((frame, meta) => serial.write(frame, meta));
```

这里的 `RequestManager` 不直接知道串口，它只拿到一个“写帧函数”。这样协议请求和物理串口解耦。

Preload，即预加载脚本，是 [preload.js](D:/01_Workspace/RM/dart/dart_new/HostComm/preload.js)。它运行在网页和主进程之间。`contextBridge` 是 Electron 的安全桥，作用是：不给网页整个 `ipcRenderer`，只暴露你允许的函数。

当前暴露的是：

|renderer 调用名|IPC channel|
|---|---|
|`getSettings()`|`host:get-settings`|
|`listPorts()`|`host:list-ports`|
|`connect(args)`|`host:connect`|
|`disconnect()`|`host:disconnect`|
|`readDartTable()`|`host:read-dart-table`|
|`writeDartBatch(args)`|`host:write-dart-batch`|
|`readSequence()`|`host:read-sequence`|
|`writeSequence(sequence)`|`host:write-sequence`|
|`readPreTension()`|`host:read-pre-tension`|
|`writePreTension(value)`|`host:write-pre-tension`|
|`resetTestRound()`|`host:reset-test-round`|
|`exportLog(text)`|`host:export-log`|
|`onStatus(cb)`|`host:status`|
|`onDevice(cb)`|`host:device`|
|`onFrame(cb)`|`host:frame`|
|`onFrameLog(cb)`|`host:frame-log`|
|`onLog(cb)`|`host:log`|
|`onError(cb)`|`host:error`|

Renderer，即渲染进程，是网页逻辑：[renderer.js](D:/01_Workspace/RM/dart/dart_new/HostComm/src/renderer/renderer.js)。它负责：

- `$()` 取 DOM 元素
- `bindEvents()` 给按钮绑定点击事件
- 调用 `window.hostComm` 的 API
- 用 `state` 保存 UI 状态
- `applyDartRow()` 更新 16 发表格
- `applySequenceState()` 更新四发 sequence
- `applyPreTensionState()` 更新 pre_tension
- `addLog()` 写日志

几个 JavaScript 概念顺手讲一下：

- `Promise`：表示“未来完成的结果”。串口请求不会立刻有结果，所以 `requests.send(...)` 返回 Promise。
- `async/await`：让 Promise 写起来像同步流程，比如 `const result = await api.readDartTable()`。
- `EventEmitter`：Node 事件系统。`SerialService` 和 `RequestManager` 都继承它，用 `.emit(...)` 发事件，用 `.on(...)` 监听事件。
- IPC，Inter-Process Communication，进程间通信。renderer 和 main 是两个进程，靠 `ipcRenderer.invoke` / `ipcMain.handle` 通信。

5. HostComm 目录树

```
HostComm/
├── package.json
├── package-lock.json
├── main.js
├── preload.js
├── index.html
├── styles.css
├── NOTICE.md
├── api.md
├── docs/
│   └── protocol-audit.md
├── src/
│   ├── protocol/
│   │   ├── constants.js
│   │   ├── crc16.js
│   │   ├── frame-builder.js
│   │   ├── frame-parser.js
│   │   ├── payload-builders.js
│   │   └── payload-decoders.js
│   ├── serial/
│   │   ├── request-manager.js
│   │   └── serial-service.js
│   └── renderer/
│       └── renderer.js
└── tests/
    ├── all.test.js
    ├── crc16.test.js
    ├── frame-builder.test.js
    ├── frame-parser.test.js
    ├── payload.test.js
    └── run-tests.js
```

当前 `src/renderer` 下没有 `dart-table.js/device-state.js/logger.js` 等拆分模块，实际只有 `renderer.js`。

6. 每个文件的作用

|文件|核心职责|被谁调用|调用了谁|注意点|
|---|---|---|---|---|
|`package.json`|npm 脚本、依赖、electron-builder 配置|npm|Electron/SerialPort|当前有 `overrides.yauzl`|
|`main.js`|Electron 主进程、IPC、串口请求、心跳|Electron|`SerialService`、`RequestManager`、payload builders|串口只在 main 中直接操作|
|`preload.js`|安全暴露 API|renderer|`ipcRenderer`|没暴露整个 IPC|
|`index.html`|UI DOM 结构|Electron window|`renderer.js`|只有静态 HTML|
|`styles.css`|UI 样式|HTML|无|工作台式布局|
|`NOTICE.md`|说明参考来源|人读|无|记录参考 GPL 项目但未复制业务协议|
|`docs/protocol-audit.md`|协议审计矩阵|人读|无|H7 实际协议的依据|
|`constants.js`|Type、枚举、范围|多个模块|无|区分 `CONFIG_TARGET` 和 `REFEREE_TARGET`|
|`crc16.js`|CRC16-Modbus|builder/parser/test|无|和 H7 `Get_CRC16_Modbus_Check_Sum` 对齐|
|`frame-builder.js`|组完整帧|`RequestManager`|`crc16Modbus`|返回 `Buffer`|
|`frame-parser.js`|字节流拆帧|`SerialService`|CRC、constants|支持半包/粘包/坏帧|
|`payload-builders.js`|业务 payload 编码|`main.js`|constants|写 float 使用 `writeFloatLE`|
|`payload-decoders.js`|payload 解码|`SerialService`|constants|状态未支持时返回提示|
|`serial-service.js`|串口扫描、连接、读写、parser 接入|`main.js`|`serialport`、`FrameParser`|只管串口和帧事件|
|`request-manager.js`|Seq、pending、ACK/ERROR/timeout|`main.js`|`buildFrame`|判断请求何时真正完成|
|`renderer.js`|UI 事件、状态、表格、日志|HTML|preload API|不直接 require Node 模块|
|`tests/run-tests.js`|当前实际测试入口|`npm test`|协议/请求模块|当前 9/9 通过|
|`tests/*.test.js`|node:test 风格测试|当前未被 npm 直接跑|协议模块|保留但 `package.json` 跑 custom runner|
|`api.md`|空文件|无|无|当前内容为空|

7. 协议帧结构

当前帧格式：

```
SOF1 SOF2 Ver Type Seq Len Payload CRC16
```

|字段|意义|
|---|---|
|`SOF1/SOF2`|Start Of Frame，帧头，固定 `AA 55`，用于从字节流里找帧起点|
|`Ver`|协议版本，当前 `0x01`|
|`Type`|消息类型，比如 heartbeat、读取表格、ACK|
|`Seq`|通信帧序号，`uint8`，用于请求和响应配对|
|`Len`|Payload 长度，`uint8`，H7 最大 64|
|`Payload`|业务数据，不同 Type 不同|
|`CRC16`|校验码，防止串口噪声导致误解析|

无 Payload 示例：`HEARTBEAT_REQ`，`type=0x01`，`seq=0x00`：

```
aa 55 01 01 00 00 50 18
```

拆开：

```
aa 55    SOF
01       Version
01       Type HEARTBEAT_REQ
00       Seq
00       Len = 0
50 18    CRC16 little-endian，覆盖 01 01 00 00
```

有 Payload 示例：`SET_PRE_TENSION`，设置 `32.5 kg`，`type=0x15`，`seq=0x2a`：

```
aa 55 01 15 2a 04 00 00 02 42 b2 5d
```

拆开：

```
aa 55          SOF
01             Version
15             Type SET_PRE_TENSION
2a             Seq
04             Len = 4
00 00 02 42    float32 little-endian，表示 32.5
b2 5d          CRC16 little-endian
```

JS 中写 float：

```
const payload = Buffer.alloc(4);
payload.writeFloatLE(value, 0);
```

H7 中读 float：

```
float value = 0.0f;
memcpy(&value, payload + offset, sizeof(value));
```

CRC 覆盖：

```
Ver Type Seq Len Payload
```

不覆盖 `AA 55`，也不覆盖 CRC 自己。

8. Frame Builder 和 Parser

Frame Builder 在 [frame-builder.js](D:/01_Workspace/RM/dart/dart_new/HostComm/src/protocol/frame-builder.js)。

输入：

```
buildFrame(type, seq, payload)
```

内部流程：

```
payload 转 Buffer
检查 payload.length <= 64
分配 Buffer.alloc(8 + payload.length)
写 AA 55
写 Version / Type / Seq / Len
复制 Payload
对 frame[2..payload末尾] 算 CRC
CRC 用 writeUInt16LE 写入末尾
返回完整 frame Buffer
```

如果没有 Builder，UI 或 main 很容易到处手拼字节，出现 Len/CRC/float endian 错误。

Frame Parser 在 [frame-parser.js](D:/01_Workspace/RM/dart/dart_new/HostComm/src/protocol/frame-parser.js)。

为什么不能假设一次 `data` 就是一帧？串口是字节流，不是消息队列。可能出现：

```
一次 data = 半帧
一次 data = 3 帧
一次 data = 垃圾 + 半帧
一次 data = 坏 CRC 帧 + 好帧
```

Parser 做法：

```
this.buffer += chunk
findSof() 搜索 AA 55
前面垃圾丢掉
如果不足 6 字节，等待下次
读 Len
Len > 64 报 LENGTH_ERROR，往后移动 1 字节重同步
如果不足完整帧，等待下次
计算 CRC
CRC 错则报 CRC_ERROR，往后移动 1 字节继续找
Version 错则报 VERSION_UNSUPPORTED
成功则 emit('frame')，并从 buffer 删除这帧
继续 while 处理剩余粘包
```

`AA AA 55` 的例子：

```
输入：AA AA 55 01 12 04 00 ... CRC
```

第一次从 index 0 看不是 `AA 55`，继续找；index 1 是 `AA 55`，于是能正确同步到第二个 AA。

具体流例子：

```
chunk1: 00 99 AA 55 01 02
chunk2: 07 08 ...payload... crc
```

第一次 push：

- 丢 `00 99`
- 找到 `AA 55`
- 发现长度不够，保留 buffer

第二次 push：

- 拼成完整帧
- 校验 CRC
- 输出 frame

9. Seq / RequestManager / ACK / ERROR / Timeout

[request-manager.js](D:/01_Workspace/RM/dart/dart_new/HostComm/src/serial/request-manager.js) 是“请求账本”。

为什么需要它？因为“串口 write 成功”只表示字节交给操作系统，不表示 H7 已处理。真正成功要等：

```
ACK
或者 ERROR_RSP
或者某些 STATE 回传
或者 timeout
```

pending request 是：

```
{
  seq,
  type,
  sentAt,
  timeoutMs,
  mode,
  ack,
  rows,
  states,
  state,
  resolve,
  reject,
  timer
}
```

`Seq` 分配：`allocateSeq()` 从 0 到 255 循环，跳过仍在 pending 的 seq。

ACK 关联：

```
ACK frame type = 0x70
payload.ref_type
payload.ref_seq
```

`handleAck()` 用 `decoded.refSeq` 找 pending，并检查 `refType` 是否等于原请求 Type。

ERROR 关联类似，`handleError()` 找 pending，构造 Error，并 reject 这个请求。

Timeout：`send()` 里创建 `setTimeout`。时间到还没完成，就：

```
pending.delete(seq)
reject(Error)
emit('timeout')
```

普通写请求例子：`SET_PRE_TENSION`

```
renderer.writePreTension()
→ preload host:write-pre-tension
→ main.js buildSetPreTension()
→ RequestManager.send(type=0x15, mode='setPreTension')
→ H7 ACK
→ RequestManager 记录 ack
→ H7 PRE_TENSION_STATE
→ RequestManager 记录 state
→ ack + state 都有，resolve
→ renderer applyPreTensionState()
```

读取整表：

```
GET_DART_TABLE
→ DART_PARAM_STATE x16
→ ACK
→ RequestManager.rows 收 16 个不同 dart_id
→ request.ack 存在
→ tryCompleteDartTable resolve
```

批量写超过 8 支：

```
选择 16 支
→ main.js sendDartBatch()
→ 分成 [1..8] 和 [9..16]
→ 第一包 RequestManager 完成后
→ 才发送第二包
```

不能两批同时乱发的原因：H7 每包会改 config 并回传同 seq 的多行状态；同时发会让 UI 和人很难判断哪批失败，写命令也不应无序并发。

10. Heartbeat

Heartbeat 在 [main.js](D:/01_Workspace/RM/dart/dart_new/HostComm/main.js) 的 `startHeartbeat()`。

策略：

```
连接成功后 startHeartbeat()
每 300 ms 定时
如果上一个 heartbeat 还没完成，不重入
发送 HEARTBEAT_REQ
timeout 260 ms
收到 HEARTBEAT_RSP：missedHeartbeats = 0，online=true
连续 timeout：missedHeartbeats++
missedHeartbeats >= 3：认为设备 offline
```

请求：

```
HEARTBEAT_REQ Type 0x01，Payload 空
```

响应：

```
HEARTBEAT_RSP Type 0x02
device_state:u8
protocol_version:u8
uptime_ticks:u32
config_revision:u16
```

注意 H7 里用的是 `tx_time_get()`，当前代码显示为 ticks，不假装是 ms。

Renderer 的 `updateDevice()` 还做了一件事：如果 `uptimeTicks` 比上一次小，认为 MCU 可能重启，于是调用：

```
readTable()
readSequence()
readPreTension()
```

因为 MCU 重启后 RAM 配置可能回到默认值，上位机表格可能过期。

重要边界：Heartbeat online 只表示通信在线，不表示 Launcher 可以安全发射。本上位机也没有发射命令。

11. 16 发参数表与批量写入

UI 表格字段：

|字段|含义|
|---|---|
|ID|物理飞镖 ID，固定 1..16，只读|
|yaw_offset|该物理飞镖的 yaw 偏置|
|Base tension|打基地目标时的 tension kg|
|Outpost tension|打前哨站目标时的 tension kg|
|checkbox|选择要批量修改的飞镖|

为什么 ID 只读：H7 `Dart_Config_t dart[17]` 按 ID 索引，ID 是物理飞镖编号，不是配置项。

为什么表格值以下位机回传为准：点击“写入”后，上位机不知道 H7 是否接受、是否范围错误、是否重复 ID、是否状态禁止。必须等 `DART_PARAM_STATE`。

批量写 payload：

```
target_type:u8
update_mask:u8
dart_count:u8
dart_ids:u8[dart_count]
yaw_offset:f32
tension:f32
```

`update_mask`：

```
0x01 更新 yaw
0x02 更新 tension
0x03 同时更新 yaw + tension
```

`CONFIG_TARGET` 和 `REFEREE_TARGET` 必须分开：

```
CONFIG_TARGET.BASE = 0
CONFIG_TARGET.OUTPOST = 1

REFEREE_TARGET.OUTPOST = 0
REFEREE_TARGET.BASE = 1
```

配置命令里 `target_type=0` 表示写 base tension。运行状态里 `chosen_target=0` 表示前哨站。数值刚好相反，混用会把基地/前哨 tension 写错。

12. Sequence / Pre-tension / Reset

三个 sequence 容易混：

|名称|含义|
|---|---|
|通信 `Seq`|帧序号，`uint8 0..255`，用于 ACK/响应配对|
|飞镖 `sequence`|本轮四发用哪些物理飞镖，例如 `[3,4,5,8]`|
|`current_shot_number`|当前是本轮第几发，通常 1..4|

H7 里：

```
runtime.current_dart_id = config.sequence[runtime.current_shot_number - 1];
```

`SET_SEQUENCE` payload 是 4 字节：

```
shot1_id shot2_id shot3_id shot4_id
```

当前 H7 只检查 1..16，不拒绝重复。UI 只 warning，不禁止。

`pre_tension` 是预拉力，单位 kg。`SET_PRE_TENSION` payload 是一个 float32。

`RESET_TEST_ROUND` payload 为空。H7 做的是：

```
runtime.current_shot_number = 1
Update_Current_Dart_Id()
runtime.fired_count_this_open = 0
runtime.last_fire_finished = lch2sys.is_fire_finished
runtime.autoAim.yaw_ok = false
```

它不清：

```
config.dart[1..16]
config.sequence
config.pre_tension_kg
config_revision
```

13. UI 各区域

|区域|按钮|Type|等待响应|成功后|
|---|---|---|---|---|
|串口连接区|Refresh|无串口帧|SerialPort list|更新端口列表|
|串口连接区|Connect|连接动作|port open|启动 heartbeat|
|串口连接区|Disconnect|断开动作|close|停心跳、清 pending|
|设备状态区|自动心跳|`0x01`|`0x02`|更新 Online/Protocol/Uptime/Revision|
|16 发表格|Read Table|`0x10`|`0x20 x16 + 0x70`|更新 16 行|
|批量写入区|Write Selected Darts|`0x11`|`ACK + DART_PARAM_STATE`|更新对应行|
|Sequence 区|Read|`0x12`|`0x21`|更新 4 个输入|
|Sequence 区|Write|`0x13`|`ACK + 0x21`|更新 4 个输入|
|Pre-tension 区|Read|`0x14`|`0x22`|更新输入|
|Pre-tension 区|Write|`0x15`|`ACK + 0x22`|更新输入|
|Reset 区|Reset Test Round|`0x16`|`ACK` 或 `ERROR_RSP`|日志显示结果|
|Runtime/Door/Launcher/Telemetry|无控制按钮|当前无请求|当前固件未发送|显示 Not supported|
|日志区|Pause/Clear/Export|无协议|本地 UI|控制日志显示|

14. 七条完整调用链
    
15. 点击“连接串口”
    

```
renderer.js::bindEvents connectBtn click
→ api.connect(...)
→ preload.js::connect
→ IPC host:connect
→ main.js ipcMain.handle('host:connect')
→ serial-service.js::SerialService.connect
→ new SerialPort({ path, baudRate, 8N1 })
→ port.open(...)
→ port.on('data', parser.push)
→ emit status
→ main.js serial.on('status')
→ renderer.js::updateConnectionUi
```

2. Heartbeat

```
main.js::startHeartbeat
→ RequestManager.send(HEARTBEAT_REQ, emptyPayload, mode='heartbeat')
→ frame-builder.js::buildFrame
→ serial-service.js::write
→ USB-UART
→ H7 HostComm.cpp::ParseByte / DecodeFrame
→ om_publish hostreq
→ DartHostService::Poll
→ ProcessRequest HOST_REQ_HEARTBEAT
→ HostSendHeartbeat
→ hosttx
→ HostComm.cpp::ProcessTxQueue / StartTxFrame
→ USART10 TX
→ serial-service.js port.on('data')
→ FrameParser.push
→ decodeHeartbeatRsp
→ RequestManager.handleFrame
→ main.js send host:device
→ renderer.js::updateDevice
```

3. `GET_DART_TABLE`

```
renderer.js::readTable
→ preload readDartTable
→ IPC host:read-dart-table
→ main.js requests.send(GET_DART_TABLE, mode='dartTable')
→ H7 HostComm.cpp DecodeFrame HOST_TYPE_GET_DART_TABLE
→ DartHostService::ProcessRequest HOST_REQ_GET_DART_TABLE
→ for id 1..16 HostSendDartParamState
→ HostSendAck
→ PC FrameParser / payload decoder
→ RequestManager.collectDartRow x16
→ RequestManager.handleAck
→ tryCompleteDartTable
→ renderer readTable result.rows
→ applyDartRow
```

4. `SET_DART_PARAMS_BATCH`

```
renderer.js::writeBatch
→ finiteInput / selected IDs / targetSegment / update mask
→ preload writeDartBatch
→ IPC host:write-dart-batch
→ main.js::sendDartBatch
→ buildSetDartParamsBatch
→ RequestManager.send(SET_DART_PARAMS_BATCH, mode='setDartBatch')
→ H7 HostComm.cpp DecodeFrame: target_type/update_mask/dart_ids/float
→ DartHostService::ProcessRequest HOST_REQ_SET_DART_PARAMS_BATCH
→ 参数范围检查
→ duplicate 检查
→ 修改 dart.config.dart[id].yaw_offset / tension_kg_base / tension_kg_outpost
→ config_revision++
→ HostSendAck
→ HostSendDartParamState per id
→ RequestManager 等 ACK + 对应状态行
→ renderer applyDartRow
```

5. `SET_SEQUENCE`

```
renderer.js::writeSequence
→ currentSequence()
→ preload writeSequence
→ IPC host:write-sequence
→ main.js buildSetSequence
→ RequestManager.send(SET_SEQUENCE, mode='setSequence')
→ H7 HostComm.cpp DecodeFrame memcpy sequence[4]
→ DartHostService::ProcessRequest HOST_REQ_SET_SEQUENCE
→ 检查每个 ID 1..16
→ dart.config.sequence[i] = req.sequence[i]
→ dart.Update_Current_Dart_Id()
→ config_revision++
→ ACK + SEQUENCE_STATE
→ RequestManager ack + state 完成
→ renderer applySequenceState
```

6. `SET_PRE_TENSION`

```
renderer.js::writePreTension
→ finiteInput
→ preload writePreTension
→ IPC host:write-pre-tension
→ main.js buildSetPreTension
→ payload.writeFloatLE(value)
→ RequestManager.send(SET_PRE_TENSION, mode='setPreTension')
→ H7 HostComm.cpp ReadFloat
→ DartHostService::ProcessRequest HOST_REQ_SET_PRE_TENSION
→ 检查 0..120
→ dart.config.pre_tension_kg = req.pre_tension
→ config_revision++
→ ACK + PRE_TENSION_STATE
→ renderer applyPreTensionState
```

7. `RESET_TEST_ROUND`

```
renderer.js::resetTestRound
→ window.confirm
→ preload resetTestRound
→ IPC host:reset-test-round
→ main.js requests.send(RESET_TEST_ROUND, mode='ack')
→ H7 HostComm.cpp DecodeFrame
→ DartHostService::ProcessRequest HOST_REQ_RESET_TEST_ROUND
→ IsResetTestRoundAllowed(lch2sys)
→ 若非 IDLE/ERROR_STOP：ERROR_RSP STATE_FORBIDDEN
→ 若允许：重置 runtime current_shot_number/current_dart_id/fired_count/yaw_ok
→ HostSendAck
→ renderer addLog
```

15. 上下位机职责边界

上位机负责：

```
串口连接
组帧/拆帧
Seq/ACK/ERROR/timeout 管理
参数 UI
参数读写请求
日志
```

H7 HostComm 负责：

```
USART10 DMA 收发
帧同步
CRC/Len/Version/Type 检查
发布 hostreq
发送 hosttx
```

DartHostService 负责：

```
解释请求
检查范围
修改 DartLibrary.config/runtime
生成 ACK/ERROR/STATE
```

TaskSysCtrl 负责：

```
根据遥控器、视觉、裁判系统和 DartLibrary 生成 msg_cmd_t
发布 cmd_topic 给 Launcher
```

这点很关键：当前上位机不发布 `msg_cmd_t`，也不直接控制 Launcher FSM。

16. 安全性分析

我按 Type、payload builder、renderer action、IPC、H7 Host request enum、DartHostService switch 搜了当前代码。

当前能够做：

```
HEARTBEAT
读取/写入 16 发 yaw/tension 配置
读取/写入 sequence
读取/写入 pre_tension
RESET_TEST_ROUND
接收 ACK/ERROR/STATE
```

当前不能做：

```
FIRE
PREPARE
RELAX
电机点动
修改 Launcher FSM
发布 msg_cmd_t
直接控制 prepare_state
```

H7 的 `TaskSysCtrl.cpp` 里确实存在 `DART_FIRE/DART_PREPARE/DART_RELAX`，但它们来自遥控器/自动控制逻辑，不来自 `HostComm` 请求。`HostProtocol.hpp` 的 host request enum 也没有 FIRE/PREPARE/RELAX。

间接影响：

|SET 操作|间接影响|
|---|---|
|修改 yaw|后续自动/手动流程解析目标 yaw 时可能用新偏置|
|修改 tension|后续打 base/outpost 时目标拉力会改变|
|修改 pre_tension|后续预拉力目标会改变|
|修改 sequence|后续本轮第几发对应的物理飞镖会改变|
|RESET_TEST_ROUND|改 runtime 进度，但不直接触发机械动作|

17. 测试与构建

`package.json`：

```
dependencies:
  serialport

devDependencies:
  electron
  electron-builder
```

脚本：

```
npm install      安装依赖
npm test         node tests/run-tests.js
npm start        electron .
npm run build    electron-builder --win --dir
```

几个包：

|包|作用|
|---|---|
|`electron`|桌面壳，提供 main/preload/renderer|
|`serialport`|Node 串口库|
|`@serialport/bindings-cpp`|serialport 的原生 C++ 绑定|
|`electron-builder`|打包 Electron 应用|

测试结构：

|测试|覆盖|
|---|---|
|CRC|H7 heartbeat 固定向量、空/非空输入|
|frame builder|SOF、Version、Type、Seq、Len、Payload、CRC LE|
|parser|单帧、半包、粘包、垃圾、`AA AA 55`、坏 CRC、Len 超限|
|payload|batch、float LE、sequence、pre_tension、ACK、ERROR|
|RequestManager|16 行表格事务必须 16 rows + ACK|

当前我只执行了阅读和验证命令，没有修改代码。结果：

```
npm test: 9/9 tests passed
node -c main.js: pass
node -c preload.js: pass
node -c src/renderer/renderer.js: pass
```

单元测试能证明：协议字节布局、CRC、parser、payload、部分事务逻辑在软件层成立。  
不能证明：真实 USB-UART、H7 时序、串口掉线、实际硬件状态、Launcher 状态禁止等。

```
Software verified
Hardware integration pending
```

18. 当前问题与风险

|严重程度|文件/位置|问题|可能后果|建议方向|
|---|---|---|---|---|
|High|`renderer.js::writeBatch`|即使未勾选 yaw/tension，也会先 `finiteInput` 对对应输入框取值|用户只想改 tension，但 yaw 输入为空仍失败；反之亦然|只在勾选字段时校验对应输入|
|Medium|`renderer.js::resetTestRound`|confirm 中文文本当前显示为 mojibake：`璇ユ搷...`|用户确认弹窗不可读，影响安全提示|统一文件 UTF-8，修正文案|
|Medium|`main.js::startHeartbeat`|心跳周期 300 ms，但 timeout 260 ms，较激进|H7 忙或 Windows 串口抖动时容易误判 offline|联调后放宽到 500~800 ms 或按 RTT 自适应|
|Medium|`renderer.js::readTable/applyDartRow`|`tableTransaction.seq` 初始为 `null` 时会接受任意 `DART_PARAM_STATE` 计数|若读表期间同时有写批次回传，进度可能混入其他 seq|开始读表后尽快记录 seq，或只用 RequestManager 返回进度事件|
|Medium|`RequestManager::send`|heartbeat 与手动请求共享同一个 pending 池，且 heartbeat 持续运行|极端情况下心跳请求占用 seq/串口时序，干扰调参体验|写请求期间可暂停 heartbeat 或降低优先级|
|Medium|`SerialService::connect close handler`|close 事件发送旧 `this.path`；主动 disconnect 时 `this.port=null` 后 close 也可能再发 status|UI 可能收到重复 disconnected|区分主动关闭和意外断开|
|Medium|`package.json` / build 环境|`npm run build` 需要 Electron 二进制与 VS C++ Build Tools|当前机器若缺构建工具，serialport rebuild 失败|安装 VS Build Tools，确认 Electron 下载完整|
|Low|`preload.js`|`listeners` Map 记录 callback，但没有统一 clear API|长生命周期页面一般没事，复杂页面可能残留监听|提供 removeAll 或页面卸载清理|
|Low|`tests/*.test.js`|保留 node:test 文件，但 npm 实际跑 `run-tests.js`|新人可能以为 `*.test.js` 是主测试入口|README 或注释说明当前 runner 原因|
|Low|`docs/protocol-audit.md`|uptime 标为 ticks，tick 到 ms 未确认|UI 只能显示 ticks|H7/API 统一 tick 频率或改发 ms|

19. 推荐阅读代码的顺序
    
20. [docs/protocol-audit.md](D:/01_Workspace/RM/dart/dart_new/HostComm/docs/protocol-audit.md)：先懂协议。
    
21. [constants.js](D:/01_Workspace/RM/dart/dart_new/HostComm/src/protocol/constants.js)：记住 Type 和枚举。
    
22. [frame-builder.js](D:/01_Workspace/RM/dart/dart_new/HostComm/src/protocol/frame-builder.js) + [crc16.js](D:/01_Workspace/RM/dart/dart_new/HostComm/src/protocol/crc16.js)：理解发出去的字节。
    
23. [frame-parser.js](D:/01_Workspace/RM/dart/dart_new/HostComm/src/protocol/frame-parser.js)：理解收回来的字节流。
    
24. [payload-builders.js](D:/01_Workspace/RM/dart/dart_new/HostComm/src/protocol/payload-builders.js) + [payload-decoders.js](D:/01_Workspace/RM/dart/dart_new/HostComm/src/protocol/payload-decoders.js)：理解业务字段。
    
25. [request-manager.js](D:/01_Workspace/RM/dart/dart_new/HostComm/src/serial/request-manager.js)：理解 Seq/ACK/事务。
    
26. [serial-service.js](D:/01_Workspace/RM/dart/dart_new/HostComm/src/serial/serial-service.js)：理解 SerialPort。
    
27. [main.js](D:/01_Workspace/RM/dart/dart_new/HostComm/main.js)：理解 Electron 主进程如何串起来。
    
28. [preload.js](D:/01_Workspace/RM/dart/dart_new/HostComm/preload.js)：理解 IPC 安全桥。
    
29. [renderer.js](D:/01_Workspace/RM/dart/dart_new/HostComm/src/renderer/renderer.js)：理解 UI。
    
30. H7 的 `HostComm.cpp`、`DartHostService.cpp`、`TaskSysCtrl.cpp`：对照上下位机闭环。
    
31. [tests/run-tests.js](D:/01_Workspace/RM/dart/dart_new/HostComm/tests/run-tests.js)：看哪些行为已有软件保证。
    
32. 最后给你一份速查表
    

|我想找什么|应该看哪个文件|
|---|---|
|Electron 窗口创建|[main.js](D:/01_Workspace/RM/dart/dart_new/HostComm/main.js) `createWindow()`|
|串口扫描和连接|[serial-service.js](D:/01_Workspace/RM/dart/dart_new/HostComm/src/serial/serial-service.js)|
|IPC|[main.js](D:/01_Workspace/RM/dart/dart_new/HostComm/main.js) `ipcMain.handle`， [preload.js](D:/01_Workspace/RM/dart/dart_new/HostComm/preload.js)|
|帧打包|[frame-builder.js](D:/01_Workspace/RM/dart/dart_new/HostComm/src/protocol/frame-builder.js)|
|帧解析|[frame-parser.js](D:/01_Workspace/RM/dart/dart_new/HostComm/src/protocol/frame-parser.js)|
|CRC|[crc16.js](D:/01_Workspace/RM/dart/dart_new/HostComm/src/protocol/crc16.js)|
|Type / 枚举|[constants.js](D:/01_Workspace/RM/dart/dart_new/HostComm/src/protocol/constants.js)|
|Payload 编码|[payload-builders.js](D:/01_Workspace/RM/dart/dart_new/HostComm/src/protocol/payload-builders.js)|
|Payload 解码|[payload-decoders.js](D:/01_Workspace/RM/dart/dart_new/HostComm/src/protocol/payload-decoders.js)|
|Seq 和 pending request|[request-manager.js](D:/01_Workspace/RM/dart/dart_new/HostComm/src/serial/request-manager.js)|
|Heartbeat|[main.js](D:/01_Workspace/RM/dart/dart_new/HostComm/main.js) `startHeartbeat()`|
|16 发表格|[renderer.js](D:/01_Workspace/RM/dart/dart_new/HostComm/src/renderer/renderer.js) `buildDartTable()` / `applyDartRow()`|
|批量写入|[renderer.js](D:/01_Workspace/RM/dart/dart_new/HostComm/src/renderer/renderer.js) `writeBatch()`， [main.js](D:/01_Workspace/RM/dart/dart_new/HostComm/main.js) `sendDartBatch()`|
|Sequence|[renderer.js](D:/01_Workspace/RM/dart/dart_new/HostComm/src/renderer/renderer.js) `writeSequence()`，H7 `DartHostService.cpp`|
|Pre-tension|[payload-builders.js](D:/01_Workspace/RM/dart/dart_new/HostComm/src/protocol/payload-builders.js) `buildSetPreTension()`|
|Reset|[renderer.js](D:/01_Workspace/RM/dart/dart_new/HostComm/src/renderer/renderer.js) `resetTestRound()`，H7 `DartHostService.cpp`|
|H7 协议入口|`RM26_H7/User/Task/Src/HostComm.cpp`|
|H7 Type 定义|`RM26_H7/User/Task/Inc/HostProtocol.hpp`|
|H7 业务处理|`RM26_H7/User/Task/Src/DartHostService.cpp`|
|H7 当前参数/运行逻辑|`RM26_H7/User/Task/Src/TaskSysCtrl.cpp`|
|USART10 波特率|`RM26_H7/Core/Src/usart.c`|
|测试入口|[tests/run-tests.js](D:/01_Workspace/RM/dart/dart_new/HostComm/tests/run-tests.js)|