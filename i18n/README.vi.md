[English](../README.md) · [العربية](README.ar.md) · [Español](README.es.md) · [Français](README.fr.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Tiếng Việt](README.vi.md) · [中文 (简体)](README.zh-Hans.md) · [中文（繁體）](README.zh-Hant.md) · [Deutsch](README.de.md) · [Русский](README.ru.md)


[![LazyingArt banner](https://github.com/lachlanchen/lachlanchen/raw/main/figs/banner.png)](https://github.com/lachlanchen/lachlanchen/blob/main/figs/banner.png)

# Điều Khiển Phần Cứng NHI và Thu Thập Sự Kiện

![Python](https://img.shields.io/badge/Python-3.x-blue)
![Platform](https://img.shields.io/badge/Platform-Windows%20%2F%20Linux-informational)
![Status](https://img.shields.io/badge/Status-Research%20Prototype-orange)
![Hardware](https://img.shields.io/badge/Hardware-EVK5%20%7C%20FMC4030%20%7C%20Arduino-success)
![UI](https://img.shields.io/badge/Web_UI-Tornado-0ea5e9)
![Docs](https://img.shields.io/badge/Docs-English%20%2B%20i18n-0f766e)

## 📖 Điều Hướng Nhanh

| Mục dùng | Mục đích |
|---|---|
| [Cài đặt](#installation) | Chuẩn bị môi trường và phụ thuộc |
| [Sử dụng](#usage) | Chạy orchestrator web và luồng CLI |
| [Cấu hình](#configuration) | Điều chỉnh serial, mạng, và các giá trị mặc định |
| [Ví dụ](#examples) | Chạy các ví dụ lệnh thực tế |
| [Khắc phục sự cố](#troubleshooting) | Sửa các lỗi thiết lập phổ biến |

## 🧭 Tóm Tắt Dự Án

| Trọng tâm | Chi tiết |
|---|---|
| Mục tiêu | Điều phối thu thập sự kiện, điều khiển chuyển động và điều khiển đèn LED cho quy trình thí nghiệm phòng lab có thể lặp lại |
| Điểm vào chính | `app.py` (tuyến kích hoạt web Tornado + điều phối chuỗi bất đồng bộ) |
| Dữ liệu đầu vào chính | Luồng sự kiện EVK5, điều khiển trục FMC4030, lệnh serial Arduino |
| Kết quả chính | `data/axis_1_positions.csv`, xuất CSV sự kiện tùy chọn |
| Nền tảng | Windows / Linux (SDK và phần cứng phụ thuộc vào hệ thống chạy) |

Một dự án orchestration phần cứng cho các thí nghiệm camera sự kiện, kết hợp:
- Thu thập camera sự kiện EVK5 (Prophesee Metavision stack)
- Điều khiển chuyển động FMC4030
- Điều khiển LED qua serial Arduino
- Giao diện web trigger tối giản (Tornado)

> Giả định: môi trường DLL, hardware và SDK khác nhau theo máy chạy và được suy ra từ mã nguồn dự án; hành vi lệnh cụ thể có thể khác theo hệ điều hành, phiên bản driver và khả dụng runtime.

## 🧠 Tổng Quan

Quy trình end-to-end chính được triển khai trong `app.py`:

1. Tùy chọn xoay chuyển thư mục `data/` hiện có sang thư mục đánh dấu thời gian (`data_YYYYMMDD_HHMMSS`)
2. Kết nối Arduino LED (mặc định `COM4`)
3. Khởi tạo bộ điều khiển CNC qua `cnc/FMC4030Lib-x64-20220329/FMC4030-Dll.dll`
4. Chạy chuỗi LED và chuyển động trục Y
5. Tùy chọn ghi nhận sự kiện EVK5 (hiện đang tắt trong luồng hoạt động chính; xem ghi chú bên dưới)
6. Lưu nhật ký vị trí trục vào `data/axis_1_positions.csv`

### Ảnh chụp tổng quan luồng làm việc

| Giai đoạn | Thành phần | Kết quả |
|---|---|---|
| Trigger | Tornado `/start` | Thực thi chuỗi bất đồng bộ |
| Chuyển động | Bộ điều khiển FMC4030 | Di chuyển trục + truy vấn vị trí |
| Chiếu sáng | Serial Arduino (`'1'` / `'0'`) | Điều khiển trạng thái LED |
| Cảm biến | EVK5 + Metavision | Luồng sự kiện / xuất CSV |
| Lưu trữ | Hệ thống tệp cục bộ | `data/*.csv`, thư mục đã xoay vòng |

Kho này cũng chứa các script camera thay thế/legacy, tiện ích xử lý khung hình hậu kỳ, và mẫu Metavision Python đi kèm.

## ✨ Tính Năng

- Endpoint web Tornado (`/start`) để khởi chạy chuỗi motion/capture bất đồng bộ
- Ghi sự kiện EVK5 với bật kênh trigger (`MAIN`) qua Metavision HAL
- Xuất CSV sự kiện với cả timestamp sự kiện và timestamp hệ thống
- Wrapper điều khiển động cơ FMC4030 dùng `ctypes` và DLL của nhà cung cấp
- Ghi log vị trí chuyển động ra CSV khi trục di chuyển
- Điều khiển LED Arduino qua serial (`'1'`/`'0'`)
- Script tiện ích khung hình (`.npy` kiểm tra shape và `.npy` sang MP4)
- Các ví dụ Metavision trong `python_samples/` kèm theo để tham khảo và thí nghiệm

## 🗂️ Cấu Trúc Dự Án

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
├── motor_system.ini                          # Soft-origin config
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

## 🧰 Yêu cầu môi trường

### Phần cứng

- Camera sự kiện tương thích EVK5 và driver/SDK
- Bộ điều khiển chuyển động tương thích FMC4030 có thể truy cập theo IP/cổng đã cấu hình
- Board Arduino để điều khiển LED

### Phần mềm

- Python 3.x
- Hỗ trợ vendor/runtime cho:
  - Các module Python Prophesee Metavision (`metavision_core`, `metavision_hal`, các module SDK liên quan)
  - Nạp DLL FMC4030 qua Python `ctypes` (`windll` trong `cnc/cnc.py` ngụ ý đường CNC chạy trên Windows)
- Các thư viện Python dùng trên nhiều script:
  - `tornado`, `numpy`, `opencv-python`, `pytz`, `pyserial`
  - Ngăn xếp thay thế: `dv`

### Mặc định môi trường

| Cài đặt | Mặc định | Vị trí |
|---|---|---|
| Cổng serial Arduino | `COM4` | `app.py`, `led.py` |
| IP CNC | `192.168.0.30` | `cnc/cnc.py` |
| Port CNC | `8088` | `cnc/cnc.py` |

Ghi chú:
- `requirements.txt` hoặc `pyproject.toml` không có trong snapshot hiện tại.
- Cổng serial mặc định là `COM4` trong `app.py` và `led.py`.
- Cấu hình mạng CNC mặc định trong `cnc/cnc.py`: IP `192.168.0.30`, port `8088`.

<a id="installation"></a>
## 🔧 Cài đặt

1. Clone kho mã:

```bash
git clone <your-repo-url>
cd nhi_hardware__lachlanchen
```

2. Tạo và kích hoạt môi trường Python:

```bash
python -m venv .venv
# Windows
.venv\Scripts\activate
# Linux/macOS
source .venv/bin/activate
```

3. Cài đặt các dependency Python nền tảng dùng cho các script core:

```bash
pip install tornado numpy opencv-python pytz pyserial
```

4. Cài đặt các dependency SDK camera theo môi trường của bạn:

```bash
# Example placeholder: install Prophesee Metavision Python packages
# Follow your SDK/distribution instructions for your OS and camera version.
```

<a id="usage"></a>
## 🧪 Sử Dụng

### 1) Chuỗi điều phối qua web (chính)

Chạy từ root repository (quan trọng cho đường dẫn tương đối trong `app.py`):

```bash
python app.py
```

Sau đó mở:

```text
http://localhost:8888
```

Nhấn **Start Sequence** để kích hoạt luồng motion/LED.

Tham số tùy chọn đang được phân tích cú pháp bởi app:

```bash
python app.py --record_events True
```

Lưu ý hành vi quan trọng: `start_sequence()` hiện tại đặt lại `record_events = False` ở giai đoạn sau khi thực thi, vì vậy ghi EVK5 có thể vẫn bị tắt nếu chưa chỉnh sửa mã.

### 2) Ghi sự kiện EVK5 (CLI trực tiếp)

```bash
python event_sensor_evk5.py -i "" -d 10 -z Asia/Hong_Kong -o event_output
```

Tham số:
- `-i, --input`: Nguồn đầu vào / đường dẫn cho thiết bị hoặc bản ghi EVK5
- `-d, --duration`: Thời lượng ghi theo giây
- `-z, --timezone`: Nhãn múi giờ cho định dạng timestamp
- `-o, --output`: Tên cơ sở đầu ra (lưu tại `data/<name>.csv`)

### 3) Điều Khiển CNC (CLI trực tiếp)

Chạy từ `cnc/` để đường dẫn DLL tương đối trong `cnc.py` resolve đúng:

```bash
cd cnc
python cnc.py --axis 1 --dir 1 --distance 30 --speed 20
```

Các tùy chọn khác:

```bash
# Set current position as soft origin
python cnc.py --set-origin

# Move to absolute coordinates x,y,z
python cnc.py --move 0,30,0 --speed 20

# Set origin after moving to provided coordinates
python cnc.py --set-origin 0,30,0 --speed 20
```

### 4) Script kiểm tra LED

```bash
python led.py
```

Cập nhật cổng trong mã nếu không dùng `COM4`.

### 5) Tiện ích

```bash
# Print frame-array shape from one folder
python npy_shape.py data_20250101_120000

# Print frame-array shape for all data_* folders in cwd
python npy_shape.py -a
```

`npy2video.py` cung cấp `npy_to_video(npy_file_path, output_video_path, fps=5)` và có thể import hoặc chỉnh sửa cho đường dẫn cục bộ của bạn.

<a id="configuration"></a>
## ⚙️ Cấu Hình

- `motor_system.ini` và `cnc/motor_system.ini`:
  - Lưu trữ tọa độ gốc phần mềm (`ORIGIN` section cho X/Y/Z)
- `app.py`:
  - Cổng serial Arduino: `ArduinoLED(port='COM4')`
  - Đường dẫn DLL CNC: `cnc/FMC4030Lib-x64-20220329/FMC4030-Dll.dll`
- `event_sensor.py` (đường DV):
  - Mặc định `DV_PORT` là `7777`
  - Mặc định `DV_PORT_FRAME` là `7778`

<a id="examples"></a>
## 📸 Ví Dụ

### Ví dụ A: Chạy toàn bộ chuỗi qua web

```bash
python app.py
# visit http://localhost:8888 and press Start Sequence
```

Kết quả dự kiến bao gồm:
- `data/axis_1_positions.csv` (dấu vết vị trí CNC)
- `data/<events>.csv` tùy chọn nếu ghi sự kiện được bật trong luồng hoạt động hiện tại

### Ví dụ B: Ghi sự kiện EVK5 trong 60 giây

```bash
python event_sensor_evk5.py -d 60 -o run_001_events -z Asia/Hong_Kong
```

Kết quả dự kiến:
- `data/run_001_events.csv`

### Ví dụ C: Di chuyển trục Y qua lại từ CLI

```bash
cd cnc
python cnc.py --axis 1 --dir 1 --distance 30 --speed 100
python cnc.py --axis 1 --dir -1 --distance 30 --speed 100
```

## 🧭 Ghi Chú Phát Triển

- Repository hiện tại dường như là không gian nghiên cứu/prototype, với script đang dùng và script đã lưu trữ lẫn lộn.
- Các file sinh ra lớn (`event_output.csv`, `data-0503/`) đã được commit; cân nhắc chiến lược giữ dữ liệu và cập nhật `.gitignore` nếu repo sẽ được phát hành.
- `python_samples/` có các ví dụ SDK camera hữu ích nhưng có thể gồm các dependency không cần thiết cho orchestration cốt lõi.
- Tiềm năng cải thiện chất lượng code:
  - Xử lý boolean của `argparse` trong `app.py` có thể tối ưu hơn (`type=bool` thường gây hiểu nhầm khi parse CLI).
  - Flag ghi sự kiện trong `start_sequence()` hiện tại ghi đè giá trị CLI đầu vào.

<a id="troubleshooting"></a>
## 🛠️ Khắc Phục Sự Cố

- `ImportError: metavision_*` thiếu module:
  - Cài đặt/cấu hình môi trường Python của Prophesee Metavision SDK.
- `ImportError: No module named dv`:
  - Cài đặt gói Python DV nếu dùng đường `event_sensor.py`.
- Lỗi không nạp được DLL CNC:
  - Kiểm tra khả năng tương thích OS và `FMC4030-Dll.dll` có sẵn tại đường dẫn tương đối kỳ vọng.
  - Chạy `cnc.py` từ thư mục `cnc/` hoặc điều chỉnh `dll_path`.
- Lỗi kết nối serial LED:
  - Kiểm tra cổng Arduino (`COM4` so với cổng thực tế).
  - Đảm bảo không có tiến trình khác chiếm dụng thiết bị serial.
- Không có file trong `data/` sau khi chạy web:
  - Kiểm tra quyền ghi và xem luồng ghi sự kiện có được bật hay không.

## 🗺️ Lộ Trình

- Thêm file phụ thuộc (`requirements.txt` hoặc `pyproject.toml`) và phiên bản pinned
- Tách cấu hình runtime (port, IP, đường dẫn DLL, profile tốc độ) ra một file cấu hình thống nhất
- Chuẩn hóa backend camera (EVK5 và DV) sau một lớp giao diện chung, với lựa chọn mode rõ ràng
- Thêm test/mock cho giao diện chuyển động và cảm biến để có thể chạy CI không cần phần cứng
- Thêm logging có cấu trúc và metadata cho từng lần chạy thí nghiệm
- Tạo và duy trì các file README đã dịch trong `i18n/`

## 🤝 Đóng góp

Đóng góp được chào đón cho:
- Cải tiến lớp trừu tượng phần cứng
- Cấu hình tốt hơn và khả năng tái lập
- Mở rộng tài liệu và dịch thuật
- Kiểm tra an toàn và rào chắn vận hành cho điều khiển chuyển động

Quy trình đóng góp đề xuất:
1. Fork và tạo branch tính năng
2. Thực hiện thay đổi tập trung, dễ review
3. Kiểm chứng trên thiết lập phần cứng của bạn
4. Tạo pull request kèm bước tái tạo và log

## Giấy phép

Không có tệp license trong snapshot kho mã này.

Giả định: mọi quyền được bảo lưu cho đến khi dự án thêm tệp giấy phép một cách rõ ràng. Hãy thêm tệp `LICENSE` để xác định điều khoản tái sử dụng.


## ❤️ Support

| Donate | PayPal | Stripe |
| --- | --- | --- |
| [![Donate](https://camo.githubusercontent.com/24a4914f0b42c6f435f9e101621f1e52535b02c225764b2f6cc99416926004b7/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f446f6e6174652d4c617a79696e674172742d3045413545393f7374796c653d666f722d7468652d6261646765266c6f676f3d6b6f2d6669266c6f676f436f6c6f723d7768697465)](https://chat.lazying.art/donate) | [![PayPal](https://camo.githubusercontent.com/d0f57e8b016517a4b06961b24d0ca87d62fdba16e18bbdb6aba28e978dc0ea21/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f50617950616c2d526f6e677a686f754368656e2d3030343537433f7374796c653d666f722d7468652d6261646765266c6f676f3d70617970616c266c6f676f436f6c6f723d7768697465)](https://paypal.me/RongzhouChen) | [![Stripe](https://camo.githubusercontent.com/1152dfe04b6943afe3a8d2953676749603fb9f95e24088c92c97a01a897b4942/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f5374726970652d446f6e6174652d3633354246463f7374796c653d666f722d7468652d6261646765266c6f676f3d737472697065266c6f676f436f6c6f723d7768697465)](https://buy.stripe.com/aFadR8gIaflgfQV6T4fw400) |
