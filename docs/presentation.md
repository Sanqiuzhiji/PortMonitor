---
marp: true
theme: default
paginate: true
backgroundColor: #ffffff
color: #333333
style: |
  section {
    font-family: 'Microsoft YaHei', 'Segoe UI', sans-serif;
    font-size: 24px;
    line-height: 1.5;
  }
  h1 {
    color: #2196F3;
    border-bottom: 3px solid #2196F3;
    padding-bottom: 10px;
    font-size: 48px;
  }
  h2 {
    color: #455A64;
    font-size: 36px;
    margin-bottom: 20px;
  }
  h3 {
    font-size: 28px;
    color: #0277BD;
    margin-bottom: 15px;
  }
  p {
    margin-bottom: 15px;
    font-size: 24px;
  }
  code {
    background: #f5f5f5;
    padding: 2px 6px;
    border-radius: 3px;
    color: #e91e63;
    font-family: Consolas, 'Courier New', monospace;
  }
  pre {
    background: #263238;
    border-radius: 8px;
    padding: 15px;
    font-size: 20px;
  }
  pre code {
    color: #aed581;
    background: transparent;
  }
  table {
    margin: 20px auto;
    font-size: 22px;
  }
  table th {
    background: #2196F3;
    color: white;
  }
  table td {
    border: 1px solid #e0e0e0;
  }
  .highlight {
    background: #fff3cd;
    padding: 15px;
    border-left: 4px solid #ffc107;
    margin: 20px 0;
    font-size: 22px;
  }
  .columns {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 30px;
  }
  .description {
    color: #666;
    font-style: italic;
    margin-bottom: 25px;
    font-size: 22px;
  }
  ul li {
    margin-bottom: 8px;
  }
---

<!-- _class: lead -->
<!-- _paginate: false -->

# 串口监控程序设计与实现
## 基于 PyQt5 的即时通信系统

<div class="description">
本演示文稿将详细介绍一个跨平台串口调试工具的设计思路、核心技术难点及其解决方案。
</div>

---

## 项目概览

<div class="description">
这是一个专为嵌入式开发人员设计的调试工具，旨在解决传统串口助手在跨平台支持和实时可视化方面的痛点。
</div>

### 🎯 核心功能

- 🔌 **全平台支持**：一套代码同时运行在 Windows、Linux 和 macOS 上。
- ⚡ **毫秒级响应**：采用异步 I/O 设计，保证数据收发零延迟。
- 📊 **可视化分析**：不仅看数据，还能通过波形图直观分析数据变化。
- 💾 **智能记忆**：自动记录上一次的调试配置，提升工作效率。

**技术栈概览**
> Python + PyQt5 (GUI) + QSerialPort (通信) + PyQtGraph (绘图)

---

## 软件架构设计

<div class="description">
为了保证软件的长期可维护性和扩展性，我们没有将代码写在一个文件里，而是采用了经典的 MVC 分层架构。
</div>

### MVC 架构与信号槽

<div class="columns">

<div>

**逻辑与界面分离**

- **Model (数据层)**：`SerialProcess` 负责脏活累活，由它直接和硬件打交道。
- **View (视图层)**：`Ui_Serial_MainWindow` 只负责画皮，展示漂亮的界面。
- **Controller (控制层)**：`SerialAppClass` 是大管家，协调数据和界面的交互。

</div>

<div>

**通信机制：信号槽 (Signals & Slots)**

我们不使用死循环去查询数据，而是让数据"通知"我们需要处理。

- `readyRead` -> 通知有新数据
- `data_received` -> 通知界面更新
- `errorOccurred` -> 通知出错了

</div>

</div>

---

<!-- _class: lead -->

# 第一部分
## 核心通信模块实现

<div class="description">
即时通讯是本软件的基石。我们将深入探讨如何使用 QSerialPort 实现稳定、高效的串口数据流转。
</div>

---

## 为什么选择 QSerialPort？

<div class="description">
在 Python 生态中，PySerial 也是一个常见的选择，但为了追求极致的 GUI 体验，我们选择了 Qt 原生的解决方案。
</div>

### 技术对比分析

| 特性 | PySerial (传统方案) | QSerialPort (本方案) ✅ |
|-------|-------------------|-------------------------|
| **运行模式** | 阻塞式 (容易卡死界面) | **异步非阻塞 (界面流畅)** |
| **线程模型** | 必须手动开线程 | **事件驱动 (单线程即可)** |
| **集成度** | 第三方库，需自行封装 | **Qt 原生对象，配合完美** |
| **错误处理** | 抛出异常，中断流程 | **信号通知，平滑处理** |

<div class="highlight">

💡 **核心优势**：QSerialPort 的"事件驱动"机制，允许我们在不创建复杂多线程的情况下，依然保持界面的丝般顺滑。

</div>

---

## 串口初始化流程

<div class="description">
一个健壮的串口程序，始于严谨的初始化过程。我们需要确保硬件资源被正确加载和配置。
</div>

<div class="columns">

<div>

**初始化的三个关键步骤**

1.  **实例化对象**：创建一个系统级的串口句柄。
2.  **建立连接**：铺设好"电话线"，一旦有数据(readyRead)或者有故障(error)，立即通知处理函数。
3.  **参数预备**：准备好波特率等参数，等待用户发令。

</div>

<div>

**代码片段示意**

```python
def __init__(self):
    # 1. 创建串口对象
    self.serial = QSerialPort()
    
    # 2. 绑定生命线 (信号连接)
    # 有数据来了 -> 去读数据
    self.serial.readyRead.connect(
        self.read_data
    )
    # 有错误发生 -> 去处理错误
    self.serial.errorOccurred.connect(
        self.handle_error
    )
```

</div>

</div>

---

## 参数配置与打开

<div class="description">
串口通信就像对暗号，只有通信双方的波特率、校验位等参数完全一致，才能正确交换信息。
</div>

### 严谨的六步配置法

在点击"打开串口"的那一瞬间，程序在后台为您做了这些事：

1.  **选端口**：确定是 `COM3` 还是 `COM4`。
2.  **定速度**：设置波特率 (如 `115200`)，决定传输快慢。
3.  **定长度**：设置数据位 (标准为 `8` 位)。
4.  **定校验**：设置奇偶校验，保证数据准确。
5.  **定停止**：设置停止位，标识数据包结束。
6.  **定流控**：设置硬件流控制，防止缓冲区溢出。

只有这六项全部设置成功，我们才会调用 `open()` 真正接管硬件。

---

## 数据接收：零拷贝与异步

<div class="description">
这是程序最繁忙的部分。我们需要以最快速度把硬件缓冲区里的数据搬运到内存，并且不能卡顿。
</div>

<div class="columns">

<div>

**数据是如何流动的？**

1.  **硬件触发**：网卡收到电信号，底层驱动填满缓冲区。
2.  **信号通知**：Qt 发出 `readyRead` 信号，就像门铃响了。
3.  **批量读取**：使用 `readAll()` 一次性拿走所有数据，甚至都不需要我们自己去循环读取。
4.  **画面更新**：数据传给 UI，界面随之刷新。

</div>

<div>

**高效的关键**

```python
def read_data(self):
    # 一次性读完所有暂存数据
    # 避免了反复调用的开销
    data = self.serial.readAll()
    
    if len(data) > 0:
        # 发送信号，把数据甩给界面层
        self.data_received.emit(data)
```

</div>

</div>

---

## 数据的发送逻辑

<div class="description">
发送不仅仅是把字符串丢出去，还需要考虑到用户是想发文字还是发十六进制指令。
</div>

<div class="columns">

<div>

**模式 A：文本模式**
简单直接，所见即所得。
- 输入："Hello"
- 发送：`48 65 6C 6C 6F` (ASCII码)

</div>

<div>

**模式 B：Hex 模式**
专业调试常用，精确控制每一个字节。
- 输入："AA BB CC"
- 自动清洗：去除空格
- 发送：`0xAA 0xBB 0xCC`

</div>

</div>

**智能容错设计**
> 如果用户在 Hex 模式下输入了奇数个字符（如 "A"），程序会自动补零变为 "0A"，防止程序崩溃，提升用户体验。

---

<!-- _class: lead -->

# 第二部分
## 测速系统的工作原理

<div class="description">
为了监控通信质量，我们需要实时计算传输速率。这不仅是个数字，更是系统稳定性的晴雨表。
</div>

---

## 测速的核心思想

<div class="description">
测速的本质，就是算术题：在单位时间内，究竟通过了多少字节？
</div>

### "桶"的原理

想象我们有一个水桶（计数器）：

1.  每收到一个字节的数据，就往桶里扔一颗豆子。
2.  不需要实时计算速度，只需要埋头数豆子。
3.  **每隔一秒钟**，除了一个人（定时器）来看一眼桶里有多少豆子。
4.  看的过程就是**计算速度**，看完之后把桶倒空（清零），重新开始数。

$$ 速度 (Bytes/s) = \frac{当前这一秒新增的字节数}{1秒} $$

---

## 测速代码实现

<div class="description">
利用 QTimer 定时器，我们可以轻松实现上述的"一秒一看"逻辑。
</div>

**1. 埋头计数 (在接收函数中)**

```python
# 只要来数据，就累加计数器
self.receive_count += len(data)
```

**2. 定时计算 (每 1000ms 触发一次)**

```python
def update_speed(self):
    # 当前总数 - 上一秒的总数 = 这一秒的增量
    bytes_per_sec = self.receive_count - self.last_count
    
    # 记录当前总数，为下一秒做准备
    self.last_count = self.receive_count
    
    # 显示结果
    print(f"当前网速: {bytes_per_sec} B/s")
```

---

<!-- _class: lead -->

# 第三部分
## 配置管理与持久化

<div class="description">
好的软件应该像老朋友一样，记得你的习惯。无论是上次用的波特率，还是窗口的位置，都应该被自动保存。
</div>

---

## JSON 配置管理

<div class="description">
我们选择 JSON 文件来存储配置。它既方便程序读取，也方便人类直接阅读和修改。
</div>

### 自动保存机制

程序在关闭时 (`closeEvent`)，会触发一次"快照"保存：

- **保存什么？**
    - 串口设置（端口、波特率...）
    - 窗口几何位置（下次打开还在原来的位置）
    - 发送区的历史文本

- **存在哪？**
    - `config.json` 文件位于程序根目录，随拷随走。

---

## 总结与展望

<div class="description">
通过本项目，我们实现了一个功能完备的串口调试工具，验证了 PyQt5 在工控领域的强大能力。
</div>

### 🎯 项目成果
1.  **稳定性**：依托 Qt 的底层机制，解决了丢包和卡顿问题。
2.  **可用性**：自动保存和智能补全大大降低了操作门槛。
3.  **扩展性**：MVC 架构为未来增加"波形显示"或"自动应答"功能留下了接口。

### 🚀 未来计划
- 加入脚本支持，允许用户编写 Python 脚本自动处理数据。
- 增加网络转发功能，实现串口转 TCP/IP。

---

<!-- _class: lead -->

# 谢谢观看
## Q & A

<div class="description">
欢迎提问，共同探讨 Python 桌面开发技术
</div>
