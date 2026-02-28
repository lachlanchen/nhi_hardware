[English](../README.md) · [العربية](README.ar.md) · [Español](README.es.md) · [Français](README.fr.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Tiếng Việt](README.vi.md) · [中文 (简体)](README.zh-Hans.md) · [中文（繁體）](README.zh-Hant.md) · [Deutsch](README.de.md) · [Русский](README.ru.md)


# NHI 하드웨어 제어 및 이벤트 캡처

![Python](https://img.shields.io/badge/Python-3.x-blue)
![Platform](https://img.shields.io/badge/Platform-Windows%20%2F%20Linux-informational)
![Status](https://img.shields.io/badge/Status-Research%20Prototype-orange)
![Hardware](https://img.shields.io/badge/Hardware-EVK5%20%7C%20FMC4030%20%7C%20Arduino-success)
![UI](https://img.shields.io/badge/Web_UI-Tornado-0ea5e9)

다음 요소를 결합한 이벤트 카메라 실험용 하드웨어 오케스트레이션 프로젝트입니다.
- EVK5 이벤트 카메라 캡처 (Prophesee Metavision 스택)
- FMC4030 기반 CNC 모션 제어
- Arduino 시리얼 LED 제어
- 최소 웹 트리거 UI (Tornado)

> 이 README는 현재 저장소 스냅샷 기준 첫 번째 완성 초안입니다.
> 가정: 이 체크아웃에는 기존 루트 `README.md`가 없었으며, 본 문서는 소스 코드와 파이프라인 분석 산출물을 기반으로 작성되었습니다.

## 개요

주요 엔드투엔드 워크플로우는 `app.py`에 구현되어 있습니다.

1. 기존 `data/`를 타임스탬프 폴더(`data_YYYYMMDD_HHMMSS`)로 선택적으로 회전
2. Arduino LED에 연결 (기본값 `COM4`)
3. `cnc/FMC4030Lib-x64-20220329/FMC4030-Dll.dll`을 통해 CNC 컨트롤러 초기화
4. LED 및 Y축 모션 시퀀스 실행
5. EVK5 이벤트를 선택적으로 기록 (현재 활성 흐름에서는 비활성화됨, 아래 참고)
6. 축 위치 로그를 `data/axis_1_positions.csv`에 저장

### 워크플로우 스냅샷

| 단계 | 구성 요소 | 출력 |
|---|---|---|
| 트리거 | Tornado `/start` | 비동기 시퀀스 실행 |
| 모션 | FMC4030 컨트롤러 | 축 이동 + 위치 폴링 |
| 조명 | Arduino 시리얼 (`'1'` / `'0'`) | LED 상태 제어 |
| 센싱 | EVK5 + Metavision | 이벤트 스트림 / CSV 내보내기 |
| 저장 | 로컬 파일시스템 | `data/*.csv`, 회전된 폴더 |

이 저장소에는 대체/레거시 카메라 스크립트, 프레임 후처리 유틸리티, 번들된 Metavision Python 샘플도 포함되어 있습니다.

## 기능

- 모션/캡처 시퀀스를 비동기로 시작하는 Tornado 웹 엔드포인트 (`/start`)
- Metavision HAL을 통한 트리거 채널(`MAIN`) 활성화 기반 EVK5 이벤트 기록
- 이벤트 타임스탬프와 시스템 타임스탬프를 모두 포함한 CSV 내보내기
- `ctypes`와 벤더 DLL을 사용하는 FMC4030 모터 제어 래퍼
- 축 이동 중 모션 위치를 CSV로 로깅
- Arduino LED 시리얼 제어 (`'1'`/`'0'` 명령)
- 프레임 유틸리티 스크립트 (`.npy` shape 확인, `.npy` -> MP4 변환)
- 실험 및 참고용으로 번들된 `python_samples/` Metavision 예제

## 프로젝트 구조

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

## 사전 요구사항

### 하드웨어

- EVK5 호환 이벤트 카메라 및 드라이버/SDK
- 설정된 IP/포트로 접근 가능한 FMC4030 호환 모션 컨트롤러
- LED 제어용 Arduino 보드

### 소프트웨어

- Python 3.x
- 다음 벤더/런타임 지원:
  - Prophesee Metavision Python 모듈 (`metavision_core`, `metavision_hal`, 관련 SDK 모듈)
  - Python `ctypes`를 통한 FMC4030 DLL 로딩 (`cnc/cnc.py`의 `windll` 사용은 CNC 경로에서 Windows를 시사)
- 스크립트 전반에서 사용하는 Python 라이브러리:
  - `tornado`, `numpy`, `opencv-python`, `pytz`, `pyserial`
  - 선택/대체 스택: `dv`

### 기본 환경값

| 설정 | 기본값 | 위치 |
|---|---|---|
| Arduino 시리얼 포트 | `COM4` | `app.py`, `led.py` |
| CNC IP | `192.168.0.30` | `cnc/cnc.py` |
| CNC 포트 | `8088` | `cnc/cnc.py` |

참고:
- 현재 스냅샷에는 `requirements.txt` 또는 `pyproject.toml`이 없습니다.
- 시리얼 포트 기본값은 `app.py`와 `led.py`에서 `COM4`입니다.
- `cnc/cnc.py`의 기본 CNC 네트워크 설정: IP `192.168.0.30`, 포트 `8088`.

## 설치

1. 저장소 클론:

```bash
git clone <your-repo-url>
cd nhi_hardware__lachlanchen
```

2. Python 환경 생성 및 활성화:

```bash
python -m venv .venv
# Windows
.venv\Scripts\activate
# Linux/macOS
source .venv/bin/activate
```

3. 코어 스크립트에서 사용하는 기본 Python 의존성 설치:

```bash
pip install tornado numpy opencv-python pytz pyserial
```

4. 사용 환경에 필요한 카메라 SDK 의존성 설치:

```bash
# Example placeholder: install Prophesee Metavision Python packages
# Follow your SDK/distribution instructions for your OS and camera version.
```

## 사용 방법

### 1) 웹 오케스트레이션 시퀀스 (주요 경로)

저장소 루트에서 실행하세요 (`app.py`의 상대 경로 해석에 중요).

```bash
python app.py
```

그다음 열기:

```text
http://localhost:8888
```

**Start Sequence**를 클릭하면 모션/LED 워크플로우가 트리거됩니다.

현재 앱에서 파싱하는 선택 인수:

```bash
python app.py --record_events True
```

중요 동작 참고: `start_sequence()`가 실행 중 후반에 `record_events = False`로 재설정하므로, 코드를 조정하지 않으면 EVK5 기록이 계속 비활성 상태일 수 있습니다.

### 2) EVK5 이벤트 기록 (직접 CLI)

```bash
python event_sensor_evk5.py -i "" -d 10 -z Asia/Hong_Kong -o event_output
```

인수:
- `-i, --input`: EVK5 디바이스 또는 녹화 입력 소스/경로
- `-d, --duration`: 기록 시간(초)
- `-z, --timezone`: 타임스탬프 포맷용 타임존 라벨
- `-o, --output`: 출력 기본 이름 (`data/<name>.csv`로 저장)

### 3) CNC 제어 (직접 CLI)

`cnc.py`의 상대 DLL 경로가 올바르게 해석되도록 `cnc/`에서 실행하세요.

```bash
cd cnc
python cnc.py --axis 1 --dir 1 --distance 30 --speed 20
```

기타 사용 가능한 옵션:

```bash
# Set current position as soft origin
python cnc.py --set-origin

# Move to absolute coordinates x,y,z
python cnc.py --move 0,30,0 --speed 20

# Set origin after moving to provided coordinates
python cnc.py --set-origin 0,30,0 --speed 20
```

### 4) LED 테스트 스크립트

```bash
python led.py
```

`COM4`를 사용하지 않는 경우 코드에서 포트를 수정하세요.

### 5) 유틸리티

```bash
# Print frame-array shape from one folder
python npy_shape.py data_20250101_120000

# Print frame-array shape for all data_* folders in cwd
python npy_shape.py -a
```

`npy2video.py`는 `npy_to_video(npy_file_path, output_video_path, fps=5)`를 제공하며, 로컬 경로에 맞게 import하여 사용하거나 수정할 수 있습니다.

## 구성

- `motor_system.ini` 및 `cnc/motor_system.ini`:
  - 소프트웨어 원점 좌표를 유지 (`ORIGIN` 섹션의 X/Y/Z)
- `app.py`:
  - Arduino 시리얼 포트: `ArduinoLED(port='COM4')`
  - CNC DLL 경로: `cnc/FMC4030Lib-x64-20220329/FMC4030-Dll.dll`
- `event_sensor.py` (DV 경로):
  - `DV_PORT` 기본값 `7777`
  - `DV_PORT_FRAME` 기본값 `7778`

## 예시

### 예시 A: 웹에서 전체 시퀀스 시작

```bash
python app.py
# visit http://localhost:8888 and press Start Sequence
```

예상 출력:
- `data/axis_1_positions.csv` (CNC 위치 추적)
- 활성 경로에서 이벤트 기록이 켜진 경우 선택적으로 `data/<events>.csv`

### 예시 B: 60초 동안 EVK5 이벤트 기록

```bash
python event_sensor_evk5.py -d 60 -o run_001_events -z Asia/Hong_Kong
```

예상 출력:
- `data/run_001_events.csv`

### 예시 C: CLI에서 Y축 왕복 이동

```bash
cd cnc
python cnc.py --axis 1 --dir 1 --distance 30 --speed 100
python cnc.py --axis 1 --dir -1 --distance 30 --speed 100
```

## 개발 노트

- 현재 저장소는 활성 스크립트와 아카이브 스크립트가 혼재된 연구/프로토타이핑 작업공간으로 보입니다.
- 대용량 생성 산출물(`event_output.csv`, `data-0503/`)이 커밋되어 있습니다. 이 저장소를 배포할 계획이라면 데이터 보존 전략과 `.gitignore` 업데이트를 고려하세요.
- `python_samples/`에는 유용한 카메라 SDK 예제가 있지만, 코어 오케스트레이션에 필요하지 않은 의존성이 포함될 수 있습니다.
- 잠재적 코드 품질 개선점:
  - `app.py`의 `argparse` 불리언 처리 개선 가능 (`type=bool`은 CLI 파싱에서 오해를 부르기 쉬움)
  - `start_sequence()`의 이벤트 기록 플래그 처리가 현재 초기 CLI 값을 덮어씀

## 문제 해결

- `ImportError: metavision_*` 모듈 누락:
  - Metavision SDK Python 환경을 설치/구성하세요.
- `ImportError: No module named dv`:
  - `event_sensor.py` 경로를 사용할 경우 DV Python 패키지를 설치하세요.
- CNC DLL 로드 실패:
  - OS 호환성과 `FMC4030-Dll.dll`의 예상 상대 경로 존재 여부를 확인하세요.
  - `cnc/` 디렉터리에서 `cnc.py`를 실행하거나 `dll_path`를 조정하세요.
- LED 시리얼 연결 오류:
  - Arduino 포트 할당(`COM4`와 실제 포트)을 확인하세요.
  - 다른 프로세스가 시리얼 디바이스를 점유하지 않았는지 확인하세요.
- 웹 실행 후 `data/`에 파일이 없음:
  - 쓰기 권한 및 이벤트 기록 경로 활성 여부를 확인하세요.

## 로드맵

- 의존성 매니페스트(`requirements.txt` 또는 `pyproject.toml`) 추가 및 버전 고정
- 런타임 설정(포트, IP, DLL 경로, 속도 프로파일)을 통합 설정 파일로 외부화
- 카메라 백엔드(EVK5, DV)를 하나의 인터페이스로 정규화하고 모드 선택을 명확화
- 하드웨어 없이 CI를 가능하게 하기 위한 모션/센서 인터페이스 테스트 및 목(mock) 추가
- 실험별 구조화된 로깅과 실행 메타데이터 추가
- `i18n/` 아래 번역 README 생성 및 유지

## 기여

다음 항목에 대한 기여를 환영합니다.
- 하드웨어 추상화 개선
- 더 나은 구성 관리와 재현성
- 문서 확장 및 번역
- 모션 제어용 안전 점검 및 운영 가드레일

권장 기여 흐름:
1. 포크 후 기능 브랜치 생성
2. 범위가 명확하고 리뷰 가능한 변경 적용
3. 하드웨어 환경에서 검증
4. 재현 가능한 절차와 로그를 포함해 풀 리퀘스트 제출

## 라이선스

현재 저장소 스냅샷에는 라이선스 파일이 없습니다.

가정: 프로젝트 라이선스가 명시적으로 추가되기 전까지 모든 권리는 보유됩니다. 재사용 조건 정의를 위해 `LICENSE` 파일을 추가하세요.

## 지원

현재 스냅샷에서는 스폰서/기부 메타데이터를 찾을 수 없었습니다. 지원 링크를 포함하려면 여기에 추가하고 번역 README에도 반영하세요.
