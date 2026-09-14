# WeFlow HTML 聊天记录转换工具

## 项目介绍

本工具用于将 WeFlow 导出的微信聊天记录 HTML 文件批量转换为 **TXT** 和 **XLSX** 两种格式，方便阅读、归档和二次分析。

### 核心功能

- **批量转换**：自动扫描程序同目录下所有 HTML 文件，一键转换
- **智能消息分类**：自动识别文本消息、图片消息、引用消息、通话消息、动画表情、系统消息、链接卡片等
- **引用消息还原**：完整保留引用的发送者和引用内容
- **链接卡片提取**：提取链接标题和 URL
- **XLSX 格式化输出**：生成带表头、边框、列宽调整的 Excel 文件，包含会话元信息（微信ID、昵称、导出时间等）
- **TXT 纯文本输出**：简洁的时间-发送者-内容格式

### 支持的消息类型

| 消息类型 | 说明 |
|--------->|------|
| 文本消息 | 普通文字内容 |
| 图片消息 | 图片类消息，输出为 `[图片]` |
| 引用消息 | 回复/引用其他消息，格式为 `[引用 发送者：内容]` |
| 通话消息 | 语音通话、视频通话 |
| 动画表情 | 表情包，输出为 `[表情包]` |
| 系统消息 | 撤回消息等系统提示 |
| 其他消息 | 转账、拍一拍、链接卡片等 |

## 快速开始

### 面向使用者(使用exe文件)

1. 将 `dist/main.exe` 复制到存放 HTML 聊天记录文件的目录中
2. 双击运行 `main.exe`
3. 程序会自动扫描同目录下所有 `.html` 文件并转换
4. 转换完成后，同目录下会生成对应的 `.txt` 和 `.xlsx` 文件
5. 按任意键退出程序

> **注意**：HTML 文件必须是由 WeFlow 导出的微信聊天记录，包含 `window.WEFLOW_DATA` 数据。

### 面向开发者（从源码运行）
```bash
# 克隆项目
git clone <项目地址>

# 进入项目目录
cd <项目目录>

# 激活虚拟环境
venv\Scripts\activate  # Windows
source venv/bin/activate  # Linux/Mac

# 安装依赖
pip install -r requirements.txt

# 运行程序
python main.py
```

### 打包为 exe
```bash
# 激活虚拟环境后执行
pyinstaller main.spec
```
打包产物位于 `dist/main.exe`。

## 实现方法

### 整体流程

```
HTML 文件 → 正则提取元信息 → 解析 WEFLOW_DATA JSON → 逐条解析消息 → 分类 → 输出 TXT/XLSX
```

### 关键模块

| 模块/函数 | 职责 |
|-----------|------|
| `MessageContentParser` | 基于 `html.parser.HTMLParser` 的自定义 HTML 解析器，提取消息文本、引用内容、链接卡片 |
| `parse_message_body()` | 解析单条消息的 HTML body，返回结构化解析结果 |
| `classify_message()` | 根据解析结果和原始文本对消息进行分类 |
| `build_txt_content()` | 构建 TXT 输出的纯文本内容 |
| `build_xlsx_content()` | 构建 XLSX 输出的单元格内容 |
| `extract_avatar_info()` | 从头像 HTML 中提取 alt 文本和微信ID |
| `parse_html_file()` | 解析整个 HTML 文件，提取会话元信息和所有消息 |
| `write_txt()` | 将解析数据写入 TXT 文件 |
| `write_xlsx()` | 使用 openpyxl 将解析数据写入格式化的 XLSX 文件 |

### HTML 解析策略

- **元信息提取**：使用正则表达式匹配标题、消息数量、聊天类型等
- **消息数据提取**：通过正则匹配 `window.WEFLOW_DATA = [...]` 获取 JSON 数据
- **消息内容解析**：使用 Python 标准库 `html.parser.HTMLParser` 的子类 `MessageContentParser`，通过 CSS class 名（`message-text`、`quoted-message`、`quoted-sender`、`quoted-text`、`message-link-card`）识别不同内容区域

### XLSX 输出结构

| 行 | 内容 |
|----|------|
| 第1行 | 会话信息（合并单元格标题） |
| 第2行 | 微信ID、昵称 |
| 第3行 | 导出工具(WeFlow)、导出版本(0.0.2)、平台(wechat)、导出时间 |
| 第4行 | 列头：序号、时间、发送者身份、消息类型、内容 |
| 第5行起 | 逐条消息数据 |

## 版本日志

### v1.0.0    ----2026.9.14
- 支持 WeFlow 导出的 HTML 聊天记录转换为 TXT 和 XLSX
- 支持消息分类：文本、图片、引用、通话、表情、系统、链接等
- XLSX 输出包含会话元信息和格式化表头
- 支持批量处理同目录下多个 HTML 文件
- 支持 PyInstaller 打包为 exe

## 许可证

本项目采用 Prosperity Public License 2.0.0 许可证，详见 LICENSE 文件。