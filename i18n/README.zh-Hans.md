[English](../README.md) · [العربية](README.ar.md) · [Español](README.es.md) · [Français](README.fr.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Tiếng Việt](README.vi.md) · [中文 (简体)](README.zh-Hans.md) · [中文（繁體）](README.zh-Hant.md) · [Deutsch](README.de.md) · [Русский](README.ru.md)


[![LazyingArt banner](https://github.com/lachlanchen/lachlanchen/raw/main/figs/banner.png)](https://github.com/lachlanchen/lachlanchen/blob/main/figs/banner.png)

# NHI 硬件控制与事件捕获

![Python](https://img.shields.io/badge/Python-3.x-blue)
![Platform](https://img.shields.io/badge/Platform-Windows%20%2F%20Linux-informational)
![Status](https://img.shields.io/badge/Status-Research%20Prototype-orange)
![Hardware](https://img.shields.io/badge/Hardware-EVK5%20%7C%20FMC4030%20%7C%20Arduino-success)
![UI](https://img.shields.io/badge/Web_UI-Tornado-0ea5e9)
![Docs](https://img.shields.io/badge/Docs-English%20%2B%20i18n-0f766e)

## 📖 快速导航

| 使用本节 | 用途 |
|---|---|
| 安装 | 准备环境和依赖 |
| 使用方法 | 运行 Web 编排器和 CLI 工作流 |
| 配置 | 调整串口、网络与默认值 |
| 示例 | 运行实际命令示例 |
| 故障排查 | 修复常见设置问题 |

## 🧭 项目概览

| 关注点 | 细节 |
|---|---|
| 使命 | 为可重复的实验流程协调事件捕获、运动控制与 LED 信号 |
| 核心入口 | `app.py`（Tornado Web 触发 + 异步序列编排） |
| 关键输入 | EVK5 事件流、FMC4030 轴控制、Arduino 串口命令 |
| 主要输出 | `data/axis_1_positions.csv`，可选事件 CSV 导出 |
| 平台 | Windows / Linux（受 SDK 与硬件可用性影响） |

这是一个面向事件相机实验的硬件编排项目，结合了：
- EVK5 事件相机采集（Prophesee Metavision 技术栈）
- 基于 FMC4030 的 CNC 运动控制
- Arduino 串口 LED 控制
- 精简 Web 触发界面（Tornado）

> 假设：硬件、DLL 与 SDK 环境依赖于主机并由项目代码推断；实际命令行为可能因操作系统、驱动版本与运行时可用性而异。

## 🧠 概览

主要端到端工作流由 `app.py` 实现：

1. 可选：将现有 `data/` 移入按时间戳命名的文件夹（`data_YYYYMMDD_HHMMSS`）
2. 连接 Arduino LED（默认 `COM4`）
3. 通过 `cnc/FMC4030Lib-x64-20220329/FMC4030-Dll.dll` 初始化 CNC 控制器
4. 执行 LED 与 Y 轴动作序列
5. 可选记录 EVK5 事件（当前主流程中已禁用，见下文说明）
6. 将轴位置信息保存到 `data/axis_1_positions.csv`

### 工作流快照

| 阶段 | 组件 | 输出 |
|---|---|---|
| 触发 | Tornado `/start` | 异步序列执行 |
| 运动 | FMC4030 控制器 | 轴运动 + 位置轮询 |
| 灯光 | Arduino 串口（`'1'` / `'0'`） | LED 状态控制 |
| 采集 | EVK5 + Metavision | 事件流 / CSV 导出 |
| 持久化 | 本地文件系统 | `data/*.csv`、轮转文件夹 |

仓库还包含替代/旧版相机脚本、帧后处理工具，以及打包的 Metavision Python 示例。

## ✨ 功能

- Tornado Web 端点（`/start`）用于异步启动运动/采集序列
- 通过 Metavision HAL 触发通道使能（`MAIN`）进行 EVK5 事件记录
- 导出事件为 CSV，包含事件时间戳与系统时间戳
- 使用 `ctypes` 与厂商 DLL 的 FMC4030 电机控制封装
- 轴运动过程中的运动位置日志输出到 CSV
- Arduino LED 串口控制（`'1'`/`'0'` 命令）
- 帧工具脚本（`.npy` 形状检查和 `.npy` 转 MP4）
- 携带 `python_samples/` Metavision 示例，方便实验与参考

## 🗂️ 项目结构

```text
.
├── app.py                                   # Main web orchestrator
├── event_sensor_evk5.py                     # EVK5 recorder (Metavision)
├── event_sensor.py                          # Alternative recorder (dv package)
├── evk5_test.py                             # Minimal EVK5 test recorder
├── EventCamera_extTrig_Version 240506_ForEVK5.py
├── led.py                                   # Arduino LED serial control
├── npy2video.py                             # Convert NPY frame stack -> MP4
├── npy_shape.py                             # Print frame-array shapes
├── motor_system.ini                         # Soft-origin config
├── cnc/
│   ├── cnc.py                               # FMC4030 control + CLI
│   ├── motor_system.ini
│   ├── FMC4030Lib-x64-20220329/
│   │   ├── FMC4030-Dll.dll
│   │   ├── FMC4030-Dll.h
│   │   └── FMC4030-Dll.lib
│   └── archived/                            # Older controller scripts
├── led/
│   └── led.ino                              # Arduino firmware sketch
├── templates/
│   └── index.html                           # Start button UI
├── python_samples/                          # Metavision sample programs
├── data-0503/                               # Historical experiment datasets
├── i18n/                                    # Reserved for translated READMEs
└── .auto-readme-work/20260228_231403/      # README pipeline artifacts
```

## 🧰 前置条件

### 硬件

- EVK5 兼容的事件相机及其驱动/SDK
- 可通过已配置 IP/端口访问的 FMC4030 兼容运动控制器
- 用于 LED 控制的 Arduino 板卡

### 软件

- Python 3.x
- 厂商/运行时支持：
  - Prophesee Metavision Python 模块（`metavision_core`、`metavision_hal`、相关 SDK 模块）
  - 通过 Python `ctypes` 加载 FMC4030 DLL（`cnc/cnc.py` 中的 `windll` 用法表明 CNC 路径偏向 Windows）
- 脚本中使用的 Python 库：
  - `tornado`、`numpy`、`opencv-python`、`pytz`、`pyserial`
  - 可选/替代栈：`dv`

### 环境默认值

| 设置项 | 默认值 | 位置 |
|---|---|---|
| Arduino 串口 | `COM4` | `app.py`、`led.py` |
| CNC IP | `192.168.0.30` | `cnc/cnc.py` |
| CNC 端口 | `8088` | `cnc/cnc.py` |

说明：
- 当前快照中没有 `requirements.txt` 或 `pyproject.toml`。
- `app.py` 与 `led.py` 中串口默认值均为 `COM4`。
- `cnc/cnc.py` 中默认 CNC 网络配置为 IP `192.168.0.30`、端口 `8088`。

## 🔧 安装

1. 克隆仓库：

```bash
git clone <your-repo-url>
cd nhi_hardware__lachlanchen
```

2. 创建并激活 Python 环境：

```bash
python -m venv .venv
# Windows
.venv\Scripts\activate
# Linux/macOS
source .venv/bin/activate
```

3. 安装核心脚本所用的基础依赖：

```bash
pip install tornado numpy opencv-python pytz pyserial
```

4. 安装在你环境中所需的相机 SDK 依赖：

```bash
# Example placeholder: install Prophesee Metavision Python packages
# Follow your SDK/distribution instructions for your OS and camera version.
```

## 🧪 使用方法

### 1) Web 编排序列（主流程）

从仓库根目录运行（`app.py` 的相对路径解析依赖于此）：

```bash
python app.py
```

然后打开：

```text
http://localhost:8888
```

点击 **Start Sequence** 触发运动/LED 工作流。

`app.py` 当前解析的可选参数：

```bash
python app.py --record_events True
```

关键行为说明：`start_sequence()` 在执行过程中会将 `record_events = False` 重新设置，因此除非修改代码，否则 EVK5 记录可能仍保持禁用。

### 2) EVK5 事件采集（直接 CLI）

```bash
python event_sensor_evk5.py -i "" -d 10 -z Asia/Hong_Kong -o event_output
```

参数：
- `-i, --input`：EVK5 设备或录制的输入源/路径
- `-d, --duration`：录制时长（秒）
- `-z, --timezone`：时间戳格式化使用的时区标签
- `-o, --output`：输出基础名称（保存为 `data/<name>.csv`）

### 3) CNC 控制（直接 CLI）

从 `cnc/` 目录运行，以确保 `cnc.py` 中的相对 DLL 路径能正确解析：

```bash
cd cnc
python cnc.py --axis 1 --dir 1 --distance 30 --speed 20
```

其他可用选项：

```bash
# Set current position as soft origin
python cnc.py --set-origin

# Move to absolute coordinates x,y,z
python cnc.py --move 0,30,0 --speed 20

# Set origin after moving to provided coordinates
python cnc.py --set-origin 0,30,0 --speed 20
```

### 4) LED 测试脚本

```bash
python led.py
```

如果未使用 `COM4`，请在代码中更新端口。

### 5) 实用工具

```bash
# Print frame-array shape from one folder
python npy_shape.py data_20250101_120000

# Print frame-array shape for all data_* folders in cwd
python npy_shape.py -a
```

`npy2video.py` 提供 `npy_to_video(npy_file_path, output_video_path, fps=5)`，可直接导入，也可按你的本地路径进行修改。

## ⚙️ 配置

- `motor_system.ini` 和 `cnc/motor_system.ini`：
  - 持久化软件原点坐标（`ORIGIN` 段用于 X/Y/Z）
- `app.py`：
  - Arduino 串口：`ArduinoLED(port='COM4')`
  - CNC DLL 路径：`cnc/FMC4030Lib-x64-20220329/FMC4030-Dll.dll`
- `event_sensor.py`（DV 路径）：
  - `DV_PORT` 默认 `7777`
  - `DV_PORT_FRAME` 默认 `7778`

## 📸 示例

### 示例 A：通过 Web 启动完整序列

```bash
python app.py
# visit http://localhost:8888 and press Start Sequence
```

预期输出包含：
- `data/axis_1_positions.csv`（CNC 位置轨迹）
- 若主流程中启用事件采集，将额外生成 `data/<events>.csv`

### 示例 B：记录 60 秒 EVK5 事件

```bash
python event_sensor_evk5.py -d 60 -o run_001_events -z Asia/Hong_Kong
```

预期输出：
- `data/run_001_events.csv`

### 示例 C：通过 CLI 让 Y 轴往复运动

```bash
cd cnc
python cnc.py --axis 1 --dir 1 --distance 30 --speed 100
python cnc.py --axis 1 --dir -1 --distance 30 --speed 100
```

## 🧭 开发说明

- 当前仓库更像是一个研究/原型工作区，混合了在用脚本与归档脚本。
- 提交了较大的生成产物（`event_output.csv`、`data-0503/`）；若仓库对外分发，建议制定数据保留策略并更新 `.gitignore`。
- `python_samples/` 包含有用的相机 SDK 示例，但可能包含核心编排不必需的依赖。
- 可能的代码质量改进点：
  - `app.py` 中 `argparse` 的布尔值处理可改进（CLI 中 `type=bool` 往往具有误导性）。
  - `start_sequence()` 中事件录制标志目前会覆盖初始 CLI 值。

## 🛠️ 故障排查

- `ImportError: metavision_*` 模块缺失：
  - 安装并配置你的 Metavision SDK Python 环境。
- `ImportError: No module named dv`：
  - 若使用 `event_sensor.py` 路径，请安装 DV Python 包。
- CNC DLL 加载失败：
  - 确认操作系统兼容性，以及 `FMC4030-Dll.dll` 是否位于预期的相对路径。
  - 在 `cnc/` 目录下运行 `cnc.py`，或调整 `dll_path`。
- LED 串口连接错误：
  - 检查 Arduino 端口设置（`COM4` 与实际端口是否匹配）。
  - 确保没有其他进程占用该串口设备。
- Web 运行后 `data/` 没有文件：
  - 核实写入权限，以及事件采集路径是否已启用。

## 🗺️ 发展路线图

- 添加依赖清单（`requirements.txt` 或 `pyproject.toml`）及版本锁定
- 将运行时配置（端口、IP、DLL 路径、速度参数）外置到统一配置文件
- 将摄像头后端（EVK5 与 DV）统一到同一接口并明确模式选择
- 为运动与传感器接口补充测试/模拟，以便在无硬件时运行 CI
- 为每次实验增加结构化日志和运行元数据
- 在 `i18n/` 下生成并维护多语言 README

## 🤝 贡献

欢迎以下方向的贡献：
- 改进硬件抽象层
- 更好的配置能力和可复现性
- 文档扩展与翻译
- 为运动控制补充安全检查和运行保护

建议的贡献流程：
1. Fork 并创建 feature 分支
2. 提交聚焦、便于评审的改动
3. 在你的硬件环境中验证
4. 提交包含可复现步骤和日志的 Pull Request

## 许可证

该仓库当前快照中未包含许可证文件。

假设：在明确添加项目许可证前，默认保留所有权利。请补充 `LICENSE` 文件以明确复用条款。


## ❤️ Support

| Donate | PayPal | Stripe |
| --- | --- | --- |
| [![Donate](https://camo.githubusercontent.com/24a4914f0b42c6f435f9e101621f1e52535b02c225764b2f6cc99416926004b7/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f446f6e6174652d4c617a79696e674172742d3045413545393f7374796c653d666f722d7468652d6261646765266c6f676f3d6b6f2d6669266c6f676f436f6c6f723d7768697465)](https://chat.lazying.art/donate) | [![PayPal](https://camo.githubusercontent.com/d0f57e8b016517a4b06961b24d0ca87d62fdba16e18bbdb6aba28e978dc0ea21/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f50617950616c2d526f6e677a686f754368656e2d3030343537433f7374796c653d666f722d7468652d6261646765266c6f676f3d70617970616c266c6f676f436f6c6f723d7768697465)](https://paypal.me/RongzhouChen) | [![Stripe](https://camo.githubusercontent.com/1152dfe04b6943afe3a8d2953676749603fb9f95e24088c92c97a01a897b4942/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f5374726970652d446f6e6174652d3633354246463f7374796c653d666f722d7468652d6261646765266c6f676f3d737472697065266c6f676f436f6c6f723d7768697465)](https://buy.stripe.com/aFadR8gIaflgfQV6T4fw400) |
