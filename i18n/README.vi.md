[English](../README.md) · [العربية](README.ar.md) · [Español](README.es.md) · [Français](README.fr.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Tiếng Việt](README.vi.md) · [中文 (简体)](README.zh-Hans.md) · [中文（繁體）](README.zh-Hant.md) · [Deutsch](README.de.md) · [Русский](README.ru.md)


# Điều Khiển Phần Cứng NHI và Thu Thập Sự Kiện

![Python](https://img.shields.io/badge/Python-3.x-blue)
![Platform](https://img.shields.io/badge/Platform-Windows%20%2F%20Linux-informational)
![Status](https://img.shields.io/badge/Status-Research%20Prototype-orange)
![Hardware](https://img.shields.io/badge/Hardware-EVK5%20%7C%20FMC4030%20%7C%20Arduino-success)
![UI](https://img.shields.io/badge/Web_UI-Tornado-0ea5e9)

Một dự án điều phối phần cứng cho thí nghiệm camera sự kiện, kết hợp:
- Thu thập camera sự kiện EVK5 (ngăn xếp Prophesee Metavision)
- Điều khiển chuyển động CNC dựa trên FMC4030
- Điều khiển LED qua cổng nối tiếp Arduino
- UI kích hoạt web tối giản (Tornado)

> README này là bản nháp hoàn chỉnh đầu tiên cho snapshot của kho mã này.
> Giả định: không có `README.md` gốc đã tồn tại trước đó trong lần checkout này, vì vậy tài liệu này được xây dựng từ mã nguồn và các hiện vật phân tích pipeline.

## Tổng quan

Quy trình end-to-end chính được triển khai trong `app.py`:

1. Tùy chọn xoay vòng thư mục `data/` hiện có sang thư mục có dấu thời gian (`data_YYYYMMDD_HHMMSS`)
2. Kết nối Arduino LED (mặc định `COM4`)
3. Khởi tạo bộ điều khiển CNC qua `cnc/FMC4030Lib-x64-20220329/FMC4030-Dll.dll`
4. Chạy chuỗi LED và chuyển động trục Y
5. Tùy chọn ghi sự kiện EVK5 (hiện đang bị vô hiệu trong luồng đang hoạt động; xem ghi chú bên dưới)
6. Lưu log vị trí trục vào `data/axis_1_positions.csv`

### Ảnh Chụp Nhanh Quy Trình

| Giai đoạn | Thành phần | Đầu ra |
|---|---|---|
| Kích hoạt | Tornado `/start` | Thực thi chuỗi bất đồng bộ |
| Chuyển động | Bộ điều khiển FMC4030 | Di chuyển trục + thăm dò vị trí |
| Chiếu sáng | Arduino serial (`'1'` / `'0'`) | Điều khiển trạng thái LED |
| Cảm biến | EVK5 + Metavision | Luồng sự kiện / xuất CSV |
| Lưu trữ | Hệ thống tệp cục bộ | `data/*.csv`, thư mục đã xoay vòng |

Kho mã cũng chứa các script camera thay thế/cũ, tiện ích hậu xử lý khung hình và các mẫu Python Metavision đi kèm.

## Tính năng

- Endpoint web Tornado (`/start`) để khởi chạy chuỗi chuyển động/thu thập bất đồng bộ
- Ghi sự kiện EVK5 với bật kênh trigger (`MAIN`) qua Metavision HAL
- Xuất CSV sự kiện với cả timestamp sự kiện và timestamp hệ thống
- Wrapper điều khiển động cơ FMC4030 dùng `ctypes` và DLL của nhà cung cấp
- Ghi log vị trí chuyển động ra CSV trong khi trục di chuyển
- Điều khiển LED Arduino qua serial (lệnh `'1'`/`'0'`)
- Script tiện ích khung hình (kiểm tra shape `.npy` và chuyển `.npy` sang MP4)
- Các ví dụ Metavision `python_samples/` đi kèm để thử nghiệm và tham khảo

## Cấu trúc dự án

```text
.
├── app.py                                   # Bộ điều phối web chính
├── event_sensor_evk5.py                     # Trình ghi EVK5 (Metavision)
├── event_sensor.py                          # Trình ghi thay thế (gói dv)
├── evk5_test.py                             # Trình ghi kiểm thử EVK5 tối giản
├── EventCamera_extTrig_Version 240506_ForEVK5.py
├── led.py                                   # Điều khiển LED Arduino qua serial
├── npy2video.py                             # Chuyển stack frame NPY -> MP4
├── npy_shape.py                             # In kích thước mảng frame
├── motor_system.ini                         # Cấu hình soft-origin
├── cnc/
│   ├── cnc.py                               # Điều khiển FMC4030 + CLI
│   ├── motor_system.ini
│   ├── FMC4030Lib-x64-20220329/
│   │   ├── FMC4030-Dll.dll
│   │   ├── FMC4030-Dll.h
│   │   └── FMC4030-Dll.lib
│   └── archived/                            # Script bộ điều khiển cũ hơn
├── led/
│   └── led.ino                              # Firmware sketch Arduino
├── templates/
│   └── index.html                           # UI nút Start
├── python_samples/                          # Chương trình mẫu Metavision
├── data-0503/                               # Tập dữ liệu thí nghiệm lịch sử
├── i18n/                                    # Dành cho các README đã dịch
└── .auto-readme-work/20260228_231403/      # Hiện vật pipeline README
```

## Điều kiện tiên quyết

### Phần cứng

- Camera sự kiện tương thích EVK5 và driver/SDK
- Bộ điều khiển chuyển động tương thích FMC4030 có thể truy cập tại IP/port đã cấu hình
- Bo mạch Arduino để điều khiển LED

### Phần mềm

- Python 3.x
- Hỗ trợ vendor/runtime cho:
  - Các mô-đun Python Prophesee Metavision (`metavision_core`, `metavision_hal`, các mô-đun SDK liên quan)
  - Nạp DLL FMC4030 qua Python `ctypes` (việc dùng `windll` trong `cnc/cnc.py` ngụ ý đường CNC chạy trên Windows)
- Các thư viện Python được dùng trong nhiều script:
  - `tornado`, `numpy`, `opencv-python`, `pytz`, `pyserial`
  - Ngăn xếp tùy chọn/thay thế: `dv`

### Mặc định môi trường

| Thiết lập | Mặc định | Vị trí |
|---|---|---|
| Cổng serial Arduino | `COM4` | `app.py`, `led.py` |
| CNC IP | `192.168.0.30` | `cnc/cnc.py` |
| CNC port | `8088` | `cnc/cnc.py` |

Ghi chú:
- Không có `requirements.txt` hoặc `pyproject.toml` trong snapshot hiện tại.
- Cổng serial mặc định là `COM4` trong `app.py` và `led.py`.
- Thiết lập mạng CNC mặc định trong `cnc/cnc.py`: IP `192.168.0.30`, port `8088`.

## Cài đặt

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

3. Cài các dependency Python cơ bản dùng bởi script cốt lõi:

```bash
pip install tornado numpy opencv-python pytz pyserial
```

4. Cài các dependency SDK camera cần thiết trong môi trường của bạn:

```bash
# Example placeholder: install Prophesee Metavision Python packages
# Follow your SDK/distribution instructions for your OS and camera version.
```

## Cách sử dụng

### 1) Chuỗi Điều Phối Qua Web (Chính)

Chạy từ thư mục gốc của kho mã (quan trọng để các đường dẫn tương đối trong `app.py` hoạt động đúng):

```bash
python app.py
```

Sau đó mở:

```text
http://localhost:8888
```

Nhấn **Start Sequence** để kích hoạt luồng chuyển động/LED.

Đối số tùy chọn hiện được app parse:

```bash
python app.py --record_events True
```

Lưu ý hành vi quan trọng: `start_sequence()` hiện đặt lại `record_events = False` ở phần sau của quá trình thực thi, nên việc ghi EVK5 có thể vẫn bị vô hiệu trừ khi mã được điều chỉnh.

### 2) Ghi Sự Kiện EVK5 (CLI Trực Tiếp)

```bash
python event_sensor_evk5.py -i "" -d 10 -z Asia/Hong_Kong -o event_output
```

Đối số:
- `-i, --input`: Nguồn/đường dẫn đầu vào cho thiết bị EVK5 hoặc bản ghi
- `-d, --duration`: Thời lượng ghi theo giây
- `-z, --timezone`: Nhãn múi giờ để định dạng timestamp
- `-o, --output`: Tên cơ sở đầu ra (lưu dưới `data/<name>.csv`)

### 3) Điều Khiển CNC (CLI Trực Tiếp)

Chạy từ `cnc/` để đường dẫn DLL tương đối trong `cnc.py` được resolve chính xác:

```bash
cd cnc
python cnc.py --axis 1 --dir 1 --distance 30 --speed 20
```

Các tùy chọn khả dụng khác:

```bash
# Set current position as soft origin
python cnc.py --set-origin

# Move to absolute coordinates x,y,z
python cnc.py --move 0,30,0 --speed 20

# Set origin after moving to provided coordinates
python cnc.py --set-origin 0,30,0 --speed 20
```

### 4) Script Kiểm Thử LED

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

`npy2video.py` cung cấp `npy_to_video(npy_file_path, output_video_path, fps=5)` và có thể được import hoặc chỉnh sửa cho các đường dẫn cục bộ của bạn.

## Cấu hình

- `motor_system.ini` và `cnc/motor_system.ini`:
  - Lưu tọa độ gốc phần mềm (`ORIGIN` section cho X/Y/Z)
- `app.py`:
  - Cổng serial Arduino: `ArduinoLED(port='COM4')`
  - Đường dẫn DLL CNC: `cnc/FMC4030Lib-x64-20220329/FMC4030-Dll.dll`
- `event_sensor.py` (đường DV):
  - `DV_PORT` mặc định `7777`
  - `DV_PORT_FRAME` mặc định `7778`

## Ví dụ

### Ví dụ A: Khởi chạy chuỗi đầy đủ qua web

```bash
python app.py
# visit http://localhost:8888 and press Start Sequence
```

Đầu ra dự kiến gồm:
- `data/axis_1_positions.csv` (vết vị trí CNC)
- Tùy chọn `data/<events>.csv` nếu ghi sự kiện được bật trong đường chạy đang hoạt động

### Ví dụ B: Ghi sự kiện EVK5 trong 60 giây

```bash
python event_sensor_evk5.py -d 60 -o run_001_events -z Asia/Hong_Kong
```

Đầu ra dự kiến:
- `data/run_001_events.csv`

### Ví dụ C: Di chuyển trục Y qua lại từ CLI

```bash
cd cnc
python cnc.py --axis 1 --dir 1 --distance 30 --speed 100
python cnc.py --axis 1 --dir -1 --distance 30 --speed 100
```

## Ghi chú phát triển

- Kho mã hiện tại có vẻ là không gian làm việc nghiên cứu/prototype với script hoạt động lẫn script lưu trữ cũ.
- Các hiện vật sinh ra lớn (`event_output.csv`, `data-0503/`) đã được commit; nên cân nhắc chiến lược lưu giữ dữ liệu và cập nhật `.gitignore` nếu kho mã này được phân phối.
- `python_samples/` chứa các ví dụ SDK camera hữu ích nhưng có thể bao gồm dependency không cần cho điều phối cốt lõi.
- Cải tiến chất lượng mã tiềm năng:
  - Xử lý boolean của `argparse` trong `app.py` có thể được cải thiện (`type=bool` thường gây hiểu nhầm khi parse CLI).
  - Cách xử lý cờ ghi sự kiện trong `start_sequence()` hiện đang ghi đè giá trị CLI ban đầu.

## Khắc phục sự cố

- `ImportError: metavision_*` thiếu mô-đun:
  - Cài đặt/cấu hình môi trường Python Metavision SDK của bạn.
- `ImportError: No module named dv`:
  - Cài gói Python DV nếu dùng luồng `event_sensor.py`.
- Lỗi nạp DLL CNC:
  - Xác nhận tương thích hệ điều hành và `FMC4030-Dll.dll` có sẵn tại đường dẫn tương đối kỳ vọng.
  - Chạy `cnc.py` từ thư mục `cnc/` hoặc điều chỉnh `dll_path`.
- Lỗi kết nối serial LED:
  - Kiểm tra gán cổng Arduino (`COM4` so với cổng thực tế).
  - Đảm bảo không có tiến trình khác đang giữ thiết bị serial.
- Không có tệp trong `data/` sau khi chạy web:
  - Xác minh quyền ghi và liệu đường ghi sự kiện có được bật hay không.

## Lộ trình

- Thêm manifest dependency (`requirements.txt` hoặc `pyproject.toml`) và phiên bản được ghim
- Tách cấu hình runtime (port, IP, đường dẫn DLL, hồ sơ tốc độ) ra tệp cấu hình thống nhất
- Chuẩn hóa backend camera (EVK5 và DV) sau một giao diện với lựa chọn chế độ rõ ràng
- Thêm test/mock cho giao diện chuyển động và cảm biến để bật CI không cần phần cứng
- Thêm logging có cấu trúc và metadata chạy theo từng thí nghiệm
- Tạo và duy trì các tệp README dịch dưới `i18n/`

## Đóng góp

Chào đón đóng góp cho:
- Cải tiến lớp trừu tượng phần cứng
- Cấu hình tốt hơn và khả năng tái lập
- Mở rộng tài liệu và dịch thuật
- Kiểm tra an toàn và lan can vận hành cho điều khiển chuyển động

Quy trình đóng góp đề xuất:
1. Fork và tạo nhánh tính năng
2. Thực hiện thay đổi tập trung, dễ review
3. Xác thực với thiết lập phần cứng của bạn
4. Gửi pull request kèm các bước tái lập và log

## Giấy phép

Không tìm thấy tệp giấy phép trong snapshot kho mã này.

Giả định: mọi quyền được bảo lưu cho đến khi giấy phép dự án được thêm rõ ràng. Hãy thêm tệp `LICENSE` để xác định điều khoản tái sử dụng.

## Hỗ trợ

Không tìm thấy metadata tài trợ/quyên góp trong snapshot này. Nếu muốn thêm liên kết hỗ trợ, hãy bổ sung tại đây và trong các README đã dịch.
