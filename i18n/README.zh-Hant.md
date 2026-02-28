[English](../README.md) · [العربية](README.ar.md) · [Español](README.es.md) · [Français](README.fr.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Tiếng Việt](README.vi.md) · [中文 (简体)](README.zh-Hans.md) · [中文（繁體）](README.zh-Hant.md) · [Deutsch](README.de.md) · [Русский](README.ru.md)


# NHI 硬體控制與事件擷取

![Python](https://img.shields.io/badge/Python-3.x-blue)
![Platform](https://img.shields.io/badge/Platform-Windows%20%2F%20Linux-informational)
![Status](https://img.shields.io/badge/Status-Research%20Prototype-orange)
![Hardware](https://img.shields.io/badge/Hardware-EVK5%20%7C%20FMC4030%20%7C%20Arduino-success)
![UI](https://img.shields.io/badge/Web_UI-Tornado-0ea5e9)

這是一個用於事件相機實驗的硬體協同控制專案，整合了：
- EVK5 事件相機擷取（Prophesee Metavision 堆疊）
- 基於 FMC4030 的 CNC 運動控制
- Arduino 序列埠 LED 控制
- 精簡 Web 觸發介面（Tornado）

> 本 README 是此儲存庫快照的第一份完整草稿。
> 假設：此 checkout 的根目錄先前不存在 `README.md`，因此本文件是根據原始碼與 pipeline 分析產物建立。

## 概述

主要的端對端流程實作於 `app.py`：

1. 可選擇將現有 `data/` 輪替為帶時間戳記的資料夾（`data_YYYYMMDD_HHMMSS`）
2. 連線至 Arduino LED（預設 `COM4`）
3. 透過 `cnc/FMC4030Lib-x64-20220329/FMC4030-Dll.dll` 初始化 CNC 控制器
4. 執行 LED 與 Y 軸運動序列
5. 可選擇錄製 EVK5 事件（目前在有效流程中停用；見下方說明）
6. 將軸位置記錄儲存至 `data/axis_1_positions.csv`

### 工作流程快照

| 階段 | 元件 | 輸出 |
|---|---|---|
| 觸發 | Tornado `/start` | 非同步序列執行 |
| 運動 | FMC4030 控制器 | 軸移動 + 位置輪詢 |
| 照明 | Arduino 序列埠（`'1'` / `'0'`） | LED 狀態控制 |
| 感測 | EVK5 + Metavision | 事件串流 / CSV 匯出 |
| 持久化 | 本機檔案系統 | `data/*.csv`、輪替資料夾 |

此儲存庫也包含替代/舊版相機腳本、影格後處理工具，以及隨附的 Metavision Python 範例。

## 功能

- Tornado Web 端點（`/start`）可非同步啟動運動/擷取序列
- 透過 Metavision HAL 啟用觸發通道（`MAIN`）進行 EVK5 事件錄製
- 事件 CSV 匯出，包含事件時間戳與系統時間戳
- 使用 `ctypes` 與廠商 DLL 的 FMC4030 馬達控制封裝
- 軸移動期間的位置 CSV 記錄
- Arduino LED 序列埠控制（`'1'`/`'0'` 指令）
- 影格工具腳本（`.npy` 形狀檢查與 `.npy` 轉 MP4）
- 隨附 `python_samples/` Metavision 範例，供實驗與參考

## 專案結構

```text
.
├── app.py                                   # 主 Web 協同控制器
├── event_sensor_evk5.py                     # EVK5 錄製器（Metavision）
├── event_sensor.py                          # 替代錄製器（dv 套件）
├── evk5_test.py                             # 精簡 EVK5 測試錄製器
├── EventCamera_extTrig_Version 240506_ForEVK5.py
├── led.py                                   # Arduino LED 序列控制
├── npy2video.py                             # 將 NPY 影格堆疊轉為 MP4
├── npy_shape.py                             # 輸出影格陣列形狀
├── motor_system.ini                         # 軟原點設定
├── cnc/
│   ├── cnc.py                               # FMC4030 控制 + CLI
│   ├── motor_system.ini
│   ├── FMC4030Lib-x64-20220329/
│   │   ├── FMC4030-Dll.dll
│   │   ├── FMC4030-Dll.h
│   │   └── FMC4030-Dll.lib
│   └── archived/                            # 較舊控制器腳本
├── led/
│   └── led.ino                              # Arduino 韌體 sketch
├── templates/
│   └── index.html                           # Start button UI
├── python_samples/                          # Metavision 範例程式
├── data-0503/                               # 歷史實驗資料集
├── i18n/                                    # 保留給翻譯版 README
└── .auto-readme-work/20260228_231403/      # README pipeline 產物
```

## 先決條件

### 硬體

- 與 EVK5 相容的事件相機，以及對應驅動程式/SDK
- 可透過設定 IP/port 連線的 FMC4030 相容運動控制器
- 用於 LED 控制的 Arduino 開發板

### 軟體

- Python 3.x
- 下列廠商/執行環境支援：
  - Prophesee Metavision Python 模組（`metavision_core`、`metavision_hal`、相關 SDK 模組）
  - 透過 Python `ctypes` 載入 FMC4030 DLL（`cnc/cnc.py` 中使用 `windll`，表示 CNC 路徑需 Windows）
- 各腳本使用的 Python 函式庫：
  - `tornado`, `numpy`, `opencv-python`, `pytz`, `pyserial`
  - 可選/替代堆疊：`dv`

### 預設環境設定

| 設定 | 預設值 | 位置 |
|---|---|---|
| Arduino 序列埠 | `COM4` | `app.py`, `led.py` |
| CNC IP | `192.168.0.30` | `cnc/cnc.py` |
| CNC port | `8088` | `cnc/cnc.py` |

說明：
- 目前快照中沒有 `requirements.txt` 或 `pyproject.toml`。
- `app.py` 與 `led.py` 的序列埠預設皆為 `COM4`。
- `cnc/cnc.py` 的 CNC 預設網路設定為 IP `192.168.0.30`、port `8088`。

## 安裝

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

3. 安裝核心腳本使用的基礎 Python 相依套件：

```bash
pip install tornado numpy opencv-python pytz pyserial
```

4. 安裝你環境中所需的相機 SDK 相依套件：

```bash
# Example placeholder: install Prophesee Metavision Python packages
# Follow your SDK/distribution instructions for your OS and camera version.
```

## 使用方式

### 1) Web 協同流程（主要）

請從儲存庫根目錄執行（這對 `app.py` 的相對路徑很重要）：

```bash
python app.py
```

然後開啟：

```text
http://localhost:8888
```

點擊 **Start Sequence** 以觸發運動/LED 工作流程。

應用程式目前會解析的可選參數：

```bash
python app.py --record_events True
```

重要行為說明：`start_sequence()` 目前會在後續執行中將 `record_events = False` 重設，因此除非調整程式碼，EVK5 錄製可能仍維持停用。

### 2) EVK5 事件錄製（直接 CLI）

```bash
python event_sensor_evk5.py -i "" -d 10 -z Asia/Hong_Kong -o event_output
```

參數：
- `-i, --input`：EVK5 裝置或錄製來源的輸入來源/路徑
- `-d, --duration`：錄製秒數
- `-z, --timezone`：用於時間戳格式化的時區標籤
- `-o, --output`：輸出基底名稱（儲存為 `data/<name>.csv`）

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

若不是使用 `COM4`，請在程式碼中更新埠號。

### 5) 工具

```bash
# Print frame-array shape from one folder
python npy_shape.py data_20250101_120000

# Print frame-array shape for all data_* folders in cwd
python npy_shape.py -a
```

`npy2video.py` 提供 `npy_to_video(npy_file_path, output_video_path, fps=5)`，可直接匯入使用，或依本機路徑需求修改。

## 設定

- `motor_system.ini` 與 `cnc/motor_system.ini`：
  - 持久化儲存軟體原點座標（X/Y/Z 的 `ORIGIN` 區段）
- `app.py`：
  - Arduino 序列埠：`ArduinoLED(port='COM4')`
  - CNC DLL 路徑：`cnc/FMC4030Lib-x64-20220329/FMC4030-Dll.dll`
- `event_sensor.py`（DV 路徑）：
  - `DV_PORT` 預設 `7777`
  - `DV_PORT_FRAME` 預設 `7778`

## 範例

### 範例 A：透過 Web 啟動完整序列

```bash
python app.py
# visit http://localhost:8888 and press Start Sequence
```

預期輸出包含：
- `data/axis_1_positions.csv`（CNC 位置軌跡）
- 若有效流程中啟用事件錄製，則會有 `data/<events>.csv`

### 範例 B：錄製 60 秒 EVK5 事件

```bash
python event_sensor_evk5.py -d 60 -o run_001_events -z Asia/Hong_Kong
```

預期輸出：
- `data/run_001_events.csv`

### 範例 C：透過 CLI 讓 Y 軸往返移動

```bash
cd cnc
python cnc.py --axis 1 --dir 1 --distance 30 --speed 100
python cnc.py --axis 1 --dir -1 --distance 30 --speed 100
```

## 開發備註

- 目前儲存庫看起來是研究/原型工作區，混合了啟用中與封存腳本。
- 大型產生檔（`event_output.csv`、`data-0503/`）已提交；若此儲存庫要對外發佈，建議制定資料保留策略並更新 `.gitignore`。
- `python_samples/` 包含實用的相機 SDK 範例，但可能含有核心協同流程不需要的相依項目。
- 可能的程式碼品質改進：
  - `app.py` 的 `argparse` 布林值處理可改善（CLI 中 `type=bool` 常造成誤解）。
  - `start_sequence()` 目前對事件錄製旗標的處理會覆蓋初始 CLI 值。

## 疑難排解

- `ImportError: metavision_*` 模組缺失：
  - 安裝/設定你的 Metavision SDK Python 環境。
- `ImportError: No module named dv`：
  - 若使用 `event_sensor.py` 路徑，請安裝 DV Python 套件。
- CNC DLL 載入失敗：
  - 確認 OS 相容性，並確認 `FMC4030-Dll.dll` 位於預期相對路徑。
  - 請在 `cnc/` 目錄執行 `cnc.py`，或調整 `dll_path`。
- LED 序列連線錯誤：
  - 檢查 Arduino 埠號設定（`COM4` 是否為實際埠號）。
  - 確保沒有其他程序佔用序列裝置。
- Web 執行後 `data/` 沒有檔案：
  - 確認寫入權限，以及事件錄製路徑是否已啟用。

## 路線圖

- 新增相依清單（`requirements.txt` 或 `pyproject.toml`）並固定版本
- 將執行時設定（埠、IP、DLL 路徑、速度設定檔）外部化到統一設定檔
- 將相機後端（EVK5 與 DV）統一在同一介面並提供明確模式選擇
- 為運動與感測介面新增測試/mock，以便在無硬體時進行 CI
- 新增結構化日誌與每次實驗的執行中繼資料
- 在 `i18n/` 下產生並維護翻譯版 README

## 貢獻

歡迎以下方向的貢獻：
- 硬體抽象層改進
- 更好的設定管理與可重現性
- 文件擴充與翻譯
- 運動控制的安全檢查與操作防護

建議貢獻流程：
1. Fork 並建立功能分支
2. 進行聚焦且易於審查的變更
3. 依你的硬體環境完成驗證
4. 提交包含可重現步驟與日誌的 pull request

## 授權

此儲存庫快照中尚未包含授權檔案。

假設：在明確新增專案授權前，所有權利均保留。請新增 `LICENSE` 檔以定義重用條款。

## 支援

在此快照中未找到 sponsor/donation 中繼資料。若你希望加入支援連結，請在此處以及翻譯版 README 一併補充。
