[English](../README.md) · [العربية](README.ar.md) · [Español](README.es.md) · [Français](README.fr.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Tiếng Việt](README.vi.md) · [中文 (简体)](README.zh-Hans.md) · [中文（繁體）](README.zh-Hant.md) · [Deutsch](README.de.md) · [Русский](README.ru.md)


[![LazyingArt banner](https://github.com/lachlanchen/lachlanchen/raw/main/figs/banner.png)](https://github.com/lachlanchen/lachlanchen/blob/main/figs/banner.png)

# NHI 硬體控制與事件擷取

![Python](https://img.shields.io/badge/Python-3.x-blue)
![Platform](https://img.shields.io/badge/Platform-Windows%20%2F%20Linux-informational)
![Status](https://img.shields.io/badge/Status-Research%20Prototype-orange)
![Hardware](https://img.shields.io/badge/Hardware-EVK5%20%7C%20FMC4030%20%7C%20Arduino-success)
![UI](https://img.shields.io/badge/Web_UI-Tornado-0ea5e9)
![Docs](https://img.shields.io/badge/Docs-English%20%2B%20i18n-0f766e)

## 📖 快速導覽

| 使用區塊 | 用途 |
|---|---|
| 安裝 | 準備環境與相依套件 |
| 用法 | 執行 Web 編排器與 CLI 流程 |
| 組態 | 調整序列埠、網路與預設值 |
| 範例 | 執行實際指令範例 |
| 故障排除 | 修正常見設定問題 |

## 🧭 專案概覽

| 焦點 | 細節 |
|---|---|
| 使命 | 協調事件擷取、運動控制與 LED 指示，打造可重複的實驗流程 |
| 核心入口 | `app.py`（Tornado Web 觸發 + 非同步序列編排） |
| 關鍵輸入 | EVK5 事件串流、FMC4030 軸控、Arduino 序列埠指令 |
| 主要輸出 | `data/axis_1_positions.csv`，可選擇性事件 CSV 匯出 |
| 平台 | Windows / Linux（依 SDK 與硬體可用性而定） |

這是一個用於事件相機實驗的硬體協同控制專案，結合：
- EVK5 事件相機擷取（Prophesee Metavision 技術堆疊）
- 基於 FMC4030 的 CNC 運動控制
- Arduino 序列埠 LED 控制
- 精簡 Web 觸發介面（Tornado）

> 假設：硬體、DLL 與 SDK 環境依主機而異，且多為依專案程式碼推論；實際指令行為可能因作業系統、驅動版本與執行時可用性不同。

## 🧠 概覽

主要端對端流程實作在 `app.py`：

1. 可選：將既有 `data/` 轉入時間戳資料夾（`data_YYYYMMDD_HHMMSS`）
2. 連線 Arduino LED（預設 `COM4`）
3. 透過 `cnc/FMC4030Lib-x64-20220329/FMC4030-Dll.dll` 初始化 CNC 控制器
4. 執行 LED 與 Y 軸運動序列
5. 可選擇錄製 EVK5 事件（目前在主流程已停用，詳見下方說明）
6. 將軸位置記錄儲存到 `data/axis_1_positions.csv`

### 工作流程快照

| 階段 | 元件 | 輸出 |
|---|---|---|
| 觸發 | Tornado `/start` | 非同步序列執行 |
| 運動 | FMC4030 控制器 | 軸移動 + 位置輪詢 |
| 照明 | Arduino 序列埠（`'1'` / `'0'`） | LED 狀態控制 |
| 感測 | EVK5 + Metavision | 事件串流 / CSV 匯出 |
| 永續儲存 | 本機檔案系統 | `data/*.csv`、輪替資料夾 |

本儲存庫也包含替代/舊版相機腳本、影格後處理工具，以及隨附的 Metavision Python 範例。

## ✨ 功能

- Tornado Web 端點（`/start`）可非同步啟動運動/擷取序列
- 透過 Metavision HAL 啟用觸發通道（`MAIN`）進行 EVK5 事件錄製
- 匯出事件到 CSV，包含事件時間戳與系統時間戳
- 使用 `ctypes` 與廠商 DLL 的 FMC4030 馬達控制封裝
- 軸運動過程中的位置日誌輸出到 CSV
- Arduino LED 序列埠控制（`'1'`/`'0'` 指令）
- 影格工具腳本（`.npy` 形狀檢查與 `.npy` 轉 MP4）
- 隨附 `python_samples/` Metavision 範例，供實驗與參考

## 🗂️ 專案結構

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

## 🧰 先決條件

### 硬體

- EVK5 相容的事件相機與其驅動程式／SDK
- 可透過設定 IP/port 存取的 FMC4030 相容運動控制器
- 用於 LED 控制的 Arduino 板子

### 軟體

- Python 3.x
- 廠商/執行時支援：
  - Prophesee Metavision Python 模組（`metavision_core`、`metavision_hal`、相關 SDK 模組）
  - 透過 Python `ctypes` 載入 FMC4030 DLL（`cnc/cnc.py` 內的 `windll` 用法顯示 CNC 路徑偏向 Windows）
- 各腳本使用的 Python 函式庫：
  - `tornado`, `numpy`, `opencv-python`, `pytz`, `pyserial`
  - 可選/替代套件：`dv`

### 環境預設值

| 設定 | 預設值 | 位置 |
|---|---|---|
| Arduino 序列埠 | `COM4` | `app.py`, `led.py` |
| CNC IP | `192.168.0.30` | `cnc/cnc.py` |
| CNC port | `8088` | `cnc/cnc.py` |

註記：
- 目前快照中沒有 `requirements.txt` 或 `pyproject.toml`。
- `app.py` 與 `led.py` 的序列埠預設皆為 `COM4`。
- `cnc/cnc.py` 的 CNC 預設網路設定為 IP `192.168.0.30`、port `8088`。

## 🔧 安裝

1. 複製儲存庫：

```bash
git clone <your-repo-url>
cd nhi_hardware__lachlanchen
```

2. 建立並啟用 Python 環境：

```bash
python -m venv .venv
# Windows
.venv\Scripts\activate
# Linux/macOS
source .venv/bin/activate
```

3. 安裝核心腳本使用的基礎相依套件：

```bash
pip install tornado numpy opencv-python pytz pyserial
```

4. 安裝你環境所需的相機 SDK 相依套件：

```bash
# Example placeholder: install Prophesee Metavision Python packages
# Follow your SDK/distribution instructions for your OS and camera version.
```

## 🧪 用法

### 1) Web 協調序列（主要）

從儲存庫根目錄執行（這對 `app.py` 的相對路徑解析很重要）：

```bash
python app.py
```

然後開啟：

```text
http://localhost:8888
```

點擊 **Start Sequence** 觸發運動與 LED 工作流程。

應用程式目前可解析的可選參數：

```bash
python app.py --record_events True
```

關鍵行為：`start_sequence()` 在執行中目前會再把 `record_events = False` 重設，因此除非修改程式碼，EVK5 錄製可能仍維持停用。

### 2) EVK5 事件錄製（直接 CLI）

```bash
python event_sensor_evk5.py -i "" -d 10 -z Asia/Hong_Kong -o event_output
```

參數：
- `-i, --input`: EVK5 裝置或錄製來源的輸入來源/路徑
- `-d, --duration`: 錄製時長（秒）
- `-z, --timezone`: 用於時間戳格式化的時區標籤
- `-o, --output`: 輸出基底名稱（儲存為 `data/<name>.csv`）

### 3) CNC 控制（直接 CLI）

請從 `cnc/` 執行，讓 `cnc.py` 中的相對 DLL 路徑可正確解析：

```bash
cd cnc
python cnc.py --axis 1 --dir 1 --distance 30 --speed 20
```

其他可用選項：

```bash
# Set current position as soft origin
python cnc.py --set-origin

# Move to absolute coordinates x,y,z
python cnc.py --move 0,30,0 --speed 20

# Set origin after moving to provided coordinates
python cnc.py --set-origin 0,30,0 --speed 20
```

### 4) LED 測試腳本

```bash
python led.py
```

若你未使用 `COM4`，請在程式碼中更新埠號。

### 5) 工具

```bash
# Print frame-array shape from one folder
python npy_shape.py data_20250101_120000

# Print frame-array shape for all data_* folders in cwd
python npy_shape.py -a
```

`npy2video.py` 提供 `npy_to_video(npy_file_path, output_video_path, fps=5)`，可直接匯入使用，或依你的本機路徑自行修改。

## ⚙️ 組態

- `motor_system.ini` 與 `cnc/motor_system.ini`：
  - 持久化軟體原點座標（`ORIGIN` 區段，供 X/Y/Z 使用）
- `app.py`：
  - Arduino 序列埠：`ArduinoLED(port='COM4')`
  - CNC DLL 路徑：`cnc/FMC4030Lib-x64-20220329/FMC4030-Dll.dll`
- `event_sensor.py`（DV 路徑）：
  - `DV_PORT` 預設 `7777`
  - `DV_PORT_FRAME` 預設 `7778`

## 📸 範例

### 範例 A：透過 Web 啟動完整序列

```bash
python app.py
# visit http://localhost:8888 and press Start Sequence
```

預期輸出包含：
- `data/axis_1_positions.csv`（CNC 位置軌跡）
- 若主流程有啟用事件錄製，將另外產生 `data/<events>.csv`

### 範例 B：錄製 60 秒 EVK5 事件

```bash
python event_sensor_evk5.py -d 60 -o run_001_events -z Asia/Hong_Kong
```

預期輸出：
- `data/run_001_events.csv`

### 範例 C：用 CLI 讓 Y 軸往復運動

```bash
cd cnc
python cnc.py --axis 1 --dir 1 --distance 30 --speed 100
python cnc.py --axis 1 --dir -1 --distance 30 --speed 100
```

## 🧭 開發說明

- 當前儲存庫更像研究/原型工作區，混合了可用腳本與封存腳本。
- 已提交了大型產物（例如 `event_output.csv`、`data-0503/`）；若對外發佈，建議補上資料保留策略並更新 `.gitignore`。
- `python_samples/` 有用於相機 SDK 參考，但其依賴不一定都為核心編排流程所需。
- 可能的改進方向：
  - `app.py` 中 `argparse` 的布林值處理可優化（`type=bool` 在 CLI 常會導致預期外行為）。
  - `start_sequence()` 目前會覆寫事件錄製旗標的初始 CLI 值。

## 🛠️ 故障排除

- `ImportError: metavision_*` 模組缺少：
  - 安裝並設定你的 Metavision SDK Python 環境。
- `ImportError: No module named dv`：
  - 若使用 `event_sensor.py` 路徑，請安裝 DV Python 套件。
- CNC DLL 載入失敗：
  - 確認作業系統相容性，以及 `FMC4030-Dll.dll` 是否位於預期相對路徑。
  - 在 `cnc/` 目錄下執行 `cnc.py`，或調整 `dll_path`。
- LED 序列埠連線錯誤：
  - 檢查 Arduino 端口設定（`COM4` 是否為實際連線埠）。
  - 確認沒有其他程式搶用該序列埠。
- Web 運行後 `data/` 未新增檔案：
  - 確認寫入權限，及事件錄製路徑是否已啟用。

## 🗺️ 里程碑

- 新增相依套件清單（`requirements.txt` 或 `pyproject.toml`）並加上版本鎖定
- 將執行期設定（序列埠、IP、DLL 路徑、速度參數）外部化為統一設定檔
- 將相機後端（EVK5 與 DV）整合為共通介面並明確模式選擇
- 為運動與感測介面補齊測試/模擬，以便無硬體時仍能執行 CI
- 為每次實驗增加結構化日誌與運行中繼資料
- 在 `i18n/` 下持續維護多語系 README

## 🤝 貢獻

歡迎以下方向的參與：
- 改善硬體抽象層
- 提升設定能力與可重現性
- 擴充文件與翻譯
- 為運動控制補上安全檢查與保護機制

建議流程：
1. Fork 並建立 feature 分支
2. 提交聚焦、便於 review 的變更
3. 在你的硬體環境中驗證
4. 以可重現步驟與日誌提交 Pull Request

## 授權

此儲存庫目前快照未包含授權檔案。

假設在明確新增專案授權前，預設保留所有權利；請補上 `LICENSE` 以明確授權條款。


## ❤️ Support

| Donate | PayPal | Stripe |
| --- | --- | --- |
| [![Donate](https://camo.githubusercontent.com/24a4914f0b42c6f435f9e101621f1e52535b02c225764b2f6cc99416926004b7/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f446f6e6174652d4c617a79696e674172742d3045413545393f7374796c653d666f722d7468652d6261646765266c6f676f3d6b6f2d6669266c6f676f436f6c6f723d7768697465)](https://chat.lazying.art/donate) | [![PayPal](https://camo.githubusercontent.com/d0f57e8b016517a4b06961b24d0ca87d62fdba16e18bbdb6aba28e978dc0ea21/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f50617950616c2d526f6e677a686f754368656e2d3030343537433f7374796c653d666f722d7468652d6261646765266c6f676f3d70617970616c266c6f676f436f6c6f723d7768697465)](https://paypal.me/RongzhouChen) | [![Stripe](https://camo.githubusercontent.com/1152dfe04b6943afe3a8d2953676749603fb9f95e24088c92c97a01a897b4942/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f5374726970652d446f6e6174652d3633354246463f7374796c653d666f722d7468652d6261646765266c6f676f3d737472697065266c6f676f436f6c6f723d7768697465)](https://buy.stripe.com/aFadR8gIaflgfQV6T4fw400) |
