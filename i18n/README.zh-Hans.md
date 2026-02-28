[English](../README.md) · [العربية](README.ar.md) · [Español](README.es.md) · [Français](README.fr.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Tiếng Việt](README.vi.md) · [中文 (简体)](README.zh-Hans.md) · [中文（繁體）](README.zh-Hant.md) · [Deutsch](README.de.md) · [Русский](README.ru.md)


# NHI 硬件控制与事件捕获

![Python](https://img.shields.io/badge/Python-3.x-blue)
![Platform](https://img.shields.io/badge/Platform-Windows%20%2F%20Linux-informational)
![Status](https://img.shields.io/badge/Status-Research%20Prototype-orange)
![Hardware](https://img.shields.io/badge/Hardware-EVK5%20%7C%20FMC4030%20%7C%20Arduino-success)
![UI](https://img.shields.io/badge/Web_UI-Tornado-0ea5e9)

这是一个面向事件相机实验的硬件协同项目，整合了：
- EVK5 事件相机采集（Prophesee Metavision 技术栈）
- 基于 FMC4030 的 CNC 运动控制
- Arduino 串口 LED 控制
- 极简 Web 触发界面（Tornado）

> 本 README 是该仓库当前快照的第一份完整草稿。
> 假设：本次检出中此前不存在根目录 `README.md`，因此本文档由源码与流水线分析产物构建而成。

## 概览

主要端到端流程实现在 `app.py` 中：

1. 可选地将现有 `data/` 轮转到带时间戳的文件夹（`data_YYYYMMDD_HHMMSS`）
2. 连接 Arduino LED（默认 `COM4`）
3. 通过 `cnc/FMC4030Lib-x64-20220329/FMC4030-Dll.dll` 初始化 CNC 控制器
4. 执行 LED 与 Y 轴运动序列
5. 可选地记录 EVK5 事件（当前活跃流程中默认禁用；见下方说明）
6. 将轴位置日志保存到 `data/axis_1_positions.csv`

### 流程快照

| 阶段 | 组件 | 输出 |
|---|---|---|
| 触发 | Tornado `/start` | 异步序列执行 |
| 运动 | FMC4030 控制器 | 轴运动 + 位置轮询 |
| 灯光 | Arduino 串口（`'1'` / `'0'`） | LED 状态控制 |
| 传感 | EVK5 + Metavision | 事件流 / CSV 导出 |
| 持久化 | 本地文件系统 | `data/*.csv`、轮转文件夹 |

仓库中还包含了替代/历史相机脚本、帧后处理工具，以及打包的 Metavision Python 示例。

## 功能特性

- Tornado Web 端点（`/start`）用于异步启动运动/采集序列
- 通过 Metavision HAL 启用触发通道（`MAIN`）的 EVK5 事件录制
- 事件数据导出为 CSV，同时包含事件时间戳与系统时间戳
- 使用 `ctypes` 与厂商 DLL 的 FMC4030 电机控制封装
- 轴运动过程中的位置日志 CSV 记录
- Arduino LED 串口控制（`'1'`/`'0'` 命令）
- 帧处理工具脚本（`.npy` 形状检查与 `.npy` 转 MP4）
- 附带 `python_samples/` Metavision 示例，便于实验与参考

## 项目结构

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

## 前置条件

### 硬件

- 兼容 EVK5 的事件相机及其驱动/SDK
- 可通过已配置 IP/端口访问的 FMC4030 兼容运动控制器
- 用于 LED 控制的 Arduino 开发板

### 软件

- Python 3.x
- 以下厂商/运行时支持：
  - Prophesee Metavision Python 模块（`metavision_core`、`metavision_hal` 及相关 SDK 模块）
  - 通过 Python `ctypes` 加载 FMC4030 DLL（`cnc/cnc.py` 中使用 `windll`，意味着 CNC 路径偏向 Windows）
- 各脚本使用到的 Python 库：
  - `tornado`、`numpy`、`opencv-python`、`pytz`、`pyserial`
  - 可选/替代技术栈：`dv`

### 环境默认值

| 设置项 | 默认值 | 位置 |
|---|---|---|
| Arduino 串口 | `COM4` | `app.py`, `led.py` |
| CNC IP | `192.168.0.30` | `cnc/cnc.py` |
| CNC 端口 | `8088` | `cnc/cnc.py` |

说明：
- 当前快照中没有 `requirements.txt` 或 `pyproject.toml`。
- `app.py` 与 `led.py` 中串口默认值为 `COM4`。
- `cnc/cnc.py` 中 CNC 默认网络配置：IP `192.168.0.30`，端口 `8088`。

## 安装

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

3. 安装核心脚本使用的基础 Python 依赖：

```bash
pip install tornado numpy opencv-python pytz pyserial
```

4. 安装你所在环境所需的相机 SDK 依赖：

```bash
# Example placeholder: install Prophesee Metavision Python packages
# Follow your SDK/distribution instructions for your OS and camera version.
```

## 使用方法

### 1) Web 编排序列（主要方式）

在仓库根目录运行（这对 `app.py` 中的相对路径很重要）：

```bash
python app.py
```

然后打开：

```text
http://localhost:8888
```

点击 **Start Sequence** 以触发运动/LED 工作流。

应用当前会解析的可选参数：

```bash
python app.py --record_events True
```

重要行为说明：`start_sequence()` 在后续执行中目前会将 `record_events = False` 重置，因此除非调整代码，否则 EVK5 录制可能仍保持禁用。

### 2) EVK5 事件录制（直接 CLI）

```bash
python event_sensor_evk5.py -i "" -d 10 -z Asia/Hong_Kong -o event_output
```

参数：
- `-i, --input`：EVK5 设备或录制文件的输入源/路径
- `-d, --duration`：录制时长（秒）
- `-z, --timezone`：时间戳格式化所用时区标签
- `-o, --output`：输出基础名称（保存到 `data/<name>.csv`）

### 3) CNC 控制（直接 CLI）

从 `cnc/` 目录运行，以便 `cnc.py` 中的相对 DLL 路径能正确解析：

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

若不是使用 `COM4`，请在代码中更新端口。

### 5) 实用工具

```bash
# Print frame-array shape from one folder
python npy_shape.py data_20250101_120000

# Print frame-array shape for all data_* folders in cwd
python npy_shape.py -a
```

`npy2video.py` 提供 `npy_to_video(npy_file_path, output_video_path, fps=5)`，可直接导入使用，或按你的本地路径进行修改。

## 配置

- `motor_system.ini` 和 `cnc/motor_system.ini`：
  - 持久化软件原点坐标（X/Y/Z 的 `ORIGIN` 段）
- `app.py`：
  - Arduino 串口：`ArduinoLED(port='COM4')`
  - CNC DLL 路径：`cnc/FMC4030Lib-x64-20220329/FMC4030-Dll.dll`
- `event_sensor.py`（DV 路径）：
  - `DV_PORT` 默认 `7777`
  - `DV_PORT_FRAME` 默认 `7778`

## 示例

### 示例 A：通过 Web 启动完整序列

```bash
python app.py
# visit http://localhost:8888 and press Start Sequence
```

预期输出包括：
- `data/axis_1_positions.csv`（CNC 位置轨迹）
- 若活跃路径中启用事件录制，则可选生成 `data/<events>.csv`

### 示例 B：录制 60 秒 EVK5 事件

```bash
python event_sensor_evk5.py -d 60 -o run_001_events -z Asia/Hong_Kong
```

预期输出：
- `data/run_001_events.csv`

### 示例 C：通过 CLI 让 Y 轴往返运动

```bash
cd cnc
python cnc.py --axis 1 --dir 1 --distance 30 --speed 100
python cnc.py --axis 1 --dir -1 --distance 30 --speed 100
```

## 开发说明

- 当前仓库更像是一个研究/原型工作区，包含混合的在用脚本和归档脚本。
- 大体量生成产物（`event_output.csv`、`data-0503/`）已被提交；若此仓库将对外分发，建议制定数据保留策略并更新 `.gitignore`。
- `python_samples/` 含有实用的相机 SDK 示例，但其中可能包含并非核心编排所必需的依赖。
- 潜在代码质量改进点：
  - 可改进 `app.py` 中的 `argparse` 布尔处理（CLI 解析中 `type=bool` 往往会产生误导）。
  - `start_sequence()` 中事件录制标志目前会覆盖初始 CLI 值。

## 故障排查

- `ImportError: metavision_*` 模块缺失：
  - 安装/配置 Metavision SDK 的 Python 环境。
- `ImportError: No module named dv`：
  - 若使用 `event_sensor.py` 路径，请安装 DV Python 包。
- CNC DLL 加载失败：
  - 确认操作系统兼容性，以及 `FMC4030-Dll.dll` 位于预期的相对路径。
  - 在 `cnc/` 目录下运行 `cnc.py`，或调整 `dll_path`。
- LED 串口连接错误：
  - 检查 Arduino 端口分配（`COM4` 与实际端口是否一致）。
  - 确保没有其他进程占用串口设备。
- Web 运行后 `data/` 中没有文件：
  - 检查写入权限，以及事件录制路径是否已启用。

## 路线图

- 添加依赖清单（`requirements.txt` 或 `pyproject.toml`）并固定版本
- 将运行时配置（端口、IP、DLL 路径、速度曲线）外置到统一配置文件
- 在统一接口下规范化相机后端（EVK5 与 DV），并提供清晰模式选择
- 为运动与传感器接口添加测试/Mock，使 CI 可在无硬件条件下运行
- 为每次实验增加结构化日志与运行元数据
- 在 `i18n/` 下生成并维护多语言 README 文件

## 贡献

欢迎围绕以下方向贡献：
- 硬件抽象改进
- 更好的配置与可复现性
- 文档扩展与翻译
- 面向运动控制的安全检查与运行护栏

建议的贡献流程：
1. Fork 并创建功能分支
2. 提交聚焦、便于审查的改动
3. 在你的硬件环境中完成验证
4. 提交包含可复现实验步骤与日志的 Pull Request

## 许可证

该仓库当前快照中未包含许可证文件。

假设：在明确添加项目许可证之前，默认保留所有权利。请添加 `LICENSE` 文件以定义复用条款。

## 支持

当前快照中未发现 sponsor/donation 元数据。若你希望加入支持链接，请在此处和各语言 README 中一并添加。
