# 串口监控程序 - UML 图表说明

本目录包含串口监控程序的各种 UML 图表，用于理解系统架构和运行流程。

## 📊 类图 (Class Diagrams)

### 1. class_diagram_simple.puml ⭐ **推荐用于PPT**
- **用途**: 简化版类图，适合PPT展示
- **内容**: 核心类和关键方法
- **特点**: 简洁清晰，一页显示完整

### 2. class_diagram.puml (已重命名为 class_diagram_full.puml)
- **用途**: 完整详细版，适合文档查阅
- **内容**: 所有类的所有方法和属性
- **特点**: 信息完整但较长

## 🔄 序列图 (Sequence Diagrams)

已将原始长序列图切分为 7 个独立流程，每个都适合放入 PPT 的一页：

### seq_01_initialization.puml
**应用程序初始化流程**
- 启动应用
- 加载配置
- 初始化UI
- 连接信号

### seq_02_open_port.puml
**打开串口流程**
- 获取串口参数
- 设置串口配置
- 打开串口
- 成功/失败处理

### seq_03_receive_data.puml
**接收数据流程**
- 串口数据到达
- 读取数据
- 格式化显示（文本/十六进制）
- 时间戳处理

### seq_04_send_data.puml
**发送数据流程**
- 检查串口状态
- 格式转换
- 发送数据
- 统计更新

### seq_05_auto_send.puml
**自动发送流程**
- 启动定时器
- 循环发送
- 停止自动发送

### seq_06_save_config.puml
**配置保存流程**
- 监听设置变更
- 收集配置数据
- 写入JSON文件
- 自动保存机制

### seq_07_close.puml
**关闭串口与退出流程**
- 关闭串口
- 保存配置
- 清理资源
- 应用退出

## 📑 Mermaid 格式

### class_diagram.md
包含 Mermaid 格式的图表，可在 VS Code 中直接预览（Ctrl+Shift+V）

## 🛠️ 如何使用

### 在 VS Code 中查看
1. 安装 PlantUML 插件：`jebbs.plantuml`
2. 打开任意 `.puml` 文件
3. 按 `Alt + D` 预览图表
4. 右键选择 "Export Current Diagram" 导出为 PNG/SVG

### 导入到 PPT
1. 使用 VS Code 导出为 PNG 格式
2. 推荐分辨率：1920x1080 或更高
3. 每个序列图占用一页 PPT
4. 使用简化版类图作为总体架构说明

### 在线查看
可以使用以下在线工具：
- [PlantUML Online Server](http://www.plantuml.com/plantuml/uml/)
- [PlantUML Editor](https://www.planttext.com/)

## 📐 PPT 推荐布局

### 方案一：按流程讲解（推荐）
1. **第1页**: 简化版类图 (class_diagram_simple.puml) - 总体架构
2. **第2页**: 初始化流程 (seq_01)
3. **第3页**: 打开串口 (seq_02)
4. **第4页**: 数据收发 (seq_03 + seq_04 合并)
5. **第5页**: 高级功能 (seq_05 自动发送)
6. **第6页**: 配置管理 (seq_06)
7. **第7页**: 退出流程 (seq_07)

### 方案二：按功能模块
1. **架构概览**: class_diagram_simple.puml
2. **核心功能**: seq_02 + seq_03 + seq_04
3. **高级功能**: seq_05 + seq_06
4. **生命周期**: seq_01 + seq_07

## 🎨 图表特点

- ✅ 中文标注，易于理解
- ✅ 颜色区分不同类型（PyQt5基类、自定义类、外部类）
- ✅ 清晰的注释说明
- ✅ 信号用特殊颜色标注（红色/绿色/蓝色）
- ✅ 每个流程独立完整
- ✅ 适合打印和投影

## 🔧 自定义修改

如需修改图表样式，可以调整以下参数：

```plantuml
' 字体大小
skinparam defaultFontSize 13

' 箭头粗细
skinparam sequenceArrowThickness 2

' 圆角
skinparam roundcorner 20

' 背景色
skinparam BackgroundColor white
```

## 📝 相关文档

- 完整的 Mermaid 格式文档：[class_diagram.md](class_diagram.md)
- 源代码文件：
  - [app_SerialProcess.py](../Serial_Port/app_SerialProcess.py)
  - [app_SerialWindows.py](../Serial_Port/app_SerialWindows.py)
  - [config_manager.py](../Serial_Port/config_manager.py)
