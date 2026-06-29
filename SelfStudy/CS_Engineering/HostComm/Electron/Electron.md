- 用于开发桌面应用的框架，来编写Windows，macOS和Linux桌面软件
	- HTML: 描述页面结构
	- CSS：设置界面样式
	- JavaScript: 交互与程序逻辑
- 普通网页只能运行在浏览器标签页。Electron把网页技术封装到独立桌面应用窗口，通过node.js调用部分系统能力

## 为什么适合做上位机
- Node.js可以通过npm安装`serialport` 库

## 进程类型
- Main Process + Renderer Process(渲染进程)
1. Main Process 对应main.js
	1. 创建 BrowserWindow
	2. 枚举串口
	3. 打开和关闭串口
	4. 发送串口数据
	5. 接收串口数据
2. Renderer Process
	- 可以粗鲁理解为UI层
	1. 显示HTML界面
	2. 相应按钮点击
	3. 读取输入框内容
	4. 更新表格和日志
	5. 展示数据

## preload.js
渲染进程本质上运行网页代码。出于安全考虑，不应该允许网页直接调用所有 Node.js 和 Electron 功能。
例如，不应该让网页随意执行：
```
require('node:fs');
```
否则网页代码可以直接读取电脑文件。
因此，Electron 通常使用一个中间层：
```
preload.js
```
它只把经过筛选的少量方法暴露给网页。

eg.
```JavaScript
contextBridge.exposeInMainWorld('electronAPI', { 
	listPorts: () => ipcRenderer.invoke('list-ports'), 
	connectPort: (portPath) => 
		ipcRenderer.invoke('connect-port', portPath), 
});
```
渲染进程中只能调用：
```JavaScript
window.electronAPI.listPorts();
window.electronAPI.connectPort('COM5');
```
而不能随意访问整个 Node.js 环境。

## IPC
Inter-Process Communication 进程间通信