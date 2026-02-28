[English](../README.md) · [العربية](README.ar.md) · [Español](README.es.md) · [Français](README.fr.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Tiếng Việt](README.vi.md) · [中文 (简体)](README.zh-Hans.md) · [中文（繁體）](README.zh-Hant.md) · [Deutsch](README.de.md) · [Русский](README.ru.md)


# Управление аппаратурой NHI и захват событий

![Python](https://img.shields.io/badge/Python-3.x-blue)
![Platform](https://img.shields.io/badge/Platform-Windows%20%2F%20Linux-informational)
![Status](https://img.shields.io/badge/Status-Research%20Prototype-orange)
![Hardware](https://img.shields.io/badge/Hardware-EVK5%20%7C%20FMC4030%20%7C%20Arduino-success)
![UI](https://img.shields.io/badge/Web_UI-Tornado-0ea5e9)

Проект оркестрации аппаратуры для экспериментов с event-камерой, объединяющий:
- захват с event-камеры EVK5 (стек Prophesee Metavision)
- управление движением CNC на базе FMC4030
- управление светодиодом через Arduino по serial
- минимальный веб-интерфейс запуска (Tornado)

> Этот README — первый полный черновик для данного снимка репозитория.
> Предположение: в этой копии ранее не было корневого `README.md`, поэтому документ собран по исходному коду и артефактам анализа пайплайна.

## Обзор

Основной сквозной workflow реализован в `app.py`:

1. При необходимости переносит существующую `data/` в папку с меткой времени (`data_YYYYMMDD_HHMMSS`)
2. Подключается к Arduino LED (по умолчанию `COM4`)
3. Инициализирует CNC-контроллер через `cnc/FMC4030Lib-x64-20220329/FMC4030-Dll.dll`
4. Выполняет последовательность LED и движения по оси Y
5. При необходимости записывает события EVK5 (в текущем активном потоке отключено; см. примечания ниже)
6. Сохраняет логи позиций оси в `data/axis_1_positions.csv`

### Снимок workflow

| Этап | Компонент | Результат |
|---|---|---|
| Триггер | Tornado `/start` | Асинхронное выполнение последовательности |
| Движение | Контроллер FMC4030 | Перемещение оси + опрос позиции |
| Освещение | Arduino serial (`'1'` / `'0'`) | Управление состоянием LED |
| Сенсорика | EVK5 + Metavision | Поток событий / экспорт CSV |
| Хранение | Локальная файловая система | `data/*.csv`, ротация папок |

Репозиторий также содержит альтернативные/устаревшие скрипты камеры, утилиты постобработки кадров и встроенные Python-примеры Metavision.

## Возможности

- Веб-эндпоинт Tornado (`/start`) для асинхронного запуска последовательности движения/захвата
- Запись событий EVK5 с включением trigger-канала (`MAIN`) через Metavision HAL
- Экспорт событий в CSV с временными метками событий и системы
- Обертка управления мотором FMC4030 на `ctypes` и vendor DLL
- Логирование позиций в CSV во время движения оси
- Управление LED через Arduino serial (команды `'1'`/`'0'`)
- Скрипты-утилиты для кадров (проверка формы `.npy` и конвертация `.npy` в MP4)
- Встроенные примеры Metavision в `python_samples/` для экспериментов и справки

## Структура проекта

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

## Требования

### Аппаратное обеспечение

- Event-камера, совместимая с EVK5, и соответствующие драйверы/SDK
- Контроллер движения, совместимый с FMC4030, доступный по настроенным IP/port
- Плата Arduino для управления LED

### Программное обеспечение

- Python 3.x
- Поддержка vendor/runtime для:
  - Python-модулей Prophesee Metavision (`metavision_core`, `metavision_hal`, связанные модули SDK)
  - Загрузки FMC4030 DLL через Python `ctypes` (использование `windll` в `cnc/cnc.py` предполагает Windows для CNC-пути)
- Python-библиотеки, используемые в скриптах:
  - `tornado`, `numpy`, `opencv-python`, `pytz`, `pyserial`
  - Опциональный/альтернативный стек: `dv`

### Значения среды по умолчанию

| Параметр | По умолчанию | Расположение |
|---|---|---|
| Arduino serial port | `COM4` | `app.py`, `led.py` |
| CNC IP | `192.168.0.30` | `cnc/cnc.py` |
| CNC port | `8088` | `cnc/cnc.py` |

Примечания:
- В текущем снимке нет `requirements.txt` или `pyproject.toml`.
- Serial port в `app.py` и `led.py` по умолчанию `COM4`.
- Настройки сети CNC по умолчанию в `cnc/cnc.py`: IP `192.168.0.30`, port `8088`.

## Установка

1. Клонируйте репозиторий:

```bash
git clone <your-repo-url>
cd nhi_hardware__lachlanchen
```

2. Создайте и активируйте Python-окружение:

```bash
python -m venv .venv
# Windows
.venv\Scripts\activate
# Linux/macOS
source .venv/bin/activate
```

3. Установите базовые Python-зависимости, используемые основными скриптами:

```bash
pip install tornado numpy opencv-python pytz pyserial
```

4. Установите зависимости SDK камеры, необходимые в вашей среде:

```bash
# Example placeholder: install Prophesee Metavision Python packages
# Follow your SDK/distribution instructions for your OS and camera version.
```

## Использование

### 1) Веб-оркестрация последовательности (основной режим)

Запускайте из корня репозитория (это важно для относительных путей в `app.py`):

```bash
python app.py
```

Затем откройте:

```text
http://localhost:8888
```

Нажмите **Start Sequence**, чтобы запустить workflow движения/LED.

Опциональный аргумент, который сейчас обрабатывается приложением:

```bash
python app.py --record_events True
```

Важное примечание по поведению: `start_sequence()` позже по ходу выполнения сбрасывает `record_events = False`, поэтому запись EVK5 может оставаться отключенной, если не скорректировать код.

### 2) Запись событий EVK5 (прямой CLI)

```bash
python event_sensor_evk5.py -i "" -d 10 -z Asia/Hong_Kong -o event_output
```

Аргументы:
- `-i, --input`: Источник/путь ввода для устройства EVK5 или записи
- `-d, --duration`: Длительность записи в секундах
- `-z, --timezone`: Метка часового пояса для форматирования временных меток
- `-o, --output`: Базовое имя выходного файла (сохраняется в `data/<name>.csv`)

### 3) Управление CNC (прямой CLI)

Запускайте из `cnc/`, чтобы корректно разрешался относительный путь к DLL в `cnc.py`:

```bash
cd cnc
python cnc.py --axis 1 --dir 1 --distance 30 --speed 20
```

Другие доступные параметры:

```bash
# Set current position as soft origin
python cnc.py --set-origin

# Move to absolute coordinates x,y,z
python cnc.py --move 0,30,0 --speed 20

# Set origin after moving to provided coordinates
python cnc.py --set-origin 0,30,0 --speed 20
```

### 4) Тестовый скрипт LED

```bash
python led.py
```

Обновите порт в коде, если используете не `COM4`.

### 5) Утилиты

```bash
# Print frame-array shape from one folder
python npy_shape.py data_20250101_120000

# Print frame-array shape for all data_* folders in cwd
python npy_shape.py -a
```

`npy2video.py` предоставляет `npy_to_video(npy_file_path, output_video_path, fps=5)`; функцию можно импортировать или адаптировать под ваши локальные пути.

## Конфигурация

- `motor_system.ini` и `cnc/motor_system.ini`:
  - Хранят координаты программного нуля (`ORIGIN` section для X/Y/Z)
- `app.py`:
  - Arduino serial port: `ArduinoLED(port='COM4')`
  - CNC DLL path: `cnc/FMC4030Lib-x64-20220329/FMC4030-Dll.dll`
- `event_sensor.py` (DV path):
  - `DV_PORT` по умолчанию `7777`
  - `DV_PORT_FRAME` по умолчанию `7778`

## Примеры

### Пример A: Запуск полной последовательности через web

```bash
python app.py
# visit http://localhost:8888 and press Start Sequence
```

Ожидаемые результаты:
- `data/axis_1_positions.csv` (трасса позиций CNC)
- Опционально `data/<events>.csv`, если запись событий включена в активном пути

### Пример B: Запись событий EVK5 в течение 60 секунд

```bash
python event_sensor_evk5.py -d 60 -o run_001_events -z Asia/Hong_Kong
```

Ожидаемый результат:
- `data/run_001_events.csv`

### Пример C: Перемещение оси Y туда-обратно из CLI

```bash
cd cnc
python cnc.py --axis 1 --dir 1 --distance 30 --speed 100
python cnc.py --axis 1 --dir -1 --distance 30 --speed 100
```

## Примечания по разработке

- Текущий репозиторий выглядит как исследовательское/прототипное рабочее пространство со смешением активных и архивных скриптов.
- Крупные сгенерированные артефакты (`event_output.csv`, `data-0503/`) закоммичены; если репозиторий будет распространяться, стоит продумать стратегию хранения данных и обновления `.gitignore`.
- `python_samples/` содержит полезные примеры SDK камеры, но может включать зависимости, не обязательные для основной оркестрации.
- Потенциальные улучшения качества кода:
  - Обработку boolean через `argparse` в `app.py` можно улучшить (`type=bool` часто вводит в заблуждение при CLI-парсинге).
  - Текущая логика флага записи событий в `start_sequence()` переопределяет начальное CLI-значение.

## Устранение неполадок

- `ImportError: metavision_*` modules missing:
  - Установите/настройте Python-окружение SDK Metavision.
- `ImportError: No module named dv`:
  - Установите пакет DV для Python, если используете путь `event_sensor.py`.
- Ошибки загрузки CNC DLL:
  - Проверьте совместимость OS и наличие `FMC4030-Dll.dll` по ожидаемому относительному пути.
  - Запускайте `cnc.py` из директории `cnc/` или скорректируйте `dll_path`.
- Ошибки serial-подключения LED:
  - Проверьте назначение порта Arduino (`COM4` и фактический порт).
  - Убедитесь, что устройство serial не занято другим процессом.
- После веб-запуска нет файлов в `data/`:
  - Проверьте права записи и включен ли путь записи событий.

## Дорожная карта

- Добавить манифест зависимостей (`requirements.txt` или `pyproject.toml`) и зафиксированные версии
- Вынести runtime-конфигурацию (ports, IP, DLL path, speed profiles) в единый конфигурационный файл
- Нормализовать бэкенды камеры (EVK5 и DV) за одним интерфейсом с явным выбором режима
- Добавить тесты/моки для интерфейсов движения и сенсоров, чтобы запускать CI без аппаратуры
- Добавить структурированное логирование и метаданные запуска для каждого эксперимента
- Генерировать и поддерживать переведенные README-файлы в `i18n/`

## Вклад

Приветствуются вклады в:
- улучшение аппаратной абстракции
- более качественную конфигурацию и воспроизводимость
- расширение и перевод документации
- проверки безопасности и эксплуатационные защитные ограничения для управления движением

Рекомендуемый процесс вклада:
1. Сделайте fork и создайте ветку фичи
2. Вносите точечные изменения, удобные для ревью
3. Проверьте работу на своей аппаратной конфигурации
4. Отправьте pull request с воспроизводимыми шагами и логами

## Лицензия

В текущем снимке репозитория файл лицензии отсутствует.

Предположение: все права защищены до явного добавления лицензии проекта. Добавьте файл `LICENSE`, чтобы определить условия использования.

## Поддержка

В этом снимке не найдены метаданные о спонсорстве/донатах. Если вы хотите добавить ссылки на поддержку, укажите их здесь и в переведенных README.
