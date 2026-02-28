[English](README.md) · [العربية](i18n/README.ar.md) · [Español](i18n/README.es.md) · [Français](i18n/README.fr.md) · [日本語](i18n/README.ja.md) · [한국어](i18n/README.ko.md) · [Tiếng Việt](i18n/README.vi.md) · [中文 (简体)](i18n/README.zh-Hans.md) · [中文（繁體）](i18n/README.zh-Hant.md) · [Deutsch](i18n/README.de.md) · [Русский](i18n/README.ru.md)


[![LazyingArt banner](https://github.com/lachlanchen/lachlanchen/raw/main/figs/banner.png)](https://github.com/lachlanchen/lachlanchen/blob/main/figs/banner.png)

# NHI Hardware Control and Event Capture

![Python](https://img.shields.io/badge/Python-3.x-blue)
![Platform](https://img.shields.io/badge/Platform-Windows%20%2F%20Linux-informational)
![Status](https://img.shields.io/badge/Status-Research%20Prototype-orange)
![Hardware](https://img.shields.io/badge/Hardware-EVK5%20%7C%20FMC4030%20%7C%20Arduino-success)
![UI](https://img.shields.io/badge/Web_UI-Tornado-0ea5e9)
![Docs](https://img.shields.io/badge/Docs-English%20%2B%20i18n-0f766e)

## 📖 Quick Navigation

| Use this section | Purpose |
|---|---|
| [Installation](#installation) | Prepare environment and dependencies |
| [Usage](#usage) | Run the web orchestrator and CLI workflows |
| [Configuration](#configuration) | Tune serial, networking, and defaults |
| [Examples](#examples) | Run practical command examples |
| [Troubleshooting](#troubleshooting) | Fix common setup issues |

## 🧭 Project at a Glance

| Focus | Details |
|---|---|
| Mission | Coordinate event capture, motion control, and LED signaling for repeatable lab workflows |
| Core Entry | `app.py` (Tornado web trigger + async sequence orchestration) |
| Key Inputs | EVK5 event stream, FMC4030 axis control, Arduino serial commands |
| Primary Outputs | `data/axis_1_positions.csv`, optional event CSV exports |
| Platforms | Windows / Linux (SDK and hardware availability dependent) |

A hardware-orchestration project for event-camera experiments combining:
- EVK5 event camera capture (Prophesee Metavision stack)
- FMC4030-based CNC motion control
- Arduino serial LED control
- Minimal web trigger UI (Tornado)

> Assumption: hardware, DLL, and SDK environments vary by host and are inferred from project code; exact command behavior may differ by OS, driver versions, and runtime availability.

## 🧠 Overview

The primary end-to-end workflow is implemented in `app.py`:

1. Optionally rotate existing `data/` into a timestamped folder (`data_YYYYMMDD_HHMMSS`)
2. Connect to Arduino LED (default `COM4`)
3. Initialize CNC controller through `cnc/FMC4030Lib-x64-20220329/FMC4030-Dll.dll`
4. Run LED and Y-axis motion sequence
5. Optionally record EVK5 events (currently disabled in active flow; see notes below)
6. Save axis position logs to `data/axis_1_positions.csv`

### Workflow Snapshot

| Stage | Component | Output |
|---|---|---|
| Trigger | Tornado `/start` | Async sequence execution |
| Motion | FMC4030 controller | Axis movement + position polling |
| Lighting | Arduino serial (`'1'` / `'0'`) | LED state control |
| Sensing | EVK5 + Metavision | Event stream / CSV export |
| Persistence | Local filesystem | `data/*.csv`, rotated folders |

The repository also contains alternative/legacy camera scripts, utilities for frame post-processing, and bundled Metavision Python samples.

## ✨ Features

- Tornado web endpoint (`/start`) to launch a motion/capture sequence asynchronously
- EVK5 event recording with trigger channel enablement (`MAIN`) via Metavision HAL
- CSV export of events with both event and system timestamps
- FMC4030 motor control wrapper using `ctypes` and vendor DLL
- Motion-position logging to CSV during axis movement
- Arduino LED serial control (`'1'`/`'0'` commands)
- Frame utility scripts (`.npy` shape inspection and `.npy` to MP4 conversion)
- Bundled `python_samples/` Metavision examples for experimentation and reference

## 🗂️ Project Structure

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

## 🧰 Prerequisites

### Hardware

- EVK5-compatible event camera and drivers/SDK
- FMC4030-compatible motion controller reachable at configured IP/port
- Arduino board for LED control

### Software

- Python 3.x
- Vendor/runtime support for:
  - Prophesee Metavision Python modules (`metavision_core`, `metavision_hal`, related SDK modules)
  - FMC4030 DLL loading via Python `ctypes` (`windll` usage in `cnc/cnc.py` implies Windows for CNC path)
- Python libraries used across scripts:
  - `tornado`, `numpy`, `opencv-python`, `pytz`, `pyserial`
  - Optional/alternative stack: `dv`

### Environment Defaults

| Setting | Default | Location |
|---|---|---|
| Arduino serial port | `COM4` | `app.py`, `led.py` |
| CNC IP | `192.168.0.30` | `cnc/cnc.py` |
| CNC port | `8088` | `cnc/cnc.py` |

Notes:
- There is no `requirements.txt` or `pyproject.toml` in the current snapshot.
- Serial port defaults to `COM4` in `app.py` and `led.py`.
- Default CNC network settings in `cnc/cnc.py`: IP `192.168.0.30`, port `8088`.

## 🔧 Installation

1. Clone repository:

```bash
git clone <your-repo-url>
cd nhi_hardware__lachlanchen
```

2. Create and activate a Python environment:

```bash
python -m venv .venv
# Windows
.venv\Scripts\activate
# Linux/macOS
source .venv/bin/activate
```

3. Install baseline Python dependencies used by core scripts:

```bash
pip install tornado numpy opencv-python pytz pyserial
```

4. Install camera SDK dependencies required in your environment:

```bash
# Example placeholder: install Prophesee Metavision Python packages
# Follow your SDK/distribution instructions for your OS and camera version.
```

## 🧪 Usage

### 1) Web-Orchestrated Sequence (Primary)

Run from repository root (important for relative paths in `app.py`):

```bash
python app.py
```

Then open:

```text
http://localhost:8888
```

Click **Start Sequence** to trigger the motion/LED workflow.

Optional argument currently parsed by app:

```bash
python app.py --record_events True
```

Important behavior note: `start_sequence()` currently resets `record_events = False` later in execution, so EVK5 recording may remain disabled unless code is adjusted.

### 2) EVK5 Event Recording (Direct CLI)

```bash
python event_sensor_evk5.py -i "" -d 10 -z Asia/Hong_Kong -o event_output
```

Arguments:
- `-i, --input`: Input source/path for EVK5 device or recording
- `-d, --duration`: Recording duration in seconds
- `-z, --timezone`: Timezone label for timestamp formatting
- `-o, --output`: Output base name (saved under `data/<name>.csv`)

### 3) CNC Control (Direct CLI)

Run from `cnc/` so relative DLL path in `cnc.py` resolves correctly:

```bash
cd cnc
python cnc.py --axis 1 --dir 1 --distance 30 --speed 20
```

Other available options:

```bash
# Set current position as soft origin
python cnc.py --set-origin

# Move to absolute coordinates x,y,z
python cnc.py --move 0,30,0 --speed 20

# Set origin after moving to provided coordinates
python cnc.py --set-origin 0,30,0 --speed 20
```

### 4) LED Test Script

```bash
python led.py
```

Update port in code if not using `COM4`.

### 5) Utilities

```bash
# Print frame-array shape from one folder
python npy_shape.py data_20250101_120000

# Print frame-array shape for all data_* folders in cwd
python npy_shape.py -a
```

`npy2video.py` provides `npy_to_video(npy_file_path, output_video_path, fps=5)` and can be imported or edited for your local paths.

## ⚙️ Configuration

- `motor_system.ini` and `cnc/motor_system.ini`:
  - Persist software origin coordinates (`ORIGIN` section for X/Y/Z)
- `app.py`:
  - Arduino serial port: `ArduinoLED(port='COM4')`
  - CNC DLL path: `cnc/FMC4030Lib-x64-20220329/FMC4030-Dll.dll`
- `event_sensor.py` (DV path):
  - `DV_PORT` default `7777`
  - `DV_PORT_FRAME` default `7778`

## 📸 Examples

### Example A: Start full sequence via web

```bash
python app.py
# visit http://localhost:8888 and press Start Sequence
```

Expected outputs include:
- `data/axis_1_positions.csv` (CNC position trace)
- Optional `data/<events>.csv` if event recording is enabled in active path

### Example B: Record EVK5 events for 60 seconds

```bash
python event_sensor_evk5.py -d 60 -o run_001_events -z Asia/Hong_Kong
```

Expected output:
- `data/run_001_events.csv`

### Example C: Move Y axis back-and-forth from CLI

```bash
cd cnc
python cnc.py --axis 1 --dir 1 --distance 30 --speed 100
python cnc.py --axis 1 --dir -1 --distance 30 --speed 100
```

## 🧭 Development Notes

- Current repository appears to be a research/prototyping workspace with mixed active and archived scripts.
- Large generated artifacts (`event_output.csv`, `data-0503/`) are committed; consider a data retention strategy and `.gitignore` updates if this repository will be distributed.
- `python_samples/` contains useful camera SDK examples but may include dependencies not required for core orchestration.
- Potential code quality improvement:
  - `argparse` boolean handling in `app.py` can be improved (`type=bool` is often misleading in CLI parsing).
  - Event recording flag handling in `start_sequence()` currently overrides initial CLI value.

## 🛠️ Troubleshooting

- `ImportError: metavision_*` modules missing:
  - Install/configure your Metavision SDK Python environment.
- `ImportError: No module named dv`:
  - Install DV Python package if using `event_sensor.py` path.
- CNC DLL load failures:
  - Confirm OS compatibility and that `FMC4030-Dll.dll` is available at expected relative path.
  - Run `cnc.py` from `cnc/` directory or adjust `dll_path`.
- LED serial connection errors:
  - Check Arduino port assignment (`COM4` vs actual port).
  - Ensure no other process is holding the serial device.
- No files in `data/` after web run:
  - Verify write permissions and whether event recording path is enabled.

## 🗺️ Roadmap

- Add dependency manifest (`requirements.txt` or `pyproject.toml`) and pinned versions
- Externalize runtime config (ports, IP, DLL path, speed profiles) to a unified config file
- Normalize camera backends (EVK5 and DV) behind one interface with clear mode selection
- Add tests/mocks for motion and sensor interfaces to enable CI without hardware
- Add structured logging and run metadata per experiment
- Generate and maintain translated README files under `i18n/`

## 🤝 Contributing

Contributions are welcome for:
- Hardware abstraction improvements
- Better configuration and reproducibility
- Documentation expansion and translation
- Safety checks and operational guardrails for motion control

Suggested contribution flow:
1. Fork and create a feature branch
2. Make focused, reviewable changes
3. Validate against your hardware setup
4. Submit a pull request with reproducible steps and logs

## ❤️ Support

| Donate | PayPal | Stripe |
| --- | --- | --- |
| [![Donate](https://camo.githubusercontent.com/24a4914f0b42c6f435f9e101621f1e52535b02c225764b2f6cc99416926004b7/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f446f6e6174652d4c617a79696e674172742d3045413545393f7374796c653d666f722d7468652d6261646765266c6f676f3d6b6f2d6669266c6f676f436f6c6f723d7768697465)](https://chat.lazying.art/donate) | [![PayPal](https://camo.githubusercontent.com/d0f57e8b016517a4b06961b24d0ca87d62fdba16e18bbdb6aba28e978dc0ea21/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f50617950616c2d526f6e677a686f754368656e2d3030343537433f7374796c653d666f722d7468652d6261646765266c6f676f3d70617970616c266c6f676f436f6c6f723d7768697465)](https://paypal.me/RongzhouChen) | [![Stripe](https://camo.githubusercontent.com/1152dfe04b6943afe3a8d2953676749603fb9f95e24088c92c97a01a897b4942/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f5374726970652d446f6e6174652d3633354246463f7374796c653d666f722d7468652d6261646765266c6f676f3d737472697065266c6f676f436f6c6f723d7768697465)](https://buy.stripe.com/aFadR8gIaflgfQV6T4fw400) |

## License

No license file is present in this repository snapshot.

Assumption: all rights are reserved until a project license is explicitly added. Add a `LICENSE` file to define reuse terms.
